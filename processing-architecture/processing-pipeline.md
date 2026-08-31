# Obdelovalni tok

Chloros1.2.0 uporablja obdelovalni tok s 4 niti, ki deluje kot stopnjevana tekoča linija. Vsaka nit obdeluje ločeno fazo delovnega toka, tako da je lahko več slik hkrati v obdelavi na različnih stopnjah.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Arhitektura poteka obdelave

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Vsaka slika poteka skozi vsa štiri niti po vrstnem redu. Pri večnitni obdelavi Chloros+ več slik hkrati zaseda različne niti – medtem ko nit 3 obdeluje eno sliko, nit 1 lahko zazna naslednjo, nit 2 kalibrira drugo, nit 4 pa končano sliko zapiše na disk.

Napredek se poroča po posameznih niti in se prenaša prek dogodkov Server-Sent Events (backend jih objavlja na `/api/events`). V prikazu napredka v realnem času v CLI so štiri stopnje označene kot **Zaznavanje, Analiziranje, Obdelava, Izvoz**.***

## Podrobnosti o niti

### Nit 1: Zaznavanje

**Namen**: Naložiti slike in zaznati kalibracijske cilje.

* Bere slikovne datoteke z diska — pare Survey3 `.raw`+`.jpg`, posnetki LATTICE `.tif`/`.tiff` ter `.dng`
* Izvleče metapodatke EXIF (GPS, model kamere, časovni žigi, osvetlitev)
* Zazna kalibracijske tarče: geometrijo tarč z oznakami ArUco za posnetke LATTICE ter klasični ploščati detektor za fotografije kalibracijskih tarč Survey3
* Izhodi: podatki o sliki + metapodatki + rezultati zaznavanja tarč

Predvsem niti, vezane na vhodno-izhodne operacije in procesor.

### Niti 2: Kalibracija

**Namen**: Izračun kalibracijskih parametrov iz zaznanih ciljev.

* Izračuna kalibracijske koeficiente odbojnosti iz slik ciljev
* Izračuna parametre korekcije vinjetiranja
* Določi kalibracijske krivulje za posamezne pasove
* Izhodi: kalibracijski parametri za vsako sliko

Računski niz, vezan na CPU. Niz 3 čaka na njega, ko je omogočena kalibracija odbojnosti, tako da so njegovi koeficienti pripravljeni, preden se obdela katera koli slika.

### Nit 3: Obdelava (GPU)

**Namen**: Uporaba popravkov in izračun indeksov vegetacije.**To je računsko najbolj zahtevna nit.*** **Debayering**: pretvori surove podatke Bayer v večkanalne slike
  * Standardno (hitro, srednja kakovost) — privzeto, `--debayer standard`
  * Texture Aware (počasno, najvišja kakovost) — samo za Chloros+, `--debayer texture-aware`, uporablja model za odstranjevanje šuma z umetno inteligenco/strojnim učenjem
  * Posnetki LATTICE mono (M3M) so enopasovni: pri njih se preskočita koraka demosaicinga in uravnoteženja beline (z enovrstičnim dnevniškim sporočilom), medtem ko se pri vseh slikah M3C/Bayer v istem ciklu ti koraki še vedno izvedejo
* **Korekcija vinjetiranja**: izvede korekcijo vinjetiranja objektiva po celotni sliki
* **Kalibracija odbojnosti**: uporabi kalibracijske koeficiente za pretvorbo v vrednosti odbojnosti
* **Izračun indeksov**: izračuna vegetacijske indekse (NDVI, NDRE, GNDVI, …)
* Izhodi: obdelani podatki slike, pripravljeni za izvoz

Ta niti najbolj koristi pospešitev s pomočjo grafičnega procesorja (GPU), in prav to je niti, ki jo [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) optimizira.

### Niti 4: Izvoz

**Namen**: Zapiše obdelane slike na disk.

* Zapiše izhodne datoteke v izbrani obliki — `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* V izhodne datoteke vključi metapodatke (GPS, časovni žigi, parametri obdelave)
* Izhodne datoteke razvrsti v projektni mapi kot `<camera>/<format>/<Product>_Images/` — na primer `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Izvožene datoteke ohranijo ime izvorne datoteke; mapa identificira izdelek.**
* Pri zajemih LATTICE se lahko en izvorni okvir razširi v več izdelkov (Debayered, Preview, Radiance, Reflectance, Index), od katerih je vsak v svoji mapi izdelka
* Izhodi: končne datoteke na disku

Predvsem niti, vezane na vhodno-izhodne operacije — shranjevanje na SSD-ju jih opazno izboljša.

***

## Pod pokrovom: izvajalci

Znotraj niti 3 je delo na posamezni sliki vzporedno izvedeno s standardnim `concurrent.futures` iz paketa »Python«:

* **Strategije za GPU**(`GPU_SINGLE`, `GPU_PARALLEL`) uporabljajo `ProcessPoolExecutor` z metodo**spawn** — vsak delavec je ločen proces z lastnim CUDA-kontekstom (`fork` bi podedoval inicializirano CUDA-stanje starševskega procesa in s tem poškodoval podrejene procese)
* **`CPU_PARALLEL`** uporablja `ThreadPoolExecutor` — NumPy in OpenCV sprostita GIL, zato zadostujejo niti
* Naprave Jetson z 8 GB ali manj skupnega pomnilnika RAM izvrševalca v celoti preskočijo in obdelujejo v okviru istega procesa, zaporedno
* Texture Aware na grafičnem procesorju z manj kot 7 GB VRAM prav tako teče zaporedno — model za odstranjevanje šuma se ne more prilagoditi več kot enkrat

Chlorosne uporablja nobenega distribucijskega okvira tretjih oseb (kot je na primer Ray). Glejte [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) za informacije o tem, kako se izbereta strategija in število delavcev.

***

## Zaporedna obdelava v primerjavi s potekom po poti

### Prosti način (zaporedni)

V brezplačni različici programa Chloros se slike obdelujejo **po ena naenkrat**, zaporedno skozi vse štiri stopnje:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

GUI v prostem načinu prikazuje poenostavljen kazalnik napredka; njegove zaporedne faze so označene kot **Target Detection**(Zaznavanje ciljev) in nato**Processing** (Obdelava).

### Način „Chloros“ (vzporedna obdelava)

Z licenco „Chloros“ vsi štirje niti delujejo **vzporedno** na različnih slikah:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

Napredovalna vrstica v grafičnem uporabniškem vmesniku prikazuje štiri stopnje; če z miško preletite vrstico, si lahko ogledate napredek po posameznih niti. V storitvi »CLI« se iste štiri stopnje prenašajo v živo kot **Detecting, Analyzing, Processing, Exporting**.

{% hint style="info" %}
**Ena oznaka, dve imeni.** V datoteki »CLI« se faza 3 imenuje _Obdelava_. V toku napredka v premium načinu na strežniški strani – tistem, ki ga prikazuje napredovalna vrstica v grafičnem vmesniku – je ista faza označena kot _Kalibriranje_. Gre za isto nit, ki opravlja isto delo (nit 3: debayer, popravki, indeksi).
{% endhint %}

{% hint style="success" %}
**Obdelava v cevnem toku z Chloros+** je lahko 3–5-krat hitrejša od zaporedne obdelave, odvisno od vaše strojne opreme in velikosti niza podatkov. Največje pospešitev je mogoče doseči na sistemih s hitrimi grafičnimi procesorji (GPU) in SSD-ji.
{% endhint %}

***

## Napredek izvoza v niti 4

Nit za izvoz ima lastno sledenje napredka, ki ga lahko preverjate ločeno:**CLI:**

```bash
chloros-cli export-status
```

**SDK:**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Obdelava je zaključena, ko nit 4 doseže 100 %.

{% hint style="info" %}
**Izvedba, ki ne zapiše nobene slike, je neuspešna.**Ob uspehu `chloros-cli process` poroča, koliko slikovnih izdelkov je zapisal (`Image products written: N`). Če so bili zahtevani izdelki, a ni bil zapišan**noben**— le `project.json` in `calibration_data.json` — program »CLI« izpiše `Processing finished but wrote no image products.` in**se zaključi z vrednostjo, ki ni nič**, pri čemer navede ime projektne mape in običajne vzroke (vhodna mapa ni bila prepoznana kot zajem – preverite razpored in `--input-level` – ali pa noben od zahtevanih izdelkov ni bil primeren za te kamere). Skripti se lahko zanašajo na izhodno kodo.
{% endhint %}

***

## Povezava z dinamično prilagoditvijo računalniških zmogljivosti

[Dinamična prilagoditev računalniških zmogljivosti](dynamic-compute-adaptation.md) vpliva predvsem na **nit 3 (obdelava)**:

* **`GPU_PARALLEL`**: Nit 3 hkrati obdeluje več slik prek grafičnega procesorja (GPU) z uporabo poteka `fused_gpu`
* **`GPU_SINGLE`**: Nit 3 serializira dostop do GPU-ja s semaforjem, medtem ko delovni procesi prekrivajo vnosno-izhodne operacije, pri čemer uporablja pot `fused_gpu` ali pomnilniško učinkovito pot `tiled_gpu`
* **`CPU_PARALLEL`**: Nit 3 uporablja obdelavo na CPU z večnitnim vzporednim delovanjem

Dodelitev pomnilnika GPU za niti 3 se poveča tudi, ko niti 1 in 2 končata delo — glejte [Dinamično dodeljevanje pomnilnika GPU](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Naslednji koraki

* [Dinamična prilagoditev računanja](dynamic-compute-adaptation.md) — Kako Chloros izbere optimalno strategijo za vašo strojno opremo
* [Vodnik za NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Delovanje poteka, specifično za platformo Jetson
* [Spremljanje obdelave](../processing-images-gui/monitoring-the-processing.md) — Spremljanje napredka prek grafičnega vmesnika
* [Referenca CLI](../reference/cli-reference.md) — `process`, `export-status`, izhodne kode in razpored izhodnih podatkov

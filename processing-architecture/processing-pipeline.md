# Obdelovalni tok Program

Chloros 1.3.0 uporablja obdelovalni tok s 4 nitmi, ki deluje kot stopnjevana tekoča linija. Vsaka nit obdeluje ločeno fazo delovnega toka, tako da je lahko več slik hkrati v obdelavi v različnih fazah.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Arhitektura procesnega toka

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Vsaka slika poteka skozi vsa štiri niti v določenem vrstnem redu. Pri večnitni obdelavi v Chloros+ več slik hkrati zaseda različne niti – medtem ko nit 3 obdeluje eno sliko, nit 1 lahko zazna naslednjo, nit 2 kalibrira še eno, nit 4 pa končano sliko zapiše na disk.

Napredek se poroča po posameznih niti in se prenaša prek dogodkov Server-Sent Events (zadnji del sistema jih objavlja na `/api/events`). V prikazu napredka v realnem času v CLI so štiri stopnje označene kot **Zaznavanje, Analiza, Obdelava, Izvoz**.

***

## Podrobnosti o niti

### Nit 1: Zaznavanje

**Namen**: Naložiti slike in zaznati kalibracijske cilje.

* Bere slikovne datoteke z diska – pare Survey3 `.raw`+`.jpg`, posnetki LATTICE `.tif`/`.tiff` ter `.dng`
* Izvleče metapodatke EXIF (GPS, model kamere, časovni žigi, osvetlitev)
* Zazna kalibracijske tarče: geometrije tarč, označene z ArUco, za posnetke LATTICE ter klasični ploščati detektor za fotografije kalibracijskih tarč Survey3
* Izhodi: podatki o sliki + metapodatki + rezultati zaznavanja tarč

Predvsem niti, vezane na vhodno-izhodne operacije in procesor.

### Niti 2: Kalibracija

**Namen**: Izračun kalibracijskih parametrov iz zaznanih ciljev.

* Izračuna kalibracijske koeficiente odbojnosti iz slik ciljev
* Izračuna parametre za korekcijo vinjetiranja
* Določi kalibracijske krivulje za posamezne pasove
* Izhodi: kalibracijski parametri za vsako sliko

Računski niz, vezan na CPU. Pri izvajanju programa Survey3 ali mešanem izvajanju niz 3 čaka na ta niz, kadar je omogočena kalibracija odbojnosti, tako da so njegovi koeficienti pripravljeni, preden se obdela katera koli slika. Pri izvajanju, ki vključuje samo LATTICE, niz 3 teče med zaznavanjem: vsak okvir se začne, ko je njegova ciljna sidrna točka dokončna. Okvir upošteva časovno najbližji cilj na obeh straneh, zato počaka, dokler niso skenirani vsi okvirji kamere, ki so časovno bližje od trenutno najbližjega cilja. Rezultati so enaki, le da so na voljo prej.

### Nit 3: Obdelava (GPU)

**Namen**: Uporabi popravke in izračuna indekse vegetacije.**To je niti, ki zahteva največ računskih zmogljivosti.*** **Debayering**: pretvori surove podatke Bayer v večkanalne slike
  * Standardno (hitro, srednja kakovost) — privzeto, `--debayer standard`
  * Texture Aware (počasno, najvišja kakovost) — samo za Chloros+, `--debayer texture-aware`, uporablja model za odstranjevanje šuma z AI/ML
  * Posnetki LATTICE mono (M3M) so enopasovni: pri njih se preskočita koraka demosaika in uravnoteženja beline (z enovrstičnim sporočilom v dnevniku), medtem ko se ti koraki še vedno izvedejo za vse slike M3C/Bayer v istem izvajanju
* **Popravek vinjetiranja**: uporabi popravek vinjetiranja objektiva po celotni sliki
* **Kalibracija odbojnosti**: uporabi kalibracijske koeficiente za pretvorbo v vrednosti odbojnosti
* **Izračun indeksov**: izračuna vegetacijske indekse (NDVI, NDRE, GNDVI, …)
* Izhodi: obdelani podatki slike, pripravljeni za izvoz

Ta niti najbolj izkorišča pospešitev s pomočjo grafičnega procesorja (GPU) in je niti, ki jo [Dinamična prilagoditev računanja](dynamic-compute-adaptation.md) optimizira.

### Niz 4: Izvoz

**Namen**: Zapiše obdelane slike na disk.

* Zapiše izhodne datoteke v izbranem formatu — `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* V izhodne datoteke vključi metapodatke (GPS, časovni žigi, parametri obdelave) — vključno z oznakami obdelave `Chloros:*`, ki se od različice 1.3.0 zapisujejo v vsakem načinu obdelave, tako za Survey3 kot za LATTICE
* Izhodne datoteke razvrsti v projektno mapo kot `<camera>/<format>/<Product>_Images/` — na primer `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Izvožene datoteke ohranijo ime izvorne datoteke; mapa identificira produkt.**
* Pri zajemih LATTICE se lahko en izvorni okvir razširi v več produktov (Debayered, Preview, Radiance, Reflectance, Index), vsak v svoji mapi za produkte. Indeksni produkti se izračunajo iz produkta odbojnosti, ki ga je zapisal tek (če ga ni, iz drugega produkta, ki ga je zapisal tek, na primer iz produkta svetilnosti). Izdelki sevanja in odbojnosti multispektralnih kamer M3C lahko v `<format>/clamp_masks/` dobijo tudi masko omejitve na piksel (glej [Formati izhodnih slik](../output-image-formats.md#clamp-masks))
* Izhodi: končne datoteke na disku

Predvsem niti, vezane na vhodno-izhodne operacije — shranjevanje na SSD-ju jih opazno izboljša.

***

## Pod pokrovom: izvajalci

Znotraj niti 3 je delo na posamezni sliki vzporedno izvedeno na naslednji način:

* **Strategije GPU**(`GPU_SINGLE`, `GPU_PARALLEL`) uporabljajo skupino**delovnih procesov**, od katerih je vsak ločen proces operacijskega sistema z lastnim CUDA-kontekstom. Chloros zažene vsakega delavca tako, da ponovno zažene svoj nameščen program v ozadju — ne prek Pythonovega `multiprocessing`, katerega zagon delavca ne deluje znotraj pakirane aplikacije
* **`CPU_PARALLEL`** uporablja zbirko niti (Pythonov `ThreadPoolExecutor`) — NumPy in OpenCV sprostita GIL, zato so niti zadostne
* Naprave Jetson z 8 GB ali manj skupnega pomnilnika RAM izvrševalca v celoti preskočijo in obdelujejo v okviru istega procesa, zaporedno
* Texture Aware na grafičnem procesorju z manj kot 7 GB VRAM prav tako teče zaporedno – model za odstranjevanje šuma se ne more prilagoditi več kot enkrat

Chloros ne uporablja nobenega distribuiranega okvira tretjih ponudnikov (kot je Ray). Glejte [Dinamično prilagajanje računalniške moči](dynamic-compute-adaptation.md) za informacije o tem, kako se izbirata strategija in število delavcev.

***

## Zaporedna obdelava v primerjavi s cevno obdelavo

### Prosti način (zaporedni)

V brezplačni različici programa Chloros se slike obdelujejo **po ena naenkrat**, zaporedno skozi vse štiri faze:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

GUI v prostem načinu prikazuje poenostavljen indikator napredka; njegove zaporedne faze so prikazane kot **Target Detection**(Zaznavanje ciljev) in nato**Processing** (Obdelava).

### Način Chloros+ (v cevovodu)

Z licenco Chloros+ vsi štirje niti delujejo **sočasno** na različnih slikah:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

Napredovalna vrstica v grafičnem vmesniku prikazuje štiri stopnje; če z miško preletite vrstico, si lahko ogledate napredek po posameznih niti. V CLI se iste štiri stopnje v živo prikazujejo kot **Detecting, Analyzing, Processing, Exporting**.

{% hint style="info" %}
**Ena oznaka, dve imeni.** CLI tretjo fazo imenuje _Obdelava_. Napredek v premium načinu na strežniku – tisti, ki ga prikazuje vrstica napredka v grafičnem vmesniku – isto fazo označuje kot _Kalibriranje_. Gre za isto nit, ki opravlja isto delo (nit 3: debayer, popravki, indeksi).
{% endhint %}

{% hint style="success" %}
**Obdelava v potoku z Chloros+** je lahko 3–5-krat hitrejša od zaporedne obdelave, odvisno od vaše strojne opreme in velikosti niza podatkov. Največje pospešitev je mogoče doseči na sistemih s hitrimi grafičnimi procesorji (GPU) in SSD-ji.
{% endhint %}

***

## Napredek izvoza v niti 4

Nit za izvoz ima lastno sledenje napredka, ki ga lahko preverjate ločeno:**V CLI-ju:**

```bash
chloros-cli export-status
```

**V SDK-ju:**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Obdelava je končana, ko niti 4 doseže 100 %.

{% hint style="info" %}
**Izvedba, ki ne zapiše nobene slike, je neuspešna.**Ob uspehu `chloros-cli process` poroča, koliko slikovnih izdelkov je zapisal (`Image products written: N`). Če so bili zahtevani izdelki, vendar**ni bil**zapišan noben — le `project.json` in `calibration_data.json` — CLI izpiše `Processing finished but wrote no image products.` in**se zaključi z vrednostjo, ki ni nič**, pri čemer navede ime projektne mape in običajne vzroke (vhodna mapa ni bila prepoznana kot zajem – preverite razpored in `--input-level` – ali pa noben od zahtevanih izdelkov ni bil primeren za te kamere). Skripti se lahko zanašajo na izhodno kodo.
{% endhint %}

***

## Povezava z dinamično prilagoditvijo računalniške zmogljivosti

[Dinamična prilagoditev računalniške zmogljivosti](dynamic-compute-adaptation.md) vpliva predvsem na **nit 3 (obdelava)**:

* **`GPU_PARALLEL`**: Nit 3 hkrati obdeluje več slik prek grafičnega procesorja (GPU) z uporabo poteka `fused_gpu`
* **`GPU_SINGLE`**: Nit 3 serializira dostop do grafičnega procesorja s semaforjem, medtem ko delovni procesi prekrivajo vhodno-izhodne operacije, pri čemer uporablja pot `fused_gpu` ali pomnilniško učinkovito pot `tiled_gpu`
* **`CPU_PARALLEL`**: Nit 3 uporablja obdelavo na CPU z večnitnim vzporednim delovanjem

Dodelitev pomnilnika GPU za niti 3 se poveča tudi, ko niti 1 in 2 zaključita delo — glej [Dinamično dodeljevanje pomnilnika GPU](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Naslednji koraki

* [Dinamična prilagoditev računalniških zmogljivosti](dynamic-compute-adaptation.md) — Kako Chloros izbere optimalno strategijo za vašo strojno opremo
* [Vodnik za NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Delovanje poteka, specifično za platformo Jetson
* [Spremljanje obdelave](../processing-images-gui/monitoring-the-processing.md) — Spremljanje napredka prek grafičnega uporabniškega vmesnika
* [Referenca za CLI](../reference/cli-reference.md) — `process`, `export-status`, izhodne kode in razpored izhodnih podatkov

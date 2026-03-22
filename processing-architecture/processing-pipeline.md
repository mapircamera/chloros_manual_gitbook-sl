# Obdelovalni tok

Chloros 1.1.0 uporablja 4-nitni obdelovalni tok, ki deluje kot stopnjevita tekoča linija. Vsaka nit obdeluje ločeno fazo obdelovalnega poteka, kar omogoča sočasno obdelavo več slik v različnih fazah.

***

## Arhitektura toka

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Vsaka slika poteka skozi vse štiri niti v določenem vrstnem redu. Z večnitno obdelavo Chloros+ lahko več slik hkrati poteka v različnih niti – medtem ko nit 3 obdeluje eno sliko, nit 1 lahko zazna naslednjo, nit 2 lahko kalibrira drugo, nit 4 pa lahko zapiše predhodno obdelano sliko na disk.

***

## Podrobnosti o niti

### Nit 1: Zaznavanje

**Namen**: Naložitev slik in zaznavanje kalibracijskih ciljev.

* Bere slikovne datoteke z diska (RAW, JPG)
* Izvleče metapodatke EXIF (GPS, model kamere, časovni žigi, osvetlitev)
* Zazna kalibracijske cilje ArUco v označenih ciljnih slikah
* Izhodi: podatki slike + metapodatki + rezultati zaznavanja ciljev

To je predvsem niz, vezan na vhodno-izhodne operacije in procesor.

### Niz 2: Kalibracija

**Namen**: Izračun kalibracijskih parametrov iz zaznanih ciljev.

* Izračuna kalibracijske koeficiente odbojnosti iz ciljnih slik
* Izračuna parametre korekcije vinjetiranja
* Določi kalibracijske krivulje za vsak pas
* Izhodi: kalibracijski parametri za vsako sliko

To je računski niz, vezan na CPU.

### Niz 3: Obdelava (GPU)

**Namen**: Uporabi korekcije in izračuna indekse vegetacije.**To je računsko najbolj intenziven niz.*** **Debayering**: Pretvori surove podatke v vzorcu Bayer v večkanalne slike
  * Standardno (hitro, srednja kakovost) — privzeto
  * Texture Aware (počasno, najvišja kakovost) — samo Chloros+, uporablja AI/ML odstranjevanje šuma
* **Korekcija vinjetiranja**: Uporabi korekcijo vinjetiranja objektiva po celotni sliki
* **Kalibracija odbojnosti**: Uporabi kalibracijske koeficiente za pretvorbo v vrednosti odbojnosti
* **Izračun indeksov**: Izračuna vegetacijske indekse (NDVI, NDRE, GNDVI itd.)
* Izhodi: obdelani podatki slike, pripravljeni za izvoz

Ta niz najbolj izkorišča pospešitev GPU. Sistem [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) v prvi vrsti optimizira delovanje tega niza.

### Niti 4: Izvoz

**Namen**: Zapiše obdelane slike na disk.

* Zapiše izhodne datoteke v izbranem formatu (TIFF 16-bitni, TIFF 32-bitni %, PNG, JPG)
* V izhodne datoteke vključi metapodatke EXIF (GPS, časovni žigi, parametri obdelave)
* Izhodne datoteke razvrsti v podmapke po modelih fotoaparatov
* Izhodi: končne datoteke na disku

To je v glavnem niz, vezan na vhodno-izhodne operacije. Shranjevanje na SSD znatno izboljša zmogljivost niza 4.

***

## Zaporedna obdelava v primerjavi s cevno obdelavo

### Brezplačni način (zaporedni)

V brezplačni različici Chloros se slike obdelujejo **po ena naenkrat**, zaporedno skozi vse štiri stopnje:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

Napredovalna vrstica v grafičnem vmesniku prikazuje 2 stopnji: zaznavanje cilja in obdelava.

### Način Chloros+ (vzporedno)

Z licenco Chloros+ vsi štirje niti delujejo **sočasno** na različnih slikah:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

Napredovalna vrstica v grafičnem vmesniku prikazuje 4 faze: zaznavanje, analiziranje, kalibriranje, izvoz. Preletite z miško nad napredovalno vrstico, da vidite napredek po posameznih niti.

{% hint style="success" %}
**Obdelava v poti z Chloros+** je lahko 3-5-krat hitrejša od zaporedne obdelave, odvisno od vaše strojne opreme in velikosti podatkovnega niza. Pospešek je največji na sistemih s hitrimi grafičnimi procesorji (GPU) in SSD-ji.
{% endhint %}

***

## Napredek izvoza niti 4

V Chloros 1.1.0 ima izvozna nit (nit 4) lastno namensko sledenje napredka. Napredek izvoza lahko spremljate ločeno:**CLI:**
```bash
chloros-cli export-status
```

**SDK:**
```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Obdelava je končana, ko nit 4 doseže 100 %.

***

## Povezava z dinamično prilagoditvijo računalniških zmogljivosti

Sistem [dinamične prilagoditve računalniške moči](dynamic-compute-adaptation.md) vpliva predvsem na **nit 3 (obdelava)**:

* Strategija **`GPU_PARALLEL`**: nit 3 hkrati obdeluje več slik prek grafičnega procesorja z uporabo poti `fused_gpu`
* Strategija **`GPU_SINGLE`**: Nit 3 obdeluje po eno sliko naenkrat z uporabo pomnilniško učinkovitega poteka `tiled_gpu`
* Strategija **`CPU_PARALLEL`**: Nit 3 uporablja obdelavo na podlagi CPU z večnitnim vzporednim delovanjem

Dodelitev pomnilnika GPU niti 3 se prav tako dinamično spreminja, ko niti 1 in 2 zaključita delo — glejte [Dinamično dodeljevanje pomnilnika GPU](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Naslednji koraki

* [Dinamična prilagoditev računalniških zmogljivosti](dynamic-compute-adaptation.md) — Kako Chloros izbere optimalno strategijo za vašo strojno opremo
* [Vodnik za NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Delovanje poteka, specifično za platformo Jetson
* [Spremljanje obdelave](../processing-images-gui/monitoring-the-processing.md) — Spremljanje napredka prek grafičnega vmesnika

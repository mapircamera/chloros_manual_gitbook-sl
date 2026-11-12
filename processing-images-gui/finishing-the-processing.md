# Zaključek obdelave

Ko program Chloros zaključi obdelavo, je čas, da pregledate rezultate, preverite kakovost izhodnih datotek in pripravite obdelane slike za uporabo v svojem delovnem toku. Ta stran vas vodi skozi zadnje korake in nadaljnje ukrepe.

## Oznaka zaključene obdelave

Ko se obdelava uspešno zaključi, se prikažejo naslednji kazalniki:

* ✅ **Indikator napredka**: doseže 100 % dokončanja
* ✅ **Dnevnik odpravljanja napak**: prikaže zadnje vrstice `[RUN-SUMMARY]` s številkami (slike, skupine kamer, cilji, kalibrirane slike, zapisane datoteke)
* ✅ **Gumb »Začni«**: Ponovno postane aktiven (pripravljen za naslednji krog obdelave)
* ✅ **Izhodne datoteke**: Vse obdelane slike so shranjene v izhodni mapi projekta (spodaj)

{% hint style="warning" %}
**Cikel, ki ne zapiše nobene slike, je neuspešen.** Če ste zahtevali slikovne izdelke, izvedba pa ni zapisala nobene slike, Chloros to poroča kot neuspeh — `[RUN-SUMMARY]` v imenu dnevnika namiguje na verjetni vzrok (ni bilo uvoženo nič, ni bil zaznan noben cilj ali so bili vsi zahtevani izdelki preskočeni kot neuporabni). Enakovredna CLI-ukazna vrstica se zaključi z izhodno vrednostjo, ki ni nič. Namerno izvajanje, namenjeno izključno metapodatkom (vsi izvozni izdelki izklopljeni, brez indeksov), se še vedno šteje za uspešno. Glejte [referenco za CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Iskanje obdelanih slik

### Odpiranje izhodne mape

1. Kliknite ikono **Glavni meni** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">
(zgoraj levo)
2. Izberite **»Odpri mapo projekta«**

3. Odpre se vaš raziskovalec datotek v imeniku projekta
4. Poiščite svoj projekt po imenu

### Drevesna struktura izhodnih datotek

Izdelki so shranjeni **v projektni mapi, razvrščeni po fotoaparatih in nato po datotečnih formatih**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per selected index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

* **Mapa fotoaparata**: `LATT-<sensor>-<lens>-F<filter>` za LATTICE (kar ustreza EXIF-podatkom posnetka `Model`), `<model>_<filter>` za Survey3 (npr. `Survey3N_RGN`). Dve kamere, ki imata skupen senzor in filter, a se razlikujeta po objektivu, imata ločeni drevesni strukturi — vinjeta, vidno polje in popačenje se razlikujejo.
* **Mapa formata**: sledi vaši nastavitvi izvoznega formata — `tiff16`, `tiff8`, `png8`, `jpg8`, ali `tiff32` za TIFF (32-bitni, v odstotkih). Sijaj je vedno v formatu float32 in se vedno shrani pod `tiff32`.
* **Mape izdelkov**:
  * `Reflectance_Calibrated_Images/` — kalibrirana odbojnost
  * `Debayered_Images/` — linearno razbajeriranje (LATTICE): odstranjen temni odmik in demosaicing, brez flat field ali unmix, v izvorni orientaciji senzorja
  * `Preview_Images/` — predogled na zaslonu (LATTICE)
  * `Radiance_Images/` — spektralna svetilnost float32, W/m²/sr/nm (LATTICE multispektralno)
  * `Vignette_Corrected_Images/` **ali** `Sensor_Response_Images/` — nekalibrirana nadomestna možnost za posnetke brez referenčne odbojnosti; na vsakem ciklu obstaja natanko eden od teh dveh, izbran glede na nastavitev korekcije vinjetiranja
  * `<INDEX>_Index_Images/` — ena mapa na izbrani indeks (npr. `NDVI_Index_Images`)
* **`clamp_masks/`** (novost v različici 1.3.0, samo za multispektralne kamere LATTICE M3C): pojavi se v mapi formata, poleg map s produkti, kadar pas razmešava odrezane negativne vrednosti na nič v produktu sevanja ali odbojnosti. Vsebuje eno masko na piksel za vsak prizadet produkt, npr. `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif`. Maske nikoli niso znotraj mape s produkti, zato je mogoče mapo s produkti v celoti prenesti v programsko opremo za fotogrametrijo. Glej [Maske omejevanja](../output-image-formats.md#clamp-masks).

{% hint style="info" %}
**Vsak izvožen produkt ohrani ime izvorne datoteke.**Izvoz sevanja `capture_..._raw.tif` se še vedno imenuje `capture_..._raw.tif` — le da se nahaja v `tiff32/Radiance_Images/`.**Mapa identificira izdelek, ne pa ime datoteke**, zato iskanje po `*radiance*.tif` ne prinese rezultatov; namesto tega poiščite po imenu mape.
{% endhint %}

<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-M3C multispectral camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images, clamp_masks) and tiff32 (Radiance_Images, clamp_masks) subfolders visible -->


### Koliko datotek naj bi bilo?

Ne računajte po formuli — število izhodnih datotek je odvisno od tega, kateri izdelki so bili omogočeni in kateri veljajo za posamezno kamero (npr. kamere RGB nimajo podatkov o sevanju/odbojnosti). Uradno število je navedeno v dnevniku: zadnja vrstica `[RUN-SUMMARY]` natančno poroča, koliko datotek je bilo zapisano, vrstice z namigi pa pojasnjujejo vse, kar je bilo preskočeno.

***

## Pregled obdelanih slik

### Hiter predogled v Raziskovalcu datotek

**Vgrajeni predogled v Windows:**

1. Preidite v mapo z izdelki (npr. `tiff16/Reflectance_Calibrated_Images/`)
2. Izberite datoteko s sliko
3. Predogled se prikaže v oknu za predogled v Raziskovalcu Windows

4. S puščičnimi tipkami brskajte med slikami

### Predogled v zunanjih pregledovalnikih slik

**Priporočeni pregledovalniki:**

* **QGIS** – brezplačna programska oprema GIS (najboljša za georeferencirano multispektralno analizo)
* **IrfanView** – hiter, lahek pregledovalnik slik (podpira format TIFF)
* **Adobe Photoshop** – profesionalno urejanje (podpora formatu TIFF)
* **GIMP** – brezplačna alternativa programu Photoshop
* **Windows Photos** – osnovno pregledovanje (morda ne podpira 16-bitnega formata TIFF)

### Predogled v pregledovalniku slik Chloros


Za napredno vizualizacijo uporabite vgrajeni pregledovalnik slik Chloros:

1. Kliknite miniaturno sliko v brskalniku datotek
2. Slika se odpre v glavnem območju predogleda
3. V levem stranskem meniju kliknite zavihek **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">

4. Za interaktivno analizo uporabite [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md)

Podrobna navodila najdete v [Image Viewer](../image-viewer-gui/opening-an-image-full-screen.md).

***

## Branje vrednosti odbojnosti pikslov (GIS / Pix4D / Skripti)

Odbojnost je shranjena kot celo število DN, **DN, ki pomeni ρ = 1,0, pa je odvisen od izvorne kamere**:

| Izvor          | ρ = 1,0 je | Kako ugotoviti                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (rezerva do ρ 2,0) | V datoteko je vtisnjena oznaka XMP `Chloros:PixelScale=32768` |
| Survey3          | **65535** (omejeno na ρ 1,0)     | Ni oznake `Chloros:PixelScale` — ta odsotnost je znak (izdelki Survey3 vsebujejo druge oznake obdelave `Chloros:*`) |

**Preberite oznako `Chloros:PixelScale` in delite z njo**, namesto da predpostavljate splošno vrednost 65535 — deljenje odbojnosti LATTICE z 65535 tiho prepolovi vsako vrednost. Datoteka odbojnosti iz časa pred različico 1.3.0, zapisana v 8-bitni obliki, je bila odrezana, ne pa ponovno skalirana, in nima oznake merila: namesto deljenja jo ponovno izvozite v 16-bitni ali 32-bitni obliki. Za celotno pojasnilo glejte [Formati izhodnih slik](../output-image-formats.md).

***

## Metapodatki, preneseni v izvoze

Vsak izdelek ohrani **GPS-blok**izvirnega posnetka in njegov**EXIF-pod-IFD**, zato
izvoz prenaša `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` in
`CameraSerialNumber` ter georeferenco.

{% hint style="warning" %}
**Če se ortomosaik prikaže v nesmiselnem merilu, najprej preverite `FocalLength`.**
Pix4D izračuna razdaljo med točkami na tleh iz goriščne razdalje in nadmorske višine. Brez oznake
se vrne k močno napačnemu merilu – pri enem izmerjenem letu s 49 posnetki je bil 411 m × 160 m
pomarančev nasad, ki je bil rekonstruiran kot 47,8 km × 13 km, kar je ustvarilo 455-megapikselno ortomosaiko, ki je v glavnem
prazna. Počasno sestavljanje in nepričakovano velika datoteka sta simptoma tega, ne pa ločeni
težavi.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Ni kopirana *vsaka* oznaka. Strukturne oznake IFD0 so namerno izpuščene (njihovo kopiranje
povzroči poškodbo izhodnih podatkov LATTICE), izključeni pa so tudi `ExifImageWidth` / `ExifImageHeight` so izključeni,
ker opisujejo prvotni posnetek – izvoz, katerega velikost je bila spremenjena, bi sicer
navajal dimenzije, ki so v nasprotju z njegovo lastno rastrsko sliko.

Chloros prav tako zapiše svoje lastne oznake XMP `Chloros:*`, ki opisujejo, kaj je bilo narejeno z vsakim izdelkom
(na primer `Chloros:ProcessingLevel`, `Chloros:CalibrationApplied`,
`Chloros:VignetteCorrected`). Od različice 1.3.0 naprej jih izdelki Survey3 vsebujejo v vsakem načinu obdelave
načinu obdelave, indeksni izdelki opisujejo sam indeks (`Chloros:ProcessingLevel=index`,
formulo in ali je bil izračunan na podlagi odbojnosti ali sevanja), namesto da bi ponavljali
oznake izvirnih surovih podatkov, izdelki v mapah, katerih imena so daljša od 260 znakov, pa ohranijo tudi svoje metapodatke. Glejte [Formati izhodnih slik](../output-image-formats.md#index-products).

***

## Pregled dnevnika odpravljanja napak

### Preverjanje opozoril ali napak

1. Odprite zavihek **Dnevnik odpravljanja napak** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

2. Prebrskajte sporočila
3. Poiščite rumena opozorila ali rdeče napake
4. Preberite vrstice `[RUN-SUMMARY]` in morebitne namige
5. Za pomoč se obrnite na podporo MAPIR


### Shranjevanje dnevnika

Za arhiviranje obdelave ali pošiljanje podpori MAPIR:

1. Kliknite gumb **»Kopiraj«**ali**»Prenesi«**

2. Shranite kot besedilno datoteko v mapo projekta
3. Priložite k dokumentaciji projekta
4. V primeru težav ga pošljite podpori MAPIR Če želite težavo poslati neposredno podjetju MAPIR, uporabite ikono napake (**Prijavi napako**) na vrhu zavihka »Dnevnik odpravljanja napak« — glejte [Prijava napake iz zavihka »Dnevnik«](monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).

***

## Pogoste težave z izhodnimi datotekami in rešitve

### Težava: Manjkajoče izhodne datoteke

**Možni vzroki:**

* Izdelek ni primeren za to kamero (npr. sevanje/odbojnost za RGB-kamere — to je navedeno v dnevniku)
* Manjkala je potrebna referenca (npr. odbojnost brez cilja in brez navzdolnega sevanja `.daq`)
* Polje za izvoz izdelka je bilo onemogočeno v nastavitvah projekta
* Med izvozom je zmanjkalo prostora na disku

**Rešitve:**

1. Preverite namige `[RUN-SUMMARY]` in vrstice `[EXPORT-CHECK]` v dnevniku odpravljanja napak – pojasnjujejo preskoke po posameznih kamerah
2. Preverite potrditvena polja za izvoz izdelka v [Nastavitvah projekta](adjusting-project-settings.md)
3. Preverite, ali je bilo dovolj prostora na disku
4. Po odpravi vzroka ponovno izvedite obdelavo

### Težava: Temni ali svetli robovi (vignetting je še vedno viden)

**Možni vzroki:**

* Popravek vinjetiranja je onemogočen
* Kamera/objektiv ni v bazi profilov Chloros

* Izrazito vinjetiranje, ki presega zmogljivosti popravka

**Rešitve:**

1. Preverite, ali je bil popravek vinjetiranja omogočen v nastavitvah projekta
2. Preverite, ali je bil model kamere pravilno zaznan
3. Če vinjetiranje še vedno obstaja, se obrnite na podporo MAPIR


### Težava: Nepravilne barve ali vrednosti

**Možni vzroki:**

* Kalibracijski cilji niso bili zaznani
* Izbran je bil napačen model kalibracijskega cilja
* Kalibracija odbojnosti je onemogočena
* Slaba kakovost slik ciljev

**Rešitve:**

1. Preverite, ali je bila kalibracija odbojnosti omogočena
2. Preverite sporočila »Target found« v dnevniku za odpravljanje napak
3. Preverite kakovost slik ciljev
4. Ponovno obdelajte s pravilno označenimi cilji

### Težava: Vrednosti NDVI se zdijo napačne

**Pričakovani razponi vrednosti NDVI:**

* **Voda, kamnine, tla**: od -0,1 do 0,2
* **Redka/nezdrava vegetacija**: od 0,2 do 0,4
* **Zmerna vegetacija**: od 0,4 do 0,6
* **Zdrava, gosta vegetacija**: od 0,6 do 0,9**Če so vrednosti zunaj teh razponov:**

1. Preverite, ali je bila uporabljena kalibracija odbojnosti
2. Preverite, ali je bil vključen dnevnik svetlobnega senzorja
3. Preverite, ali so bili zaznani kalibracijski cilji
4. Preverite, ali je bil zaznan pravi model kamere
5. Preverite čas in pogoje zajema ciljnih slik
6. Če indekse izračunavate sami iz datotek odbojnosti, preverite, ali ste vrednosti delili z vrednostjo `Chloros:PixelScale` iz datoteke (glej zgoraj)

***

## Uporaba obdelanih slik

### Za fotogrametrijo / izdelavo ortomosaika

**Priporočeni potek dela:**

1.**Uvozite kalibrirane slike odbojnosti** v programsko opremo za fotogrametrijo:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Ohranite metapodatke EXIF**: Poskrbite, da se ohranijo podatki GPS za geoznačevanje
3. **Kalibrirani delovni postopki**: Uporabite slike odbojnosti za znanstveno natančnost — odbojnost LATTICE vsebuje kalibracijske oznake XMP, ki jih bere Pix4D
4. **Obdelajte indeksne mozaike**: Iz posameznih indeksnih slik ustvarite ortomozaike NDVI

5. **Izvozite georeferencirane datoteke GeoTIFF**: Za uporabo v aplikacijah GIS

### Za analizo GIS

**Priporočeni delovni tok:**

1.**Naložite v QGIS, ArcGIS ali podobno**

2.**Uporabite 16-bitne TIFF** slike odbojnosti za večpasovno analizo (delite z vrednostjo `Chloros:PixelScale` v datoteki)
3. **Uporabite indeksne slike** (NDVI, NDRE) kot pripravljene vegetacijske plasti
4. **Rasterski kalkulator**: združite pasove za lastno analizo
5. **Izvoz**: ustvarite klasifikacijske karte, zaznavanje sprememb, karte zdravstvenega stanja vegetacije

### Za neposredno analizo / poročanje

**Priporočeni potek dela:**

1.**Uporabite indeksne slike z barvami LUT** za vizualna poročila
2. **Izpis statistike**: povprečni NDVI na polje/parcelo
3. **Časovne vrste**: primerjava indeksov med več sejami
4. **Priprava poročil**: vključite karte, statistike in vizualizacije***

## Arhiviranje in varnostno kopiranje

### Priporočena strategija varnostnega kopiranja

**Kaj shraniti:**

* ✅ **Izvirne slike v formatu RAW/JPG ali surove posnetke LATTICE** – arhivirajte na ločenem disku/v oblaku; surovi podatki so vir obdelave, iz njih pa je mogoče ponovno ustvariti vse ostalo
* ✅ **Datoteke svetlobnih senzorjev `.daq` / `.csv`** – potrebne za poznejšo ponovno izračunavanje odbojnosti
* ✅ **Obdelani izhodni podatki** – shranite kalibrirane slike in indekse
* ✅ **Projektna mapa** (`project.json` in spremljajoče datoteke) – Vsebuje vse nastavitve za ponovno obdelavo, če bo to potrebno
* ✅ **Dnevnik odpravljanja napak** – Dokumentira podrobnosti obdelave
* ✅ **Slike kalibracijskih ciljev** – Za preverjanje in ponovno obdelavo

**Priporočila za shranjevanje:**

* **Takojšnja varnostna kopija**: zunanji trdi disk
* **Dolgoročno arhiviranje**: shranjevanje v oblaku (Google Drive, Dropbox itd.)
* **Ključni podatki**: shranite 2–3 kopije na različnih lokacijah***

## Naslednje obdelave

### Ponovna uporaba nastavitev projekta

Če boste v prihodnosti obdelovali podobne nize podatkov:

1. **Shranite predlogo projekta** (če tega še niste storili)
2. **Ustvarite nov projekt** z uporabo shranjene predloge
3. **Uvozite nove slike**

4.**Obdelajte**z enakimi nastavitvami za doslednost

### Obdelava več sej v paketu

Za več sej/naborov podatkov:**Možnost 1: GUI – več projektov**

* Ustvarite ločen projekt za vsako sejo
* Uporabite dosledne nastavitve predloge
* Obdelujte po eno naenkrat

**Možnost 2: Chloros CLI (samo Chloros+)**

* Avtomatizirajte obdelavo v serijah
* Obdelajte več map s skripti
* Glejte [Dokumentacijo CLI](../CLI.md) in [Referenco CLI](../reference/cli-reference.md)

**Možnost 3: Python SDK (samo Chloros+)**

* Programsko upravljanje
* Integracija z analitičnimi poteki
* Glejte [dokumentacijo API](../api-python-sdk.md) in [referenčni priročnik SDK](../reference/sdk-reference.md)

***

## Odpravljanje težav pri naknadni obdelavi

### Ponovna obdelava z drugačnimi nastavitvami

Če rezultati niso zadovoljivi:

1. Ohranite izvirne slike (nikoli jih ne izbrišite)
2. Odprite isti projekt v Chloros u 3. Prilagodite nastavitve v oknu »Nastavitve projekta«
4. Ponovno obdelajte – izhodni podatki se shranijo v iste mape izdelkov, zato se datoteke z istim imenom iz prejšnjega izvajanja nadomestijo

### Obdelava podskupine slik

Če želite ponovno obdelati le določene slike:

1. Ustvarite nov projekt
2. Uvozite le slike, ki jih je treba ponovno obdelati
3. Uporabite isto predlogo nastavitev
4. Obdelajte manjši nabor podatkov

### Pomoč

Če naletite na težave:

* 📧 **E-pošta**: info@mapir.camera (priložite dnevnik odpravljanja napak)
* 🌐 **Podpora**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Pogosta vprašanja**: [Pogosta vprašanja](../faq.md)
* 📖 **Dokumentacija**: [Priročnik za Chloros](../)

***

## Povzetek: Celoten potek dela

Sedaj ste zaključili celoten potek obdelave v Chloros u:

1. ✅ **Ustvarjen projekt** – glejte [Projekti](../projects.md)
2. ✅ **Dodali ste datoteke** – glejte [Dodajanje datotek](adding-files-to-a-project.md)
3. ✅ **Prilagodili nastavitve** – glejte [Prilagajanje nastavitev projekta](adjusting-project-settings.md)
4. ✅ **Označili ciljne slike** – glejte [Izbira ciljnih slik](choosing-target-images.md)
5. ✅ **Začetek obdelave** – glej [Začetek obdelave](starting-the-processing.md)
6. ✅ **Spremljanje napredka** – glej [Spremljanje obdelave](monitoring-the-processing.md)
7. ✅ **Pregled rezultatov** – Ta stran**Vaše kalibrirane, glede na odbojnost popravljene multispektralne slike so pripravljene za analizo!**

***

## Dodatni viri

### Napredne funkcije

* [**Pregledovalnik slik**](../image-viewer-gui/opening-an-image-full-screen.md) – Interaktivna vizualizacija in analiza
* [**Index/LUT Sandbox**](../image-viewer-gui/index-lut-sandbox.md) – Testiranje indeksov po meri
* [**Formule multispektralnih indeksov**](../project-settings/multispectral-index-formulas.md) – Celoten referenčni seznam indeksov

### Avtomatizacija in integracija

* [**Dokumentacija CLI**](../CLI.md) – Obdelava v paketih prek ukazne vrstice
* [**Python SDK**](../api-python-sdk.md) – Programska avtomatizacija
* [**Funkcije Chloros+**](../#chloros) – Napredne zmogljivosti obdelave

### Podpora in učenje

* [**Pogosta vprašanja**](../faq.md) – Odgovori na pogosta vprašanja
* [**Kalibracijski cilji**](../calibration-targets.md) – Razumevanje kalibracije odbojnosti
* [**Podprte kamere**](../supported-cameras.md) – Združljiva strojna oprema

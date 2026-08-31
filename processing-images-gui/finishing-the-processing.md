# Zaključek obdelave

Ko program Chloros zaključi obdelavo, je čas, da pregledate rezultate, preverite kakovost izhodnih podatkov in pripravite obdelane slike za uporabo v svojem delovnem toku. Ta stran vas vodi skozi zadnje korake in nadaljnje ukrepe.

## Oznaka zaključene obdelave

Ko se obdelava uspešno zaključi, se prikažejo naslednji kazalniki:

* ✅ **Napredovalna vrstica**: doseže 100 % zaključenosti
* ✅ **Dnevnik odpravljanja napak**: Prikaže zadnje vrstice programa `[RUN-SUMMARY]` s številkami (slike, skupine kamer, cilji, kalibrirane slike, zapisane datoteke)
* ✅ **Gumb za zagon**: Ponovno postane aktiven (pripravljen za naslednji obdelovalni cikel)
* ✅ **Izhodne datoteke**: vse obdelane slike so shranjene v izhodno drevesno strukturo projekta (spodaj)

{% hint style="warning" %}
**Obdelava, ki ne zapiše nobene slike, je neuspešna.** Če ste zahtevali slikovne izdelke, izvedba pa ni zapisala nobene slike, Chloros to poroča kot neuspeh — `[RUN-SUMMARY]` v imenu dnevnika namiguje na verjetni vzrok (ni bilo uvoženo nič, ni bil zaznan noben cilj ali so bili vsi zahtevani izdelki preskočeni kot neuporabni). Ekvivalent CLI se zaključi z vrednostjo, ki ni nič. Namerno izvajanje, ki zajema samo metapodatke (vsi izvozni izdelki izklopljeni, brez indeksov), se še vedno šteje za uspešno. Glejte [referenco za CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Kje najti obdelane slike

### Odpiranje izhodne mape

1. Kliknite ikono **Glavni meni** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (zgoraj levo)
2. Izberite **»Odpri mapo projekta«**

3. Odpre se raziskovalec datotek v imeniku projekta
4. Poiščite svoj projekt po imenu

### Drevo izhodnih datotek

Proizvodi so shranjeni **v mapi projekta, razvrščeni po kameri in nato po formatu datoteke**:

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

* **Mapa kamere**: `LATT-<sensor>-<lens>-F<filter>` za LATTICE (ustreza EXIF-podatkom posnetka `Model`), `<model>_<filter>` za Survey3 (npr. `Survey3N_RGN`). Dve kameri, ki imata isti senzor in filter, a se razlikujeta po objektivu, imata ločeni drevesni strukturi – vinjeta, vidno polje in popačenje se razlikujejo.
* **Mapa formata**: sledi vaši nastavitvi izvoznega formata – `tiff16`, `tiff8`, `png8`, `jpg8` ali `tiff32` za TIFF (32-bitno, v odstotkih). Sijaj je vedno tipa float32 in se vedno shrani pod `tiff32`.
* **Mape izdelkov**:
  * `Reflectance_Calibrated_Images/` — kalibrirana odbojnost
  * `Debayered_Images/` — linearno razbajeriranje (LATTICE)
  * `Preview_Images/` — predogled na zaslonu (LATTICE)
  * `Radiance_Images/` — spektralna svetilnost v formatu float32, W/m²/sr/nm (LATTICE multispektralno)
  * `Vignette_Corrected_Images/` **ali** `Sensor_Response_Images/` — nekalibrirana nadomestna vrednost za sličice brez referenčne odbojnosti; na vsakem izvajanju obstaja natanko ena od teh dveh vrednosti, izbrana z nastavitvijo korekcije Vignette
  * `<INDEX>_Index_Images/` — ena mapa na izbrani indeks (npr. `NDVI_Index_Images`)

{% hint style="info" %}
**Vsak izvožen produkt ohrani ime izvorne datoteke.**Izvoz sevanja datoteke `capture_..._raw.tif` se še vedno imenuje `capture_..._raw.tif` — le da se nahaja v mapi `tiff32/Radiance_Images/`.**Mapa identificira izdelek, ne pa ime datoteke**, zato iskanje po `*radiance*.tif` ne prinese rezultatov; namesto tega poiščite po imenu mape.
{% endhint %}



<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-… camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images) and tiff32 (Radiance_Images) subfolders visible -->### Koliko datotek naj bi bilo?

Ne računajte po formuli – število izhodnih datotek je odvisno od tega, kateri izdelki so bili omogočeni in kateri veljajo za posamezno kamero (npr. kamere RGB ne dobijo podatkov o sevanju/odbojnosti). Uradno število je navedeno v dnevniku: zadnja vrstica `[RUN-SUMMARY]` natančno navaja, koliko datotek je bilo zapisano, vrstice z namigi pa pojasnjujejo vse, kar je bilo preskočeno.

***

## Pregled obdelanih slik

### Hiter predogled v Raziskovalcu datotek

**Vgrajeni predogled Windows:**

1. Prejdite v mapo z izdelkom (npr. `tiff16/Reflectance_Calibrated_Images/`)
2. Izberite datoteko s sliko
3. Predogled se prikaže v oknu za predogled v Raziskovalcu Windows
4. Z puščičnimi tipkami brskajte med slikami

### Predogled v zunanjih pregledovalnikih slik

**Priporočeni pregledovalniki:*** **QGIS** – brezplačna programska oprema GIS (najboljša za georeferencirano multispektralno analizo)
* **IrfanView** – hiter, lahek pregledovalnik slik (podpira TIFF)
* **Adobe Photoshop** – profesionalno urejanje (podpora za TIFF)
* **GIMP** – brezplačna alternativa programu Photoshop
* **Windows Photos** – osnovno pregledovanje (morda ne podpira 16-bitnega formata TIFF)

### Predogled v pregledovalniku slik Chloros

Za napredno vizualizacijo uporabite vgrajeni pregledovalnik slik v Chloros:

1. V brskalniku datotek kliknite miniaturno sliko
2. Slika se odpre v glavnem območju predogleda
3. V levem stranskem meniju kliknite zavihek **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
4. Za interaktivno analizo uporabite [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md)

Podrobna navodila najdete v [Image Viewer](../image-viewer-gui/opening-an-image-full-screen.md).

***

## Branje vrednosti odbojnosti pikslov (GIS / Pix4D / skripti)

Odbojnost je shranjena kot celo število DN, **DN, ki pomeni ρ = 1,0, pa je odvisen od izvorne kamere**:

| Izvor          | ρ = 1,0 je | Kako to ugotoviti                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (razpon do ρ 2,0) | V datoteko je vtisnjena oznaka XMP `Chloros:PixelScale=32768` |
| Survey3         | **65535** (odrezano pri ρ 1,0)     | Ni oznak XMP `Chloros:*` — ta odsotnost je znak |

**Preberite oznako `Chloros:PixelScale` in delite z njo**, namesto da predpostavljate splošno vrednost 65535 — deljenje odbojnosti LATTICE z 65535 tiho prepolovi vsako vrednost. En mejni primer po zasnovi nima lestvice: zajem iz 8-bitnega vira, zapisan kot 8-bitni izhod, je odrezan, ne pa ponovno skaliran, in namerno nima oznake lestvice — namesto deljenja ga ponovno izvozite v 16-bitni ali 32-bitni obliki. Za celotno zgodbo glejte [Oblike izhodnih slik](../output-image-formats.md).***

## Metapodatki, preneseni v izvoze

Vsak izdelek ohrani **GPS-blok**izvirnega posnetka in njegov**EXIF-pod-IFD**, zato
izvoz vsebuje `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` in
`CameraSerialNumber` ter georeferenco.

{% hint style="warning" %}
**Če se ortomosaik prikaže v nesmiselnem merilu, najprej preverite `FocalLength`.**
Pix4D izračuna razdaljo vzorca na tleh iz goriščne razdalje in nadmorske višine. Brez te oznake
se vrne k skrajno napačnemu merilu – pri enem izmerjenem poletu s 49 posnetki je bil pomarančev nasad velikosti 411 m × 160 m
rekonstruiran kot 47,8 km × 13 km, kar je ustvarilo ortofoto z 455 milijoni pikslov, ki je v glavnem
prazno. Počasno sestavljanje in nepričakovano velika datoteka sta simptoma tega, ne pa ločeni
težavi.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Ne *vsaka* oznaka se kopira. Strukturne oznake IFD0 so namerno izpuščene (njihovo kopiranje
poškoduje izhod LATTICE), `ExifImageWidth` in `ExifImageHeight` pa sta izključena,
ker opisujeta izvirni posnetek – izvoz, ki je bil spremenjen v velikosti, bi sicer
navajal dimenzije, ki so v nasprotju z njegovim lastnim rastrom.

***

## Pregled dnevnika odpravljanja napak

### Preverjanje opozoril ali napak

1. Odprite zavihek **Dnevnik odpravljanja napak** v<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">
2. Pomaknite se po sporočilih
3. Poiščite rumena opozorila ali rdeče napake
4. Preberite vrstice `[RUN-SUMMARY]` in morebitne namige
5. Za pomoč se obrnite na podporo MAPIR

### Shranjevanje dnevnika

Za arhiviranje obdelave ali pošiljanje podpori MAPIR:

1. Kliknite gumb **»Kopiraj«**ali**»Prenesi«**

2. Shranite kot besedilno datoteko v mapo projekta
3. Priložite jo k dokumentaciji projekta
4. V primeru težav jo pošljite podpori MAPIR

***

## Pogoste težave z izhodnimi datotekami in rešitve

### Težava: Manjkajoče izhodne datoteke

**Možni vzroki:**

* Produkt ni primeren za to kamero (npr. sevanje/odbojnost za kamere RGB – tako je navedeno v dnevniku)
* Manjkala je potrebna referenca (npr. odbojnost brez cilja in brez navzdolnega sevanja `.daq`)
* Polje za izvoz izdelka je bilo onemogočeno v nastavitvah projekta
* Med izvozom se je izčrpal prostor na disku

**Rešitve:**

1. Preverite namige `[RUN-SUMMARY]` in vrstice `[EXPORT-CHECK]` v dnevniku odpravljanja napak – pojasnjujejo preskoke po posameznih kamerah
2. Preverite potrditvena polja za izvoz izdelkov v [Nastavitve projekta](adjusting-project-settings.md)
3. Preverite, ali je bilo dovolj prostora na disku
4. Po odpravi vzroka ponovno izvedite obdelavo

### Težava: Temni ali svetli robovi (vigneting je še vedno viden)

**Možni vzroki:**

* Popravek vinjetiranja je onemogočen
* Kamera/objektiv ni v bazi profilov Chloros
* Izrazito vinjetiranje, ki presega zmogljivosti popravka

**Rešitve:**

1. Preverite, ali je bila korekcija vinjetiranja omogočena v nastavitvah projekta
2. Preverite, ali je bil model kamere pravilno zaznan
3. Če vinjetiranje še vedno obstaja, se obrnite na podporo MAPIR

### Težava: Nepravilne barve ali vrednosti

**Možni vzroki:**

* Ni zaznanih kalibracijskih ciljev
* Izbran je napačen model kalibracijskega cilja
* Kalibracija odbojnosti je onemogočena
* Slaba kakovost slik ciljev

**Rešitve:**

1. Preverite, ali je bila omogočena kalibracija odbojnosti
2. Preverite sporočila »Cilj najden« v dnevniku odpravljanja napak
3. Preverite kakovost slik ciljev
4. Ponovno obdelajte s pravilno označenimi cilji

### Težava: Vrednosti NDVI se zdijo napačne

**Pričakovani razponi vrednosti NDVI:*** **Voda, kamnine, tla**: od -0,1 do 0,2
* **Redka/nezdrava vegetacija**: od 0,2 do 0,4
* **Zmerna vegetacija**: od 0,4 do 0,6
* **Zdrava, gosta vegetacija**: od 0,6 do 0,9**Če so vrednosti zunaj teh razponov:**

1. Preverite, ali je bila uporabljena kalibracija odbojnosti
2. Preverite, ali je bil vključen dnevnik svetlobnega senzorja
3. Preverite, ali so bili zaznani kalibracijski cilji
4. Preverite, ali je bil zaznan pravi model kamere
5. Preverite čas in pogoje zajema ciljnih slik
6. Če indekse izračunavate sami iz datotek odbojnosti, preverite, ali ste vrednost delili z vrednostjo `Chloros:PixelScale` iz datoteke (glejte zgoraj)

***

## Uporaba obdelanih slik

### Za fotogrametrijo / izdelavo ortomosaika

**Priporočeni potek dela:**

1.**Uvozite kalibrirane slike odbojnosti** v programsko opremo za fotogrametrijo:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Ohranite metapodatke EXIF**: Poskrbite, da se ohranijo podatki GPS za geotagiranje
3. **Kalibrirani delovni postopki**: Uporabite slike odbojnosti za znanstveno natančnost — odbojnost LATTICE vsebuje oznake kalibracije XMP, ki jih bere Pix4D
4. **Obdelajte indeksne mozaike**: Iz posameznih indeksnih slik ustvarite ortomosaike NDVI
5. **Izvozite georeferencirane slike GeoTIFF**: Za uporabo v aplikacijah GIS

### Za analizo GIS

**Priporočeni delovni tok:**

1.**Naložite v QGIS, ArcGIS ali podobno**

2.**Uporabite 16-bitne slike odbojnosti TIFF** za večpasovno analizo (delite z vrednostjo `Chloros:PixelScale` iz datoteke)
3. **Uporabite indeksne slike** (NDVI, NDRE) kot pripravljene sloje vegetacije
4. **Rasterski kalkulator**: združite pasove za prilagojeno analizo
5. **Izvoz**: ustvarite klasifikacijske karte, karte zaznavanja sprememb, karte zdravstvenega stanja vegetacije

### Za neposredno analizo / poročanje

**Priporočeni potek dela:**

1.**Uporabite indeksne slike z barvami LUT** za vizualna poročila
2. **Izpis statistike**: Povprečna vrednost NDVI na polje/parcelo
3. **Časovne vrste**: Primerjava indeksov med več sejami
4. **Priprava poročil**: Vključite karte, statistike in vizualizacije***

## Arhiviranje in varnostno kopiranje

### Priporočena strategija varnostnega kopiranja

**Kaj shraniti:*** ✅ **Izvirne slike RAW/JPG ali surove posnetke LATTICE** – arhivirajte na ločenem disku/v oblaku; surovi podatki so vir za obdelavo, iz njih pa je mogoče ponovno ustvariti vse ostalo
* ✅ **Datoteke svetlobnih senzorjev `.daq` / `.csv`** – potrebne za poznejšo ponovno izračunavanje odbojnosti
* ✅ **Obdelani izhodni podatki** – shranite kalibrirane slike in indekse
* ✅ **Projektna mapa** (`project.json` in spremljevalne datoteke) – Vsebuje vse nastavitve za ponovno obdelavo, če bo to potrebno
* ✅ **Dnevnik odpravljanja napak** – Dokumentira podrobnosti obdelave
* ✅ **Slike kalibracijskih tarč** – Za preverjanje in ponovno obdelavo**Priporočila za shranjevanje:*** **Takojšnja varnostna kopija**: Zunanji trdi disk
* **Dolgoročno arhiviranje**: Shranjevanje v oblaku (Google Drive, Dropbox itd.)
* **Kritični podatki**: Shranite 2–3 kopije na različnih lokacijah***

## Naslednja obdelava

### Ponovna uporaba nastavitev projekta

Če boste v prihodnosti obdelovali podobne nize podatkov:

1. **Shranite predlogo projekta** (če tega še niste storili)
2. **Ustvarite nov projekt** z uporabo shranjene predloge
3. **Uvozite nove slike**

4.**Obdelajte**z enakimi nastavitvami zaradi doslednosti

### Skupinska obdelava več sej

Za več sej/naborov podatkov:**Možnost 1: Grafični vmesnik – več projektov**

* Za vsako sejo ustvarite ločen projekt
* Uporabite dosledne nastavitve predloge
* Obdelujte po enega naenkrat

**Možnost 2: Chloros CLI (samo Chloros+)**

* Avtomatizirajte obdelavo v serijah
* Obdelajte več map s skripti
* Glejte [Dokumentacijo CLI](../CLI.md) in [Referenco CLI](../reference/cli-reference.md)

**Možnost 3: Python SDK (samo za Chloros+)**

* Programsko upravljanje
* Integracija z analitičnimi potmi
* Glejte [Dokumentacijo API](../api-python-sdk.md) in [Referenco SDK](../reference/sdk-reference.md)

***

## Odpravljanje težav pri naknadni obdelavi

### Ponovna obdelava z drugačnimi nastavitvami

Če rezultati niso zadovoljivi:

1. Ohranite izvirne slike (nikoli jih ne izbrišite)
2. Odprite isti projekt v Chloros
3. Prilagodite nastavitve v oknu »Nastavitve projekta«
4. Ponovno obdelajte — izhodni podatki se shranijo v iste mape izdelkov, zato se datoteke z istim imenom iz prejšnjega obdelovanja nadomestijo

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
* 📖 **Dokumentacija**: [Chloros Priročnik](../)***

## Povzetek: Celoten potek dela

Sedaj ste zaključili celoten potek obdelave Chloros:

1. ✅ **Ustvarjen projekt** – glejte [Projekti](../projects.md)
2. ✅ **Dodali ste datoteke** – glejte [Dodajanje datotek](adding-files-to-a-project.md)
3. ✅ **Prilagodili ste nastavitve** – glejte [Prilagajanje nastavitev projekta](adjusting-project-settings.md)
4. ✅ **Označene tarče** – glej [Izbiranje ciljnih slik](choosing-target-images.md)
5. ✅ **Začetek obdelave** – glej [Začetek obdelave](starting-the-processing.md)
6. ✅ **Spremljanje napredka** – glej [Spremljanje obdelave](monitoring-the-processing.md)
7. ✅ **Pregled rezultatov** – Ta stran**Vaše kalibrirane, glede na odbojnost popravljene multispektralne slike so pripravljene za analizo!**

***

## Dodatni viri

### Napredne funkcije

* [**Pregledovalnik slik**](../image-viewer-gui/opening-an-image-full-screen.md) – Interaktivna vizualizacija in analiza
* [**Indeks/LUT Sandbox**](../image-viewer-gui/index-lut-sandbox.md) – Testiranje indeksov po meri
* [**Formule multispektralnih indeksov**](../project-settings/multispectral-index-formulas.md) – Celoten referenčni seznam indeksov

### Avtomatizacija in integracija

* [**Dokumentacija CLI**](../CLI.md) – Obdelava v paketih prek ukazne vrstice
* [**Python SDK**](../api-python-sdk.md) – Programska avtomatizacija
* [**Chloros+ Funkcije**](../#chloros) – Napredne zmogljivosti obdelave

### Podpora in učenje

* [**Pogosta vprašanja**](../faq.md) - Odgovori na pogosta vprašanja
* [**Kalibracijski cilji**](../calibration-targets.md) - Razumevanje kalibracije odbojnosti
* [**Podprte kamere**](../supported-cameras.md) - Združljiva strojna oprema

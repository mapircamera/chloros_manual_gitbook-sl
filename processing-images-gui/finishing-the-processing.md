# Zaključek obdelave

Ko program Chloros zaključi obdelavo, je čas, da pregledate rezultate, preverite kakovost izhodnih podatkov in pripravite obdelane slike za uporabo v svojem delovnem toku. Ta stran vas vodi skozi zadnje korake in nadaljnje ukrepe.

## Oznaka zaključene obdelave

Ko se obdelava uspešno zaključi, boste videli več indikatorjev:

* ✅ **Napredovalna vrstica**: doseže 100 % zaključenosti
* ✅ **Dnevnik odpravljanja napak**: prikaže sporočilo »Obdelava zaključena«
* ✅ **Gumb Začetek**: ponovno postane aktiven (pripravljen za naslednji krog obdelave)
* ✅ **Izhodne datoteke**: Vse obdelane slike so shranjene v podmapo modela kamere***

## Iskanje obdelanih slik

### Odpiranje izhodne mape

1. Kliknite ikono **Glavni meni** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (zgoraj levo)
2. Izberite **»Odpri mapo projekta«**

3. Odpre se raziskovalec datotek v imeniku projekta
4. Poiščite svoj projekt po imenu

***

## Pregled obdelanih slik

### Hitri predogled v raziskovalcu datotek

**Vgrajeni predogled Windows:**

1. Prejdite v podmapo modela kamere
2. Izberite slikovno datoteko
3. Predogled se prikaže v oknu predogleda raziskovalca Windows
4. S puščičnimi tipkami brskajte po slikah

### Predogled v zunanjih pregledovalnikih slik

**Priporočeni pregledovalniki:*** **QGIS** – brezplačna programska oprema GIS (najboljša za georeferencirano multispektralno analizo)
* **IrfanView** – hiter, lahek pregledovalnik slik (podpira TIFF)
* **Adobe Photoshop** – profesionalno urejanje (podpora za TIFF)
* **GIMP** – brezplačna alternativa za Photoshop
* **Windows Photos** – osnovno pregledovanje (morda ne podpira 16-bitnega TIFF)

### Predogled v Chloros Image Viewer

Za napredno vizualizacijo uporabite vgrajeni Image Viewer v Chloros:

1. Kliknite miniaturno sliko v brskalniku datotek
2. Slika se odpre v glavnem območju predogleda
3. Kliknite zavihek **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> v levem stranskem meniju
4. Za interaktivno analizo uporabite [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md)

Podrobna navodila najdete v [Image Viewer](../image-viewer-gui/opening-an-image-full-screen.md).

***

## Pregled dnevnika odpravljanja napak

### Preverite, ali so prisotna opozorila ali napake

1. Odprite zavihek **Dnevnik odpravljanja napak** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> 2. Pomaknite se po sporočilih
3. Poiščite rumena opozorila ali rdeče napake
4. Preglejte vse zabeležene težave
5. Za pomoč se obrnite na podporo MAPIR

### Shranjevanje dnevnika

Za arhiviranje obdelave ali pošiljanje podpori MAPIR:

1. Kliknite gumb **&quot;Kopiraj&quot;**ali**&quot;Prenesi&quot;**

2. Shranite kot besedilno datoteko v mapo projekta
3. Priložite k dokumentaciji projekta
4. Pošljite podpori MAPIR, če se pojavijo težave

***

## Pogoste težave z izhodnimi datotekami in rešitve

### Težava: Manjkajoče izhodne datoteke

**Možni vzroki:**

* Datoteke niso izpolnjevale meril za obdelavo
* Slike, namenjene le cilju (izključene iz izvoza)
* Med izvozom je zmanjkalo prostora na disku
* Poškodba datoteke med obdelavo

**Rešitve:**

1. Preverite dnevnik odpravljanja napak za sporočila o preskočenih datotekah/napakah
2. Preverite, ali je bilo dovolj prostora na disku
3. Preštejte datoteke: Število mora ustrezati (izvirno število – ciljno število) × (indeksi + 1)
4. Ponovno uvozite in ponovno obdelajte vse manjkajoče datoteke

### Težava: Temni ali svetli robovi (vignetting je še vedno viden)

**Možni vzroki:**

* Popravek vignettinga je onemogočen
* Kamera/objektiv ni v bazi profilov Chloros
* Ekstremni vignetting, ki presega zmogljivosti popravka

**Rešitve:**

1. Preverite, ali je bila korekcija vinjetiranja omogočena v nastavitvah projekta
2. Preverite, ali je bil model kamere pravilno zaznan
3. Če vinjetiranje vztraja, se obrnite na podporo MAPIR

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
4. Ponovno obdelajte z označenimi pravilnimi cilji

### Težava: Vrednosti NDVI se zdijo napačne

**Pričakovani razponi NDVI:*** **Voda, kamnine, tla**: od -0,1 do 0,2
* **Redka/nezdrava vegetacija**: od 0,2 do 0,4
* **Zmerna vegetacija**: od 0,4 do 0,6
* **Zdrava, gosta vegetacija**: od 0,6 do 0,9**Če so vrednosti zunaj teh razponov:**

1. Preverite, ali je bila uporabljena kalibracija odbojnosti
2. Preverite, ali je bil vključen dnevnik svetlobnega senzorja
3. Preverite, ali so bili zaznani kalibracijski cilji
4. Preverite, ali je bil zaznan pravi model kamere
5. Preglejte čas zajema ciljne slike in pogoje

***

## Uporaba obdelanih slik

### Za fotogrametrijo / ustvarjanje ortomosaika

**Priporočeni potek dela:**

1.**Uvozite kalibrirane slike odbojnosti** v programsko opremo za fotogrametrijo:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Ohranite metapodatke EXIF**: Poskrbite, da so podatki GPS ohranjeni za geotagiranje
3. **Kalibrirani delovni tokovi**: Uporabite slike odbojnosti za znanstveno natančnost
4. **Obdelajte indeksne mozaike**: Iz posameznih indeksnih slik ustvarite ortomosaike NDVI
5. **Izvozite georeferencirane GeoTIFF**: Za uporabo v aplikacijah GIS

### Za analizo GIS

**Priporočeni potek dela:**

1.**Naložite v QGIS, ArcGIS ali podobno**

2.**Uporabite 16-bitne slike odbojnosti TIFF** za večpasovno analizo
3. **Uporabite indeksne slike** (NDVI, NDRE) kot pripravljene vegetacijske plasti
4. **Rasterski kalkulator**: združite pasove za prilagojeno analizo
5. **Izvoz**: Ustvarite klasifikacijske karte, zaznavanje sprememb, karte zdravstvenega stanja vegetacije

### Za neposredno analizo / poročanje

**Priporočeni potek dela:**

1.**Uporabite indeksne slike z barvami LUT** za vizualna poročila
2. **Izpis statistike**: Povprečje NDVI na polje/parcelo
3. **Časovne vrste**: Primerjajte indekse med več sejami
4. **Ustvarjanje poročil**: Vključite karte, statistike in vizualizacije***

## Arhiviranje in varnostno kopiranje

### Priporočena strategija varnostnega kopiranja

**Kaj shraniti:*** ✅ **Izvirne slike RAW/JPG** – Arhivirajte na ločenem disku/v oblaku
* ✅ **Obdelani izhodi** – Shranite kalibrirane slike in indekse
* ✅ **Datoteka projekta** – vsebuje vse nastavitve za ponovno obdelavo, če je potrebno
* ✅ **Dnevnik odpravljanja napak** – dokumentira podrobnosti obdelave
* ✅ **Slike kalibracijskih ciljev** – za preverjanje in ponovno obdelavo**Priporočila za shranjevanje:*** **Takojšnja varnostna kopija**: zunanji trdi disk
* **Dolgoročno arhiviranje**: Shranjevanje v oblaku (Google Drive, Dropbox itd.)
* **Kritični podatki**: Shranite 2–3 kopije na različnih lokacijah***

## Naslednje obdelave

### Ponovna uporaba nastavitev projekta

Če boste v prihodnosti obdelovali podobne nize podatkov:

1. **Shranite predlogo projekta** (če tega še niste storili)
2. **Ustvarite nov projekt** z uporabo shranjene predloge
3. **Uvozite nove slike**

4.**Obdelajte**z enakimi nastavitvami za doslednost

### Obdelava več sej v seriji

Za več sej/naborov podatkov:**Možnost 1: GUI – več projektov**

* Ustvarite ločen projekt za vsako sejo
* Uporabite dosledne nastavitve predloge
* Obdelujte po eno naenkrat

**Možnost 2: Chloros CLI (samo Chloros+)**

* Avtomatizirajte obdelavo v serijah
* Obdelajte več map s skripti
* Glej [Dokumentacija CLI](../CLI.md)

**Možnost 3: Python SDK (samo Chloros+)**

* Programsko upravljanje
* Integracija z analitičnimi potmi
* Glej [API Dokumentacija](../api-python-sdk.md)

***

## Odpravljanje težav pri naknadni obdelavi

### Ponovna obdelava z drugačnimi nastavitvami

Če rezultati niso zadovoljivi:

1. Ohranite izvirne slike (nikoli jih ne izbrišite)
2. Odprite isti projekt v Chloros
3. Prilagodite nastavitve v oknu Project Settings
4. Ponovno obdelajte – izhodi bodo prepisali prejšnje rezultate

### Obdelava podsklopa slik

Za ponovno obdelavo le določenih slik:

1. Ustvarite nov projekt
2. Uvozite le slike, ki jih je treba ponovno obdelati
3. Uporabite isto predlogo nastavitev
4. Obdelajte manjši niz podatkov

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
4. ✅ **Označeni cilji** – glej [Izbiranje ciljnih slik](choosing-target-images.md)
5. ✅ **Začetek obdelave** – glej [Začetek obdelave](starting-the-processing.md)
6. ✅ **Spremljanje napredka** – glej [Spremljanje obdelave](monitoring-the-processing.md)
7. ✅ **Pregled rezultatov** – ta stran**Vaše kalibrirane, odbojno popravljene multispektralne slike so pripravljene za analizo!**

***

## Dodatni viri

### Napredne funkcije

* [**Pregledovalnik slik**](../image-viewer-gui/opening-an-image-full-screen.md) – Interaktivna vizualizacija in analiza
* [**Peskovnik indeksov/LUT**](../image-viewer-gui/index-lut-sandbox.md) – Testiranje indeksov po meri
* [**Formule multispektralnih indeksov**](../project-settings/multispectral-index-formulas.md) – Celoten referenčni indeks

### Avtomatizacija in integracija

* [**Dokumentacija CLI**](../CLI.md) – Obdelava v paketih prek ukazne vrstice
* [**Python SDK**](../api-python-sdk.md) – Programska avtomatizacija
* [**Chloros+ Funkcije**](../#chloros) – Napredne zmogljivosti obdelave

### Podpora in učenje

* [**Pogosta vprašanja**](../faq.md) – Odgovori na pogosta vprašanja
* [**Kalibracijski cilji**](../calibration-targets.md) – Razumevanje kalibracije odbojnosti
* [**Podprte kamere**](../supported-cameras.md) – Združljiva strojna oprema

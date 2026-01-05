# Zaključek obdelave

Ko Chloros zaključi obdelavo, je čas, da pregledate rezultate, preverite kakovost izhodnih podatkov in pripravite obdelane slike za uporabo v svojem delovnem procesu. Ta stran vas vodi skozi zadnje korake in naslednje ukrepe.

## Kazalnik zaključene obdelave

Ko je obdelava uspešno zaključena, se prikažejo več kazalnikov:

* ✅ **Kazalnik napredka**: doseže 100 % zaključka
* ✅ **Dnevnik odpravljanja napak**: prikaže sporočilo »Obdelava zaključena«
* ✅ **Gumb za zagon**: ponovno postane aktiven (pripravljen za naslednjo obdelavo)
* ✅ **Izhodne datoteke**: vse obdelane slike so shranjene v podmapo modela kamere***

## Iskanje obdelanih slik

### Odpiranje izhodne mape

1. Kliknite ikono **Glavni meni** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (zgoraj levo)
2. Izberite **»Odpri mapo projekta«**

3. Vaš brskalnik datotek se odpre v imeniku projekta
4. Poiščite svoj projekt po imenu

***

## Pregled obdelanih slik

### Hitri predogled v raziskovalcu datotek

**Vgrajeni predogled Windows:**

1. Prejdite v podmapo modela kamere.
2. Izberite datoteko slike.
3. Predogled se prikaže v oknu predogleda raziskovalca Windows.
4. S puščičnimi tipkami brskajte po slikah.

### Predogled v zunanjih pregledovalnikih slik

**Priporočeni pregledovalniki:*** **QGIS** – brezplačna programska oprema GIS (najboljša za georeferencirano multispektralno analizo)
* **IrfanView** – hiter, lahek pregledovalnik slik (podpira TIFF)
* **Adobe Photoshop** – profesionalno urejanje (podpora TIFF)
* **GIMP** – brezplačna alternativa programu Photoshop
* **Windows Photos** – osnovno pregledovanje (morda ne podpira 16-bitnega TIFF)

### Predogled v Chloros Image Viewer

Za napredno vizualizacijo uporabite vgrajeni Image Viewer v Chloros:

1. Kliknite na miniaturno sliko v brskalniku datotek.
2. Slika se odpre v glavnem predoglednem področju.
3. Kliknite na **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> v levem stranskem meniju.
4. Uporabite [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) za interaktivno analizo.

Podrobna navodila najdete v [Image Viewer](../image-viewer-gui/opening-an-image-full-screen.md).

***

## Pregled dnevnika odpravljanja napak

### Preverjanje opozoril ali napak

1. Odprite **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> .
2. Preglejte sporočila.
3. Poiščite rumena opozorila ali rdeče napake.
4. Preglejte vse zabeležene težave.
5. Za pomoč se obrnite na podporo MAPIR.

### Shranjevanje dnevnika

Za shranjevanje zapisa obdelave ali pošiljanje podpori MAPIR:

1. Kliknite gumb **„Kopiraj”**ali**„Prenesi”**.
2. Shranite kot besedilno datoteko v mapo projekta.
3. Priložite dokumentaciji projekta.
4. Pošljite podpori MAPIR, če se pojavijo težave.

***

## Pogoste težave z izhodnimi datotekami in rešitve

### Težava: Manjkajoče izhodne datoteke

**Možni vzroki:**

* Datoteke niso izpolnjevale meril za obdelavo.
* Samo ciljne slike (izključene iz izvoza).
* Med izvozom je zmanjkalo prostora na disku.
* Poškodba datoteke med obdelavo.

**Rešitve:**

1. Preverite dnevnik odpravljanja napak za sporočila o preskoku/napakah.
2. Preverite, ali je bilo dovolj prostora na disku.
3. Preštejte datoteke: mora se ujemati (izvirno število – ciljno število) × (indeksi + 1)
4. Ponovno uvozite in ponovno obdelajte vse manjkajoče datoteke.

### Težava: Temni ali svetli robovi (vignetting še vedno viden)

**Možni vzroki:**

* Popravek vignettinga onemogočen.
* Kamera/objektiv ni v podatkovni bazi profilov Chloros.
* Ekstremni vignetting, ki presega zmogljivosti popravka.

**Rešitve:**

1. Preverite, ali je bila popravek vinjetiranja omogočen v nastavitvah projekta.
2. Preverite, ali je bil model kamere pravilno zaznan.
3. Če vinjetiranje ostaja, se obrnite na podporo MAPIR.

### Težava: Nepravilne barve ali vrednosti

**Možni vzroki:**

* Ni zaznanih kalibracijskih ciljev.
* Izbran napačen model kalibracijskega cilja.
* Kalibracija odbojnosti onemogočena.
* Slaba kakovost ciljnih slik.

**Rešitve:**

1. Preverite, ali je bila omogočena kalibracija odbojnosti.
2. Preverite sporočila »Cilj najden« v dnevniku odpravljanja napak.
3. Preverite kakovost ciljnih slik.
4. Ponovno obdelajte z označenimi ustreznimi cilji.

### Težava: Vrednosti NDVI se zdijo napačne

**Pričakovani razponi NDVI:*** **Voda, kamnine, tla**: od -0,1 do 0,2
* **Redka/nezdrava vegetacija**: od 0,2 do 0,4
* **Zmerna vegetacija**: od 0,4 do 0,6
* **Zdrava, gosta vegetacija**: od 0,6 do 0,9**Če vrednosti presegajo ta razpon:**

1. Preverite, ali je bila uporabljena kalibracija odbojnosti.
2. Preverite, ali je bil vključen dnevnik svetlobnega senzorja.
3. Preverite, ali so bili zaznani kalibracijski cilji.
4. Preverite, ali je bil zaznan pravi model kamere.
5. Preverite čas in pogoje zajema ciljne slike.

***

## Uporaba obdelanih slik

### Za fotogrametrijo/ustvarjanje ortomosaika

**Priporočeni potek dela:**

1.**Uvozite kalibrirane slike odbojnosti** v programsko opremo za fotogrametrijo:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Ohranite metapodatke EXIF**: Preverite, ali so podatki GPS ohranjeni za geografsko označevanje.
3. **Kalibrirani delovni postopki**: Uporabite slike odbojnosti za znanstveno natančnost.
4. **Obdelajte indeksne mozaike**: Ustvarite ortomosaike NDVI iz posameznih indeksnih slik
5. **Izvozite georeferencirane GeoTIFF**: za uporabo v aplikacijah GIS

### Za analizo GIS

**Priporočeni potek dela:**

1.**Naložite v QGIS, ArcGIS ali podobno**

2.**Uporabite 16-bitne slike odbojnosti TIFF** za večpasovno analizo
3. **Uporabite indeksne slike** (NDVI, NDRE) kot pripravljene plasti vegetacije
4. **Rasterski kalkulator**: združite pasove za prilagojeno analizo
5. **Izvoz**: ustvarite klasifikacijske zemljevide, zaznavanje sprememb, zemljevide zdravja vegetacije

### Za neposredno analizo/poročanje

**Priporočeni potek dela:**

1.**Uporabite indeksne slike z barvami LUT** za vizualna poročila
2. **Izpis statistike**: povprečje NDVI na polje/parcelo
3. **Časovne vrste**: primerjajte indekse v več sejah
4. **Ustvarjanje poročil**: vključite zemljevide, statistike in vizualizacije***

## Arhiviranje in varnostno kopiranje

### Priporočena strategija varnostnega kopiranja

**Kaj shraniti:*** ✅ **Originalne slike RAW/JPG** – arhivirate na ločenem disku/v oblaku
* ✅ **Obdelani izhodi** – shranite kalibrirane slike in indekse
* ✅ **Projektna datoteka** – vsebuje vse nastavitve za ponovno obdelavo, če je potrebno
* ✅ **Dnevnik odpravljanja napak** – dokumentira podrobnosti obdelave
* ✅ **Kalibrirane ciljne slike** – za preverjanje in ponovno obdelavo**Priporočila za shranjevanje:*** **Takojšnja varnostna kopija**: zunanji trdi disk
* **Dolgoročno arhiviranje**: shranjevanje v oblaku (Google Drive, Dropbox itd.)
* **Kritični podatki**: shranite 2–3 kopije na različnih lokacijah***

## Naslednje obdelave

### Ponovna uporaba nastavitev projekta

Če boste v prihodnosti obdelovali podobne podatkovne nize:

1. **Shranite predlogo projekta** (če tega še niste storili)
2. **Ustvarite nov projekt** z uporabo shranjene predloge
3. **Uvozite nove slike**

4.**Obdelajte**z identičnimi nastavitvami za doslednost

### Skupinska obdelava več sej

Za več sej/podatkovnih nizov:**Možnost 1: GUI – več projektov**

* Ustvarite ločen projekt za vsako sejo.
* Uporabite dosledne nastavitve predloge.
* Obdelujte po eno naenkrat.

**Možnost 2: Chloros CLI (samo Chloros+)**

* Avtomatizirajte obdelavo več datotek.
* Obdelujte več map s skripti.
* Glej [CLI Dokumentacija](../CLI.md)

**Možnost 3: Python SDK (samo Chloros+)**

* Programsko nadzorovanje
* Integracija z analitičnimi potmi
* Glej [API dokumentacijo](../api-python-sdk.md)

***

## Odpravljanje težav po obdelavi

### Ponovna obdelava z drugačnimi nastavitvami

Če rezultati niso zadovoljivi:

1. Ohranite izvirne slike (nikoli jih ne izbrišite)
2. Odprite isti projekt v Chloros
3. Prilagodite nastavitve v oknu Nastavitve projekta
4. Ponovno obdelajte – izhodi bodo nadomestili prejšnje rezultate

### Obdelava podskupine slik

Če želite ponovno obdelati samo določene slike:

1. Ustvarite nov projekt
2. Uvozite samo slike, ki jih je treba ponovno obdelati
3. Uporabite isto predlogo nastavitev
4. Obdelajte manjši niz podatkov

### Pomoč

Če naletite na težave:

* 📧 **E-pošta**: info@mapir.camera (vključite dnevnik odpravljanja napak)
* 🌐 **Podpora**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Pogosta vprašanja**: [Pogosta vprašanja](../faq.md)
* 📖 **Dokumentacija**: [Chloros Priročnik](../)***

## Povzetek: Celoten potek dela

Sedaj ste zaključili celoten potek dela za obdelavo Chloros:

1. ✅ **Ustvarjen projekt** – glejte [Projekti](../projects.md)
2. ✅ **Dodane datoteke** – glej [Dodajanje datotek](adding-files-to-a-project.md)
3. ✅ **Prilagojene nastavitve** – glej [Prilagajanje nastavitev projekta](adjusting-project-settings.md)
4. ✅ **Označeni cilji** – glej [Izbira ciljnih slik](choosing-target-images.md)
5. ✅ **Začetek obdelave** – glej [Začetek obdelave](starting-the-processing.md)
6. ✅ **Spremljanje napredka** – glej [Spremljanje obdelave](monitoring-the-processing.md)
7. ✅ **Pregled rezultatov** – ta stran**Vaše kalibrirane, reflektance popravljene multispektralne slike so pripravljene za analizo!**

***

## Dodatni viri

### Napredne funkcije

* [**Pregledovalnik slik**](../image-viewer-gui/opening-an-image-full-screen.md) – Interaktivna vizualizacija in analiza
* [**Indeks/LUT Sandbox**](../image-viewer-gui/index-lut-sandbox.md) – Testiranje prilagojenega indeksa
* [**Formule multispektralnega indeksa**](../project-settings/multispectral-index-formulas.md) – popolna referenca indeksa

### Avtomatizacija in integracija

* [**CLI Dokumentacija**](../CLI.md) – obdelava v paketu prek ukazne vrstice
* [**Python SDK**](../api-python-sdk.md) – Programska avtomatizacija
* [**Chloros+ Funkcije**](../#chloros) – Napredne zmogljivosti obdelave

### Podpora in učenje

* [**Pogosta vprašanja**](../faq.md) – odgovori na pogosta vprašanja
* [**Kalibracijske tarče**](../calibration-targets.md) – razumevanje kalibracije odbojnosti
* [**Podprte kamere**](../supported-cameras.md) – Združljiva strojna oprema

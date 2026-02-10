# Nastavitev projektnih nastavitev

Pred obdelavo slik je pomembno, da konfigurirate projektne nastavitve, da ustrezajo vašim zahtevam delovnega toka. Projektne nastavitve <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> omogoča celovit nadzor nad kalibracijo, možnostmi obdelave, multispektralnimi indeksi in izvoznimi formati.

## Dostop do nastavitev projekta

1. Odprite projekt v Chloros
2. Kliknite ikono **Nastavitve projekta** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> v levem stranskem meniju
3. Plošča Nastavitve projekta prikaže vse možnosti konfiguracije

{% hint style="info" %}
**Nastavitve se samodejno shranijo** z vašim projektom. Ko ponovno odprete projekt, se vse nastavitve obnovijo.
{% endhint %}

***

## Hitra nastavitev za običajne delovne tokove

### Privzete nastavitve (priporočljive za večino uporabnikov)

Za tipične delovne tokove kamere MAPIR Survey3 so privzete nastavitve primerne:

* ✅ **Popravek vinjete**: omogočeno
* ✅ **Kalibracija odbojnosti**: omogočeno (zahteva slike ciljev MAPIR)
* ✅ **Metoda Debayer**: Standardna (hitra, srednja kakovost)
* ✅ **Izvozni format**: TIFF (16-bitni)

Preprosto uvozite slike in začnite obdelavo s temi privzetimi nastavitvami.

***

## Pregled nastavitev projekta

Plošča Nastavitve projekta je razdeljena na več kategorij. Spodaj je povzetek posameznih razdelkov. Celotno dokumentacijo najdete v [Nastavitve projekta](../project-settings/project-settings.md).

### Zaznavanje ciljev

Nadzira, kako Chloros identificira kalibracijske cilje v vaših slikah.

**Ključne nastavitve:*** **Minimalna površina vzorca za kalibracijo**: prag velikosti za zaznavanje ciljev (privzeto: 25 pikslov)
* **Minimalno združevanje ciljev**: prag podobnosti za združevanje ciljnih območij (privzeto: 60)**Kdaj prilagoditi:**

* Povečajte površino vzorca, če pride do napačnih zaznav
* Zmanjšajte, če cilji niso zaznani
* Prilagodite združevanje, če so cilji razdeljeni na več zaznav

### Obdelava

Glavne možnosti obdelave slik in kalibracije.

**Ključne nastavitve:*** **Popravek vinjete**: Kompenzira zatemnitev objektiva na robovih ✅ Priporočeno
* **Kalibracija odbojnosti**: Normalizira vrednosti z uporabo kalibracijskih ciljev ✅ Priporočeno
* **Debayerjeva metoda**: Algoritem za pretvorbo RAW v 3-kanalni multispektralni
* **Minimalni interval ponovne kalibracije**: Čas med uporabo kalibracijskih ciljev (0 = uporabi vse)**Napredne nastavitve:*** **Časovni zamik svetlobnega senzorja**: Za sinhronizacijo časa PPK (privzeto: 0)
* **Uporabi PPK popravke**: Uporabi podatke GPS/izpostavljenosti iz datotek .daq
* **Izpostavljenost 1/2**: Dodeli kamere izpostavljenosti za nastavitve z dvojno kamero

### Debayerjeva metoda

Trenutno ponujamo 2 debayerjeve metode v Chloros:

#### Standardna (hitra, srednja kakovost)

Standardni debayer deluje hitro, vendar prikazuje barvni šum debayeringa, kar povzroča manj natančne in bolj šumne slike.

#### Texture Aware (počasen, najvišja kakovost) \[Chloros+ Only]

Texture Aware uporablja visokokakovosten debayer, ki upošteva robove, v kombinaciji z modelom AI/ML za odstranjevanje šuma, ki odstrani skoraj ves šum debayeringa. Model Texture Aware za delovanje potrebuje pomnilnik GPU (VRAM). Priporočamo, da ga uporabljate, če imate na voljo več kot 4 GB VRAM za hitrejše obdelovanje.

### Indeks (multispektralni indeksi)

Nastavite, katere vegetacijske indekse želite izračunati in izvoziti.

**Kako dodati indekse:**

1. Kliknite gumb**„Dodaj indeks“**

2. Izberite indeks iz spustnega menija (NDVI, NDRE, GNDVI itd.)
3. Konfigurirajte nastavitve vizualizacije (barve LUT, razponi vrednosti)
4. Po potrebi dodajte več indeksov

**Priljubljeni indeksi:*** **NDVI**: Splošno zdravje vegetacije (najpogostejši)
* **NDRE**: Zgodnje odkrivanje stresa z RedEdge
* **GNDVI**: Občutljiv na koncentracijo klorofila
* **OSAVI**: deluje dobro z vidno tlemi
* **EVI**: regije z visokim indeksom listne površine (LAI)**Prilagojene formule (samo Chloros+):**

* Ustvarjanje prilagojenih formul za multispektralni indeks
* Uporaba matematičnih operacij s pasovi za vse kanale slike
* Shranjevanje prilagojenih formul za ponovno uporabo

Vse razpoložljive indekse in formule najdete v [Formule za multispektralni indeks](../project-settings/multispectral-index-formulas.md).

### Izvoz

Nadzoruje format in kakovost izhodne datoteke.

**Razpoložljivi formati:*** **TIFF (16-bit)**: Priporočljivo za GIS in znanstvene analize (razpon 0–65.535)
* **TIFF (32-bit, odstotek)**: Vrednosti odbojnosti s plavajočo vejico (razpon 0,0–1,0)
* **PNG (8-bit)**: Brezizgubna kompresija za vizualizacijo (razpon 0–255)
* **JPG (8-bit)**: Najmanjše datoteke, izgubna kompresija (razpon 0–255)***

## Shranjevanje in nalaganje nastavitev

### Shranjevanje predloge projekta

Ustvarite ponovno uporabne predloge za dosledne delovne tokove:

1. Konfigurirajte vse želene nastavitve v oknu Nastavitve projekta.
2. Pomaknite se do razdelka **„Shrani predlogo projekta“** na dnu.
3. Vnesite opisno ime predloge (npr. „Survey3N\_RGN\_Kmetijstvo“).
4. Kliknite ikono za shranjevanje.

**Prednosti:**

* Uporabite enake nastavitve v več projektih.
* Delite konfiguracije s člani ekipe.
* Ohranite doslednost pri ponavljajočih se raziskavah.

### Naložite predlogo v nov projekt

Pri ustvarjanju novega projekta:

1. V glavnem meniju izberite **»Nov projekt«**.
2. Izberite možnost **»Naloži iz predloge«**.
3. Izberite shranjeno predlogo.
4. Vse nastavitve se samodejno uporabijo.

### Delovni imenik

Nastavitev **„Shrani mapo projekta“** določa, kje se privzeto ustvarjajo novi projekti:

* **Privzeta lokacija**: `C:\Users\[Username]\Chloros Projects`
* **Spremeni lokacijo**: kliknite ikono za urejanje in izberite novo mapo
* **Kdaj spremeniti**:
  * Mrežni disk za sodelovanje v ekipi
  * Drugi disk z več prostora za shranjevanje
  * Urejena struktura map po letu/stranki

***

## Nastavitev PPK (Post-Processed Kinematic)

Če uporabljate snemalnike MAPIR DAQ z GPS za natančno geografsko lokacijo:

### Predpogoji

* MAPIR DAQ z modulom GPS (GNSS)
* .daq dnevniška datoteka z vnosi izpostavljenosti
* Kamera, priključena na izpostavljenostne pine DAQ med zajemom

### Koraki konfiguracije

1. Dnevniško datoteko .daq shranite v mapo projekta
2. V nastavitvah projekta omogočite **&quot;Uporabi PPK popravke&quot;**

3. Po potrebi nastavite**»Časovni zamik svetlobnega senzorja«** (privzeto: 0 za UTC)
4. Kamere dodelite izpostavljenim pinom:
   * **Ena kamera**: samodejno dodeljena pinu 1
   * **Dve kameri**: vsako kamero ročno dodelite pravemu pinu**Dodelitev izpostavljenih pinov:*** **Izpostavljenostni pin 1**: Izberite model kamere iz spustnega menija
* **Izpostavljenostni pin 2**: Izberite drugo kamero ali »Ne uporabljaj«
* Ista kamera ne more biti dodeljena obema pinoma

{% hint style="warning" %}
**Pomembno**: Izpostavljenostni pini morajo biti pravilno dodeljeni ustreznim kameram. Nepravilna dodelitev bo povzročila napačne podatke o geografski lokaciji.
{% endhint %}

***

## Napredni scenariji

### Projekti z več kamerami

Pri obdelavi slik iz več kamer MAPIR v enem projektu:

1. Chloros samodejno zazna vsak model kamere
2. Vsaka kamera dobi ustrezen profil obdelave
3. PPK: ročno dodelite vsaki kameri pravilni izpostavljenostni pin
4. Vse kamere uporabljajo isti izvozni format in indekse

**Primer**: Survey3W RGN + Survey3N OCN dvojna kamera

### Časovni preskok ali večdatumske raziskave

Za ponavljajoče se raziskave istega območja v daljšem časovnem obdobju:

1. Ustvarite predlogo s standardnimi nastavitvami.
2. V vsaki seji uporabite enotno nastavitev kalibracijskega cilja.
3. Vsak datum obdelajte kot ločen projekt.
4. Za primerljive rezultate uporabite enake nastavitve.
5. Izvozite v istem formatu za časovno analizo.

### Veliki nizi podatkov

Za projekte z veliko slikami (500+):

* Razmislite o razdelitvi na manjše projekte po datumu ali območju.
* Uporabite vzporedno obdelavo Chloros+ za hitrejše rezultate.
* Razmislite o uporabi CLI ali API za avtomatizacijo serijske obdelave.
* Prilagodite minimalni interval ponovne kalibracije, da skrajšate čas zaznavanja cilja.

***

## Preverjanje nastavitev

Preden začnete z obdelavo, preglejte te ključne nastavitve:

* [ ] Model kamere je bil pravilno zaznan v brskalniku datotek.
* [ ] Popravek vinjete je omogočen.
* [ ] Kalibracija odbojnosti je omogočena.
* [ ] Uvožena je vsaj ena slika cilja kalibracije.
* [ ] Dodani so želeni multispektralni indeksi.
* [ ] Izvozni format je primeren za vaš delovni tok.
* [ ] Nastavitve PPK so konfigurirane (če uporabljate .daq z dogodki izpostavljenosti).

***

## Naslednji koraki

Ko so nastavitve konfigurirane:

1. **Označite kalibracijske ciljne slike** – glejte [Izbira ciljnih slik](choosing-target-images.md)
2. **Začnite obdelavo** – glejte [Začetek obdelave](starting-the-processing.md)
3. **Spremljajte napredek** – glejte [Spremljanje obdelave](monitoring-the-processing.md)

Za podrobne informacije o vseh razpoložljivih nastavitvah glejte referenčno dokumentacijo [Nastavitve projekta](../project-settings/project-settings.md).

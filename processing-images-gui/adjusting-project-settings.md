# Nastavljanje projektnih nastavitev

Pred obdelavo slik je pomembno, da projektne nastavitve prilagodite svojim delovnim potrebam. <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> omogočajo celovit nadzor nad kalibracijo, možnostmi obdelave, multispektralnimi indeksi in formati izvoza.

## Dostop do nastavitev projekta

1. Odprite svoj projekt v Chloros
2. Kliknite ikono **Nastavitve projekta** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> v levem stranskem meniju
3. Plošča Nastavitve projekta prikaže vse možnosti konfiguracije

{% hint style="info" %}
**Nastavitve se samodejno shranijo** skupaj z vašim projektom. Ko ponovno odprete projekt, se vse nastavitve obnovijo.
{% endhint %}

***

## Hitra nastavitev za običajne delovne tokove

### Privzete nastavitve (Priporočeno za večino uporabnikov)

Za tipične delovne tokove s kamerami MAPIR Survey3 so privzete nastavitve primerne:

* ✅ **Popravek vinjetiranja**: Vklopljeno
* ✅ **Kalibracija odbojnosti**: Vklopljeno (zahteva slike ciljev MAPIR)
* ✅ **Metoda debayeringa**: Standardna (hitra, srednja kakovost)
* ✅ **Format izvoza**: TIFF (16-bitni)

Preprosto uvozite slike in začnite obdelavo s temi privzetimi nastavitvami.

***

## Pregled nastavitev projekta

Plošča z nastavitvami projekta je razdeljena na več kategorij. Spodaj je povzetek vsakega oddelka. Celotno dokumentacijo najdete v [Nastavitve projekta](../project-settings/project-settings.md).

### Odkrivanje ciljev

Upravlja, kako Chloros prepoznava kalibracijske cilje na vaših slikah.

**Ključne nastavitve:*** **Najmanjša površina vzorca za kalibracijo**: Prag velikosti za zaznavanje ciljev (privzeto: 25 pikslov)
* **Najmanjše združevanje ciljev**: Prag podobnosti za združevanje ciljnih območij (privzeto: 60)**Kdaj prilagoditi:**

* Povečajte površino vzorca, če se pojavljajo napačne zaznave
* Zmanjšajte, če cilji niso zaznani
* Prilagodite združevanje, če se cilji razdelijo na več zaznav

### Obdelava

Glavne možnosti obdelave slik in kalibracije.

**Ključne nastavitve:*** **Popravek vinjetiranja**: Kompenzira potemnitev objektiva na robovih ✅ Priporočeno
* **Kalibracija odbojnosti**: Normalizira vrednosti z uporabo kalibracijskih ciljev ✅ Priporočeno
* **Metoda Debayer**: Algoritem za pretvorbo RAW v 3-kanalni multispektralni format
* **Minimalni interval ponovne kalibracije**: Čas med uporabo kalibracijskih ciljev (0 = uporabi vse)**Napredne nastavitve:*** **Časovni zamik svetlobnega senzorja**: Za sinhronizacijo časa PPK (privzeto: 0)
* **Uporabi PPK popravke**: Uporabi podatke GPS/izpostavljenosti iz .daq datotek
* **Izpostavljenost 1/2**: Dodeli kamere izpostavljenostnim pinom za konfiguracije z dvema kamerama

### Metoda Debayer

Trenutno v Chloros ponujamo 2 metode debayeringa:

#### Standardna (hitra, srednja kakovost)

Standardni debayer deluje hitro, vendar prikazuje barvni šum debayeringa, kar povzroča manj natančne in bolj šumne slike.

#### Texture Aware (počasen, najvišja kakovost) \[Samo Chloros+]

Texture Aware uporablja visokokakovostni debayer, ki upošteva robove, v kombinaciji z modelom za odstranjevanje šuma AI/ML, ki odstrani skoraj ves šum debayeringa. Model Texture Aware za delovanje potrebuje pomnilnik GPU (VRAM). Priporočamo, da ga uporabljate, če imate na voljo več kot 4 GB VRAM za hitrejše obdelovanje.

### Indeks (multispektralni indeksi)

Nastavite, katere vegetacijske indekse želite izračunati in izvoziti.

**Kako dodati indekse:**

1. Kliknite gumb**&quot;Dodaj indeks&quot;**

2. Izberite indeks iz spustnega menija (NDVI, NDRE, GNDVI itd.)
3. Nastavite vizualizacijske nastavitve (barve LUT, razponi vrednosti)
4. Po potrebi dodajte več indeksov

**Priljubljeni indeksi:*** **NDVI**: Splošno zdravje vegetacije (najpogostejši)
* **NDRE**: Zgodnje odkrivanje stresa z RedEdge
* **GNDVI**: Občutljiv na koncentracijo klorofila
* **OSAVI**: Dobro deluje pri vidni tleh
* **EVI**: Območja z visokim indeksom listne površine (LAI)**Prilagojene formule (samo Chloros+):**

* Ustvarjanje prilagojenih formul za multispektralne indekse
* Uporaba matematičnih operacij s pasovi za vse kanale slike
* Shranjevanje prilagojenih formul za ponovno uporabo

Za vse razpoložljive indekse in formule glejte [Formule za multispektralne indekse](../project-settings/multispectral-index-formulas.md).

### Izvoz

Nadzoruje format in kakovost izhodne datoteke.

**Razpoložljivi formati:*** **TIFF (16-bitni)**: Priporočljiv za GIS in znanstvene analize (razpon 0–65.535)
* **TIFF (32-bitni, odstotki)**: Vrednosti odbojnosti s plavajočo vejico (razpon 0,0–1,0)
* **PNG (8-bit)**: Brezizgubno stiskanje za vizualizacijo (razpon 0–255)
* **JPG (8-bit)**: Najmanjše datoteke, izgubno stiskanje (razpon 0–255)***

## Shranjevanje in nalaganje nastavitev

### Shranite predlogo projekta

Ustvarite ponovno uporabne predloge za dosledne delovne tokove:

1. Konfigurirajte vse želene nastavitve v oknu Nastavitve projekta
2. Pomaknite se do razdelka **&quot;Shrani predlogo projekta&quot;** na dnu
3. Vnesite opisno ime predloge (npr. &quot;Survey3N\_RGN\_Kmetijstvo&quot;)
4. Kliknite ikono za shranjevanje

**Prednosti:**

* Uporabite enake nastavitve v več projektih
* Delite konfiguracije s člani ekipe
* Ohranite doslednost pri ponavljajočih se anketah

### Naložite predlogo v nov projekt

Pri ustvarjanju novega projekta:

1. Iz glavnega menija izberite **»Nov projekt«**

2. Izberite možnost**»Naloži iz predloge«**

3. Izberite shranjeno predlogo
4. Vse nastavitve se samodejno uporabijo

### Delovni imenik

Nastavitev **&quot;Shrani mapo projekta&quot;** določa, kje se privzeto ustvarjajo novi projekti:

* **Privzeta lokacija**: `C:\Users\[Username]\Chloros Projects`
* **Spremeni lokacijo**: Kliknite ikono za urejanje in izberite novo mapo
* **Kdaj spremeniti**:
  * Mrežni pogon za sodelovanje v ekipi
  * Drug pogon z več prostora za shranjevanje
  * Urejena struktura map po letu/stranki

***

## Nastavitev PPK (Post-Processed Kinematic)

Če uporabljate snemalnike DAQ MAPIR z GPS za natančno geolokacijo:

### Predpogoji

* MAPIR DAQ z modulom GPS (GNSS)
* .daq dnevniška datoteka z vnosi izpostavljenih pinov
* Kamera, priključena na izpostavljene pine DAQ med snemanjem

### Koraki konfiguracije

1. Dnevniško datoteko .daq shranite v mapo projekta
2. V nastavitvah projekta omogočite potrditveno polje **&quot;Uporabi PPK popravke&quot;**

3. Po potrebi nastavite**&quot;Časovni zamik svetlobnega senzorja&quot;** (privzeto: 0 za UTC)
4. Dodelite kamere priključnim točkam za osvetlitev:
   * **Ena kamera**: samodejno dodeljena priključni točki 1
   * **Dve kameri**: ročno dodelite vsako kamero pravi priključni točki**Dodelitev priključnih točk za osvetlitev:*** **Pin za osvetlitev 1**: Izberite model kamere iz spustnega menija
* **Pin za osvetlitev 2**: Izberite drugo kamero ali »Ne uporabljaj«
* Ista kamera ne more biti dodeljena obema pinoma

{% hint style="warning" %}
**Pomembno**: Pini za osvetlitev morajo biti pravilno dodeljeni ustreznim kameram. Nepravilna dodelitev bo povzročila napačne podatke o geografski lokaciji.
{% endhint %}

***

## Napredni scenariji

### Projekti z več kamerami

Pri obdelavi slik iz več kamer MAPIR v enem projektu:

1. Chloros samodejno zazna vsak model kamere
2. Vsaka kamera dobi ustrezen profil obdelave
3. PPK: Vsaki kameri ročno dodelite pravilni pin za osvetlitev
4. Vse kamere uporabljajo isti izvozni format in indekse

**Primer**: Survey3W RGN + Survey3N OCN naprava z dvema kamerama

### Posnetki s časovnim zamikom ali raziskave z več datumi

Za ponavljajoče se raziskave istega območja v daljšem časovnem obdobju:

1. Ustvarite predlogo s svojimi standardnimi nastavitvami
2. Pri vsaki seji uporabite enako nastavitev kalibracijskega cilja
3. Obdelajte vsak datum kot ločen projekt
4. Uporabite enake nastavitve za primerljive rezultate
5. Izvozite v enakem formatu za časovno analizo

### Veliki nizi podatkov

Za projekte z veliko slikami (500+):

* Razmislite o razdelitvi na manjše projekte po datumu ali območju
* Uporabite vzporedno obdelavo Chloros+ za hitrejše rezultate
* Razmislite o CLI ali API za avtomatizacijo obdelave v serijah
* Prilagodite minimalni interval ponovne kalibracije, da skrajšate čas zaznavanja ciljev

***

## Preverjanje nastavitev

Preden začnete z obdelavo, preverite te ključne nastavitve:

* [ ] Model kamere je pravilno zaznan v brskalniku datotek
* [ ] Popravek vinjete je omogočen
* [ ] Kalibracija odbojnosti je omogočena
* [ ] Uvožena je vsaj ena slika kalibracijskega cilja
* [ ] Dodani so željeni multispektralni indeksi
* [ ] Izvozni format je primeren za vaš delovni tok
* [ ] Nastavitve PPK so konfigurirane (če uporabljate .daq z dogodki osvetlitve)

***

## Naslednji koraki

Ko so nastavitve konfigurirane:

1. **Označite ciljne slike za kalibracijo** – glejte [Izbira ciljnih slik](choosing-target-images.md)
2. **Začnite obdelavo** – glejte [Začetek obdelave](starting-the-processing.md)
3. **Spremljajte napredek** – glejte [Spremljanje obdelave](monitoring-the-processing.md)

Za podrobne informacije o vseh razpoložljivih nastavitvah glejte referenčno dokumentacijo [Nastavitve projekta](../project-settings/project-settings.md).

# Nastavljanje projektnih nastavitev

Pred obdelavo slik je pomembno, da projektne nastavitve prilagodite svojim zahtevam glede delovnega toka. Plošča »Projektne nastavitve« (<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">) omogoča celovit nadzor nad kalibracijo, možnostmi obdelave, multispektralnimi indeksi in formati izvoza.

## Dostop do nastavitev projekta

1. Odprite svoj projekt v Chloros
2. V levem stranskem meniju kliknite ikono **Nastavitve projekta** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">
3. V oknu »Nastavitve projekta« so prikazane vse možnosti konfiguracije

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>Plošča »Nastavitve projekta« — Prikaz, zaznavanje ciljev in obdelava</p></figcaption></figure>{% hint style="info" %}
**Nastavitve se samodejno shranijo** skupaj z vašim projektom. Ko ponovno odprete projekt, se vse nastavitve obnovijo.
{% endhint %}

***

## Hitra nastavitev za običajne delovne tokove

### Privzete nastavitve (priporočene za večino uporabnikov)

Privzete nastavitve dobro delujejo za tipične delovne tokove Survey3 in LATTICE:

* ✅ **Korekcija vinjete**: Vklopljena
* ✅ **Kalibracija odbojnosti / uravnoteženje beline**: Vklopljena (uporablja tarče MAPIR in/ali podatke svetlobnega senzorja DAQ)
* ✅ **Metoda debayiranja**: Standardna (hitra, srednja kakovost)
* ✅ **Format izvoza**: TIFF (16-bitni)
* ✅ **Vsi izvozni izdelki**: Omogočeno (LATTICE samodejno zajame razširitev v debayered, predogled, sevanje in odbojnost)

Preprosto uvozite svoje slike in začnite obdelavo s temi privzetimi nastavitvami.

***

## Pregled nastavitev projekta

Plošča »Nastavitve projekta« je razdeljena na spodnje razdelke. Dva dodatna razdelka — **DAQ Light Sensor**in**Poravnava matrike** — se samodejno prikažeta, če vaš projekt vsebuje ustrezne datoteke. Celotno dokumentacijo najdete v [Nastavitvah projekta](../project-settings/project-settings.md).

### Prikaz

* **Ločljivost sličic slik**: Ločljivost sličic v mreži slik. Možnosti:**Privzeto (512 px)**,**1024 px**,**2048 px**,**Polna ločljivost**. Samo za prikaz — nikoli ne vpliva na obdelavo. Višje vrednosti so pri povečavi videti ostrejše, vendar se nalagajo počasneje.

### Prepoznavanje ciljev

Upravlja, kako Chloros prepoznava kalibracijske cilje na vaših slikah.

**Ključne nastavitve:*** **Najmanjša površina kalibracijskega vzorca (px)**: Prag velikosti za zaznavanje ciljev (privzeto:**25**, razpon 0–10000)
* **Najmanjša stopnja združevanja ciljev (0–100)**: Prag podobnosti za združevanje območij ciljev (privzeto:**60**)**Kdaj prilagoditi:**

* Povečajte območje vzorca, če se pojavljajo napačna zaznavanja
* Zmanjšajte ga, če se cilji ne zaznavajo
* Prilagodite združevanje v skupine, če se cilji razdelijo na več zaznavanj

{% hint style="info" %}
Te nastavitve so sive, če je **Kalibracija odbojnosti / belina** izklopljena — če je izklopljena, zaznavanje ciljev sploh ne deluje.
{% endhint %}

### Obdelava

Glavne možnosti za obdelavo slik in kalibracijo.

**Ključne nastavitve:*** **Popravek vinjetiranja**: Kompenzira potemnitev objektiva na robovih ✅ Priporočeno
* **Kalibracija odbojnosti / belina**: Kalibrira slike z uporabo zaznanih ciljev (Survey3) in/ali podatkov svetlobnega senzorja DAQ (LATTICE) ✅ Priporočeno
* **Metoda debayeringa**: Algoritem za pretvorbo RAW v 3-kanalni multispektralni format
* **Najkrajši interval ponovne kalibracije**: Najkrajši čas v sekundah med uporabo kalibracijskega cilja (privzeto:**0** = uporabi vse, razpon 0–3600)**Nekalibrirani nadomestni izdelki:**Ko slike ni mogoče kalibrirati glede na odbojnost (cilj ni na voljo ali je kalibracija onemogočena), se izvozi kot eden od dveh nadomestnih izdelkov —**na vsakem izvajanju obstaja natanko eden od para**, izbran s stikalom za popravek vinjete:

* **Izvoz odziva senzorja**: zapiše `Sensor_Response_Images` — uporablja se, ko je korekcija vinjete**izklopljena*** **Izvoz s popravkom vinjete**: zapiše `Vignette_Corrected_Images` — uporablja se, ko je popravek vinjete**vklopljen**Potrditveno polje, ki ni v uporabi, je sivo. Če odkljukate aktivno polje, se ta datoteka sploh ne zapiše.**Izdelki za izvoz LATTICE** (prikazani za vsak projekt; veljajo za posnetke LATTICE):

* **Izvoz brez bayera**: linearna slika brez bayera (`Debayered_Images`). Velja za RGB in multispektralne module.
* **Izvoz predogleda**: predogled na zaslonu (`Preview_Images`). RGB = belina (DAQ-svetlobni vir, če je na voljo, sicer siva skala) + gama; multispektralni = raztezek v lažnih barvah.
* **Izvoz sevanja**: spektralno sevanje float32 (`Radiance_Images`, W/m²/sr/nm). Samo za multispektralne module — ne velja za glavne module RGB.
* **Izvoz odbojnosti**: odbojnost uint16 (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0), kadar okvir pokriva odčitek navzdol usmerjenega sevanja `.daq` ali cilj znotraj okvirja. Samo za multispektralne module.

Vse štiri so **privzeto vklopljene**– en uvožen surovi okvir LATTICE se v enem samem ciklu obdelave razširi na vsak omogočen in ustrezen izdelek. Polje**Izvoz odbojnosti** je sivo, če je kalibracija odbojnosti izklopljena. Nastavitve, ki jih nadrejeno stikalo onemogoča, so vedno sive, ob njih pa se prikaže namig z imenom stikala, ki ga je treba spremeniti.**Napredne nastavitve:*** **Časovni zamik svetlobnega senzorja**: Število ur od UTC za usklajevanje časa svetlobnega senzorja (privzeto: 0, razpon od −12 do +12)
* **Uporabi PPK-popravke**: Uporablja podatke GPS/izpostavljenostnih pinov iz datotek `.daq` (privzeto: izklopljeno)
* **Izpostavljenostni pini 1/2**: Dodeli kamere izpostavljenostnim pinom za konfiguracije z dvema kamerama

{% hint style="info" %}
**Raven vhoda LATTICE je samodejna.** Posnetki LATTICE nosijo svojo raven obdelave v metapodatkih XMP, obdelava pa vedno vstopi v procesni tok na ravni surovega okvirja — v grafičnem vmesniku ni treba ničesar konfigurirati. (Zastavica CLI `--input-level` obstaja kot možnost prepisovanja za napredne uporabnike v primeru zajemov z izgubljenimi metapodatki; glej [CLI Referenco](../reference/cli-reference.md).)
{% endhint %}

### Metoda odstranjevanja pik

Trenutno v Chloros ponujamo 2 metode odstranjevanja pik:

#### Standardna (hitra, srednja kakovost)

Standardna metoda odstranjevanja pik deluje hitro, vendar pri tem nastane barvni šum, kar povzroči manj natančne in bolj šumne slike.

#### Texture Aware (počasen, najvišja kakovost) \[Samo Chloros+]

Texture Aware uporablja visokokakovostno odstranjevanje pik, ki upošteva robove, v kombinaciji z modelom za odstranjevanje šuma AI/ML, ki odstrani skoraj ves šum, povezan z odstranjevanjem pik. Model za delovanje potrebuje pomnilnik grafične kartice (VRAM): z **7 GB ali več VRAM-a** lahko hkrati obdeluje več slik; pod 7 GB obdeluje po eno sliko naenkrat (opazno počasneje). Glejte [Dinamično prilagajanje računalniške moči](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**Posnetki LATTICE vedno uporabljajo standardno demosaiciranje.** Ker ni modela Texture Aware, usposobljenega za LATTICE, ta možnost ni na voljo za slike LATTICE — slike Survey3 v istem projektu pa ga še vedno lahko uporabljajo.
{% endhint %}

### Indeks (multispektralni indeksi)

Nastavite, katere vegetacijske indekse želite izračunati in izvoziti. Spustni meni v grafičnem uporabniškem vmesniku ponuja **27 vnaprej določenih formul za indekse**.**Kako dodati indekse:**

1. Kliknite gumb**»Dodaj indeks««**

2. Iz spustnega menija izberite indeks (NDVI, NDRE, GNDVI itd.)
3. Nastavite vizualizacijo (barve LUT, vrednostna območja)
4. Po potrebi dodajte več indeksov

**Pogosti indeksi:*** **NDVI**: Splošno zdravstveno stanje vegetacije (najpogostejši)
* **NDRE**: Zgodnje odkrivanje stresa z indeksom RedEdge
* **GNDVI**: Občutljiv na koncentracijo klorofila
* **OSAVI**: Dobro deluje pri vidni tleh
* **EVI**: Območja z visokim indeksom listne površine (LAI)**Lastne formule:**

* Ustvarite lastne formule multispektralnih indeksov z matematičnimi operacijami na vseh kanalih slike
* Shranite lastne formule za ponovno uporabo
* Lastne formule so funkcija Chloros+; razpoložljivost je odvisna od vaše stopnje naročniškega paketa

Za vse razpoložljive indekse in formule – vključno s tem, katera imena so na voljo le v grafičnem vmesniku (GUI) in katera delujejo tudi v CLI/SDK – glejte [Formule multispektralnih indeksov](../project-settings/multispectral-index-formulas.md).

### Izvoz

Nadzira format izhodne datoteke.

**Razpoložljivi formati**(nastavitev:**Format kalibrirane slike**, privzeto**TIFF (16-bitni)**):

* **TIFF (16-bitni)**: Priporočljiv za GIS in znanstvene analize
* **TIFF (32-bitni, v odstotkih)**: Vrednosti s plavajočo vejico
* **PNG (8-bit)**: stiskanje brez izgube za vizualizacijo
* **JPG (8-bit)**: najmanjše datoteke, stiskanje z izgubo

Izhodne datoteke se shranijo v projektni mapi, razvrščene po kameri in formatu: `<project>/<camera>/<format>/<Product>_Images/`. Sijaj se **vedno** shrani kot float32 v mapo `tiff32`, ne glede na to nastavitev. Izvožene datoteke ohranijo ime izvorne datoteke — mapo, ki identificira izdelek. Celotno drevo izhodnih datotek si oglejte v [Zaključevanje obdelave](finishing-the-processing.md).

{% hint style="warning" %}
**Branje vrednosti odbojnosti**: DN, ki pomeni ρ = 1,0, je odvisen od izvorne kamere — LATTICE uporablja 32768 (označeno kot XMP `Chloros:PixelScale`), Survey3 pa 65535. Preberite oznako, namesto da predpostavljate konstanto. Glejte [Formati izhodnih slik](../output-image-formats.md).
{% endhint %}

### Senzor svetlobe DAQ

V tem poglavju so navedene vse datoteke DAQ o padajočem sončnem sevanju (`.daq` / `.csv`) v vašem projektu, po ena vrstica na datoteko, s prikazom modela senzorja, imena datoteke in popravka **pokrova** difuzorja, ki velja za to datoteko.

* **Prepis omejitve (vse datoteke)**: en sam spustni meni za celoten projekt.**Auto** (privzeto) uporabi zabeleženo omejitev vsake datoteke – če ni nič zabeleženo, se predpostavlja sončno vreme, saj so vsi DAQ-ji MAPIR opremljeni s korektorjem za sončno vreme. Izbor zgornje meje prevlada nad vsako datoteko: surove zapise popravi z njo, zapisi, ki že vsebujejo zgornjo mejo, pa se ponovno referencirajo (zabeležena korekcija se razveljavi, uporabi se izbrana zgornja meja).
* V vrsticah se prikaže opozorilo, če je bila zabeležena zgornja meja privzeta vrednost vozlišča in ni bila potrjena s strani operaterja, ter če izbrana zgornja meja nima profila za ta model naprave (prepis za to datoteko je zavrnjen).

Zapisi DAQ, narejeni v zavihku Svetlobni senzorji, se samodejno dodajo v odprti projekt, uvožene datoteke `.daq` / `.csv` pa se tukaj prikažejo takoj, ko so dodane.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Spodnje nastavitve projekta — Indeks, format izvoza, razdelek DAQ svetlobnih senzorjev ter nadzorni elementi za predloge/mape projekta</p></figcaption></figure>### Poravnava matrik

Ta razdelek se prikaže **samo**, če vsaj ena slika v projektu vsebuje transformacijo za poravnavo med moduli, ki jo matrike LATTICE vtisnejo ob zajemu (`Chloros:Alignment*` XMP). Prikaže, koliko slik vsebuje oznake in katera kamera je referenčna, s temi nadzornimi elementi:

* **Uporabi poravnavo matrike** (privzeto: vklopljeno): vsak obdelan produkt (debayering / predogled / sevanje / odbojnost / indeks) poravna v skupno referenčno geometrijo matrike. Izklopljeno = izvoz v izvirni geometriji senzorja.
* **Obreži na skupno prekrivanje** (privzeto: vklopljeno): izvožene usklajene slike obreže na območje, ki je skupno vsem modulom, tako da ima vsak pas enak odtis. Izklop ohrani celotno platno senzorja (črna zapolnitev zunaj vira).
* **Ponovno vzorčenje**:**Bilinearno (gladko, privzeto)**,**Najbližje (ohranjanje natančnih vrednosti)**— brez mešanja med pikslami, za strogo radiometrično analizo — ali**Kubično (najostrejše)**.***

## Shranjevanje in nalaganje nastavitev

### Shranjevanje predloge projekta

Ustvarite ponovno uporabne predloge za dosledne delovne tokove:

1. V panelu »Nastavitve projekta« konfigurirajte vse želene nastavitve
2. Pomaknite se do oddelka **»Shrani predlogo projekta«** na dnu
3. Vnesite opisno ime predloge (npr. »Survey3N\_RGN\_Agriculture«)
4. Kliknite ikono za shranjevanje

**Prednosti:**

* Uporaba identičnih nastavitev v več projektih
* Delite konfiguracije s člani ekipe
* Ohranite doslednost pri ponavljajočih se anketah

### Naložite predlogo v nov projekt

Pri ustvarjanju novega projekta:

1. V glavnem meniju izberite **»Nov projekt«**

2. V izbirniku predlog (če je na voljo) izberite predlogo projekta
3. Vse nastavitve iz predloge se samodejno uporabijo

### Delovni imenik

Nastavitev **»Delovni imenik«** določa, kje se novi projekti privzeto ustvarjajo:

* **Privzeta lokacija**: `C:\Users\[Username]\Chloros Projects`
* **Spremeni lokacijo**: Kliknite ikono za urejanje in izberite novo mapo
* **Skupna raba z CLI**: `chloros-cli` uporablja isto privzeto nastavitev mape za projekte
* **Kdaj spremeniti**:
  * Mrežni pogon za sodelovanje v ekipi
  * Drugi pogon z več prostora za shranjevanje
  * Urejena struktura map po letu/stranki

***

## Nastavitev PPK (Post-Processed Kinematic)

Če uporabljate snemalnike MAPIR DAQ z GPS za natančno geolokacijo:

### Predpogoji

* Naprava za zajem podatkov (DAQ) MAPIR z modulom GPS (GNSS)
* Dnevniška datoteka .daq z vnosi o izpostavljenosti
* Kamera, priključena na priključke za izpostavljenost naprave DAQ med zajemom

### Koraki konfiguracije

1. Datoteko dnevnika .daq shranite v mapo projekta
2. V nastavitvah projekta označite potrditveno polje **»Uporabi PPK popravke«**

3. Po potrebi nastavite**»Časovni zamik svetlobnega senzorja«** (privzeto: 0 za UTC)
4. Dodelite kamere priključnim točkam za osvetlitev:
   * **Ena kamera**: samodejno dodeljena priključni točki 1
   * **Dve kameri**: vsako kamero ročno dodelite ustrezni priključni točki**Dodelitev priključnih točk za osvetlitev:*** **Pin za osvetlitev 1**: Iz spustnega menija izberite model kamere
* **Pin za osvetlitev 2**: Izberite drugo kamero ali »Ne uporabljaj«
* Eni in isti kameri ni mogoče dodeliti obeh pinov

{% hint style="warning" %}
**Pomembno**: Pini za osvetlitev morajo biti pravilno dodeljeni ustreznim kameram. Nepravilna dodelitev bo povzročila napačne podatke o geografski lokaciji.
{% endhint %}

***

## Napredni scenariji

### Projekti z več kamerami

Pri obdelavi slik iz več kamer MAPIR v enem projektu:

1. Chloros samodejno zazna vsak model kamere (tako Survey3 kot LATTICE)
2. Vsaka kamera dobi ustrezne profile obdelave in svojo lastno drevesno strukturo izhodnih map
3. PPK: Vsaki kameri Survey3 ročno dodelite pravilni pin za osvetlitev
4. Vse kamere uporabljajo isti format izvoza in indekse

**Primeri**: Survey3W RGN + Survey3N OCN konfiguracija z dvema kamerama, ali LATTICE-matrika, ki združuje glavno kamero RGB z ozkopasovnimi moduli

### Posnetki s časovnim zamikom ali raziskave z več datumi

Za ponavljajoče se raziskave istega območja v daljšem časovnem obdobju:

1. Ustvarite predlogo s svojimi standardnimi nastavitvami
2. Pri vsaki seji uporabite enako nastavitev kalibracijskih ciljev
3. Vsak datum obdelajte kot ločen projekt
4. Uporabite enake nastavitve za primerljive rezultate
5. Izvozite v enak format za časovno analizo

### Veliki nizi podatkov

Za projekte z veliko slikami (500+):

* Razmislite o razdelitvi na manjše projekte po datumu ali območju
* Uporabite vzporedno obdelavo Chloros+ za hitrejše rezultate
* Razmislite o uporabi CLI ali API za avtomatizacijo obdelave v serijah
* Prilagodite minimalni interval ponovne kalibracije, da skrajšate čas zaznavanja ciljev

***

## Preverjanje nastavitev

Preden začnete z obdelavo, preverite te ključne nastavitve:

* [ ] Model kamere je pravilno zaznan v brskalniku datotek
* [ ] Popravek vinjetiranja je omogočen
* [ ] Kalibracija odbojnosti je omogočena
* [ ] Za Survey3: uvožena in preverjena je vsaj ena slika kalibracijskega cilja; za LATTICE: prisoten je cilj in/ali posnetek navzdol usmerjenega sevanja `.daq`
* [ ] Želeni multispektralni indeksi so dodani
* [ ] Izvozni format je primeren za vaš delovni tok
* [ ] Nastavitve PPK so konfigurirane (če uporabljate datoteke .daq z izpostavljenostnimi dogodki)

***

## Naslednji koraki

Ko so vaše nastavitve konfigurirane:

1. **Označite kalibracijske ciljne slike** – glejte [Izbira ciljnih slik](choosing-target-images.md)
2. **Začnite obdelavo** – glejte [Začetek obdelave](starting-the-processing.md)
3. **Spremljajte napredek** – glejte [Spremljanje obdelave](monitoring-the-processing.md)

Za podrobnejše informacije o vseh razpoložljivih nastavitvah si oglejte referenčno dokumentacijo [Nastavitve projekta](../project-settings/project-settings.md).

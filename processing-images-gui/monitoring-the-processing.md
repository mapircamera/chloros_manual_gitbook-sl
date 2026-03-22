# Spremljanje obdelave

Ko se obdelava začne, Chloros ponuja več načinov za spremljanje napredka, preverjanje morebitnih težav in razumevanje dogajanja z vašim naborom podatkov. Na tej strani je pojasnjeno, kako spremljati obdelavo in razlagati informacije, ki jih zagotavlja Chloros.

## Pregled kazalnika napredka

Kazalnik napredka v zgornjem naslovu prikazuje stanje obdelave v realnem času in odstotek dokončanja.

### Kazalnik napredka v brezplačnem načinu

Za uporabnike brez licence Chloros+:

**Prikaz napredka v 2 fazah:**

1.**Zaznavanje ciljev** – iskanje kalibracijskih ciljev v slikah
2. **Obdelava** – Uporaba popravkov in izvoz**Napredovalna vrstica prikazuje:**

* Splošni odstotek dokončanja (0–100 %)
* Ime trenutne stopnje
* Preprosta vizualizacija v obliki vodoravne vrstice

### Napredovalna vrstica Chloros+

Za uporabnike z licenco Chloros+:

**4-stopenjski prikaz napredka:**

1.**Zaznavanje** – iskanje kalibracijskih ciljev
2. **Analiza** – pregledovanje slik in priprava poteka
3. **Kalibracija** – uporaba popravkov vinjete in odbojnosti
4. **Izvoz** – shranjevanje obdelanih datotek**Interaktivne funkcije:*** **Premaknite miško nad** kazalnik napredka, da se prikaže razširjen 4-stopenjski panel
* **Kliknite** kazalnik napredka, da zamrznete/pritrdite razširjeni panel
* **Kliknite še enkrat**, da odmrznete in se samodejno skrije, ko miško odstranite
* Vsaka stopnja prikazuje posamezni napredek (0–100 %)

***

## Razumevanje posameznih stopenj obdelave

{% hint style="info" %}
**Arhitektura poteka**: Te 4 stopnje grafičnega vmesnika ustrezajo [4-nitnemu poteku obdelave](../processing-architecture/processing-pipeline.md). Na sistemih s pospešitvijo GPU nit 3 (Kalibriranje) izkorišča [dinamično prilagajanje računanja](../processing-architecture/dynamic-compute-adaptation.md), ki optimizira obdelavo za vašo specifično strojno opremo.
{% endhint %}

### Stopnja 1: Zaznavanje (zaznavanje ciljev)

**Kaj se dogaja:**

* Chloros pregleda slike, označene s potrditvenim poljem Cilj
* Algoritmi računalniškega vida identificirajo 4 kalibracijske plošče
* Iz vsake plošče se izločijo vrednosti odbojnosti
* Za pravilno načrtovanje kalibracije se zabeležijo časovni žigi ciljev

**Trajanje:**

* Z označenimi cilji: 10–60 sekund
* Brez označenih ciljev: 5–30+ minut (skenira vse slike)

**Kazalnik napredka:**

* Odkrivanje: 0 % → 100 %
* Število skeniranih slik
* Število najdenih ciljev

**Na kaj je treba paziti:**

* Če so cilji pravilno označeni, naj bi se postopek hitro zaključil
* Če traja predolgo, cilji morda niso označeni
* V dnevniku za odpravljanje napak preverite sporočila »Cilj najden«

### Faza 2: Analiza

**Kaj se dogaja:**

* Branje metapodatkov EXIF slike (časovni žigi, nastavitve osvetlitve)
* Določanje strategije kalibracije na podlagi časovnih žigov ciljev
* Urejanje čakalne vrste za obdelavo slik
* Priprava delavcev za vzporedno obdelavo (samo Chloros+)

**Trajanje:** 5–30 sekund**Kazalnik napredka:**

* Analiza: 0 % → 100 %
* Hitra faza, običajno se hitro zaključi

**Na kaj je treba paziti:**

* Napredek mora potekati enakomerno brez premorov
* Opozorila o manjkajočih metapodatkih se bodo pojavila v dnevniku za odpravljanje napak

### Faza 3: Kalibriranje

**Kaj se dogaja:*** **Debayering**: Pretvorba RAW-vzorca Bayer v 3 kanale
* **Popravek vinjetiranja**: Odstranjevanje potemnitve robov objektiva
* **Kalibracija odbojnosti**: Normalizacija s ciljnimi vrednostmi
* **Izračun indeksa**: Izračun multispektralnih indeksov
* Obdelava vsake slike skozi celoten proces

**Trajanje:** Večina skupnega časa obdelave (60–80 %)**Kazalnik napredka:**

* Kalibriranje: 0 % → 100 %
* Trenutna slika v obdelavi
* Obdelane slike / Skupno število slik

**Način obdelave:*** **Prosti način**: Obdeluje po eno sliko naenkrat zaporedno
* **Način Chloros+**: Obdeluje do 16 slik hkrati
* **Pospešitev z GPU**: znatno pospeši to fazo**Na kaj je treba paziti:**

* Enakomeren napredek skozi število slik
* Preverite dnevnik razhroščevanja za sporočila o zaključku posameznih slik
* Opozorila o kakovosti slik ali težavah s kalibracijo

### Faza 4: Izvoz

**Kaj se dogaja:**

* Zapišanje kalibriranih slik na disk v izbranem formatu
* Izvoz multispektralnih indeksnih slik z barvami LUT
* Ustvarjanje podmap za modele kamer
* Ohranjanje izvirnih imen datotek z ustreznimi končnicami

**Trajanje:** 10–20 % skupnega časa obdelave**Kazalnik napredka:**

* Izvoz: 0 % → 100 %
* Zapisovanje datotek
* Izvozni format in ciljna lokacija

**Na kaj je treba paziti:**

* Opozorila o prostem prostoru na disku
* Napake pri pisanju datotek
* Zaključek vseh nastavljenih izhodov

***

## Zavihek Dnevnik odpravljanja napak

Dnevnik odpravljanja napak zagotavlja podrobne informacije o napredku obdelave in morebitnih težavah.

### Dostop do dnevnika odpravljanja napak

1. Kliknite ikono **Dnevnik odpravljanja napak** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> v levem stranskem meniju
2. Odpre se okno z dnevnikom, ki prikazuje sporočila o obdelavi v realnem času
3. Samodejno se pomika navzdol, da prikaže najnovejša sporočila

### Razumevanje sporočil v dnevniku

#### Informacijska sporočila (bela/siva)

Normalne posodobitve obdelave:

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Opozorilna sporočila (rumena)

Nekritične težave, ki ne ustavijo obdelave:

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Ukrep:** Po obdelavi preglejte opozorila, vendar ne prekinjajte obdelave

#### Sporočila o napakah (Red)

Kritične težave, ki lahko povzročijo neuspešno obdelavo:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Ukrep:** Prekinite obdelavo, odpravite napako in ponovno zaženite

### Pogosta sporočila v dnevniku

| Sporočilo                          | Pomen                                | Potrebni ukrep                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| &quot;Cilj zaznan v \[ime datoteke]&quot; | Kalibracijski cilj uspešno najden  | Nobenega – normalno                                         |
| &quot;Obdelava slike X od Y&quot;        | Trenutna posodobitev napredka                | Nobenega – normalno                                         |
| &quot;Ciljev ni bilo najdenih&quot;               | Kalibracijskih ciljev ni bilo zaznanih        | Označite ciljne slike ali onemogočite kalibracijo odbojnosti |
| &quot;Premajhen prostor na disku&quot;        | Premajhno prostora za izhodne podatke          | Osvobodite prostor na disku                                    |
| &quot;Preskočitev poškodovane datoteke&quot;        | Datoteka slike je poškodovana                  | Ponovno kopirajte datoteko s kartice SD                             |
| &quot;PPK podatki uporabljeni&quot;               | Uporabljene so bile GPS popravke iz datoteke .daq | Nobenih – normalno                                         |

### Kopiranje podatkov dnevnika

Za kopiranje dnevnika za odpravljanje težav ali podporo:

1. Odprite okno Debug Log
2. Kliknite gumb **&quot;Copy Log&quot;** (ali desni klik → Izberi vse)
3. Prilepite v besedilno datoteko ali e-pošto
4. Po potrebi pošljite podpori MAPIR

***

## Nadzor sistemskih virov

### Poraba CPU

**Prosti način:**

* 1 jedro CPU pri ~100 %
* Druga jedra so v mirovanju ali na voljo
* Sistem ostaja odziven

**Chloros+ vzporedni način:**

* Več jedr pri 80–100 % (do 16 jedr)
* Visoka splošna izkoriščenost CPU
* Sistem se lahko zdi manj odziven

**Za spremljanje:**

* Windows Upravitelj opravil (Ctrl+Shift+Esc)
* Zavihek Zmogljivost → razdelek CPU
* Poiščite procese »Chloros« ali »chloros-backend«

### Poraba pomnilnika (RAM)

**Tipična poraba:**

* Majhni projekti (&lt; 100 slik): 2–4 GB
* Srednji projekti (100–500 slik): 4–8 GB
* Veliki projekti (500+ slik): 8–16 GB
* Vzporedni način Chloros+ porabi več RAM-a

**Če je pomnilnika malo:**

* Obdelujte manjše serije
* Zaprite druge aplikacije
* Nadgradite RAM, če redno obdelujete velike podatkovne nize

### Poraba GPU (Chloros+ s CUDA)

Ko je pospeševanje GPU omogočeno:

* GPU NVIDIA kaže visoko izkoriščenost (60–90 %)
* Poraba VRAM se poveča (zahteva 4 GB+ VRAM)
* Faza kalibriranja je znatno hitrejša

**Za spremljanje:**

* Ikona NVIDIA v sistemski vrstici
* Upravitelj opravil → Zmogljivost → GPU
* GPU-Z ali podobno orodje za spremljanje

### Diskovni vhod/izhod

**Kaj lahko pričakujete:**

* Visoka hitrost branja diska med fazo analiziranja
* Visoka hitrost pisanja diska med fazo izvoza
* SSD je bistveno hitrejši od HDD

**Nasvet za zmogljivost:**

* Po možnosti uporabite SSD za mapo projekta
* Izogibajte se omrežnim pogonom za velike podatkovne nize
* Preverite, da disk ni skoraj poln (vpliva na hitrost pisanja)

***

## Odkrivanje težav med obdelavo

### Opozorilni znaki

**Napredek se ustavi (brez sprememb več kot 5 minut):**

* Preverite dnevnik odpravljanja napak za napake
* Preverite razpoložljiv prostor na disku
* Preverite upravitelja opravil, da se prepričate, da Chloros teče

**Napake se pojavljajo pogosto:**

* Ustavite obdelavo in preglejte napake
* Pogosti vzroki: prostor na disku, poškodovane datoteke, težave s pomnilnikom
* Glejte poglavje Odpravljanje težav spodaj

**Sistem ne odziva:**

* Chloros+ vzporedni način porablja preveč virov
* Razmislite o zmanjšanju števila sočasnih nalog ali nadgradnji strojne opreme
* Prosti način porablja manj virov

### Kdaj ustaviti obdelavo

Ustavite obdelavo, če opazite:

* ❌ Napake »Disk poln« ali »Datoteke ni mogoče zapisati«
* ❌ Ponavljajoče se napake pri poškodovanih slikovnih datotekah
* ❌ Sistem se je popolnoma zamrznil (ne odziva se)
* ❌ Ugotovljeno je bilo, da so bile nastavljene napačne nastavitve
* ❌ Uvožene so bile napačne slike

**Kako ustaviti:**

1. Kliknite**gumb Stop/Prekliči** (nadomesti gumb Začni)
2. Obdelava se ustavi, napredek se izgubi
3. Odpravite težave in začnite znova od začetka

***

## Odpravljanje težav med obdelavo

### Obdelava poteka zelo počasi

**Možni vzroki:**

* Neoznačene ciljne slike (skeniranje vseh slik)
* Shranjevanje na HDD namesto na SSD
* Premajhni sistemski viri
* Konfiguriranih je veliko indeksov
* Dostop do omrežnega diska

**Rešitve:**

1. Če ste ravno začeli in ste v fazi zaznavanja: Prekličite, označite cilje, ponovno zaženite
2. Za prihodnost: Uporabite SSD, zmanjšajte število indeksov, nadgradite strojno opremo
3. Razmislite o uporabi CLI za obdelavo velikih podatkovnih nizov v paketih

### Opozorila o »prostoru na disku«

**Rešitve:**

1. Takoj sprostite prostor na disku
2. Premestite projekt na pogon z več prostora
3. Zmanjšajte število indeksov za izvoz
4. Uporabite format JPG namesto TIFF (manjše datoteke)

### Pogosta sporočila o »poškodovanih datotekah«

**Rešitve:**

1. Ponovno kopirajte slike s kartice SD, da zagotovite njihovo celovitost
2. Preverite kartico SD za napake
3. Odstranite poškodovane datoteke iz projekta
4. Nadaljujte z obdelavo preostalih slik

### Pregrevanje sistema / omejevanje zmogljivosti

**Rešitve:**

1. Zagotovite ustrezno prezračevanje
2. Očistite prah iz prezračevalnih odprtin računalnika
3. Zmanjšajte obremenitev obdelave (uporabite način Free namesto Chloros+)
4. Obdelujte v hladnejših delih dneva

***

## Obvestilo o zaključku obdelave

Ko se obdelava konča:

* Napredovalna vrstica doseže 100 %
* V dnevniku odpravljanja napak se prikaže sporočilo **»Obdelava zaključena«**
* Gumb »Začni« postane ponovno aktiven
* Vse izhodne datoteke so v podmapah modelov fotoaparatov

***

## Naslednji koraki

Ko se obdelava zaključi:

1. **Preglejte rezultate** – glejte [Zaključek obdelave](finishing-the-processing.md)
2. **Preverite izhodno mapo** – Preverite, ali so bile vse datoteke pravilno izvožene
3. **Preglejte dnevnik odpravljanja napak** – Preverite, ali so prisotna kakšna opozorila ali napake
4. **Predogled obdelanih slik** – Uporabite pregledovalnik slik ali zunanjo programsko opremo

Za informacije o pregledu in uporabi obdelanih rezultatov glejte [Zaključek obdelave](finishing-the-processing.md).

# Spremljanje obdelave

Ko se obdelava začne, Chloros ponuja več načinov za spremljanje napredka, preverjanje težav in razumevanje, kaj se dogaja z vašim nizom podatkov. Na tej strani je pojasnjeno, kako spremljati obdelavo in razlagati informacije, ki jih ponuja Chloros.

## Pregled napredovalne vrstice

Napredek v zgornjem naslovu prikazuje stanje obdelave v realnem času in odstotek dokončanja.

### Napredek v prostem načinu

Za uporabnike brez licence Chloros+:

**Prikaz napredka v dveh fazah:**

1. **Zaznavanje cilja** – iskanje kalibracijskih ciljev v slikah
2. **Obdelava** – uporaba popravkov in izvoz

**Napredek prikazuje:**

* Skupni odstotek dokončanja (0–100 %)
* Ime trenutne stopnje
* Preprosta vizualizacija vodoravne vrstice

### Napredek Chloros

Za uporabnike z licenco Chloros+:

**4-stopenjski prikaz napredka:**

1. **Zaznavanje** – iskanje kalibracijskih ciljev
2. **Analiziranje** – pregledovanje slik in priprava poteka
3. **Kalibriranje** – uporaba popravkov vinjete in odbojnosti
4. **Izvoz** – shranjevanje obdelanih datotek

**Interaktivne funkcije:**

* **Premaknite miško** nad kazalnik napredka, da se prikaže razširjen 4-stopenjski panel
* **Kliknite** na vrstico napredka, da zamrznete/pritrdite razširjeni panel
* **Kliknite ponovno**, da odmrznete in samodejno skrijete, ko umaknete miško
* Vsaka stopnja prikazuje posamezni napredek (0–100 %)

***

## Razumevanje vsake stopnje obdelave

### Stopnja 1: Zaznavanje (zaznavanje ciljev)

**Kaj se dogaja:**

* Chloros skenira slike, označene s potrditvenim poljem Cilj
* Algoritmi računalniškega vida identificirajo 4 kalibracijske plošče
* Vrednosti odbojnosti, izvlečene iz vsake plošče
* Časovni žigi ciljev, zabeleženi za pravilno načrtovanje kalibracije

**Trajanje:**

* Z označenimi cilji: 10–60 sekund
* Brez označenih ciljev: 5–30+ minut (skenira vse slike)

**Kazalnik napredka:**

* Odkrivanje: 0 % → 100 %
* Število skeniranih slik
* Število najdenih ciljev

**Na kaj je treba paziti:**

* Če so cilji pravilno označeni, naj bi se postopek hitro zaključil.
* Če traja predolgo, cilji morda niso označeni.
* V dnevniku odpravljanja napak preverite, ali so prisotna sporočila »Target found« (Cilj najden).

### Faza 2: Analiza

**Kaj se dogaja:**

* Branje metapodatkov EXIF slike (časovni žigi, nastavitve osvetlitve)
* Določanje strategije kalibracije na podlagi časovnih žigov ciljev
* Urejanje čakalne vrste za obdelavo slik
* Priprava delavcev za vzporedno obdelavo (samo Chloros+)

**Trajanje:** 5–30 sekund

**Kazalnik napredka:**

* Analiza: 0 % → 100 %
* Hitra faza, običajno se hitro zaključi

**Na kaj je treba paziti:**

* Napredek mora potekati enakomerno brez premorov
* Opozorila o manjkajočih metapodatkih se prikažejo v dnevniku odpravljanja napak

### Faza 3: Kalibriranje

**Kaj se dogaja:**

* **Debayering**: pretvorba RAW Bayerjevega vzorca v 3 kanale
* **Popravek vinjete**: odstranjevanje potemnjenja robov objektiva
* **Kalibracija odbojnosti**: normalizacija s ciljnimi vrednostmi
* **Izračun indeksa**: izračun multispektralnih indeksov
* Obdelava vsake slike skozi celoten proces

**Trajanje:** večina celotnega časa obdelave (60–80 %)

**Kazalnik napredka:**

* Kalibriranje: 0 % → 100 %
* Trenutna slika se obdeluje
* Slike dokončane / Skupno število slik

**Obnašanje obdelave:**

* **Prosti način**: Obdeluje eno sliko naenkrat zaporedno
* **Način Chloros+**: Obdeluje do 16 slik hkrati
* **Pospeševanje GPU**: znatno pospeši to fazo

**Na kaj je treba paziti:**

* Stalen napredek skozi število slik
* Preverite dnevnik odpravljanja napak za sporočila o dokončanju posamezne slike
* Opozorila o kakovosti slike ali težavah s kalibracijo

### Faza 4: Izvoz

**Kaj se dogaja:**

* Zapis kalibriranih slik na disk v izbranem formatu
* Izvoz multispektralnih indeksnih slik z barvami LUT
* Ustvarjanje podmap za modele kamer
* Ohranjanje izvirnih imen datotek z ustreznimi končnicami

**Trajanje:** 10–20 % skupnega časa obdelave

**Kazalnik napredka:**

* Izvoz: 0 % → 100 %
* Zapisovanje datotek
* Format izvoza in ciljna lokacija

**Na kaj je treba paziti:**

* Opozorila o prostem prostoru na disku
* Napake pri zapisovanju datotek
* Zaključek vseh nastavljenih izhodov

***

## Zavihek Debug Log (Dnevnik odpravljanja napak)

Dnevnik odpravljanja napak vsebuje podrobne informacije o napredku obdelave in morebitnih težavah.

### Dostop do dnevnika odpravljanja napak

1. Kliknite ikono **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> v levem stranskem meniju.
2. Odpre se okno dnevnika, ki prikazuje sporočila o obdelavi v realnem času.
3. Samodejno se pomika navzdol, da prikaže najnovejša sporočila.

### Razumevanje sporočil dnevnika

#### Informacijska sporočila (bela/siva)

Običajne posodobitve obdelave:

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

**Ukrep:** Po obdelavi preglejte opozorila, vendar ne prekinjajte obdelave.

#### Napake (Red)

Kritične težave, ki lahko povzročijo napako pri obdelavi:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Ukrep:** Prekinite obdelavo, odpravite napako in ponovno zaženite.

### Pogosta sporočila dnevnika

| Sporočilo                          | Pomen                                | Potrebni ukrep                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| „Cilj zaznan v \[ime datoteke]” | Kalibracijski cilj uspešno najden  | Nič – normalno                                         |
| „Obdelava slike X od Y”        | Trenutna posodobitev napredka                | Nič – normalno                                         |
| „Cilji niso bili najdeni”               | Kalibracijski cilji niso bili zaznanih        | Označite ciljne slike ali onemogočite kalibracijo odbojnosti |
| „Premalo prostora na disku“        | Premalo prostora za shranjevanje izhodnih podatkov          | Osvobodite prostor na disku                                    |
| „Preskočitev poškodovane datoteke“        | Datoteka slike je poškodovana                  | Ponovno kopirajte datoteko s kartice SD                             |
| „Uporabljeni podatki PPK“               | Uporabljene popravke GPS iz datoteke .daq | Nič – normalno                                         |

### Kopiranje podatkov dnevnika

Kopiranje dnevnika za odpravljanje težav ali podporo:

1. Odprite okno Debug Log (Dnevnik odpravljanja napak).
2. Kliknite gumb **„Copy Log“** (Kopiraj dnevnik) (ali desni klik → Izberi vse).
3. Prilepite v besedilno datoteko ali e-pošto.
4. Po potrebi pošljite podpori MAPIR.

***

## Nadzor sistemskih virov

### Poraba CPU

**Prosti način:**

* 1 jedro CPU pri ~100 %
* Druga jedra so v mirovanju ali na voljo
* Sistem ostane odziven

**Chloros+ vzporedni način:**

* Več jeder pri 80–100 % (do 16 jeder)
* Visoka splošna poraba CPU
* Sistem je lahko manj odziven

**Za spremljanje:**

* Windows Task Manager (Ctrl+Shift+Esc)
* Zavihek Performance → razdelek CPU
* Poiščite procese »Chloros« ali »chloros-backend«

### Poraba pomnilnika (RAM)

**Tipična uporaba:**

* Majhni projekti (&lt; 100 slik): 2–4 GB
* Srednji projekti (100–500 slik): 4–8 GB
* Veliki projekti (500+ slik): 8–16 GB
* Chloros+ vzporedni način uporablja več RAM-a

**Če je pomnilnik poln:**

* Obdelujte manjše serije
* Zaprite druge aplikacije
* Nadgradite RAM, če redno obdelujete velike podatkovne nize

### Poraba GPU (Chloros+ s CUDA)

Ko je omogočena pospešitev GPU:

* NVIDIA GPU kaže visoko izkoriščenost (60–90 %)
* Poraba VRAM se poveča (zahteva 4 GB+ VRAM)
* Faza kalibriranja je znatno hitrejša

**Za spremljanje:**

* Ikona NVIDIA v sistemski vrstici
* Upravitelj opravil → Zmogljivost → GPU
* GPU-Z ali podobno orodje za spremljanje

### Disk I/O

**Kaj lahko pričakujete:**

* Visoka hitrost branja diska med fazo analiziranja
* Visoka hitrost pisanja diska med fazo izvoza
* SSD je znatno hitrejši od HDD

**Nasvet za zmogljivost:**

* Po možnosti uporabite SSD za projektno mapo
* Izogibajte se omrežnim pogonom za velike podatkovne nize
* Preverite, da disk ni skoraj poln (vpliva na hitrost pisanja)

***

## Odkrivanje težav med obdelavo

### Opozorilni znaki

**Napredek se ustavi (brez sprememb več kot 5 minut):**

* Preverite dnevnik odpravljanja napak za napake
* Preverite razpoložljivi prostor na disku
* Preverite upravitelja opravil, da se prepričate, da Chloros deluje

**Pogosto se pojavljajo sporočila o napakah:**

* Ustavi obdelavo in preglej napake
* Pogosti vzroki: prostor na disku, poškodovane datoteke, težave s pomnilnikom
* Glejte poglavje Odpravljanje težav spodaj

**Sistem ne odgovarja:**

* Chloros+ vzporedni način porablja preveč virov
* Razmislite o zmanjšanju sočasnih opravil ali nadgradnji strojne opreme
* Prosti način porablja manj virov

### Kdaj ustaviti obdelavo

Ustavite obdelavo, če opazite:

* ❌ Napake »Disk poln« ali »Datoteke ni mogoče zapisati«
* ❌ Ponavljajoče se napake poškodovanih slikovnih datotek
* ❌ Sistem je popolnoma zamrznil (ne odziva se)
* ❌ Ugotovljeno je bilo, da so bile nastavljene napačne nastavitve
* ❌ Uvožene so bile napačne slike

**Kako ustaviti:**

1. Kliknite **gumb Stop/Cancel** (nadomesti gumb Start)
2. Obdelava se ustavi, napredek se izgubi
3. Popravite težave in začnite znova od začetka

***

## Odpravljanje težav med obdelavo

### Obdelava je zelo počasna

**Možni vzroki:**

* Neoznačene ciljne slike (skeniranje vseh slik)
* HDD namesto SSD pomnilnika
* Nezadostni sistemski viri
* Konfiguriranih je veliko indeksov
* Dostop do omrežnega diska

**Rešitve:**

1. Če ste ravno začeli in ste v fazi zaznavanja: Prekličite, označite cilje, ponovno zaženite
2. Za prihodnost: Uporabite SSD, zmanjšajte indekse, nadgradite strojno opremo
3. Razmislite o uporabi CLI za obdelavo velikih podatkovnih nizov v serijah

### Opozorila o prostem prostoru na disku

**Rešitve:**

1. Takoj sprostite prostor na disku
2. Premestite projekt na pogon z več prostora
3. Zmanjšajte število indeksov za izvoz.
4. Uporabite format JPG namesto TIFF (manjše datoteke).

### Pogosta sporočila »Površinska datoteka«

**Rešitve:**

1. Ponovno kopirajte slike s kartice SD, da zagotovite njihovo celovitost.
2. Preverite kartico SD za napake.
3. Odstranite površinske datoteke iz projekta.
4. Nadaljujte z obdelavo preostalih slik.

### Pregrevanje/upočasnjevanje sistema

**Rešitve:**

1. Zagotovite ustrezno prezračevanje.
2. Očistite prah iz prezračevalnih odprtin računalnika.
3. Zmanjšajte obremenitev obdelave (uporabite prosti način namesto Chloros+).
4. Obdelujte v hladnejših delih dneva.

***

## Obvestilo o končani obdelavi

Ko se obdelava konča:

* Napredek doseže 100 %
* V dnevniku odpravljanja napak se prikaže sporočilo **»Obdelava zaključena«**
* Gumb za zagon se ponovno vklopi
* Vse izhodne datoteke so v podmapah modelov kamer

***

## Naslednji koraki

Ko je obdelava zaključena:

1. **Preglejte rezultate** – glejte [Zaključek obdelave](finishing-the-processing.md)
2. **Preverite izhodno mapo** – preverite, ali so vse datoteke pravilno izvožene
3. **Preglejte dnevnik odpravljanja napak** – preverite, ali so prisotna opozorila ali napake
4. **Preglejte obdelane slike** – uporabite pregledovalnik slik ali zunanjo programsko opremo

Za informacije o pregledovanju in uporabi obdelanih rezultatov glejte [Zaključek obdelave](finishing-the-processing.md).

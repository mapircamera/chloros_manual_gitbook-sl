# Spremljanje obdelave

Ko se obdelava začne, Chloros ponuja več načinov za spremljanje napredka, preverjanje morebitnih težav in razumevanje, kaj se dogaja z vašim naborom podatkov. Na tej strani je pojasnjeno, kako spremljati obdelavo in razlagati informacije, ki jih zagotavlja Chloros.

## Pregled kazalnika napredka

Napredovalna vrstica v zgornjem naslovnem pasu prikazuje stanje obdelave v realnem času in odstotek dokončanja. Napredek se prenaša v živo iz strežniškega ozadja prek Server-Sent Events (SSE), zato vrstica odraža, kaj obdelovalni tok dejansko počne.

### Napredovalna vrstica v brezplačnem načinu

Za uporabnike brez licence Chloros+:

**Dvostopenjski prikaz napredka:**

1.**Zaznavanje ciljev** – Iskanje kalibracijskih ciljev na slikah
2. **Obdelava** – Uporaba popravkov in izvoz**Indikator napredka prikazuje:**

* Skupni odstotek dokončanja (0–100 %)
* Ime trenutne stopnje
* Preprosta vizualizacija v obliki vodoravne črte

### Kazalnik napredka Chloros+

Za uporabnike z licenco Chloros+:

**Prikaz napredka v 4 stopnjah:**

1.**Zaznavanje** – iskanje kalibracijskih ciljev
2. **Analiza** – pregledovanje slik in priprava procesnega toka
3. **Kalibracija** – Uporaba popravkov vinjete in odbojnosti
4. **Izvoz** – Shranjevanje obdelanih datotek**Interaktivne funkcije:*** **Preletite z miško** nad kazalnikom napredka, da se prikaže razširjen 4-stopenjski panel
* **Kliknite** na kazalnik napredka, da zamrznete/pritrdite razširjeni panel
* **Kliknite še enkrat**, da razmrzite in se panel samodejno skrije ob umiku miške
* Vsaka faza prikazuje posamezen napredek (0–100 %)

{% hint style="info" %}
**Pariteta CLI**: med izvajanjem `chloros-cli process` se v vseh štirih niti prikazujejo stanja »Zaznavanje«, »Analiziranje«, »Obdelava« in »Izvoz«, `chloros-cli export-status` pa v realnem času prikazuje napredek izvoza v niti 4 z drugega terminala. Glejte [Referenco CLI](../reference/cli-reference.md).
{% endhint %}

***

## Razumevanje posameznih faz obdelave

{% hint style="info" %}
**Arhitektura poteka obdelave**: Te 4 faze v grafičnem vmesniku ustrezajo [poteku obdelave s 4 niti](../processing-architecture/processing-pipeline.md). Na sistemih s pospešitvijo prek grafičnega procesorja (GPU) nit 3 (kalibracija) izkorišča [dinamično prilagajanje računalniške moči](../processing-architecture/dynamic-compute-adaptation.md), ki optimizira obdelavo za vašo specifično strojno opremo.
{% endhint %}

### Faza 1: Zaznavanje (zaznavanje ciljev)

**Kaj se dogaja:**

* Chloros pregleda slike, ki ste jih označili s potrditvenim poljem »Cilj« (vse slike, če ni označena nobena)
* Algoritmi računalniškega vida prepoznajo kalibracijske plošče
* Iz vsake plošče se izluščijo vrednosti odbojnosti
* Zabeležijo se časovni žigi ciljev za pravilno načrtovanje kalibracije

**Trajanje:**

* Z označenimi cilji: 10–60 sekund
* Brez označenih ciljev: 5–30+ minut (skenira vse slike)

**Kazalnik napredka:**

* Prepoznavanje: 0 % → 100 %
* Število pregledanih slik (šteje se le slike, ki se dejansko pregledujejo)
* Število najdenih ciljev

**Na kaj je treba paziti:**

* Postopek naj bi se hitro zaključil, če so cilji pravilno označeni
* Če traja predolgo, cilji morda niso označeni
* V dnevniku odpravljanja napak preverite sporočila »Cilj najden«

### Faza 2: Analiza

**Kaj se dogaja:**

* Branje metapodatkov EXIF slik (časovni žigi, nastavitve osvetlitve)
* Določanje strategije kalibracije na podlagi časovnih žigov ciljev in razpoložljivih podatkov DAQ o navzdolnem sevanju
* Urejanje čakalne vrste za obdelavo slik
* Priprava delavcev za vzporedno obdelavo (samo Chloros+)

**Trajanje:** 5–30 sekund**Kazalnik napredka:**

* Analiza: 0 % → 100 %
* Hitra faza, običajno se hitro zaključi

**Na kaj je treba paziti:**

* Napredek mora potekati enakomerno brez premorov
* Opozorila o manjkajočih metapodatkih se bodo pojavila v dnevniku odpravljanja napak (Debug Log)

### Faza 3: Kalibracija

**Kaj se dogaja:*** **Debayering**: Pretvorba RAW-vzorca Bayer v 3 kanale (preskoči se za mono module LATTICE, z opombo)
* **Korekcija vinjetiranja**: Odstranjevanje potemnjenja na robovih objektiva
* **Kalibracija odbojnosti**: Normalizacija s ciljnimi vrednostmi in/ali DAQ-ovim zmanjšanjem
* **Izračun indeksov**: Izračun multispektralnih indeksov
* Obdelava vsake slike skozi celoten proces

**Trajanje:** Večina skupnega časa obdelave (60–80 %)**Kazalnik napredka:**

* Kalibriranje: 0 % → 100 %
* Trenutna slika v obdelavi
* Obdelane slike / Skupno število slik

**Način obdelave:*** **Prosti način**: Zaporedno obdeluje po eno sliko naenkrat
* **Način Chloros+**: Upravlja s skupino delavcev, ki se prilagaja strojni opremi – 1–4 sočasnih delavcev na sistemih z grafičnimi procesorji (glede na VRAM), en delavec na fizično jedro (minus eno) na sistemih, ki uporabljajo samo procesorje. Glej [Dinamično prilagajanje računalniške moči](../processing-architecture/dynamic-compute-adaptation.md)
* **Pospešitev z GPU**: znatno pospeši to fazo**Na kaj je treba paziti:**

* Enakomeren napredek pri številu slik
* Preverite dnevnik odpravljanja napak za sporočila o zaključku posameznih slik
* Opozorila o težavah s kakovostjo slik ali kalibracijo

### Faza 4: Izvoz

**Kaj se dogaja:**

* Zapišanje obdelanih slik na disk v izbranem formatu, takoj ko so dokončane
* **LATTICE**: vsak okvir se razširi v vsak omogočen izdelek (debayering / predogled / sevanje / odbojnost)
* Izvoz multispektralnih indeksnih slik z barvami LUT
* Ustvarjanje izhodnega drevesa `<project>/<camera>/<format>/<Product>_Images/` — izvožene datoteke ohranijo ime izvorne datoteke; mapa označuje produkt

**Trajanje:** 10–20 % skupnega časa obdelave**Kazalnik napredka:**

* Izvoz: 0 % → 100 %
* Pisanje datotek
* Izvozni format in ciljna mapa

**Na kaj je treba paziti:**

* Opozorila o prostem prostoru na disku
* Napake pri pisanju datotek
* Zaključek vseh nastavljenih izhodov

***

## Zavihek »Dnevnik odpravljanja napak«

Dnevnik odpravljanja napak ponuja podrobne informacije o napredku obdelave in morebitnih težavah. V dnevniško konzolo se prenašajo tudi sporočila o zagonu strežniškega dela, zato dnevnik prikazuje celotno zgodbo, tudi če ga odprete pozneje.

### Dostop do dnevnika odpravljanja napak

1. V levem stranskem meniju kliknite ikono **Dnevnik odpravljanja napak** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">


2. Odpre se okno dnevnika, ki prikazuje sporočila o obdelavi v realnem času
3. Vsebina se samodejno pomika navzdol, da se prikažejo najnovejša sporočila

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written), with the 1.3.0 bug icon (Report a bug) visible at the top of the tab -->

### Razumevanje sporočil v dnevniku V vrsticah dnevnika

Chloros a so na začetku oznake v oklepajih, ki poimenujejo podsistem — na primer `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Najpomembnejše, kar morate vedeti, je **povzetek izvajanja**, ki se izpiše na koncu vsakega izvajanja (vključno z ustavljenimi izvajanji):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Dodatne vrstice z namigi `[RUN-SUMMARY]` sledijo vedno, kadar je potrebno kaj pojasniti — na primer izvedba, ki ni prinesla nobenih rezultatov, ali kamero, katere zahtevani izdelek je bil preskočen kot neuporaben. Vrstice `[EXPORT-CHECK]` pojasnjujejo preskoke po posameznih kamerah (npr. zakaj RGB-kamera ni dobila izdelka sevanja).

Splošne stopnje resnosti sporočil (spodnji primeri so ilustrativni, ne dobesedni):

#### Informacijska sporočila (bela/siva)

Običajne posodobitve obdelave: začetek obdelave, zaznane tarče (s številom panelov), napredek kalibracije po posameznih slikah, izvožene datoteke, obdelava zaključena.

#### Opozorilna sporočila (rumena)

Nekritične težave, ki ne ustavijo obdelave — npr. manjkajoči podatki GPS v posnetku, velika vrzel v časovnih oznakah med ciljnimi slikami ali nizek kontrast v kalibracijskem panelu.

**Ukrep:** Po obdelavi preglejte opozorila, vendar ne prekinjajte obdelave

#### Sporočila o napakah (v Red

barvi)

Kritične težave, ki lahko povzročijo neuspešno obdelavo – npr. poln disk, poškodovana slikovna datoteka ali nezaznani cilji, čeprav je bila zahtevana kalibracija odbojnosti.

**Ukrep:** Prekinite obdelavo, odpravite napako, ponovno zaženite

### Pogoste situacije v dnevniku

| Situacija                             | Pomen                                       | Potrebni ukrepi                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Cilj zaznan v \[ime datoteke]        | Kalibracijski cilj uspešno najden         | Noben – normalno                                         |
| Črte napredka po posameznih slikah              | Trenutna posodobitev napredka                       | Noben – normalno                                         |
| Cilji niso bili najdeni                      | Kalibracijski cilji niso bili zaznani               | Označite ciljne slike ali onemogočite kalibracijo odbojnosti |
| Premajhen prostor na disku               | Premalo prostora za shranjevanje izhodnih podatkov                 | Osvobodite prostor na disku                                    |
| Preskočitev poškodovane datoteke               | Datoteka slike je poškodovana                         | Ponovno kopirajte datoteko s kartice SD                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | Posnetka brez surovega okvirja ni mogoče obdelati | Ponovno posnemite s surovim okvirjem, ali uporabite CLI `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | Izvedba ni ustvarila nobenih slikovnih izdelkov — poročano kot napaka z namigi | Preberite vrstice z namigi; preverite, kaj je bilo preskočeno in zakaj |
| `[PROCESS] refused: The project folder is too deep for this computer…` | Dolge poti v Windows u so izklopljene, projektna mapa pa je pregloboka za omejitev poti na 260 znakov; nič ni bilo obdelano | Uporabite krajšo projektno mapo (sporočilo navaja najdaljšo, ki ustreza), ali vklopite dolge poti v sistemu Windows in ponovno zaženite program Chloros — glejte [Začetek obdelave](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer) |
| `[unmix-clamp-mask] …` | Maske omejitve izdelka LATTICE ni bilo mogoče zapisati; sam izdelek je v redu in prikazuje `Chloros:UnmixClampMask=unavailable` | Preverite prostora na disku in dovoljenja za mape, nato pa ponovno obdelajte, če potrebujete masko |

### Kopiranje podatkov dnevnika

Za kopiranje dnevnika za odpravljanje težav ali podporo:

1. Odprite okno »Debug Log«
2. Kliknite gumb **Kopiraj**(dnevnik se skopira v odložišče) ali**Prenesi**, da ga shranite kot besedilno datoteko
3. Vstavite v besedilno datoteko ali e-pošto
4. Po potrebi ga pošljite podpori MAPIR



### Prijava napake iz zavihka »Dnevnik odpravljanja napak«

Novost v različici 1.3.0: ikona napake (**Prijavi napako**) na vrhu zavihka »Dnevnik odpravljanja napak« pošlje poročilo neposredno podjetju MAPIR.

1. Kliknite ikono napake na vrhu zavihka »Dnevnik odpravljanja napak«
2. Opišite težavo: kaj ste počeli, kaj ste pričakovali in kaj se je namesto tega zgodilo (obvezno)
3. Po želji označite **Dodaj nekaj slik (do 10)**in izberite slike iz odprtega projekta ali uporabite**Prebrskaj...** za katero koli drugo slikovno datoteko
4. Kliknite **Pošlji**

Poročilo vključuje vaš opis, vse priložene slike, dnevnike Chloros ter podrobnosti o sistemu (različica Chloros, operacijski sistem, grafična kartica in nastavitve projekta). Za pošiljanje poročila morate biti prijavljeni v Chloros (zavihek »User«). Med pošiljanjem lahko okno zaprete; ko se pošiljanje konča, se v dnevnik zapiše sklic, na primer `Bug report sent. Reference: BR-…` – navedite ga, ko se obrnete na MAPIR. Če poročila ni mogoče naložiti (na primer zaradi pomanjkanja internetne povezave), ga Chloros shrani na vaš računalnik in prikaže pot do datoteke, da ga lahko pošljete po e-pošti na naslov support@mapir.camera.

***

## Spremljanje sistemskih virov

### Poraba procesorja

**Prosti način:**

* 1 jedro procesorja pri ~100 %
* Druga jedra so v mirovanju ali na voljo
* Sistem ostaja odziven

**Vzporedni način Chloros+:**

* Več jeder z visoko izkoriščenostjo – število je odvisno od strategije, izbrane v [Dinamični prilagoditvi računalniške moči](../processing-architecture/dynamic-compute-adaptation.md)
* Sistem se lahko zdi manj odziven

**Za spremljanje:**

* Upravitelj opravil Windows a (Ctrl+Shift+Esc)
* Zavihek »Zmogljivost« → razdelek »Procesor«
* Poiščite procese »Chloros « ali »chloros-backend«

### Poraba pomnilnika (RAM)

**Tipična poraba:**

* Majhni projekti (< 100 slik): 2–4 GB
* Srednji projekti (100–500 slik): 4–8 GB
* Veliki projekti (500+ slik): 8–16 GB
* Vzporedni način delovanja »Chloros+« porabi več pomnilnika RAM

**Če je pomnilnika premalo:**

* Obdelujte manjše serije
* Zaprite druge aplikacije
* Nadgradite RAM, če redno obdelujete velike nize podatkov

### Poraba GPU (»Chloros+« s CUDA)

Ko je pospeševanje s GPU omogočeno:

* GPU NVIDIA kaže visoko izkoriščenost (60–90 %)
* Poraba VRAM se poveča (zahteva 4 GB+ VRAM; 7 GB+ za sočasno odstranjevanje pikslov z upoštevanjem tekstur)
* Faza kalibracije je znatno hitrejša

**Za spremljanje:**

* Ikona NVIDIA v sistemski vrstici
* Upravitelj opravil → Zmogljivost → GPU
* GPU-Z ali podobno orodje za spremljanje

### Vnos/iznos podatkov na disku

**Kaj lahko pričakujete:**

* Visoka hitrost branja z diska med fazo analiziranja
* Visoka hitrost zapisovanja na disk med fazo izvoza
* SSD je znatno hitrejši od HDD

**Nasvet za izboljšanje zmogljivosti:**

* Po možnosti uporabite SSD za projektno mapo
* Pri velikih naborih podatkov se izogibajte omrežnim pogonom
* Poskrbite, da disk ni skoraj poln (vpliva na hitrost pisanja)

***

## Odkrivanje težav med obdelavo

### Opozorilni znaki

**Napredek se ustavi (brez sprememb več kot 5 minut):**

* Preverite dnevnik odpravljanja napak za morebitne napake
* Preverite razpoložljiv prostor na disku
* V upravitelju opravil preverite, ali Chloros deluje

**Pogosto se pojavljajo sporočila o napakah:**

* Prekinite obdelavo in preglejte napake
* Pogosti vzroki: prostor na disku, poškodovane datoteke, težave s pomnilnikom
* Glejte poglavje »Odpravljanje težav« spodaj

**Sistem ne odziva:**

* Vzporedni način delovanja programa Chloros porablja preveč virov
* Razmislite o zmanjšanju števila sočasnih nalog ali nadgradnji strojne opreme
* Prosti način porablja manj virov

### Kdaj ustaviti obdelavo

Ustavite obdelavo, če opazite:

* ❌ Napake »Disk poln« ali »Datoteke ni mogoče zapisati«
* ❌ Ponavljajoče se napake poškodovanih slikovnih datotek
* ❌ Sistem se je popolnoma zamrznil (ne odziva se)
* ❌ Ugotovili ste, da so bile nastavljene napačne nastavitve
* ❌ Uvožene so bile napačne slike

**Kako ustaviti:**

1. Kliknite**gumb »Stop«** (nadomešča gumb »Start«) – enkrat je dovolj
2. Na vrstici se prikaže »Ustavljanje...«, medtem ko se zaključi obdelava slike, ki je v teku, nato pa se izvajanje konča v ustavljenem stanju
3. Že izvoženi izdelki ostanejo na disku; dnevnik natisne natančen zapis `[RUN-SUMMARY]` o tem, kaj je bilo zaključeno
4. Odpravite težave in ponovno zaženite — izvajanje se začne od začetka

***

## Odpravljanje težav med obdelavo

### Obdelava poteka zelo počasi

**Možni vzroki:**

* Neoznačene ciljne slike (skeniranje vseh slik)
* Shranjevanje na HDD namesto na SSD
* Nezadostni sistemski viri
* Konfiguriranih je veliko indeksov
* Dostop do omrežnega diska

**Rešitve:**

1. Če se je obdelava šele začela in je v fazi zaznavanja: ustavite, označite ciljne slike, ponovno zaženite
2. Za prihodnost: uporabite SSD, zmanjšajte število indeksov, nadgradite strojno opremo
3. Razmislite o uporabi CLI-ja za obdelavo velikih naborov podatkov v paketih

### Opozorila o »prostoru na disku«

**Rešitve:**

1. Takoj sprostite prostor na disku
2. Premestite projekt na pogon z več prostora
3. Zmanjšajte število indeksov za izvoz
4. Onemogočite izvozne produkte LATTICE, ki jih ne potrebujete (Nastavitve projekta → Obdelava)
5. Uporabite format JPG namesto TIFF a (manjše datoteke)

### Pogosta sporočila o »poškodovanih datotekah«

**Rešitve:**

1. Ponovno kopirajte slike s kartice SD, da zagotovite njihovo celovitost
2. Preverite kartico SD za napake
3. Odstranite poškodovane datoteke iz projekta
4. Nadaljujte z obdelavo preostalih slik

### Pregrevanje sistema / omejevanje zmogljivosti

**Rešitve:**

1. Poskrbite za ustrezno prezračevanje
2. Očistite prah iz prezračevalnih odprtin računalnika
3. Zmanjšajte obremenitev obdelave (uporabite način »Free« namesto »Chloros+«)
4. Obdelujte v hladnejših delih dneva

***

## Obvestilo o zaključku obdelave

Ko se obdelava konča:

* Napredovalna vrstica doseže 100 %
* V dnevniku odpravljanja napak se prikažejo vrstice `[RUN-SUMMARY]` s končnimi številkami
* Gumb »Začni« se ponovno omogoči
* Vse izhodne datoteke so v izhodni mapi projekta za posamezno kamero: `<project>/<camera>/<format>/<Product>_Images/`

***

## Naslednji koraki

Ko se obdelava zaključi:

1. **Preglejte rezultate** – glejte [Zaključek obdelave](finishing-the-processing.md)
2. **Preverite izhodno mapo** – preverite, ali so bile vse datoteke pravilno izvožene
3. **Preglejte dnevnik odpravljanja napak** – Preverite, ali so prisotna kakšna opozorila ali napake
4. **Oglejte si predogled obdelanih slik** – Uporabite pregledovalnik slik ali zunanjo programsko opremo

Za informacije o pregledovanju in uporabi obdelanih rezultatov glejte [Zaključek obdelave](finishing-the-processing.md).

# Spremljanje obdelave

Ko se obdelava začne, Chloros ponuja več načinov za spremljanje napredka, preverjanje morebitnih težav in razumevanje, kaj se dogaja z vašim naborom podatkov. Na tej strani je pojasnjeno, kako spremljati obdelavo in razlagati informacije, ki jih zagotavlja Chloros.

## Pregled kazalnika napredka

Kazalnik napredka v zgornjem naslovnem pasu prikazuje stanje obdelave v realnem času in odstotek dokončanja. Napredek se v živo prenaša iz strežniškega dela prek Server-Sent Events (SSE), zato trak odraža dejansko delovanje procesne verige.

### Trak napredka v brezplačnem načinu

Za uporabnike brez licence Chloros+:

**Prikaz napredka v dveh fazah:**

1.**Zaznavanje ciljev** – iskanje kalibracijskih ciljev na slikah
2. **Obdelava** – uporaba popravkov in izvoz**Indikator napredka prikazuje:**

* Skupni odstotek dokončanja (0–100 %)
* Ime trenutne stopnje
* Preprosta vizualizacija v obliki vodoravne vrstice

### Kazalnik napredka Chloros+

Za uporabnike z licenco Chloros+:

**4-stopenjski prikaz napredka:**

1.**Zaznavanje** – iskanje kalibracijskih ciljev
2. **Analiza** – pregledovanje slik in priprava procesnega toka
3. **Kalibracija** – uporaba popravkov vinjetiranja in odbojnosti
4. **Izvoz** – shranjevanje obdelanih datotek**Interaktivne funkcije:*** **Preletite z miško** nad kazalnikom napredka, da se prikaže razširjen 4-stopenjski panel
* **Kliknite** na kazalnik napredka, da zamrznete/pritrdite razširjeni panel
* **Kliknite še enkrat**, da odmrznete in se panel samodejno skrije ob umiku miške
* Vsaka stopnja prikazuje posamezen napredek (0–100 %)

{% hint style="info" %}
**CLI parnost**: med izvajanjem `chloros-cli process` se za iste štiri niti prikaže stanje »Zaznavanje«, »Analiziranje«, »Processing«, »Exporting« in »`chloros-cli export-status`« prikazuje v realnem času napredek izvoza niti 4 iz drugega terminala. Glej [CLI Referenca](../reference/cli-reference.md).
{% endhint %}

***

## Razumevanje posameznih stopenj obdelave

{% hint style="info" %}
**Arhitektura poteka obdelave**: Te 4 stopnje grafičnega uporabniškega vmesnika ustrezajo [poteku obdelave s 4 niti](../processing-architecture/processing-pipeline.md). V sistemih s pospešitvijo prek grafičnega procesorja (GPU) niti 3 (kalibracija) izkorišča [dinamično prilagajanje izračunov](../processing-architecture/dynamic-compute-adaptation.md), ki optimizira obdelavo za vašo specifično strojno opremo.
{% endhint %}

### Stopnja 1: Zaznavanje (zaznavanje ciljev)

**Kaj se dogaja:**

* Chloros pregleda slike, ki ste jih označili s potrditvenim poljem »Target« (vse slike, če ni označena nobena)
* Algoritmi računalniškega vida prepoznajo kalibracijske plošče
* Iz vsake plošče se izluščijo vrednosti odbojnosti
* Za pravilno načrtovanje kalibracije se zabeležijo časovni žigi ciljev

**Trajanje:**

* Z označenimi cilji: 10–60 sekund
* Brez označenih ciljev: 5–30+ minut (skenira vse slike)

**Kazalnik napredka:**

* Prepoznavanje: 0 % → 100 %
* Število skeniranih slik (šteje le slike, ki se dejansko skenirajo)
* Število najdenih ciljev

**Na kaj je treba paziti:**

* Postopek naj bi se hitro zaključil, če so cilji pravilno označeni
* Če traja predolgo, cilji morda niso označeni
* V dnevniku za odpravljanje napak preverite sporočila »Cilj najden«

### Faza 2: Analiza

**Kaj se dogaja:**

* Branje metapodatkov EXIF slik (časovni žigi, nastavitve osvetlitve)
* Določanje strategije kalibracije na podlagi časovnih žigov ciljev in razpoložljivih podatkov DAQ o navzdolnem toku
* Urejanje čakalne vrste za obdelavo slik
* Priprava delavcev za vzporedno obdelavo (samo Chloros+)

**Trajanje:** 5–30 sekund**Kazalnik napredka:**

* Analiza: 0 % → 100 %
* Hitra faza, običajno se hitro zaključi

**Na kaj je treba paziti:**

* Napredek mora potekati enakomerno brez premorov
* Opozorila o manjkajočih metapodatkih se bodo pojavila v dnevniku odpravljanja napak

### Faza 3: Kalibracija

**Kaj se dogaja:*** **Debayering**: Pretvorba RAW Bayerjevega vzorca v 3 kanale (preskoči se za LATTICE mono module, z opombo)
* **Korekcija vinjetiranja**: Odstranjevanje potemnjenja na robovih objektiva
* **Kalibracija odbojnosti**: Normalizacija s ciljnimi vrednostmi in/ali zmanjšanjem signala DAQ
* **Izračun indeksov**: Izračun multispektralnih indeksov
* Obdelava vsake slike skozi celoten proces

**Trajanje:** Večina celotnega časa obdelave (60–80 %)**Kazalnik napredka:**

* Kalibriranje: 0 % → 100 %
* Trenutno se obdeluje slika
* Obdelane slike / Skupno število slik

**Način obdelave:*** **Prosti način**: Zaporedno obdeluje po eno sliko naenkrat
* **Način Chloros+**: Zaganjata skupino delavcev, ki se prilagaja strojni opremi — 1–4 sočasnih delavcev na sistemih z grafičnimi procesorji (glede na VRAM), en delavec na fizično jedro (minus eno) na sistemih, ki uporabljajo le procesor. Glej [Dinamično prilagajanje računalniške moči](../processing-architecture/dynamic-compute-adaptation.md)
* **Pospešitev z GPU**: znatno pospeši to fazo**Na kaj je treba paziti:**

* Enakomeren napredek glede na število slik
* Preverite dnevnik razhroščevanja za sporočila o zaključku posameznih slik
* Opozorila o težavah s kakovostjo slike ali kalibracijo

### Faza 4: Izvoz

**Kaj se dogaja:**

* Zapišanje obdelanih slik na disk v izbranem formatu, takoj ko so dokončane
* **LATTICE**: vsak okvir se razširi v vsak omogočen produkt (debayering / predogled / sevanje / odbojnost)
* Izvoz multispektralnih indeksnih slik z barvami LUT
* Ustvarjanje izhodnega drevesa `<project>/<camera>/<format>/<Product>_Images/` — izvožene datoteke ohranijo ime izvorne datoteke; mapa označuje produkt

**Trajanje:** 10–20 % celotnega časa obdelave**Kazalnik napredka:**

* Izvoz: 0 % → 100 %
* Pisanje datotek
* Izvozni format in ciljna mapa

**Na kaj je treba paziti:**

* Opozorila o prostem prostoru na disku
* Napake pri pisanju datotek
* Zaključek vseh nastavljenih izvozov

***

## Zavihek »Debug Log« (Dnevnik odpravljanja napak)

Dnevnik odpravljanja napak vsebuje podrobne informacije o napredku obdelave in morebitnih težavah. V dnevniško konzolo se prenašajo tudi sporočila o zagonu ozadja, zato dnevnik prikazuje celoten potek dogodkov, tudi če ga odprete pozneje.

### Dostop do dnevnika odpravljanja napak

1. V levem stranskem meniju kliknite ikono **Dnevnik odpravljanja napak**<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">


2. Odpre se okno z dnevnikom, ki prikazuje sporočila o obdelavi v realnem času
3. Vsebina se samodejno pomika navzdol, da se prikažejo najnovejša sporočila

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written) -->

### Razumevanje sporočil v dnevniku

Vrstice dnevnika Chloros imajo na začetku oznake v oklepajih, ki poimenujejo podsistem – na primer `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Najpomembnejše, kar morate vedeti, je **povzetek izvedbe**, ki se izpiše na koncu vsake izvedbe (vključno z ustavljenimi izvedbami):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Dodatne vrstice z namigi `[RUN-SUMMARY]` sledijo vsakič, ko je potrebno nekaj pojasniti — na primer izvedba, ki ni prinesla nobenega rezultata, ali kamera, katere zahtevani produkt je bil preskočen kot neuporaben. Vrstice `[EXPORT-CHECK]` pojasnjujejo preskoke po posameznih kamerah (npr. zakaj kamera RGB ni dobila produkta sevanja).

Splošne stopnje resnosti sporočil (primeri spodaj so ilustrativni, ne dobesedni):

#### Informacijska sporočila (bela/siva)

Običajne posodobitve obdelave: začetek obdelave, zaznane tarče (s številom panelov), napredek kalibracije po posameznih slikah, izvožene datoteke, obdelava zaključena.

#### Opozorilna sporočila (Rumena)

Nekritične težave, ki ne ustavijo obdelave — npr. manjkajoči podatki GPS v posnetku, velika vrzel v časovnih oznakah med slikami ciljev ali nizek kontrast v kalibracijskem panelu.

**Ukrep:** Po obdelavi preglejte opozorila, vendar obdelave ne prekinjajte

#### Napaka (Red)

Kritične težave, ki lahko povzročijo neuspešno obdelavo — npr. poln disk, poškodovana slikovna datoteka ali ni bilo zaznanih ciljev, medtem ko je bila zahtevana kalibracija odbojnosti.

**Ukrep:** Ustavi obdelavo, odpravi napako, ponovno zaženi

### Pogoste situacije v dnevniku

| Situacija                             | Pomen                                       | Potreben ukrep                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Cilj zaznan v \[ime datoteke]        | Kalibracijski cilj uspešno najden         | Noben – normalno                                         |
| Črte napredka za posamezno sliko              | Posodobitev trenutnega napredka                       | Noben – normalno                                         |
| Cilji niso bili najdeni                      | Kalibracijski cilji niso bili zaznani               | Označite slike ciljev ali onemogočite kalibracijo odbojnosti |
| Premajhen prostor na disku               | Premalo prostora za shranjevanje izhodnih podatkov                 | Osvobodite prostor na disku                                    |
| Preskočitev poškodovane datoteke               | Datoteka slike je poškodovana                         | Ponovno kopirajte datoteko s kartice SD                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | Posnetka brez surovega okvirja ni mogoče obdelati | Ponovno posnemite s surovim okvirjem ali uporabite CLI `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | Izvedba ni ustvarila nobenih slikovnih izdelkov — poročano kot napaka z namigi | Preberite vrstice z namigi; preverite, kaj je bilo preskočeno in zakaj |

### Kopiranje podatkov dnevnika

Za kopiranje dnevnika za odpravljanje težav ali podporo:

1. Odprite okno »Debug Log«
2. Kliknite gumb **»Kopiraj dnevnik«** (ali desni klik → Izberi vse)
3. Prilepite v besedilno datoteko ali e-pošto
4. Po potrebi pošljite podpori MAPIR

***

## Nadzor sistemskih virov

### Poraba procesorja

**Prosti način:**

* 1 jedro procesorja pri ~100 %
* Druga jedra so v mirovanju ali na voljo
* Sistem ostaja odziven

**Chloros+ vzporedni način:**

* Več jeder z visoko izkoriščenostjo — število je odvisno od strategije, izbrane v [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md)
* Sistem se lahko zdi manj odziven

**Za spremljanje:**

* Windows Upravitelj opravil (Ctrl+Shift+Esc)
* Zavihek Zmogljivost → razdelek Procesor
* Poiščite procese »Chloros« ali »chloros-backend«

### Poraba pomnilnika (RAM)

**Tipična poraba:**

* Majhni projekti (&lt; 100 slik): 2–4 GB
* Srednji projekti (100–500 slik): 4–8 GB
* Veliki projekti (500+ slik): 8–16 GB
* Vzporedni način Chloros+ porabi več pomnilnika RAM

**Če je pomnilnika premalo:**

* Obdelujte manjše serije
* Zaprite druge aplikacije
* Nadgradite pomnilnik RAM, če redno obdelujete velike nize podatkov

### Poraba GPU (Chloros+ s CUDA)

Ko je pospeševanje z grafičnim procesorjem omogočeno:

* Grafični procesor NVIDIA kaže visoko izkoriščenost (60–90 %)
* Poraba VRAM se poveča (zahteva 4 GB+ VRAM; 7 GB+ za sočasno odstranjevanje pik pri teksturah)
* Faza kalibracije je znatno hitrejša

**Za spremljanje:**

* Ikona NVIDIA v sistemski vrstici
* Upravitelj opravil → Zmogljivost → GPU
* GPU-Z ali podobno orodje za spremljanje

### Vhodno-izhodne operacije na disku

**Kaj lahko pričakujete:**

* Visoka hitrost branja z diska med fazo analize
* Visoka hitrost pisanja na disk med fazo izvoza
* SSD je znatno hitrejši od HDD

**Nasvet za zmogljivost:**

* Če je mogoče, za mapo projekta uporabite SSD
* Pri velikih naborih podatkov se izogibajte omrežnim pogonom
* Poskrbite, da disk ni skoraj poln (vpliva na hitrost pisanja)

***

## Odkrivanje težav med obdelavo

### Opozorilni znaki

**Napredek se ustavi (brez sprememb več kot 5 minut):**

* Preverite dnevnik razhroščevanja za napake
* Preverite razpoložljiv prostor na disku
* Preverite upravitelja opravil, da se prepričate, da teče Chloros

**Pogosto se pojavljajo sporočila o napakah:**

* Prekinite obdelavo in preglejte napake
* Pogosti vzroki: prostora na disku, poškodovane datoteke, težave s pomnilnikom
* Glejte poglavje »Odpravljanje težav« spodaj

**Sistem ne odziva:**

* Chloros+ v vzporednem načinu porablja preveč virov
* Razmislite o zmanjšanju števila sočasnih nalog ali nadgradnji strojne opreme
* Prosti način porablja manj virov

### Kdaj ustaviti obdelavo

Ustavite obdelavo, če opazite:

* ❌ Napake »Disk je poln« ali »Datoteke ni mogoče zapisati«
* ❌ Ponavljajoče se napake poškodovanih slikovnih datotek
* ❌ Sistem se je popolnoma zamrznil (ne odziva se)
* ❌ Ugotovili ste, da so bile nastavljene napačne nastavitve
* ❌ Uvožene so bile napačne slike

**Kako ustaviti:**

1. Kliknite**gumb »Stop«** (nadomešča gumb »Start«) – enkrat je dovolj
2. Na vrstici se prikaže »Ustavljanje...«, medtem ko se zaključi obdelava slike, ki je v teku, nato se izvajanje konča v ustavljenem stanju
3. Že izvoženi izdelki ostanejo na disku; dnevnik prikaže natančen zapis `[RUN-SUMMARY]` o tem, kaj je bilo zaključeno
4. Odpravite težave in ponovno zaženite — izvajanje se začne od začetka

***

## Odpravljanje težav med obdelavo

### Obdelava poteka zelo počasi

**Možni vzroki:**

* Neoznačene ciljne slike (skeniranje vseh slik)
* Shranjevanje na HDD namesto na SSD
* Nezadostni sistemski viri
* Konfiguriranih je veliko indeksov
* Dostop do omrežnega pogona

**Rešitve:**

1. Če ste šele začeli in ste v fazi »Detecting«: ustavite, označite cilje, ponovno zaženite
2. Za prihodnost: Uporabite SSD, zmanjšajte število indeksov, nadgradite strojno opremo
3. Razmislite o uporabi CLI za obdelavo velikih naborov podatkov v paketu

### Opozorila »Prostor na disku«

**Rešitve:**

1. Takoj sprostite prostor na disku
2. Premestite projekt na pogon z več prostora
3. Zmanjšajte število indeksov za izvoz
4. Onemogočite izvoz LATTICE izdelkov, ki jih ne potrebujete (Nastavitve projekta → Obdelava)
5. Uporabite format JPG namesto TIFF (manjše datoteke)

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
3. Zmanjšajte obremenitev pri obdelavi (uporabite način Free namesto Chloros+)
4. Obdelujte v hladnejših delih dneva

***

## Obvestilo o zaključku obdelave

Ko se obdelava konča:

* Kazalnik napredka doseže 100 %
* V dnevniku za odpravljanje napak se prikažejo vrstice `[RUN-SUMMARY]` s končnimi številkami
* Gumb »Začni« postane spet aktiven
* Vse izhodne datoteke so v izhodni mapi projekta za posamezno kamero: `<project>/<camera>/<format>/<Product>_Images/`

***

## Naslednji koraki

Ko se obdelava zaključi:

1. **Preglejte rezultate** – glejte [Zaključek obdelave](finishing-the-processing.md)
2. **Preverite izhodno mapo** – preverite, ali so bile vse datoteke pravilno izvožene
3. **Preglejte dnevnik odpravljanja napak** – Preverite, ali so prisotna kakšna opozorila ali napake
4. **Predogled obdelanih slik** – Uporabite pregledovalnik slik ali zunanjo programsko opremo

Za informacije o pregledu in uporabi obdelanih rezultatov glejte [Zaključek obdelave](finishing-the-processing.md).

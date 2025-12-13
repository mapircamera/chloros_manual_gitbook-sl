# Indeks/LUT Sandbox

Indeks/LUT Sandbox je interaktivno delovno okolje znotraj Chloros Image Viewer, ki vam omogoča eksperimentiranje z izračuni multispektralnih indeksov in vizualizacijami barv v realnem času. To zmogljivo orodje vam pomaga preizkušati različne indekse, izpopolnjevati vrednostna območja in ustvarjati vizualizacije, pripravljene za objavo, brez ponovne obdelave celotnega niza podatkov.

## Kaj je indeks/LUT peskovnik?

### Namen

Peskovnik omogoča:

* **Izračun indeksa v realnem času** – takojšnjo uporabo katerega koli vegetacijskega indeksa
* **Interaktivno prilagajanje LUT** – natančno prilagajanje barvnih prehodov in razponov
* **Optimizacijo delovnega toka** – določitev najboljših nastavitev pred obdelavo v seriji

### Sandbox v primerjavi z obdelavo projekta

**Index/LUT Sandbox (interaktiven):**

* En slika naenkrat
* Takojšnja povratna informacija
* Eksperimentalno in iterativno
* Brez trajnih sprememb datotek
* Idealno za raziskovanje in testiranje

**Obdelava projekta (skupinska):**

* Celoten niz podatkov naenkrat
* Vnaprej konfigurirane nastavitve
* Trajne izhodne datoteke
* Časovno intenzivno
* Najboljše, ko so nastavitve dokončne

{% hint style=&quot;success&quot; %}
**Najboljši potek dela**: Uporabite peskovnik za eksperimentiranje in poiščite optimalne nastavitve indeksa in LUT, nato pa te nastavitve uporabite med obdelavo projekta za celoten niz podatkov.
{% endhint %}

***

## Delovanje s peskovnikom indeksa/LUT

### Razumevanje vnaprej izračunanih indeksov

V Chloros se indeksi lahko uporabijo med obdelavo projekta. Da bi določili, katere nastavitve indeksa in LUT želite uporabiti za izvoz, je najlažje uporabiti peskovnik za pregledovanje slik.

Peskovnik vam omogoča:

* **Uporabite nove indekse in barvne prehode (LUT)** za vizualizacijo podatkov.
* **Prilagodite nastavitve vizualizacije** interaktivno.
* **Ogledate** že izračunane indeksne slike.
* **Pregledate** vrednosti pikslov na vseh ravneh povečave.

### Odpiranje peskovnika

Do peskovnika indeksa/LUT dostopate v **pregledovalniku slik** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Kliknite sliko v mreži slik brskalnika datotek, da se odpre v **pregledovalniku slik** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> .
2. Kliknite **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> , da odprete levi stranski meni, če še ni odprt.

### Izbiranje slike za uporabo indeksa/LUT

Za delo z indeksom v pregledovalniku slik <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. **Odprite sliko** iz glavne mreže slik s klikom nanjo.
2. Odpre se zavihka **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> .
3. Kliknite **spustni meni Sloj** (v zgornjem desnem kotu pregledovalnika).
4. Iz spustnega menija izberite sloj:
   * RAW (odbojnost)

### Uporaba indeksa na sliki

Ko je slika v polnem zaslonu in je odprt stranski trak **Pregledovalnik slik** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> odprt:

1. Potrdite polje Indeks na vrhu stranskega menija.
2. Izberite filter kamere iz spustnega menija na levi.
3. Izberite želeno indeksno formulo iz spustnega menija na desni.
4. Povlecite barvne kroge filtracijskega kanala na mesta v indeksni formuli spodaj.
5. Ko je formula veljavna, se slika posodobi in prikaže indeksne vrednosti.
6. Premikajte kurzor miške, da vidite vrednosti na mestu kurzorja.
7. Povečajte sliko, da vidite posamezne piksle in njihove vrednosti.

Vsak indeks ima določen razpon vrednosti in pomen:

#### NDVI Primer

```
Formula: (NIR - Red) / (NIR + Red)

For Survey3W RGN camera:
NIR = 850nm band
Red = 661nm band

Result range: -1.0 to +1.0
Typical vegetation: 0.4 to 0.9
Stressed vegetation: 0.2 to 0.4
Bare soil: 0.0 to 0.2
Water: -0.1 to 0.1
```

Za popolno dokumentacijo indeksnih formul glejte [Multispektralne indeksne formule](../project-settings/multispectral-index-formulas.md).

***

## Delovanje z LUT-ji (preglednimi tabelami)

### Kaj je LUT?

**Pregledna tabela (LUT)** preslika numerične vrednosti indeksa v barve za vizualizacijo:

* **Vnos**: vrednost pikslov indeksa (npr. NDVI 0,65)
* **Izhod**: barva RGB (npr. svetlo zelena)
* **Namen**: olajšati preglednost in razlago vzorcev

**Siva lestvica v primerjavi z barvno LUT:**

* Siva lestvica: znanstvena in nevtralna, prikazuje surove podatke
* Barvna LUT: intuitivna in učinkovita, poudarja vzorce in razlike

{% hint style=&quot;success&quot; %}
**Moč vizualizacije**: Uporaba barvne LUT na sivo lestvični indeksni sliki znatno olajša prepoznavanje vzorcev, anomalij in zanimivih področij na prvi pogled.
{% endhint %}

### Uporaba LUT na indeksni sliki

Ko imate indeksno sliko, ki prikazuje

1. Kliknite gumb <img src="../.gitbook/assets/image.png" alt="" data-size="line"> gumb »+Dodaj LUT«
2. Izberite barvni prehod
3. Prilagodite minimalne/maksimalne končne točke izreza
4. Prilagodite način izreza
5. V **pogledovalniku slik** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> , da uporabite LUT.

### Izbiranje barvnega prehoda

**Izbiranje prehoda:**

1. V plošči LUT poiščite **barvno vrstico prehoda**.
2. Premaknite miško nad njo, da si ogledate razpoložljive prednastavitve prehoda.
3. Izberite želeni prehod.
4. Slika se **takoj posodobi** z novimi barvami, ko je potrjeno polje Indeks.

{% namig style=&quot;success&quot; %}
**Najboljša praksa**: Za vegetacijske indekse, kot je NDVI, je prehod Red-Yellow-Green najbolj intuitiven, ker je usklajen z naravnimi barvnimi povezavami (zelena = zdrava, rumena = zmerna, rdeča = obremenjena).
{% endhint %}

### Prilagajanje barvnih razredov

**Nadzor razredov** določa, koliko ločenih barvnih stopenj se prikaže v vašem prehodu:

**Možnosti števila razredov:**

* **2–5 razredov**: zelo široke kategorije, različne cone
* **6–10 razredov**: uravnoteženo, primerno za razvrščanje
* **11–20 razredov**: gladki prehodi, neprekinjen videz
* **20+ razredov**: skoraj neprekinjen, največja gladkost

**Kako prilagoditi:**

1. V plošči LUT poiščite **barvne kvadratke pod drsnikom barvnega prehoda**.
2. Število razredov prilagodite z dodajanjem s tipko +.
3. Število razredov odstranite z dvojnim klikom na barvni kvadratek.
4. Barvni prehod se **v realnem času** posodobi na sliki.

**Učinek na vizualizacijo:**

* **Manj razredov** (3–5): Ustvari različne cone, poenostavljeno razvrščanje, lažje razlikovanje kategorij.
* **Srednje razrede** (6–10): Uravnotežen pristop, primeren za večino uporab.
* **Več razredov** (15–20): Gladki prehodi, podrobne razlike, fotografski videz.

**Kdaj uporabiti:**

* **Malo razredov (3–5)**: Predstavitvene diapozitive, razvrstitvene karte, preprosta poročila.
* **Srednje razrede (6–10)**: splošna analiza, uravnoteženi podrobnosti, standardna poročila
* **Veliko razredov (15–20)**: znanstvena analiza, podrobna inšpekcija, izhodi v kakovosti za objavo

### Natančno nastavljanje vrednostnih razponov

**Nadzor vrednostnih razponov** določa, katere indeksne vrednosti se pripisujejo katerim barvam v vašem prelivu:

**Nadzor razponov v plošči LUT:**

* **Najmanjša vrednost**: spodnja meja barvne lestvice
* **Največja vrednost**: zgornja meja barvne lestvice
* **Vmesne vrednosti**: samodejno porazdeljene med najmanjšo in največjo vrednostjo (glede na število razredov)

#### Prilagajanje najmanjših/največjih vrednosti

**Prilagajanje vrednostnih razponov:**

1. V plošči LUT poiščite vnosna polja **Najmanjša vrednost** in **Največja vrednost**
2. Kliknite polje **Minimalna vrednost**.
3. Vnesite želeno minimalno vrednost (npr. `0.2`).
4. Pritisnite **Enter** ali kliknite zunaj polja.
5. Ponovite za polje **Maksimalna vrednost** (npr. `0.9`).
6. Vizualizacija se **takoj posodobi**.

{% namig style=&quot;info&quot; %}
**Samodejno prilagajanje**: Ko prvič uporabite LUT, Chloros samodejno nastavi minimalno/maksimalno vrednost na dejanski razpon podatkov v sliki. Nato lahko to območje zožite, da se osredotočite na določena območja vrednosti, ki vas zanimajo.
{% endhint %}

**Primer NDVI prilagoditev območja:**

* **Celoten razpon**: `-1.0` do `1.0` (prikaži vse možne vrednosti)
* **Osredotočeno na vegetacijo**: `0.2` do `0.9` (izključi golo zemljo in vodo)
* **Samo zdrava vegetacija**: `0.5` do `0.9` (poudarite samo bujne rastline)
* **Zaznavanje stresa**: `0.2` do `0.5` (poudarite problematična območja)
* **Prilagojeno območje**: prilagodite na podlagi opazovanih vrednosti pikslov

**Zakaj prilagajati območja?**

* **Povečajte kontrast** v območju, ki vas zanima
* **Izključite nepomembne vrednosti** (npr. vodne površine, gola tla)
* **Standardizirajte vizualizacijo** za več slik ali datumov
* **Poudarite subtilne razlike** znotraj ozkega območja vrednosti

### Izrezovanje vrednosti zunaj območja

Ko vrednosti pikslov padejo zunaj vašega opredeljenega minimalnega/maksimalnega območja, lahko nadzirate, kako se prikazujejo, z uporabo **načinov izrezovanja**.

#### **Razpoložljive možnosti načinov izrezovanja:**

#### 1. Minimalno in maksimalno

* Piksli **pod minimalno vrednostjo** → prikaz z uporabo **prve barve** v prelivu (npr. rdeča)
* Piksli **nad maksimumom** → prikaz z **zadnjo barvo** v prelivu (npr. zelena)
* **Primer uporabe**: poudarjanje skrajnosti, prikaz celotnega obsega podatkov z nasičenimi barvami na mejah
* **Primer**: vrednosti NDVI pod 0,2 so prikazane rdeče, vrednosti nad 0,9 pa zelene

#### 2. Prozorno ozadje

* Piksli **zunaj obsega** postanejo **popolnoma prozorni**
* Samo piksli **znotraj obsega** prikazujejo barvni prehod
* **Primer uporabe**: GIS prekrivanje, izolacija določenih obsegov vrednosti, poudarjanje samo zanimivih področij
* **Primer**: Prikaz samo NDVI 0,4–0,7 v barvi, vse ostalo prozorno

{% hint style=&quot;warning&quot; %}
**Omejitev prosojnosti**: Prosojni piksli se v pregledovalniku prikažejo kot barva ozadja. Pri izvozu med obdelavo se prosojnost ohrani v formatu PNG, vendar ne v formatu JPG.
{% endhint %}

#### 3. Indeks ozadja

* Piksli **zunaj območja** se prikažejo v **sivi barvi** (prikazujejo surove indeksne vrednosti)
* Piksli **znotraj območja** prikazujejo **barvni prehod**
* **Primer uporabe**: Nežno poudarjanje, ohranjanje konteksta ob poudarjanju zanimivih območij
* **Primer**: Barvno poudarite obremenjeno vegetacijo (NDVI 0,3–0,5), zdrava območja pa prikazujte v sivi barvi

#### 4. Originalno ozadje

* Piksli **zunaj območja** prikazujejo **izvirno multispektralno sliko**
* Piksli **znotraj območja** prikazujejo **barvni prehod**
* **Primer uporabe**: Najbolj intuitiven – združuje naravni kontekst slike z analitičnim barvnim prekrivanjem
* **Primer**: Oglejte si dejanski videz polja/pridelka s prekrivanjem območij stresa, označenih z barvami

### Izbiranje pravega načina izrezovanja

| Način izrezovanja              | Najbolj primerno za                                   | Stil vizualizacije          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimalno in maksimalno**    | Prikaz vseh podatkov, znanstvena analiza     | Vsi piksli so obarvani           |
| **Prozorno ozadje** | GIS prekrivanja, izolacija določenih območij    | Barva na območju, prazno zunaj območja |
| **Indeksno ozadje**       | Nežno poudarjanje, ohranjanje konteksta podatkov  | Barva na območju, sivo zunaj območja  |
| **Originalno ozadje**    | Poročila, predstavitve, intuitivna analiza | Barva na območju, fotografija zunaj območja |

### Ustvarjanje prilagojenih barv LUT

Za popoln nadzor nad vizualizacijo lahko ustvarite **prilagojene barvne prehode** z urejanjem posameznih barvnih prehodov.

**Ustvarjanje prilagojenega prehoda:**

1. V plošči LUT poiščite **pregledno vrstico prehoda**
2. Poiščite **barvne kvadratke** pod prehodom
3. **Kliknite barvno stopnjo**, da jo izberete.
4. Odpre se **izbirnik barv**.
5. Izberite novo barvo z uporabo:
   * **Barvnega kolesa**: vizualna izbira barv.
   * **RGB/HSV drsnikov**: natančen nadzor barv.
   * **Vnos šestnajstiškega kode**: natančna specifikacija barve (npr. `#FF0000` za rdečo)
6. Kliknite izven izbirnika barv **, da uporabite novo barvo**
7. Prehod **se takoj posodobi** na sliki

**Dodajanje ali odstranjevanje barvnih prehodov:**

* **Dodajanje prehoda**: kliknite ikono +, da dodate novo barvno vzorčico na konec
* **Odstranjevanje prehoda**: dvokliknite barvni kvadrat, da odstranite vzorčico

**Strategije prilagajanja:**

* **Obratni prehod**: obrnite vrstni red barv, da obratite pomen (npr. zelena = nizka, rdeča = visoka)
* **Barve blagovne znamke**: prilagodite barvno paleto vaše organizacije za poročila
* **Primerno za barvno slepe**: uporabite kombinacije oranžno-modre ali vijolično-rumene
* **Optimizacija tiskanja**: izberite barve, ki delujejo tako pri barvnem kot pri sivinskem tiskanju
* **Več pragov**: uporabite različne barve pri določenih pragovnih vrednostih za razvrščanje

{% namig style=&quot;info&quot; %}
**Shranjevanje prilagojenih prehodov**: Prilagojene prehode lahko shranite in ponovno uporabite. Kliknite ikono za shranjevanje v oknu LUT, da shranite prilagojene barvne sheme za prihodnjo uporabo.
{% endhint %}

***

## Interaktivni delovni tok

### Posodobitve v realnem času

Vse prilagoditve LUT v peskovniku posodobijo sliko **takoj in interaktivno**:

* **Preklopite sloj** → Slika se takoj spremeni
* **Izberite preliv** → Barve se takoj posodobijo
* **Prilagodite razpon vrednosti** → Kontrast se spremeni v realnem času
* **Spremeni razrede** → Gladkost prehoda se takoj posodobi
* **Spremeni izrez** → Prikaz ozadja se takoj spremeni
* **Uredi barve** → Prilagojeni prehod se takoj uporabi

**Gumb »Uporabi« ni potreben** – vse spremembe so v realnem času in interaktivne!

{% hint style=&quot;success&quot; %}
**Takojšnja povratna informacija**: Takojšnja vizualna povratna informacija vam omogoča hitro preizkušanje različnih nastavitev, dokler ne najdete optimalne vizualizacije za vaše potrebe analize.
{% endhint %}

### Iterativni postopek izpopolnjevanja

**Tipični postopek optimizacije LUT:**

1. **Izberite indeksno plast** (npr. RAW (odbojnost))
2. **Uporabite indeks** – izberite filter kamere in indeksno formulo, povlecite barvne kroge na ustrezno mesto v indeksni formuli
3. **Uporabite LUT gradient** – začnite s prednastavitvijo Red-Yellow-Green
4. **Preglejte vrednosti pikslov** – premikajte kurzor in si zabeležite vrednostne razpone
5. **Prilagodite min/max** – zožite, da se osredotočite na vegetacijo (npr. 0,2 do 0,9)
6. **Izberite izrez** – poskusite z »Original Background« za kontekst
7. **Izboljšajte barve** – po potrebi prilagodite gradient za posebno poudarjanje
8. **Dokončajte nastavitve** – Dokumentirajte nastavitve in jih kopirajte v nastavitve projekta za izvoz

### Pregled vrednosti pikslov

Razumevanje dejanskih vrednosti pikslov je ključnega pomena za nastavitev učinkovitih razponov LUT:

**Kako pregledati vrednosti:**

1. Vrednosti pikslov se prikažejo, ko je za sliko **označeno** polje Index ali pa sta označeni polji Index in LUT.
2. **Premaknite kurzor** nad različne dele slike
3. **Opazujte vrednosti pikslov**, ki se prikažejo v legendi, ko se z miško približate
4. Povečajte sliko, da vidite posamezne piksle, označene s plavajočo vrednostjo
5. **Zapišite si** vrednostna območja za različne značilnosti:
   * **Zdrava vegetacija**: npr. NDVI 0,55–0,85
   * **Obremenjena vegetacija**: npr. NDVI 0,30–0,50
   * **Gola tla**: npr. NDVI 0,05–0,25
   * **Voda** (če je prisotna): npr. NDVI -0,05 do 0,10

**Uporaba vrednosti pikslov za nastavitev obsega LUT:**

Po pregledu vrednosti pikslov ustrezno prilagodite minimalno/maksimalno vrednost LUT:

**Primer scenarija:**

* **Opazovanje**: vrednosti tal = 0,05–0,25, stres = 0,25–0,50, zdravo = 0,50–0,85
* **Cilj**: Vizualizirajte samo zdravje rastlin (izključite tla)
* **Nastavitve LUT**: Min = `0.25`, Max = `0.85`
* **Obrezovanje**: »Originalno ozadje«, da se tla prikažejo v naravni barvi
* **Rezultat**: Barvni prehod se nanaša samo na vegetacijo, tla se prikažejo kot originalna slika

{% hint style=&quot;info&quot; %}
**Dinamični razpon**: Različne kulture, letni časi in faze rasti imajo različne vrednostne razpone. Pred nastavitvijo razponov LUT vedno preverite vrednosti pikslov v svojem konkretnem nizu podatkov.
{% endhint %}

***

## Prilagojeni indeksi (Chloros+)

### Ustvarjanje prilagojenih indeksnih formul

{% hint style=&quot;info&quot; %}
**Kje ustvariti**: Prilagojene indekse lahko konfigurirate v **Nastavitvah projekta** pred obdelavo, pa tudi v stranskem pasu peskovnika pregledovalnika slik.
{% endhint %}

**Za ustvarjanje prilagojenega indeksa:**

1. **Odprite nastavitve projekta** (pred obdelavo) ali stranski pas Image Viewer sandbox.
2. Prejdite na **spustni meni formule indeksa**.
3. Poiščite možnost **„Prilagojeno“** (morate biti prijavljeni z licenco Chloros+).
4. **Določite svojo formulo** z uporabo spremenljivk pasov:
   * Imena pasov: `NIR`, `Red`, `Green`, `Blue`, `RedEdge` itd.
   * Operacije: `+`, `-`, `*`, `/`, `^` (eksponent)
   * Funkcije: `sqrt()`, `abs()` itd. (če je podprto)
   * Oklepaji: `()` za vrstni red operacij
5. **Poimenujte svoj indeks** (npr. »MyIndex« ali »CustomNDVI«)
6. **Shranite konfiguracijo**

**Primeri prilagojenih formul:**

```
Modified NDVI with offset:
(NIR - Red) / (NIR + Red + 0.5)

Simple ratio:
NIR / Red

Complex multi-band:
(NIR - Red) / (NIR + Red - Blue)

Exponential index:
(NIR / Red) ^ 2
```

{% hint style=&quot;warning&quot; %}
**Preverjanje formule**: Preverite, ali formula uporablja pasove, ki so na voljo v vaši kameri. Na primer, RedEdge je na voljo samo na kamerah s filtrom RedEdge.
{% endhint %}

***

## Naslednji koraki

Zdaj, ko razumete Index/LUT Sandbox:

* **Uporabi za obdelavo**: Uporabi odkrita nastavitev v [Nastavitve projekta](../project-settings/project-settings.md)
* **Obdelava v seriji**: Uporabi optimizirane indekse za celotne podatkovne nize
* **Več informacij**: Preberi [Formule za multispektralne indekse](../project-settings/multispectral-index-formulas.md)

Povezana dokumentacija:

* [**Sloji slike**](image-layers.md) – Upravljanje slojev in vizualizacija
* [**Odpiranje slike v polnem zaslonu**](opening-an-image-full-screen.md) – Osnove pregledovalnika slik
* [**Obdelava slik (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Celoten potek obdelave

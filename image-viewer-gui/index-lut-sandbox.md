# Sandbox za indekse/LUT

Sandbox za indekse/LUT je interaktiven delovni prostor znotraj programa Chloros Image Viewer, ki vam omogoča eksperimentiranje z izračuni multispektralnih indeksov in barvnimi vizualizacijami v realnem času. To zmogljivo orodje vam pomaga preizkušati različne indekse, izpopolnjevati vrednostna območja in ustvarjati vizualizacije, pripravljene za objavo, brez ponovne obdelave celotnega niza podatkov.

## Kaj je Index/LUT Sandbox?

### Namen

Sandbox omogoča:

* **Izračun indeksa v realnem času** – takojšnjo uporabo katerega koli vegetacijskega indeksa
* **Interaktivno prilagajanje LUT** – natančno nastavitev barvnih prehodov in razponov
* **Optimizacijo delovnega toka** – določitev najboljših nastavitev pred obdelavo v serijah

### Sandbox v primerjavi z obdelavo projekta

**Index/LUT Sandbox (interaktivno):**

* Po ena slika naenkrat
* Takojšnja povratna informacija
* Eksperimentalno in iterativno
* Brez trajnih sprememb datotek
* Idealno za raziskovanje in testiranje

**Obdelava projekta (v seriji):**

* Celoten niz podatkov naenkrat
* Vnaprej konfigurirane nastavitve
* Trajne izhodne datoteke
* Časovno intenzivno
* Najboljše, ko so nastavitve dokončne

{% hint style="success" %}
**Najboljši delovni tok**: Uporabite Sandbox za eksperimentiranje in iskanje optimalnih nastavitev indeksa in LUT, nato pa te nastavitve uporabite med obdelavo projekta za celoten niz podatkov.
{% endhint %}

***

## Delovanje s peskovnikom za indeks/LUT

### Razumevanje vnaprej izračunanih indeksov

V Chloros se indeksi lahko uporabijo med obdelavo projekta. Da bi določili, katere nastavitve indeksa in LUT želite uporabiti za izvoz, je najlažje uporabiti peskovnik za pregledovalnik slik.

Peskovnik vam omogoča:

* **uporabite nove indekse in barvne prehode (LUT)** za vizualizacijo podatkov
* **interaktivno prilagodite nastavitve vizualizacije*** **si ogledate** že izračunane indeksne slike
* **pregledate** vrednosti pikslov na vseh ravneh povečave

### Odpiranje peskovnika

Do peskovnika indeksov/LUT dostopate v zavihku **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Kliknite sliko v mreži slik brskalnika datotek, da se odpre v zavihku **Image Viewer**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> 2. Kliknite zavihek**Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> , da odprete levi izskočni stranski trak, če še ni odprt

### Izbiranje slike, na katero želite uporabiti indeks/LUT

Za delo z indeksom v <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. **Odprite sliko** iz glavne mreže slik s klikom nanjo
2. Nato se odpre zavihek **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> se nato odpre
3. Kliknite **spustni meni Sloji** (zgoraj desno v pregledovalniku)
4. Iz spustnega menija izberite sloj:
   * RAW (Odbojnost)

### Uporaba indeksa na sliki

Ko je slika v polnem zaslonu in je odprt stranski meni **Pregledovalnik slik** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Potrdite polje Indeks na vrhu stranskega menija
2. Izberite filter vaše kamere iz spustnega menija na levi
3. Izberite želeno formulo indeksa iz spustnega menija na desni
4. Povlecite barvne kroge kanalov filtra na mesta v formuli indeksa spodaj
5. Ko je formula veljavna, se bo slika posodobila in prikazala vrednosti indeksa
6. Premikajte kurzor miške, da vidite vrednosti na mestu kurzorja
7. Povečajte sliko, da vidite posamezne piksle in njihove vrednosti

Vsak indeks ima določen razpon vrednosti in pomen:

#### Primer NDVI

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

## Delovanje z LUT-i (preglednicami)

### Kaj je LUT?

**Preglednica (LUT)** preslika numerične vrednosti indeksov v barve za vizualizacijo:

* **Vhod**: Vrednost indeksa pikslov (npr. NDVI 0,65)
* **Izhod**: Barva (npr. svetlo zelena)
* **Namen**: Olajšati prepoznavanje in razlago vzorcev**LUT v sivih odtenkih proti barvnemu LUT:**

* Sivi odtenki: Znanstveno in nevtralno, prikazuje surove podatke
* Barvni LUT: Intuitivno in učinkovito, poudari vzorce in razlike

{% hint style="success" %}
**Moč vizualizacije**: Uporaba barvne LUT na indeksno sliko v sivih odtenkih znatno olajša prepoznavanje vzorcev, anomalij in območij interesa na prvi pogled.
{% endhint %}

### Uporaba LUT na indeksni sliki

Ko imate indeksno sliko, ki prikazuje

1. Kliknite gumb <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> gumb »+Dodaj LUT«
2. Izberite barvni prehod
3. Prilagodite minimalno in maksimalno končno točko izreza
4. Prilagodite način izreza
5. V stranskem meniju **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> v stranskem meniju, da uporabite LUT

### Izbiranje barvnega prehoda

**Izbiranje prehoda:**

1. V plošči LUT poiščite**barvno prehodno vrstico**

2. Premaknite miško nad njo, da si ogledate razpoložljive prednastavitve prehodov
3. Izberite želeni prehod
4. Slika se **takoj posodobi** z novimi barvami, ko je polje Indeks označeno

{% hint style="success" %}
**Najboljša praksa**: Za indekse vegetacije, kot je NDVI, je preliv Red-Rumena-Green najbolj intuitiven, saj se ujema z naravnimi barvnimi povezavami (zelena = zdrava, rumena = zmerna, rdeča = pod stresom).
{% endhint %}

### Nastavljanje barvnih razredov

**Nadzor razredov**določa, koliko ločenih barvnih stopenj se pojavi v vašem gradientu:**Možnosti števila razredov:*** **2–5 razredov**: Zelo široke kategorije, ločene cone
* **6–10 razredov**: Uravnoteženo, primerno za klasifikacijo
* **11–20 razredov**: Gladki gradienti, neprekinjen videz
* **20+ razredov**: skoraj neprekinjen, največja gladkost**Kako prilagoditi:**

1. V plošči LUT poiščite**kvadratke barvnih vzorcev pod vrstico prehoda**

2. Število razredov prilagodite z dodajanjem s tipko +
3. Število razredov odstranite z dvojnim klikom na barvni vzorec
4. Prehod se **v realnem času** posodobi na sliki**Vpliv na vizualizacijo:*** **Manj razredov** (3–5): Ustvari ločene cone, poenostavljeno razvrščanje, lažje razlikovanje kategorij
* **Srednje število razredov** (6–10): Uravnotežen pristop, primeren za večino uporab
* **Več razredov** (15–20): Gladki prehodi, podrobne razlike, fotografski videz**Kdaj uporabiti:*** **Malo razredov (3–5)**: Predstavitvene diapozitive, klasifikacijske karte, preprosta poročila
* **Srednje število razredov (6–10)**: Splošna analiza, uravnoteženi podrobnosti, standardna poročila
* **Veliko razredov (15–20)**: Znanstvena analiza, podrobna preiskava, izhodi v kakovosti za objavo

### Natančno nastavljanje razponov vrednosti

**Nadzorni elementi razpona vrednosti**določajo, katere indeksne vrednosti se preslikajo v katere barve v vašem prelivu:**Nadzorni elementi razpona v panelu LUT:*** **Minimalna vrednost**: Spodnja meja barvne lestvice
* **Maksimalna vrednost**: Zgornja meja barvne lestvice
* **Vmesne vrednosti**: Samodejno porazdeljene med min in max (glede na število razredov)

#### Nastavitev minimalnih/maksimalnih vrednosti

**Za nastavitev vrednostnih razponov:**

1. V plošči LUT poiščite vnosna polja**Minimalna vrednost**in**Maksimalna vrednost**

2. Kliknite polje**Minimalna vrednost**

3. Vnesite želeno minimalno vrednost (npr. `0.2`)
4. Pritisnite **Enter** ali kliknite zunaj polja
5. Ponovite za polje **Največja vrednost** (npr. `0.9`)
6. Vizualizacija se **takoj posodobi**{% hint style="info" %}**Samodejno prilagajanje**: Ko prvič uporabite LUT, Chloros samodejno nastavi najmanjšo/največjo vrednost na dejansko območje podatkov v sliki. To območje lahko nato zožite, da se osredotočite na določena območja vrednosti, ki vas zanimajo.
{% endhint %}

**Primer prilagoditev obsega NDVI:*** **Celoten obseg**: `-1.0` do `1.0` (prikaži vse možne vrednosti)
* **Osredotočeno na vegetacijo**: od `0.2` do `0.9` (izključi golo zemljo in vodo)
* **Samo zdrava vegetacija**: od `0.5` do `0.9` (poudari samo bujne rastline)
* **Zaznavanje stresa**: `0.2` do `0.5` (poudari problematična območja)
* **Prilagojeno območje**: Prilagodi na podlagi opazovanih vrednosti pikslov**Zakaj prilagajati območja?*** **Povečaj kontrast** na območju, ki te zanima
* **Izključite nepomembne vrednosti** (npr. vodne površine, gola tla)
* **Standardizirajte vizualizacijo** med več slikami ali datumi
* **Poudarite subtilne razlike** znotraj ozkega razpona vrednosti

### Izrezovanje vrednosti zunaj razpona

Ko vrednosti pikslov padejo zunaj vašega opredeljenega minimalnega/maksimalnega razpona, lahko z **načini izrezovanja** nadzorujete, kako se prikazujejo.

#### **Na voljo so naslednje možnosti načinov izrezovanja:**

#### 1. Najmanjša in največja vrednost

* Piksli **pod najmanjšo vrednostjo**→ prikaz z**prvo barvo** v barvnem prehodu (npr. rdeča)
* Piksli **nad največjo vrednostjo**→ prikaz z**zadnjo barvo** v barvnem prehodu (npr. zelena)
* **Primer uporabe**: poudarjanje skrajnosti, prikaz celotnega obsega podatkov z nasičenimi barvami na mejah
* **Primer**: vrednosti NDVI pod 0,2 so vse rdeče, vrednosti nad 0,9 pa so vse zelene

#### 2. Prosojno ozadje

* Piksli **zunaj obsega**postanejo**popolnoma prosojni*** Le piksli **znotraj obsega** prikazujejo barvni prehod
* **Primer uporabe**: GIS-prekrivni sloj, izolacija določenih obsegov vrednosti, poudarjanje le območij, ki so zanimiva
* **Primer**: Prikaži le vrednosti NDVI 0,4–0,7 v barvi, vse ostalo naj bo prosojno

{% hint style="warning" %}
**Omejitev prosojnosti**: Prosojni piksli se v pregledovalniku prikažejo v barvi ozadja. Pri izvozu med obdelavo se prosojnost ohrani v formatu PNG, vendar ne v formatu JPG.
{% endhint %}

#### 3. Ozadje indeksa

* Piksli **zunaj območja**se prikažejo v**sivi lestvici** (prikazujejo surove indeksne vrednosti)
* Piksli **znotraj območja**prikazujejo**barvni prehod*** **Primer uporabe**: Nežno poudarjanje, ohranjanje konteksta ob poudarjanju območij, ki so v središču zanimanja
* **Primer**: Barvno poudarjanje obremenjene vegetacije (NDVI 0,3–0,5), medtem ko se zdrava območja prikazujejo v sivi barvi

#### 4. Izvirno ozadje

* Piksli **zunaj območja**prikazujejo**izvirno multispektralno sliko*** Piksli **znotraj območja**prikazujejo**barvni prehod*** **Primer uporabe**: Najbolj intuitivno – združuje naravni kontekst slike z analitičnim barvnim prekrivom
* **Primer**: Ogled dejanskega videza polja/pridelka s prekrivom območij pod stresom, označenih z barvami

### Izbiranje pravega načina izreza

| Način izreza              | Najbolj primerno za                                   | Slog vizualizacije          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimalno in maksimalno**    | Prikaz vseh podatkov, znanstvena analiza     | Vsi piksli obarvani           |
| **Prosojno ozadje** | GIS prekrivni sloji, izolacija določenih območij    | Barva v območju, prazno zunaj |
| **Indeksno ozadje**       | Nežna poudaritev, ohranjanje konteksta podatkov  | Barva v območju, sivo zunaj  |
| **Izvirno ozadje**    | Poročila, predstavitve, intuitivna analiza | Barva v območju, fotografija zunaj |

### Ustvarjanje barv po meri v LUT

Za popoln nadzor nad vizualizacijo lahko ustvarite **barvne prehode po meri** z urejanjem posameznih barvnih točk.**Ustvarjanje prehoda po meri:**

1. V plošči LUT poiščite**pasico za predogled prehoda**

2. Poiščite**kvadratke z barvnimi vzorci** pod prehodom
3. **Kliknite barvno točko**, da jo izberete
4. Odpre se **izbirnik barv**

5. Izberite novo barvo z uporabo:
   * **Barvno kolo**: vizualna izbira barve
   * **Drsniki RGB/HSV**: natančen nadzor barve
   * **Vnos šestnajstiškega kode**: Natančna specifikacija barve (npr. `#FF0000` za rdečo)
6. Kliknite zunaj izbirnika barv, **da uporabite novo barvo**

7. Prehod se**takoj posodobi** na sliki**Dodajanje ali odstranjevanje barvnih točk:*** **Dodajanje barvne točke**: Kliknite ikono +, da na konec dodate novo barvno vzorčico
* **Odstranjevanje barvne točke**: Dvakrat kliknite barvni kvadrat, da odstranite vzorčico**Strategije prilagajanja:*** **Obratni barvni prehod**: Obratno vrstno red barv, da se pomen obrne (npr. zelena = nizka, rdeča = visoka)
* **Barve blagovne znamke**: Uskladite barvno paleto vaše organizacije za poročila
* **Primerno za barvno slepe**: Uporabite kombinacije oranžno-modre ali vijolično-rumene barve
* **Optimizacija tiskanja**: Izberite barve, ki delujejo tako pri barvnem kot pri sivem tiskanju
* **Več pragov**: Uporabite različne barve pri določenih vrednostnih pragovih za razvrščanje

{% hint style="info" %}
**Shranjevanje prilagojenih prehodov**: Prilagojene prehode je mogoče shraniti in ponovno uporabiti. Kliknite ikono za shranjevanje v plošči LUT, da shranite svoje prilagojene barvne sheme za prihodnjo uporabo.
{% endhint %}

***

## Interaktiven delovni tok

### Posodobitve v realnem času

Vse prilagoditve LUT v peskovniku **takoj in interaktivno** posodobijo sliko:

* **Preklopite sloj** → Slika se takoj spremeni
* **Izberite preliv** → Barve se posodobijo takoj
* **Prilagodite obseg vrednosti** → Kontrast se spremeni v realnem času
* **Spremenite razrede** → Gladkost preliva se posodobi takoj
* **Spremenite izrez** → Prikaz ozadja se spremeni takoj
* **Uredite barve** → Prilagojeni preliv se uporabi takoj**Gumb »Uporabi« ni potreben** – vse spremembe so v živo in interaktivne!

{% hint style="success" %}
**Povratne informacije v realnem času**: Takojšnje vizualne povratne informacije vam omogočajo hitro eksperimentiranje z različnimi nastavitvami, dokler ne najdete optimalne vizualizacije za vaše potrebe analize.
{% endhint %}

### Iterativni delovni tok izpopolnjevanja

**Tipični delovni tok optimizacije LUT:**

1.**Izberite indeksno plast** (npr. RAW (odbojnost))
2. **Uporabite indeks** – izberite filter kamere in indeksno formulo, povlecite barvne kroge na ustrezno mesto v indeksni formuli
3. **Uporabite LUT-gradient** – začnite s prednastavitvijo Red-Yellow-Green
4. **Preglejte vrednosti pikslov** – premikajte kurzor po sliki in si zabeležite razpone vrednosti
5. **Prilagodite min/max** – zožite razpon, da se osredotočite na vegetacijo (npr. 0,2 do 0,9)
6. **Izberite izrez** – poskusite z »Original Background« za kontekst
7. **Izboljšajte barve** – po potrebi prilagodite gradient za posebne poudarke
8. **Dokončajte nastavitve**– zabeležite nastavitve in jih kopirajte v nastavitve projekta za obdelavo izvoza

### Pregled vrednosti pikslov

Razumevanje dejanskih vrednosti pikslov je ključnega pomena za nastavitev učinkovitih razponov LUT:**Kako pregledati vrednosti:**

1. Vrednosti pikslov se prikažejo, ko je na sliki**označeno polje** Indeks ali pa sta označena oba polja, Indeks in LUT.
2. **Premaknite kurzor** nad različne dele slike
3. **Opazujte vrednosti pikslov**, ki se prikažejo v legendi, ko se z miško približate
4. Povečajte sliko, da vidite posamezne piksle, označene s plavajočo vrednostjo
5. **Zapišite si** razpone vrednosti za različne značilnosti:
   * **Zdrava vegetacija**: npr. NDVI 0,55–0,85
   * **Stresirana vegetacija**: npr. NDVI 0,30–0,50
   * **Gola tla**: npr. NDVI 0,05–0,25
   * **Voda** (če je prisotna): npr. NDVI -0,05 do 0,10**Uporaba vrednosti pikslov za nastavitev obsega LUT:**Po pregledu vrednosti pikslov ustrezno prilagodite minimalno in maksimalno vrednost LUT:**Primer scenarija:*** **Opazovanje**: Vrednosti tal = 0,05–0,25, Stres = 0,25–0,50, Zdrav = 0,50–0,85
* **Cilj**: Vizualizirajte samo zdravje rastlin (izključite tla)
* **Nastavitve LUT**: Min = `0.25`, Max = `0.85`
* **Izrez**: »Originalno ozadje«, da vidite tla v naravni barvi
* **Rezultat**: Barvni prehod se nanaša le na vegetacijo, tla se prikazujejo kot izvirna slika

{% hint style="info" %}
**Dinamični razpon**: Različne kulture, letni časi in faze rasti bodo imele različne razpone vrednosti. Vedno preglejte vrednosti pikslov v vašem konkretnem nizu podatkov, preden nastavite razpone LUT.
{% endhint %}

***

## Prilagojeni indeksi (Chloros+)

### Ustvarjanje formul za prilagojene indekse

{% hint style="info" %}
**Kje ustvariti**: Prilagojene indekse lahko konfigurirate v**Nastavitvah projekta** pred obdelavo, pa tudi v stranskem meniju peskovnika v pregledovalniku slik.
{% endhint %}

**Ustvarjanje indeksa po meri:**

1.**Odprite nastavitve projekta** (pred obdelavo) ali stranski trak v peskovniku pregledovalnika slik
2. Preidite na **spustni meni Formule indeksa**

3. Poiščite možnost**»Po meri«** (morate biti prijavljeni z licenco Chloros+)
4. **Določite svojo formulo** z uporabo spremenljivk pasov:
   * Imena pasov: `NIR`, `Red`, `Green`, `Blue`, `RedEdge` itd.
   * Operatorji: `+`, `-`, `*`, `/`, `^` (eksponent)
   * Funkcije: `sqrt()`, `abs()` itd. (če je podprto)
   * Oklepaji: `()` za vrstni red operacij
5. **Poimenujte svoj indeks** (npr. »MyIndex« ali »CustomNDVI«)
6. **Shranite konfiguracijo**

**Primeri po meri izdelanih formul:**

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

{% hint style="warning" %}
**Preverjanje formule**: Prepričajte se, da vaša formula uporablja pasove, ki so na voljo v vaši kameri. Na primer, RedEdge je na voljo le na kamerah s filtrom RedEdge.
{% endhint %}

***

## Naslednji koraki

Sedaj, ko razumete Index/LUT Sandbox:

* **Uporabite pri obdelavi**: Uporabite odkrite nastavitve v [Nastavitvah projekta](../project-settings/project-settings.md)
* **Obdelava v seriji**: Uporabite optimizirane indekse za celotne podatkovne nize
* **Več informacij**: Preberite [Formule multispektralnih indeksov](../project-settings/multispectral-index-formulas.md)

Povezana dokumentacija:

* [**Sloji slik**](image-layers.md) – Upravljanje in vizualizacija slojev
* [**Odpiranje slike v polnem zaslonu**](opening-an-image-full-screen.md) – Osnove pregledovalnika slik
* [**Obdelava slik (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Celoten potek obdelave

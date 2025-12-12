# Indeks/LUT Sandbox

Indeks/LUT Sandbox je interaktivno delovno okolje znotraj Chloros Image Viewer, ki vam omogoča eksperimentiranje z izračuni multispektralnih indeksov in vizualizacijami barv v realnem času. To zmogljivo orodje vam pomaga testirati različne indekse, izpopolniti vrednostna območja in ustvariti vizualizacije, pripravljene za objavo, brez ponovne obdelave celotnega niza podatkov.

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

## Delo s peskovnikom indeksa/LUT

### Razumevanje vnaprej izračunanih indeksov

V Chloros se indeksi lahko uporabijo med obdelavo projekta. Da bi določili, katere nastavitve indeksa in LUT želite uporabiti za izvoz, je najlažje uporabiti peskovnik za pregledovanje slik.

Peskovnik vam omogoča:

* **Uporabite nove indekse in barvne prehode (LUT)** za vizualizacijo podatkov.
* **Prilagodite nastavitve vizualizacije** interaktivno.
* **Ogledate** že izračunane indeksne slike.
* **Pregledate** vrednosti pikslov na vseh ravneh povečave.

### Odpiranje peskovnika

Do peskovnika indeksa/LUT dostopate v **pregledovalniku slik** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Kliknite sliko v mreži slik v brskalniku datotek, da se odpre v **pregledovalniku slik** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> .
2. Kliknite **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> , da odprete levi stranski meni, če še ni odprt

### Izbiranje slike za uporabo indeksa/LUT

Za delo z indeksom v pregledovalniku slik <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. **Odprite sliko** iz glavne mreže slik s klikom nanjo.
2. Odpre se zavihka **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> .
3. Kliknite **spustni meni Sloj** (v zgornjem desnem kotu pregledovalnika).
4. Iz spustnega menija izberite sloj:
   * RAW (odbojnost)

### Uporaba indeksa na sliki

Ko je slika v polnem zaslonu in je odprt stranski meni **Pregledovalnik slik** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> odprt:

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

**Pregledna tabela (LUT)** preslika numerične indeksne vrednosti v barve za vizualizacijo:

* **Vnos**: indeksna vrednost piksla (npr. NDVI 0,65)
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
5. V **pregledovalniku slik** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> , da uporabite LUT.

### Izbiranje barvnega prehoda

**Izbiranje prehoda:**

1. V plošči LUT poiščite **barvno barvno lestvico**.
2. Premaknite miško nad njo, da si ogledate razpoložljive prednastavitve barvne lestvice.
3. Izberite želeno barvno lestvico.
4. Slika se **takoj posodobi** z novimi barvami, ko je potrjeno polje Indeks.

{% namig style=&quot;success&quot; %}
**Najboljša praksa**: Za vegetacijske indekse, kot je NDVI, je prehod Red-Yellow-Green najbolj intuitiven, ker je v skladu z naravnimi barvnimi povezavami (zelena = zdrava, rumena = zmerna, rdeča = obremenjena).
{% endhint %}

### Prilagajanje barvnih razredov

**Nadzor razredov** določa, koliko ločenih barvnih stopenj se prikaže v vašem prehodu:

**Možnosti števila razredov:**

* **2–5 razredov**: zelo široke kategorije, ločene cone
* **6–10 razredov**: uravnoteženo, primerno za razvrščanje
* **11–20 razredov**: gladki prehodi, neprekinjen videz
* **20+ razredov**: skoraj neprekinjeno, največja gladkost

**Kako prilagoditi:**

1. V plošči LUT poiščite **barvne kvadratke pod vrstico prehoda**
2. Število razredov prilagodite z dodajanjem s tipko +.
3. Število razredov odstranite z dvojnim klikom na barvni vzorec.
4. Prehod se **v realnem času** posodobi na sliki.

**Učinek na vizualizacijo:**

* **Manj razredov** (3–5): ustvari ločene cone, poenostavljeno razvrščanje, lažje razlikovanje kategorij
* **Srednje število razredov** (6–10): uravnotežen pristop, primeren za večino uporab
* **Več razredov** (15–20): gladki prehodi, podrobne razlike, fotografski videz

**Kdaj uporabiti:**

* **Malo razredov (3–5)**: predstavitvene diapozitive, klasifikacijske karte, preprosta poročila
* **Srednje število razredov (6–10)**: splošna analiza, uravnoteženi podrobnosti, standardna poročila
* **Veliko razredov (15–20)**: znanstvena analiza, podrobna inšpekcija, izhodi v kakovosti za objavo

### Natančno nastavljanje vrednostnih razponov

**Nadzor vrednostnih razponov** določa, katere indeksne vrednosti se pripisujejo katerim barvam v vašem prelivu:

**Nadzor razponov v plošči LUT:**

* **Najmanjša vrednost**: spodnja meja barvne lestvice
* **Največja vrednost**: zgornja meja barvne lestvice
* **Vmesne vrednosti**: samodejno razporejene med najmanjšo in največjo vrednostjo (glede na število razredov)

#### Nastavljanje najmanjših/največjih vrednosti

**Nastavljanje vrednostnih razponov:**

1. V plošči LUT poiščite vnosna polja **Najmanjša vrednost** in **Največja vrednost**.
2. Kliknite polje **Najmanjša vrednost**.
3. Vnesite želeno minimalno vrednost (npr. `0.2`).
4. Pritisnite **Enter** ali kliknite zunaj polja.
5. Ponovite za polje **Max Value** (npr. `0.9`).
6. Vizualizacija se **takoj posodobi**.

{% namig style=&quot;info&quot; %}
**Samodejno prilagajanje**: Ko prvič uporabite LUT, Chloros samodejno nastavi najmanjšo/največjo vrednost na dejanski razpon podatkov v sliki. Nato lahko ta razpon zožite, da se osredotočite na določene razpone vrednosti, ki vas zanimajo.
{% endhint %}

**Primer NDVI prilagoditev obsega:**

* **Celoten obseg**: `-1.0` do `1.0` (prikaži vse možne vrednosti)
* **Osredotočeno na vegetacijo**: `0.2` do `0.9` (izključi golo zemljo in vodo)
* **Samo zdrava vegetacija**: `0.5` do `0.9` (poudari samo bujne rastline)
* **Zaznavanje stresa**: `0.2` do `0.5` (poudarite problematična območja)
* **Prilagojeno območje**: prilagodite na podlagi opazovanih vrednosti pikslov

**Zakaj prilagajati območja?**

* **Povečajte kontrast** v območju, ki vas zanima
* **Izključite nepomembne vrednosti** (npr. vodne površine, gola tla)
* **Standardizirajte vizualizacijo** za več slik ali datumov
* **Poudarite subtilne razlike** znotraj ozkega razpona vrednosti

### Izrezovanje vrednosti zunaj razpona

Ko vrednosti pikslov padejo zunaj vašega opredeljenega minimalnega/maksimalnega razpona, lahko nadzirate, kako se prikazujejo, z uporabo **načinov izrezovanja**.

#### **Na voljo so naslednje možnosti načinov izrezovanja:**

#### 1. Najmanjša in največja vrednost

* Piksli **pod najmanjšo vrednostjo** → prikaz z **prvo barvo** v barvnem prehodu (npr. rdeča)
* Piksli **nad največjo vrednostjo** → prikaz z **zadnjo barvo** v barvnem prehodu (npr. zelena)
* **Primer uporabe**: poudarjanje skrajnosti, prikaz celotnega obsega podatkov z nasičenimi barvami na mejah
* **Primer**: vrednosti NDVI pod 0,2 se prikažejo rdeče, vrednosti nad 0,9 pa zelene

#### 2. Prozorno ozadje

* Piksli **zunaj obsega** postanejo **popolnoma prosojni**
* Le piksli **znotraj obsega** prikazujejo barvni prehod
* **Primer uporabe**: GIS prekrivanje, izolacija določenih obsegov vrednosti, poudarjanje le zanimivih območij
* **Primer**: Prikaži le NDVI 0,4–0,7 v barvi, vse ostalo pa prosojno

{% hint style=&quot;warning&quot; %}
**Omejitev prozornosti**: Prozorni piksli se v pregledovalniku prikažejo kot barva ozadja. Pri izvozu med obdelavo se prozornost ohrani v formatu PNG, vendar ne v formatu JPG.
{% endhint %}

#### 3. Indeks ozadja

* Piksli **zunaj območja** se prikažejo v **sivi barvi** (prikazujejo surove indeksne vrednosti)
* Piksli **znotraj območja** prikazujejo **barvni prehod**
* **Primer uporabe**: Nežno poudarjanje, ohranjanje konteksta ob poudarjanju zanimivih območij
* **Primer**: Barvno poudarjena vegetacija (NDVI 0,3–0,5), zdrava območja pa so prikazana v sivi barvi

#### 4. Originalno ozadje

* Piksli **zunaj območja** prikazujejo **izvirno multispektralno sliko**
* Piksli **znotraj območja** prikazujejo **barvni prehod**
* **Primer uporabe**: Najbolj intuitiven – združuje naravni kontekst slike z analitičnim barvnim prekrivanjem
* **Primer**: Oglejte si dejanski videz polja/pridelka s prekrivanjem območij stresa, označenih z barvami

### Izbiranje pravega načina izrezovanja

| Način izrezovanja              | Najbolj primerno za                                   | Stil vizualizacije          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimalno in maksimalno**    | Prikaz vseh podatkov, znanstvena analiza     | Vsi piksli so obarvani           |
| **Prozorno ozadje** | GIS prekrivni sloji, izolacija določenih območij    | Barva na območju, prazno zunaj območja |
| **Indeksno ozadje**       | Nežno poudarjanje, ohranjanje konteksta podatkov  | Barva na območju, sivo zunaj območja  |
| **Originalno ozadje**    | Poročila, predstavitve, intuitivna analiza | Barva na območju, fotografija zunaj območja |

### Ustvarjanje prilagojenih barv LUT

Za popoln nadzor nad vizualizacijo lahko ustvarite **prilagojene barvne prehode** z urejanjem posameznih barvnih prehodov.

**Ustvarjanje prilagojenega prehoda:**

1. V plošči LUT poiščite **pregledno vrstico prehoda**.
2. Poiščite **barvne kvadratke** pod prehodom.
3. **Kliknite barvni prehod**, da ga izberete.
4. Odpre se **izbirnik barv**.
5. Izberite novo barvo z uporabo:
   * **Barvnega kolesa**: vizualna izbira barve.
   * **RGB/HSV drsnikov**: natančen nadzor barve.
   * **Vnos heksadecimalne kode**: natančna specifikacija barve (npr. `#FF0000` za rdečo).
6. Kliknite zunaj izbirnika barv **, da uporabite novo barvo**.
7. Prehod **se takoj posodobi** na sliki.

**Dodajanje ali odstranjevanje barvnih prehodov:**

* **Dodajanje prehoda**: kliknite ikono +, da dodate novo barvno paleto na konec.
* **Odstranjevanje prehoda**: dvokliknite barvni kvadrat, da odstranite barvno paleto.

**Strategije prilagajanja:**

* **Obratni prehod**: Obratite vrstni red barv, da obratite pomen (npr. zelena = nizka, rdeča = visoka)
* **Barve blagovne znamke**: Ujemite barvno paleto vaše organizacije za poročila
* **Primerno za barvno slepe**: Uporabite kombinacije oranžno-modre ali vijolično-rumene
* **Optimizacija tiskanja**: Izberite barve, ki delujejo tako pri barvnem kot pri sivinskem tiskanju
* **Več pragov**: uporabite različne barve za določene vrednosti pragov za razvrščanje

{% hint style=&quot;info&quot; %}
**Shranjevanje prilagojenih prehodov**: prilagojene prehode lahko shranite in ponovno uporabite. Kliknite ikono za shranjevanje v oknu LUT, da shranite prilagojene barvne sheme za prihodnjo uporabo.
{% endhint %}

***

## Interaktivni delovni tok

### Posodobitve v realnem času

Vse prilagoditve LUT v peskovniku posodobijo sliko **takoj in interaktivno**:

* **Preklopite sloj** → Slika se takoj spremeni
* **Izberite preliv** → Barve se posodobijo takoj
* **Prilagodite razpon vrednosti** → Kontrast se spremeni v realnem času
* **Spremenite razrede** → Gladkost preliva se posodobi takoj
* **Spremenite izrez** → Prikaz ozadja se spremeni takoj
* **Uredite barve** → Prilagojeni preliv se uporabi takoj

**Gumb »Uporabi« ni potreben** – vse spremembe so v živo in interaktivne!

{% hint style=&quot;success&quot; %}
**Takojšnja povratna informacija**: Takojšnja vizualna povratna informacija vam omogoča hitro preizkušanje različnih nastavitev, dokler ne najdete optimalne vizualizacije za vaše potrebe analize.
{% endhint %}

### Iterativni postopek izpopolnjevanja

**Tipični postopek optimizacije LUT:**

1. **Izberite indeksni sloj** (npr. RAW (odbojnost))
2. **Uporabite indeks** – izberite filter kamere in indeksno formulo, povlecite barvne kroge na ustrezno mesto v indeksni formuli
3. **Uporabite LUT-gradient** – začnite s prednastavitvijo Red-Yellow-Green
4. **Preglejte vrednosti pikslov** – premikajte kurzor in si zabeležite vrednostne razpone
5. **Prilagodite min/max** – zožite, da se osredotočite na vegetacijo (npr. 0,2 do 0,9)
6. **Izberite izrez** – poskusite z »Original Background« za kontekst
7. **Izboljšajte barve** – po potrebi prilagodite gradient za posebno poudarjanje
8. **Dokončajte nastavitve** – Dokumentirajte nastavitve in jih kopirajte v nastavitve projekta za izvozno obdelavo.

### Pregled vrednosti pikslov

Razumevanje dejanskih vrednosti pikslov je ključnega pomena za nastavitev učinkovitih razponov LUT:

**Kako pregledati vrednosti:**

1. Vrednosti pikslov se prikažejo, ko je na sliki **označeno** polje Index ali pa sta označeni obe polji Index in LUT.
2. **Premaknite kurzor** po različnih delih slike.
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

* **Opazovanje**: Vrednosti tal = 0,05–0,25, Stres = 0,25–0,50, Zdravje = 0,50–0,85
* **Cilj**: Vizualizirajte samo zdravje rastlin (izključite tla)
* **Nastavitve LUT**: Min = `0.25`, Max = `0.85`
* **Obrezovanje**: »Originalno ozadje«, da vidite tla v naravni barvi
* **Rezultat**: Barvni prehod se nanaša samo na vegetacijo, tla se prikažejo kot izvirna slika

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

1. **Odprite Nastavitve projekta** (pred obdelavo) ali stranski trak Image Viewer sandbox.
2. Prejdite na **spustni meni Formula indeksa**.
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
**Preverjanje formule**: Preverite, ali formula uporablja pasove, ki so na voljo v vaši kameri. Na primer, RedEdge je na voljo samo v kamerah s filtrom RedEdge.
{% endhint %}

***

## Naslednji koraki

Zdaj, ko razumete indeks/LUT peskovnik:

* **Uporabite v obdelavi**: uporabite odkrita nastavitev v [Nastavitve projekta](../project-settings/project-settings.md)
* **Skupinska obdelava**: uporabite optimizirane indekse za celotne podatkovne nize
* **Več informacij**: preberite [Formule za multispektralne indekse](../project-settings/multispectral-index-formulas.md)

Povezana dokumentacija:

* [**Sloji slike**](image-layers.md) - Upravljanje in vizualizacija plasti
* [**Odpiranje slike v polnem zaslonu**](opening-an-image-full-screen.md) - Osnove pregledovalnika slik
* [**Obdelava slik (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Celoten potek obdelave

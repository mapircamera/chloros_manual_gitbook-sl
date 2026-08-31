# Začetek obdelave

Ko ste uvozili slike, označili kalibracijske cilje in konfigurirali nastavitve projekta, ste pripravljeni za začetek obdelave. Ta stran vas vodi skozi zagon obdelovalnega procesa Chloros.

## Seznam za preverjanje pred obdelavo

Preden kliknete gumb »Začni«, preverite, ali je vse pripravljeno:

* [ ] **Datoteke uvožene** – Vse slike so prikazane v brskalniku datotek
* [ ] **Ciljne slike označene** – V stolpcu »Cilj« so označene kalibracijske slike (ali uvožen posnetek `.daq` za LATTICE)
* [ ] **Zaznani modeli kamer** – stolpec »Model kamere« prikazuje prave kamere
* [ ] **Nastavitve konfigurirane** – nastavitve projekta so pregledane in prilagojene
* [ ] **Izbrani indeksi** – Dodani so željeni multispektralni indeksi (če je potrebno)
* [ ] **Izbrana oblika izvoza** – Izhodna oblika, primerna za vaš delovni tok

{% hint style="info" %}
**Nasvet**: Pred obdelavo kliknite na nekaj slik v brskalniku datotek, da preverite, ali so se pravilno naložile.
{% endhint %}

***

## Začetek obdelave

### Poiščite gumb »Start«

Gumb »Začni/Predvajaj« se nahaja v zgornji vrstici Chloros:

* Položaj: na vrhu okna, na sredini
* Ikona: **gumb »Predvajaj/Začni«** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Stanje: gumb je omogočen (svetel), ko je pripravljen za obdelavo

### Kliknite za zagon

1. Kliknite **gumb Predvajaj/Začni** v zgornjem naslovnem pasu
2. Obdelava se začne takoj
3. Med obdelavo se gumb spremeni v gumb **Stop**

4. Napredovalna vrstica se posodablja in prikazuje stanje obdelave

{% hint style="success" %}
**Obdelava se je začela**: Ko kliknete, Chloros samodejno opravi vse korake obdelave – zaznavanje cilja, debayering, kalibracijo, izračun indeksa in izvoz. Samodejno zazna, ali je vaš projekt Survey3, LATTICE ali mešanica, ter za vsako kamero uporabi ustrezen potek obdelave.
{% endhint %}

***

## Razumevanje načinov obdelave

Chloros deluje v dveh različnih načinih obdelave, odvisno od vaše licence:

### Brezplačni način (zaporedna obdelava)

**Na voljo vsem uporabnikom**

**Kako deluje:**

* Slike obdeluje po eno naenkrat, zaporedno
* Delovanje z enim niti
* Manjša poraba pomnilnika

**Napredovalna vrstica prikazuje 2 stopnji:**

1.**Zaznavanje ciljev** – iskanje kalibracijskih ciljev
2. **Obdelava** – uporaba kalibracije in izvoz slik**Čas obdelave:**

* Precej počasnejši od vzporednega načina Chloros+
* Primeren za majhne do srednje velike nize podatkov (&lt; 200 slik)

### Način Chloros+ (Vzporedna obdelava)

**Zahteva licenco Chloros+**

**Kako deluje:**

* Hkrati obdeluje več slik z uporabo [4-nitnega obdelovalnega potoka](../processing-architecture/processing-pipeline.md)
* [Dinamična prilagoditev računalniške zmogljivosti](../processing-architecture/dynamic-compute-adaptation.md) ob zagonu samodejno izbere optimalno strategijo za vašo strojno opremo
* Pospeševanje z GPU (CUDA) z grafičnimi karticami NVIDIA (namiznimi in Jetson)
* **Število delavcev se prilagaja strojni opremi**: strategije za GPU izvajajo**1–4 sočasne delavce** (prilagojeno glede na VRAM — Jetson z manj pomnilnika izvaja 1, namizni GPU z 12 GB+ pa do 4); sistemi, ki uporabljajo samo CPU, izvajajo enega delavca na fizično jedro, minus enega**Indikator napredka prikazuje 4 stopnje** (ki ustrezajo 4 nitim v poti):

1. **Zaznavanje** (nit 1) – iskanje kalibracijskih ciljev
2. **Analiza** (nit 2) – Preučevanje metapodatkov slike in izračun kalibracije
3. **Kalibracija** (nit 3) – Odstranjevanje bayerjevega vzorca, popravljanje vinjetiranja, kalibracija, izračun indeksa
4. **Izvoz** (nit 4) – Shranjevanje obdelanih slik in indeksov**Uporaba kazalnika napredka:*** **Premaknite miško** nad kazalnik, da se prikaže podrobni spustni meni s 4 stopnjami
* **Kliknite** na kazalnik napredka, da se spustni meni fiksira na mestu
* **Kliknite še enkrat**, da se fiksiranje razveljavi in se meni skrije**Čas obdelave:**

* Znatno hitrejši od brezplačnega načina
* Pospeševanje s GPU dodatno izboljša hitrost

{% hint style="info" %}
**Chloros+ Hitrost**: Vzporedna obdelava je pri velikih naborih podatkov lahko 5- do 10-krat hitrejša od zaporednega načina. Projekt s 500 slikami, ki v brezplačnem načinu traja 2 uri, se z Chloros+ lahko zaključi v 15–20 minutah.
{% endhint %}

***

## Kaj se dogaja med obdelavo

### Faza 1: Prepoznavanje ciljev

**Kaj počne Chloros:**

* Pregleda slike, ki ste jih označili v stolpcu »Cilj« (vse slike, če ni označena nobena)
* Prepozna kalibracijske plošče v vsakem cilju
* Iz ciljnih plošč izvleče vrednosti odbojnosti
* Zabeleži časovne oznake ciljev za načrtovanje kalibracije

**Trajanje:** 1–30 sekund (z označenimi cilji), 5–30+ minut (neoznačeni)

### Faza 2: Debayering (pretvorba RAW)

**Kaj naredi Chloros:**

* Pretvori surove podatke v vzorcu Bayer v polne 3-kanalne slike (mono moduli LATTICE ostanejo enopasovni – pri njih se odstranjevanje Bayerjevega vzorca preskoči, v dnevniku pa se zabeleži opomba)
* Uporabi izbrani algoritem demosaicinga
* Ohranja najvišjo kakovost slike in podrobnosti

**Trajanje:** Odvisno od števila slik ter hitrosti CPU/GPU

### Faza 3: Kalibracija

**Kaj počne Chloros:*** **Korekcija vinjetiranja**: Odstrani potemnitev objektiva na robovih
* **Kalibracija odbojnosti**: Normalizira z uporabo ciljnih vrednosti odbojnosti in/ali podatkov DAQ o padajočem sevanju
* Uporabi korekcije na vseh pasovih/kanalih
* Uporabi ustrezno kalibracijsko referenco za vsako sliko na podlagi časovnega žiga

**Trajanje:** Večina časa obdelave

### Faza 4: Izračun indeksov

**Kaj počne Chloros:**

* Izračuna konfigurirane multispektralne indekse (NDVI, NDRE itd.)
* Na kalibrirane slike uporabi matematične operacije na pasovih
* Ustvari indeksne slike za vsak izbran indeks

**Trajanje:** Nekaj sekund na sliko

### Faza 5: Izvoz

**Kaj počne Chloros:**

* Shrani obdelane slike v izbranem formatu
* **LATTICE fan-out**: vsak surovi LATTICE-okvir se izvozi kot vsak omogočen produkt v enem prehodu — debayering, predogled, sevanje (vedno float32), odbojnost
* Zapiše datoteke v izhodno drevesno strukturo projekta: `<project>/<camera>/<format>/<Product>_Images/`
* **Ohrani izvirno ime datoteke** — izdelek je opredeljen s mapo, pripona se ne doda**Trajanje:** Odvisno od formata izvoza in velikosti datoteke***

## Delovanje obdelave

### Avtomatski obdelovalni tok

Ko se procesni tok enkrat zažene, teče samodejno:

* Ni potrebno posredovanje uporabnika
* Vsi nastavljeni koraki se izvajajo zaporedno
* Napredek se prikazuje v realnem času
* Izvožene datoteke se zapisujejo na disk takoj po zaključku – končne izhodne datoteke lahko odprete, medtem ko se proces še nadaljuje

### Poraba računalniških virov med obdelavo

**Prosti način:**

* Razmeroma nizka poraba procesorske moči (enonitni)
* Računalnik ostaja odziven za druge naloge
* Brez skrbi lahko zmanjšate okno Chloros in delate v drugih programih

**Chloros+ vzporedni način:**

* Visoka poraba procesorske moči v delovni skupini strategije
* Z grafično pospešitvijo: Visoka poraba grafične kartice
* Računalnik je med obdelavo morda manj odziven
* Izogibajte se zagonu drugih nalog, ki močno obremenjujejo procesor

{% hint style="warning" %}
**Nasvet za zmogljivost**: Za najboljšo zmogljivost Chloros+ zaprite druge aplikacije in omogočite Chloros, da uporabi vse sistemske vire.
{% endhint %}

### Obdelave ni mogoče začasno ustaviti (vendar je ustavitev čista)

* Ko se obdelava enkrat začne, je ni mogoče začasno ustaviti in kasneje nadaljevati
* S klikom na **Stop** se izvajanje čisto ustavi že ob prvem kliku
* Izdelki, ki so bili izvoženi pred ustavitvijo, ostanejo na disku
* Ustavljeno izvajanje natančno poroča, kaj je bilo zaključeno (glejte vrstice `[RUN-SUMMARY]` v dnevniku)
* Novi izvedbi se poteka začne od začetka

**Nasvet za načrtovanje:** Pri zelo velikih projektih razmislite o obdelavi v serijah ali uporabi CLI za boljši nadzor.***

## Spremljanje obdelave

Med potekom obdelave lahko:

* **Spremljate vrstico napredka** – ogledate si skupni odstotek dokončanja
* **Pregledati trenutno fazo** – zaznavanje, analiziranje, kalibriranje ali izvoz
* **Preveriti zavihek dnevnika** – ogled podrobnih sporočil in opozoril o obdelavi
* **Predogledati dokončane slike** – datoteke za izvoz se med obdelavo pojavijo na disku

Za podrobne informacije o spremljanju glejte [Spremljanje obdelave](monitoring-the-processing.md).

***

## Zaustavitev obdelave

Če morate zaustaviti obdelavo:

### Kako zaustaviti

1. Poiščite **gumb »Stop«** (med obdelavo nadomesti gumb »Začni«)
2. Kliknite ga enkrat – na vrstici se prikaže **»Ustavljanje...«**, medtem ko se zaključi obdelava trenutne slike
3. Obdelava se konča v dokončnem ustavljenem stanju, v dnevniku pa se natisne natančen zapis `[RUN-SUMMARY]` o tem, kar je bilo zaključeno

### Kdaj ustaviti

**Utemeljene razloge za ustavitev:**

* Ugotovili ste, da so bile uporabljene napačne nastavitve
* Pozabili ste označiti ciljne slike
* Uvožene so bile napačne slike
* Sistem deluje prepočasi ali se ne odziva

**Po ustavitvi:**

* Izdelki, izvoženi pred zaustavitvijo, ostanejo na disku
* Preglejte in odpravite morebitne težave, po potrebi prilagodite nastavitve
* Ponovno zaženite obdelavo — izvajanje se začne od začetka

***

## Ocene časa obdelave

Dejanski čas obdelave se močno razlikuje glede na:

* Število slik
* Ločljivost slik
* Vhodni format RAW ali JPG
* Načinu obdelave (Free ali Chloros+)
* Hitrosti procesorja in številu jeder
* Razpoložljivosti grafične kartice (samo Chloros+)
* Številu indeksov, ki jih je treba izračunati
* Številu omogočenih izvoznih izdelkov (LATTICE)

### Grobe ocene (Chloros+, slike z ločljivostjo 12 MP, sodoben procesor)

| Število slik | Brezplačni način | Chloros+ (procesor) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 slik   | 15–20 min | 5–8 min        | 3–5 min        |
| 100 slik  | 30–40 min | 10–15 min      | 5–8 min        |
| 200 slik  | 1–1,5 ure | 20–30 min      | 10–15 min      |
| 500 slik  | 2–3 ure   | 45–60 min      | 20–30 min      |
| 1000 slik | 4–6 ur   | 1,5–2 uri      | 40–60 min      |

{% hint style="info" %}
**Prvi zagon**: Začetna obdelava lahko traja dlje, saj Chloros ustvarja predpomnilnike in profile. Nadaljnja obdelava podobnih naborov podatkov bo hitrejša.
{% endhint %}

***

## Pogoste težave ob zagonu

### Gumb za zagon je onemogočen (siv)

**Možni vzroki:**

* Ni uvoženih slik
* Ozadje ni v celoti zagnano
* Prejšnja obdelava še vedno poteka
* Projekt ni v celoti naložen

**Rešitve:**

1. Počakajte, da se ozadje v celoti inicializira (preverite ikono v glavnem meniju)
2. Preverite, ali so slike uvožene v brskalniku datotek
3. Če gumb ostane onemogočen, ponovno zaženite Chloros
4. Preverite dnevnik odpravljanja napak za sporočila o napakah

### Obdelava se začne, nato pa takoj propade

**Možni vzroki:**

* V projektu ni veljavnih slik
* Poškodovane slikovne datoteke
* Premajhen prostor na disku
* Premalo pomnilnika (RAM)

**Rešitve:**

1. Preverite dnevnik odpravljanja napak <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> za sporočila o napakah
2. Preverite razpoložljiv prostor na disku
3. Poskusite obdelati manjši podsklop slik
4. Preverite, ali slike niso poškodovane

### Izvedba se zaključi, vendar ne zapiše nobene slike

Izvedba, ki je zahtevala slikovne izdelke, vendar ni zapisala nobenega, se obravnava kot **neuspeh, ne kot uspeh** — Chloros to glasno sporoča:

* Dnevnik grafičnega vmesnika izpiše napotke `[RUN-SUMMARY]`, ki navajajo verjetni vzrok – slike niso bile uvožene, cilj ni bil zaznan ali so bili vsi zahtevani izdelki preskočeni kot neuporabni (npr. zahteva po sevanju/odbojnosti iz kamer, ki podpirajo le RGB)
* Ekvivalent CLI (`chloros-cli process`) izpiše `Processing finished but wrote no image products.` in **se zaključi z vrednostjo, ki ni nič**, tako da ga skripti lahko zaznajo
* Namerno izvajanje, ki zajema le metapodatke (vsi izvozni izdelki onemogočeni, brez indeksov), se še vedno šteje za uspešno

Za celotno semantiko glej [Referenco za CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails).

### Opozorilo »Ni zaznanih ciljev«

**Možni vzroki:**

* Pozabili ste označiti ciljne slike
* Ciljne slike ne vsebujejo vidnih ciljev
* Nastavitve za zaznavanje ciljev so preveč stroge

**Rešitve:**

1. Preberite [Izbiranje ciljnih slik](choosing-target-images.md)
2. Označite ustrezne slike v stolpcu »Cilj«
3. Preverite, ali so cilji vidni na označenih slikah
4. Po potrebi prilagodite nastavitve zaznavanja ciljev

***

## Nasveti za uspešno obdelavo

### Pred začetkom

1. **Najprej opravite preskus z majhnim podnaborom** – obdelajte 10–20 slik, da preverite nastavitve
2. **Preverite razpoložljivi prostor na disku** – zagotovite 2–3-kratno prosto prostora glede na velikost podatkovnega niza (več, če so omogočeni vsi izdelki LATTICE)
3. **Zaprite nepotrebne aplikacije** – sprostite sistemske vire
4. **Preverite slike ciljev** – oglejte si predogled označenih ciljev, da zagotovite kakovost
5. **Shranite projekt** – projekt se samodejno shrani, vendar je priporočljivo, da ga shranite ročno

### Med obdelavo

1. **Izogibajte se mirovanju sistema** – Onemogočite načine varčevanja z energijo
2. **Chloros naj ostane v ospredju** – Ali pa naj bo vsaj viden na opravilni vrstici
3. **Občasno spremljajte napredek** – Preverite, ali so prisotna opozorila ali napake
4. **Ne zagnajte drugih zahtevnih programov** – še posebej v vzporednem načinu Chloros+

### Chloros+ Pospeševanje s GPU

Če uporabljate pospeševanje z grafičnim procesorjem NVIDIA:

1. Posodobite gonilnike NVIDIA na najnovejšo različico
2. Preverite, ali ima grafična kartica 4 GB ali več VRAM (7 GB ali več za sočasno odstranjevanje pikslov s podporo za teksture)
3. Zaprite aplikacije, ki intenzivno obremenjujejo grafično kartico (igre, urejanje videov)
4. Spremljajte temperaturo grafične kartice (poskrbite za ustrezno hlajenje)

***

## Naslednji koraki

Ko se obdelava začne:

1. **Spremljajte napredek** – glejte [Spremljanje obdelave](monitoring-the-processing.md)
2. **Počakajte na zaključek** – obdelava poteka samodejno
3. **Preglejte rezultate** – glejte [Zaključek obdelave](finishing-the-processing.md)

Za informacije o tem, kaj storiti med obdelavo, glejte [Spremljanje obdelave](monitoring-the-processing.md).

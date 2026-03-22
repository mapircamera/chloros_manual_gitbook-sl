# Začetek obdelave

Ko ste uvozili slike, označili kalibracijske tarče in nastavili projektne nastavitve, ste pripravljeni za začetek obdelave. Ta stran vas vodi skozi zagon obdelovalnega procesa Chloros.

## Seznam za preverjanje pred obdelavo

Preden kliknete gumb Začni, preverite, ali je vse pripravljeno:

* [ ] **Datoteke uvožene** – Vse slike so prikazane v brskalniku datotek
* [ ] **Ciljne slike označene** – Stolpec Cilj je označen za kalibracijske slike
* [ ] **Modeli kamer zaznani** – Stolpec Model kamere prikazuje prave kamere
* [ ] **Nastavitve konfigurirane** – Nastavitve projekta so pregledane in prilagojene
* [ ] **Izbrani indeksi** – Dodani so željeni multispektralni indeksi (če je potrebno)
* [ ] **Izbrani izvozni format** – Izvozni format, primeren za vaš delovni tok

{% hint style="info" %}
**Nasvet**: Pred obdelavo kliknite na nekaj slik v brskalniku datotek, da preverite, ali so se pravilno naložile.
{% endhint %}

***

## Začetek obdelave

### Poiščite gumb Začni

Gumb Začni/Predvajaj se nahaja v zgornji vrstici Chloros:

* Položaj: Zgornji srednji del okna
* Ikona: **Gumb Predvajaj/Začni** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
* Stanje: Gumb je omogočen (svetel), ko je pripravljen za obdelavo

### Kliknite za zagon

1. Kliknite **gumb Predvajaj/Začni** v zgornjem naslovnem pasu
2. Obdelava se začne takoj
3. Gumb postane onemogočen (siv) med obdelavo
4. Napredovalna vrstica se posodablja in prikazuje stanje obdelave

{% hint style="success" %}
**Obdelava se je začela**: Ko kliknete, Chloros samodejno opravi vse korake obdelave – zaznavanje cilja, debayering, kalibracijo, izračun indeksa in izvoz.
{% endhint %}

***

## Razumevanje načinov obdelave

Chloros deluje v dveh različnih načinih obdelave, odvisno od vaše licence:

### Brezplačni način (zaporedna obdelava)

**Na voljo vsem uporabnikom**

**Kako deluje:**

* Slike obdeluje po eno naenkrat, zaporedno
* Enonitno delovanje
* Manjša poraba pomnilnika

**Napredovalna vrstica prikazuje 2 fazi:**

1.**Zaznavanje ciljev** – iskanje kalibracijskih ciljev
2. **Obdelava** – uporaba kalibracije in izvoz slik**Čas obdelave:**

* Precej počasnejši od vzporednega načina Chloros+
* Primeren za majhne do srednje velike nize podatkov (&lt; 200 slik)

### Način Chloros+ (vzporedna obdelava)

**Zahteva licenco Chloros+**

**Kako deluje:**

* Hkrati obdeluje več slik z uporabo [4-nitnega obdelovalnega potoka](../processing-architecture/processing-pipeline.md)
* [Dinamična prilagoditev računalniške moči](../processing-architecture/dynamic-compute-adaptation.md) samodejno izbere optimalno strategijo za vašo strojno opremo
* Pospeševanje z grafičnimi karticami NVIDIA (namizni računalniki in Jetson)
* Prilagodljivost od Jetson Nano (1 delavec) do namiznega računalnika z grafično kartico 12 GB+ (3–4 delavci)

**Napredovalna vrstica prikazuje 4 stopnje** (ki ustrezajo 4 niti v poti):

1. **Zaznavanje** (Nit 1) – Iskanje ciljev kalibracije
2. **Analiziranje** (Nit 2) – Preučevanje metapodatkov slike in izračun kalibracije
3. **Kalibriranje** (Nit 3) – GPU debayering, popravek vinjete, izračun indeksa
4. **Izvoz** (nit 4) – Shranjevanje obdelanih slik in indeksov**Interakcija z vrstico napredka:*** **Premaknite miško** nad vrstico, da se prikaže podrobni spustni meni s 4 stopnjami
* **Kliknite** na vrstico napredka, da zamrznete spustni meni na mestu
* **Kliknite še enkrat**, da odmrznete in skrijete meni**Čas obdelave:**

* Precej hitreje kot v brezplačnem načinu
* Prilagaja se številu jeder CPU
* Pospeševanje s GPU dodatno izboljša hitrost

{% hint style="info" %}
**Chloros+ Hitrost**: Vzporedna obdelava je pri velikih naborih podatkov lahko 5- do 10-krat hitrejša od zaporednega načina. Projekt s 500 slikami, ki v brezplačnem načinu traja 2 uri, se z Chloros+ lahko zaključi v 15–20 minutah.
{% endhint %}

***

## Kaj se dogaja med obdelavo

### Faza 1: Odkrivanje ciljev

**Kaj počne Chloros:**

* Pregleda označene ciljne slike (ali vse slike, če nobena ni označena)
* Prepozna 4 kalibracijske plošče v vsakem cilju
* Izvleče vrednosti odbojnosti iz ciljnih plošč
* Zapiše časovne oznake ciljev za načrtovanje kalibracije

**Trajanje:** 1–30 sekund (z označenimi cilji), 5–30+ minut (neoznačeni)

### Faza 2: Debayering (pretvorba RAW)

**Kaj počne Chloros:**

* Pretvori podatke RAW v vzorcu Bayer v polne slike RGB
* Uporabi visokokakovosten algoritem demosaicinga
* Ohrani najvišjo kakovost slike in podrobnosti

**Trajanje:** Odvisno od števila slik in hitrosti procesorja

### Faza 3: Kalibracija

**Kaj počne Chloros:*** **Popravek vinjetiranja**: Odstrani potemnitev objektiva na robovih
* **Kalibracija odbojnosti**: Normalizira z uporabo ciljnih vrednosti odbojnosti
* Uporabi popravke na vseh pasovih/kanalih
* Uporabi ustrezen kalibracijski cilj za vsako sliko na podlagi časovnega žiga

**Trajanje:** Večina časa obdelave

### Faza 4: Izračun indeksa

**Kaj počne Chloros:**

* Izračuna konfigurirane multispektralne indekse (NDVI, NDRE itd.)
* Uporabi matematične operacije na pasovih za kalibrirane slike
* Ustvari indeksne slike za vsak izbrani indeks

**Trajanje:** Nekaj sekund na sliko

### Faza 5: Izvoz

**Kaj počne Chloros:**

* Shrani kalibrirane slike v izbranem formatu
* Izvozi indeksne slike z nastavljenimi barvami LUT
* Zapiše datoteke v podmapah modelov kamer
* Ohrani originalna imena datotek s končnicami

**Trajanje:** Odvisno od formata izvoza in velikosti datoteke***

## Delovanje obdelave

### Avtomatski potek obdelave

Ko se zagnati, celoten potek teče avtomatsko:

* Ni potrebno posredovanje uporabnika
* Vsi nastavljeni koraki se izvajajo zaporedno
* Napredek se prikazuje v realnem času

### Poraba računalnika med obdelavo

**Prosti način:**

* Razmeroma nizka poraba CPU (enonitni)
* Računalnik ostane odziven za druge naloge
* Varno je zmanjšati Chloros in delati v drugih aplikacijah

**Chloros+ Vzporedni način:**

* Visoka poraba CPU (večnitni, do 16 jeder)
* Z GPU pospešitvijo: Visoka poraba GPU
* Računalnik je med obdelavo morda manj odziven
* Izogibajte se zagonu drugih nalog, ki intenzivno obremenjujejo CPU

{% hint style="warning" %}
**Nasvet za zmogljivost**: Za najboljšo zmogljivost Chloros+ zaprite druge aplikacije in pustite, da Chloros uporabi vse sistemske vire.
{% endhint %}

### Obdelave ni mogoče zaustaviti

**Pomembne omejitve:**

* Ko se obdelava začne, je ni mogoče zaustaviti
* Obdelavo lahko prekinete, vendar se napredek izgubi
* Delni rezultati se ne shranijo
* V primeru prekinitve je treba začeti znova od začetka

**Nasvet za načrtovanje:** Pri zelo velikih projektih razmislite o obdelavi v serijah ali uporabi CLI za boljši nadzor.***

## Spremljanje obdelave

Med potekom obdelave lahko:

* **Spremljate vrstico napredka** – Ogledate si skupni odstotek dokončanja
* **Ogledate trenutno fazo** – Odkrivanje, analiza, kalibracija ali izvoz
* **Preverite zavihek z dnevnikom** – oglejte si podrobna obvestila in opozorila o obdelavi
* **Predogled zaključenih slik** – med obdelavo se lahko prikažejo nekatere izvozne datoteke

Podrobne informacije o spremljanju najdete v [Spremljanje obdelave](monitoring-the-processing.md).

***

## Prekinitev obdelave

Če morate ustaviti obdelavo:

### Kako prekiniti

1. Poiščite **gumb Stop/Prekliči** (med obdelavo nadomesti gumb Start)
2. Kliknite gumb Stop
3. Obdelava se takoj ustavi
4. Delni rezultati se zavržejo

### Kdaj preklicati

**Utemeljene razloge za preklic:**

* Ugotovili ste, da so bile uporabljene napačne nastavitve
* Pozabili ste označiti ciljne slike
* Uvožene so bile napačne slike
* Sistem deluje prepočasi ali se ne odziva

**Po preklicu:**

* Preglejte in odpravite morebitne težave
* Po potrebi prilagodite nastavitve
* Obdelavo ponovno začnite od začetka
* Za najboljšo izkušnjo popolnoma zaprite Chloros in ponovno zaženite

{% hint style="warning" %}
**Brez delnih rezultatov**: Preklic zavrže ves napredek. Chloros ne shrani delno obdelanih slik.
{% endhint %}

***

## Ocene časa obdelave

Dejanski čas obdelave se močno razlikuje glede na:

* Število slik
* Ločljivost slik
* Vhodni format RAW ali JPG
* Način obdelave (Free proti Chloros+)
* Hitrost procesorja in število jeder
* Razpoložljivost grafične kartice (samo Chloros+)
* Število indeksov za izračun
* Zapletenost izvoznega formata

### Grobe ocene (Chloros+, slike 12 MP, sodoben procesor)

| Število slik | Brezplačni način | Chloros+ (CPU) | Chloros+ (GPU) |
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
* Backend ni v celoti zagnan
* Prejšnja obdelava še vedno poteka
* Projekt ni v celoti naložen

**Rešitve:**

1. Počakajte, da se backend v celoti inicializira (preverite ikono v glavnem meniju)
2. Preverite, ali so slike uvožene v brskalniku datotek
3. Ponovno zaženite Chloros, če gumb ostane onemogočen
4. Preverite dnevnik razhroščevanja za sporočila o napakah

### Obdelava se začne, nato pa takoj odpove

**Možni vzroki:**

* V projektu ni veljavnih slik
* Poškodovane slikovne datoteke
* Premajhen prostor na disku
* Premajhno pomnilniško prosto mesto (RAM)

**Rešitve:**

1. Preverite dnevnik razhroščevanja <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> za sporočila o napakah
2. Preverite razpoložljiv prostor na disku
3. Poskusite obdelati manjši podsklop slik
4. Preverite, ali slike niso poškodovane

### Opozorilo »Ni zaznanih ciljev«

**Možni vzroki:**

* Pozabili ste označiti ciljne slike
* Ciljne slike ne vsebujejo vidnih ciljev
* Nastavitve zaznavanja ciljev so preveč stroge

**Rešitve:**

1. Preberite [Izbira ciljnih slik](choosing-target-images.md)
2. Označite ustrezne slike v stolpcu Cilj
3. Preverite, ali so cilji vidni na označenih slikah
4. Po potrebi prilagodite nastavitve zaznavanja ciljev

***

## Nasveti za uspešno obdelavo

### Pred začetkom

1. **Najprej preizkusite z majhnim podnizom** – obdelajte 10–20 slik, da preverite nastavitve
2. **Preverite razpoložljivi prostor na disku** – zagotovite 2–3-kratno prosto mesto v primerjavi z velikostjo podatkovnega niza
3. **Zaprite nepotrebne aplikacije** – sprostite sistemske vire
4. **Preverite ciljne slike** – si oglejte predogled označenih ciljev, da zagotovite kakovost
5. **Shranite projekt** – Projekt se samodejno shrani, vendar je dobro, da ga shranite ročno

### Med obdelavo

1. **Izogibajte se mirovanju sistema** – Onemogočite načine varčevanja z energijo
2. **Pustite Chloros v ospredju** – Ali pa vsaj vidnega v opravilni vrstici
3. **Občasno spremljajte napredek** – Preverite, ali so prisotna opozorila ali napake
4. **Ne nalagajte drugih zahtevnih programov** – še posebej v vzporednem načinu Chloros+

### Chloros+ Pospeševanje s GPU

Če uporabljate pospeševanje s GPU NVIDIA:

1. Posodobite gonilnike NVIDIA na najnovejšo različico
2. Preverite, ali ima GPU 4 GB ali več VRAM
3. Zaprite aplikacije, ki intenzivno obremenjujejo grafično kartico (igre, urejanje videov)
4. Spremljajte temperaturo grafične kartice (poskrbite za ustrezno hlajenje)

***

## Naslednji koraki

Ko se obdelava začne:

1. **Spremljajte napredek** – glejte [Spremljanje obdelave](monitoring-the-processing.md)
2. **Počakajte na zaključek** – obdelava poteka samodejno
3. **Preglejte rezultate** – glejte [Zaključek obdelave](finishing-the-processing.md)

Za informacije o tem, kaj storiti med obdelavo, glejte [Spremljanje obdelave](monitoring-the-processing.md).

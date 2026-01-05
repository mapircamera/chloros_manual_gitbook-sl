# Začetek obdelave

Ko ste uvozili slike, označili kalibracijske cilje in konfigurirali nastavitve projekta, ste pripravljeni za začetek obdelave. Ta stran vas vodi skozi začetek obdelave Chloros.

## Seznam za predobdelavo

Preden kliknete gumb Začni, preverite, ali je vse pripravljeno:

* [ ] **Uvožene datoteke** – vse slike se prikažejo v brskalniku datotek
* [ ] **Označene ciljne slike** – stolpec Cilj je preverjen za kalibracijske slike
* [ ] **Zaznani modeli kamer** – stolpec Model kamere prikazuje prave kamere
* [ ] **Nastavitve konfigurirane** – nastavitve projekta pregledane in prilagojene
* [ ] **Indeksi izbrani** – dodani želeni multispektralni indeksi (če je potrebno)
* [ ] **Izbran izvozni format** – izhodni format, primeren za vaš delovni tok

{% hint style=&quot;info&quot; %}
**Nasvet**: Pred obdelavo kliknite na nekaj slik v brskalniku datotek, da preverite, ali so se pravilno naložile.
{% endhint %}

***

## Začetek obdelave

### Poiščite gumb Začetek

Gumb Začetek/Predvajanje se nahaja v zgornji vrstici naslova Chloros:

* Položaj: zgornji srednji del okna
* Ikona: **gumb Predvajanje/Začetek** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
* Stanje: Gumb je omogočen (svetel), ko je pripravljen za obdelavo

### Kliknite za začetek

1. Kliknite **gumb Predvajaj/Začni** v zgornjem naslovnem vrstnem redu
2. Obdelava se začne takoj
3. Gumb je med obdelavo onemogočen (siv)
4. Napredek se posodablja in prikazuje stanje obdelave

{% hint style=&quot;success&quot; %}
**Obdelava se je začela**: Ko kliknete, Chloros samodejno opravi vse korake obdelave – zaznavanje cilja, debayering, kalibracijo, izračun indeksa in izvoz.
{% endhint %}

***

## Razumevanje načinov obdelave

Chloros deluje v dveh različnih načinih obdelave, odvisno od vaše licence:

### Brezplačni način (zaporedna obdelava)

**Na voljo vsem uporabnikom**

**Kako deluje:**

* Obdeluje slike eno po eno, zaporedno.
* Enonitno delovanje.
* Manjša poraba pomnilnika.

**Napredek prikazuje 2 stopnji:**

1.**Zaznavanje cilja** – iskanje kalibracijskih ciljev.
2. **Obdelava** – uporaba kalibracije in izvoz slik.**Čas obdelave:**

* Precej počasnejši od vzporednega načina Chloros+
* Primeren za majhne do srednje velike podatkovne nize (&lt; 200 slik)

### Način Chloros+ (vzporedna obdelava)

**Zahteva licenco Chloros+**

**Kako deluje:**

* Hkrati obdeluje več slik
* Večnitno delovanje (do 16 vzporednih delavcev)
* Izkorišča več jedr CPU
* Izbirna pospešitev GPU (CUDA) z grafičnimi karticami NVIDIA

**Napredek prikazuje 4 stopnje:**

1.**Zaznavanje** – iskanje kalibracijskih ciljev
2. **Analiza** – pregledovanje metapodatkov slike in priprava poteka
3. **Kalibriranje** – uporaba popravkov in kalibracij
4. **Izvoz** – shranjevanje obdelanih slik in indeksov**Interakcija z napredovalno vrstico:*** **Premaknite miško** nad vrstico, da se prikaže podrobni spustni meni s 4 stopnjami
* **Kliknite** napredovalno vrstico, da se spustni meni zamrzne na mestu
* **Kliknite še enkrat**, da se spustni meni odmrzne in skrije**Čas obdelave:**

* Precej hitrejši od prostega načina
* Prilagaja se številu jedr CPU
* Pospešek GPU dodatno izboljša hitrost

{% hint style=&quot;info&quot; %}
**Chloros+ Hitrost**: Vzporedna obdelava je lahko 5-10-krat hitrejša od zaporednega načina za velike podatkovne nize. Projekt s 500 slikami, ki v brezplačnem načinu traja 2 uri, se z Chloros+ lahko zaključi v 15-20 minutah.
{% endhint %}

***

## Kaj se dogaja med obdelavo

### Faza 1: Zaznavanje cilja

**Kaj počne Chloros:**

* Skenira označene ciljne slike (ali vse slike, če nobena ni označena)
* Identificira 4 kalibracijske plošče v vsakem cilju
* Izvleče vrednosti odbojnosti iz ciljnih plošč
* Zabeleži časovne oznake ciljev za načrtovanje kalibracije

**Trajanje:** 1–30 sekund (z označenimi cilji), 5–30+ minut (neoznačeni)

### Faza 2: Debayering (pretvorba RAW)

**Kaj počne Chloros:**

* Pretvori podatke RAW Bayerjevega vzorca v polne slike RGB
* Uporabi visokokakovosten algoritem demosaicinga
* Ohrani najvišjo kakovost slike in podrobnosti

**Trajanje:** Odvisno od števila slik in hitrosti CPU

### Stopnja 3: Kalibracija

**Kaj počne Chloros:*** **Popravek vinjete**: odstrani zatemnitev objektiva na robovih
* **Kalibracija odbojnosti**: normalizira z uporabo ciljnih vrednosti odbojnosti
* Uporabi popravke za vse pasove/kanale
* Uporabi ustrezen kalibracijski cilj za vsako sliko na podlagi časovnega žiga

**Trajanje:** Večina časa obdelave

### Stopnja 4: Izračun indeksa

**Kaj počne Chloros:**

* Izračuna konfigurirane multispektralne indekse (NDVI, NDRE itd.)
* Uporabi pasovno matematiko za kalibrirane slike
* Ustvari indeksne slike za vsak izbran indeks

**Trajanje:** nekaj sekund na sliko

### Stopnja 5: Izvoz

**Kaj počne Chloros:**

* Shrani kalibrirane slike v izbranem formatu
* Izvozi indeksne slike s konfiguriranimi barvami LUT
* Zapiše datoteke v podmapo modela kamere
* Ohrani originalna imena datotek s končnicami

**Trajanje:** Odvisno od formata izvoza in velikosti datoteke***

## Obnašanje obdelave

### Avtomatski proces obdelave

Ko se proces začne, poteka avtomatsko:

* Ni potrebno sodelovanje uporabnika
* Vsi nastavljeni koraki se izvajajo zaporedno
* Napredek se prikazuje v realnem času

### Uporaba računalnika med obdelavo

**Prosti način:**

* Relativno nizka poraba CPU (enonitni)
* Računalnik ostane odziven za druge naloge
* Varno je zmanjšati Chloros in delati v drugih aplikacijah

**Chloros+ vzporedni način:**

* Visoka poraba CPU (večnitni, do 16 jeder)
* Z GPU pospešitvijo: Visoka poraba GPU
* Računalnik je med obdelavo lahko manj odziven
* Izogibajte se zagonu drugih nalog, ki intenzivno obremenjujejo CPU

{% hint style=&quot;warning&quot; %}
**Nasvet za izboljšanje zmogljivosti**: Za najboljšo zmogljivost Chloros+ zaprite druge aplikacije in pustite, da Chloros uporabi vse sistemske vire.
{% endhint %}

### Obdelave ni mogoče zaustaviti

**Pomembne omejitve:**

* Ko se obdelava začne, je ni mogoče zaustaviti.
* Obdelavo lahko prekličete, vendar se napredek izgubi.
* Delni rezultati se ne shranijo.
* Če obdelavo prekličete, jo morate začeti znova od začetka.

**Nasvet za načrtovanje:** Pri zelo velikih projektih razmislite o obdelavi v serijah ali uporabi CLI za boljši nadzor.***

## Spremljanje obdelave

Med obdelavo lahko:

* **Spremljate napredek** – ogledate si skupni odstotek dokončanja
* **Ogledate si trenutno fazo** – zaznavanje, analiza, kalibracija ali izvoz
* **Preverite zavihek dnevnika** – ogledate si podrobna sporočila in opozorila o obdelavi
* **Ogledate si predogled dokončanih slik** – med obdelavo se lahko prikažejo nekatere izvozne datoteke

Podrobne informacije o spremljanju najdete v [Spremljanje obdelave](monitoring-the-processing.md).

***

## Preklic obdelave

Če morate ustaviti obdelavo:

### Kako preklicati

1. Poiščite **gumb Stop/Cancel** (med obdelavo nadomesti gumb Start)
2. Kliknite gumb Stop
3. Obdelava se takoj ustavi.
4. Delni rezultati se zavržejo.

### Kdaj preklicati

**Veljavni razlogi za preklic:**

* Ugotovljeno je bilo, da so bile uporabljene napačne nastavitve.
* Pozabili ste označiti ciljne slike.
* Uvožene so bile napačne slike.
* Sistem deluje prepočasi ali se ne odziva.

**Po preklicu:**

* Preglejte in odpravite morebitne težave.
* Po potrebi prilagodite nastavitve.
* Ponovno zaženite obdelavo od začetka.
* Za najboljšo izkušnjo popolnoma zaprite Chloros in ponovno zaženite.

{% hint style=&quot;warning&quot; %}
**Brez delnih rezultatov**: Preklic zavrže ves napredek. Chloros ne shrani delno obdelanih slik.
{% endhint %}

***

## Ocene časa obdelave

Dejanski čas obdelave se močno razlikuje glede na:

* Število slik
* Ločljivost slike
* Vhodni format RAW ali JPG
* Način obdelave (Free ali Chloros+)
* hitrosti procesorja in števila jeder
* razpoložljivosti grafične kartice (samo Chloros+)
* števila indeksov, ki jih je treba izračunati
* zapletenosti izvoznega formata

### Grobe ocene (Chloros+, slike 12 MP, sodoben procesor)

| Število slik | Brezplačni način | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 slik   | 15–20 min | 5–8 min        | 3–5 min        |
| 100 slik  | 30–40 min | 10–15 min      | 5–8 min        |
| 200 slik  | 1–1,5 ure | 20–30 min      | 10–15 min      |
| 500 slik  | 2–3 ure   | 45–60 min      | 20–30 min      |
| 1000 slik | 4–6 ur   | 1,5–2 uri      | 40–60 min      |

{% hint style=&quot;info&quot; %}
**Prvi zagon**: Začetna obdelava lahko traja dlje, ker Chloros ustvarja predpomnilnike in profile. Nadaljnja obdelava podobnih podatkovnih nizov bo hitrejša.
{% endhint %}

***

## Pogoste težave ob zagonu

### Gumb za zagon je onemogočen (siv)

**Možni vzroki:**

* Ni uvoženih slik
* Backend ni v celoti zagnan
* Prejšnja obdelava še vedno teče
* Projekt ni v celoti naložen

**Rešitve:**

1. Počakajte, da se backend v celoti inicializira (preverite ikono glavnega menija).
2. Preverite, ali so slike uvožene v brskalniku datotek.
3. Ponovno zaženite Chloros, če gumb ostane onemogočen.
4. Preverite dnevnik odpravljanja napak za sporočila o napakah.

### Obdelava se začne, nato pa takoj ne uspe

**Možni vzroki:**

* V projektu ni veljavnih slik
* Poškodovane slikovne datoteke
* Premajhen prostor na disku
* Premajhen pomnilnik (RAM)

**Rešitve:**

1. Preverite dnevnik odpravljanja napak <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> za sporočila o napakah
2. Preverite razpoložljivi prostor na disku
3. Poskusite obdelati manjši podsklop slik
4. Preverite, ali slike niso poškodovane

### Opozorilo „Ni zaznanih ciljev“

**Možni vzroki:**

* Pozabili ste označiti ciljne slike
* Ciljne slike ne vsebujejo vidnih ciljev
* Nastavitve zaznavanja ciljev so preveč stroge

**Rešitve:**

1. Preglejte [Izbira ciljnih slik](choosing-target-images.md)
2. Označite ustrezne slike v stolpcu Cilj
3. Preverite, ali so cilji vidni na označenih slikah
4. Po potrebi prilagodite nastavitve zaznavanja ciljev

***

## Nasveti za uspešno obdelavo

### Pred začetkom

1. **Najprej preizkusite z majhnim podsklopom** – obdelajte 10–20 slik, da preverite nastavitve.
2. **Preverite razpoložljivi prostor na disku** – zagotovite 2–3-krat več prostora, kot je velikost podatkovnega niza.
3. **Zaprite nepotrebne aplikacije** – sprostite sistemske vire.
4. **Preverite ciljne slike** – predogledajte označene cilje, da zagotovite kakovost.
5. **Shranite projekt** – projekt se samodejno shrani, vendar je dobro, da ga shranite ročno.

### Med obdelavo

1. **Izogibajte se mirovanju sistema** – onemogočite načine varčevanja z energijo.
2. **Chloros pustite v ospredju** – ali vsaj vidno v opravilni vrstici.
3. **Občasno preverjajte napredek** – preverite, ali so prisotna opozorila ali napake.
4. **Ne nalagajte drugih težkih aplikacij** – še posebej v vzporednem načinu Chloros+.

### Chloros+ GPU pospeševanje

Če uporabljate NVIDIA GPU pospeševanje:

1. Posodobite NVIDIA gonilnike na najnovejšo različico.
2. Preverite, ali ima GPU 4 GB+ VRAM.
3. Zaprite aplikacije, ki intenzivno uporabljajo GPU (igre, urejanje videov)
4. Spremljajte temperaturo GPU (poskrbite za ustrezno hlajenje)

***

## Naslednji koraki

Ko se obdelava začne:

1. **Spremljajte napredek** – glejte [Spremljanje obdelave](monitoring-the-processing.md)
2. **Počakajte na zaključek** – obdelava poteka samodejno.
3. **Preglejte rezultate** – glejte [Zaključek obdelave](finishing-the-processing.md).

Za informacije o tem, kaj storiti med obdelavo, glejte [Spremljanje obdelave](monitoring-the-processing.md).

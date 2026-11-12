# Začetek obdelave

Ko ste uvozili slike, označili kalibracijske cilje in konfigurirali nastavitve projekta, ste pripravljeni za začetek obdelave. Ta stran vas vodi skozi zagon obdelovalnega procesa programa Chloros.

## Seznam za preverjanje pred obdelavo

Preden kliknete gumb »Začni«, preverite, ali je vse pripravljeno:

* [ ] **Datoteke uvožene** – Vse slike so prikazane v brskalniku datotek
* [ ] **Ciljne slike označene** – V stolpcu »Cilj« so označene slike za kalibracijo (ali je za LATTICE uvožen posnetek `.daq`)
* [ ] **Zaznani modeli kamer** – stolpec »Model kamere« prikazuje pravilne kamere
* [ ] **Nastavitve konfigurirane** – nastavitve projekta so pregledane in prilagojene
* [ ] **Izbrani indeksi** – dodani so željeni multispektralni indeksi (če je potrebno)
* [ ] **Izbrana oblika izvoza** – Izhodna oblika, primerna za vaš delovni tok

{% hint style="info" %}
**Nasvet**: Pred obdelavo kliknite na nekaj slik v brskalniku datotek, da preverite, ali so se pravilno naložile.
{% endhint %}

***

## Začetek obdelave

### Poiščite gumb »Začni«

Gumb »Začni/Predvajaj« se nahaja v zgornji glavi programa Chloros:

* Položaj: Zgornji srednji del okna
* Ikona: **Gumb »Predvajaj/Začni«** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Stanje: Gumb je omogočen (svetel), ko je program pripravljen za obdelavo

### Kliknite za zagon

1. Kliknite **gumb Predvajaj/Začni** v zgornjem naslovnem pasu
2. Obdelava se začne takoj
3. Med obdelavo se gumb spremeni v gumb **Stop**

4. Kazalnik napredka se posodablja in prikazuje stanje obdelave

{% hint style="success" %}
**Obdelava se je začela**: Ko kliknete, Chloros samodejno opravi vse korake obdelave – zaznavanje cilja, debayering, kalibracijo, izračun indeksa in izvoz. Samodejno zazna, ali je vaš projekt Survey3, LATTICE ali mešanica, in za vsako kamero uporabi ustrezen postopek obdelave.
{% endhint %}

***

## Razumevanje načinov obdelave Program

Chloros deluje v dveh različnih načinih obdelave, odvisno od vaše licence:

### Brezplačni način (zaporedna obdelava)

**Na voljo vsem uporabnikom**

**Kako deluje:**

* Slike obdeluje po eno naenkrat, zaporedno
* Enonitno delovanje
* Manjša poraba pomnilnika

**Napredovalna vrstica prikazuje 2 fazi:**

1.**Zaznavanje ciljev** – iskanje kalibracijskih ciljev
2. **Obdelava** – izvedba kalibracije in izvoz slik**Čas obdelave:**

* Precej počasnejši od vzporednega načina Chloros+
* Primeren za majhne do srednje velike nize podatkov (< 200 slik)

### Način Chloros+ (vzporedna obdelava)

**Zahteva licenco Chloros+**

**Kako deluje:**

* Hkrati obdeluje več slik z uporabo [4-nitnega obdelovalnega potoka](../processing-architecture/processing-pipeline.md)
* [Dinamična prilagoditev računalniške moči](../processing-architecture/dynamic-compute-adaptation.md) ob zagonu samodejno izbere optimalno strategijo za vašo strojno opremo
* Pospeševanje z GPU (CUDA) z grafičnimi karticami NVIDIA
* **Število delavcev se prilagaja strojni opremi**: strategije za GPU izvajajo**2–4 sočasne delavce** (prilagojeno glede na VRAM — 3 na GPU z 2–12 GB, 2 do 4 na GPU z 12 GB ali več); Sistemi, ki uporabljajo izključno CPU, izvajajo enega delavca na fizično jedro, minus enega**Indikator napredka prikazuje 4 stopnje** (ki ustrezajo 4 nitim v poti):

1. **Zaznavanje** (nit 1) – Iskanje kalibracijskih ciljev
2. **Analiza** (nit 2) – Preučevanje metapodatkov slike in izračun kalibracije
3. **Kalibracija** (nit 3) – Odstranjevanje bayerjevega vzorca, popravljanje vinjetiranja, kalibracija, izračun indeksa
4. **Izvoz** (nit 4) – Shranjevanje obdelanih slik in indeksov

**Uporaba kazalca napredka:**

* **Premaknite miško** nad kazalec, da se prikaže podrobni spustni meni s 4 stopnjami
* **Kliknite** na kazalec napredka, da se spustni meni fiksira na mestu
* **Kliknite še enkrat**, da se fiksiranje prekliče in meni skrije**Čas obdelave:**

* Precej hitrejši od brezplačnega načina
* Pospeševanje s pomočjo grafičnega procesorja (GPU) dodatno izboljša hitrost

{% hint style="info" %}
**Hitrost Chloros+**: Vzporedna obdelava je pri velikih naborih podatkov lahko 5- do 10-krat hitrejša od zaporednega načina. Projekt s 500 slikami, ki v brezplačnem načinu traja 2 uri, se z Chloros+ lahko zaključi v 15–20 minutah.
{% endhint %}

***

## Kaj se dogaja med obdelavo

### Faza 1: Prepoznavanje ciljev

**Kaj počne Chloros:**

* Pregleda slike, ki ste jih označili v stolpcu »Cilj« (vse slike, če ni označena nobena)
* Prepozna kalibracijske plošče na vsakem cilju
* Iz ciljnih plošč izvleče vrednosti odbojnosti
* Zabeleži časovne oznake ciljev za načrtovanje kalibracije

**Trajanje:** 1–30 sekund (z označenimi cilji), 5–30+ minut (neoznačeni)

### Faza 2: Debayering (pretvorba RAW)

**Kaj počne Chloros:**

* Pretvori podatke v formatu RAW z vzorcem Bayer v polne 3-kanalne slike (mono moduli LATTICE ostanejo enopasovni – za njih se debayering preskoči, v dnevniku pa se prikaže opomba)
* Uporabi izbrani algoritem demosaicinga
* Ohrani najvišjo kakovost slike in podrobnosti

**Trajanje:** Odvisno od števila slik ter hitrosti CPU/GPU

### Faza 3: Kalibracija

**Kaj počne Chloros:**

* **Popravek vinjetiranja**: Odstrani potemnitev robov zaradi objektiva
* **Kalibracija odbojnosti**: Normalizira z uporabo ciljnih vrednosti odbojnosti in/ali podatkov o navzdolnem sevanju iz sistema DAQ
* Uporabi popravke za vse pasove/kanale
* Za vsako sliko uporabi ustrezno kalibracijsko referenco na podlagi časovnega žiga

**Trajanje:** Večina časa obdelave

### Faza 4: Izračun indeksov

**Kaj naredi Chloros:**

* Izračuna nastavljene multispektralne indekse (NDVI, NDRE itd.)
* Na kalibrirane slike uporabi matematične operacije na pasovih
* Ustvari indeksne slike za vsak izbrani indeks

**Trajanje:** Nekaj sekund na sliko

### Faza 5: Izvoz

**Kaj naredi Chloros:**

* Shrani obdelane slike v izbranem formatu
* **LATTICE fan-out**: vsak surovi LATTICE-okvir se v enem prehodu izvozi kot vsak omogočen produkt — debayered, predogled, radiance (vedno float32), reflectance
* Zapiše datoteke v izhodno drevesno strukturo projekta: `<project>/<camera>/<format>/<Product>_Images/`
* **Ohrani ime izvorne datoteke** — produkt je identificiran s pomočjo mape, pripona se ne dodaja**Trajanje:** Odvisno od formata izvoza in velikosti datoteke***

## Delovanje obdelave

### Avtomatski proces obdelave

Ko se proces zažene, celoten potek obdelave poteka avtomatsko:

* Ni potrebno posredovanje uporabnika
* Vsi nastavljeni koraki se izvajajo zaporedno
* Napredek se prikazuje v realnem času
* Izvožene datoteke se shranjujejo na disk takoj, ko so pripravljene – končne izhodne datoteke lahko odprete, medtem ko se proces še nadaljuje

### Poraba računalnika med obdelavo

**Prosti način:**

* Razmeroma nizka poraba procesorja (enonitno)
* Računalnik ostaja odziven za druge naloge
* Varno je zmanjšati okno programa Chloros na minimum in delati v drugih aplikacijah

**Vzporedni način Chloros+:**

* Visoka poraba procesorske moči v delovni skupini strategije
* Z grafično pospešitvijo: visoka poraba grafične kartice
* Računalnik je med obdelavo morda manj odziven
* Izogibajte se zagonu drugih nalog, ki močno obremenjujejo procesor

{% hint style="warning" %}
**Nasvet za zmogljivost**: Za najboljšo zmogljivost programa Chloros+ zaprite druge aplikacije in omogočite programu Chloros, da uporabi vse sistemske vire.
{% endhint %}

### Obdelave ni mogoče začasno ustaviti (vendar se program čisto ustavi)

* Ko se obdelava enkrat začne, je ni mogoče začasno ustaviti in kasneje nadaljevati
* S klikom na **Stop** se izvajanje čisto ustavi že ob prvem kliku
* Izdelki, ki so bili izvoženi pred ustavitvijo, ostanejo na disku
* Ustavljeno izvajanje natančno poroča, kaj je bilo zaključeno (glejte vrstice `[RUN-SUMMARY]` v dnevniku)
* Novo izvajanje začne potek obdelave od začetka

**Nasvet za načrtovanje:** Pri zelo velikih projektih razmislite o obdelavi v serijah ali uporabi CLI za boljši nadzor.

***

## Spremljanje obdelave

Med izvajanjem obdelave lahko:

* **Spremljate vrstico napredka** – ogledate si lahko skupni odstotek dokončanja
* **Ogledate trenutno fazo** – zaznavanje, analiza, kalibracija ali izvoz
* **Preverite zavihek dnevnika** – ogledate si lahko podrobna obvestila in opozorila o obdelavi
* **Predogledate dokončane slike** – izvozne datoteke se med obdelavo pojavijo na disku

Za podrobne informacije o spremljanju glejte [Spremljanje obdelave](monitoring-the-processing.md).

***

## Ustavitev obdelave

Če morate ustaviti obdelavo:

### Kako ustaviti

1. Poiščite **gumb »Stop«** (med obdelavo nadomesti gumb »Začni«)
2. Kliknite ga enkrat – na vrstici se prikaže **»Ustavljanje...«**, medtem ko se zaključi obdelava trenutne slike
3. Obdelava se konča v dokončnem ustavljenem stanju, v dnevniku pa se natisne natančen zapis `[RUN-SUMMARY]` o tem, kaj je bilo zaključeno

### Kdaj ustaviti

**Utemeljene razloge za ustavitev:**

* Ugotovili ste, da so bile uporabljene napačne nastavitve
* Pozabili ste označiti ciljne slike
* Uvožene so bile napačne slike
* Sistem deluje prepočasi ali se ne odziva

**Po ustavitvi:**

* Izdelki, izvoženi pred ustavitvijo, ostanejo na disku
* Preglejte in odpravite morebitne težave, po potrebi prilagodite nastavitve
* Ponovno zaženite obdelavo – izvajanje se začne od začetka

***

## Ocene časa obdelave

Dejanski čas obdelave se močno razlikuje glede na:

* Število slik
* Ločljivost slik
* Vhodni format RAW ali JPG
* Način obdelave (Free ali Chloros+)
* Hitrost procesorja in število jeder
* Razpoložljivost grafične kartice (samo Chloros+)
* Število indeksov, ki jih je treba izračunati
* Število omogočenih izvoznih izdelkov (LATTICE)

### Grobe ocene (Chloros+, slike 12 MP, sodoben procesor)

| Število slik | Brezplačni način | Chloros+ (procesor) | Chloros+ (grafična kartica) |
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

* Slike niso uvožene
* Ozadje ni v celoti zagnano
* Prejšnja obdelava še poteka
* Projekt ni v celoti naložen

**Rešitve:**

1. Počakajte, da se strežnik v celoti inicializira (preverite ikono v glavnem meniju)
2. Preverite, ali so slike uvožene v brskalniku datotek
3. Ponovno zaženite Chloros, če gumb ostane onemogočen
4. Preverite dnevnik odpravljanja napak za sporočila o napakah

### Obdelava se začne, nato pa takoj odpove

**Možni vzroki:**

* V projektu ni veljavnih slik
* Poškodovane slikovne datoteke
* Premajhen prostor na disku
* Premajhna količina pomnilnika (RAM)

**Rešitve:**

1. Preverite dnevnik odpravljanja napak <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> za sporočila o napakah
2. Preverite razpoložljiv prostor na disku
3. Poskusite obdelati manjši podsklop slik
4. Preverite, ali slike niso poškodovane

### Izvedba se konča, vendar ne zapiše nobene slike

Izvedba, ki je zahtevala slikovne izdelke, vendar ni zapisala nobenega, se obravnava kot **neuspeh, ne kot uspeh** — program Chloros to jasno sporoči:

* Dnevnik grafičnega vmesnika (GUI) izpiše napotke `[RUN-SUMMARY]`, ki navajajo verjetni vzrok — slike niso bile uvožene, cilj ni bil zaznan ali so bili vsi zahtevani izdelki preskočeni kot neuporabni (npr. zahteva po sevanju/odbojnosti iz kamer, ki snemajo le v RGB)
* Ekvivalent v CLI-ju (`chloros-cli process`) izpiše `Processing finished but wrote no image products.` in **se zaključi z vrednostjo, ki ni nič**, tako da ga skripti lahko zaznajo
* Namerno izvajanje, ki zajema le metapodatke (vsi izvozni izdelki onemogočeni, brez indeksov), še vedno velja za uspešno

Za celotno semantiko glej [referenco CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails).

### Projektna mapa je za ta računalnik pregloboka

**Samo za Windows, novost v različici 1.3.0.** Windows je privzeto nastavljen tako, da so dolge poti onemogočene, zato nobena pot do datoteke ne sme biti daljša od 260 znakov. Izdelki Chloros se nahajajo do približno 130 znakov pod projektno mapo, zato projektna mapa, ki je globlje od tega, ne more prejeti svojih izdelkov. Namesto da bi se izvedba začela in ne bi zapisala ničesar, Chloros zdaj pred začetkom obdelave preveri:

* S klikom na »Start« se izvajanje ne začne: napredovalna vrstica prikaže razlog, v dnevniku za odpravljanje napak pa se prikaže `[PROCESS] refused: The project folder is too deep for this computer…`. Sporočilo navaja dolžino projektne mape in najdaljšo projektno mapo, ki ustreza.
* Tudi ustvarjanje ali odpiranje projekta v mapi, ki je pregloboka celo za lastne datoteke projekta, je zavrnjeno. V oknu s sporočilom je naveden razlog, najdaljša mape projekta, ki ustreza omejitvi, in navodila za vklop dolgih poti; nič se ne ustvari ali odpre; v dnevniku razhroščevanja se to zabeleži kot `[PROJECT] refused: …`. Tudi kopiranje projekta pod daljšim imenom lahko preseže omejitev: kopija se ustvari, vendar se ne odpre, isto sporočilo pa navaja razlog.

**Rešitve:**

1. Shranite projekte v krajšo mapo — spremenite**Delovni imenik** v [Nastavitvah projekta](../project-settings/project-settings.md) in projekt ustvarite na novo, ali
2. Vklopite dolge poti v sistemu Windows in ponovno zaženite Chloros: ponovno zaženite namestitveni program Chloros z označeno možnostjo **Omogoči dolge poti v sistemu Windows** (ta je privzeto označena) ali uporabite skupinsko politiko: *Konfiguracija računalnika > Upravne predloge > Sistem > Datotečni sistem > Omogoči dolge poti Win32*.

### Opozorilo »Ni zaznanih ciljev«

**Možni vzroki:**

* Pozabili ste označiti ciljne slike
* Ciljne slike ne vsebujejo vidnih ciljev
* Nastavitve zaznavanja ciljev so preveč stroge

**Rešitve:**

1. Preglejte [Izbira ciljnih slik](choosing-target-images.md)
2. Označite ustrezne slike v stolpcu »Cilj«
3. Preverite, ali so cilji vidni na označenih slikah
4. Po potrebi prilagodite nastavitve zaznavanja ciljev

***

## Nasveti za uspešno obdelavo

### Pred začetkom

1. **Najprej opravite preskus z majhnim podnaborom** – obdelajte 10–20 slik, da preverite nastavitve
2. **Preverite razpoložljivi prostor na disku** – zagotovite 2–3-kratno prosto prostora glede na velikost podatkovnega niza (več, če so omogočeni vsi izdelki LATTICE)
3. **Zaprite nepotrebne aplikacije** – sprostite sistemske vire
4. **Preverite slike ciljev** – si oglejte predogled označenih ciljev, da zagotovite kakovost
5. **Shranite projekt** – projekt se samodejno shrani, vendar je priporočljivo, da ga shranite ročno

### Med obdelavo

1. **Izogibajte se mirovanju sistema** – Onemogočite načine varčevanja z energijo
2. **Chloros naj ostane v ospredju** – Ali pa naj bo vsaj viden na opravilni vrstici
3. **Občasno spremljajte napredek** – Preverite, ali so prisotna opozorila ali napake
4. **Ne zagnajte drugih zahtevnih programov** – Še posebej v vzporednem načinu Chloros+

### Pospeševanje z grafičnim procesorjem (GPU) v programu Chloros+

Če uporabljate pospeševanje z grafičnim procesorjem (GPU) NVIDIA:

1. Posodobite gonilnike NVIDIA na najnovejšo različico
2. Preverite, ali ima grafični procesor vsaj 4 GB VRAM (7 GB ali več za sočasno odstranjevanje pik (debayering) z upoštevanjem tekstur)
3. Zaprite aplikacije, ki intenzivno obremenjujejo grafični procesor (igre, urejanje videov)
4. Spremljajte temperaturo grafičnega procesorja (poskrbite za ustrezno hlajenje)

***

## Naslednji koraki

Ko se obdelava začne:

1. **Spremljajte napredek** – glejte [Spremljanje obdelave](monitoring-the-processing.md)
2. **Počakajte na zaključek** – obdelava poteka samodejno
3. **Preglejte rezultate** – glejte [Zaključek obdelave](finishing-the-processing.md)

Za informacije o tem, kaj storiti med obdelavo, glejte [Spremljanje obdelave](monitoring-the-processing.md).

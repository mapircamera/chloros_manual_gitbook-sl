# Izbiranje ciljnih posnetkov

Z označevanjem posnetkov, ki vsebujejo kalibracijske cilje, programu Chloros natančno sporočite, kje naj jih išče. Kadar je v stolpcu »Target« označen vsaj en posnetek, program Chloros skenira **samo označene posnetke** — tako z označevanjem ciljev pospešite obdelavo in hkrati preprečite, da bi se posnetki raziskave pomotoma šteli za cilje.

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

## Zakaj označiti ciljne slike?

### Označevanje nadzira skeniranje

Ko v stolpcu »Cilj« označite določene slike:

* Chloros skenira le označene slike za iskanje ciljev
* Odkrivanje ciljev poteka veliko hitreje
* Geodetske slike ne morejo povzročiti napačnih zaznav ciljev

Če **ni** označena nobena slika, program »Chloros« preide na skeniranje vseh slik v projektu:

* Algoritmi za zaznavanje ciljev se izvajajo na vsaki sliki
* Nepotrebno se pregleda na stotine ali tisoče slik
* Obdelava traja znatno dlje, zlasti pri velikih naborih podatkov

{% hint style="success" %}
**Izboljšanje hitrosti**: Označitev 2–3 slik s cilji v naboru podatkov s 500 slikami lahko skrajša čas zaznavanja ciljev s 30+ minut na manj kot 1 minuto.
{% endhint %}

***

## Kako označiti slike ciljev

### Korak 1: Prepoznajte slike ciljev

Preglejte uvožene slike v brskalniku datotek in ugotovite, katere slike vsebujejo kalibracijske cilje.

**Pogosti scenariji:**

* **Cilj pred zajemom**: zajet pred začetkom seje
* **Cilj po zajemu**: posnet po zaključku seje
* **Cilji na terenu**: cilji, nameščeni znotraj območja zajema
* **Več ciljev**: 2–3 slike ciljev na sejo (priporočeno)

### Korak 2: Preverite stolpec »Target« (<img src="../.gitbook/assets/image (33).png" alt="" data-size="original">)

Za vsako sliko, ki vsebuje kalibracijski cilj:

1. Poiščite sliko v tabeli brskalnika datotek
2. Poiščite stolpec **Target** (skrajni desni stolpec)
3. Kliknite potrditveno polje v stolpcu »Target« za to sliko
4. Ponovite za vse slike, ki vsebujejo cilje

### Korak 3: Preverite izbor

Pred obdelavo še enkrat preverite:

* [ ] Vse slike s kalibracijskimi cilji so označene
* [ ] Nobena slika, ki ni cilj, ni bila pomotoma označena
* [ ] Cilji so na označenih slikah jasno vidni

***

## LATTICE: Cilji so neobvezni, ko DAQ snema

Pri multispektralnih kamerah LATTICE je kalibracijski cilj v okviru **eden od dveh** možnih referenčnih vrednosti odbojnosti:

* **Cilj v okviru**: ko označena slika cilja izpolni merila kakovosti (QA) sistemChloros, cilj postane**absolutna referenca odbojnosti** za slike v njegovi okolici.
* **Svetlobni tok navzdol iz DAQ**: kadar cilj ni prisoten (ali ne izpolni meril kakovosti QA), sistem za avtomatsko kalibracijo (Chloros) namesto tega izračuna odbojnost iz svetlobnega toka navzdol svetlobnega senzorja DAQ (ρ = π·L/E). Če vaš posnetek zajema snemanje `.daq` ali DAQ-M `.csv`, dobite kalibrirano odbojnost**brez kakršnih koli ciljnih slik**.

To samodejno delovanje je privzeto. V datoteki »CLI« / »SDK« to ustreza `--reflectance-source auto`; lahko pa tudi prisilite uporabo `target` (strogo — brez nadomestitve s strani DAQ) ali `daq` (prednost DAQ). Glejte [CLI Referenca](../reference/cli-reference.md#per-product-export-toggles-lattice-multispectral).

**Geometrije ciljev LATTICE**: poleg klasičnega zaznavanja plošč, ki se uporablja za Survey3, obdelava LATTICE podpira**cilje, označene z ArUco**,**cilje s fiksnim ROI**in**cilje v obliki trakov**, ki se konfigurirajo za vsak projekt posebej.**Izmerjene** skene odbojnosti ciljev na enoto je mogoče zagotoviti po serijski številki (CLI: `--target-reflectance-dir`, en `<serial>.csv` na ciljno enoto), pri čemer se kot rezerva uporabijo nominalni spektri T3/T4P.

{% hint style="info" %}
**Modul F988**: Odbojnost F988 se kalibrira z uporabo plošče za merjenje odbojnosti v prizorišču: pas leži zunaj kalibriranega območja svetlobnega senzorja DAQ, zato Chloros uporabi vaš najnovejši posnetek plošče in ga ohrani med posameznimi opazovanji plošče. Če se modul F988 obdeluje izključno z DAQ, program »Chloros« zavrne odbojnost na podlagi DAQ za ta pas (razlog za preskok `dls-uncalibrated-band-988`) — podprt postopek je delo s ploščo.
{% endhint %}

***

## Najboljše prakse za slike ciljnih predmetov

### Smernice za zajemanje ciljnih predmetov

**Časovni razpored:**

* Slike ciljnih predmetov zajemajte takoj pred in med celotno sejo zajemanja
* V enakih svetlobnih pogojih kot svetlobni senzor DAQ
* Za najboljše rezultate slike cilja idealno zajemajte čim pogosteje. V nasprotnem primeru se bodo podatki svetlobnega senzorja sčasoma uporabili za prilagajanje kalibracije.

**Položaj kamere:**

* Kamero držite nad ciljem tako, da je ta v središču in zapolnjuje približno 40–60 % sredine slike.
* Kamero držite vzporedno s površino cilja ali v nadirni legi glede na njo

**Osvetlitev:**

* Enaka okoljska osvetlitev kot pri svetlobnem senzorju DAQ
* Izogibajte se sencam na površinah cilja
* Ne zaslanjajte vira svetlobe s svojim telesom, vozilom ali rastlinjem
* Oblačni pogoji zagotavljajo najbolj dosledne rezultate

**Stanje cilja:**

* Plošče cilja naj bodo čiste in suhe.
* Vse plošče cilja (npr. vse 4 na T4) morajo biti jasno vidne in neovirane.
* Cilji naj bodo, če je mogoče, pravokotni na vir svetlobe oziroma v nadirni legi glede na vir svetlobe.

### Koliko slik cilja?

**Najmanj:**1 slika cilja na sejo.**Priporočeno:** 3–5 slik cilja na sejo.**Priporočeni urnik:**

* 3–5 slik, posnetih kmalu po začetku snemanja svetlobnega senzorja
* Za najboljše rezultate med posameznimi posnetki obračajte kamero
* Izbirno: občasno med sejo, če se svetlobne razmere nenehno spreminjajo

***

## Delovanje z več kamerami

### Konfiguracije z dvema kamerama

Če hkrati uporabljate dve kameri MAPIR (npr. Survey3W RGN + Survey3N OCN):

1. Posnemite slike ciljev z **obe kameri** hkrati
2. Za obe kameri uporabite **isti fizični cilj**

3. V brskalniku datotek označite slike ciljev za**oba tipa kamer**

4. Program »Chloros« bo za kalibracijo vsake kamere uporabil ustrezne cilje

### Stolpec »Model kamere«

Stolpec **»Model kamere«** pomaga ugotoviti, katere slike izhajajo iz katere kamere:

* Survey3W\_RGN
* Survey3N\_OCN
* LATT-M3M-L41-F550
* LATT-M3C-L87-FRGN
* itd.

Ta stolpec uporabite za preverjanje, ali ste v svojem projektu označili cilje za vsak tip kamere.

***

## Nastavitve zaznavanja ciljev

### Prilagajanje občutljivosti zaznavanja

Če program »Chloros« ne zazna vaših ciljev pravilno, prilagodite te nastavitve v [Nastavitvah projekta](adjusting-project-settings.md):

**Najmanjša površina kalibracijskega vzorca (px):**

* **Privzeto**: 25 pikslov
* **Povečajte**, če prihaja do napačnih zaznav pri majhnih artefaktih
* **Zmanjšajte**, če se cilji ne zaznavajo

**Najmanjše združevanje ciljev (0–100):**

* **Privzeto**: 60
* **Povečajte**, če se cilji razdelijo na več zaznav
* **Zmanjšajte**, če cilji z barvnimi odstopanji niso v celoti zaznani

{% hint style="info" %}
**Nasvet za CLI**: `chloros-cli process` sprejema iste parametre (`--min-target-size`, `--target-clustering`), njegov zastavici `--target`/`--targets` pa označita celotno vhodno mapo kot »samo za ciljni panel«. Glejte [CLI Referenca](../reference/cli-reference.md).
{% endhint %}

***

## Pogoste težave s ciljnimi slikami

### Težava: Cilji niso bili zaznani

**Možni vzroki:**

* Ciljne slike niso označene v brskalniku datotek
* Cilj je v okvirju premajhen (< 30 % slike)
* Slaba osvetlitev (sence, bleščanje)
* Preveč stroge nastavitve zaznavanja ciljev

**Rešitve:**

1. Preverite, ali je pri ustreznih slikah označen stolpec »Cilj«
2. Preverite kakovost ciljnih slik v predogledu
3. Če je kakovost slaba, cilje posnemite na novo
4. Po potrebi prilagodite nastavitve zaznavanja ciljev

### Težava: Napačno zaznavanje ciljev

**Možni vzroki:**

* Bele stavbe, vozila ali talna odeja, ki se zmotno zamenjujejo za cilje
* Svetle lise v vegetaciji
* Premajhna občutljivost zaznavanja

**Rešitve:**

1. Označite le dejanske slike ciljev – skenirane bodo le označene slike
2. Povečajte minimalno površino kalibracijskega vzorca
3. Povečajte minimalno vrednost združevanja ciljev
4. Poskrbite, da slike ciljev prikazujejo le cilj (čim manj motečega ozadja)

***

## Seznam za preverjanje

Pred začetkom obdelave preverite izbor slik ciljev:

* [ ] Označena je vsaj 1 slika cilja na sejo (ali, za LATTICE, posnetek `.daq`/`.csv`, ki pokriva sejo)
* [ ] Polja za cilje so označena za vse slike ciljev
* [ ] Slike ciljev so posnete v istem časovnem okviru kot raziskava
* [ ] Cilji so jasno vidni v predogledu ob kliku
* [ ] Vsi kalibracijski paneli so vidni na vsaki sliki cilja
* [ ] Na ciljih ni senč ali ovir
* [ ] Pri uporabi dveh kamer: cilji so označeni za oba tipa kamer

***

## Obdelava brez ciljev

### LATTICE: Z DAQ-posnetkom

Če je svetlobni senzor DAQ med vašimi zajemi LATTICE zabeležil navzdol usmerjeno sevanje, cilji niso potrebni:

1. Uvozite datoteko `.daq` (ali DAQ-M `.csv`) s slikami
2. Stolpec »Target« pustite neoznačen
3. Odbojnost se samodejno izračuna iz referenčnega navzdolnega sevanja DAQ
4. Za sevanje nikoli ni potreben cilj ali DAQ — izhaja izključno iz tovarniške radiometrične kalibracije kamere

### Obdelava brez kakršne koli reference

Obdelavo lahko izvedete tudi brez ciljev in brez DAQ:

1. Pustite vsa polja v stolpcu »Cilj« neizbrana
2. **Onemogočite** »Kalibracijo odbojnosti / belo ravnovesje« v nastavitvah projekta — zaznavanje ciljnih točk se nato v celoti preskoči
3. Popravek vinjetiranja se bo še vedno uporabil
4. Izhodni podatki ne bodo kalibrirani za absolutno odbojnost (LATTICE multispektralni program še vedno izvaža izdelke brez bayerske matrike, predogled in sevanje)

{% hint style="warning" %}
**Ni priporočljivo za znanstveno delo z Survey3**: Brez kalibracije odbojnosti vrednosti pikslov v datotekah »Survey3« predstavljajo le relativno svetlost, ne pa znanstvenih meritev odbojnosti. Za natančne in ponovljive rezultate uporabite kalibracijske cilje (ali, za LATTICE, svetlobni senzor DAQ).
{% endhint %}

***

## Naslednji koraki

Ko ste označili ciljne slike:

1. **Preverite nastavitve** – glejte [Prilagajanje nastavitev projekta](adjusting-project-settings.md)
2. **Začnite obdelavo** – glejte [Začetek obdelave](starting-the-processing.md)
3. **Spremljajte napredek** – glejte [Spremljanje obdelave](monitoring-the-processing.md)

Za več informacij o samih kalibracijskih ciljih glejte [Kalibracijski cilji](../calibration-targets.md).

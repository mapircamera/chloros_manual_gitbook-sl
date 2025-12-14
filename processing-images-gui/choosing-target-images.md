# Izbiranje ciljnih slik

Označevanje slik, ki vsebujejo kalibracijske cilje, je ključni korak, ki znatno pospeši obdelavo Chloros. Z vnaprejšnjo izbiro ciljnih slik odpravite potrebo, da Chloros skenira vsako sliko v vašem nizu podatkov za kalibracijske cilje.

## Zakaj označiti ciljne slike?

### Hitrost obdelave

Brez označevanja ciljnih slik mora Chloros:

* pregledati vsako posamezno sliko v projektu
* izvesti algoritme za zaznavanje ciljev na vsaki sliki
* nepotrebno preveriti na stotine ali tisoče slik

**Rezultat**: obdelava lahko traja bistveno dlje, zlasti pri velikih podatkovnih nizih.

### Z označenimi ciljnimi slikami

Ko v stolpcu Cilj označite določene slike:

* Chloros pregleda le označene slike za cilje
* Odkrivanje ciljev je veliko hitrejše
* Skupni čas obdelave je znatno krajši

{% hint style=&quot;success&quot; %}
**Izboljšanje hitrosti**: Označitev 2–3 ciljnih slik v nizu podatkov s 500 slikami lahko skrajša čas zaznavanja ciljev s 30+ minut na manj kot 1 minuto.
{% endhint %}

***

## Kako označiti ciljne slike

### Korak 1: Prepoznajte ciljne slike

Preglejte uvožene slike v brskalniku datotek in prepoznajte, katere slike vsebujejo kalibracijske cilje.

**Pogosti scenariji:**

* **Cilj pred zajemom**: zajet pred začetkom seje
* **Cilj po zajemu**: zajet po zaključku seje
* **Cilji na terenu**: cilji, nameščeni znotraj območja zajema
* **Več ciljev**: 2–3 ciljne slike na sejo (priporočeno)

### Korak 2: Preverite stolpec Cilj

Za vsako sliko, ki vsebuje kalibracijski cilj:

1. Poiščite sliko v tabeli brskalnika datotek.
2. Poiščite stolpec **Cilj** (skrajni desni stolpec).
3. Kliknite potrditveno polje v stolpcu Cilj za to sliko.
4. Ponovite za vse slike, ki vsebujejo cilje.

### Korak 3: Preverite svojo izbiro

Pred obdelavo še enkrat preverite:

* [ ] Vse slike s cilji kalibracije so označene.
* [ ] Nobena slika, ki ni ciljna, ni bila naključno označena.
* [ ] Cilji so jasno vidni na označenih slikah.

***

## Najboljše prakse za ciljne slike

### Smernice za zajem ciljev

**Čas:**

* Ciljne slike zajemajte takoj pred in med sejo zajema.
* V enakih svetlobnih pogojih kot svetlobni senzor DAQ.
* Za najboljše rezultate ciljne slike zajemajte čim pogosteje. V nasprotnem primeru se bodo za prilagajanje kalibracije sčasoma uporabili podatki svetlobnega senzorja.

**Položaj kamere:**

* Kamero držite nad ciljem tako, da je centrirana in zapolnjuje približno 40–60 % sredine slike.
* Kamero držite vzporedno/nadirno s ciljno površino

**Osvetlitev:**

* Enaka osvetlitev okolice kot pri senzorju svetlobe DAQ.
* Izogibajte se sencam na ciljnih površinah.
* Ne prekrivajte vira svetlobe s svojim telesom, vozilom ali vegetacijo.
* Oblačni pogoji zagotavljajo najbolj dosledne rezultate.

**Pogoji cilja:**

* Ciljne plošče morajo biti čiste in suhe.
* Vse 4 plošče morajo biti jasno vidne in neovirane.
* Cilji morajo biti po možnosti pravokotni/nadirni glede na vir svetlobe.

### Koliko ciljnih slik?

**Minimalno:** 1 slika cilja na sejo. **Priporočeno:** 3–5 slik cilja na sejo.

**Najboljši urnik:**

* 3–5 slik, posnetih kmalu po začetku snemanja svetlobnega senzorja
* Za najboljše rezultate med posnetki vrtite kamero
* Izbirno: redno med sejo, če se svetlobne razmere nenehno spreminjajo

***

## Delovanje z več kamerami

### Nastavitve z dvema kamerama

Če hkrati uporabljate dve kameri MAPIR (npr. Survey3W RGN + Survey3N OCN):

1. Hkrati zajemajte ciljne slike z **obe kamerami**.
2. Za obe kameri uporabite **isti fizični cilj**.
3. V brskalniku datotek označite ciljne slike za **obe vrsti kamer**.
4. Chloros bo za kalibracijo vsake kamere uporabil ustrezne cilje.

### Stolpec Model kamere

Stolpec **Model kamere** pomaga identificirati, katere slike so bile posnete s katero kamero:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* itd.

Ta stolpec uporabite za preverjanje, ali ste v svojem projektu označili cilje za vsak tip kamere.

***

## Nastavitve zaznavanja ciljev

### Nastavitev občutljivosti zaznavanja

Če Chloros ne zazna vaših ciljev pravilno, prilagodite te nastavitve v [Nastavitve projekta](adjusting-project-settings.md):

**Minimalno kalibrirano vzorčno območje:**

* **Privzeto**: 25 pikslov
* **Povečajte**, če pride do napačnih zaznav na majhnih artefaktih.
* **Zmanjšajte**, če cilji niso zaznavani.

**Minimalno združevanje ciljev:**

* **Privzeto**: 60
* **Povečajte**, če so cilji razdeljeni na več zaznav
* **Zmanjšajte**, če cilji z barvnimi razlikami niso v celoti zaznavni

***

## Pogoste težave s slikami ciljev

### Težava: Cilji niso zaznavni

**Možni vzroki:**

* Slike ciljev niso označene v brskalniku datotek
* Cilj je premajhen v okviru (&lt; 30 % slike)
* Slaba osvetlitev (sence, bleščanje)
* Preveč stroge nastavitve zaznavanja ciljev

**Rešitve:**

1. Preverite, ali je stolpec Cilj označen za pravilne slike.
2. Preverite kakovost slike cilja v predogledu.
3. Ponovno zajemite cilje, če je kakovost slaba.
4. Po potrebi prilagodite nastavitve zaznavanja ciljev.

### Težava: napačno zaznavanje ciljev

**Možni vzroki:**

* Bele stavbe, vozila ali talna pokrivnost, ki se zamenjujejo za cilje
* Svetle lise v vegetaciji
* Premajhna občutljivost zaznavanja

**Rešitve:**

1. Označite samo dejanske ciljne slike, da omejite obseg zaznavanja
2. Povečajte minimalno kalibrirano vzorčno območje
3. Povečajte minimalno vrednost združevanja ciljev
4. Preverite, da ciljne slike prikazujejo samo cilj (minimalno ozadje)

***

## Seznam za preverjanje

Pred začetkom obdelave preverite izbiro slik ciljev:

* [ ] Vsaj 1 označena slika cilja na sejo
* [ ] Potrditvena polja v stolpcu Cilji so potrjena za vse slike ciljev
* [ ] Slike ciljev so posnete v istem časovnem okviru kot raziskava
* [ ] Cilji so jasno vidni v predogledu, ko nanje kliknete
* [ ] Vsi 4 kalibracijski paneli so vidni na vsaki sliki cilja
* [ ] Na ciljih ni senc ali ovir
* [ ] Za dvojno kamero: cilji so označeni za oba tipa kamer

***

## Obdelava brez ciljev

### Obdelava brez kalibracijskih ciljev

Čeprav to ni priporočljivo za znanstveno delo, lahko obdelujete brez ciljev:

1. Vse potrditvene polja v stolpcu Cilj pustite neizpolnjena.
2. **Onemogočite** »Kalibracijo odbojnosti« v nastavitvah projekta.
3. Popravek vinjete bo še vedno uporabljen.
4. Izhod ne bo kalibriran za absolutno odbojnost.

{% hint style=&quot;warning&quot; %}
**Ni priporočljivo**: Brez kalibracije odbojnosti vrednosti pikslov predstavljajo le relativno svetlost, ne pa znanstvene meritve odbojnosti. Za natančne in ponovljive rezultate uporabite kalibracijske cilje.
{% endhint %}

***

## Naslednji koraki

Ko označite ciljne slike:

1. **Preglejte nastavitve** – glejte [Prilagajanje nastavitev projekta](adjusting-project-settings.md)
2. **Začnite obdelavo** – glejte [Začetek obdelave](starting-the-processing.md)
3. **Spremljajte napredek** – glejte [Spremljanje obdelave](monitoring-the-processing.md)

Za več informacij o kalibracijskih ciljih glejte [Kalibracijski cilji](../calibration-targets.md).

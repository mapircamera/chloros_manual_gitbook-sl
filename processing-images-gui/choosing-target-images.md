# Izbiranje ciljnih slik

Označevanje slik, ki vsebujejo kalibracijske cilje, je ključni korak, ki znatno pospeši potek obdelave v Chloros. Z vnaprejšnjo izbiro ciljnih slik Chloros ni več treba pregledovati vsake slike v vašem nizu podatkov za iskanje kalibracijskih ciljev.

## Zakaj označiti ciljne slike?

### Hitrost obdelave

Brez označevanja ciljnih slik mora Chloros:

* Pregledati vsako posamezno sliko v vašem projektu
* Izvesti algoritme za zaznavanje ciljev na vsaki sliki
* Nepotrebno preveriti na stotine ali tisoče slik

**Rezultat**: Obdelava lahko traja bistveno dlje, še posebej pri velikih podatkovnih nizih.

### Z označenimi ciljnimi slikami

Ko v stolpcu Cilj označite določene slike:

* Chloros skenira cilje le na označenih slikah
* Prepoznavanje ciljev se zaključi veliko hitreje
* Skupni čas obdelave se močno skrajša

{% hint style="success" %}
**Izboljšanje hitrosti**: Označitev 2–3 ciljnih slik v nizu podatkov s 500 slikami lahko skrajša čas prepoznavanja ciljev s 30+ minut na manj kot 1 minuto.
{% endhint %}

***

## Kako označiti slike ciljev

### Korak 1: Prepoznajte slike ciljev

Preglejte uvožene slike v brskalniku datotek in prepoznajte, katere slike vsebujejo kalibracijske cilje.

**Pogosti scenariji:*** **Cilj pred zajemom**: zajet pred začetkom seje
* **Cilj po zajemu**: zajet po zaključku seje
* **Cilji na terenu**: cilji, nameščeni znotraj območja zajema
* **Več ciljev**: 2–3 ciljne slike na sejo (priporočeno)

### Korak 2: Preverite stolpec Cilj

Za vsako sliko, ki vsebuje kalibracijski cilj:

1. Poiščite sliko v tabeli brskalnika datotek
2. Poiščite stolpec **Cilj** (skrajni desni stolpec)
3. Kliknite potrditveno polje v stolpcu Cilj za to sliko
4. Ponovite za vse slike, ki vsebujejo cilje

### Korak 3: Preverite svojo izbiro

Pred obdelavo še enkrat preverite:

* [ ] So označene vse slike s kalibracijskimi cilji
* [ ] Niso bile po pomoti označene slike, ki niso cilji
* [ ] So cilji jasno vidni na označenih slikah

***

## Najboljše prakse za ciljne slike

### Smernice za zajemanje ciljev

**Čas:**

* Zajemajte ciljne slike takoj pred in med sejo zajemanja
* V enakih svetlobnih pogojih kot vaš svetlobni senzor DAQ
* Za najboljše rezultate slike ciljev zajemajte čim pogosteje. V nasprotnem primeru se bodo podatki svetlobnega senzorja uporabili za prilagajanje kalibracije skozi čas.

**Položaj kamere:**

* Kamero držite nad ciljem tako, da je ta v središču in zapolnjuje približno 40–60 % sredine slike.
* Kamero držite vzporedno/navpično nad površino cilja

**Osvetlitev:**

* Enaka okoljska osvetlitev kot pri vašem svetlobnem senzorju DAQ
* Izogibajte se sencam na površinah ciljev
* Ne blokirajte vira svetlobe s svojim telesom, vozilom ali rastlinjem
* Oblačni pogoji zagotavljajo najbolj dosledne rezultate

**Stanje cilja:**

* Plošče ciljev naj bodo čiste in suhe
* Vse 4 plošče morajo biti jasno vidne in neovirane
* Cilji naj bodo po možnosti pravokotni/nadirni glede na vir svetlobe

### Koliko slik ciljev?

**Najmanj:**1 slika cilja na sejo.**Priporočeno:** 3–5 slik cilja na sejo.**Najboljši urnik:**

* 3–5 slik, posnetih kmalu po tem, ko senzor svetlobe začne snemati
* Za najboljše rezultate med posnetki obračajte kamero
* Izbirno: občasno med sejo, če se svetlobne razmere nenehno spreminjajo

***

## Delovanje z več kamerami

### Nastavitve z dvema kamerama

Če hkrati uporabljate dve kameri MAPIR (npr. Survey3W RGN + Survey3N OCN):

1. Posnemite ciljne slike z **obe kameri** hkrati
2. Uporabite **isti fizični cilj** za obe kameri
3. Označite ciljne slike za **oba tipa kamer** v brskalniku datotek
4. Chloros bo uporabil ustrezne cilje za kalibracijo vsake kamere

### Stolpec Model kamere

Stolpec **Model kamere** pomaga ugotoviti, katere slike izhajajo iz katere kamere:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* itd.

Uporabite ta stolpec, da preverite, ali ste v svojem projektu označili cilje za vsak tip kamere.

***

## Nastavitve zaznavanja ciljev

### Nastavitev občutljivosti zaznavanja

Če Chloros ne zazna vaših ciljev pravilno, prilagodite te nastavitve v [Nastavitvah projekta](adjusting-project-settings.md):**Najmanjša površina kalibracijskega vzorca:*** **Privzeto**: 25 pikslov
* **Povečajte**, če prihaja do napačnih zaznavanj na majhnih artefaktih
* **Zmanjšajte**, če cilji niso zaznavani**Najmanjše združevanje ciljev:*** **Privzeto**: 60
* **Povečajte**, če se cilji razdelijo na več zaznavanj
* **Zmanjšajte**, če cilji z barvnimi odstopanji niso v celoti zaznani***

## Pogoste težave s slikami ciljev

### Težava: Ni zaznanih ciljev

**Možni vzroki:**

* Slike ciljev niso označene v brskalniku datotek
* Cilj je v okviru premajhen (&lt; 30 % slike)
* Slaba osvetlitev (sence, bleščanje)
* Nastavitve zaznavanja ciljev so preveč stroge

**Rešitve:**

1. Preverite, ali je stolpec Cilj označen za pravilne slike
2. Preverite kakovost slike cilja v predogledu
3. Ponovno zajemite cilje, če je kakovost slaba
4. Po potrebi prilagodite nastavitve zaznavanja ciljev

### Težava: Napačno zaznavanje ciljev

**Možni vzroki:**

* Bele stavbe, vozila ali talna odeja, ki se zamenjujejo za cilje
* Svetle lise v vegetaciji
* Preveč nizka občutljivost zaznavanja

**Rešitve:**

1. Označite le dejanske slike ciljev, da omejite obseg zaznavanja
2. Povečajte minimalno površino kalibracijskega vzorca
3. Povečajte minimalno vrednost združevanja ciljev
4. Poskrbite, da slike ciljev prikazujejo le cilj (minimalno motenje ozadja)

***

## Seznam za preverjanje

Pred začetkom obdelave preverite izbor slik ciljev:

* [ ] Označena je vsaj 1 slika cilja na sejo
* [ ] Polja v stolpcu Cilj so označena za vse slike ciljev
* [ ] Slike ciljev so posnete v istem časovnem okviru kot raziskava
* [ ] Cilji so jasno vidni v predogledu, ko se nanje klikne
* [ ] Vsi 4 kalibracijski paneli so vidni na vsaki sliki cilja
* [ ] Na ciljih ni senc ali ovir
* [ ] Pri dvojni kameri: Cilji so označeni za oba tipa kamer

***

## Obdelava brez ciljev

### Obdelava brez kalibracijskih ciljev

Čeprav to ni priporočljivo za znanstveno delo, lahko obdelujete brez ciljev:

1. Pustite vsa potrditvena polja v stolpcu Cilj neoznačena
2. **Onemogočite** »Kalibracijo odbojnosti« v nastavitvah projekta
3. Popravek vinjetiranja bo še vedno uporabljen
4. Izhod ne bo kalibriran za absolutno odbojnost

{% hint style="warning" %}
**Ne priporočljivo**: Brez kalibracije odbojnosti vrednosti pikslov predstavljajo le relativno svetlost, ne pa znanstvene meritve odbojnosti. Za natančne in ponovljive rezultate uporabite kalibracijske tarče.
{% endhint %}

***

## Naslednji koraki

Ko ste označili svoje ciljne slike:

1. **Preverite nastavitve** – glejte [Prilagajanje nastavitev projekta](adjusting-project-settings.md)
2. **Začnite obdelavo** – glejte [Začetek obdelave](starting-the-processing.md)
3. **Spremljajte napredek** – glejte [Spremljanje obdelave](monitoring-the-processing.md)

Za več informacij o samih kalibracijskih ciljih glejte [Kalibracijski cilji](../calibration-targets.md).

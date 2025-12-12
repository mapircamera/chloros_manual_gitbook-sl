# Nastavitve projekta

Nastavitve projekta <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> v Chloros vam omogočajo konfiguracijo vseh vidikov obdelave slik, zaznavanja kalibracijskih ciljev, izračunov multispektralnih indeksov in možnosti izvoza za vaš projekt. Te nastavitve se shranijo skupaj z vašim projektom in jih lahko shranite kot predloge za ponovno uporabo v več projektih.

## Dostop do nastavitev projekta

Za dostop do nastavitev projekta:

1. Odprite projekt v Chloros
2. Kliknite na zavihek **Nastavitve projekta**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> na levi stranski vrstici.
3. Na plošči nastavitev se prikažejo vse razpoložljive možnosti konfiguracije, razvrščene po kategorijah.

***

## Zaznavanje ciljev

Te nastavitve nadzirajo, kako Chloros zazna in obdeluje kalibracijske cilje v vaših slikah.

### Minimalna površina kalibracijskega vzorca (px)

* **Vrsta**: Številka
* **Območje**: 0 do 10.000 pikslov
* **Privzeto**: 25 pikslov
* **Opis**: Nastavi minimalno površino (v pikslov), potrebno za to, da se zaznan območje šteje za veljaven kalibracijski ciljni vzorec. Manjše vrednosti bodo zaznale manjše cilje, vendar lahko povečajo število lažnih pozitivnih rezultatov. Večje vrednosti zahtevajo večja, jasnejša ciljna območja za zaznavanje.
* **Kdaj prilagoditi**:
  * Povečajte, če dobivate lažne zaznave na majhnih artefaktih slike.
  * Zmanjšajte, če se vaši kalibracijski cilji na slikah zdijo majhni in se ne zaznavajo.

### Minimalno združevanje ciljev (0–100)

* **Vrsta**: Številka
* **Območje**: 0 do 100
* **Privzeto**: 60
* **Opis**: Nadzira prag združevanja za združevanje podobnih barvnih območij pri zaznavanju kalibracijskih ciljev. Višje vrednosti zahtevajo združevanje bolj podobnih barv, kar ima za posledico bolj konzervativno zaznavanje ciljev. Nižje vrednosti omogočajo večjo barvno variacijo znotraj ciljne skupine.
* **Kdaj prilagoditi**:
  * Povečajte, če se kalibracijske tarče razdelijo na več zaznavanj.
  * Zmanjšajte, če kalibracijske tarče z barvnimi razlikami niso v celoti zaznavane.

***

## Obdelava

Te nastavitve nadzorujejo, kako Chloros obdeluje in kalibrira vaše slike.

### Popravek vinjete

* **Vrsta**: Potrditveno polje
* **Privzeto**: Omogočeno (potrjeno)
* **Opis**: Uporabi popravek vinjete, da kompenzira potemnitev objektiva na robovih slik. Vinjeta je pogost optični pojav, pri katerem so koti in robovi slike zaradi lastnosti objektiva temnejši od sredine.
* **Kdaj onemogočiti**: Onemogočite samo, če je vaša kombinacija kamere/objektiva že uporabila popravek vinjete ali če želite vinjeto ročno popraviti v naknadni obdelavi.

### Kalibracija odbojnosti / bela bilanca

* **Tip**: Potrditveno polje
* **Privzeto**: Omogočeno (potrjeno)
* **Opis**: Omogoča samodejno kalibracijo odbojnosti z uporabo zaznanih kalibracijskih ciljev v vaših slikah. To normalizira vrednosti odbojnosti v vašem nizu podatkov in zagotavlja dosledne meritve ne glede na svetlobne pogoje.
* **Kdaj onemogočiti**: Onemogočite samo, če želite obdelati surove, nekalibrirane slike ali če uporabljate drugačen kalibracijski delovni tok.

### Metoda Debayer

* **Vrsta**: Izbirni seznam
* **Možnosti**:
  * Visoka kakovost (hitrejša) – trenutno edina razpoložljiva možnost
* **Privzeto**: Visoka kakovost (hitrejša)
* **Opis**: Izbere algoritem demosaicinga, ki se uporablja za pretvorbo surovih podatkov senzorja Bayerjevega vzorca v barvne slike. Metoda »Visoka kakovost (hitrejša)« zagotavlja optimalno ravnovesje med hitrostjo obdelave in kakovostjo slike.
* **Opomba**: V prihodnjih različicah Chloros bodo morda dodane dodatne metode debayer.

### Minimalni interval ponovne kalibracije

* **Vrsta**: Številka
* **Območje**: 0 do 3600 sekund
* **Privzeto**: 0 sekund
* **Opis**: Nastavi minimalni časovni interval (v sekundah) med uporabo kalibracijskih ciljev. Če je nastavljeno na 0, bo Chloros uporabil vsak zaznan kalibracijski cilj. Če je nastavljeno na višjo vrednost, bo Chloros uporabil samo kalibracijske cilje, ki so med seboj oddaljeni vsaj toliko sekund, s čimer se skrajša čas obdelave za podatkovne nize s pogostimi zajemi kalibracijskih ciljev.
* **Kdaj prilagoditi**:
  * Nastavite na 0 za največjo natančnost kalibracije, ko se razmere osvetlitve spreminjajo.
  * Povečajte (npr. na 60–300 sekund) za hitrejšo obdelavo, ko je osvetlitev konstantna in imate pogoste slike kalibracijskih ciljev.

### Časovni zamik svetlobnega senzorja

* **Vrsta**: Številka
* **Območje**: od -12 do +12 ur
* **Privzeto**: 0 ur
* **Opis**: Določa časovni zamik (v urah od UTC) za časovne oznake podatkov svetlobnega senzorja. Uporablja se pri obdelavi datotek PPK (Post-Processed Kinematic), da se zagotovi pravilna sinhronizacija časa med zajetimi slikami in podatki GPS.
* **Kdaj prilagoditi**: Nastavite to na časovni zamik vašega lokalnega časovnega pasu, če vaši podatki PPK uporabljajo lokalni čas namesto UTC. Na primer:
  * Pacifiški čas: -8 ali -7 (odvisno od DST)
  * Vzhodni čas: -5 ali -4 (odvisno od DST)
  * Srednjeevropski čas: +1 ali +2 (odvisno od DST)

### Uporabi popravke PPK

* **Vrsta**: Potrditveno polje
* **Privzeto**: Onemogočeno (nepotrjeno)
* **Opis**: Omogoča uporabo popravkov Post-Processed Kinematic (PPK) iz snemalnikov MAPIR DAQ, ki vsebujejo GPS (GNSS). Ko je omogočeno, bo Chloros uporabil vse datoteke dnevnika .daq, ki vsebujejo podatke o izpostavljenosti v vaši projektni mapi, in uporabil natančne popravke geolokacije za vaše slike.
* **Zahteva**: datoteka dnevnika .daq z vnosi izpostavljenosti mora biti prisotna v vaši projektni mapi
* **Kdaj omogočiti**: Priporočljivo je, da vedno omogočite popravke PPK, če imate v datoteki dnevnika .daq vnose povratnih informacij o izpostavljenosti.

### Izpostavljenostni pin 1

* **Tip**: Izbirni seznam
* **Vidnost**: Vidno samo, če je omogočeno »Uporabi PPK popravke« IN so na voljo podatki o izpostavljenosti za pin 1.
* **Možnosti**:
  * Imena modelov kamer, zaznanih v projektu.
  * »Ne uporabljaj« – prezri ta izpostavljenostni pin.
* **Privzeto**: Samodejno izbrano na podlagi konfiguracije projekta.
* **Opis**: Dodeli določeno kamero izpostavljenosti Pin 1 za sinhronizacijo časa PPK. Izpostavljenostni pin zabeleži natančen čas sprožitve zaklopa kamere, kar je ključnega pomena za natančno geografsko lokacijo PPK.
* **Avtomatična izbira**:
  * Ena kamera + en pin: Samodejno izbere kamero.
  * Ena kamera + dva pina: Pin 1 se samodejno dodeli kameri.
  * Več kamer: potrebna ročna izbira

### Izpostavljenost Pin 2

* **Tip**: Izbirni seznam
* **Vidnost**: Vidno samo, če je omogočeno »Uporabi PPK popravke« IN so na voljo podatki o izpostavljenosti za Pin 2
* **Možnosti**:
  * Imena modelov kamer, zaznanih v projektu
  * »Ne uporabljaj« – prezri ta pin izpostavljenosti
* **Privzeto**: Samodejno izbrana na podlagi konfiguracije projekta
* **Opis**: Dodeli določeno kamero izpostavljenosti zatiču 2 za sinhronizacijo časa PPK pri uporabi nastavitve z dvema kamerama.
* **Samodejno izbiranje**:
  * Ena kamera + en zatič: Zatič 2 se samodejno nastavi na »Ne uporabljaj«
  * Ena kamera + dva pina: Pin 2 se samodejno nastavi na „Ne uporabljaj“
  * Več kamer: Potrebna je ročna izbira
* **Opomba**: Ista kamera ne more biti hkrati dodeljena pinu 1 in pinu 2.

***

## Indeks

Te nastavitve omogočajo konfiguracijo multispektralnih indeksov za analizo in vizualizacijo.

### Dodaj indeks

* **Tip**: Poseben panel za konfiguracijo indeksa
* **Opis**: Odpre interaktivni panel, kjer lahko izberete in konfigurirate multispektralne vegetacijske indekse (NDVI, NDRE, EVI itd.), ki se izračunajo med obdelavo slike. Dodate lahko več indeksov, vsak z lastnimi nastavitvami vizualizacije.
* **Razpoložljivi indeksi**: Sistem vključuje več kot 30 vnaprej določenih multispektralnih indeksov, med drugim:
  * NDVI (normalizirani razlikovni vegetacijski indeks)
  * NDRE (normalizirani razlikovni RedEdge)
  * EVI (izboljšani indeks vegetacije)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * In še mnogo več (za popoln seznam glej [Formule večspektralnih indeksov](multispectral-index-formulas.md))
* **Značilnosti**:
  * Izbiranje med vnaprej določenimi formulami indeksov
  * Konfiguriranje barvnih prehodov vizualizacije (LUT – preglednice)
  * Nastavitev mejnih vrednosti za analizo
  * Ustvarjanje lastnih formul indeksov

### Lastne formule (funkcija Chloros+)

* **Vrsta**: Niz definicij lastnih formul
* **Opis**: Omogoča ustvarjanje in shranjevanje lastnih multispektralnih indeksnih formul z uporabo matematičnih operacij s pasovi. Lastne formule se shranijo z nastavitvami projekta in se lahko uporabljajo enako kot vgrajeni indeksi.
* **Kako ustvariti**:
  1. V oknu za konfiguracijo indeksa poiščite možnost za prilagojene formule.
  2. Definirate svojo formulo z uporabo identifikatorjev pasov (npr. NIR, Red, Green, Blue).
  3. Shranite formulo z opisnim imenom.
* **Sintaksa formule**: Podprte so standardne matematične operacije, vključno z:
  * Aritmetika: `+`, `-`, `*`, `/`
  * Oklepaji za vrstni red operacij
  * Sklici na pasove: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Izvoz

Te nastavitve nadzorujejo format in kakovost izvoženih obdelanih slik.

### Kalibrirani format slike

* **Tip**: Izbirni seznam
* **Možnosti**:
  * **TIFF (16-bit)** – Nekomprimiran 16-bitni format TIFF
  * **TIFF (32-bitni, odstotek)** – 32-bitni TIFF s plavajočo vejico z vrednostmi odbojnosti v odstotkih
  * **PNG (8-bitni)** - Stisnjen 8-bitni format PNG
  * **JPG (8-bitni)** - Stisnjen 8-bitni format JPEG
* **Privzeto**: TIFF (16-bitni)
* **Opis**: Izberite format datoteke za shranjevanje obdelanih in kalibriranih slik.
* **Priporočila glede formata**:
  * **TIFF (16-bitni)**: Priporočljiv za znanstvene analize in profesionalne delovne tokove. Ohranja najvišjo kakovost podatkov brez artefaktov stiskanja. Najboljši za multispektralno analizo in nadaljnjo obdelavo v programski opremi GIS.
  * **TIFF (32-bitni, odstotek)**: Najboljši za delovne tokove, ki zahtevajo vrednosti odbojnosti v odstotkih (0–100 %). Nudi največjo natančnost za radiometrične meritve.
  * **PNG (8-bitni)**: Primeren za ogled na spletu in splošno vizualizacijo. Manjše velikosti datotek z brezizgubno stiskanjem, vendar zmanjšan dinamični razpon.
  * **JPG (8-bitni)**: Najmanjše velikosti datotek, najbolj primeren za predoglede in prikaz na spletu. Uporablja izgubno stiskanje, ki ni primerno za znanstvene analize.

***

## Shranjevanje predloge projekta

Ta funkcija vam omogoča, da shranite trenutne nastavitve projekta kot ponovno uporabno predlogo.

* **Vrsta**: Vnos besedila + gumb Shrani
* **Opis**: Vnesite opisno ime za predlogo nastavitev in kliknite ikono Shrani. Predloga bo shranila vse trenutne nastavitve projekta (zaznavanje cilja, možnosti obdelave, indeksi in format izvoza) za enostavno ponovno uporabo v prihodnjih projektih.
* **Primeri uporabe**:
  * Ustvarite predloge za različne kamere (RGB, multispektralne, NIR)
  * Shranite standardne konfiguracije za določene vrste pridelkov ali analitične delovne tokove
  * Delite enotne nastavitve znotraj ekipe
* **Kako uporabljati**:
  1. Konfigurirajte vse želene nastavitve projekta
  2. Vnesite ime predloge (npr. „RedEdge Survey3 NDVI Standard“).
  3. Kliknite ikono za shranjevanje.
  4. Predlogo lahko zdaj naložite pri ustvarjanju novih projektov.

***

## Shranjevanje projektne mape

Ta nastavitev določa, kam se novi projekti privzeto shranijo.

* **Vrsta**: Prikaz poti do imenika + gumb Uredi
* **Privzeto**: `C:\Users\[Username]\Chloros Projects`
* **Opis**: Prikaže trenutni privzeti imenik, v katerem se ustvarjajo novi projekti Chloros. Kliknite ikono za urejanje, da izberete drug imenik.
* **Kdaj spremeniti**:
  * Nastavite na omrežni pogon za sodelovanje v ekipi.
  * Spremenite na pogon z več prostora za shranjevanje za velike podatkovne nize.
  * Organizirajte projekte po letu, stranki ali vrsti projekta v različnih mapah.
* **Opomba**: Sprememba te nastavitve vpliva samo na NOVE projekte. Obstoječi projekti ostanejo na prvotnih mestih.

***

## Trajnost nastavitev

Vse nastavitve projekta se samodejno shranijo z datoteko projekta (format projekta `.mapir`). Ko ponovno odprete projekt, se vse nastavitve obnovijo točno takšne, kot ste jih pustili.

### Hierarhija nastavitev

Nastavitve se uporabljajo v naslednjem vrstnem redu:

1. **Privzete nastavitve sistema** – vgrajene privzete nastavitve, ki jih določi Chloros
2. **Nastavitve predloge** – če pri ustvarjanju projekta naložite predlogo
3. **Shranjene nastavitve projekta** – nastavitve, shranjene s projektno datoteko
4. **Ročne prilagoditve** – vse spremembe, ki jih naredite med trenutno sejo

### Nastavitve in obdelava slik

Večina sprememb nastavitev (zlasti v kategorijah Obdelava in Izvoz) sproži ponovno obdelavo slik, da se upoštevajo nove nastavitve. Nekatere nastavitve pa so »samo za izvoz« in ne zahtevajo takojšnje ponovne obdelave:

* Shrani predlogo projekta
* Delovni imenik
* Kalibrirani format slike (velja pri izvozu)

***

## Najboljše prakse

1. **Začnite s privzetimi nastavitvami**: Privzete nastavitve delujejo dobro za večino sistemov kamer MAPIR in tipičnih delovnih tokov.
2. **Ustvarite predloge**: Ko optimizirate nastavitve za določen delovni tok ali kamero, jih shranite kot predlogo, da zagotovite doslednost med projekti.
3. **Preizkusite pred popolno obdelavo**: Ko preizkušate nove nastavitve, jih preizkusite na majhnem podsklopu slik, preden obdelate celoten niz podatkov.
4. **Dokumentirajte nastavitve**: Uporabite opisna imena predlog, ki navajajo sistem kamere, vrsto obdelave in namembnost (npr. „Survey3\_RGB\_NDVI\_Agriculture“).
5. **Izbira formata izvoza**: Izberite format izvoza glede na končno uporabo:
   * Znanstvena analiza → TIFF (16-bitni ali 32-bitni)
   * Obdelava GIS → TIFF (16-bitni)
   * Hitra vizualizacija → PNG (8-bitni)
   * Spletno deljenje → JPG (8-bitni)

***

Za več informacij o multispektralnih indeksih v Chloros glejte stran [Formule multispektralnih indeksov](multispectral-index-formulas.md).

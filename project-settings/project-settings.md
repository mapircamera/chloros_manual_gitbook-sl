# Nastavitve projekta

V stranskem meniju »Nastavitve projekta« <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> v Chloros vam omogočajo konfiguracijo vseh vidikov obdelave slik, zaznavanja kalibracijskih ciljev, izračunov multispektralnih indeksov in možnosti izvoza za vaš projekt. Te nastavitve se shranijo skupaj z vašim projektom in jih lahko shranite kot predloge za ponovno uporabo v več projektih.

## Dostop do nastavitev projekta

Za dostop do nastavitev projekta:

1. Odprite projekt v Chloros
2. Kliknite zavihek **Nastavitve projekta**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> v levem stranskem meniju
3. Na plošči z nastavitvami se prikažejo vse razpoložljive možnosti konfiguracije, razvrščene po kategorijah

***

## Prepoznavanje ciljev

Te nastavitve nadzorujejo, kako Chloros prepoznava in obdeluje kalibracijske cilje v vaših slikah.

### Najmanjša površina kalibracijskega vzorca (px)

* **Vrsta**: Številka
* **Območje**: od 0 do 10.000 pikslov
* **Privzeto**: 25 pikslov
* **Opis**: Določa najmanjšo površino (v pikslov), ki je potrebna, da se zaznano območje šteje za veljaven kalibracijski vzorec. Manjše vrednosti bodo zaznale manjše cilje, vendar lahko povečajo število lažnih pozitivnih rezultatov. Večje vrednosti zahtevajo večje in jasnejše ciljne regije za zaznavanje.
* **Kdaj prilagoditi**:
  * Povečajte, če dobivate napačne zaznave na majhnih artefaktih slike
  * Zmanjšajte, če se vaši kalibracijski cilji na slikah zdijo majhni in se ne zaznavajo

### Minimalno združevanje ciljev (0–100)

* **Tip**: Številka
* **Območje**: 0 do 100
* **Privzeto**: 60
* **Opis**: Nadzira prag združevanja za razvrščanje podobnih barvnih območij v skupine pri zaznavanju kalibracijskih ciljev. Višje vrednosti zahtevajo, da se v skupino združi več podobnih barv, kar ima za posledico bolj konzervativno zaznavanje ciljev. Nižje vrednosti omogočajo večje barvne razlike znotraj ciljne skupine.
* **Kdaj prilagoditi**:
  * Povečajte, če se kalibracijske tarče razdelijo na več zaznav
  * Zmanjšajte, če se kalibracijske tarče z barvnimi odstopanji ne zaznavajo v celoti

***

## Obdelava

Te nastavitve nadzorujejo, kako Chloros obdeluje in kalibrira vaše slike.

### Popravek vinjetiranja

* **Vrsta**: Potrditveno polje
* **Privzeto**: Omogočeno (označeno)
* **Opis**: Uporabi popravek vinjetiranja, da kompenzira potemnitev objektiva na robovih slik. Vinjetiranje je pogost optični pojav, pri katerem so koti in robovi slike zaradi lastnosti objektiva temnejši od sredine.
* **Kdaj onemogočiti**: Onemogočite le, če je vaša kombinacija kamere/objektiva že uporabila popravek vinjetiranja ali če želite vinjetiranje popraviti ročno v naknadni obdelavi.

### Kalibracija odbojnosti / bela barva

* **Vrsta**: Potrditveno polje
* **Privzeto**: Omogočeno (označeno)
* **Opis**: Omogoča samodejno kalibracijo odbojnosti z uporabo zaznanih kalibracijskih ciljev v vaših slikah. To normalizira vrednosti odbojnosti v vašem nizu podatkov in zagotavlja dosledne meritve ne glede na svetlobne pogoje.
* **Kdaj onemogočiti**: Onemogočite le, če želite obdelati surove, nekalibrirane slike ali če uporabljate drugačen kalibracijski potek dela.

### Metoda debayeringa

* **Vrsta**: Izbirni seznam
* **Možnosti**:
  * Standardno (hitro, srednja kakovost)
  * Upoštevanje teksture (počasno, najvišja kakovost) \[Chloros+]
* **Privzeto**: Standardno (hitro, srednja kakovost)
* **Opis**: Izbere algoritem demosaicinga, ki se uporablja za pretvorbo surovih podatkov senzorja z vzorcem Bayer v barvne slike. Metoda »Standardno (hitro, srednja kakovost)« zagotavlja optimalno ravnovesje med hitrostjo obdelave in kakovostjo slike. Metoda »Texture Aware (počasna, najvišja kakovost)« \[Chloros+] uporablja visokokakovostno demosaicing z upoštevanjem robov v kombinaciji z modelom za odstranjevanje šuma AI/ML, ki odstrani skoraj ves šum demosaicinga. Model Texture Aware za delovanje potrebuje pomnilnik GPU (VRAM). Priporočamo, da ga uporabljate, če imate na voljo več kot 4 GB VRAM za hitrejšo obdelavo.
* **Opomba**: V prihodnjih različicah Chloros bodo morda dodane dodatne metode debayeringa.

### Minimalni interval ponovne kalibracije

* **Vrsta**: Številka
* **Območje**: 0 do 3.600 sekund
* **Privzeto**: 0 sekund
* **Opis**: Nastavi najkrajši časovni interval (v sekundah) med uporabo kalibracijskih ciljev. Če je nastavljen na 0, bo Chloros uporabil vsak zaznan kalibracijski cilj. Če je nastavljen na višjo vrednost, bo Chloros uporabil le kalibracijske cilje, ki so med seboj ločeni vsaj za toliko sekund, s čimer se skrajša čas obdelave za podatkovne nize s pogostim zajemanjem kalibracijskih ciljev.
* **Kdaj prilagoditi**:
  * Nastavite na 0 za največjo natančnost kalibracije, ko se svetlobne razmere spreminjajo
  * Povečajte (npr. na 60–300 sekund) za hitrejšo obdelavo, ko je osvetlitev konstantna in imate pogoste slike kalibracijskih ciljev

### Časovni zamik svetlobnega senzorja

* **Vrsta**: Številka
* **Območje**: od -12 do +12 ur
* **Privzeto**: 0 ur
* **Opis**: Določa časovni zamik (v urah od UTC) za časovne oznake podatkov svetlobnega senzorja. To se uporablja pri obdelavi datotek PPK (Post-Processed Kinematic), da se zagotovi pravilna časovna sinhronizacija med zajetimi slikami in podatki GPS.
* **Kdaj prilagoditi**: To nastavite na časovni zamik vašega lokalnega časovnega pasu, če vaši PPK podatki uporabljajo lokalni čas namesto UTC. Na primer:
  * Pacifiški čas: -8 ali -7 (odvisno od poletnega časa)
  * Vzhodni čas: -5 ali -4 (odvisno od poletnega časa)
  * Srednjeevropski čas: +1 ali +2 (odvisno od poletnega časa)

### Uporabi PPK popravke

* **Vrsta**: Potrditveno polje
* **Privzeto**: Onemogočeno (nepotrjeno)
* **Opis**: Omogoča uporabo popravkov Post-Processed Kinematic (PPK) iz snemalnikov MAPIR DAQ, ki vsebujejo GPS (GNSS). Ko je omogočeno, bo Chloros uporabil vse datoteke dnevnika .daq, ki vsebujejo podatke o izpostavljenosti v vašem projektnem imeniku, in na vaše slike uporabil natančne popravke geografske lokacije.
* **Zahteva**: Datoteka dnevnika .daq z vnosi o izpostavljenosti mora biti prisotna v vašem projektnem imeniku
* **Kdaj omogočiti**: Priporočljivo je, da vedno omogočite PPK popravke, če imate v datoteki dnevnika .daq vnose o povratnih informacijah o izpostavljenosti.

### Izpostavljenostni pin 1

* **Vrsta**: Izbirni seznam
* **Vidnost**: Vidno le, če je omogočena možnost »Uporabi PPK popravke« IN so na voljo podatki o izpostavljenosti za pin 1
* **Možnosti**:
  * Imena modelov kamer, zaznanih v projektu
  * »Ne uporabi« – Ta izpostavljenostni pin se prezre
* **Privzeto**: Samodejno izbrano na podlagi konfiguracije projekta
* **Opis**: Dodeli določeno kamero izhodu za osvetlitev 1 za časovno sinhronizacijo PPK. Izhod za osvetlitev beleži natančen čas sprožitve zaklopa kamere, kar je ključnega pomena za natančno geolokacijo PPK.
* **Delovanje samodejnega izbora**:
  * Ena kamera + en izhod: Samodejno izbere kamero
  * Ena kamera + dva izhoda: Izhod 1 se samodejno dodeli kameri
  * Več kamer: Potrebna je ročna izbira

### Izpostavljenostni pin 2

* **Vrsta**: Izbirni seznam
* **Vidnost**: Vidno le, če je omogočena možnost »Uporabi PPK popravke« IN so na voljo podatki o izpostavljenosti za pin 2
* **Možnosti**:
  * Imena modelov kamer, zaznanih v projektu
  * »Ne uporabljaj« – Ta izpostavljenostni pin se prezre
* **Privzeto**: Samodejno izbrano na podlagi konfiguracije projekta
* **Opis**: Dodeli določeno kamero priključku za osvetlitev 2 za časovno sinhronizacijo PPK pri uporabi konfiguracije z dvema kamerama.
* **Delovanje samodejnega izbora**:
  * Ena kamera + en priključek: Priključek 2 je samodejno nastavljen na »Ne uporabljaj«
  * Ena kamera + dva priključka: Priključek 2 je samodejno nastavljen na »Ne uporabljaj«
  * Več kamer: Potrebna je ročna izbira
* **Opomba**: Iste kamere ni mogoče hkrati dodeliti na priključek 1 in priključek 2.***

## Indeks

Te nastavitve omogočajo konfiguracijo multispektralnih indeksov za analizo in vizualizacijo.

### Dodaj indeks

* **Vrsta**: Posebni konfiguracijski panel za indekse
* **Opis**: Odpre interaktivni panel, kjer lahko izberete in konfigurirate multispektralne indekse vegetacije (NDVI, NDRE, EVI itd.), ki se izračunajo med obdelavo slike. Dodate lahko več indeksov, vsak z lastnimi nastavitvami vizualizacije.
* **Razpoložljivi indeksi**: Sistem vključuje več kot 30 vnaprej določenih multispektralnih indeksov, med drugim:
  * NDVI (Normalizirani razlikovni vegetacijski indeks)
  * NDRE (Normalizirani razlikovni RedEdge)
  * EVI (izboljšani indeks vegetacije)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * In še veliko več (za popoln seznam glej [Formule multispektralnih indeksov](multispectral-index-formulas.md))
* **Značilnosti**:
  * Izbirajte med vnaprej določenimi formulami indeksov
  * Konfigurirajte barvne prehode vizualizacije (LUT – iskalne tabele)
  * Nastavite mejne vrednosti za analizo
  * Ustvarite lastne formule indeksov

### Lastne formule (funkcija Chloros+)

* **Vrsta**: Niz definicij lastnih formul
* **Opis**: Omogoča ustvarjanje in shranjevanje lastnih multispektralnih indeksnih formul z uporabo matematičnih operacij na pasovih. Lastne formule se shranijo skupaj z nastavitvami projekta in jih je mogoče uporabljati enako kot vgrajene indekse.
* **Kako ustvariti**:
  1. V oknu za konfiguracijo indeksov poiščite možnost za lastne formule
  2. Definirajte formulo z uporabo identifikatorjev pasov (npr. NIR, Red, Green, Blue)
  3. Shranite formulo z opisnim imenom
* **Sintaksa formule**: Podprte so standardne matematične operacije, vključno z:
  * Aritmetika: `+`, `-`, `*`, `/`
  * Oklepaji za vrstni red operacij
  * Sklici na pasove: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Izvoz

Te nastavitve nadzorujejo format in kakovost izvoženih obdelanih slik.

### Kalibrirani format slike

* **Vrsta**: Izbirni seznam
* **Možnosti**:
  * **TIFF (16-bitni)** – Nekomprimiran 16-bitni format TIFF
  * **TIFF (32-bitni, odstotek)** – 32-bitni format TIFF s plavajočo vejico in vrednostmi odbojnosti v odstotkih
  * **PNG (8-bitni)** - Stisnjen 8-bitni format PNG
  * **JPG (8-bitni)** - Stisnjen 8-bitni format JPEG
* **Privzeto**: TIFF (16-bitni)
* **Opis**: Izbere format datoteke za shranjevanje obdelanih in kalibriranih slik.
* **Priporočila glede formata**:
  * **TIFF (16-bitni)**: Priporočljiv za znanstvene analize in profesionalne delovne tokove. Ohranja najvišjo kakovost podatkov brez artefaktov stiskanja. Najboljši za multispektralno analizo in nadaljnjo obdelavo v programski opremi GIS.
  * **TIFF (32-bitni, odstotek)**: Najboljši za delovne tokove, ki zahtevajo vrednosti odbojnosti v odstotkih (0–100 %). Nudi največjo natančnost za radiometrične meritve.
  * **PNG (8-bitni)**: Primeren za ogled na spletu in splošno vizualizacijo. Manjše datoteke z brezizgubno stiskanjem, vendar zmanjšan dinamični razpon.
  * **JPG (8-bit)**: Najmanjše datoteke, najboljše le za predoglede in prikazovanje na spletu. Uporablja izgubno stiskanje, ki ni primerno za znanstvene analize.***

## Shrani predlogo projekta

Ta funkcija vam omogoča, da shranite trenutne nastavitve projekta kot ponovno uporabno predlogo.

* **Vrsta**: Vnos besedila + gumb Shrani
* **Opis**: Vnesite opisno ime za predlogo nastavitev in kliknite ikono za shranjevanje. Predloga bo shranila vse trenutne nastavitve projekta (azaznavanje ciljev, možnosti obdelave, indeksi in format izvoza) za enostavno ponovno uporabo v prihodnjih projektih.
* **Primeri uporabe**:
  * Ustvarjanje predlog za različne kamere (RGB, multispektralne, NIR)
  * Shranjevanje standardnih konfiguracij za določene vrste pridelkov ali analitične delovne tokove
  * Deljenje enotnih nastavitev znotraj ekipe
* **Kako uporabljati**:
  1. Konfigurirajte vse želene nastavitve projekta
  2. Vnesite ime predloge (npr. »RedEdge Survey3 NDVI Standard«)
  3. Kliknite ikono za shranjevanje
  4. Predlogo je zdaj mogoče naložiti pri ustvarjanju novih projektov

***

## Mapa za shranjevanje projektov

Ta nastavitev določa, kam se novi projekti privzeto shranjujejo.

* **Vrsta**: Prikaz poti do mape + gumb za urejanje
* **Privzeto (Windows)**: `C:\Users\[Username]\Chloros Projects`
* **Privzeto (Linux)**: `~/.local/share/chloros/projects`
* **Opis**: Prikaže trenutni privzeti imenik, v katerem se ustvarjajo novi projekti Chloros. Kliknite ikono za urejanje, da izberete drug imenik.
* **Kdaj spremeniti**:
  * Nastavite na omrežni pogon za sodelovanje v ekipi
  * Spremenite na pogon z več prostora za shranjevanje za velike podatkovne nize
  * Razvrstite projekte po letu, naročniku ali vrsti projekta v različne mape
* **Opomba**: Sprememba te nastavitve vpliva le na NOVE projekte. Obstoječi projekti ostanejo na prvotnih lokacijah.***

## Trajnost nastavitev

Vse nastavitve projekta se samodejno shranijo skupaj z datoteko projekta (format projekta `.mapir`). Ko ponovno odprete projekt, se vse nastavitve obnovijo točno takšne, kot ste jih zapustili.

### Hierarhija nastavitev

Nastavitve se uporabljajo v naslednjem vrstnem redu:

1. **Sistemski privzeti vrednosti** – vgrajene privzete vrednosti, ki jih določa Chloros
2. **Nastavitve predloge** – če pri ustvarjanju projekta naložite predlogo
3. **Shranjene nastavitve projekta** – nastavitve, shranjene skupaj s projektno datoteko
4. **Ročne prilagoditve** – Vse spremembe, ki jih naredite med trenutno sejo

### Nastavitve in obdelava slik

Večina sprememb nastavitev (zlasti v kategorijah Obdelava in Izvoz) bo sprožila ponovno obdelavo slik, da se upoštevajo nove nastavitve. Vendar so nekatere nastavitve »samo za izvoz« in ne zahtevajo takojšnje ponovne obdelave:

* Shrani predlogo projekta
* Delovni imenik
* Kalibrirani format slike (velja pri izvozu)

***

## Najboljše prakse

1. **Začnite s privzetimi nastavitvami**: Privzete nastavitve delujejo dobro za večino sistemov kamer MAPIR in tipičnih delovnih tokov.
2. **Ustvarite predloge**: Ko optimizirate nastavitve za določen delovni tok ali kamero, jih shranite kot predlogo, da zagotovite doslednost med projekti.
3. **Preizkusite pred celotno obdelavo**: Ko eksperimentirate z novimi nastavitvami, jih preizkusite na majhnem podsklopu slik, preden obdelate celoten niz podatkov.
4. **Dokumentirajte svoje nastavitve**: Uporabite opisna imena predlog, ki navajajo sistem kamere, vrsto obdelave in namembno uporabo (npr. »Survey3\_RGB\_NDVI\_Agriculture«).
5. **Izbor formata izvoza**: Izberite format izvoza glede na končno uporabo:
   * Znanstvena analiza → TIFF (16-bitni ali 32-bitni)
   * Obdelava GIS → TIFF (16-bitni)
   * Hitra vizualizacija → PNG (8-bitni)
   * Deljenje na spletu → JPG (8-bitni)

***

Za več informacij o multispektralnih indeksih v Chloros glejte stran [Formule multispektralnih indeksov](multispectral-index-formulas.md).

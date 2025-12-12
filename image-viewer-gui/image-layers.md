# Sloji slike

Spustni meni Sloji slike v pregledovalniku slik Chloros vam omogoča hitro preklapljanje med različnimi različicami iste slike – od originalnih posnetkov do obdelanih izhodnih odbojnih slik in izračunanih indeksnih slik.

## Kaj so sloji slike?

V Chloros se **plasti** nanašajo na različne izhodne slike, ki so na voljo za eno izvorno sliko. Ko obdelujete slike, Chloros ustvari več različic:

* **Izvorne slike** (datoteke JPG in RAW iz vaše kamere)
* **Kalibrirani izhodi odbojnosti** (če je bila omogočena kalibracija odbojnosti)
* **Ciljne slike** (če slika vsebuje kalibracijske cilje)
* **Indeksne slike** (NDVI, NDRE, GNDVI itd., če so bili konfigurirani indeksi)

**Spustni meni za izbiro plasti** v zgornjem desnem kotu pregledovalnika slik vam omogoča takojšnje preklapljanje med temi različicami, ne da bi zapustili pregledovalnik.

***

## Razpoložljivi tipi plasti

### JPG

* Izvirna predogledna slika JPG iz vaše kamere
* Vedno na voljo za vse slike
* Nepredelana, kot jo je zajela kamera
* Najhitrejša za nalaganje in prikazovanje

**Kdaj si jo ogledati:**

* Hitri predogled originalnega posnetka
* Preverjanje kompozicije in kadriranja slike
* Preverjanje kakovosti posnetka pred obdelavo

### RAW (original)

* Originalni podatki senzorja RAW iz vaše kamere
* Brez post-obdelave
* Višja bitna globina kot JPG (običajno 12-bitni ali 14-bitni podatki senzorja)

**Kdaj si ogledati:**

* Preverjanje kakovosti originalnih podatkov senzorja
* Preverjanje težav s senzorjem ali artefaktov
* Primerjava rezultatov pred in po obdelavi

### RAW (cilj)

* Prikaže se samo za slike, ki vsebujejo kalibracijske cilje
* Prikaže originalno RAW sliko z zaznanim ciljem
* Uporablja se za preverjanje uspešnosti zaznavanja cilja

**Kdaj si ogledati:**

* Potrditev, da so bili kalibracijski cilji pravilno zaznan
* Preverjanje kakovosti ciljne slike
* Odpravljanje težav s kalibracijo

{% hint style=&quot;info&quot; %}
**Ciljna plast**: Ta plast se prikaže le v spustnem meniju za slike, ki vsebujejo kalibracijske cilje. Običajne zajete slike te možnosti nimajo.
{% endhint %}

### RAW (odbojnost)

* Kalibrirana slika odbojnosti
* Popravljena vinjeta (če je omogočena v obdelavi)
* Odbojnost kalibrirana z uporabo ciljnih podatkov (če je omogočeno)
* Večpasovni TIFF z vsemi kanali kamere
* Vrednosti pikslov predstavljajo odstotek odbojnosti (pri uporabi odstotkovnega načina)
* Pripravljen za manipulacijo z [Index/LUT Sandbox](index-lut-sandbox.md)

**Kdaj si ogledati:**

* Pregled kalibriranih rezultatov
* Preverjanje kakovosti kalibracije
* Preverjanje vrednosti pikslov za znanstveno natančnost
* Primerjava z originalom za pregled učinkov kalibracije

{% hint style=&quot;success&quot; %}
**Priporočeno**: Pri preverjanju vrednosti pikslov za znanstvene meritve in analize uporabite sloj RAW (odbojnost).
{% endhint %}

### RAW (NDVI indeks)... in podobno

* Izračunana slika vegetacijskega indeksa (v tem primeru NDVI)
* Ime indeksa se spremeni glede na to, kateri indeks je bil konfiguriran med obdelavo
* Primeri: RAW (NDVI indeks), RAW (NDRE indeks), RAW (GNDVI indeks) itd.
* Enopasovna siva slika, ki prikazuje rezultate izračuna indeksa
* Za vsak indeks, konfiguriran v nastavitvah projekta, se prikaže en sloj

**Možna imena indeksov:**

* RAW (NDVI indeks)
* RAW (NDRE indeks)
* RAW (GNDVI indeks)
* RAW (OSAVI indeks)
* RAW (EVI indeks)
* RAW (SAVI indeks)
* In še mnogi drugi... (glej [Formule za multispektralne indekse](../project-settings/multispectral-index-formulas.md))

**Kdaj si jih ogledati:**

* Preverjanje rezultatov izračuna indeksa
* Preverjanje razponov vrednosti indeksa
* Identificiranje območij, ki vas zanimajo
* Preverjanje slik indeksa pred uporabo v GIS ali analizi

***

## Uporaba izbirnika plasti

### Odpiranje spustnega menija

1. Odprite sliko v načinu polnega zaslona (kliknite katero koli sličico v pregledovalniku slik).
2. Poiščite **spustni meni plasti** v zgornjem desnem kotu pregledovalnika.
3. Spustni meni prikazuje trenutno izbrano plast (npr. „JPG”)
4. Kliknite spustni meni, da si ogledate vse razpoložljive plasti

### Preklapljanje med plastmi

1. Kliknite spustni meni plasti, da odprete seznam
2. Prikazane so vse razpoložljive plasti za trenutno sliko
3. Kliknite katero koli ime plasti, da preklopite na to različico
4. Slika se takoj posodobi in prikaže izbrano plast

**Hitro preklapljanje:**

* Spustni meni si zapomni vašo zadnjo izbiro.
* Pri prehodu na naslednjo sliko Chloros poskuša prikazati isti tip plasti.
* Če ta plast na naslednji sliki ne obstaja, se privzeto prikaže JPG.

### Razpoložljivost plasti

Vse plasti niso na voljo za vsako sliko:

**Vedno na voljo:**

* ✅ JPG (vsaka slika ima predogled JPG)

**Pogojno na voljo:**

* ⚠️ RAW (izvirnik) – samo če je bila slika zajeta v načinu RAW ali RAW+JPG
* ⚠️ RAW (cilj) – samo če slika vsebuje zaznavne kalibracijske cilje
* ⚠️ RAW (odbojnost) – samo po obdelavi z omogočeno kalibracijo odbojnosti
* ⚠️ RAW (\[Index] Index) – samo po obdelavi s konfiguriranimi indeksi

***

## Obstojnost plasti

### Navigacija med slikami

Ko preidete na drugo sliko (z uporabo puščičnih tipk ali klikom na miniaturne slike):

**Nastavitev plasti se ohrani:**

* Če si ogledujete »RAW (odbojnost)«, naslednja slika prikazuje »RAW (odbojnost)« (če je na voljo)
* Če si ogledujete »RAW (NDVI Indeks)«, naslednja slika prikaže »RAW (NDVI Indeks)« (če je na voljo)
* Če isti sloj ne obstaja, se privzeto prikaže JPG

**Primer poteka dela:**

1. Odprite sliko 1, preklopite na RAW (NDVI Index)
2. Pritisnite →, da si ogledate sliko 2.
3. Slika 2 samodejno prikaže sloj RAW (NDVI Index).
4. Nadaljujte z navigacijo – vse slike prikazujejo sloj NDVI.
5. Zelo učinkovito za pregledovanje rezultatov indeksa na več slikah.

***

## Pogosti delovni tokovi

### Delovni tok 1: Primerjava pred/po

**Cilj**: Primerjava originalne in kalibrirane slike

1. Odprite obdelano sliko v pregledovalniku slik.
2. Iz spustnega menija izberite **RAW (Original)**.
3. Upoštevajte vinjetiranje in nekalibrirane vrednosti.
4. Iz spustnega menija preklopite na **RAW (Reflectance)**.
5. Primerjava – vinjetiranje odstranjeno, vrednosti kalibrirane.

### Delovni tok 2: Pregled indeksa

**Cilj**: Hitri pregled rezultatov NDVI v podatkovni zbirki.

1. Odprite prvo obdelano sliko.
2. Iz spustnega menija izberite **RAW (NDVI Index)**.
3. Uporabite puščično tipko → za prehod na naslednjo sliko
4. Plast NDVI ostane samodejno
5. Nadaljujte z vsemi slikami in preverite vzorce NDVI
6. Preklopite na **RAW (NDRE Index)** za primerjavo

### Delovni tok 3: Preverjanje cilja

**Cilj**: Preverite, ali so bile vse ciljne slike pravilno zaznale.

1. Preidite na ciljno sliko.
2. Iz spustnega menija izberite **RAW (cilj)**.
3. Preverite, ali so kalibracijske ciljne točke jasno vidne in zaznale.
4. Preidite na naslednjo ciljno sliko.
5. Ponovite preverjanje za vse ciljne točke.

### Delovni tok 4: Pregled vrednosti pikslov

**Cilj**: Preverite vrednosti odbojnosti za znanstveno natančnost.

1. Odprite obdelano sliko.
2. Izberite sloj **RAW (odbojnost)**.
3. Vklopite način **Pixel Percent** (gumb v zgornjem desnem orodnem pasu).
4. Premaknite kurzor nad območja vegetacije.
5. Preverite, ali so vrednosti pik v pričakovanih razponih (30–70 % za NIR, 5–15 % za Red).
6. Preverite, ali so vrednosti za območja tal in vode ustrezne.

***

## Razumevanje vrednosti pik po slojih

Različni sloji prikazujejo različne razpone vrednosti pik:

### Sloj JPG

* **Območje**: 0–255 (8-bitno)
* **Pomen**: prikaz vrednosti, popravljeno z gama korekcijo
* **Uporaba**: samo vizualni pregled, ne za znanstvene meritve

### RAW (izvirno)

* **Območje**: 0–65535 (16-bitno)
* **Pomen**: surove digitalne številke senzorja
* **Uporaba**: preverjanje delovanja senzorja, ni kalibrirano

### RAW (odbojnost)

* **Območje**: 0–65.535 (16-bitni TIFF) ali 0,0–1,0 (32-bitni odstotek)
* **Pomen**: Kalibrirani odstotek odbojnosti
* **Uporaba**: Znanstvene meritve in analize

**Za 16-bitni TIFF:** Delite z 65.535, da dobite odstotek odbojnosti **Za 32-bitni odstotek:** Vrednosti neposredno predstavljajo odstotek (0,5 = 50 % odbojnosti)

### RAW (indeksne slike)

* **Območje**: razlikuje se glede na indeks (običajno od -1,0 do +1,0 za normalizirane indekse)
* **Pomen**: rezultat izračuna indeksa
* **Primeri**:
  * NDVI: od -1 do +1 (vegetacija običajno od 0,4 do 0,9)
  * NDRE: od -1 do +1 (zaznavanje stresa)
  * EVI: od 0 do 1 (izboljšana vegetacija)

***

## Nasveti in najboljše prakse

### Učinkovito preklapljanje med sloji

* **Poznavanje bližnjic na tipkovnici**: Čeprav za plasti ni bližnjic na tipkovnici, puščice za navigacijo (←/→) delujejo na vseh plasteh
* **Dosledni delovni tokovi**: Izberite eno plast (npr. NDVI) in preglejte celoten niz podatkov, preden preklopite na drugo
* **Hitra primerjava**: Preklopite med Original in Reflectance, da preverite kakovost obdelave

### Upoštevanje zmogljivosti

* **JPG se nalaga najhitreje**: uporabite ga za hitro navigacijo med številnimi slikami.
* **RAW sloji se nalagajo počasneje**: višja ločljivost in bitna globina.
* **Indeksni sloji**: podobna hitrost kot sloji odbojnosti.
* **Prvo nalaganje je najpočasnejše**: nadaljnji ogledi istega sloja se shranijo v predpomnilnik in so hitrejši.

### Preverjanje kakovosti

* **Vedno preverite RAW (original)**: Preverite kakovost izvornih podatkov, preden zaupate obdelanim izhodnim podatkom.
* **Primerjajte plasti**: uporabite preklapljanje plasti, da preverite, ali je obdelava potekala pravilno.
* **Preverite indeksna območja**: uporabite način Pixel Percent z indeksnimi plastmi, da preverite, ali so vrednosti razumne.

***

## Odpravljanje težav

### Plast ni na voljo

**Težava**: pričakovana plast se ne prikaže v spustnem meniju.

**Možni vzroki:**

* Slika ni bila obdelana (na voljo sta samo JPG in RAW (Original)).
* Kalibracija odbojnosti je bila med obdelavo onemogočena.
* Posebni indeks ni bil konfiguriran v nastavitvah projekta.
* Slika je slika, namenjena samo ciljem (za cilje niso bili ustvarjeni indeksi).

**Rešitve:**

1. Preverite, ali je bila slika obdelana (preverite mapo z izhodnimi datotekami za obdelane datoteke).
2. Preverite nastavitve projekta, da se prepričate, da so bili indeksi konfigurirani.
3. Ponovno obdelajte sliko z omogočenimi želenimi indeksi.

### Prikazana napačna plast

**Problem**: Slika se odpre v nepričakovani plasti.

**Vzrok**: Nastavitve plasti iz prejšnje slike so bile prenesene, vendar ta plast ne obstaja na trenutni sliki.

**Rešitev**: Chloros samodejno preide na JPG, ko želeno plast ni na voljo – to je normalno delovanje.

### Kalibracijski cilji niso vidni

**Problem**: Sloj RAW (cilj) ne prikaže zaznavanja ciljev.

**Možni vzroki:**

* Cilji niso bili zaznani med obdelavo.
* Slika dejansko ne vsebuje ciljev.
* Nastavitve zaznavanja ciljev so preveč stroge.

**Rešitve:**

1. V dnevniku odpravljanja napak preverite, ali so prisotna sporočila »Cilj najden«.
2. Preverite, ali slika dejansko vsebuje vidne kalibracijske cilje.
3. Prilagodite nastavitve zaznavanja ciljev v nastavitvah projekta.
4. Glejte [Izbira ciljnih slik](../processing-images-gui/choosing-target-images.md).

***

## Sorodne funkcije

### Orodja za pregledovanje slik

Pri pregledovanju katere koli plasti lahko uporabite:

* **Nadzorne funkcije povečave**: povečajte sliko, da pregledate podrobnosti.
* **Premikanje**: kliknite in povlecite, da se premikate po povečani sliki.
* **Pregled vrednosti pikslov**: oglejte si vrednosti na mestu kazalca.
* **Navigacijske puščice**: premikajte se med slikami, pri čemer ohranite plast.
* **Način odstotka pikslov**: preklopite med prikazom DN in odstotkom.

Glejte [Odpiranje slike v polnem zaslonu](opening-an-image-full-screen.md) za popolno dokumentacijo pregledovalnika slik.

### Indeks/LUT Sandbox

Za interaktivno testiranje indeksa in vizualizacijo:

* **Izračun indeksa v realnem času**: testirajte različne formule indeksa
* **Barvno preslikavanje LUT**: uporabite barvne prehode za indekse v sivih odtenkih
* **Izvoz vizualizacij**: shranite barvne slike indeksa

Za podrobnosti glejte [Indeks/LUT Sandbox](index-lut-sandbox.md).

***

## Naslednji koraki

Sedaj, ko razumete plasti slik:

* [**Odpiranje slike v polnem zaslonu**](opening-an-image-full-screen.md) – Celoten vodnik za pregledovalnik slik
* [**Indeks/LUT Sandbox**](index-lut-sandbox.md) – Interaktivna vizualizacija indeksa
* [**Formule za multispektralni indeks**](../project-settings/multispectral-index-formulas.md) – Referenca razpoložljivih indeksov
* [**Zaključek obdelave**](../processing-images-gui/finishing-the-processing.md) – Razumevanje obdelanih izhodov

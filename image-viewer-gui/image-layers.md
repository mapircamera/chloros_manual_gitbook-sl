# Slike v plasteh

Spustni meni »Slike v plasteh« v pregledovalniku slik Chloros omogoča hitro preklapljanje med različnimi različicami iste slike – od izvirnih posnetkov do obdelanih izhodnih podatkov o odbojnosti in izračunanih indeksnih slik.

## Kaj so slike v plasteh?

V programu Chloros se **plasti** nanašajo na različne izhodne slike, ki so na voljo za eno izvorno sliko. Ko obdelujete slike, program Chloros ustvari več različic:

* **Izvorne slike** (datoteke JPG in RAW iz vaše kamere)
* **Izhodi s kalibrirano odbojnostjo** (če je bila omogočena kalibracija odbojnosti)
* **Ciljne slike** (če slika vsebuje kalibracijske cilje)
* **Indeksne slike** (NDVI, NDRE, GNDVI itd., če so bili indeksi konfigurirani)**Spustni meni za izbiro plasti** v zgornjem desnem kotu pregledovalnika slik vam omogoča takojšnje preklapljanje med temi različicami, ne da bi morali zapustiti pregledovalnik.***

## Razpoložljive vrste plasti

### JPG

* Izvirna predogledna slika JPG iz vaše kamere
* Vedno na voljo za vse slike
* Nepredelana, takšna, kot jo je zajela kamera
* Najhitrejša za nalaganje in prikaz

**Kdaj si ogledati:**

* Hitri predogled izvirnega posnetka
* Preverjanje kompozicije in kadriranja slike
* Preverjanje kakovosti posnetka pred obdelavo

### RAW (izvirnik)

* Izvirni podatki senzorja RAW iz vaše kamere
* Debayered brez uporabljene naknadne obdelave
* Višja bitna globina kot pri JPG (tipično 12-bitni ali 14-bitni podatki senzorja)

**Kdaj si ogledati:**

* Pregled kakovosti izvirnih podatkov senzorja
* Preverjanje težav s senzorjem ali artefaktov
* Primerjava rezultatov pred in po obdelavi

### RAW (Cilj)

* Prikaže se le za slike, ki vsebujejo kalibracijske cilje
* Prikaže izvirno sliko RAW z zaznanim ciljem
* Uporablja se za preverjanje, ali je bilo zaznavanje cilja uspešno

**Kdaj si ogledati:**

* Potrditev, da so bili kalibracijski cilji pravilno zaznani
* Preverjanje kakovosti slike cilja
* Odpravljanje težav s kalibracijo

{% hint style="info" %}
**Sloj cilja**: Ta sloj se prikaže v spustnem meniju le za slike, ki vsebujejo kalibracijske cilje. Običajne posnete slike te možnosti nimajo.
{% endhint %}

### RAW (odbojnost)

* Kalibrirana izhodna slika odbojnosti
* Popravljena vinjeta (če je omogočena v obdelavi)
* Odbojnost kalibrirana z uporabo ciljnih podatkov (če je omogočeno)
* Večpasovni TIFF z vsemi kanali kamere
* Vrednosti pik predstavljajo odstotek odbojnosti (pri uporabi odstotnega načina)
* Pripravljeno za obdelavo z [Index/LUT Sandbox](index-lut-sandbox.md)

**Kdaj si ogledati:**

* Pregled kalibriranih rezultatov
* Preverjanje kakovosti kalibracije
* Preverjanje vrednosti pikslov za znanstveno natančnost
* Primerjava z izvirnikom za pregled učinkov kalibracije

{% hint style="success" %}
**Priporočeno**: Uporabite sloj RAW (odbojnost) pri preverjanju vrednosti pikslov za znanstvene meritve in analize.
{% endhint %}

### RAW (NDVI indeks)... in podobno

* Izračunana slika vegetacijskega indeksa (v tem primeru NDVI)
* Ime indeksa se spreminja glede na to, kateri indeks je bil konfiguriran med obdelavo
* Primeri: RAW (NDVI indeks), RAW (NDRE indeks), RAW (GNDVI indeks) itd.
* Enopasovna slika v sivih odtenkih, ki prikazuje rezultate izračuna indeksa
* Za vsak indeks, nastavljen v nastavitvah projekta, se prikaže ena plast

**Možna imena indeksov:**

* RAW (indeks NDVI)
* RAW (indeks NDRE)
* RAW (indeks GNDVI)
* RAW (indeks OSAVI)
* RAW (indeks EVI)
* RAW (indeks SAVI)
* In še mnoge druge... (glej [Formule multispektralnih indeksov](../project-settings/multispectral-index-formulas.md))

**Kdaj si ogledati:**

* Pregledovanje rezultatov izračuna indeksa
* Preverjanje razponov vrednosti indeksa
* Identificiranje območij interesa
* Preverjanje indeksnih slik pred uporabo v GIS ali analizi

***

## Uporaba izbirnika plasti

### Odpiranje spustnega menija

1. Odprite sliko v načinu polnega zaslona (kliknite katero koli miniaturno sliko v pregledovalniku slik)
2. Poiščite **spustni meni plasti** v zgornjem desnem kotu pregledovalnika
3. Spustni meni prikazuje trenutno izbrano plast (npr. »JPG«)
4. Kliknite spustni meni, da si ogledate vse razpoložljive plasti

### Preklapljanje med plastmi

1. Kliknite spustni meni za plasti, da odprete seznam
2. Prikazane so vse razpoložljive plasti za trenutno sliko
3. Kliknite katero koli ime plasti, da preklopite na to različico
4. Slika se takoj posodobi in prikaže izbrano plast

**Hitro preklapljanje:**

* Spustni meni si zapomni vašo zadnjo izbiro
* Pri prehodu na naslednjo sliko Chloros poskuša prikazati isti tip plasti
* Če ta plast na naslednji sliki ne obstaja, se privzeto prikaže JPG

### Razpoložljivost plasti

Vse plasti niso na voljo za vsako sliko:

**Vedno na voljo:*** ✅ JPG (vsaka slika ima predogled v formatu JPG)

**Pogojno na voljo:**

* ⚠️ RAW (Original) – Samo če je bila slika posneta v načinu RAW ali RAW+JPG
* ⚠️ RAW (Target) – Samo če slika vsebuje zaznane kalibracijske tarče
* ⚠️ RAW (Reflectance) – Samo po obdelavi z omogočeno kalibracijo odbojnosti
* ⚠️ RAW (\[Index] Index) – samo po obdelavi z nastavljenimi indeksi

***

## Ohranjanje plasti

### Prehod med slikami

Ko preidete na drugo sliko (z uporabo puščičnih tipk ali s klikom na sličice):**Nastavitve plasti se ohranijo:**

* Če si ogledujete »RAW (Reflectance)«, naslednja slika prikazuje »RAW (Reflectance)« (če je na voljo)
* Če si ogledujete »RAW (NDVI indeks)«, naslednja slika prikazuje »RAW (NDVI indeks)« (če je na voljo)
* Če isti sloj ne obstaja, se privzeto prikaže JPG

**Primer poteka dela:**

1. Odprite sliko 1, preklopite na RAW (NDVI Index)
2. Pritisnite → za ogled slike 2
3. Slika 2 samodejno prikaže sloj RAW (NDVI Index)
4. Nadaljujte z navigacijo – vse slike prikazujejo sloj NDVI
5. Zelo učinkovito za pregledovanje rezultatov indeksa na več slikah

***

## Pogosti delovni tokovi

### Delovni tok 1: Primerjava pred/po

**Cilj**: Primerjava izvirne in kalibrirane slike

1. Odprite obdelano sliko v pregledovalniku slik
2. Iz spustnega menija izberite **RAW (izvirno)**

3. Upoštevajte vinjetiranje in nekalibrirane vrednosti
4. Iz spustnega menija preklopite na **RAW (odbojnost)**

5. Primerjajte – vinjetiranje je odstranjeno, vrednosti so kalibrirane

### Delovni tok 2: Pregled indeksa

**Cilj**: Hitro pregledati rezultate NDVI v celotnem nizu podatkov

1. Odprite prvo obdelano sliko
2. Iz spustnega menija izberite **RAW (NDVI Index)**

3. Uporabite puščico → za prehod na naslednjo sliko
4. Sloj NDVI se samodejno ohrani
5. Nadaljujte skozi vse slike in preverjajte vzorce NDVI
6. Preklopite na **RAW (NDRE Index)** za primerjavo

### Delovni tok 3: Preverjanje ciljev

**Cilj**: Preverite, ali so bile vse slike ciljev pravilno zaznane

1. Preidite na sliko cilja
2. Iz spustnega menija izberite **RAW (Cilj)**

3. Preverite, ali so kalibracijski cilji jasno vidni in zaznani
4. Preidite na naslednjo sliko cilja
5. Ponovite preverjanje za vse cilje

### Delovni tok 4: Pregled vrednosti pikslov

**Cilj**: Preverite vrednosti odbojnosti za znanstveno točnost

1. Odprite obdelano sliko
2. Izberite sloj **RAW (Reflectance)**

3. Vklopite način**Pixel Percent** (gumb v zgornjem desnem orodnem pasu)
4. Premaknite kurzor nad območja vegetacije
5. Preverite, ali so vrednosti pikslov v pričakovanih razponih (30–70 % za NIR, 5–15 % za Red)
6. Preverite, ali so vrednosti na območjih tal in vode ustrezne

***

## Razumevanje vrednosti pikslov po slojih

Različni sloji prikazujejo različne razpone vrednosti pikslov:

### Sloj JPG

* **Razpon**: 0–255 (8-bitni)
* **Pomen**: Prikazne vrednosti, popravljene z gama korekcijo
* **Uporaba**: Samo vizualni pregled, ne za znanstvene meritve

### RAW (izvirni)

* **Razpon**: 0–65535 (16-bitni)
* **Pomen**: Surove digitalne vrednosti senzorja
* **Uporaba**: Preverjanje delovanja senzorja, nekalibrirano

### RAW (odbojnost)

* **Razpon**: 0–65.535 (16-bitni TIFF) ali 0,0–1,0 (32-bitni odstotek)
* **Pomen**: Kalibrirani odstotek odbojnosti
* **Uporaba**: Znanstvene meritve in analize**Za 16-bitni TIFF:**Delite z 65.535, da dobite odstotek odbojnosti**Za 32-bitni odstotek:** Vrednosti neposredno predstavljajo odstotek (0,5 = 50 % odbojnosti)

### RAW (indeksne slike)

* **Območje**: Razlikuje se glede na indeks (običajno od -1,0 do +1,0 za normalizirane indekse)
* **Pomen**: Rezultat izračuna indeksa
* **Primeri**:
  * NDVI: od -1 do +1 (vegetacija običajno od 0,4 do 0,9)
  * NDRE: od -1 do +1 (odkrivanje stresa)
  * EVI: od 0 do 1 (izboljšana vegetacija)

***

## Nasveti in najboljše prakse

### Učinkovito preklapljanje med sloji

* **Poznavanje bližnjic na tipkovnici**: Čeprav za plasti ni bližnjic na tipkovnici, puščice za navigacijo (←/→) delujejo na vseh plasteh
* **Dosledni delovni tokovi**: Izberite eno plast (npr. NDVI) in preglejte celoten niz podatkov, preden preklopite na drugo
* **Hitro primerjanje**: Preklopite med Original in Reflectance, da preverite kakovost obdelave

### Upoštevanje zmogljivosti

* **JPG se nalaga najhitreje**: Uporabite ga za hitro navigacijo med številnimi slikami
* **Sloji RAW se nalagajo počasneje**: Višja ločljivost in bitna globina
* **Indeksni sloji**: Podobna hitrost kot pri slojih Reflectance
* **Prvo nalaganje je najpočasnejše**: Nadaljnji ogledi istega sloja so shranjeni v predpomnilniku in so hitrejši

### Preverjanje kakovosti

* **Vedno preverite RAW (Original)**: Preverite kakovost izvornih podatkov, preden zaupate obdelanim izhodnim podatkom
* **Primerjajte plasti**: Uporabite preklapljanje med plastmi, da preverite, ali je obdelava delovala pravilno
* **Preverite indeksne razpone**: Uporabite način Pixel Percent z indeksnimi plastmi, da preverite, ali so vrednosti razumne***

## Odpravljanje težav

### Plast ni na voljo

**Težava**: Pričakovana plast se ne prikaže v spustnem meniju**Možni vzroki:**

* Slika ni bila obdelana (na voljo sta le JPG in RAW (izvirnik))
* Kalibracija odbojnosti je bila med obdelavo onemogočena
* Poseben indeks ni bil konfiguriran v nastavitvah projekta
* Slika je slika, namenjena izključno ciljem (za cilje niso bili ustvarjeni indeksi)

**Rešitve:**

1. Preverite, ali je bila slika obdelana (v izhodni mapi preverite, ali so datoteke obdelane)
2. Preverite nastavitve projekta, da se prepričate, da so bili indeksi konfigurirani
3. Ponovno obdelajte s omogočenimi želenimi indeksi

### Prikazana je napačna plast

**Problem**: Slika se odpre v nepričakovani plasti**Vzrok**: Nastavitev plasti iz prejšnje slike se je prenesla, vendar ta plast na trenutni sliki ne obstaja**Rešitev**: Chloros samodejno preklopi na JPG, če prednostna plast ni na voljo – to je normalno delovanje

### Ne vidim kalibracijskih ciljev

**Problem**: Plast RAW (Cilj) ne prikazuje zaznave ciljev**Možni vzroki:**

* Cilji niso bili zaznani med obdelavo
* Slika dejansko ne vsebuje ciljev
* Nastavitve zaznavanja ciljev so preveč stroge

**Rešitve:**

1. Preverite dnevnik odpravljanja napak za sporočila »Cilj najden«
2. Preverite, ali slika dejansko vsebuje vidne kalibracijske cilje
3. Prilagodite nastavitve zaznavanja ciljev v nastavitvah projekta
4. Glejte [Izbira ciljnih slik](../processing-images-gui/choosing-target-images.md)

***

## Sorodne funkcije

### Orodja za pregledovanje slik

Pri pregledovanju katere koli plasti lahko uporabite:

* **Upravljalniki povečave**: Povečajte sliko, da pregledate podrobnosti
* **Premikanje**: Kliknite in povlecite, da se premikate po povečani sliki
* **Pregled vrednosti pikslov**: Oglejte si vrednosti na mestu kazalca
* **Navigacijske puščice**: Premikajte se med slikami, pri čemer ohranite plast
* **Način odstotkov pikslov**: Preklopite med prikazom DN in odstotkov

Za celotno dokumentacijo pregledovalnika slik glejte [Odpiranje slike v polnem zaslonu](opening-an-image-full-screen.md).

### Indeks/LUT Sandbox

Za interaktivno testiranje in vizualizacijo indeksov:

* **Izračun indeksa v realnem času**: Preizkusite različne formule indeksov
* **Barvno preslikavanje LUT**: Uporabite barvne prehode na indeksih v sivih odtenkih
* **Izvoz vizualizacij**: Shranite barvne slike indeksov

Za podrobnosti glejte [Indeks/LUT Sandbox](index-lut-sandbox.md).

***

## Naslednji koraki

Sedaj, ko razumete sloje slik:

* [**Odpiranje slike v polnem zaslonu**](opening-an-image-full-screen.md) – Celoten vodnik za Image Viewer
* [**Index/LUT Sandbox**](index-lut-sandbox.md) – Interaktivna vizualizacija indeksov
* [**Formule za multispektralne indekse**](../project-settings/multispectral-index-formulas.md) – Referenca razpoložljivih indeksov
* [**Zaključek obdelave**](../processing-images-gui/finishing-the-processing.md) – Razumevanje obdelanih izhodnih podatkov

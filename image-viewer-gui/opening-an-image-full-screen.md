# Odpiranje slike v polnem zaslonu

Chloros Image Viewer ponuja namenski vmesnik v polnem zaslonu za pregledovanje, analiziranje in obdelavo vaših multispektralnih slik. Ne glede na to, ali pregledujete originalne slike ali obdelane izhodne podatke, Image Viewer ponuja zmogljiva orodja za pregledovanje in analizo.

## Dostop do pregledovalnika slik

### Iz brskalnika datotek

Najpogostejši način za odpiranje slike v pregledovalniku slik:

1. Preverite, ali ste v zavihku **Brskalnik datotek**. <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Kliknite katero koli **miniaturno sliko** v mreži slik
3. Slika se odpre v **glavnem predoglednem področju** (sredi zaslona)
4. Slika je zdaj naložena in pripravljena za ogled v polnem zaslonu

### Odpiranje zavihka Image Viewer

Ko je slika naložena v predoglednem področju:

1. Kliknite ikono **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> v levem stranskem meniju.
2. Odpre se zavihek pregledovalnika slik, ki prikaže izbrano sliko v celozaslonskem načinu.
3. V levem stranskem meniju so na voljo napredna orodja za pregledovanje in analizo.

***

## Pregled vmesnika pregledovalnika slik

### Glavno območje zaslona

Največji del zaslona prikazuje sliko:

* **Polna ločljivost**: slike so prikazane v izvirni ločljivosti.
* **Povečljivost**: za povečavo uporabite gumbe ali kolesce miške
* **Premikanje**: kliknite in povlecite, da se premikate po povečani sliki
* **Ohranjeno razmerje stranic**: slike se sorazmerno prilagajajo

***

## Možnosti pregledovanja

### Osnovno navajanje po slikah

#### Brskanje po slikah

Po naboru slik se premikajte s pomočjo bližnjic na tipkovnici ali gumbov:

* **Naslednja slika**: kliknite gumb → ali pritisnite tipko **→** (desna puščica)
* **Prejšnja slika**: kliknite gumb ← ali pritisnite tipko **←** (leva puščica)
* **Preskok na določeno sliko**: vrnite se v brskalnik datotek in kliknite želeno sličico

#### Nadzorne tipke za povečavo

Prilagodite povečavo, da si ogledate podrobnosti slike:

**Povečajte:**

* Kliknite gumb **+** (plus)
* Pritisnite tipko **+** ali **=**
* Pomaknite kolesce miške **navzgor**

**Pomanjšajte:**

* Kliknite gumb **−** (minus)
* Pritisnite tipko **−** (minus)
* Pomaknite kolesce miške **navzdol**

#### Premikanje slike pri povečavi

Ko je slika povečana preko velikosti zaslona:

1. Premaknite kurzor miške nad sliko.
2. Kliknite in **držite levi gumb miške**.
3. **Povlecite**, da premaknete sliko.
4. Spustite, da ustavite premikanje.

**Alternativa**: Uporabite puščične tipke za premikanje v majhnih korakih.

***

## Pregled vrednosti pikslov

### Ogled vrednosti pikslov na kazalcu

Ko premaknete miškin kazalec nad sliko, se vrednosti pikslov prikažejo v realnem času:

**Mesto prikaza vrednosti:**

* **Plavajoče število in rdeča črta v legendi gradienta indeksa LUT na desni strani**
* **Pri dodatnem povečanju se plavajoča vrednost prikaže blizu kazalca in označenega piksla**
* Prikaže vrednosti za piksel **pod kazalcem ali označenim**
* Posodablja se, ko premikate miško

***

## Vrste slik, ki jih lahko pregledujete

### JPG

**JPG slike iz kamere:**

* Prikaz JPG podatkov kot v predogledu
* Prikaz originalnih, nepopravljenih vrednosti
* Koristno za preverjanje kakovosti slike pred obdelavo

### RAW (original)

### RAW (odbojnost)

**Po obdelavi:**

* Popravljena vinjeta
* Kalibrirana odbojnost
* Večpasovni TIFF (Red, Green, NIR itd.)
* Znanstveni podatki pripravljeni za analizo

### RAW (indeks)

**NDVI, NDRE, GNDVI itd. (datoteke \_NDVI.tif):**

* Enopasovne sive slike
* Vrednosti pikslov predstavljajo rezultate izračuna indeksa
* Običajno v razponu od -1 do +1 za normalizirane indekse
* Za vizualizacijo je mogoče uporabiti barvne LUT-e

***

## Uporaba indeksa in LUT-a

Uporabite multispektralne indekse in barvne preglednice:

1. Poiščite **Index/LUT Sandbox** v **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> stranski vrstici
2. Izberite indeks vegetacije (NDVI, NDRE itd.)
3. Izberite multispektralno formulo ali ustvarite svojo lastno (samo Chloros+)
4. Uporabite barvni LUT-gradient za vizualizacijo
5. Prilagodite vrednostna območja in pragove

Podrobna navodila najdete v [Index/LUT Sandbox](index-lut-sandbox.md).

***

## Bližnjice na tipkovnici

### Navigacija

* **→** (desna puščica): naslednja slika
* **←** (leva puščica): prejšnja slika
* **Home**: prva slika na seznamu
* **End**: Zadnja slika na seznamu

### Povečava

* **+** ali **=**: Povečava
* **−**: Pomanjšanje
* **Miškin kolesce**: Povečava/pomanjšanje

***

### Preverjanje izračunov indeksa

Preverite, ali so indeksi izračunani pravilno:

1. Odprite NDVI ali drugo sliko indeksa
2. Preverite vegetacijska območja:
   * **NDVI**: Za zdrave rastline mora prikazati 0,4–0,9
   * **NDRE**: Višje vrednosti za bujno rast
   * **GNDVI**: podobno kot NDVI, vendar občutljivo na klorofil
3. Preverite ne-vegetacijo:
   * **Tla**: blizu 0 ali rahlo negativna
   * **Voda**: negativne vrednosti (-0,5 do 0)

***

## Odpravljanje težav pri prikazovanju

### Slika se ne odpre

**Možni vzroki:**

* Datoteka je bila med obdelavo poškodovana.
* Nepodprt format datoteke.
* Premajhen pomnilnik za veliko sliko.

**Rešitve:**

1. Poskusite odpreti v zunanjem pregledovalniku, da preverite celovitost datoteke.
2. Preverite, ali format datoteke ustreza pričakovanemu tipu.
3. Zaprite druge aplikacije, da sprostite pomnilnik.
4. Poskusite z manjšo/drugo sliko.

### Črno-bela slika

**Možni vzroki:**

* Vrednostni razpon zunaj zmogljivosti zaslona.
* 32-bitna slika s plavajočo vejico z neobičajnimi vrednostmi.
* Napaka pri izračunu indeksa.

**Rešitve:**

1. Preverite vrednosti pikslov – če so vse zelo nizke ali zelo visoke, prilagodite razpon prikaza.
2. Poskusite odpreti v QGIS ali podobnem programu z avtomatskim prilagajanjem razpona.
3. Preverite dnevnik odpravljanja napak iz obdelave za napake.

### Vrednosti pikslov se zdijo napačne

**Možni vzroki:**

* Ogled napačne slike (izvirnik proti obdelani sliki)
* Kalibracija ni bila pravilno uporabljena
* Podatki svetlobnega senzorja niso bili vključeni v vnos
* Način odstotkov je bil nepravilno preklopljen

**Rešitve:**

1. Preverite, ali si ogledujete obdelano izhodno sliko (preverite končnico imena datoteke)
2. Preverite stanje gumba za način odstotkov
3. Primerjajte s sliko iz istega niza podatkov, za katero veste, da je pravilna

***

## Naslednji koraki

Zdaj, ko lahko slike pregledujete v polnem zaslonu:

* [**Sloji slik**](image-layers.md) – spoznajte vizualizacijo več pasov
* [**Indeks/LUT Sandbox**](index-lut-sandbox.md) – uporabite prilagojene indekse in barvno mapiranje
* [**Formule večspektralnega indeksa**](../project-settings/multispectral-index-formulas.md) – razumevanje razpoložljivih indeksov

Za potek obdelave glejte:

* [**Obdelava slik (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – popoln vodnik za obdelavo

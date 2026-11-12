# Odpiranje slike v polnem zaslonu

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption><p>Slika, odprta v polnem zaslonu, z izbirnikom plasti v zgornjem desnem kotu</p></figcaption></figure>

Pregledovalnik slik Chloros je vmesnik v polnem zaslonu za pregledovanje, preučevanje in merjenje vaših slik. Tu lahko odčitate **dejanske vrednosti pikslov** — DN po kanalih, odstotek odbojnosti ali sevanje v W/m²/sr/nm — namesto raztegnjenega predogleda, ki ga prikazuje zaslon.

## Dostop do pregledovalnika slik

### Iz brskalnika datotek

1. Odprite zavihek **Brskalnik datotek** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Kliknite katero koli **miniaturno sliko** v [mreži slik](image-grid.md)
3. Slika se odpre v polnem zaslonu v zavihku **Pregledovalnik slik**

Slika se odpre v tistem produktu, ki ga je prikazovala mreža. Če je mreža nastavljena na `RAW (Reflectance)`, se odpre prav ta sloj.

### Odpiranje stranskega menija pregledovalnika slik

Kliknite ikono **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> v levem stranskem meniju, da se odpre panel za analizo. Vsebuje, od zgoraj navzdol:

* ime slike in model kamere
* gumb **Export/Save Image(s)** (le, če je aktiven indeks ali LUT)
* potrditvena polja **Indeks**in**LUT** ter plošča za konfiguracijo indeksa — glej [Index/LUT Sandbox](index-lut-sandbox.md)
* panel **Vrednosti kazalca**: odčitki po kanalih, histogram plasti in nadzor GSD***

## Navigacija in povečava

### Brskanje po slikah

* **Naslednja slika**: gumb → ali tipka**→** (desna puščica)
* **Prejšnja slika**: gumb ← ali tipka**←** (leva puščica)
* **Preskok na določeno sliko**: vrnite se na mrežo in kliknite njeno miniaturno sliko

Povečava in premikanje po sliki ostajata nespremenjeni med prehodom med slikami, tako da lahko pregledujete niz slik, medtem ko ostajate na istem delu okvirja.

### Povečava

Povečavo upravljate z **kolescem miške** v 15-odstotnih korakih, pri čemer je ta vezana na kazalec – točka pod kazalcem ostane pod kazalcem. Obseg je omejen z velikostjo slike in okna: ne morete pomanjšati slike pod velikost »prilagodi oknu«, zgornja meja pa je določena z izvirno ločljivostjo slike.

V pregledovalniku v polnem zaslonu ni namenskih tipk za povečavo. (V mreži **Ctrl + `+` / `−`** spreminjata velikost sličic – to je druga funkcija.)

### Premikanje slike pri povečavi

Kliknite in pridržite levi gumb miške nad sliko ter jo povlecite. Premikanje je omejeno, tako da slike ni mogoče povleči izven zaslona.

### Pregled posameznih pikslov pri visokem povečanju

Ko efektivno povečanje preseže **60×**, Chloros okoli posameznega prikazanega piksla pod kazalcem nariše poudarjeno okvirje in zraven prikaže plavajočo vrednost.

»»Efektivna« povečava upošteva velikost bloka GSD: pri velikosti bloka 8 se označitev prikaže že pri 7,5× povečavi namesto pri 60×, saj en prikazan piksel že zajema 8 × 8 izvornih pikslov. Če povečavo zmanjšate pod prag, se označitev izgine.

### Tipkovniški bližnjice

| Tipka                             | Kje       | Dejanje                              |
| ------------------------------- | ----------- | ----------------------------------- |
| **→**                           | Celozaslonski način | Naslednja slika                          |
| **←**                           | Celozaslonski način | Prejšnja slika                      |
| **Ctrl + R**                    | Celozaslonski način | Ponastavi indeks/LUT peskovnik         |
| **Ctrl + `+`**/**Ctrl + `=`** | Mreža        | Večje sličice (4 px na pritisk)  |
| **Ctrl + `−`**                  | Mreža        | Manjše sličice (4 px na pritisk) |

***

## Vrednosti kazalca

Premaknite kazalec miške nad sliko in v oknu **Vrednosti kazalca** se prikažejo vrednosti vseh kanalov pod njim.

{% hint style="success" %}
**To so dejanske številčne vrednosti datoteke.** Platno na zaslonu je 8-bitni raztegnjen predogled in teh vrednosti ne more prikazati, zato Chloros za prikaz vzorči dejansko datoteko izdelka. Zato 12-bitni surovi okvir prikazuje vrednosti nad 255, plavajoča 32-bitna plast svetilnosti pa prikazuje fizikalne enote.
{% endhint %}

### Kaj pomenijo stolpci

Plošča se prilagaja sloju, ki ga pregledujete:

| Sloj, ki ga pregledujete              | Prikazani stolpci    | Opombe                                                                                           |
| ---------------------------------- | ---------------- | ----------------------------------------------------------------------------------------------- |
| Odbojnost                        | **DN**in**%** | Odstotek se izračuna glede na lastno lestvico datoteke — glejte spodaj                                      |
| Sevanje                           | **W/m²/sr/nm**   | Fizične vrednosti v obliki float; stolpec DN ni prikazan, saj je DN v tem primeru brez pomena                           |
| Surovi / brez bayera / predogled / JPG    | **DN**           | Celoštevilčne digitalne vrednosti                                                                         |
| 32-bitni izvozi odstotka odbojnosti | **%** samo       | Shranjena vrednost s plavajočo vejico ni DN, zato bi zaokroževanje na celo število povzročilo izpis nesmiselnih vrednosti, kot sta `0` ali `1` |

Vsaka vrstica je označena z imenom kanala za filter vaše kamere — `Red / Green / NIR` za RGN, `Orange / Cyan / NIR` za OCN, `NIR / Green / Blue` za NGB, `Red / Green / Blue` za RGB ter ime enopasovnega filtra za kamere RE, NIR ter mono M3M kamere. Vsaka oznaka ima barvno piko, ki ustreza krogom kanalov, uporabljenim v uredniku indeksnih formul.

Shranjene **indeksne in LUT** slike so poseben primer: vsebujejo komponente barvne karte namesto spektralnih pasov, zato so njihove vrstice označene z `Red / Green / Blue` (ali `Index` za enokanalno indeksno datoteko) namesto z imeni filtrov kamerefiltra kamere.

Ko je indeks aktiven v peskovniku, se pod kanali prikaže dodatna vrstica, ki prikazuje **vrednost indeksa** na mestu kazalca, skupaj z imenom indeksa in belo piko, ki ustreza njegovemu označevalcu na histogramu.

### Odstotek odbojnosti uporablja lastno lestvico vsake datoteke

{% hint style="warning" %}
**Ne predpostavljajte, da je 65535 = 100 %.** Chloros shranjuje odbojnost v različnih lestvicah, odvisno od kamere, ki jo je ustvarila, pregledovalnik pa za vsako datoteko določi pravo.
{% endhint %}

| Vir                  | DN, ki ustreza odbojnosti 1,0 | Kako se prepozna                                                                                                                               |
| ----------------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **LATTICE**(M3C / M3M) |**32768**                      | XMP-oznaka `Chloros:PixelScale=32768`, vpisana v vsak izvoz odbojnosti LATTICE. 2× rezerva omogoča, da datoteka vsebuje ρ nad 1,0 brez odrezovanja |
| **Survey3**|**65535**                      | Ni XMP oznake za merilo Chloros — Kalibracija Survey3 zapiše ρ × dtype-max in obreže pri 1,0                                                               |

Pregledovalnik, peskovnik indeksa/LUT in izvoz indeksa vsi določajo merilo prek istega enotnega izvajanja, zato je vrednost, ki jo preberete na kazalcu, enaka vrednosti, ki se uporablja indeksna matematika.

Dve posledici, ki jih je vredno poznati:

* **32-bitni odstotek**TIFF shranjuje DN/65535 kot float,**8-bitni** PNG/JPG izvoz shranjuje DN × 255/65535 – pregledovalnik oba pretvori nazaj, preden izpiše odstotek.
* Enega primera ni mogoče obnoviti: **8-bitni izvoz TIFF zajema iz 8-bitnega vira** je omejen na 0–255 namesto da bi bil ponovno skaliran, in namerno ne vsebuje oznake merila. Za te datoteke plošča natisne le DN, brez stolpca z odstotki. To je iskren odgovor, ne pa napaka.

***

## Histogram plasti

Pod vrsticami kazalca je v živo prikazan histogram sloja, ki ga pregledujete, v **256 razredih**. Privzeto riše eno združeno krivuljo, tehtano `(R + 2G + B) / 4` — isti merilni prostor, ki ga uporabljajo histogrami kamere LATTICE. Vklop**RGB** pa se ta nadomesti s krivuljami po posameznih kanalih v barvah kanalov, ki so aditivno premešane, tako da ostanejo prekrivanja berljiva. Monokromatske plasti vedno rišejo eno samo krivuljo.

Vodoravna os je v enoti samega sloja:

| Sloj       | Enota osi  | Največja vrednost osi                                               |
| ----------- | ---------- | ---------------------------------------------------------- |
| Odbojnost | odstotek    | 125 % — razpon izdelka omogoča ρ nad 1,0           |
| Sijaj    | W/m²/sr/nm | Lastni vrh okvirja, zaokrožen navzgor na dve pomembni števki |
| 8-bitni podatki | DN         | 255                                                        |
| 12-bitni podatki | DN         | 4095                                                       |
| 16-bitni podatki | DN         | 65535                                                      |

Ko je os v DN in se ustavi na eni od teh treh zgornjih mej, Chloros ve tudi, kakšna je bitna globina tistega, kar si ogledujete.

Nad histogramom so trije gumbi:

| Gumb     | Privzeto | Učinek                                                                                                                                                                                                                                                                                   |
| ---------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **KURZOR** | Vklopljeno | Na histogramu nariše označevalne črte na natančnih vrednostih, prikazanih v zgornjih vrsticah, tako da lahko vidite, kje se piksel pod vašim kurzorjem nahaja v porazdelitvi okvirja. V načinu RGB je na vsakem kanalu en označevalec v svoji barvi; sicer pa je en sam bel označevalec na združeni vrednosti |
| **INDEKS**| Vklopljeno      | Prikaže se le, ko je indeks aktiven. Preklopi histogram iz izvornih pasov na**porazdelitev indeksnih vrednosti**, pri čemer sta oba praga odrezanja prikazana kot oranžni prekinjeni črti, indeksna vrednost kazalca pa kot bela črta                                                          |
| **RGB**| Izklopljeno     | Preklopi s skupne krivulje na krivulje po posameznih kanalih. Na mono senzorju ta gumb prikazuje**MONO** in je onemogočen — prikazati je mogoče le en kanal                                                                                                                                  |

Histogram se izračuna na podlagi **blokov, ki jih vidite**, ne pa na podlagi izvornih pikslov za njimi: spremenite velikost bloka GSD in porazdelitev se ponovno izračuna, tako da so histogram, označevalec kazalca in prikazana slika vedno usklajeni.

***

## Velikost bloka GSD

Na dnu plošče je upravljalnik **GSD (px)**: številčno polje, drsnik od**1 do 256**in gumb**RESET**.

Z njim se _prikazana_ slika grobo poenostavi tako, da se N × N blok izvornih pikslov povpreči v en prikazan piksel. `1` je izvorna ločljivost.

* Vpliva na **pogled v polnem zaslonu, sličice v mreži, prikaz kurzorja in oba histograma** – vse, kar prikazuje sliko, se ujemajo z isto osnovno ločljivostjo.
* To velja **samo za prikaz**. Obdelava in izvoz ostajata nespremenjeni. Edina izjema je namerna: izvoz iz [Index/LUT Sandbox](index-lut-sandbox.md) shrani tisto, kar trenutno gledate, zato ohrani trenutno velikost bloka, izvozni panel pa vas opozori, če je velikost bloka večja od 1.
* Vrednost se shrani **po projektu** kot `viewer_display.gsd_bin` v `project.json`, zato ostane ohranjena tudi po zaprtju in ponovnem odprtju.
* Kadar je velikost bloka večja od 1, kazalec prikazuje vrednost bloka, ne pa izvorne piksle – prikazana vrednost je povprečje bloka pod kazalcem.

{% hint style="info" %}
**Zakaj »velikost bloka« in ne centimetrov na piksel?** Podatek v cm/px zahteva višino nad tlemi. EXIF-podatki posameznega okvirja vsebujejo GPS-nadmorsko višino nad povprečno gladino morja, ne pa nad terenom, na katerega je bila usmerjena kamera, zato Chloros ne bo izpisal razdalje do tal, ki je ne more zanesljivo izračunati. Velikost bloka v izvornih pikslah je enaka rešitev, ki jo uporabljajo orodja za oblake MAPIR, kadar je razdalja vzorčenja na tleh neznana.
{% endhint %}

***

## Vrste slik, ki jih lahko pregledujete

Spustni meni plasti v zgornjem desnem kotu pregledovalnika navaja vse različice trenutne slike. Kateri vnosi se prikažejo, je odvisno od kamere in od tega, kaj je bilo obdelano — glejte [Slikovne plasti](image-layers.md) za celoten seznam in delovanje spustnega menija.

### Survey3

* **JPG** — datoteka predogleda, ki jo ustvari kamera
* **RAW (Original)** — izvorna datoteka `.RAW`, brez bayerjevega filtra za prikaz, brez popravkov
* **RAW (cilj)** — posnetek, ki je bil prepoznan kot vsebujoč kalibracijski cilj
* **RAW (odbojnost)** — kalibrirani produkt odbojnosti (65535 = ρ 1,0)
* **Popravljeno vinjetiranje**/**Odziv senzorja** — nekalibrirani nadomestni izdelek
* **Uravnotežena belina** — izdelek z uravnoteženo belino
* **RAW (`<INDEX>` indeks)**in**`<INDEX>` LUT** — izračunane indeksne slike

### LATTICE

Pri zajemanju v LATTICE se uporablja isti spustni meni z imeni stopenj procesa:

| Sloj                 | Kaj vsebuje                                                        |
| --------------------- | -------------------------------------------------------------------- |
| **RAW (Original)**    | Izvorni surovi okvir, kot je bil zajet                                     |
| **RAW (Debayered)**   | Linearna slika brez bayersovega filtra                                           |
| **RAW (Predogled)**     | Predogled na zaslonu — raztegnitev v lažnih barvah za multispektralne kamere |
| **Bela ravnovesje**    | Predogled na zaslonu za glavne kamere RGB (bela ravnovesje + gama)   |
| **RAW (sevanje)**    | Spektralno sevanje v formatu Float32 v W/m²/sr/nm                              |
| **RAW (odbojnost)** | Odbojnost v formatu uint16, 32768 = ρ 1,0                                    |

Sevanje in odbojnost sta na voljo le v multispektralnem načinu: glavna kamera RGB nima radiometrije po posameznih pasovih, zato ti sloji zanjo niso ustvarjeni.

***

## Uporaba indeksov in LUT-ov

Uporabite multispektralne indekse in barvne preglednice (LUT) iz stranskega menija:

1. Odprite **Image Viewer** v stranskem meniju <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Označite **Index**

3. Izberite filter vaše kamere in formulo indeksa, nato pa kroge kanalov povlecite na mesta v formuli
4. Dodajte LUT in izberite barvni prehod, pragove ter način izreza
5. Preberite vrednosti na mestu kazalca in shranite rezultat z **Izvoz/Shrani sliko(-e)**Za celoten vodnik glejte [Index/LUT Sandbox](index-lut-sandbox.md).

***

## Odpravljanje težav

### Slika se ne odpre

**Možni vzroki**: datoteka je bila po uvozu premaknjena ali izbrisana; izdelek ni bil nikoli shranjen; premalo pomnilnika za zelo veliko sliko.**Kaj storiti**:

1. Preverite, ali datoteka plasti še vedno obstaja v izhodni strukturi projekta
2. Odprite datoteko v zunanjem pregledovalniku, da se prepričate, da je nepoškodovana
3. Zaprite druge aplikacije, da sprostite pomnilnik

### Slika je črna, bela ali prekomerno obarvana

**Možni vzroki**: raztezek zaslona nima s čim delati (skoraj konstanten okvir); sloj float32 z neobičajnimi vrednostmi; indeks, ki ni ustvaril veljavnih podatkov.**Kaj storiti**:

1. Preberite vrednosti kazalca – če je vsak kanal na ničli ali blizu nje, je težava v podatkih, ne v prikazu
2. Preverite histogram: en sam vrh na enem koncu kaže, da je okvir odrezan ali prazen
3. Preverite dnevnik obdelave za izvedbo, ki je ustvarila sloj

### Vrednosti izgledajo napačne

**Možni vzroki**: ste na drugačni plasti, kot mislite; primerjate odstotek z neobdelano vrednostjo DN; primerjate datoteko LATTICE z datoteko Survey3 z uporabo istega delitelja.**Kaj storiti**:

1. Preverite izbrano plast v spustnem meniju – enote na plošči sledijo plasti
2. Za odbojnost uporabite stolpec **%**, namesto da sami delite DN; če morate deliti, uporabite `Chloros:PixelScale` iz te datoteke (32768 za LATTICE, če ni navedeno, pomeni 65535 za Survey3)
3. Velikost bloka GSD ponovno nastavite na 1 – pri vrednosti nad 1 berete povprečje bloka, ne pa posameznega piksla
4. Preverite, ali je kalibracija odbojnosti za ta okvir dejansko potekala; nekalibrirani nadomestni izdelek (Sensor Response / Vignette Corrected) ni odbojnost

***

## Naslednji koraki

* [**Sloji slike**](image-layers.md) — imena vseh slojev, če obstajajo, in pomen njihovih vrednosti
* [**Indeks/LUT Sandbox**](index-lut-sandbox.md) — ustvarjanje, prilagajanje in izvoz vizualizacij indeksov
* [**Oznake na zemljevidu**](map-markers.md) — isti nabor slik na zemljevidu
* [**Formule multispektralnih indeksov**](../project-settings/multispectral-index-formulas.md) — referenca indeksa

Za potek obdelave glejte [Obdelava slik (GUI)](../processing-images-gui/adding-files-to-a-project.md).

# Index/LUT Sandbox

Index/LUT Sandbox je interaktiven delovni prostor v stranskem pasu programa Chloros Image Viewer. Izberete formulo, ji dodelite kanale vaše kamere, jo obarvate z barvnim prehodom in prilagodite razpon vrednosti – slika se medtem posodablja v realnem času. Od različice 1.2.0 lahko **shranite tudi tisto, kar ste ustvarili**, bodisi za eno sliko bodisi za celoten projekt, brez ponovne obdelave.

## Za kaj je namenjen Sandbox

| Indeks/LUT Sandbox (interaktiven)        | Obdelava projekta (v seriji)       |
| -------------------------------------- | -------------------------------- |
| Po ena slika naenkrat, takojšnja povratna informacija  | Celoten nabor podatkov v enem izvajanju     |
| Eksperimentalno in iterativno             | Vnaprej nastavljene nastavitve          |
| Renderira v realnem času; shrani le, ko to zahtevate  | Vedno zapiše končne datoteke      |
| Idealno za iskanje pravih nastavitev | Najboljše, ko so nastavitve dokončne |

{% hint style="success" %}
**Običajni potek dela**: nastavljajte v Sandboxu, dokler vizualizacija ne prikazuje želenega rezultata, nato pa izvozite neposredno iz Sandboxa ali kopirajte iste nastavitve indeksa in LUT-ja v [Nastavitve projekta](../project-settings/project-settings.md), da jih naslednji ciklus obdelave vključi v vsako sliko.
{% endhint %}

***

## Odpiranje Sandboxa

1. Kliknite sliko v mreži — odpre se v polnem zaslonu v zavihku **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Kliknite ikono **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">, da se odpre levi stranski pas, če še ni odprt
3. Iz spustnega menija plasti v zgornjem desnem kotu izberite večpasovno plast — običajno se izbere **RAW (Reflectance)**, saj so indeksne vrednosti, izračunane na podlagi kalibrirane odbojnosti, primerljive med posameznimi slikami

Stranski trak prikazuje, od zgoraj navzdol:

* ime slike in model kamere
* gumb **Izvoz/Shrani sliko(-e)**— prikaže se, ko je označena možnost**Indeks**ali**LUT*** potrditvena polja **Indeks**in**LUT**
* ploščo za konfiguracijo indeksa
* ploščo **Vrednosti kazalca** z odčitkom, histogramom in nadzorom GSD

{% hint style="warning" %}
**Ni na voljo za monokromatske kamere.** Na enopasovni sliki LATTICE M3M sta obe potrditveni polji onemogočeni, z opombo _»Ni na voljo za monokromatske (M3M) senzorje«_ — večpasovni indeks ni opredeljen za en pas. Za izračun indeksov iz kamer M3M združite dve ali več v poravnani večpasovni niz in uporabite indeksni motor LATTICE.
{% endhint %}

***

## Uporaba indeksa

1. Označite polje **Indeks** na vrhu stranskega menija
2. Iz spustnega menija na levi izberite filter vaše kamere (`RGN`, `OCN`, `NGB`, `RGB`, `RE`, `NIR`)
3. Iz desnega spustnega menija izberite indeksno formulo – na voljo je 27 vgrajenih formul ter vse morebitne lastne formule, ki ste jih shranili
4. Formula se prikaže kot matematični izraz spodaj, s praznim krogom na vsakem mestu za pas. **Povlecite barvni krog kanala na mesto**, da ga povežete
5. Ko so vsi prostori, ki jih formula uporablja, povezani, se slika posodobi in prikaže vrednosti indeksa
6. Premaknite kazalec miške nad sliko, da preberete vrednosti; plošča **Vrednosti kazalca** doda vrstico indeksa z vrednostjo pod kazalcem

Dvakrat kliknite vezano mesto, da ga izbrišete. Nepopolna formula je običajno stanje med vlečenjem, ne pa napaka — slika se preprosto ne posodobi, dokler formula ni popolna.

Krogi kanalov so označeni z barvami: rdeča = Red, zelena = Green, modra = Blue, oranžna = Orange, cian = Cyan, vijolična = NIR, magenta = RE. Enake barve se uporabljajo za točke kanalov in krivulje histograma v plošči »Vrednosti kazalca«.

### Primer NDVI

```

Formula: (NIR - Red) / (NIR + Red)

For a Survey3W RGN camera:
  NIR = 850 nm band
  Red = 661 nm band

Result range:          -1.0 to +1.0
Typical vegetation:     0.4 to 0.9
Stressed vegetation:    0.2 to 0.4
Bare soil:              0.0 to 0.2
Water:                 -0.1 to 0.1
```

Za popoln pregled formul – vseh treh seznamov prednastavitev in tega, katera imena delujejo kje – glejte [Formule multispektralnih indeksov](../project-settings/multispectral-index-formulas.md).

### Z označeno možnostjo »Index«, vendar brez LUT

Slika je narisana v **sivi lestvici**, raztegnjena med dvema pragovnima vrednostma. To je namerno: indeksna slika je skalarni podatek, siva lestvica pa je njeno verodostojno upodabljanje. Če želite barvo, dodajte LUT.

***

## Delovanje z LUT-i (preglednicami)**Iskalna tabela** preslikava indeksne vrednosti v barve: vhod NDVI 0,65, izhod določena zelena barva. Ne spreminja podatkov – spreminja način njihovega branja.

### Dodajanje LUT-a

1. Kliknite gumb **"+ Dodaj LUT"**v meniju**<img src="../.gitbook/assets/image (1) (1) (1).png" alt="" data-size="line">** pod formulo
2. Izberite barvni prehod
3. Nastavite minimalno in maksimalno vrednost obrezovanja
4. Izberite način obrezovanja
5. V stranskem meniju označite polje **LUT**, da se prikaže

Polje za potrditev **LUT** ostane onemogočeno, dokler LUT ni dejansko konfiguriran v indeksu.

### Izbiranje barvnega prehoda

Premaknite miško nad **pasico prehoda**, da odprete seznam prednastavitev — Chloros vsebuje**sedem** prednastavitev prehodov:

| # | Prehod                            | Oblika                                                               |
| - | ----------------------------------- | ------------------------------------------------------------------- |
| 1 | Red → rumena → Green (**privzeto**)  | Razhajajoča — ustreza običajni intuiciji o vegetaciji, zelena = zdrava |
| 2 | Vijolična → Rumena → Green             | Divergentna, z izrazitim spodnjim delom                                  |
| 3 | Rjava → Bela → Blue                | Divergentna okoli svetle sredine                                   |
| 4 | Črna → Vijolična → Roza → Bledo rumena | Zaporedno, od temnega k svetlemu                                           |
| 5 | Red → Rumena → Blue                 | Razhajajoče okoli svetle sredine                                   |
| 6 | Vijolična → Blue → Green → Rumena      | Zaporedno, od temne k svetli                                           |
| 7 | Orange → Bela → Vijolična             | Razhajanje okoli svetle sredine                                   |

**Razhajajoči**barvni prehod postavi nevtralno barvo v sredino okna, kar je primerno, kadar srednja točka nekaj pomeni (prag, izhodiščni datum).**Zaporedni** barvni prehod poteka monotono od temnega k svetlemu, kar je primerno za količino, ki ima le vrednosti »več« in »manj«.

Vsaka prednastavitev ima sedem barvnih točk. Kliknite prednastavitev in slika se takoj posodobi (če je polje LUT označeno).

### Urejanje barvnih točk

Pod vrstico prehoda je vrsta barvnih vzorcev, po en za vsako točko:

* **Spreminjanje barve**: kliknite vzorec, da odprete izbirnik barv (barvno kolo, drsniki RGB/HSV ali šestnajstiški kod, npr. `#FF0000`)
* **Dodajanje prehoda**: kliknite gumb**+** na koncu vrstice — dodan bo bel prehod
* **Odstranite stopnjo**:**dvakrat kliknite** vzorec
* **Shranite urejen prehod**: kliknite ikono za shranjevanje ob vrstici prehoda, da dodate urejen prehod na seznam prednastavitev, tako da ga boste lahko ponovno izbrali

Prehod, ki ste ga nastavili za indeks, se shrani skupaj s tem indeksom v nastavitvah projekta, tako da ostane ohranjen tudi po zaprtju in ponovnem odprtju projekta.

**Manj točk**ustvarja ločene cone, ki se berejo kot klasifikacija;**več točk** ustvarja gladke, skoraj fotografske prehode. Tri do pet točk je primernih za predstavitvene diapozitive in klasifikacijske karte; šest do deset je primernih za splošno analizo; petnajst ali več je primernih za podrobne preglede in publikacijske slike.

### Nastavitev obsega vrednosti

Upravljalnik praga je **drsnik z dvema ročajema**, ki sega od −1 do +1, z uredljivim besedilnim poljem na vsakem koncu za natančne vrednosti ter gumbom**AUTO**.

* Povlecite katero koli ročico ali vnesite številko v ustrezno polje in pritisnite Enter
* **AUTO**nastavi razpon na**

2. in 98. percentil** veljavnih indeksnih vrednosti slike — to je dobra izhodiščna točka, ki izključi izjemne vrednosti. Chloros rezultat prilagodljivo zaokroži na 4 decimalna mesta za zelo ozek razpon, na 3 za ozek razpon, v drugih primerih pa na 2
* Vsaka ročna prilagoditev ima prednost pred nastavitvijo AUTO, dokler ponovno ne pritisnete gumb AUTO

Primer oken NDVI:

| Cilj                                    | Min  | Max |
| --------------------------------------- | ---- | --- |
| Prikaži vse                         | −1,0 | 1,0 |
| Samo vegetacija, izključi tla in vodo | 0,2  | 0,9 |
| Samo zdrava vegetacija                 | 0,5  | 0,9 |
| Poudari stres                        | 0,2  | 0,5 |

Z ožanjem okna povečate kontrast znotraj območja, ki vas zanima, in vse ostalo potisnete izven območja – kjer **način odrezovanja** določi, kaj se z njim zgodi.

***

## Načini odrezovanja

Ko vrednost indeksa piksla pade izven okna min/max, način izreza določi, kako se ta piksel prikaže.

| Oznaka spustnega menija                  | Shranjena vrednost      | Piksli zunaj območja se prikažejo kot                                                                                                |
| ------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Minimum in maksimum** (privzeto) | `clip`            | Najbližja končna barva prehoda – vrednosti pod minimumom prevzamejo prvo barvo, vrednosti nad maksimumom pa zadnjo |
| **Prosojno ozadje**      | `transparent`     | Popolnoma prosojno (pravi alfa)                                                                                                  |
| **Indeksno ozadje**| `indexColor`      | Siva lestvica, raztegnjena čez**celoten** indeksni razpon slike, tako da je struktura zunaj tega razpona še vedno vidna v sivi barvi                |
| **Izvirno ozadje**         | `backgroundColor` | Sama osnovna slika, tako da je barvni prekrivni sloj nameščen nad dejansko sceno                                                |

| Način                       | Najbolj primerno za                               | Videz                                      |
| -------------------------- | -------------------------------------- | ----------------------------------------- |
| **Najmanjša in največja vrednost**      | Celoten prikaz podatkov, znanstvena analiza | Vsak piksel je obarvan                      |
| **Prosojno ozadje** | GIS-prekrivki, izolacija vrednostnega pasu   | Barva znotraj okna, zunaj nič |
| **Indeksno ozadje**       | Poudarjanje ob ohranjanju konteksta podatkov    | Barva znotraj, siva zunaj               |
| **Izvirno ozadje**    | Poročila in predstavitve              | Barva znotraj, fotografija zunaj         |

{% hint style="info" %}
**Piksli brez podatkov so vedno prosojni, v vsakem načinu.** Piksel, katerega indeks ni končen (deljenje z 0/0) ali je točno −1,0 ali +1,0 (saturacijski indikatorji, ko en pas kaže vrednost nič, drugi pa ne), se obravnava kot brez podatkov in ne kot skrajna vrednost. Tako se prepreči, da bi preosvetljene svetlobe in temne sence vplivale na barvno lestvico, namesto da bi se prikazovale kot skrajne vrednosti v okviru. Enako pravilo določa, kateri piksli se upoštevajo pri pragih AUTO in histogramu indeksov, tako da se vsi trije ujemajo.
{% endhint %}

Prosojnost se ohrani, če se izvoz shrani v formatu PNG. V formatu JPG je ni mogoče prikazati.

***

## Odčitavanje vrednosti med nastavljanjem

Plošča **Vrednosti kazalca** pod konfiguracijsko ploščo je merilni instrument za Sandbox:

* Premaknite kazalec po sliki in odčitajte izvorne vrednosti po posameznih kanalih ter indeksno vrednost v svoji vrstici
* Vklopite gumb **INDEX** nad histogramom, da si ogledate porazdelitev indeksnih vrednosti v okvirju, pri čemer sta vaši dve mejni vrednosti izreza prikazani kot oranžni prekinjeni črti, vrednost kazalca pa kot bela črta — to je najhitrejši način za izbiro okna, ki dejansko vsebuje vaše podatke
* Vklopite **CURSOR**, da se prikažejo označevalne črte pri vrednostih pod kazalcem
* Povečajte sliko več kot 60× (manj, če je nastavljena velikost bloka GSD), da se posamezne prikazane piksle označijo s plavajočo vrednostjo

Praktični postopek:

1. Zabeležite vrednosti nad zdravo vegetacijo, obremenjeno vegetacijo, golo tlemi in vodo
2. Pogledajte, kje se ti sklopi nahajajo na histogramu indeksa
3. Nastavite min/max, da zajamete sklop, ki vas zanima
4. Izberite način izreza — _Original Background_ ohranja vidno okolico prizora

***

## Izvoz iz peskovnika

Vse zgoraj navedeno je predogled v realnem času, dokler ga ne shranite. Gumb **Izvozi/Shrani sliko(-e)** na vrhu stranskega pasu odpre okno, ki se pomakne čez stranski pas (namesto da bi prekrilo sliko, tako da še vedno vidite, o čem se odločate).

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>### Možnosti

| Možnost                          | Učinek                                                                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Uporabi za trenutno sliko**      | Shrani točno tisto sliko, ki je prikazana, z navedenimi nastavitvami                                                                                                |
| **Uporabi za vse slike v projektu** | Ponovno izvede enako konfiguracijo na vsaki sliki v projektu. Slike brez pasov, ki jih ta indeks potrebuje, se preskočijo in se ne obravnavajo kot napake |
| **Barvna lestvica indeksa/LUT**      | Pri vsakem izvozu zapiše tudi ločeno sliko z legendo, na kateri je označen razpon vrednosti                                                                     |
| **Histogram indeksa**             | Pri vsakem izvozu zapiše tudi ločeno sliko histograma, ki prikazuje minimalne in maksimalne vrednosti podatkov ter pragove izreza                                               |

Če je **velikost bloka GSD** na zavihku slike večja od 1, vas okno na to opozori pred potrditvijo: izvoz shrani tisto, kar vidite, vključno s povprečenjem blokov. Če želite polno ločljivost, najprej nastavite GSD nazaj na 1.

### Kam se shranijo datoteke

Vsak klik na **Izvoz**dodeli**novo mapo, ki se nikoli ne ponovno uporabi**:

```
<project folder>/Sandbox_Exports/<IndexName>_<Index|LUT>_<NNN>/
```

Primeri: `Sandbox_Exports/NDVI_LUT_001/`, nato `Sandbox_Exports/NDVI_LUT_002/` za naslednji izvedbeni cikel. Številčenje se določi s pregledom vsebine, ki je že na disku, zato ostane nespremenjeno tudi po ponovnem zagonu sistema in ročnem brisanju map. Nič se nikoli ne prepiše – bistvo peskovnika je namreč primerjava posameznega poskusa z zadnjim.

V mapi, za vsako sliko:

| Datoteka                                                   | Vsebina                                                   |
| ------------------------------------------------------ | ---------------------------------------------------------- |
| `<source name>_<IndexName>_<Index\|LUT>.png`           | Renderirana slika, piksel za pikselom takšna, kot jo je prikazal pregledovalnik |
| `<source name>_<IndexName>_<Index\|LUT>_legend.png`    | Stranski datotek z barvnim prehodom, če je zahtevano                     |
| `<source name>_<IndexName>_<Index\|LUT>_histogram.png` | Stranski datotek z indeksnim histogramom, če je zahtevano                  |

Obe stranski datoteki sta vedno zapisani v **polni ločljivosti**, tudi če je glavna slika povprečena po blokih: velikost bloka je ločljivost zaslona, obe stranski datoteki pa prikazujeta dejanske indeksne vrednosti na piksel. Prav tako vsebujeta več podatkov kot različice na zaslonu – obe označujeta okno razteza _in_ dejanske najmanjše/največje vrednosti podatkov, tako da je shranjena legenda še mesece kasneje berljiva, tudi če projekt ni odprt.

### Napredek in rezultati

Izvoz celotnega projekta traja nekaj minut, zato program poroča o napredku prek kanala v realnem času, namesto da bi blokiral delovanje:

* Napredovalna vrstica prikazuje `current / total` in datoteko, ki se zapisuje
* Ko se izvoz konča, okno prikaže, koliko slik je bilo izvoženih, koliko je bilo preskočenih ter pot do izhodne mape
* Preskočene slike so navedene z razlogom (prikaže se do pet, nato pa vrstica »+N več«). Običajni razlog je sloj, ki nima kanalov, ki jih ta indeks potrebuje
* Če **nobena** slika v projektu ne more uporabiti indeksa, izvedba poroča o neuspehu, namesto da bi pustila prazno mapo

Hkrati teče le en izvoz v peskovniku. Zagon drugega izvoza, medtem ko je prvi v teku, se zavrne z jasnim sporočilom, namesto da bi se dva izvoza borila za isto projektno datoteko.

### Mreža prevzame izvedbo

Vsaka zaključena izvedba se prikaže kot samostojni gumb na orodni vrstici [mreže slik](image-grid.md) z oznako `<IndexName> <Index|LUT> <NNN>`. Tako lahko primerjate izvajanja: izvozite dvakrat z različnimi gradienti ali pragovi, nato pa preklapljajte med gumboma na mreži.

***

## Lastne formule indeksov (Chloros+)

{% hint style="info" %}
**Kje jih ustvariti**: v stranskem meniju »Sandbox« ali v**Nastavitvah projekta** pred obdelavo. Obe možnosti zapisujeta v isti seznam na ravni projekta.
{% endhint %}

1. Odprite kalkulator po meri iz spustnega menija formul indeksa (zahteva prijavo z ustreznim naročniškim paketom Chloros+)
2. Vnesite formulo z uporabo **simbolov pasovnih slotov** `x`, `y`, `z`, `a`, `b`, `c` — ne imena pasov
3. Razpoložljivi operatorji: `+`, `-`, `*`, `/`, `^` in `()` za združevanje v skupine
4. Na voljo so naslednje funkcije: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
5. Poimenujte in shranite ga – pojavi se na dnu spustnega menija formul in njegove reže povežete s povlekanjem krogov kanalov, točno tako kot pri vgrajeni prednastavitvi

```

Modified NDVI with an offset:   (y-x)/(y+x+0.5)
Simple ratio:                   y/x
Three-band difference:          (y-x)/(y+x-z)
Squared ratio:                  (y/x)^2
```

{% hint style="warning" %}
**Prilagojene formule so na voljo le v grafičnem vmesniku.** Možnost CLI/SDK `--indices` razširi 22 vgrajenih prednastavljenih imen in tiho preskoči vse ostalo, vključno z vašimi lastnimi formulami. Če želite uporabiti lastno formulo v seriji, jo nastavite v nastavitvah projekta in zaženite obdelavo ali uporabite izvozno možnost »Uporabi za vse slike v projektu« v okolju Sandbox.
{% endhint %}

***

## Odpravljanje težav

### »Ta sloj nima kanalov, ki jih ta indeks potrebuje«

Formula bere položaj kanala, ki ga trenutni sloj nima – na primer indeks s tremi mesti v datoteki z enim ali dvema kanaloma. Preklopite na večpasovno plast (odbojnost ali debayering) ali izberite indeks, ki ustreza filtru vaše kamere.

### »Ni bilo mogoče vzpostaviti povezave z ozadjem za obdelavo slik«

Ozadje ne odgovarja. Preverite zavihek »Dnevnik«; če se strežnik ponovno zaganj, se Sandbox samodejno obnovi, ko je spet na voljo.

### Slika se ni spremenila, ko sem povlekel krog

Formula še ni popolna. Nepopolna formula se obravnava kot običajno stanje med vlečenjem – nič se ne prikaže in nič se ne poroča kot napaka. Izpolnite vsa polja, ki jih formula uporablja.

### Celotna slika je enobarvna

Vaše okno izseka je verjetno daleč zunaj podatkovnega območja. Pritisnite **AUTO**, da ga poravnate z 2. ali 98. percentilom, ali vklopite histogram**INDEX**, da vidite, kje dejansko ležijo podatki.

### Izvožene barve se ne ujemajo s tistimi, ki sem jih videl

Morale bi se – pot izvoza namerno odraža predogled v realnem času, vključno z alfa v načinu izreza, povprečenje blokov pa se uporabi _po_ barvanju, točno tako, kot to stori pregledovalnik. Če se razlikujejo, preverite, ali se velikost bloka GSD med pregledovanjem in izvozom ni spremenila.

***

## Naslednji koraki

* [**Sloji slike**](image-layers.md) — na katerem sloju izvesti indeks in kaj pomenijo njegove vrednosti
* [**Odpiranje slike na celotnem zaslonu**](opening-an-image-full-screen.md) — podrobnosti o prikazu kazalca, histogramu in nadzoru GSD
* [**Formule multispektralnih indeksov**](../project-settings/multispectral-index-formulas.md) — vsaka prednastavitev na vsaki površini
* [**Nastavitve projekta**](../project-settings/project-settings.md) — vključitev najdenih nastavitev v obdelovalni cikel

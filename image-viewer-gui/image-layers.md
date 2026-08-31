# Slike v plasteh

**Spustni meni plasti** v zgornjem desnem kotu pregledovalnika slik omogoča preklapljanje med vsemi različicami slike, ki jo pregledujete – od izvirnega posnetka prek vseh obdelanih izdelkov do izračunanih indeksnih slik –, ne da bi morali zapustiti pregledovalnik.

## Kaj so sloji slik?

»Sloj« v programu Chloros je ena **datoteka izdelka**, registrirana za eno izvorno sliko. Uvoz vam prinese izvorne datoteke; obdelava doda sloj za vsak izdelek, ki ga je ustvaril programski cikel. Izvožene datoteke ohranijo ime izvorne datoteke – izdelek identificira**mapa**, ime plasti pa je oznaka programa Chloros za to mapo.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## Seznam plasti

### Vedno prisotne

| Sloj | Kaj je |
| --- | --- |
| **JPG**(ali**PNG**/**TIFF**) | Osnovna datoteka, ki je bila pridobljena z zajemom. Survey3 uvozi `.JPG` poleg vsakega `.RAW`; zajemi LATTICE prinesejo PNG ali TIFF. Označeno glede na to, kar je bilo dejansko uvoženo |
| **RAW (izvirnik)** | Izvirni surovi okvir, razbajeriran za prikaz brez uporabljenih popravkov. Na voljo od trenutka uvoza — ne potrebuje obdelave |

Posnetek LATTICE, katerega osnovna datoteka **je** surovi okvir, nima ločenega osnovnega vnosa: `RAW (Original)` ga že zajema.

### Survey3 izdelki obdelave

| Sloj | Zapisano v | Obstaja, ko |
| --- | --- | --- |
| **RAW (cilj)** | — | Okvir je bil identificiran kot tak, ki vsebuje kalibracijski cilj |
| **RAW (odbojnost)** | `Reflectance_Calibrated_Images/` | Kalibracija odbojnosti je na tem okvirju potekala uspešno |
| **Popravljeno vignetiranje**| `Vignette_Corrected_Images/` | Okvirja ni bilo mogoče kalibrirati po odbojnosti**in** je bila *korekcija vignetiranja* vklopljena |
| **Odziv senzorja**| `Sensor_Response_Images/` | Okvirja ni bilo mogoče kalibrirati glede na odbojnost**in** *popravek vinjetiranja* je bil izklopljen |
| **Bela bilanca** | `White_Balanced_Images/` | Zapisan je bil izdelek z uravnano belo bilanco |

{% hint style="info" %}
**Korekcija vinjetiranja in odziv senzorja sta alternativni možnosti, nikoli pa ne veljata hkrati.** Za vsak model kamere obstaja natanko en nekalibriran nadomestni izdelek na serijo, stikalo *Korekcija vinjetiranja* pa izbere, kateri. Glej [Nastavitve projekta](../project-settings/project-settings.md).
{% endhint %}

### Ravni LATTICE

LATTICE zajame razširitev v te ravni v enem samem koraku obdelave. Katerih ravni obstajajo, je odvisno od stikal za izvoz po posameznih izdelkih v nastavitvah projekta in od tega, kaj velja za kamero.

| Sloj | Zapisano v | Velja za |
| --- | --- | --- |
| **RAW (Debayered)** | `Debayered_Images/` | RGB in multispektralno |
| **RAW (predogled)** | `Preview_Images/` | Multispektralne (raztezek v lažnih barvah) |
| **Belina uravnotežena** | `Preview_Images/` | Glavne kamere RGB — predogled RGB je registriran pod tem imenom, da se ujema z istoimensko plastjo Survey3 |
| **RAW (sijaj)** | `Radiance_Images/` | Samo multispektralno |
| **RAW (odbojnost)** | `Reflectance_Calibrated_Images/` | Samo multispektralno in le, če okvir pokriva ustrezen zapis navzdolnega sevanja `.daq` ali cilj znotraj okvirja, ki je prestal kontrolo kakovosti |

Glavne kamere RGB nimajo radiometrije po posameznih pasovih, zato se za njih sevanje in odbojnost preskočita kot **neveljavni** — dnevnik to navaja, namesto da bi se napaka prikazala tiho.

### Plasti indeksa, LUT in peskovnika

| Vzorec plasti | Primer | Izvor |
| --- | --- | --- |
| **RAW (`<INDEX>` indeks)** | `RAW (NDVI Index)` | Ena na indeks, konfigurirana v nastavitvah projekta, izračunana med obdelavo |
| **`<INDEX>` LUT** | `NDVI LUT` | Barvno preslikana različica indeksa |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Ena na vsak izvoz [Indeksa/LUT Sandboxa](index-lut-sandbox.md) |

Če je isto ime indeksa konfigurirano več kot enkrat z različnimi nastavitvami, se drugemu in naslednjim v imenu doda številka (`RAW (NDVI2 Index)`), da se plasti med seboj razlikujejo.

***

## Uporaba izbirnika plasti

1. S klikom na sličico v mreži odprite sliko v polnem zaslonu
2. Kliknite **spustni meni plasti** v zgornjem desnem kotu pregledovalnika
3. Izberite sloj — slika se takoj posodobi

Spustni meni najprej prikaže **JPG, RAW (izvirnik), RAW (cilj), RAW (odbojnost)** v tem vrstnem redu, vse ostalo pa je navedeno za njimi v vrstnem redu, v katerem so bili izdelki registrirani.

### Prednost plasti pri navigaciji

S pritiskom na **←**/**→** se premaknete na naslednjo sliko, pri čemer se sistem poskuša ohraniti na isti plasti:

1. **Najprej natančna ujemitev** — če ima naslednja slika plast z enakim imenom, se prikaže ta. To vas ohranja na sloju `RAW (NDVI Index)` med pregledovanjem celotnega niza
2. **Nato ujemanje po vrsti** — indeksni sloj išče kateri koli indeksni sloj, LUT kateri koli LUT, odbojnost katero koli odbojnost, ciljni kateri koli ciljni, izvirni kateri koli izvirni, osnovni kateri koli osnovni
3. **Nato, samo za izvozne plasti** — ime se ohrani, tudi če seznam plasti še ni usklajen, ker datoteka že obstaja na disku. To vam omogoča pregledovanje izdelkov, medtem ko jih izvajanje še zapisuje
4. **V nasprotnem primeru** — prva razpoložljiva plast, ki je običajno osnovna slika

Datoteki sidecar `.daq` in `.csv` v projektu se pri navigaciji s puščičnimi tipkami preskočita, tako da se pri prehodu med slikami nikoli ne pristanete na posnetku svetlobnega senzorja.

Povečanje in premikanje se prenašata tudi med slikami, kar omogoča enostavno primerjavo istega položaja na polju pred in po.

***

## Razumevanje vrednosti pik po slojih

[Plošča z vrednostmi kazalca](opening-an-image-full-screen.md#cursor-values) prikazuje dejansko vrednost na kanal pod kazalcem v enoti, v kateri je sloj shranjen. Njeni stolpci se spreminjajo glede na sloj:

| Sloj | Prikazana enota | Opombe |
| --- | --- | --- |
| Osnovna (JPG / PNG / TIFF predogled) | DN, 0–255 | Prikazne vrednosti, z gama korekcijo v RGB. Samo za vizualni pregled |
| RAW (izvirnik) | DN | Surove digitalne vrednosti senzorja. Os histograma prikazuje globino: 255 (8-bitno), 4095 (12-bitno) ali 65535 (16-bitno) |
| RAW (brez bayerske matrike) | DN | Linearno, brez raztezanja prikaza |
| RAW (predogled) / uravnotežena belina | DN | Prikazni izdelek — raztegnjen ali z gama korekcijo. Ni namenjen merjenju |
| RAW (sijaj) | **W/m²/sr/nm** | Fizični sijaj v Float32. Brez stolpca DN |
| RAW (odbojnost) | DN **in %** | Odstotek izračunan po lastni lestvici datoteke — glej spodaj |
| Izvozi indeksa / LUT / peskovnika | Indeksna vrednost ali komponente RGB | Enokanalna indeksna datoteka poroča o indeksni vrednosti; datoteka LUT s barvnim preslikavanjem pa poroča o komponentah Red/Green/Blue |

### Odbojnost: lestvica velja za posamezno datoteko

{% hint style="warning" %}
**»Deljenje z 65.535« je pravilno le za Survey3.** Odbojnost LATTICE je shranjena v drugačnem merilu, zato je mešanje obeh deliteljev najpogostejši način, da dobimo vrednosti odbojnosti, ki so točno polovica tistih, ki bi morale biti.
{% endhint %}

| Vir | DN, ki ustreza odbojnosti 1,0 | Prepoznaven po |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | Oznaka XMP `Chloros:PixelScale=32768`, vtisnjena v vsak izvoz odbojnosti LATTICE. 2× rezerva pomeni, da je vrednost ρ nad 1,0 mogoče prikazati, namesto da bi bila odrezana |
| **Survey3**|**65535** | Ni oznake XMP za merilo Chloros — kalibracija Survey3 zapiše ρ × dtype-max in odreže pri vrednosti 1,0 |

Za GIS in skriptiranje: preberite `Chloros:PixelScale` iz datoteke in delite z njim. Če oznaka manjka, je datoteka v merilu Survey3 (65535). Pregledovalnik, peskovnik indeksa/LUT in izvoz indeksa vsi določajo merilo na enak način, zato je številka, ki jo preberete na kazalcu, tista, ki jo je uporabila matematična obdelava indeksa.

Shranjevanje, specifično za format, na podlagi te lestvice:

* **TIFF (32-bitni, odstotek)** shranjuje DN / 65535 kot število s plavajočo vejico
* **PNG (8-bitno)**in**JPG (8-bitno)** shranjujeta DN × 255 / 65535
* **8-bitni izvoz TIFF zajema iz 8-bitnega vira** se omeji na 0–255 namesto da bi se ponovno skaliral in namerno ne vsebuje oznake lestvice. Plošča za te datoteke prikaže le DN, brez stolpca z odstotki

### Razponi indeksnih vrednosti

| Družina indeksov | Tipični razpon | Odčitek |
| --- | --- | --- |
| Normalizirana razlika (NDVI, GNDVI, NDRE, ENDVI…) | od −1 do +1 | Zdrava vegetacija običajno 0,4–0,9; gola tla blizu 0; voda negativna |
| Prilagojeno tlem (SAVI, OSAVI, MSAVI2…) | približno od −1 do +1,5 | Podobna vrednost kot pri NDVI, pri čemer je tla v ozadju potisnjena v ozadje |
| Razmerje (GRVI, GCI, MSR, CIRE…) | brez zgornje meje | Razmerja rastejo brez omejitve, ko se pas v imenovalcu približuje ničli |
| EVI / LAI | od 0 do ~1, od 0 do ~3,5 | Oblaki in drugi zasičeni piksli potisnejo obe vrednosti izven območja — najprej jih zamaskirajte |

Glejte [Formule multispektralnih indeksov](../project-settings/multispectral-index-formulas.md) za natančno formulo vsake prednastavitve.

***

## Pogosti delovni postopki

### Primerjava pred / po

1. Izberite **RAW (Original)** in si zapomnite vinjetiranje ter nekalibrirane vrednosti
2. Preklopite na **RAW (Reflectance)**

3. Primerjajte — vinjetiranje je odstranjeno, vrednosti so kalibrirane. Povečava in premikanje ostajata nespremenjeni, tako da gledate isto površino

### Pregled enega indeksa v celotnem nizu

1. Odprite prvo obdelano sliko in izberite indeksno plast
2. Večkrat pritisnite **→** – indeksna plast sledi sliki za sliko
3. Med pregledovanjem opazujte histogram v stranskem pasu: okvir, v katerem se porazdelitev skokovito spremeni, je vredno podrobneje pregledati

### Preverjanje kalibracijskih ciljev

1. Izberite **RAW (Cilj)** na ciljnem okvirju
2. Preverite, ali je cilj jasno viden in zaznan
3. Preidite na naslednji ciljni okvir — sloj ciljev sledi

### Preverite točnost vrednosti odbojnosti

1. Izberite **RAW (Odbojnost)**

2. Preberite stolpec**%** v oknu »Cursor Values« – vrednosti so za to datoteko že pravilno prilagojene
3. Preverite skladnost z znanimi materiali v posnetku: zdrava vegetacija ima visoko vrednost NIR in nizko vrednost rdeče; kalibracijski cilj naj bi imel vrednost, ki je blizu objavljeni vrednosti odbojnosti

***

## Odpravljanje težav

### Sloj, ki sem ga pričakoval, ni v spustnem meniju

**Možni vzroki**

* Slika ni bila nikoli obdelana — obstajata le osnovni sloj in `RAW (Original)`
* V nastavitvah projekta ni označena možnost izvoza tega produkta
* Produkt ne velja za to kamero (sijaj in odbojnost na glavni kameri RGB; kateri koli indeks na enopasovni črno-beli kameri M3M)
* Kalibracija odbojnosti ni imela na čem delovati — ni bilo pokritosti navzdol usmerjenega sevanja `.daq` in ni bilo cilja znotraj okvirja, ki bi prestal kontrolo kakovosti — zato se je okvir vrnil na »Vignette Corrected« ali »Sensor Response«

**Kaj storiti**

1. Preverite dnevnik izvajanja: Chloros navaja, kdaj zahtevani izvozni produkt ni bil mogoč in zakaj
2. Preverite stikala za izvoz po posameznih produktih v [Nastavitvah projekta](../project-settings/project-settings.md)
3. Preverite, ali mapa izdelka obstaja v izhodni strukturi projekta
4. Ponovno obdelajte z omogočenim izdelkom

### Seznam plasti izgleda zastarel

Chloros med potekom izvajanja ponovno pregleda mape izdelkov projekta in popravi manjkajoče registracije plasti na podlagi dejanskega stanja na disku, zato se plast, katere izvoz je bil uspešno zaključen, samodejno prikaže v pregledu. Preklop stran od slike in nazaj prisili sistem k novi obdelavi.

### Vrednosti odbojnosti so videti polovične glede na to, kar bi morale biti

Skoraj zagotovo delite datoteko LATTICE z 65535. Uporabite `Chloros:PixelScale` (32768) ali preberite stolpec **%**, v katerem je to že upoštevano.

### Indeksna plast obstaja, vendar je slika prazna

Indeks potrebuje pasove, ki jih vaša plast nima – na primer indeks, ki bere tretji kanal, uporabljen na eno- ali dvo-kanalni datoteki. Preklopite na večpasovno plast (odbojnost ali debayering) ali izberite indeks, ki ustreza filtru kamere.

***

## Naslednji koraki

* [**Odpiranje slike v polnem zaslonu**](opening-an-image-full-screen.md) — prikaz kazalca, histogram in nadzor GSD
* [**Peskovnik indeksov/LUT**](index-lut-sandbox.md) – interaktivna vizualizacija indeksov in izvoz
* [**Formule multispektralnih indeksov**](../project-settings/multispectral-index-formulas.md) — referenca indeksov
* [**Zaključek obdelave**](../processing-images-gui/finishing-the-processing.md) — drevesna struktura izhodne mape, na katero kažejo te plasti

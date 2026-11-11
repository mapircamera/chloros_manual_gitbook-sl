# Slike v plasteh

**Spustni meni plasti** v zgornjem desnem kotu pregledovalnika slik omogoča preklapljanje med vsemi različicami slike, ki jo pregledujete – od izvirnega posnetka prek vseh obdelanih izdelkov do izračunanih indeksnih slik –, ne da bi morali zapustiti pregledovalnik.

## Kaj so sloji slik?

»Sloj« v programu Chloros je ena **datoteka izdelka**, registrirana za eno izvorno sliko. Uvoz vam prinese izvorne datoteke; obdelava doda sloj za vsak izdelek, ki ga je ustvaril programski cikel. Izvožene datoteke ohranijo ime izvorne datoteke – izdelek identificira**mapa**, ime plasti pa je oznaka, ki jo za to mapo dodeli Chloros.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## Seznam plasti

### Vedno prisotne

| Plast | Kaj je |
| --- | --- |
| **JPG**(ali**PNG**/**TIFF**) | Osnovna datoteka, ki je bila uvožena skupaj z zajemom. Survey3 uvozi datoteko `.JPG` poleg vsake datoteke `.RAW`; zajemi LATTICE prinesejo predogled v formatu PNG ali TIFF. Označena glede na to, kar je bilo dejansko uvoženo |
| **RAW (izvirnik)** | Izvirni surovi okvir, razbajeriran za prikaz brez uporabljenih popravkov. Na voljo od trenutka uvoza — ne potrebuje obdelave |

Posnetek LATTICE, katerega osnovna datoteka **je** surovi okvir, nima ločenega osnovnega vnosa: `RAW (Original)` ga že zajema.

### Izdelki obdelave v programu Survey3



| Sloj | Zapisan v | Obstaja, ko |
| --- | --- | --- |
| **RAW (cilj)** | — | Okvir je bil identificiran kot tak, ki vsebuje kalibracijski cilj |
| **RAW (odbojnost)** | `Reflectance_Calibrated_Images/` | Kalibracija odbojnosti je bila na tem okviru uspešno izvedena |
| **Popravljeno vinjetiranje**| `Vignette_Corrected_Images/` | Okvirja ni bilo mogoče kalibrirati glede na odbojnost**in** je bila *korekcija vinjetiranja* vklopljena |
| **Odziv senzorja**| `Sensor_Response_Images/` | Okvirja ni bilo mogoče kalibrirati glede na odbojnost**in** *korekcija vinjetiranja* je bila izklopljena |
| **Bela bilanca** | `White_Balanced_Images/` | Zapisan je bil izdelek z uravnano belo bilanco |

{% hint style="info" %}
**Korekcija vinjetiranja in odziv senzorja sta alternativni možnosti, nikoli pa ne veljata obe hkrati.** Za vsak model kamere obstaja natanko en nekalibriran nadomestni izdelek na vsak izvedbeni cikel, stikalo *Korekcija vinjetiranja* pa izbere, kateri. Glej [Nastavitve projekta](../project-settings/project-settings.md).
{% endhint %}

### Ravni LATTICE

LATTICE zajame razširitev v te ravni v enem samem koraku obdelave. Katere ravni obstajajo, je odvisno od stikal za izvoz po posameznih izdelkih v nastavitvah projekta in od tega, kaj velja za zadevno kamero.

| Sloj | Zapisano v | Velja za |
| --- | --- | --- |
| **RAW (Debayered)** | `Debayered_Images/` | RGB in multispektralno |
| **RAW (predogled)** | `Preview_Images/` | Multispektralno (raztezek lažnih barv) |
| **Bela bilanca** | `Preview_Images/` | Glavne kamere RGB — predogled RGB je registriran pod tem imenom, da se ujema z istoimensko plastjo v programu Survey3 |
| **RAW (sijaj)** | `Radiance_Images/` | Samo multispektralno |
| **RAW (odbojnost)** | `Reflectance_Calibrated_Images/` | Samo multispektralno in le, če okvir pokriva ustrezen zapis `.daq` za navzdolno sevanje ali cilj znotraj okvirja, ki je prestal kontrolo kakovosti |

Glavne kamere RGB nimajo radiometrije po posameznih pasovih, zato se za njih sevanje in odbojnost preskočita kot **neveljavni** — dnevnik to navaja, namesto da bi se napaka prikazala tiho.

{% hint style="info" %}
**Surove in debayirane plasti so usklajene s produkti (novost v različici 1.3.0).**Surove in debayirane datoteke LATTICE se shranijo v izvorni orientaciji senzorja, v XMP pa se zabeleži orientacija, ki jim pripada; ostali produkti se shranijo z že uporabljeno orientacijo. Pregledovalnik in mreža slik za prikaz preklopita med**RAW (Original)**in**RAW (Debayered)** glede na ta zapis, tako da se slika ne obrača več, ko preklapljate med slojem izdelka in surovim slojem. To velja le za prikaz — datoteke se nikoli ne prepišejo. Datoteke brez zapisa o orientaciji (posnete pred različico 1.3.0 ali ustvarjene z drugo programsko opremo) se prikazujejo takšne, kot so shranjene. Glejte [Izhodni slikovni formati](../output-image-formats.md#orientation).
{% endhint %}

Maske omejevanja (`*_clampmask.tif` v mapi `clamp_masks/`) niso plasti: Chloros jih pri uvozu preskoči. Glejte [Izhodni formati slik](../output-image-formats.md#clamp-masks).

### Plasti indeksa, LUT in peskovnika

| Vzorec plasti | Primer | Od kod izhaja |
| --- | --- | --- |
| **RAW (`<INDEX>` Indeks)** | `RAW (NDVI Index)` | Ena na indeks, nastavljen v nastavitvah projekta, izračunana med obdelavo |
| **`<INDEX>` LUT** | `NDVI LUT` | Barvno preslikana različica indeksa |
| **Peskovnik (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Ena na izvoz [indeksa/LUT v peskovniku](index-lut-sandbox.md) izvoz |

Če je isto ime indeksa konfigurirano več kot enkrat z različnimi nastavitvami, se drugemu in naslednjim v imenu doda številka (`RAW (NDVI2 Index)`), da se plasti lahko razlikujejo.

***

## Uporaba izbirnika plasti

1. S klikom na miniaturno sliko v mreži odprite sliko v polnem zaslonu
2. Kliknite **spustni meni plasti** v zgornjem desnem kotu pregledovalnika
3. Izberite plast — slika se takoj posodobi

Spustni meni najprej prikaže **JPG, RAW (izvirnik), RAW (cilj), RAW (odbojnost)** v tem vrstnem redu, vse ostalo pa navaja v vrstnem redu, v katerem so bili izdelki registrirani.

### Prednost sloja pri navigaciji

S pritiskom na **←**/**→** se premaknete na naslednjo sliko in sistem poskuša ohraniti isti sloj:

1. **Najprej natančna ujemitev** — če ima naslednja slika sloj z istim imenom, se prikaže ta. To je tisto, kar vas ob pregledovanju celotnega niza ohranja na sloju `RAW (NDVI Index)`
2. **Nato ujemanje po vrsti** — indeksni sloj išče kateri koli indeksni sloj, LUT kateri koli LUT, reflektanca reflektanco, ciljni sloj ciljni sloj, izvirnik izvirnik, osnovni sloj osnovni sloj
3. **Nato, samo za izvozne plasti** — ime se ohrani, tudi če seznam plasti še ni usklajen, ker datoteka že obstaja na disku. To vam omogoča pregledovanje izdelkov, medtem ko jih program še zapisuje
4. **V nasprotnem primeru** — prva razpoložljiva plast, ki je običajno osnovna slika

Datoteke sidecar `.daq` in `.csv` v projektu se pri navigaciji s puščičnimi tipkami preskočijo, tako da se pri prehodu med slikami nikoli ne ustavi na posnetku svetlobnega senzorja.

Povečanje in premikanje po sliki se prenašata tudi med slikami, kar omogoča enostavno primerjavo istega položaja na polju pred in po.

***

## Razumevanje vrednosti pikslov po slojih

[Plošča z vrednostmi kazalca](opening-an-image-full-screen.md#cursor-values) prikazuje dejansko vrednost na kanal pod vašim kazalcem v enoti, v kateri je sloj shranjen. Njeni stolpci se spreminjajo glede na sloj:

| Sloj | Prikazana enota | Opombe |
| --- | --- | --- |
| Osnovna (predogled JPG / PNG / TIFF) | DN, 0–255 | Vrednosti za prikaz, z gama korekcijo na RGB. Samo za vizualni pregled |
| RAW (izvirnik) | DN | Surove digitalne vrednosti senzorja. Os histograma prikazuje globino: 255 (8-bitno), 4095 (12-bitno) ali 65535 (16-bitno) |
| RAW (brez bayerske matrike) | DN | Linearno, brez raztezanja prikaza |
| RAW (predogled) / uravnotežena belina | DN | Prikazni izdelek — raztegnjen ali s popravkom gama. Ni za merjenje |
| RAW (sevanje) | **W/m²/sr/nm** | Fizično sevanje v formatu Float32. Brez stolpca DN |
| RAW (odbojnost) | DN **in %** | Odstotek, izračunan po lastni lestvici datoteke – glej spodaj |
| Izvozi indeksa / LUT / peskovnika | Indeksna vrednost ali RGB-komponente | Enokanalna indeksna datoteka prikazuje indeksno vrednost; datoteka LUT z barvnim preslikavanjem prikazuje Red /Green /Blue-komponente |

### Odbojnost: lestvica je določena za vsako datoteko posebej

{% hint style="warning" %}
**»Deljenje z 65.535« je pravilno le za program Survey3.** Odbojnost v programu LATTICE je shranjena v drugačni lestvici, zato je mešanje obeh deliteljev najpogostejši način, da dobimo vrednosti odbojnosti, ki so točno polovica tistih, ki bi morale biti.
{% endhint %}

| Vir | DN, ki ustreza odbojnosti 1,0 | Prepoznaven po |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | Oznaka XMP `Chloros:PixelScale=32768`, vtisnjena v vsak izvoz odbojnosti LATTICE. Dvakratni rezervni razpon pomeni, da je ρ nad 1,0 mogoče prikazati, namesto da bi bil odrezan |
| **Survey3**| **65535** | Ni XMP oznake za lestvico Chloros — kalibracija Survey3 zapiše ρ × dtype-max in odreže pri 1,0 |

Za GIS in skripte: iz datoteke preberite `Chloros:PixelScale` in delite z njim. Če oznaka manjka, je datoteka v merilu Survey3 (65535). Pregledovalnik, peskovnik indeksa/LUT in izvoz indeksa vsi določajo merilo na enak način, zato je številka, ki jo preberete na kazalcu, tista, ki jo je uporabila indeksna matematika.

Shranjevanje, specifično za posamezne formate, na podlagi te lestvice:

* **TIFF (32-bitni, odstotek)** shranjuje DN / 65535 kot plavajočo vejico
* **PNG (8-bitni)**in**JPG (8-bitni)** shranjujeta DN × 255 / 65535
* **8-bitni DN iz 8-bitnega zajema** ali datoteka odbojnosti, zapisana pred različico 1.3.0, je omejena na območje 0–255 in nima oznake lestvice. Plošča za te datoteke prikaže le DN, brez stolpca z odstotki

### Razponi indeksnih vrednosti

| Družina indeksov | Tipični razpon | Odčitek |
| --- | --- | --- |
| Normalizirana razlika (NDVI, GNDVI, NDRE, ENDVI…) | od −1 do +1 | Zdrava vegetacija običajno 0,4–0,9; gola tla blizu 0; voda negativna |
| Prilagojeni tlom (SAVI, OSAVI, MSAVI2…) | približno od −1 do +1,5 | Podobna vrednost kot pri NDVI, vendar z zatiranim ozadjem tal |
| Razmerje (GRVI, GCI, MSR, CIRE…) | neomejeno nad | Razmerja rastejo brez omejitve, ko se pas imenovalca približuje ničli |
| EVI / LAI | od 0 do ~1, od 0 do ~3,5 | Oblaki in drugi zasičeni piksli potisnejo obe vrednosti izven območja — jih najprej zamaskirajte |

Glejte [Formule multispektralnih indeksov](../project-settings/multispectral-index-formulas.md) za natančno formulo, na kateri temelji vsaka prednastavitev.

***

## Pogosti delovni postopki

### Primerjava pred in po

1. Izberite **RAW (izvirnik)** in si zabeležite vinjetiranje ter nekalibrirane vrednosti
2. Preklopite na **RAW (odbojnost)**

3. Primerjajte — vinjetiranje je odstranjeno, vrednosti so kalibrirane. Povečava in premikanje ostajata nespremenjeni, tako da gledate isto površino

### Pregled enega indeksa v celotnem nizu

1. Odprite prvo obdelano sliko in izberite indeksno plast
2. Večkrat pritisnite **→** — indeksna plast sledi sliki za sliko
3. Med pregledovanjem opazujte histogram v stranskem pasu: okvir, v katerem se porazdelitev skokovito spremeni, je vredno podrobneje pregledati

### Preverjanje kalibracijskih ciljev

1. Izberite **RAW (Cilj)** na ciljnem okvirju
2. Preverite, ali je cilj jasno viden in zaznan
3. Preidite na naslednji ciljni okvir — sloj ciljev sledi

### Preverite točnost vrednosti odbojnosti

1. Izberite **RAW (Odbojnost)**

2. Preberite stolpec**%** v oknu »Cursor Values« — vrednosti so za to datoteko že pravilno prilagojene
3. Preverite skladnost z znanimi materiali v posnetku: zdrava vegetacija ima visoko vrednost v NIR-območju in nizko v rdečem; kalibracijska tarča naj bi kazala vrednost, ki je blizu objavljeni odbojnosti

***

## Odpravljanje težav

### Sloj, ki sem ga pričakoval, ni v spustnem meniju

**Možni vzroki**

* Slika ni bila nikoli obdelana – obstajata le osnovni sloj in `RAW (Original)`
* V nastavitvah projekta ni označena možnost izvoza izdelka
* Produkt se ne uporablja za to kamero (sevanje in odbojnost na RGB-masterju; kateri koli indeks na enopasovni M3M mono kameri)
* Kalibracija odbojnosti ni imela na čem delovati — ni bilo pokritosti navzdol usmerjenega sevanja `.daq` in ni bilo cilja znotraj okvirja, ki bi prestal kontrolo kakovosti — zato se je okvir vrnil na »Vignette Corrected« ali »Sensor Response«

**Kaj storiti**

1. Preverite dnevnik izvedbe: Chloros navaja, kdaj zahtevani izvozni produkt ni bil mogoč in zakaj
2. Preverite stikala za izvoz po posameznih produktih v [Nastavitvah projekta](../project-settings/project-settings.md)
3. Preverite, ali mapa z izdelkom obstaja v izhodni strukturi projekta
4. Ponovno obdelajte z omogočenim izdelkom

### Seznam plasti izgleda zastarel

Program Chloros med potekom izvajanja ponovno pregleda mape izdelkov projekta in popravi manjkajoče registracije plasti na podlagi dejanskega stanja na disku, zato se plast, katere izvoz je bil uspešno zaključen, običajno sama prikaže v pregledu. Preklop stran od slike in nazaj prisili program, da izvede novo rešitev.

### Vrednosti odbojnosti so polovične glede na to, kar bi morale biti

Skoraj zagotovo delite datoteko LATTICE z 65535. Uporabite `Chloros:PixelScale` (32768) ali preberite stolpec **%**, v katerem je to že upoštevano.

### Indeksna plast obstaja, vendar je slika prazna

Indeks potrebuje pasove, ki jih vaša plast nima – na primer indeks, ki bere tretji kanal, uporabljen na eno- ali dvo-kanalni datoteki. Preklopite na večpasovno plast (odbojnost ali debayering) ali izberite indeks, ki ustreza filtru kamere.

***

## Naslednji koraki

* [**Odpiranje slike v polnem zaslonu**](opening-an-image-full-screen.md) — prikaz kazalca, histogram in nadzor GSD
* [**Peskovnik indeksov/LUT**](index-lut-sandbox.md) — interaktivna vizualizacija indeksov in izvoz
* [**Formule za multispektralne indekse**](../project-settings/multispectral-index-formulas.md) — referenca za indekse
* [**Zaključek obdelave**](../processing-images-gui/finishing-the-processing.md) — drevesna struktura izhodne mape, na katero kažejo te plasti

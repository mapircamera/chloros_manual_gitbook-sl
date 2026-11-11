---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Formule za multispektralne indekse

Spodnje formule za indekse uporabljajo kombinacijo območij povprečne prepustnosti filtrov Survey3:

<table><thead><tr><th align="center">Barva filtra Survey3</th><th width="196.199951171875" align="center">Ime filtra Survey3</th><th width="159.800048828125" align="center">Območje prepustnosti (FWHM)</th><th align="center">Povprečna prepustnost</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re-RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>

Pri uporabi teh formul se ime lahko konča z »\_1« ali »\_2«, kar ustreza temu, kateri NIR-filter, NIR1 ali NIR2, je bil uporabljen.

Za kamere LATTICE M3C (Bayerjev trojni pasovni filter) isti indeksni motor uporablja pasove filtra M3C:

| Filter M3C | Pas 1 (središče/FWHM) | Pas 2 (središče/FWHM) | Pas 3 (središče/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

Kamere LATTICE M3M so enopasovne (en ozkopasovni filter na kamero), zato se za posamezno sliko M3M ne izračunajo večpasovni indeksi. Za izračun indeksov z M3M združite dve ali več kamer v poravnani večpasovni niz in uporabite LATTICE-jev indeksni motor (`chloros-cli lattice index` ali indeksni kalkulator v grafičnem vmesniku v realnem času).

***

## Kje deluje posamezno ime indeksa

Program Chloros ima **tri** površine indeksov, njihovi seznami prednastavitev pa niso enaki. V tem poglavju preverite, ali bo ime delovalo tam, kjer ga nameravate uporabiti.

| Kje ste | Kateri seznam velja | Število |
| --- | --- | --- |
| Nastavitve projekta → Indeks → Dodaj indeks (grafični vmesnik) | Površina 1 | 27 |
| Pregledovalnik slik [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) (grafični vmesnik) | Površina 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | Surface 2 | 22 |
| SDK `process_folder(indices=[...])` | Surface 2 | 22 |
| `chloros-cli lattice index --preset` | Surface 3 | 22 (druga številka 22) |
| Zavihek »Kamere« – kalkulator indeksa v realnem času | Surface 3 | 22 (druga številka 22) |

Surface 1 in 2 delujeta na **enem posnetku naenkrat iz ene kamere**, pri čemer uporabljata simbolna mesta `x`/`y`/`z`(/`a`), vezana na filtrske kanale te kamere. Površina 3 deluje na**usklajenem večpasovnem nizu** — več kamer LATTICE, ki so usklajene v en kockast niz — in se na kanale sklicuje z imeni v malih črkah.

### 1. Spustni meni »GUI Project Settings / Image Viewer sandbox« — 27 formul

Spustni meni jih navaja v tem vrstnem redu (to je vrstni red vstavljanja, ne abecedni):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

V grafičnem vmesniku (GUI) povlečete filtrske kanale vaše kamere na prostore za pasove v formuli, tako da je mogoče katero koli formulo uporabiti s katero koli dodelitvijo pasov, ki jo vaša kamera podpira. Uporabniške formule, ki ste jih shranili, so dodane pod tem seznamom.

**Pet formul, ki so na voljo izključno v grafičnem vmesniku** — tiste, ki jih seznam CLI/SDK `--indices` ne sprejema — so implementirane kot:

| Prednastavitev, ki je na voljo izključno v grafičnem vmesniku | Formula (kot je implementirana) | Mesta |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a (štiri vstavna mesta) |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

Predvideno preslikavanje za vsako od njih je navedeno v posebnem poglavju nižje na tej strani (na primer GARI pričakuje x=Green, y=NIR, z=Blue, a=Red). GARI je edina formula v Chloros, ki uporablja četrto mesto.

### 2. CLI / SDK `--indices` razširitev imena — 22 prednastavitev

Možnost `chloros-cli process --indices` (in parameter SDK `indices`) sprejema naslednja imena prednastavitev:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**Neznana imena indeksov se tiho preskočijo.** Ime, ki ni na tem seznamu (vključno s petimi formulami, namenjenimi izključno grafičnemu vmesniku: `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` ter vseh po meri ustvarjenih formul, ki ste jih shranili v grafičnem vmesniku) se izpusti le z obvestilom v dnevniku – izvajanje se nadaljuje brez tega indeksa, izvajanje pa se še vedno poroča kot uspešno. Obvestilo se prikaže kot:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

Imena se primerjajo brez upoštevanja velikosti črk po odstranitvi presledkov, zato so `ndvi`, `NDVI` in ` NDVI ` ista prednastavitev. Prednastavitev se prav tako preskoči, če zahteva pas, ki ga filter vaše kamere ne zagotavlja.
{% endhint %}

Natančne formule, kot so implementirane (simboli `x`/`y`/`z` so prostori za pasove; privzeta preslikava je prikazana za vsako prednastavitev):

| Prednastavitev | Formula (kot je implementirana) | Privzeti filter | Mesta (x, y, z) |
| --- | --- | --- | --- |
| NDVI | `(y-x)/(y+x)` | RGN | Red, NIR |
| GNDVI | `(y-x)/(y+x)` | RGN | Green, NIR |
| NDRE | `(y-x)/(y+x)` | RE | RE, NIR |
| OSAVI | `(y-x)/(y+x+0.16)` | RGN | Red, NIR |
| SAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Red, NIR |
| MSAVI2 | `(2*y+1-sqrt((2*y+1)*(2*y+1)-8*(y-x)))/2` | RGN | Red, NIR |
| EVI | `2.5*(y-x)/(y+6*x-7.5*z+1)` | RGN | Red, NIR, Blue |
| MSR | `((y/x)-1)/(sqrt(y/x)+1)` | RGN | Red, NIR |
| TDVI | `1.5*(y-x)/sqrt(y*y+x+0.5)` | RGN | Red, NIR |
| LAI | `3.618*(2.5*(y-x)/(y+6*x-7.5*z+1))-0.118` | RGN | Red, NIR, Blue |
| GCI | `(y/x)-1` | RGN | Green, NIR |
| GRVI | `y/x` | RGN | Green, NIR |
| GSAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Green, NIR |
| GOSAVI | `(y-x)/(y+x+0.16)` | RGN | Green, NIR |
| NLI | `((y*y)-x)/((y*y)+x)` | RGN | Red, NIR |
| MNLI | `((y*y-x)*(1+0.5))/((y*y)+x+0.5)` | RGN | Red, NIR |
| RDVI | `(y-x)/sqrt(y+x)` | RGN | Red, NIR |
| WDRVI | `(0.2*y-x)/(0.2*y+x)` | RGN | Red, NIR |
| CVI | `(z/y)*(x/y)` | RGN | Red, Green, NIR |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | NGB | NIR, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | 

Ko vnesete samo ime, na primer `NDVI`, mora program Chloros določiti, kateri kanal katere datoteke bere vsak simbol. Uporabi to tabelo, ki filter kodo preslika na indeks v matrici za vsak kanal:

| Filter koda | Kanal → indeks v matrici |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red` se sprejme kot nadomestno ime za Orange, prav tako 0) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

**Privzeti filter** prednastavitve (stolpec »Privzeti filter« zgoraj) se uporabi, kadar projekt vsebuje slike s tem filtrom. Če jih ne vsebuje, program Chloros pregleda filtre, ki so dejansko prisotni v projektu, v vrstnem redu `RGN, OCN, NGB, RGB, RE, NIR`, in izbere prvega, ki lahko zagotovi vse kanale, ki jih prednastavitev potrebuje. Če noben ne more, se prednastavitev za to izvedbo opusti. Zato `NDVI`, zahtevan za nabor podatkov, ki vsebuje samo OCN, še vedno daje smiseln rezultat — se namreč veže na Orange in NIR pozicije OCN.

Nizi modela LATTICE M3C nosijo filter s predpono `F` (`LATT-M3C-L41-FRGN`), vendar se predpona izpusti, ko se koda filtra odčita s slike, zato kamera FRGN razreši prek zgornje vrstice `RGN` in ne potrebuje posebne obdelave.

### 3. Indeksni motor LATTICE (`lattice index --preset`, kalkulator indeksa v realnem času) — 22 prednastavitev

Motor LATTICE deluje na poravnanih večpasovnih nizih (živi nizi ali izvoženi večpasovni TIFF-ji) in uporablja imena kanalov z malimi črkami (`red`, `green`, `blue`, `red_edge`, `nir`). Njegov seznam prednastavitev se razlikuje od zgornjih dveh:

| Prednastavitev | Formula | Kanali |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | rdeča, NIR |
| GNDVI | `(nir - green) / (nir + green)` | zelena, NIR |
| BNDVI | `(nir - blue) / (nir + blue)` | modra, NIR |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | rdeči rob, NIR |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | modra, zelena, NIR |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | rdeča, NIR |
| OSAVI | `(nir - red) / (nir + red + 0.16)` | rdeča, NIR |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | rdeča, NIR |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | modra, rdeča, NIR |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | rdeča, NIR |
| CVI | `nir * red / green**2` | rdeča, zelena, NIR |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | rdeča, NIR |
| TDVI | `1.5 * (nir - red) / sqrt(nir**2 + red + 0.5)` | rdeča, NIR |
| LAI | `3.618 * (2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)) - 0.118` | modra, rdeča, NIR |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | rdeča, zelena, modra |
| NGRDI | `(green - red) / (green + red)` | rdeča, zelena |
| VARI | `(green - red) / (green + red - blue)` | rdeča, zelena, modra |
| TGI | `green - 0.39*red - 0.61*blue` | rdeča, zelena, modra |
| EXG | `2*green - red - blue` | rdeča, zelena, modra |
| CIRE | `(nir / red_edge) - 1` | rdeča_rob, nir |
| CIGREEN | `(nir / green) - 1` | zelena, nir |
| NDWI | `(green - nir) / (green + nir)` | zelena, nir |

Za tiskanje te tabele iz vaše nameščene različice zaženite `chloros-cli lattice index --list-presets`, in `--list-gradients` za razpoložljive barvne prehode. Pri simbolih kanalov se upošteva velika in mala črka, zato morajo ustrezati imenom prednastavitev v malih črkah (npr. `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI – indeks vegetacije na podlagi klorofila

Ta indeks je občutljiv na vsebnost klorofila v listih v širokem razponu gostot krošnj. Uporablja pasove NIR, rdeči in zeleni.

$$
CVI = NIR * {Red \over Green^{2}}
$$

V grafičnem uporabniškem vmesniku (GUI) ter na seznamu prednastavitev za CLI/SDK je to simbolna formula `(z/y)*(x/y)`, kjer x = Red, y = Green, z = NIR (pasovi kamere RGN); prednastavitev indeksnega motorja LATTICE je `nir * red / green**2`. Vsak površinski izračun izračuna isti indeks.

_Sklic: Vincini, M., E. Frazzi in P. D'Alessio. »Širokopasovni indeks vegetacije na podlagi klorofila v listih na ravni krošnje.« Precision Agriculture 9 (2008): 303–319._

***

## ENDVI – Izboljšani normalizirani razlikovni vegetacijski indeks

Ta indeks poleg NIR in zelenega kanala uporablja tudi modri kanal in je priljubljen pri kamerah s filtrom NGB, kjer modri pas nadomešča rdečega.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

Izvedba je simbolna formula `((x+y)-(2*z))/((x+y)+(2*z))` — kanala NIR in Green vaše kamere dodelite v reži x/y, kanal Blue pa v z (za kamero NGB: x=NIR, y=Green, z=Blue).

***

## EVI – izboljšani indeks vegetacije

Ta indeks je bil prvotno razvit za uporabo s podatki MODIS kot izboljšava indeksa NDVI z optimizacijo signala vegetacije na območjih z visokim indeksom listne površine (LAI). Najbolj uporaben je v regijah z visokim LAI, kjer lahko pride do zasičenja indeksa NDVI. Uporablja območje modre odbojnosti za korekcijo signalov talnega ozadja in zmanjšanje vplivov atmosfere, vključno z razprševanjem aerosolov.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Vrednosti EVI za piksle vegetacije morajo biti v razponu od 0 do 1. Svetli elementi, kot so oblaki in bele stavbe, skupaj s temnimi elementi, kot je voda, lahko povzročijo neobičajne vrednosti pikslov na sliki EVI. Pred ustvarjanjem slike EVI morate iz slike odbojnosti izločiti oblake in svetle elemente ter po želji določiti prag za vrednosti pikslov od 0 do 1.

_Sklic: Huete, A., et al. »Pregled radiometrične in biofizikalne zmogljivosti indeksov vegetacije MODIS.« Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 – Indeks gozdnega pokrova 1

_Samo v grafičnem vmesniku (GUI) – ni na voljo kot prednastavitev CLI/SDK `--indices`._

Ta indeks ločuje gozdne krošnje od drugih vrst vegetacije z uporabo multispektralnih slik odbojnosti, ki vključujejo pas rdečega roba.

$$
FCI1 = Red * RedEdge
$$

Gozdna območja bodo imela nižje vrednosti FCI1 zaradi manjše odbojnosti dreves in prisotnosti senc znotraj krošnj.

_Sklic: Becker, Sarah J., Craig S.T. Daughtry in Andrew L. Russ. »Robustni indeksi gozdnega pokrova za multispektralne slike.« Photogrammetric Engineering & Remote Sensing 84.8 (2018): 505–512._

***

## FCI2 – Indeks gozdnega pokrova 2

_Samo v grafičnem vmesniku (GUI) – ni na voljo kot prednastavitev CLI/SDK `--indices`._

Ta indeks ločuje gozdno krošnjo od drugih vrst vegetacije z uporabo multispektralnih slik odbojnosti, ki ne vključujejo pasu rdečega roba.

$$
FCI2 = Red * NIR
$$

Gozdna območja bodo imela nižje vrednosti indeksa FCI2 zaradi manjše odbojnosti dreves in prisotnosti senc znotraj krošnj.

_Sklic: Becker, Sarah J., Craig S.T. Daughtry in Andrew L. Russ. »Robustni indeksi gozdnega pokrova za multispektralne slike.« Photogrammetric Engineering & Remote Sensing 84.8 (2018): 505–512._

***

## GEMI – Indeks globalnega okoljskega spremljanja

_Samo GUI – ni na voljo kot prednastavitev CLI/SDK `--indices`._

Ta nelinearni indeks vegetacije se uporablja za globalno spremljanje okolja na podlagi satelitskih posnetkov in poskuša popraviti vplive atmosfere. Podoben je indeksu NDVI, vendar je manj občutljiv na atmosferske vplive. Na njega vpliva gola tla; zato se njegova uporaba ne priporoča na območjih z redko ali zmerno gosto vegetacijo.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Kjer:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Sklic: Pinty, B., in M. Verstraete. »GEMI: nelinearni indeks za spremljanje globalne vegetacije s satelitov«. Vegetation 101 (1992): 15–20._

***

## GARI – indeks Green, odporen na vplive atmosfere

_Samo GUI – ni na voljo kot prednastavitev CLI/SDK `--indices`._

Ta indeks je občutljivejši na širok razpon koncentracij klorofila in manj občutljiv na vplive atmosfere kot NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Gama konstanta je tehtalna funkcija, ki je odvisna od razmer aerosolov v atmosferi. ENVI uporablja vrednost 1,7, kar je priporočena vrednost iz Gitelson, Kaufman in Merzylak (1996, stran 296).

_Sklic: Gitelson, A., Y. Kaufman in M. Merzylak. »Uporaba zelenega kanala pri daljinskem zaznavanju globalne vegetacije iz EOS-MODIS.« Remote Sensing of Environment 58 (1996): 289–298._

***

## GCI – indeks zelenega klorofila (Green Chlorophyll Index)

Ta indeks se uporablja za oceno vsebnosti klorofila v listih pri širokem spektru rastlinskih vrst.

$$
GCI = {NIR \over Green} - 1
$$

Širok spekter valovnih dolžin v bližnjem infrardečem območju (NIR) in zelenem območju omogoča boljšo napoved vsebnosti klorofila, hkrati pa zagotavlja večjo občutljivost in višje razmerje med signalom in šumom.

_Viri: Gitelson, A., Y. Gritz in M. Merzlyak. »Povezave med vsebnostjo klorofila v listih in spektralno odbojnostjo ter algoritmi za neporušno ocenjevanje klorofila v listih višjih rastlin.« Journal of Plant Physiology 160 (2003): 271–282._

***

## GLI – indeks zelenih listov (Green Leaf Index)

Ta indeks je bil prvotno zasnovan za uporabo z digitalno RGB-kamero za merjenje pokritosti s pšenico, pri čemer se rdeče, zelene in modre digitalne vrednosti (DN) gibljejo od 0 do 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Vrednosti GLI-ja segajo od -1 do +1. Negativne vrednosti predstavljajo tla in nežive elemente, pozitivne vrednosti pa zelene liste in stebla.

_Sklic: Louhaichi, M., M. Borman in D. Johnson. »Prostorsko locirana platforma in letalska fotografija za dokumentiranje vplivov paše na pšenico.« Geocarto International 16, št. 1 (2001): 65–70._

***

## GNDVI – indeks normalizirane razlike v zeleni vegetaciji (Green)

Ta indeks je podoben indeksu NDVI, le da meri zeleni spekter od 540 do 570 nm namesto rdečega spektra. Ta indeks je bolj občutljiv na koncentracijo klorofila kot indeks NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Vir: Gitelson, A., in M. Merzlyak. »Daljinsko zaznavanje koncentracije klorofila v listih višjih rastlin.« Advances in Space Research 22 (1998): 689–692._

***

## GOSAVI – indeks vegetacije, optimiziran za zeleno barvo in prilagojen tleh (Green Optimized Soil Adjusted Vegetation Index)

Ta indeks je bil prvotno zasnovan za barvno-infrardečo fotografijo z namenom napovedovanja potreb po dušiku pri koruzi. Je podoben indeksu OSAVI, vendar rdeči pas nadomesti z zelenim.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Sklic: Sripada, R., et al. »Določanje potreb po dušiku za koruzo med rastno sezono z uporabo barvno-infrardeče fotografije iz zraka.« Doktorska disertacija, North Carolina State University, 2005._

***

## GRVI – indeks vegetacije z razmerjem zelenega (Green Ratio Vegetation Index)

Ta indeks je občutljiv na hitrost fotosinteze v gozdnih krošnjah, saj na odbojnost zelenega in rdečega valovnega območja močno vplivajo spremembe v listnih pigmentih.

$$
GRVI = {NIR \over Green }
$$

_Sklic: Sripada, R., et al. »Barvna infrardeča fotografija iz zraka za določanje potreb po dušiku v zgodnji rastni sezoni pri koruzi.« Agronomy Journal 98 (2006): 968–977._

***

## GSAVI – indeks vegetacije, prilagojen zeleni barvi tal (Green Soil Adjusted Vegetation Index)

Ta indeks je bil prvotno zasnovan za uporabo s barvno-infrardečo fotografijo za napovedovanje potreb po dušiku pri koruzi. Je podoben indeksu SAVI, vendar rdeči pas nadomešča z zelenim.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Vir: Sripada, R., et al. »Določanje potreb po dušiku za koruzo med rastno sezono z uporabo barvne infrardeče fotografije iz zraka.« Doktorska disertacija, North Carolina State University, 2005._

***

## LAI – indeks listne površine

Ta indeks se uporablja za oceno listne pokritosti ter za napovedovanje rasti in pridelka. Program ENVI izračuna zeleni LAI z naslednjo empirično formulo iz Boegh et al. (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Kjer je EVI:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Visoke vrednosti LAI se običajno gibljejo med približno 0 in 3,5. Vendar pa lahko vrednosti LAI presegajo 3,5, če prizor vsebuje oblake in druge svetle elemente, ki povzročajo zasičene piksle. Pred ustvarjanjem slike LAI je priporočljivo iz prizora izločiti oblake in svetle elemente.

_Sklic: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde in A. Thomsen. »Letalski multispektralni podatki za količinsko opredelitev indeksa listne površine, koncentracije dušika in fotosintetske učinkovitosti v kmetijstvu.« Remote Sensing of Environment 81, št. 2–3 (2002): 179–193._

***

## Indeks klorofila listov (LCI)

_Samo v grafičnem vmesniku (GUI) – ni na voljo kot prednastavitev CLI/SDK `--indices`._

Ta indeks se uporablja za oceno vsebnosti klorofila v višjih rastlinah, saj je občutljiv na spremembe odbojnosti, ki jih povzroča absorpcija klorofila.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Sklic: Datt, B. »Daljinsko zaznavanje vsebnosti vode v listih evkaliptusa.« Journal of Plant Physiology 154, št. 1 (1999): 30–36._

***

## MNLI – Modificirani nelinearni indeks

Ta indeks je izboljšana različica nelinearnega indeksa (NLI), ki vključuje indeks vegetacije, prilagojen tleh (SAVI), da se upošteva ozadje tal. Program ENVI uporablja vrednost faktorja prilagoditve ozadja krošnje (_L_) 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Sklic: Yang, Z., P. Willis in R. Mueller. »Vpliv slike AWIFS z izboljšanim razmerjem pasov na natančnost klasifikacije pridelkov.« Zbornik simpozija Pecora 17 o daljinskem zaznavanju (2008), Denver, CO._

***

## MSAVI2 – Modificirani indeks vegetacije, prilagojen tleh 2

Ta indeks je poenostavljena različica indeksa MSAVI, ki so ga predlagali Qi in sod. (1994), ki izboljšuje indeks vegetacije, prilagojen tleh (SAVI). Zmanjšuje šum tal in povečuje dinamični razpon signala vegetacije. MSAVI2 temelji na induktivni metodi, ki ne uporablja konstantne vrednosti _L_ (kot pri SAVI) za poudarjanje zdrave vegetacije.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Sklic: Qi, J., A. Chehbouni, A. Huete, Y. Kerr in S. Sorooshian. »A Modified Soil Adjusted Vegetation Index.« Remote Sensing of Environment 48 (1994): 119–126._

***

## MSR – Modificirano preprosto razmerje

Ta indeks je modifikacija preprostega razmerja NIR/Red, zasnovana za linearizacijo njegovega odnosa z biofizikalnimi parametri, in je pri višjih gostotah vegetacije občutljivejši od NDVI.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Vir: Chen, J. »Ocena indeksov vegetacije in modificiranega preprostega razmerja za uporabo v borealnih območjih.« Canadian Journal of Remote Sensing 22 (1996): 229–242._

***

## NDRE – Normalizirana razlika RedEdge

Ta indeks je podoben NDVI, vendar primerja kontrast med NIR in RedEdge namesto z Red, kar pogosto omogoča zgodnejše zaznavanje stresa rastlinja.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – Normalizirani razlikovni indeks vegetacije

Ta indeks je merilo zdrave, zelene vegetacije. Kombinacija njegove normalizirane razlike in uporabe območij klorofila z najvišjo absorpcijo in odbojnostjo zagotavlja njegovo zanesljivost v širokem spektru pogojev. Vendar pa se lahko v pogojih goste vegetacije, ko je LAI visok, pride do zasičenja.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Vrednost tega indeksa se giblje od -1 do 1. Običajno območje za zeleno vegetacijo je od 0,2 do 0,8.

_Vir: Rouse, J., R. Haas, J. Schell in D. Deering. Spremljanje vegetacijskih sistemov na Veliki ravnini z ERTS. Tretji simpozij ERTS, NASA (1973): 309–317._

***

## NLI – nelinearni indeks

Ta indeks temelji na predpostavki, da je odnos med številnimi vegetacijskimi indeksi in biofizikalnimi parametri površine nelinearen. Linearizira odnose s površinskimi parametri, ki so ponavadi nelinearni.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Sklic: Goel, N., in W. Qin. »Vplivi arhitekture krošnj na odnose med različnimi vegetacijskimi indeksi ter LAI in Fpar: računalniška simulacija.« Remote Sensing Reviews 10 (1994): 309–347._

***

## OSAVI – optimizirani indeks vegetacije, prilagojen tleh

Ta indeks temelji na indeksu vegetacije, prilagojenem tleh (SAVI). Za faktor prilagoditve ozadja krošnjega pokrova uporablja standardno vrednost 0,16. Rondeaux (1996) je ugotovil, da ta vrednost pri nizki vegetacijski pokritosti omogoča večjo variabilnost tal kot indeks SAVI, hkrati pa kaže večjo občutljivost pri vegetacijski pokritosti, večji od 50 %. Ta indeks se najbolje uporablja na območjih z relativno redko vegetacijo, kjer je tla vidna skozi krošnjo.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Sklic: Rondeaux, G., M. Steven in F. Baret. »Optimization of Soil-Adjusted Vegetation Indices.« Remote Sensing of Environment 55 (1996): 95–107._

***

## RDVI – renormalizirani indeks razlike v vegetaciji

Ta indeks uporablja razliko med valovnimi dolžinami v bližnji infrardeči in rdeči svetlobi, skupaj z indeksom NDVI, za poudarjanje zdrave vegetacije. Ni občutljiv na vplive tal in geometrije opazovanja sonca.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Sklic: Roujean, J., in F. Breon. »Ocena PAR, ki ga absorbira vegetacija, na podlagi meritev dvosmerne odbojnosti.« Remote Sensing of Environment 51 (1995): 375–384._

***

## SAVI – indeks vegetacije, prilagojen tleh

Ta indeks je podoben indeksu NDVI, vendar zmanjšuje vpliv pikslov tal. Uporablja faktor prilagoditve ozadja krošnje, _L_, ki je odvisen od gostote vegetacije in pogosto zahteva predhodno poznavanje količine vegetacije. Huete (1988) predlaga optimalno vrednost _L_=0,5 za upoštevanje variacij talnega ozadja prvega reda. Ta indeks se najbolje uporablja na območjih z relativno redko vegetacijo, kjer je tla vidna skozi krošnjo.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Sklic: Huete, A. »A Soil-Adjusted Vegetation Index (SAVI).« Remote Sensing of Environment 25 (1988): 295–309._

***

## TDVI – transformirani indeks razlike vegetacije

Ta indeks je uporaben za spremljanje vegetacijskega pokrova v urbanih okoljih. Ne pride do zasičenja, kot pri indeksih NDVI in SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Sklic: Bannari, A., H. Asalhi in P. Teillet. »Transformirani indeks razlike vegetacije (TDVI) za kartiranje vegetacijskega pokrova« V: Zbornik simpozija o geoznanosti in daljinskem zaznavanju, IGARSS '02, IEEE International, zvezek 5 (2002)._

***

## VARI – indeks za vidno območje, odporen na atmosferske vplive

Ta indeks temelji na indeksu ARVI in se uporablja za oceno deleža vegetacije v prizoru z nizko občutljivostjo na atmosferske vplive.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Sklic: Gitelson, A., et al. »Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction.« International Journal of Remote Sensing 23 (2002): 2537–2562._

***

## WDRVI – indeks vegetacije s širokim dinamičnim razponom

Ta indeks je podoben indeksu NDVI, vendar uporablja utežni koeficient (_a_) za zmanjšanje razlike med prispevki signalov v bližnji infrardeči in rdeči svetlobi k indeksu NDVI. Indeks WDRVI je še posebej učinkovit v prizorih z zmerno do visoko gostoto vegetacije, ko indeks NDVI presega 0,6. NDVI se običajno izravna, ko se delež vegetacije in indeks listne površine (LAI) povečata, medtem ko je WDRVI občutljivejši na širše razpon deležev vegetacije in na spremembe LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Tehtalni koeficient (_a_) lahko znaša od 0,1 do 0,2. Henebry, Viña in Gitelson (2004) priporočajo vrednost 0,2.

_Viri_

_Gitelson, A. »Indeks vegetacije s širokim dinamičnim razponom za daljinsko kvantifikacijo biofizikalnih značilnosti vegetacije.« Journal of Plant Physiology 161, št. 2 (2004): 165–173._

_Henebry, G., A. Viña in A. Gitelson. »Indeks vegetacije s širokim dinamičnim razponom in njegova potencialna uporabnost za analizo vrzeli.« Gap Analysis Bulletin 12: 50–56._

---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Formule za multispektralni indeks

Spodnje formule za indeks uporabljajo kombinacijo povprečnih območij prepustnosti filtra Survey3:

<table><thead><tr><th align="center">Survey3 Barva filtra</th><th width="196.199951171875" align="center">Survey3 Ime filtra</th><th width="159.800048828125" align="center">Območje prepustnosti (FWHM)</th><th align="center">Povprečna prepustnost</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>Pri uporabi teh formul se ime lahko konča z „\_1“ ali „\_2“, kar ustreza filtru NIR, pri čemer je bil uporabljen filter NIR1 ali NIR2.

***

## EVI – izboljšani indeks vegetacije

Ta indeks je bil prvotno razvit za uporabo s podatki MODIS kot izboljšava v primerjavi z NDVI z optimizacijo signala vegetacije na območjih z visokim indeksom listne površine (LAI). Najbolj uporaben je v regijah z visokim LAI, kjer se NDVI lahko zasiči. Uporablja modro odbojno območje za popravek signalov talnega ozadja in zmanjšanje vplivov atmosfere, vključno z razprševanjem aerosolov.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Vrednosti EVI morajo za vegetacijske piksle znašati od 0 do 1. Svetle značilnosti, kot so oblaki in bele stavbe, skupaj s temnimi značilnostmi, kot je voda, lahko povzročijo anomalne vrednosti pikslov v sliki EVI. Pred ustvarjanjem slike EVI morate iz slike odbojnosti zamaskirati oblake in svetle elemente ter po želji nastaviti prag vrednosti pikslov od 0 do 1.

_Sklic: Huete, A., et al. »Pregled radiometrične in biofizikalne učinkovitosti indeksov vegetacije MODIS.« Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 – Indeks gozdnega pokrova 1

Ta indeks ločuje gozdne krošnje od drugih vrst vegetacije z uporabo multispektralnih odbojnih slik, ki vključujejo rdeči robni pas.

$$
FCI1 = Red * RedEdge
$$

Gozdna območja bodo imela nižje vrednosti FCI1 zaradi nižje odbojnosti dreves in prisotnosti senc v krošnjah.

_Sklic: Becker, Sarah J., Craig S.T. Daughtry in Andrew L. Russ. „Robust forest cover indices for multispectral images.“ Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## FCI2 – Indeks gozdnega pokrova 2

Ta indeks ločuje krošnje gozda od drugih vrst vegetacije z uporabo multispektralnih odbojnih slik, ki ne vključujejo rdečega roba.

$$
FCI2 = Red * NIR
$$

Gozdna območja bodo imela nižje vrednosti FCI2 zaradi nižjega odboja dreves in prisotnosti senc v krošnjah.

_Sklic: Becker, Sarah J., Craig S.T. Daughtry in Andrew L. Russ. „Robust forest cover indices for multispectral images.“ Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## GEMI – Indeks globalnega spremljanja okolja

Ta nelinearen indeks vegetacije se uporablja za globalno spremljanje okolja iz satelitskih posnetkov in poskuša popraviti vplive atmosfere. Je podoben NDVI, vendar je manj občutljiv na vplive atmosfere. Nanj vpliva gola tla, zato se ne priporoča za uporabo na območjih z redko ali zmerno gosto vegetacijo.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Kjer:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Sklic: Pinty, B., in M. Verstraete. GEMI: nelinearni indeks za spremljanje globalne vegetacije iz satelitov. Vegetation 101 (1992): 15-20._

***

## GARI - Green Indeks odpornosti na atmosferske vplive

Ta indeks je bolj občutljiv na širok razpon koncentracij klorofila in manj občutljiv na atmosferske vplive kot NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Gama konstanta je tehtalna funkcija, ki je odvisna od aerosolnih pogojev v atmosferi. ENVI uporablja vrednost 1,7, ki je priporočena vrednost iz Gitelson, Kaufman in Merzylak (1996, stran 296).

_Sklic: Gitelson, A., Y. Kaufman in M. Merzylak. »Uporaba kanala Green pri daljinskem zaznavanju globalne vegetacije iz EOS-MODIS.« Daljinsko zaznavanje okolja 58 (1996): 289–298._

***

## GCI – Green Indeks klorofila

Ta indeks se uporablja za oceno vsebnosti klorofila v listih pri širokem spektru rastlinskih vrst.

$$
GCI = {NIR \over Green} - 1
$$

Široki NIR in zelene valovne dolžine omogočajo boljšo napoved vsebnosti klorofila, hkrati pa zagotavljajo večjo občutljivost in višje razmerje med signalom in šumom.

_Sklic: Gitelson, A., Y. Gritz in M. Merzlyak. »Odnosi med vsebnostjo klorofila v listih in spektralno odbojnostjo ter algoritmi za neporušno ocenjevanje klorofila v listih višjih rastlin.« Journal of Plant Physiology 160 (2003): 271–282._

***

## GLI – Green Indeks listov

Ta indeks je bil prvotno zasnovan za uporabo z digitalno kamero RGB za merjenje pokritosti pšenice, kjer rdeče, zelene in modre digitalne številke (DN) segajo od 0 do 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Vrednosti GLI se gibljejo od -1 do +1. Negativne vrednosti predstavljajo tla in nežive značilnosti, pozitivne vrednosti pa zelene liste in stebla.

_Sklic: Louhaichi, M., M. Borman in D. Johnson. „Prostorsko locirana platforma in letalska fotografija za dokumentiranje vplivov paše na pšenico.“ Geocarto International 16, št. 1 (2001): 65–70._

***

## GNDVI – Green Normalizirani indeks razlike v vegetaciji

Ta indeks je podoben NDVI, razen da meri zeleno spektro od 540 do 570 nm namesto rdeče spektre. Ta indeks je bolj občutljiv na koncentracijo klorofila kot NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Sklic: Gitelson, A., in M. Merzlyak. „Daljinsko zaznavanje koncentracije klorofila v listih višjih rastlin.“ Advances in Space Research 22 (1998): 689–692._

***

## GOSAVI – Green Optimizirani indeks vegetacije, prilagojen tleh

Ta indeks je bil prvotno zasnovan z barvno-infrardečo fotografijo za napovedovanje potreb po dušiku za koruzo. Je podoben OSAVI, vendar zamenjuje zeleni pas z rdečim.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Sklic: Sripada, R., et al. »Določanje potreb po dušiku za koruzo med sezono z uporabo barvne infrardeče fotografije iz zraka.« Doktorska disertacija, North Carolina State University, 2005._

***

## GRVI – Green Razmerje indeksa vegetacije

Ta indeks je občutljiv na fotosintetsko aktivnost v krošnjah gozda, saj na zeleno in rdečo odbojnost močno vplivajo spremembe v barvah listnih pigmentov.

$$
GRVI = {NIR \over Green }
$$

_Sklic: Sripada, R., et al. »Letalska barvna infrardeča fotografija za določanje potreb po dušiku v zgodnji sezoni pri koruzi.« Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green Indeks vegetacije, prilagojen tleh

Ta indeks je bil prvotno zasnovan z barvno infrardečo fotografijo za napovedovanje potreb po dušiku za koruzo. Je podoben SAVI, vendar zamenjuje zeleni pas z rdečim.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Sklic: Sripada, R., et al. »Določanje potreb po dušiku za koruzo med sezono z uporabo barvne infrardeče fotografije iz zraka.« Doktorska disertacija, North Carolina State University, 2005._

***

## LAI – Indeks listne površine

Ta indeks se uporablja za oceno pokritosti z listjem in napovedovanje rasti in donosa pridelka. ENVI izračuna zeleni LAI z naslednjo empirično formulo iz Boegh et al (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Kjer je EVI:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Visoke vrednosti LAI se običajno gibljejo od približno 0 do 3,5. Vendar, če prizor vsebuje oblake in druge svetle elemente, ki ustvarjajo nasičene piksle, lahko vrednosti LAI presegajo 3,5. Pred ustvarjanjem slike LAI je najbolje zamaskirati oblake in svetle elemente iz scene.

_Sklic: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde in A. Thomsen. „Zračni multispektralni podatki za količinsko opredelitev indeksa listne površine, koncentracije dušika in fotosintetske učinkovitosti v kmetijstvu.“ Remote Sensing of Environment 81, št. 2-3 (2002): 179-193._

***

## LCI – Indeks listnega klorofila

Ta indeks se uporablja za oceno vsebnosti klorofila v višjih rastlinah, ki so občutljive na spremembe v odbojnosti, ki jih povzroča absorpcija klorofila.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Sklic: Datt, B. „Daljinsko zaznavanje vsebnosti vode v listih evkaliptusa.“ Journal of Plant Physiology 154, št. 1 (1999): 30–36._

***

## MNLI – modificirani nelinearen indeks

Ta indeks je izboljšava nelinearnega indeksa (NLI), ki vključuje indeks vegetacije, prilagojen tleh (SAVI), da upošteva ozadje tal. ENVI uporablja vrednost faktorja prilagoditve ozadja krošnje (_L_) 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Sklic: Yang, Z., P. Willis in R. Mueller. »Vpliv slike AWIFS z izboljšanim razmerjem pasov na natančnost klasifikacije pridelkov.« Zbornik simpozija Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 – Modificirani indeks vegetacije, prilagojen tleh 2

Ta indeks je enostavnejša različica indeksa MSAVI, ki so ga predlagali Qi et al (1994) in ki izboljšuje indeks vegetacije, prilagojen tleh (SAVI). Zmanjšuje šum tal in povečuje dinamični razpon signala vegetacije. MSAVI2 temelji na induktivni metodi, ki ne uporablja konstantne vrednosti _L_ (kot pri SAVI) za poudarjanje zdrave vegetacije.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Sklic: Qi, J., A. Chehbouni, A. Huete, Y. Kerr in S. Sorooshian. „A Modified Soil Adjusted Vegetation Index.“ Remote Sensing of Environment 48 (1994): 119–126._

***

## NDRE – normalizirana razlika RedEdge

Ta indeks je podoben NDVI, vendar primerja kontrast med NIR in RedEdge namesto Red, ki pogosto prej zazna stres vegetacije.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – normalizirani indeks razlike v vegetaciji

Ta indeks je merilo zdrave, zelene vegetacije. Kombinacija njegove normalizirane razlike in uporabe območij najvišje absorpcije in odbojnosti klorofila ga naredi robustnega v širokem spektru pogojev. Vendar pa se lahko nasiči v pogojih goste vegetacije, ko LAI postane visok.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Vrednost tega indeksa je od -1 do 1. Običajno območje za zeleno vegetacijo je od 0,2 do 0,8.

_Sklic: Rouse, J., R. Haas, J. Schell in D. Deering. Spremljanje vegetacijskih sistemov v Veliki ravnini z ERTS. Tretji simpozij ERTS, NASA (1973): 309–317._

***

## NLI – nelinearni indeks

Ta indeks predpostavlja, da je razmerje med mnogimi vegetacijskimi indeksi in biofizikalnimi parametri površine nelinearno. Linearira razmerja s parametri površine, ki so ponavadi nelinearni.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Sklic: Goel, N. in W. Qin. »Vplivi arhitekture krošnje na razmerja med različnimi vegetacijskimi indeksi in LAI ter Fpar: računalniška simulacija.« Remote Sensing Reviews 10 (1994): 309–347._

***

## OSAVI – Optimizirani indeks vegetacije, prilagojen tleh

Ta indeks temelji na indeksu vegetacije, prilagojenem tleh (SAVI). Uporablja standardno vrednost 0,16 za faktor prilagoditve ozadja krošnje. Rondeaux (1996) je ugotovil, da ta vrednost zagotavlja večjo variacijo tal kot SAVI za nizko vegetacijsko pokritost, hkrati pa kaže večjo občutljivost za vegetacijsko pokritost, večjo od 50 %. Ta indeks se najbolje uporablja na območjih z relativno redko vegetacijo, kjer je tla vidna skozi krošnjo.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Sklic: Rondeaux, G., M. Steven in F. Baret. »Optimizacija indeksov vegetacije, prilagojenih tleh.« Remote Sensing of Environment 55 (1996): 95–107._

***

## RDVI – Renormalizirani indeks razlike vegetacije

Ta indeks uporablja razliko med bližnjo infrardečo in rdečo valovno dolžino, skupaj z NDVI, za poudarjanje zdrave vegetacije. Ni občutljiv na vplive tal in geometrije sončnega opazovanja.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Sklic: Roujean, J., in F. Breon. »Ocena PAR, ki ga absorbira vegetacija, iz meritev dvosmerne odbojnosti.« Remote Sensing of Environment 51 (1995): 375–384._

***

## SAVI – indeks vegetacije, prilagojen tleh

Ta indeks je podoben NDVI, vendar zatira učinke pikslov tal. Uporablja faktor prilagoditve ozadja krošnje, _L_, ki je funkcija gostote vegetacije in pogosto zahteva predhodno znanje o količini vegetacije. Huete (1988) predlaga optimalno vrednost _L_=0,5 za upoštevanje variacij ozadja tal prvega reda. Ta indeks se najbolje uporablja na območjih z relativno redko vegetacijo, kjer je tla vidna skozi krošnjo.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Sklic: Huete, A. »Indeks vegetacije, prilagojen tlom (SAVI).« Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI – transformirani razlikovni indeks vegetacije

Ta indeks je uporaben za spremljanje vegetacijskega pokrova v urbanem okolju. Ne nasiči se kot NDVI in SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Sklic: Bannari, A., H. Asalhi in P. Teillet. »Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping« V: Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, Volume 5 (2002)._

***

## VARI – indeks vidne atmosferske odpornosti

Ta indeks temelji na ARVI in se uporablja za oceno deleža vegetacije v prizoru z nizko občutljivostjo na atmosferske vplive.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Sklic: Gitelson, A., et al. „Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – Indeks vegetacije s širokim dinamičnim razponom

Ta indeks je podoben NDVI, vendar uporablja utežni koeficient (_a_), da zmanjša razliko med prispevki bližnje infrardečih in rdečih signalov k NDVI. WDRVI je še posebej učinkovit v prizorih z zmerno do visoko gostoto vegetacije, ko NDVI presega 0,6. NDVI se običajno izravna, ko se povečata delež vegetacije in indeks listne površine (LAI), medtem ko je WDRVI bolj občutljiv na širši razpon deležev vegetacije in spremembe v LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Težni koeficient (_a_) lahko znaša od 0,1 do 0,2. Henebry, Viña in Gitelson (2004).

_Sklici_

_Gitelson, A. »Indeks vegetacije s širokim dinamičnim razponom za daljinsko kvantifikacijo biofizikalnih značilnosti vegetacije.« Journal of Plant Physiology 161, št. 2 (2004): 165–173._

_Henebry, G., A. Viña in A. Gitelson. »Indeks vegetacije s širokim dinamičnim razponom in njegova potencialna uporabnost za analizo vrzeli.« Gap Analysis Bulletin 12: 50–56._

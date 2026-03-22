---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Formule za multispektralne indekse

Spodnje formule za indekse uporabljajo kombinacijo povprečnih območij prepustnosti filtra Survey3:

<table><thead><tr><th align="center">Survey3 Barva filtra</th><th width="196.199951171875" align="center">Survey3 Ime filtra</th><th width="159.800048828125" align="center">Območje prepustnosti (FWHM)</th><th align="center">Povprečna prepustnost</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB – Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN– Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN – Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>Pri uporabi teh formul se ime lahko konča z »\_1« ali »\_2«, kar ustreza temu, kateri filter NIR, bodisi NIR1 ali NIR2, je bil uporabljen.

***

## EVI – Izboljšani indeks vegetacije

Ta indeks je bil prvotno razvit za uporabo s podatki MODIS kot izboljšava indeksa NDVI z optimizacijo signala vegetacije na območjih z visokim indeksom listne površine (LAI). Najbolj uporaben je v regijah z visokim indeksom LAI, kjer se indeks NDVI lahko nasiči. Uporablja območje modre odbojnosti za popravek signalov talnega ozadja in zmanjšanje vplivov atmosfere, vključno z razprševanjem aerosolov.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Vrednosti EVI morajo za vegetacijske piksle znašati od 0 do 1. Svetle značilnosti, kot so oblaki in bele stavbe, skupaj s temnimi značilnostmi, kot je voda, lahko povzročijo anomalne vrednosti pikslov na sliki EVI. Pred ustvarjanjem slike EVI morate iz slike odbojnosti zamaskirati oblake in svetle elemente ter po želji določiti prag vrednosti pikslov od 0 do 1.

_Sklic: Huete, A., et al. »Pregled radiometrične in biofizikalne zmogljivosti indeksov vegetacije MODIS.« Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 – Indeks gozdnega pokrova 1

Ta indeks ločuje gozdne krošnje od drugih vrst vegetacije z uporabo multispektralnih slik odbojnosti, ki vključujejo pas rdečega roba.

$$
FCI1 = Red * RedEdge
$$

Gozdna območja bodo imela nižje vrednosti FCI1 zaradi nižje odbojnosti dreves in prisotnosti senc znotraj krošnje.

_Sklic: Becker, Sarah J., Craig S.T. Daughtry in Andrew L. Russ. »Robustni indeksi gozdnega pokrova za multispektralne slike.« Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## FCI2 – Indeks gozdnega pokrova 2

Ta indeks ločuje gozdno krošnjo od drugih vrst vegetacije z uporabo multispektralnih slik odbojnosti, ki ne vključujejo rdečega robnega pasu.

$$
FCI2 = Red * NIR
$$

Gozdna območja bodo imela nižje vrednosti FCI2 zaradi nižje odbojnosti dreves in prisotnosti senc v krošnji.

_Sklic: Becker, Sarah J., Craig S.T. Daughtry in Andrew L. Russ. »Robustni indeksi gozdnega pokrova za multispektralne slike.« Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## GEMI – Indeks globalnega spremljanja okolja

Ta nelinearni indeks vegetacije se uporablja za globalno spremljanje okolja na podlagi satelitskih posnetkov in poskuša popraviti vplive atmosfere. Je podoben indeksu NDVI, vendar je manj občutljiv na vplive atmosfere. Nanj vpliva gola tla; zato se ne priporoča za uporabo na območjih z redko ali zmerno gosto vegetacijo.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Kje:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Sklic: Pinty, B., in M. Verstraete. GEMI: nelinearni indeks za spremljanje globalne vegetacije iz satelitov. Vegetation 101 (1992): 15–20._

***

## GARI – Green Indeks, odporen na vplive atmosfere

Ta indeks je bolj občutljiv na širok razpon koncentracij klorofila in manj občutljiv na vplive atmosfere kot NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Konstanta gama je tehtalna funkcija, ki je odvisna od razmer aerosolov v atmosferi. ENVI uporablja vrednost 1,7, ki je priporočena vrednost iz Gitelson, Kaufman in Merzylak (1996, stran 296).

_Sklic: Gitelson, A., Y. Kaufman in M. Merzylak. »Uporaba kanala Green pri daljinskem zaznavanju globalne vegetacije iz EOS-MODIS.« Remote Sensing of Environment 58 (1996): 289–298._

***

## GCI – Green Indeks klorofila

Ta indeks se uporablja za oceno vsebnosti klorofila v listih pri širokem spektru rastlinskih vrst.

$$
GCI = {NIR \over Green} - 1
$$

Širok spekter valovnih dolžin NIR in zelenih valovnih dolžin omogoča boljšo napoved vsebnosti klorofila, hkrati pa zagotavlja večjo občutljivost in višje razmerje med signalom in šumom.

_Sklic: Gitelson, A., Y. Gritz in M. Merzlyak. »Odnosi med vsebnostjo klorofila v listih in spektralno odbojnostjo ter algoritmi za neporušno ocenjevanje klorofila v listih višjih rastlin.« Journal of Plant Physiology 160 (2003): 271–282._

***

## Indeks listov GLI – Green

Ta indeks je bil prvotno zasnovan za uporabo z digitalno kamero RGB za merjenje pokritosti s pšenico, pri čemer se rdeče, zelene in modre digitalne vrednosti (DN) gibljejo od 0 do 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Vrednosti GLI segajo od -1 do +1. Negativne vrednosti predstavljajo tla in nežive elemente, pozitivne vrednosti pa zelene liste in stebla.

_Sklic: Louhaichi, M., M. Borman in D. Johnson. »Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat.« Geocarto International 16, št. 1 (2001): 65–70._

***

## GNDVI – Green Normalizirani indeks razlike v vegetaciji

Ta indeks je podoben indeksu NDVI, razen da meri zeleni spekter od 540 do 570 nm namesto rdečega spektra. Ta indeks je bolj občutljiv na koncentracijo klorofila kot indeks NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Sklic: Gitelson, A., in M. Merzlyak. »Daljinsko zaznavanje koncentracije klorofila v listih višjih rastlin.« Advances in Space Research 22 (1998): 689–692._

***

## GOSAVI – Green Optimizirani indeks vegetacije, prilagojen tleh

Ta indeks je bil prvotno zasnovan z barvno-infrardečo fotografijo za napovedovanje potreb po dušiku za koruzo. Je podoben indeksu OSAVI, vendar zamenjuje zeleni pas z rdečim.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Sklic: Sripada, R., et al. »Določanje potreb po dušiku za koruzo med rastno sezono z uporabo barvno-infrardeče fotografije iz zraka.« Doktorska disertacija, North Carolina State University, 2005._

***

## GRVI – indeks vegetacije na podlagi razmerja Green

Ta indeks je občutljiv na stopnje fotosinteze v krošnjah gozda, saj na odbojnost zelenih in rdečih valovnih dolžin močno vplivajo spremembe v listnih pigmentih.

$$
GRVI = {NIR \over Green }
$$

_Sklic: Sripada, R., et al. »Letalska barvno-infrardeča fotografija za določanje potreb po dušiku v zgodnji sezoni pri koruzi.« Agronomy Journal 98 (2006): 968–977._

***

## GSAVI – Green Indeks vegetacije, prilagojen tleh

Ta indeks je bil prvotno zasnovan z barvno-infrardečo fotografijo za napovedovanje potreb po dušiku za koruzo. Je podoben indeksu SAVI, vendar zamenjuje zeleni pas z rdečim.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Sklic: Sripada, R., et al. »Določanje potreb po dušiku za koruzo med rastno sezono z uporabo barvne infrardeče fotografije iz zraka.« Doktorska disertacija, North Carolina State University, 2005._

***

## LAI – Indeks listne površine

Ta indeks se uporablja za oceno pokritosti z listjem ter za napovedovanje rasti in pridelka. ENVI izračuna zeleni LAI z uporabo naslednje empirične formule iz Boegh et al (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Kjer je EVI:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Visoke vrednosti LAI se običajno gibljejo od približno 0 do 3,5. Vendar pa lahko vrednosti LAI presegajo 3,5, če prizor vsebuje oblake in druge svetle elemente, ki povzročajo nasičene piksle. Pred ustvarjanjem slike LAI je idealno, da iz slike odstranite oblake in svetle elemente.

_Sklic: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde in A. Thomsen. »Zračni multispektralni podatki za količinsko opredelitev indeksa listne površine, koncentracije dušika in fotosintetske učinkovitosti v kmetijstvu.« Remote Sensing of Environment 81, št. 2–3 (2002): 179–193._

***

## LCI – Indeks klorofila v listih

Ta indeks se uporablja za oceno vsebnosti klorofila v višjih rastlinah, ki so občutljive na spremembe odbojnosti, povzročene z absorpcijo klorofila.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Sklic: Datt, B. »Daljinsko zaznavanje vsebnosti vode v listih evkaliptusa.« Journal of Plant Physiology 154, št. 1 (1999): 30–36._

***

## MNLI – Modificirani nelinearni indeks

Ta indeks je izboljšana različica nelinearnega indeksa (NLI), ki vključuje indeks vegetacije, prilagojen tleh (SAVI), da upošteva ozadje tal. ENVI uporablja vrednost faktorja prilagoditve ozadja krošnje (_L_) 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Sklic: Yang, Z., P. Willis in R. Mueller. »Vpliv slike AWIFS z izboljšanim razmerjem pasov na natančnost klasifikacije pridelkov.« Zbornik simpozija Pecora 17 o daljinskem zaznavanju (2008), Denver, CO._

***

## MSAVI2 – Modificirani indeks vegetacije, prilagojen tleh 2

Ta indeks je poenostavljena različica indeksa MSAVI, ki so ga predlagali Qi in sod. (1994) in ki izboljšuje indeks vegetacije, prilagojen tleh (SAVI). Zmanjšuje šum tal in povečuje dinamični razpon signala vegetacije. MSAVI2 temelji na induktivni metodi, ki ne uporablja konstantne vrednosti _L_ (kot pri SAVI) za poudarjanje zdrave vegetacije.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Sklic: Qi, J., A. Chehbouni, A. Huete, Y. Kerr in S. Sorooshian. »A Modified Soil Adjusted Vegetation Index.« Remote Sensing of Environment 48 (1994): 119–126._

***

## NDRE – Normalizirana razlika RedEdge

Ta indeks je podoben indeksu NDVI, vendar primerja kontrast med NIR in RedEdge namesto Red, kar pogosto omogoča zgodnejše zaznavanje stresa rastlinja.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – normalizirani indeks razlike vegetacije

Ta indeks je merilo zdrave, zelene vegetacije. Kombinacija njegove normalizirane razlike in uporabe območij klorofila z najvišjo absorpcijo in odbojnostjo ga naredi robustnega v širokem spektru pogojev. Vendar se lahko v pogojih goste vegetacije nasiči, ko LAI postane visok.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Vrednost tega indeksa se giblje od -1 do 1. Običajno območje za zeleno vegetacijo je od 0,2 do 0,8.

_Sklic: Rouse, J., R. Haas, J. Schell in D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. Third ERTS Symposium, NASA (1973): 309–317._

***

## NLI – nelinearni indeks

Ta indeks predpostavlja, da je razmerje med številnimi vegetacijskimi indeksi in biofizikalnimi parametri površine nelinearno. Linearizira razmerja s parametri površine, ki so ponavadi nelinearni.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Sklic: Goel, N. in W. Qin. »Vplivi arhitekture krošnje na odnose med različnimi vegetacijskimi indeksi ter LAI in Fpar: računalniška simulacija.« Remote Sensing Reviews 10 (1994): 309–347._

***

## OSAVI – Optimizirani indeks vegetacije, prilagojen tleh

Ta indeks temelji na indeksu vegetacije, prilagojenem tleh (SAVI). Uporablja standardno vrednost 0,16 za faktor prilagoditve ozadja krošnje. Rondeaux (1996) je ugotovil, da ta vrednost zagotavlja večjo variabilnost tal kot SAVI pri nizki vegetacijski pokritosti, hkrati pa kaže večjo občutljivost na vegetacijsko pokritost, večjo od 50 %. Ta indeks se najbolje uporablja na območjih z relativno redko vegetacijo, kjer je tla vidna skozi krošnjo.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Sklic: Rondeaux, G., M. Steven in F. Baret. »Optimization of Soil-Adjusted Vegetation Indices.« Remote Sensing of Environment 55 (1996): 95–107._

***

## RDVI – renormalizirani indeks razlike vegetacije

Ta indeks uporablja razliko med bližnjim infrardečim in rdečim valovnim območjem, skupaj z indeksom NDVI, za poudarjanje zdrave vegetacije. Ni občutljiv na vplive tal in geometrije opazovanja sonca.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Sklic: Roujean, J., in F. Breon. »Ocena PAR, ki ga absorbira vegetacija, na podlagi meritev dvosmerne odbojnosti.« Remote Sensing of Environment 51 (1995): 375–384._

***

## SAVI – Indeks vegetacije, prilagojen tleh

Ta indeks je podoben indeksu NDVI, vendar izloča vplive pikslov tal. Uporablja faktor prilagoditve ozadja krošnje, _L_, ki je funkcija gostote vegetacije in pogosto zahteva predhodno poznavanje količine vegetacije. Huete (1988) predlaga optimalno vrednost _L_=0,5 za upoštevanje variacij ozadja tal prvega reda. Ta indeks se najbolje uporablja na območjih z relativno redko vegetacijo, kjer je tla vidna skozi krošnjo.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Sklic: Huete, A. &quot;A Soil-Adjusted Vegetation Index (SAVI).&quot; Remote Sensing of Environment 25 (1988): 295–309._

***

## TDVI – transformirani indeks razlike vegetacije

Ta indeks je uporaben za spremljanje vegetacijskega pokrova v urbanih okoljih. Ne zasiči se kot NDVI in SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Sklic: Bannari, A., H. Asalhi in P. Teillet. »Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping« V: Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, zvezek 5 (2002)._

***

## VARI – Indeks vidne atmosferske odpornosti

Ta indeks temelji na indeksu ARVI in se uporablja za oceno deleža vegetacije v prizoru z nizko občutljivostjo na atmosferske vplive.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Sklic: Gitelson, A., et al. »Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction.« International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – Indeks vegetacije s širokim dinamičnim razponom

Ta indeks je podoben indeksu NDVI, vendar uporablja utežni koeficient (_a_) za zmanjšanje razlike med prispevki signalov v bližnji infrardeči in rdeči svetlobi k indeksu NDVI. WDRVI je še posebej učinkovit v prizorih z zmerno do visoko gostoto vegetacije, ko NDVI presega 0,6. NDVI se običajno izravna, ko se delež vegetacije in indeks listne površine (LAI) povečata, medtem ko je WDRVI bolj občutljiv na širši razpon deležev vegetacije in na spremembe v LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Tehtalni koeficient (_a_) lahko znaša od 0,1 do 0,2. Henebry, Viña in Gitelson (2004) priporočajo vrednost 0,2.

_Viri_

_Gitelson, A. »Indeks vegetacije s širokim dinamičnim razponom za daljinsko kvantifikacijo biofizikalnih značilnosti vegetacije.« Journal of Plant Physiology 161, št. 2 (2004): 165–173._

_Henebry, G., A. Viña in A. Gitelson. »Indeks vegetacije s širokim dinamičnim razponom in njegova potencialna uporabnost za analizo vrzeli.« Gap Analysis Bulletin 12: 50–56._

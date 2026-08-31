# Kamere LATTICE

LATTICE je modularni multispektralni kamerni sistem podjetja MAPIR za snemanje v kmetijstvu in znanosti. Vsaka kamera LATTICE temelji na senzorju Sony IMX265 z globalnim zaklopom (**3,1 MP, 3,45 µm pikslov**) in se prek omrežja Ethernet poveže kot naprava**GigE Vision**.

Chloros 1.2.0 v realnem času upravlja s kamerami LATTICE – odkrivanje, predogled v živo, zajem in sinhronizirane večkamere – s treh vmesnikov:

| Vmesnik    | Kje                                                          | Platforme                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| Grafični uporabniški vmesnik        | Zavihek **Kamere** v stranskem meniju Chloros                         | Windows 10/11 x64                                        |
| CLI        | Družina ukazov `chloros-cli lattice`                           | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **Iščete strojno opremo?**Kamerni moduli, objektivi, filtri in pasovi, okvirji in pritrditve, kabli, PoE in ožičenje sprožilcev so opisani v [**uporabniškem priročniku LATTICE**](https://mapir.gitbook.io/lattice-camera). To poglavje obravnava upravljanje kamer iz Chloros.

Posnetki LATTICE so standardne datoteke `.tif`/`.tiff`, Chloros pa jih vedno obdeluje, začenši z neobdelanim posnetkom. Glejte [Referenco za CLI](../reference/cli-reference.md) in [Referenco za SDK](../reference/sdk-reference.md) za celoten ukaz in površino API.

## Dve konfiguraciji senzorjev

| Konfiguracija | Senzor       | Filter                                | Kaj prinaša ena kamera                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Bayerjeva barvna | trojni pasovni filter                |**Trije kalibrirani pasovi iz ene same osvetlitve**                 |
| **M3M**| Monokromatska   | enojni ozek pasovni interferenčni filter |**En kalibriran pas**; za indekse združite več kamer M3M |

Ker je kamera M3M monokromatska in ima en sam filter, vsak pas dobi svojo ekspozicijo. Kamera M3C pokriva vse tri svoje pasove z eno ekspozicijo senzorja.

## Nizi modelov in poimenovanje

Vsaka kamera shrani svojo identiteto v GenICam `DeviceUserID` kot niz modela:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros ga prikaže s predpono `LATT-` (na primer `LATT-M3M-L87-F450`). Isti niz `LATT-…` se zapiše v oznako EXIF `Model` vsakega izvoza in se uporablja kot ime izhodne mape kamere v obdelanih projektih.

| Komponenta | Vrednosti                                                   | Pomen                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Senzor    | `M3C` / `M3M`                                            | Bayer barvni / črno-beli                                                                          |
| Objektiv    | `L41` / `L87`                                            | Številka označuje **horizontalno vidno polje v stopinjah**: L41 = ozek (41°), L87 = širok (87°)    |
| Filter    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) ali `F<nm>` (M3M) | Glej [Filtri in spektralni pasovi](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

Niz modela določa vse nadaljnje korake: Chloros določi profil senzorja, razpored pasov in tovarniško kalibracijo na podlagi `DeviceUserID` + `DeviceSerialNumber`. Za posamezno kamero ni treba nič konfigurirati — glej [Povezovanje kamer](connecting.md).

## Filtri in pasovi

Središča pasov, robovi FWHM in celoten katalog 23-SKU M3M so specifikacije izdelka, zato so navedeni v priročniku za strojno opremo: [**Filtri in spektralni pasovi**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Kar je pomembno na strani programske opreme: koda filtra v nizu modela določa, katere izdelke Chloros lahko sestavi. Kamere s filtri RGB (`FRGB`) oddajajo le izločene barvne podatke in predogledne izdelke – sevanje in odbojnost po posameznih pasovih nista pomembna za širokopasovni senzor, zato jih Chloros preskoči in to tudi navede. Vsak drug filter prinaša celotno verigo sevanja → odbojnosti → indeksa.

## Radiometrična kalibracija na kratko

Vsaka kamera LATTICE je v tovarni posamično kalibrirana v skladu z verigo, sledljivo do NIST, in se dobavlja s certifikatom za posamezno kamero. Kaj to zajema, kako se meri in kakšno natančnost lahko navajate, je opisano v priročniku za strojno opremo: [**Tovarniška radiometrična kalibracija**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

Na strani programske opreme je pomembno, da Chloros določi pravilno kalibracijo ob priključitvi kamere in fiksira uporabljene koeficiente v vsak izvoz – glejte [Priključevanje kamer](connecting.md).

## V tem poglavju

* [Povezovanje kamer](connecting.md) — samodejno odkrivanje, pogovorno okno za povezavo v grafičnem vmesniku, ekvivalenti CLI/SDK ter kako se ob priključitvi kamere določi tovarniška kalibracija (paket na kameri ali v oblaku).

Druge teme v zvezi s sistemom LATTICE — nastavitve kamer in nadzor v realnem času, načini zajemanja, konfiguracije z več kamerami ter mono (M3M) obdelava in indeksi — so obravnavane v ločenih poglavjih tega priročnika, celoten nabor ukazov pa je na voljo v [CLI Referenca](../reference/cli-reference.md) in [SDK Reference](../reference/sdk-reference.md).

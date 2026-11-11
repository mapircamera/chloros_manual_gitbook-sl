# Kamere LATTICE

LATTICE je modularni multispektralni sistem kamer podjetjMAPIR za kmetijsko in znanstveno slikanje. Vsaka kamera LATTICE temelji na senzorju Sony IMX265 z globalnim zaklopom (**3,1 MP, 3,45 µm pikslov**) in se prek omrežja Ethernet poveže kot naprava**GigE Vision**.

Chloros 1.3.0 v živo upravlja kamere LATTICE – odkrivanje, predogled v živo, zajem in sinhronizirane nize več kamer – s treh vmesnikov:

| Vmesnik    | Kje                                                          | Platforme                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| GUI        | Zavihek **Kamere** v stranskem meniju Chloros                         | Windows 10/11 x64                                        |
| CLI        | Družina ukazov `chloros-cli lattice`                           | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **Iščete strojno opremo?**Kamerni moduli, objektivi, filtri in pasovi, okvirji in pritrditve, kabli, PoE in ožičenje sprožilcev so opisani v [**uporabniškem priročniku LATTICE**](https://mapir.gitbook.io/lattice-camera). To poglavje obravnava upravljanje kamer iz Chlorosa.

Posnetki LATTICE so standardne datoteke `.tif`/`.tiff`, Chloros pa jih vedno obdeluje na podlagi surovih posnetkov. Za celoten nabor ukazov in vmesnik API si oglejte [Referenco CLI](../reference/cli-reference.md) in [Referenco SDK](../reference/sdk-reference.md).

## Dve konfiguraciji senzorjev

| Konfiguracija | Senzor       | Filter                                | Kaj prikaže ena kamera                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Bayer barvna | trojni pasovni filter                |**Trije kalibrirani pasovi iz ene same osvetlitve**                 |
| **M3M**| Monokromatska   | enojni ozkopasovni interferenčni filter |**En kalibriran pas**; za indekse združite več kamer M3M |

Ker je kamera M3M monokromatska in ima en sam filter, vsak pas dobi svojo ekspozicijo. Kamera M3C pokrije vse tri pasove z eno osvetlitvijo senzorja.

## Nizi modelov in poimenovanje

Vsaka kamera shrani svojo identiteto v GenICam `DeviceUserID` kot niz modela:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros ga prikaže s predpono `LATT-` (na primer `LATT-M3M-L87-F450`). Isti niz `LATT-…` se zapiše v oznako EXIF `Model` vsakega izvoza in se uporablja kot ime izhodne mape kamere v obdelanih projektih.

| Komponenta | Vrednosti                                                   | Pomen                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Senzor    | `M3C` / `M3M`                                            | Bayer barvni / črno-beli                                                                          |
| Objektiv      | `L41` / `L87`                                            | Številka predstavlja **horizontalno vidno polje v stopinjah**: L41 = ozek (41°), L87 = širok (87°)    |
| Filter    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) ali `F<nm>` (M3M) | Glej [Filtri in spektralni pasovi](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

Niz modela določa vse nadaljnje postopke: Chloros iz `DeviceUserID` + `DeviceSerialNumber` določi profil senzorja, razpored pasov in tovarniško kalibracijo. Za posamezno kamero ni treba ničesar konfigurirati — glej [Povezovanje kamer](connecting.md).

## Filtri in spektralni pasovi

Središča pasov, robovi FWHM in celoten katalog 23-SKU M3M so specifikacije izdelka, zato so navedeni v priročniku za strojno opremo: [**Filtri in spektralni pasovi**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Kar je pomembno na strani programske opreme: koda filtra v nizu modela določa, katere izdelke lahko Chloros ustvari. Kamere s RGB-filtri (`FRGB`) oddajajo le izdelke brez Bayesovega filtra in predoglede – sevanje in odbojnost po posameznih pasovih nista pomembna za širokopasovni senzor, zato jih Chloros preskoči in to tudi navede. Vsak drug filter prinaša celotno verigo sevanje → odbojnost → indeks.

## Radiometrična kalibracija na kratko

Vsaka kamera LATTICE je v tovarni posamično kalibrirana v skladu z verigo, sledljivo do NIST, in se dobavlja s certifikatom za vsako kamero posebej. Kaj to zajema, kako se meri in kakšno natančnost lahko navajate, je opisano v priročniku za strojno opremo: [**Tovarniška radiometrična kalibracija**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

Na strani programske opreme je pomembno, da Chloros ob priključitvi kamere določi pravilno kalibracijo in uporabljene koeficiente fiksira v vsak izvoz – glejte [Priključevanje kamer](connecting.md).

## V tem poglavju

* [Povezovanje kamer](connecting.md) — samodejno odkrivanje, pogovorno okno za povezavo v grafičnem vmesniku, ustrezniki v CLI in SDK ter kako se ob povezavi kamere določi tovarniška kalibracija (paket na kameri ali v oblaku).
* [Nastavitve kamer](camera-settings.md) — vsi elementi v zavihku »Kamere«: seznam kamer, okno z nastavitvami za posamezno kamero in okno z nastavitvami niza (vključno z urejevalnikom ROI in poravnavo).
* [Nastavitve in načini zajemanja](capture.md) — zajem vseh posnetkov, posamezni / neprekinjeni / intervalni zajem, vrste izvoza in struktura mape z zajetimi posnetki.
* [Nizi več kamer](arrays.md) — čarovnik za povezovanje niza, sonda sinhronizacijskega kabla, pasovna širina, PTP in pogled na niz v živo.
* [Mono kamere in indeksi vegetacije](mono-indices.md) — kamere M3M ter delovni tok poravnava → združevanje → indeksiranje.

Celoten nabor ukazov je na voljo v [Referenca CLI](../reference/cli-reference.md) in [Referenca SDK](../reference/sdk-reference.md).

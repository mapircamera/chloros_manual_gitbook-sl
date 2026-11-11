---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/supported-cameras
---

# Podprte kamere:

Chloros obdeluje posnetke iz dveh družin kamer MAPIR na **vseh platformah** (Windows, Linux amd64 in Linux arm64/Jetson):

* **Survey3** — kamere Survey3W (širokokotne) in Survey3N (ozkokotne). Vhod: `RAW+JPG`.
* **LATTICE**— multispektralni kamerni moduli M3C in M3M. Vnos: posnetki `.tif`/`.tiff`. Kamere LATTICE je mogoče**v živo upravljati** tudi iz programa Chloros — prek zavihka »Kamere« v grafičnem uporabniškem vmesniku (Windows) ali prek `chloros-cli lattice` / Python ovega SDK-ja (Windows in Linux) — vključno s sinhroniziranimi nizi več kamer. Glejte [priročnik za LATTICE](lattice/).

Obdelovalni proces sprejema tudi vhodne datoteke `.dng`.

## Survey3

<table data-header-hidden><thead><tr><th width="156">Proizvajalec</th><th width="250">Model kamere</th><th width="138">Model filtra</th><th width="187">Vrsta slike</th></tr></thead><tbody><tr><td><strong>Proizvajalec</strong></td><td><strong>Model kamere</strong></td><td><strong>Model filtra</strong></td><td><strong>Vrsta slike</strong></td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>OCN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RE</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NIR</td><td>RAW+JPG, JPG</td></tr></tbody></table>## LATTICE

Serija LATTICE je modularni multispektralni sistem kamer, zasnovan na senzorju Sony IMX265 z globalnim zaklopom (3,1 MP, pikslov 3,45 µm). Vsaka kamera shranjuje svojo identiteto kot niz znakov modela:

```
<sensor>-<lens>-F<filter>        e.g.  M3C-L41-FRGN,  M3M-L87-F550
```

Chloros jo prikaže s predpono `LATT-` (na primer `LATT-M3M-L41-F550`), niz modela pa določa vse nadaljnje postopke – profil senzorja, razpored pasov in kalibracija se določijo samodejno; za posamezno kamero ni treba ničesar nastavljati. Številka objektiva predstavlja **horizontalno vidno polje v stopinjah**: `L41` = ozek 41°, `L87` = širok 87°.

Obstajata dve konfiguraciji senzorjev:

| Konfiguracija | Senzor      | Vrsta filtra                           | Pasovi na kamero                                                        |
| ------------- | ----------- | ------------------------------------- | ----------------------------------------------------------------------- |
| **M3C**       | Bayer barvni | Trojni pasovni filter                       | 3 spektralni pasovi iz ene same osvetlitve                                 |
| **M3M**       | Monokromatski  | Enotni ozek pasovni interferenčni filter | 1 kalibriran pas — združite več kamer M3M za indekse vegetacije |

### Možnosti filtrov M3C (Bayer)

| Filter | Pasovi (ime @ središče nm / FWHM nm)       |
| ------ | ---------------------------------------- |
| `FRGB` | Blue 475/30 · Green 550/30 · Red 625/30  |
| `FRGN` | Red 660/21 · Green 550/30 · NIR 850/30   |
| `FOCN` | Orange 618,4/46 · Cyan 494,1/40 · NIR 821,6/54 (izmerjeno — glej spodaj) |
| `FNGB` | Blue 475/30 · Green 550/30 · NIR 850/30  |

Za `FOCN` so vrednosti **izmerjeni** pas polovične maksimalne vrednosti filtra (Chloros 1.3.0 in novejše različice; starejše različice so uporabljale vrednosti iz tehničnih podatkov 615/21 · 490/38 · 808/14). Imena pasov ostajajo `Orange_615`, `Cyan_490` in `NIR_808`, izmerjeni središčni valovni dolžini in širini pa sta tisti, ki ju Chloros vtisne v vsak kalibriran izvoz.

### Katalog filtrov M3M (mono) — 23 artiklov

Številka F je oznaka artikla; izmerjen pas (vtisnjen v vsak kalibriran izvoz) je sken filtra za posamezno serijo:

| Art. št.    | Središče (nm, izmerjeno) | Robovi FWHM (nm) | Širina (nm) |
| ------ | --------------------- | --------------- | ---------- |
| F385   | 379,4                 | 367–392         | 25         |
| F405   | 403,9                 | 390–417         | 27         |
| F450   | 443,7                 | 430–458         | 28         |
| F485   | 489,7                 | 478–502         | 24         |
| F520   | 519,9                 | 504–536         | 32         |
| F550   | 548,4                 | 531–566         | 35         |
| F590   | 589,0                 | 570–608         | 38         |
| F615   | 623,8                 | 614–634         | 20         |
| F632   | 633,4                 | 616–651         | 35         |
| F650   | 651,1                 | 636–666         | 30         |
| F685   | 686,2                 | 675–698         | 23         |
| F715   | — (nominalno)           | 706–724         | 18         |
| F725   | 725.2                 | 712–738         | 26         |
| F750   | 746.0                 | 729–763         | 34         |
| F780   | 775.1                 | 754–796         | 42         |
| F808   | 810.3                 | 789–832         | 43         |
| F832   | 826,1                 | 810–843         | 33         |
| F850   | 846,5                 | 828–865         | 37         |
| F880   | — (nominalno)           | 867–893         | 26         |
| F905   | — (nominalno)           | 892–920         | 28         |
| F940   | 940,6                 | 923–958         | 35         |
| F950   | 945,1                 | 929–961         | 32         |
| F988 † | 985,3                 | 968–1003        | 35         |

_»Robovi pasov so izmerjene vrednosti polne širine na polovici maksimuma iz MAPIR jevih skenov filtrov po serijah — iste vrednosti, ki jih Chloros vtisne v vsak kalibriran izvoz.«_ »— (nominalno)« = skeniranje serije še ni bilo opravljeno; za te SKU-je je navedeno središče številka SKU, širina pa je podatek proizvajalca.

† »Odbojnost F988 je kalibrirana z uporabo plošče za merjenje odbojnosti v prizoru: pas leži zunaj kalibriranega območja svetlobnega senzorja DAQ, zato Chloros uporabi vaš najnovejši posnetek plošče in ga ohrani med posameznimi meritvami plošče.« Glejte [Kalibracijske tarče](calibration-targets.md).

Za nadzor kamere v realnem času, matrike, nastavitev omrežja in radiometrično obdelovalno verigo glejte [LATTICE priročnik](lattice/).

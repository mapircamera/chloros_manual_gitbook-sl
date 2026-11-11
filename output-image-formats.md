---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Oblike izhodnih slik

Chloros izvozi obdelane izdelke v štirih oblikah datotek. Obliko izberite v nastavitvah projekta (GUI), z `--format` (CLI) ali z `export_format` (SDK). CLI in SDK sprejemata natančno spodnje nize.

| Niz formata | Končnica | Tip pikslov | Obseg pikslov | Opombe |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(privzeto)* | `.tif` | digitalno število uint16 | 0 – 65535 | Priporočeno za fotogrametrijo / GIS. |
| `TIFF (8-bit)` | `.tif` | digitalno število uint8 | 0 – 255 | Za vse 8-bitne zajeme, ki ponujajo le 8-bitne formate: Chloros ne poveča bitne globine. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100 % odbojnost. Nekatere aplikacije ne morejo brati TIFF-datotek s plavajočo vejico; datoteke so večje. |
| `PNG (8-bit)` | `.png` | digitalno število uint8 | 0 – 255 | Stiskanje brez izgube, primerno za ogled na spletu in vizualizacijo. |
| `JPG (8-bit)` | `.jpg` | digitalno število uint8 | 0 – 255 | Stiskanje z izgubo, najmanjše datoteke. |

## Kje se shranijo izhodne datoteke

Datoteke se shranijo v projektni mapi, razvrščene po kameri in nato po datotečnem formatu:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera (model+lens+filter)
    ├── tiff16/                          # follows --format: tiff16, tiff8, png8, jpg8, or tiff32
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one <INDEX>_Index_Images/ folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Mapa kamere je `LATT-<sensor>-<lens>-F<filter>` za LATTICE in `<model>_<filter>` (npr. `Survey3N_RGN`) za Survey3. **Vsak izvožen izdelek ohrani ime izvorne datoteke — izdelek identificira mapa, ne pa končnica imena datoteke.** Celotna pravila si oglejte v razdelku [Kje se shranijo izhodne datoteke](reference/cli-reference.md) v referenčnem priročniku za CLI.

## Izdelki LATTICE (ravni zajemanja in izvoza)

En surovi okvir LATTICE se v enem prehodu razširi v vsak zahtevan izdelek. Vsaka vrsta izdelka ima svoj stikalo (potrditvena polja v grafičnem vmesniku ali v CLI-ju `--debayered` / `--preview` / `--radiance` / `--reflectance`, privzeto so vsi vklopljeni):

| Raven | Vsebina | Vrsta podatkov |
| --- | --- | --- |
| `raw` | Podatki Bayer neposredno iz senzorja (črno-bele kamere: en sam pas). Obdelava se vedno začne z neobdelanimi podatki. | Tako, kot so bili zajeti |
| `debayered` | Linearna demosaika — 3-kanalna za M3C, 1-kanalna v sivih odtenkih za M3M: odstranjen je temni odmik kalibracijskega paketa, nato sledi demosaika, ki upošteva robove, brez korekcije linearnosti, izravnave polja ali ločevanja barv. Izvoz `process` v vsakem načinu obdelave zapiše isto datoteko, kot jo na tej stopnji shrani zajem. | Linearni DN |
| `radiance` | Absolutna spektralna svetilnost iz celotne radiometrične verige, v **W/m²/sr/nm**. Vedno se zapiše kot 32-bitni TIFF (`tiff32/Radiance_Images/`), ne glede na izbrani izvozni format. | float32 |
| `reflectance` | Odbojnost ρ, kjer **DN 32768 = ρ 1,0 (100 %)** z rezervo do ρ 2,0. Pripravljeno za Pix4D. | uint16 |
| `preview` | Renderiranje, pripravljeno za prikaz: RGB = bela bilanca + gama; multispektralno = raztezek lažnih barv. | 8-bitni prikaz |

### Orientacija

Datoteke `raw` in `debayered` vedno ohranijo **izvirno** usmeritev senzorja; nastavitev obračanja/vrtenja kamere se uporablja le za druge izdelke (predogled, sevanje, odbojnost, indeks). Pri zajemih matrik ostanejo datoteke vseh elementov v izvirni usmeritvi, razen če zahtevate spremembo usmeritve (`--orient` v CLI, `orient=True` v SDK). Vsaka datoteka v XMP-ju beleži orientacijo, ki pripada njenim pikslom — `ChlorosRotation`, `ChlorosFlipHorizontal`, `ChlorosFlipVertical` — datoteka `ChlorosOrientationApplied` pa navaja, ali jo nosijo njeni lastni piksli (`false` v surovih in debayiranih datotekah). Chlorosov pregledovalnik slik obdeluje surove in debayirane datoteke v skladu s tem zapisom izključno za prikaz; same datoteke se nikoli ne prepisujejo. Datoteka brez tega zapisa (ustvarjena pred različico 1.3.0 ali z drugim orodjem) se prikaže takšna, kot je shranjena.

## Maske omejitve

Proizvodi sevanja in odbojnosti multispektralne kamere M3C izvajajo razmešavanje pasov v formatu 3×3. Kadar razmešavanje v pasu ustvari negativno vrednost, se ta vrednost omeji na nič (v živo se omejuje na popolnoma enak način). Novost v različici 1.3.0 je, da produkt, v katerem je bil omejen vsaj en piksel, dobi spremljajočo **masko omejitve**, ki natančno navaja, kateri piksli in pasovi so bili omejeni:

* **Kje:**v mapi `clamp_masks/`**poleg** map z izdelki, nikoli znotraj njih: `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif` in `tiff32/Radiance_Images/x.tif` → `tiff32/clamp_masks/Radiance_Images/x_clampmask.tif`.
* **Kaj:** enopasovni TIFF-datoteka `uint8` na lastni pikselni mreži izdelka (vključno z orientacijo, območjem zanimanja (ROI), združevanjem pikslov in poravnavo matrike). Bit 0 (`mask & 1`) je nastavljen tam, kjer je bil pas v kanalu R izdelka odrezan, bit 1 (`& 2`) v kanalu G, bit 2 (`& 4`) pa v kanalu B. Delež odrezanih pikslov po posameznih pasovih je v izdelku označen tudi kot `Chloros:BandRUnmixClampFraction`, `Chloros:BandGUnmixClampFraction` in `Chloros:BandBUnmixClampFraction`.
* **Kako je to navedeno v izdelku:** oznaka XMP `Chloros:UnmixClampMask` vsebuje pot do maske glede na mapo izdelka (`../clamp_masks/<product folder>/<name>_clampmask.tif`, vedno ločena z `/`), `none`, če ni bil odrezan noben piksel (maska ni zapisana), ali `unavailable`, če maske ni bilo mogoče zapisati. Za izdelek brez oznake se ločevanje barv ni nikoli izvedlo (kamere RGB in mono, surove in debayered ravni).

Ker nobena mapa izdelka ne vsebuje maske, lahko mapo izdelka, kot je `Reflectance_Calibrated_Images/`, predate programu Pix4D, Metashape ali drugi fotogrametrični programski opremi takšno, kot je, vključno s podmapami – predajte mapo izdelka, ne pa mape formata nad njo. Chlorosov lastni uvoz preskoči maske za omejevanje. CLI jih šteje ločeno: `Image products written: N (plus M clamp masks)`.

## Indeksni izdelki

Vsak indeks se shrani v svojo mapo `<INDEX>_Index_Images/` (npr. `NDVI_Index_Images/`).

* **Celoštevilčni formati**(`TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`) shranjujejo**linearno raztezanje za vsako sliko**, ne pa preslikave −1…1: DN 0 je najnižja indeksna vrednost tega okvirja, polna skala pa najvišja. Datoteka vsebuje `Chloros:IndexScaling=linear_minmax`, `Chloros:IndexMin` in `Chloros:IndexMax`, torej tudi `value = IndexMin + DN / full_scale × (IndexMax − IndexMin)`. Ker ima vsak okvir svoje končne točke, primerjajte okvire kot vrednosti, ne kot DN.
* **`TIFF (32-bit, Percent)`** shranjuje same indeksne vrednosti kot float32, brez ponovnega merjenja — najboljša izbira za kvantitativno delo v okviru celotnega poleta.

Novost v različici 1.3.0: indeksni produkt opisuje **samega sebe**, namesto da ponavlja oznake izvirnega surovega podatka. Njegove oznake GPS in kamere se kopirajo iz produkta, na podlagi katerega je bil izračunan (produkt odbojnosti, če ta obstaja), nato pa:

| Oznaka | Vrednost |
| --- | --- |
| `Chloros:ProcessingLevel` | `index` (tako da datoteka nikoli ni ponovno uvožena kot zajem) |
| `Chloros:IndexApplied` / `Chloros:IndexExpression` | `True` / uporabljena formula |
| `Chloros:IndexLevel` | `reflectance` ali `radiance` — na podlagi česa je bil indeks izračunan (ni navedeno, če ni nobenega od obeh) |
| `Chloros:IlluminationInvariant` | `True` za indeks, izračunan na podlagi odbojnosti, `False` na podlagi sevanja ali vrednosti senzorja |
| `Chloros:ChlorosPipelineVersion` | različica programa Chloros, ki je ustvarila indeks |

`Chloros:PixelScale` je odstranjen iz indeksnih produktov — uporabite `IndexMin` / `IndexMax` zgoraj. Celoten postopek ponovnega branja je naveden v poglavju »Celoštevilčni indeksni raster je raztezanje kontrasta za posamezno sliko« v [Referenca CLI](reference/cli-reference.md).

## Branje vrednosti odbojnosti pikslov

Odbojnost je shranjena kot celoštevilčna digitalna vrednost, **DN, ki pomeni ρ = 1,0 (100 % odbojnost), pa je odvisna od izvorne kamere**:

| Izvorna kamera | ρ = 1,0 je DN | Kako ugotoviti |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (rezerva do ρ 2,0) | V datoteko je vtisnjena oznaka XMP `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (omejeno na ρ 1,0) | Ni oznake `Chloros:PixelScale` – ta odsotnost je znak. (Izdelki Survey3 vsebujejo druge oznake obdelave `Chloros:*`, kot je `Chloros:CalibrationApplied`, v vseh načinih obdelave od različice 1.3.0 naprej.) |

**Preberite oznako XMP `Chloros:PixelScale` in delite z njo**, namesto da predpostavljate konstanto. Oznaka je definirana v domeni uint16, zato ostane `32768` v vseh izhodnih formatih, ki spreminjajo merilo – najprej normalizirajte shranjeni podatkovni tip nazaj na uint16 (×257 iz 8-bitnega, ×65535 iz float32).

{% hint style="warning" %}
**8-bitni posnetki.** Izvozni podatki 8-bitnega posnetka so 8-bitni. Njegova odbojnost se ponovno skalira in označi z `Chloros:PixelScale=32768`, tako da se merilo ohrani; 8-bitni DN-izdelki in datoteke odbojnosti, zapisane pred različico 1.3.0, so omejeni na območje 0–255 in nimajo oznake merila. Če oznaka v datoteki odbojnosti LATTICE manjka, ne predpostavljajte merila; namesto tega ponovno izvozite v 16-bitni ali 32-bitni obliki.
{% endhint %}

Za celotna pravila (vključno z oznakami, združljivimi z MicaSense) glejte **»Branje pikslov odbojnosti«** v [Referenca CLI](reference/cli-reference.md).

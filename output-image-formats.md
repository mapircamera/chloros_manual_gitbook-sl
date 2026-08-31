---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Oblike izhodnih slik

Chloros izvozi obdelane izdelke v štirih datotečnih oblikah. Izberite format v nastavitvah projekta (GUI), z `--format` (CLI) ali z `export_format` (SDK). CLI in SDK sprejemata natančno navedene nize znakov.

| Niz formata | Razširitev | Tip piksla | Obseg pikslov | Opombe |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(privzeto)* | `.tif` | uint16 digitalno število | 0 – 65535 | Priporočljivo za fotogrametrijo / GIS. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100 % odbojnost. Nekatere aplikacije ne morejo brati TIFF datotek s plavajočo vejico; datoteke so večje. |
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

Mapa kamere je `LATT-<sensor>-<lens>-F<filter>` za LATTICE in `<model>_<filter>` (npr. `Survey3N_RGN`) za Survey3. **Vsak izvožen izdelek ohrani ime izvorne datoteke – izdelek identificira mapa, ne pa končnica imena datoteke.** Za celotna pravila glejte [Kje se shranijo izhodni podatki](reference/cli-reference.md) v referenčnem priročniku CLI.

## Izdelki LATTICE (ravni zajemanja in izvoza)

En surovi okvir LATTICE se v enem prehodu razširi v vsak zahtevan izdelek. Vsaka vrsta izdelka ima svoj stikalo (polja za potrditev v grafičnem vmesniku ali CLI `--debayered` / `--preview` / `--radiance` / `--reflectance`, privzeto so vsi vklopljeni):

| Raven | Vsebina | Vrsta podatkov |
| --- | --- | --- |
| `raw` | Podatki Bayer neposredno iz senzorja (črno-bele kamere: en sam pas). Obdelava se vedno začne z neobdelanimi podatki. | Tako, kot so bili zajeti |
| `debayered` | Linearni demosaik — 3-kanalni za M3C, 1-kanalni v sivih odtenkih za M3M. | Linearni DN |
| `radiance` | Absolutna spektralna svetilnost iz celotne radiometrične verige, v **W/m²/sr/nm**. Vedno zapisano kot 32-bitni TIFF (`tiff32/Radiance_Images/`), ne glede na izbrani izvozni format. | float32 |
| `reflectance` | Odbojnost ρ, kjer **DN 32768 = ρ 1,0 (100 %)** z rezervo do ρ 2,0. Pripravljeno za Pix4D. | uint16 |
| `preview` | Render za prikaz: RGB = belina + gama; multispektralno = raztezanje lažnih barv. | 8-bitni prikaz |

## Branje vrednosti pikslov odbojnosti

Odbojnost je shranjena kot celoštevilčna digitalna vrednost, **DN, ki pomeni ρ = 1,0 (100 % odbojnosti), pa je odvisna od izvorne kamere**:

| Izvorna kamera | ρ = 1,0 je DN | Kako ugotoviti |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (rezerva do ρ 2,0) | V datoteko je vtisnjena oznaka XMP `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (odrezano pri ρ 1,0) | Ni oznak XMP `Chloros:*` — ta odsotnost je znak. |

**Preberite oznako XMP `Chloros:PixelScale` in delite z njo**, namesto da predpostavljate konstanto. Oznaka je definirana v domeni uint16, zato ostane `32768` v vseh izhodnih formatih, ki spreminjajo merilo — najprej normalizirajte shranjeni podatkovni tip nazaj na uint16 (×257 iz 8-bitnega, ×65535 iz float32).

{% hint style="warning" %}
**En primer po zasnovi ne vsebuje merila.** Ko se zajem iz 8-bitnega vira (BayerRG8) zapiše kot 8-bitni TIFF, potek obdelave omeji vrednosti na 0–255 namesto ponovnega prilagajanja lestvice, zato datoteka nima določene lestvice — Chloros namerno izpušča `Chloros:PixelScale` v tem primeru. Če oznaka v datoteki odbojnosti LATTICE manjka, ne predpostavljajte merila; namesto tega datoteko ponovno izvozite v 16-bitni ali 32-bitni obliki.
{% endhint %}

Za celotna pravila (vključno z oznakami, združljivimi z MicaSense) glejte **»Branje pikslov odbojnosti«** v [CLI referenčnem priročniku](reference/cli-reference.md).

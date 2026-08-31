# Chloros CLI Referenca

**Različica:**

1.2.0**Ustvarjeno:**

29. 7. 2026 ob 19:19 ·**Revidirano:**

30. 8. 2026**Ciljna publika:** Optimizirano za uporabo v sistemih za obdelavo velikih jezikovnih modelov (LLM); berljivo za ljudi.**Obseg:** Vsi podukazi programa `chloros-cli`, namenjeni uporabnikom, z možnostmi in primeri, ki jih je mogoče kopirati in prilepiti.

Ta dokument je popoln priročnik za orodje za ukazno vrstico `chloros-cli`, ki je priloženo MAPIR Chloros. Namenoma je izčrpen, da lahko LLM (ali človek) sestavi kateri koli podprt delovni tok iz spodnjih seznamov, ne da bi pregledoval izvorno kodo.

Če vas zanimajo le najpomembnejše točke, preskočite na:
- [Petminutni hitri začetek](#five-minute-quickstart)
- [Delovni tok prve povezave kamere LATTICE](#lattice-camera-first-connect-workflow)
- [Delovni tok prve povezave senzorja DAQ](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Načini zajemanja, snemalniki in ponovna obdelava v načinu brez povezave](#capture-modes-recorders--offline-reprocess)

---

## Konvencije

- Vsi ukazi imajo predpono `chloros-cli`. Na Windows je izvedljiva datoteka `chloros-cli.exe`; na Linux /Jetson pa je `chloros-cli`.
- Izbirni argumenti so prikazani kot `--flag`. Obvezni pozicijski argumenti so prikazani brez oklepajev.
- Kadar je navedena privzeta vrednost, se ob izpustitvi oznake uporabi ta vrednost.
- CLI je »thin« HTTP odjemalec, ki deluje prek strežnika Chloros (strežnik Flask na `127.0.0.1:5000`). Večina ukazov samodejno zažene backend. `CHLOROS_BACKEND_URL=<url>` usmerja družine ukazov **`lattice`**,**`project`**in**`daq pool-*`** na oddaljeni backend — osnovne ukaze (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) namerno fiksirajo `http://127.0.0.1:<port>` in ga prezrejo (literal IPv4 prepreči kazen »Windows« za `localhost`→`::1` ~2 sna zahtevo). Glej [Okoljske spremenljivke](#environment-variables).
- Za vse klice na SDK / CLI je potrebna prijava z računom Chloros+ (izvedi `chloros-cli login` enkrat na računalnik; shranjeno v predpomnilniku v `~/.chloros/`).
- V primerih so uporabljene poti Linux; na Windows zamenjajte `/home/user/...` z `C:/Users/.../...`.

---

## Pregled na najvišji ravni

```
chloros-cli [global options] COMMAND [command options]
```

### Globalne možnosti

| Oznaka | Opis |
| --- | --- |
| `--backend-exe PATH` | Prepis samodejno zaznanega izvedljivega datoteka zadnjega dela. |
| `--port N` | Vrata strežnika HTTP (privzeto: `5000`). |
| `-v, --verbose` | Omogoči podrobni izpis. |
| `--restart` | Prisilni ponovni zagon ozadja (prekine vse tekoče procese `backend_server.py`). |
| `--version` | Izpiši različico (`Chloros CLI 1.2.0`). |
| `--help` | Prikaži pomoč najvišje ravni. |

### Kazalo ukazov

| Ukaz | Namen |
| --- | --- |
| [`process`](#chloros-cli-process) | Obdelava mape z zajemi Survey3 ali LATTICE od začetka do konca. |
| [`login`](#chloros-cli-login) | Preveri pristnost tega računalnika z računom Chloros+. |
| [`logout`](#chloros-cli-logout) | Izbriši shranjene poverilnice iz predpomnilnika. |
| [`status`](#chloros-cli-status) | Prikaži trenutno stanje licence / avtentifikacije. |
| [`export-status`](#chloros-cli-export-status) | Prikaz napredka izvoza v realnem času v Thread-4 med izvajanjem `process`. |
| [`language`](#chloros-cli-language) | Nastavi ali prikaži seznam jezikov za prikCLIja (podprtih je 38). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Privzeta projektna mapa (skupna z grafičnim vmesnikom). |
| [`update`](#chloros-cli-update) | Preveri in namesti posodobitve za CLI (Linux /Jetson). |
| [`selftest`](#chloros-cli-selftest) | Sistemsko diagnosticiranje + hitri testi. |
| [`time-sync`](#chloros-cli-time-sync) | Stanje in nadzor glavnega PTP-ja. |
| [`lattice`](#chloros-cli-lattice) | Nadzor in zajem kamere LATTICE (več kot 45 podukazov). |
| [`daq`](#chloros-cli-daq) | Nadzor spektralnih senzorjev DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Odpiranje in upravljanje shranjenega projekta »Chloros« (kamere + DAQ-ji). |

---

## Namestitev

`chloros-cli` je vključen v namestitveni program za namizje Chloros na vseh podprtih platformah — ni potreben ločen prenos CLI. Z namestitvijo paketa za platformo se `chloros-cli` doda v vaš `PATH` skupaj z namizno aplikacijo in binarnim programom v ozadju, ki ga ta poganja.

Najnovejše prenose: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Namestitveni program vsebuje tudi priročne skripte za zagon (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`), ki odprejo takoj uporabno lupino CLI; ti skripti so opisani v [CLI Uporabniškem priročniku](../CLI.md) in se tukaj ne ponavljajo.

### Windows (.exe)

1. S spletne strani za prenos prenesite namestitveni program Windows.
2. Zaženite `Chloros-Setup-x.y.z.exe` in sledite navodilom čarovnika. Privzeta pot namestitve je `C:\Program Files\Chloros\` (CLI se nahaja v `C:\Program Files\Chloros\cli\`, ki ga namestitveni program doda v PATH).
3. Odprite nov terminal (`cmd.exe`, PowerShell, ali terminal »Windows«), da se zazna posodobljeni `PATH`.

```powershell
chloros-cli --version
```

Namestitveni program samodejno doda `chloros-cli.exe` v vaš sistem `PATH` in vključuje izvedbeno okolje Arena SDK, potrebno za kamere LATTICE.

### Linux amd64 (.deb)

Za delovne postaje z operacijskim sistemom Ubuntu 22.04 LTS ali novejšim / na osnovi Debiana z arhitekturo x86_64.

> **Ubuntu 20.04 ni podprt.** Seznam odvisnosti paketa izhaja iz
> tistega, s čimer se backend dejansko povezuje, kar vključuje `libc6 (>= 2.34)`;
> Focal vključuje glibc 2.31. `apt` namesto da bi dopustil napako med
> izvajanjem, namestitev preprosto zavrne.

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

Paket .deb namesti:
- `chloros-cli` do `/usr/bin/chloros-cli`
- Prekompilirani backend v `/usr/lib/chloros/chloros-backend`
- Izvedbeno okolje Arena SDK (za kamere LATTICE)
- Modeli za odstranjevanje šuma, kalibracijski paketi in konfiguracija kanala za posodobitve

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Enaka struktura kot pri .deb datoteki za amd64, z CUDA-izgradnjo, prilagojeno za Jetson Orin / Orin NX / Orin Nano.

### Enkratna avtentifikacija na napravo

Vsaka platforma zahteva enkratno prijavo na Chloros+, preden začnejo delovati klici SDK / CLI:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Pristopni podatki so shranjeni v `~/.chloros/user_session.json`.

### Preverjanje namestitve

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **Potrebna je naročnina na Chloros+.**Za CLI je potreben aktiven načrt Chloros+.**Copper**je osnovna stopnja Chloros+ — vsaka plačljiva stopnja Chloros+ ima dostop do CLI / SDK; le brezplačna stopnja**Iron** ga nima. (Povezava med ID-ji paketov: `0`=Iron/free, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Nadgradite na [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> To omejitev uveljavlja backend, ne le CLI: zahteva z oznakama SDK / CLI brez plačanega načrta se zavrne z napako `403 PLAN_UPGRADE_REQUIRED`, ne glede na to, ali prihaja iz `chloros-cli`, Python SDK ali ročno izdelanega odjemalca HTTP. Odjavljeni uporabnik namesto tega prejme `401 AUTH_REQUIRED`. Dostop deluje brez povezave v obdobju odloga za načrt (30 dni na mesec, do izteka veljavnosti za letni načrt) in se ustavi, ko to obdobje poteče; `chloros-cli status` še naprej deluje, da je razlog viden (to je edina pot SDK / CLI, ki je izvzeta iz omejitve po stopnjah – `GET /api/license-status`).

---

## Petminutni hitri začetek

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

Obdelajte mapo s slikami skozi celoten potek obdelave »Chloros« (prepoznavanje cilja → kalibracija → vinjeta → odbojnost → izvoz indeksa).

### Pregled

```
chloros-cli process INPUT [OPTIONS]
```

### Argumenti za določanje položaja

| Argument | Opis |
| --- | --- |
| `INPUT` | Pot do vhodne mape, ki vsebuje datoteke `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) ali `.dng`. |

### Splošne možnosti

| Oznaka | Privzeto | Opis |
| --- | --- | --- |
| `-o, --output PATH` | nova mapa z časovnim žigom v vaši privzeti poti projekta (`~/Chloros Projects`, če ni drugače nastavljeno) | Projektna mapa, ki jo je treba ustvariti ali ponovno uporabiti. Če mapa že vsebuje `project.json`, se namesto prepisovanja ustvari sorodna mapa `_1`/`_2`. |
| `-n, --project-name NAME` | avtomatsko (časovni žig) | Ime projekta. |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` uporablja nevronski debayer Chloros+; počasnejši, a višja kakovost. |
| `--vignette / --no-vignette` | `--vignette` | Popravek vinjetiranja. |
| `--reflectance / --no-reflectance` | `--reflectance` | Kalibracija odbojnosti (uporablja ciljni panel, če je najden, NIST-ovo kalibracijo po serijski številki za LATTICE). Pri multispektralnih posnetkih LATTICE to hkrati deluje kot stikalo za **produkt** odbojnosti — glej [Stikala za izvoz po produktu](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | izklopljeno | Uporabi PPK GNSS popravke iz sidecar datotek. |
| `--exposure-pin-1 MODEL` | izklopljeno | Priključi »Survey3« sistem z dvema kamerama&#x27;model »pin-1«. |
| `--exposure-pin-2 MODEL` | izklopljeno | Priključi model »pin-2«. |
| `--recal-interval SECONDS` | 0 | Vsake N sekund snemanja prisili ponovno izvajanje kalibracijskih izračunov. |
| `--timezone-offset HOURS` | lokalno | Prepisati časovni zamik, vgrajen v izhodne metapodatke. |
| `--format FORMAT` | `TIFF (16-bit)` | Ena od naslednjih vrednosti: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | noben | Indeksi vegetacije (`NDVI`, `NDRE`, `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …). |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Prisili začetno točko poteka za datoteke LATTICE TIFF (datoteke Survey3 .raw to ne vpliva). Prav tako izhod v sili, ki omogoča, da se zajem brez **raw** sploh obdela — glej [Kako izgleda mapa z zajetimi posnetki](#how-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | vklopljeno | Izpiše linearni produkt brez bayersovega filtra (`Debayered_Images`). Glej [Stikala za izvoz po posameznih izdelkih](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | vklopljeno | Izpiše predogled za zaslon (`Preview_Images`): RGB = bela bilanca (DAQ-svetlobni vir, če je na voljo, sicer siva skala) + gama; multispec = raztezek lažnih barv. |
| `--radiance / --no-radiance` | vklopljeno | Izpisuje sežar float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referenca za produkt odbojnosti LATTICE: `auto` = cilj v okviru, ki je prestal kontrolo kakovosti (QA), je absolutna referenca, DAQ-downw(ρ = π·L/E) kot nadomestna vrednost; `target` = strogo (brez zamenjave DAQ); `daq` = DAQ-avtoritativno. Glej [Preklopniki za izvoz po posameznih izdelkih](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | noben | Imenik skenov **izmerjene** odbojnosti cilja na enoto (`<serial>.csv`); v primeru neuspeha se uporabijo nominalni spektri T3/T4P. |
| `--array-alignment / --no-array-alignment` | vklopljeno | LATTICE-matrike: na vsak obdelan izdelek (debayering / predogled / sevanje / odbojnost / indeks). Brez operacije za slike brez oznak. |
| `--array-alignment-crop / --no-array-alignment-crop` | izrez | Izrezani izvozi se poravnajo na skupno prekrivno območje matrike, tako da vsi moduli delijo eno stopnjo; `--no-…` ohrani celotno površino senzorja (črno zapolnjevanje zunaj vira). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Ponovno vzorčenje za popravke poravnave. `nearest` ohranja natančne DN vire (brez mešanja radiometričnih vrednosti med pikslami). |

### Možnosti zaznavanja ciljev

| Oznaka | Opis |
| --- | --- |
| `--min-target-size PIXELS` | Najmanjša velikost cilja na plošči (px) za detektor. |
| `--target-clustering 0-100` | Občutljivost združevanja v skupine. |
| `--target / --targets` | Obravnava vhodno mapo kot mapo, ki vsebuje izključno ciljne plošče (preskoči zaznavanje pregledov). |

### Primeri

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### Stikala za izvoz po posameznih izdelkih (LATTICE multispektralno)

Obdelava LATTICE se razširi na **vse ustrezne izdelke v enem prehodu**. Štiri stikala po tipu — `--debayered`, `--preview`, `--radiance`, `--reflectance` — so**privzeto vklopljeni**; za izklop enega uporabite obrazec `--no-<type>`. Glavne kamere RGB vedno oddajajo le debayered + predogled (brez sevanja/odbojnosti po posameznih pasovih), zato so `--radiance`/`--reflectance` zanje ne delujejo. Stikala se ne upoštevajo za Survey3 `.raw` (ki sledi standardni poti odbojnosti/cilja). *(Stari zastavici `--radiometric-output {reflectance,radiance,sensor-response}` sta bili **odstranjeni** in nadomeščeni s temi stikali; raven `sensor-response` ne obstaja več.)*

| Izdelek | Izhod | Potrebno zajemanje podatkov o navzdolnem sevanju? |
| --- | --- | --- |
| `--debayered` | Linearni demosaik (`Debayered_Images`). | Ne. |
| `--preview` | Predogled zaslona (`Preview_Images`): RGB = WB + gama; multispec = raztezek v lažnih barvah. | Ne. |
| `--radiance` | float32 W/m²/sr/nm iz celotne radiometrične verige (`Radiance_Images`). | Št. |
| `--reflectance` | uint16 odbojnost ρ (`32768` = 1,0), pripravljeno za Pix4D. | **Da**, razen če ga ne pritrdi cilj znotraj okvirja, ki je uspešno prestal kontrolo kakovosti (glej spodaj). |

`--reflectance-source` izbere referenco odbojnosti:**`auto`**(privzeto) določi cilj v okviru, ki je uspešno prestal kontrolo kakovosti, za**absolutno referenco**— verige empiričnih krivulj, pritrjene na cilj, se navzkrižno primerjajo na izločenih panelih, zmagovalna vrednost pa se uporabi – če cilj ni prisoten ali QA ni uspešen, se uporabi razmejitvena vrednost DAQ za navzdolno sevanje (ρ = π·L/E);**`target`**je strog (brez nadomestitve DAQ);**`daq`**se odloči za obnašanje, pri katerem je DAQ avtoritativen. Geometrija cilja (ArUco / fiksno območje zanimanja / trak) izhaja iz konfiguracije cilja projekta; `--target-reflectance-dir DIR` hrani**izmerjene** skene na enoto (`<serial>.csv`), ki se poiščejo po serijski številki/QR ciljne enote, z nominalnimi spektri T3/T4P kot rezervo.

Pot odbojnosti DAQ samodejno razreši **časovno usklajeno navzdolno sevanje**avtomatsko iz posnetega**`.daq`**(DAQ-U/M/E)**ali iz DAQ-M-ovega lastnega `.csv`**, ki se nahaja skupaj s slikovnim gradivom. Če paket kalibracij za posamezno kamero ali DAQ ni shranjen v lokalnem predpomnilniku, ga proces**ob prvi uporabi samodejno prenese iz AWS** (zahteva enkratno internetno povezavo; shranjen v predpomnilniku pod `~/.chloros/`).

#### Branje pikslov odbojnosti (Pix4D / Metashape / vaši lastni skripti)

Odbojnost je shranjena kot celo število DN, **DN, ki pomeni ρ = 1,0, pa je odvisen od izvorne kamere**:

| Izvor | ρ = 1,0 je | Kako prepoznati |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (razpon do ρ 2,0) | V datoteko je vtisnjena oznaka XMP `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (odrezano pri ρ 1,0) | Ni oznak XMP — ta odsotnost *je* signal. |

**Preberite vrednost in delite z njo** namesto da predpostavljate konstanto. Oznaka je definirana v domeni uint16, zato ostane `32768` v vseh izhodnih formatih, ki spreminjajo merilo — `TIFF (16-bit)`, `PNG (8-bit)`, `JPG (8-bit)` in `TIFF (32-bit, Percent)` so vsi samopisni (najprej normalizirajte shranjeni podatkovni tip nazaj na uint16: ×257 iz 8-bitnega, ×65535 iz float).

> **En primer po zasnovi nima lestvice.** Ko se zajem iz 8-bitnega vira (BayerRG8) zapiše kot 8-bitni TIFF, se v procesni verigi vrednosti *odrežejo* na 0,,255 namesto da bi spremenil merilo, zato se vsaka vrednost nad ρ≈0,008 izravna na 255 in datoteka nima opredeljene lestvice. Chloros namerno izpušča tako `Chloros:PixelScale` kot tudi tuplo `MicaSense:RadiometricCalibration` in zabeleži razlog za to. **Če oznaka v datoteki odbojnosti LATTICE manjka, ne predpostavljajte lestvice – ponovno izvozite v 16-bitni ali 32-bitni obliki**, namesto da delite piksle, ki nikoli niso bili deljivi.

#### EXIF prenesen v izvoz

`process` kopira **GPS-blok in njegov ExifIFD** izvirnega posnetka v vsak izdelek, zato
izvoz vsebuje `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` in
`CameraSerialNumber` skupaj z georeferenciranjem.

**`FocalLength` ni neobvezna oznaka za fotogrametrijo.** Pix4D izračuna razdaljo med točkami na tleh na podlagi
goriščne razdalje in nadmorske višine; če oznaka manjka, se program zateče k močno napačnemu merilu. Pri enem
letu nad nasadom pomaranč z 49 posnetki je manjkajoča oznaka območje velikosti 411 m × 160 m spremenila v rekonstruirano
območje velikosti 47,8 km × 13 km – ortofoto z 455 MP, ki je v večini brez podatkov, kar se je nato obravnavalo kot problem s sestavljanjem in
problem z datoteko BigTIFF, preden je kdorkoli preveril GSD. Če se vaša ortofotografija prikaže v neverjetnem
merilu, najprej zaženite `exiftool -FocalLength` na izvoženem izdelku.

Kopija namerno **ni** `-all:all`: strukturne oznake IFD0 pri kopiranju
pokvarijo izhod LATTICE, `ExifImageWidth` / `ExifImageHeight` pa sta izključena, ker opisujeta
*izvorni* posnetek – izvoz, katerega velikost je bila kdaj koli spremenjena, bi sicer vseboval dimenzije,
ki bi bile v nasprotju z njegovo lastno rastrsko sliko. XMP se zapiše neposredno namesto da bi se kopiral, saj orodje ExifTool
pri kopiranju bloka XMP zavrže oznake XMP iz istega klica (kar bi izpustilo oznake kalibracije MAPIR
).

### Kam se shranijo izhodni podatki

Izdelki se zapišejo **v projektno mapo, razvrščeni po kameri in nato po datotečnem formatu**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Mapa kamere je `LATT-<sensor>-<lens>-F<filter>` za LATTICE (kar ustreza EXIF-podatkom posnetka
`Model`) ter `<model>_<filter>` za Survey3 — dve kameri, ki si delita senzor in filter, vendar se razlikujeta
v objektivu, imata ločeni drevesni strukturi, saj se razlikujeta v vinjeti, vidnem polju in popačenju. Mapa z
formati sledi `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` ali `tiff32` za
`TIFF (32-bit, Percent)`.

> **Vsak izvožen izdelek ohrani ime izvorne datoteke.** Izvoz radianca
> `capture_…_raw.tif` se še vedno imenuje `capture_…_raw.tif` — le da se nahaja v
> `tiff32/Radiance_Images/`. **Produkt identificira mapa, ne ime datoteke**, zato iskanje
> po `*radiance*.tif` ne najde ničesar; namesto tega uporabite iskanje po mapi.

### Zapisi svetlobnega senzorja — kalibrirani `.daq` + `.csv`

`process` obdeluje tudi zapise `.daq` v vaši vhodni mapi, za to pa **ne**
potrebuje nobenih slikovnih podatkov: DAQ-U / DAQ-M / DAQ-E, ki leti samostojno, je popoln
zajem, mapa, ki vsebuje samo datoteke `.daq`, pa je veljaven vhod.

DAQ se lahko snema **brez** kalibracije — to je tisto, kar javni
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) snemalnikov
(`record_daq.py`) počnejo privzeto: zapisujejo surove števne vrednosti senzorjev in datoteko označijo tako, da
Chloros pridobi tovarniško kalibracijo tega senzorja **po serijski številki** (najprej iz lokalnega predpomnilnika,
nato v oblaku MAPIR) ter jo uporabi. `process` zapiše rezultat nazaj:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` vsebuje eno vrstico na odčitek: časovni žig UTC, čas integracije, skupno moč,
fotopični/skotopični luks, PPFD (in njegova razdelitev na modro/zeleno/rdeče), najvišja valovna dolžina, nato pa
celotno spektrum na senzorjevi lastni mreži valovnih dolžin. `.daq` se ponovno uvozi, ne da bi ga bilo treba
ponovno kalibrirati.

Ob uspešnem izvajanju poroča `Light-sensor products written: N (calibrated .daq + .csv)`.
Besedilo v oklepaju opisuje, kaj je bilo dejansko zapisano, torej se glasi
`(RAW COUNTS — this sensor has no calibration bundle)` za senzor brez svežnja in
`(N calibrated, M raw counts)` za mapo, ki vsebuje oboje. Naslovi lastnega backenda
naslova `[DAQ-EXPORT]` in `[RUN-SUMMARY]` črpata besedilo na enak način – nobeden od
teh treh ne more označiti surovega izvoza kot kalibriranega.

Zapis DAQ-U / DAQ-M / DAQ-E, katerega kalibracijski paket ni mogoče pridobiti – ker ste
brez internetne povezave ali ker za ta senzor ni shranjene kalibracije – je **preskočen z navedbo razloga** v
vrstici `[DAQ-EXPORT]`, nikoli pa se ne zapiše kot »kalibrirana« datoteka z neobdelanimi števili.
Povežite se z internetom in ponovno zaženite postopek. Razlog je tisti, ki ga je bralnik dejansko
ugotovil za to datoteko (neberljiva shema, ni paketa, napaka pri pisanju), povzetek
izvajanja pa navaja **posamezne** razloge – dvajset preskočenih datotek zaradi enega vzroka se prikaže kot en
vzrok, ne pa kot dvajset ponovitev tega vzroka.

#### Izvoz zapisov DAQ-A kot surovih števcev

Družina **DAQ-A** je starejša od sistema svežnjev po serijskih številkah in nima kalibracijskega svežnja,
ki bi ga bilo treba pridobiti – namesto tega se kalibrira na terenu z uporabo cilja odbojnosti, kar je
razlog, zakaj ga nikoli ni potrebovala. Zavrnitev teh posnetkov jim ni pustila nobene možnosti, da bi sploh pridobili
številk sploh ne bi mogli pridobiti, zato jih izvozijo pod **drugačnim imenom**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Drugačno ime datoteke namesto oznake znotraj datoteke, saj mora oznaka ostati ohranjena
tudi pri pošiljanju po e-pošti kot golo ime. Glava datoteke `.csv` navaja
`raw spectral sensor counts (NOT irradiance)` in opozarja, da so vrednosti primerljive
**znotraj** datoteke – kar je točno tisto, za kar jih uporablja kalibracija na podlagi cilja – in
ne med senzorji. Fotometrični stolpci, odvisni od moči (skupna moč, fotopični in
skotopični luks, PPFD) so zapisani kot **NULL** namesto da bi bili integrirani iz števila impulzov, povzetek
izvajanja pa navaja `RAW COUNTS`, zato »izvoženih“ v dnevniku ne more biti razumljeno kot obsevanje.

Starejši zapisi **v1.01 / v1.02** (te jih zapiše DAQ-A-SD) ne vsebujejo epohe za posamezno odčitavanje,
temveč le čas zapisa datoteke. Program za usklajevanje slike↔navzdolnega sevanja jih še vedno zavrača – usklajevanje
s časom zapisa bi bila nevidno napačna — vendar jih izvoznik prebere,
CSV pa natisne `clock=daq_created_on`, tako da izdelek navaja, na kateri uri temelji.

### Opombe

- `process` samodejno zazna, ali je vaša mapa tipSurvey3, LATTICE ali mešana.
- Napredek se prenaša prek dogodkov, ki jih pošilja strežnik (Server-Sent Events); CLI prikazuje napredek v realnem času za vsak niti (Detecting, Analyzing, Processing, Exporting).
- Pri Linux /Jetson CLI preveri zamenjavo pomnilnika in lahko izda opozorilo pred obdelavo velikih map. Debayer, ki upošteva teksturo, samodejno uporabi omejitev frekvence GPU na Jetsonih z nizko porabo energije (Nano, Orin Nano).
- Ob uspešnem zaključku izvedba poroča, koliko slikovnih izdelkov je zapisala (`Image products written: N`).

#### Izvedba, ki ne zapiše nobene slike, se ne uspe

Če ste zahtevali izdelke, izvedba pa ni zapisala **ničesar** — le `project.json` in
`calibration_data.json` — `process` to obravnava kot napako: izpiše
`Processing finished but wrote no image products.` in **se zaključi z vrednostjo, ki ni nič**, tako da ga skript lahko
zazna. Sporočilo navaja ime projektne mape in običajne vzroke:

- vhodna mapa ni bilaprepoznana kot zajem (preverite postavitev in `--input-level`) ali
- so bili vsi zahtevani produkti preskočeni, ker niso bili primerni za te kamere (npr. zahteva po
  sevanju/odbojnosti iz kamer, ki podpirajo le način »RGB«).

Ponovno zaženite z `--verbose` in v dnevniku ozadja preverite vrstice `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
ki pokamero preskočene vnoske, ki sicer ne pridejo v izhodno datoteko CLI.

Namerno izvajanje, namenjeno izključno metapodatkom — vsi izdelki so izklopljeni in ni `--indices` — je še vedno
**uspešno**, saj je v tem primeru prazen izhod slike pravilen rezultat.

Enako velja za **izvedbo, ki vključuje le svetlobni senzor**: mapa z zapisi `.daq` po definiciji ne vsebuje slik za izvoz
, zato se izvedba ocenjuje na podlagi kalibriranih datotek `.daq` / `.csv`, ki jih je namesto tega zapisal.

---

## `chloros-cli login`

Avtentificirajte ta računalnik z oblačnim računom Chloros+. Pristojnosti so varno shranjene v predpomnilniku v `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Primeri

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (odstranjevanje ali podvajanje delov gesla). Ob napaki 401 CLI samodejno ponovi poskus z dodatkom `$$`, nato pa s polovico gesla brez podvajanja; če je ponovni poskus uspešen, vas prijavi in izpiše pravilno sintakso z enim samimnavedbo, ki jo je treba uporabiti naslednjič.

> **Uporaba brez grafičnega vmesnika/prek skripta: odsotnost shranjene seje pomeni interaktivno pozivno okno, ne pa hitre napake.** Vsak ukaz, ki sproži ozadje (`process`, `status`, `export-status`, `time-sync`, …) brez shranjene licence/seje se pred nadaljevanjem preusmeri v interaktivno ukazno vrstico `Email:` / `Password:` na stdin, preden nadaljuje. Avtomatizirana naloga brez shranjene seje se bo zato zataknila in čakala na vnos — pred načrtovanjem brezzaslonskega dela enkrat na vsakem računalniku zaženite `chloros-cli login EMAIL PASSWORD`.

---

## `chloros-cli logout`

Izbriše shranjeno sejo in pri naslednjem klicu prisili k novi prijavi.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Prikaže trenutno stopnjo licence (Iron/Copper/Bronze/Silver/Gold), avtentificiranega uporabnika in število vezav na naprave.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Preveri trenutni napredek izvoza Thread-4 v realnem času. Varno je poklicati **med** izvajanjem `process` iz drugega lupine.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Nastavi jezik prikaza v CLI (podpira 38 jezikov, vključno s CJK, RTL in indijskimi jeziki). Na starejših konzolah, ki ne morejo prikazati skripta, se brez težav preklopi na angleščino.

```
chloros-cli language [LANG_CODE] [--list]
```

### Primeri

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Ukazi za projektno mapo

Ti ukazi upravljajo privzeto lokacijo projektne mape (skupno z grafičnim vmesnikom).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update`

Linux/ Samo za Jetson. Preveri `version_url` iz `/etc/chloros/update.conf` in ponudi prenos ter namestitev ustreznega `.deb`.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Na Linux /Jetson CLI izvaja tudi **avtomatsko preverjanje posodobitev ob vsakem zagonu** (ne blokira, nikoli ne zamuja izvedbe ukaza): prebere `/etc/chloros/update.conf`, rezultat shrani v predpomnilnik za 1 uro v `~/.chloros/update_cache.json` in izpiše `Update available: vX.Y.Z / Run: chloros-cli update`, če obstaja novejša različica. V primeru kakršne koli napake ali ob zagonu na Windows se tiho preskoči.

---

## `chloros-cli selftest`

Izvede 7-stopenjski preizkus delovanja: različica, razpoložljivost vrat, zagon backenda, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), prisotnost modela za odstranjevanje šuma, pripravljenost CUDA+odstranjevalca šuma.

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

Stanje in nadzor glavnega strežnika PTP. Gostitelj Chloros izvaja glavni strežnik PTP; kamere LATTICE in enote DAQ-E delujejo kot podrejeni mu za časovne oznake med napravami.

| Podukaz | Opis |
| --- | --- |
| `status` | Prikaži stanje glavnega strežnika, prioritete BMCA, identiteto ure. |
| `peers` | Seznam podrejenih naprav, zaznanih prek Delay_Req (kamere + senzorji DAQ-E). |
| `cameras` | Stanje PTP po posameznih kamerah (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Ponovni zagon procesa grandmasterja. |
| `set-priority --priority1 N --priority2 N` | Prepis prioritet BMCA. |

### Primeri

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

Upravljanje kamere LATTICE. Vsak podukaz poteka prek ozadja Chloros; ozadje upravlja z naborom kamer, zato naslednji klici CLI ponovno uporabijo isti odprti ročaj.

### Splošne možnosti (skupne večini podukazov)

| Oznaka | Opis |
| --- | --- |
| `-d, --device N` | Indeks kamere (privzeto: 0). |
| `-s, --serial SN` | Posebna serijska številka; prevlada nad `--device`. |
| `--serials SN1,SN2,…` | Serijske številke, ločene z vejico, za delovanje z več kamerami. |
| `--all` | Deluje na vsaki odkriti kameri. |
| `--exposure US` | Čas osvetlitve v mikrosekundah. |
| `--gain DB` | Ojačanje v dB. |
| `--pixel-format FMT` | npr. `BayerRG8`, `BayerRG12`. |
| `--width N` / `--height N` | Dimenzije slike. |
| `--preset {default,high_quality,high_speed,triggered}` | Uporabi prednastavljeno konfiguracijo. Vse deluje v prostem teku, razen `triggered`, ki kamero aktivira ob signalu na liniji 2 — če ta linija ni aktivirana, bo kamera čakala v nedogled, namesto da bi posnela. |
| `-o, --output DIR` | Izhodni imenik (privzeto: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Velikost paketa GVSP. `auto` izvaja ICMP+GVSP-preizkuse; `jumbo` = 9000; `standard` = 1500. |

### Potek prvega povezovanja kamere LATTICE

```bash
# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Auto-shrink frame size to fit the wire
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Referenca podukazov

#### Odkrivanje in informacije

| Podukaz | Namen |
| --- | --- |
| `lattice info` | Seznam priključenih kamer (proizvajalec, model, serijska številka, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analiza gostiteljskega sistema za optimalno konfiguracijo kamere. `--no-discover` preskoči odkrivanje kamer (hitrejša analiza, ki temelji izključno na mrežni kartici). |
| `lattice network [--fix] [--estimate] [--cameras N]` | Preverjanje/popravek nastavitev omrežne kartice; ocena pasovne širine/FPS. |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | Omrežne zmogljivosti backenda s stabilno shemo + priporočilo za matriko (vrne `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `auto_capped_fps` ohranja zahtevano ločljivost, vendar omeji ciljno število sličic na sekundo (fps) — preberi `recommended.recommended_target_fps` in ga posreduj kot cilj povezave; obravnavaj ga kot uspeh, ne kot napako. |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | Kaj-if analiza brez odpiranja kamer. **`--n-active` je skupno število kamer na omrežju, ne le v tem nizu**— povečajte ga, kadar samostojne kamere prenašajo hkrati ali če je zmogljivost omrežja izračunana na podlagi povpraševanja, ki jih ne upošteva v celoti (privzeto: `len(--models)`). Vedno izpiše agregatne vrstice `Wire budget:` (zahtevani MB/s v primerjavi z zgornjo mejo, varno pred kolizijami) in `Max cameras:`, in označi `** OVER-SUBSCRIBED**`, kadar niz prekomerno obremenjuje omrežje — glej [Model fps in burst za niz](#array-fps--burst-model). |
| `lattice gpu` | Prikaži stanje grafičnega procesorja. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Preveri ali posodobi firmware kamere. Lokalna izbira `.fwa` je fiksirana: datoteka v `firmware/<MODEL_PREFIX>/`, ki ustreza `MIN_FIRMWARE_VERSION` zgradbe, se naloži, če je prisotna (najnovejša različica služi le kot rezervna možnost), zato je novejša slika proizvajalca, shranjena na disku, neaktivna, dokler se ta pripet ne posodobi — namerno novejše izdaje pridejo do enot prek podpisanega AWS-manifesta, kar je priporočljivo, če je ta novejši. |
| `lattice presets [--apply NAME]` | Prikaži ali uporabi prednastavitve kamere. |
| `lattice status` | Stanje kamere v živo. |

#### Snemanje

| Podukaz | Namen |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | Posamezni okvir. **Privzeto shrani vse vrste izvoza** (`--processing all`); glej [Ravni izvoza zajema](#capture-export-levels-the-all-default). `--levels` shrani eksplicitno podmnožico (prevladuje nad `--processing`); `--force-daq` zapiše dodeljeno vrednost DAQ kot `.daq`, tudi pri zajemu, ki vsebuje le surove podatke. `--jpeg-quality` = JPEG kakovost 1–100 (privzeto 95). |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | Prenos na disk do pritiska Ctrl+C. |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Predogled v živo v formatu MJPEG v brskalniku. `--ae-damping` nastavi samodejnodušenje avtomatske osvetlitve (0,4–100). |

#### Nastavitev senzorja

| Podukaz | Namen |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | Branje/pisanje katerega koli vozlišča GenICam. |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | Osvetlitev in AE. |
| `lattice gain [--auto] [--off] [--set DB]` | Ojačanje in samodejno ojačanje. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | Območje zanimanja senzorja in združevanje pik. |
| `lattice format [--set FMT] [--list]` | Format pik. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Strojni/programski sprožilec. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (brez zastavic = enkratna belina) | Operacije beline. RGB Samo za kamere /Bayer; brez operacije (preskočeno) na mono M3M. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | Barvni procesni tok za prikaz »RGB«. `natural` (privzeto) je poceni obdelava v realnem času; `enhanced` doda odstranjevanje barvnih robov + živost + lokalni kontrast CLAHE za celovitz približno dvakrat višjimi stroški obdelave na posamezni sliki, torej nižjo **živo** hitrost snemanja — shranjeni posnetki vedno dobijo popolno obdelavo v obeh primerih. Samo za kamere RGB /Bayer; preskoči se pri mono M3M. |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | Nasičenost/kontrast zaslona (kamere s filtrom RGB). Preskoči se pri mono M3M. |
| `lattice filter [--set NAME] [--list]` | Nastavi model filtra kamere (`RGN-IMX265`, `OCN`, `NGB`, …). |
| `lattice power [--sleep]` | Preverjanje napajanja/termičnih vozlišč; vklop/izklop mirovanja z nizko porabo energije. |

#### Kalibracija in senzorji

| Podukaz | Namen |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibracija z uporabo cilja odbojnosti. |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | Ukazi za vgrajeni senzor padajoče svetlobe. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Uporabi korekcijo vinjete za obstoječe slike. |

#### Več kamer (prehodne seje)

| Podukaz | Namen |
| --- | --- |
| `lattice multi-info` | Prikaži seznam vseh kamer s sinhronizacijskimi vlogami. |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | En sinhroniziran okvir iz vsake kamere. **Privzeto shrani vse vrste izvoza**, ko je priključen trajni niz; začasna rešitev brez niza je**le brez debayeringa** (za ostalo). |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | Pretakanje sinhroniziranih okvirjev (začasno). |
| `lattice multi-test [--count N]` | Preskus časovnega usklajevanja GPIO. |
| `lattice multi-detect [--line LINE] [--json]` | Samodejno zaznavanje ožičenja GPIO glavnega/podrejenega. |

#### Poravnava

| Podukaz | Namen |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — ter gumbi za detektor/usklajevalnik `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, gumbi RANSAC `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, kombinacija več okvirjev `[--averaging mean\|median\|inlier_weighted]`, geometrijske omejitve `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, prostorske omejitve `[--roi X0,Y0,X1,Y1] [--mask PATH]` ter prekrivanja po sužnjih `[--per-cam-override SN:KEY=VALUE]` (ponovljivo) | Izračun profila poravnave iz žive slike kamer. `--prefilter` je privzeto nastavljen na `gradient` (zemljevid robov; ustreza GUI/poravnalnik matrik — robovi ostanejo nespremenjeni med spektralnimi pasovi). `--matcher flann` se izplača pri več kot ~5000 značilnostih; `--averaging median` je odporen na en slab posnetek, `inlier_weighted` tehtanje po številu ujemanj; `--lock-scale` projicira na najbližjo rotacijo (brez merila), `--lock-axis` eno komponento translacije nastavi na nič; `--mask` se uporablja za vsako kamero (za nastavitve po posameznih kamerah uporabite `--per-cam-override`, npr. `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` zavrne shranjevanje kalibracije, katere RMS reprojekcije presega prag. |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | Zajame en poravnan večpasovni okvir. `--bit-depth` privzeto prilagodi kamero; `--no-crop` ohrani celoten okvir (izpolni s črno barvo); `--interpolation` (privzeto `linear`) ter `--border-mode`/`--border-value` (privzeto `constant`/0) nadzorujeta deformacijo na CPU — pot prek GPU je v vsakem primeru bilinearna. |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | Tokovno usklajeni večpasovni okvirji (isti regulatorji warpa kot pri `align-apply`). |
| `lattice align-info --profile PATH [--json]` | Prikaži podrobnosti profila. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Spremeni vrstni red slojev. |

#### Indeks / Matematika vegetacije

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

Celoten nabor zastavic: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI / NDRE / EVI / SAVI / GNDVI /…), `--formula EXPR`, `--channel SYM=BAND` (ponovljivo), `--capture-level raw|debayered|radiance|reflectance|unknown` (prepisuje raven zajema, zabeleženo v izvorni datoteki TIFF; privzeto: prebrano iz metapodatkov TIFF), `--output PATH`, `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Pri `--live` se uporabljajo tudi gumbi za poravnavo: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`.

> **`--channel` – pri simbolih se upošteva velika in mala črka.** Stran s simboli mora natančno ustrezati imenom kanalov v prednastavitvi (prednastavitve uporabljajo male črke, npr. NDVI = `red`,`nir` — preverite `--list-presets`), stran s pasom pa mora ustrezati imenu pasu v poravnanem nizu (ali pa mora biti indeks pasu, ki se šteje od 0, v načinu brez povezave). `--channel red=Red_660 --channel nir=NIR_850` deluje; `--channel RED=660` ne deluje in povzroči napako `channel_map missing entries`.

#### Trajne povezave (Smart-Prep, potek, enakovreden GUI)

Ti ukazi ohranjajo kamere odprte v sklopu strežniškega bazena med posameznimi klici »CLI«.

| Podukaz | Namen |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Dodaj eno kamero v sklop (enakamera, brez niza). |
| `lattice cam-disconnect [--serial SN] [--all]` | Sprostitev. |
| `lattice cam-list` | Seznam kamer v bazenu. |
| **`lattice array-connect`**|**Povezava trajnega sinhronizirano matrico (PRIPOROČENA vstopna točka).** Izvede celoten potek pametne priprave v grafičnem vmesniku. |
| `lattice array-disconnect [--array-id ID] [--all]` | Sproščanje matrice. |
| `lattice array-list` | Seznam povezanih matric. |
| `lattice array-status [--array-id ID]` | Število sličic na sekundo v realnem času, PTP, zadnja napaka. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | En sinhroniziran posnetek iz niza v realnem času — Enkratni / Neprekinjen / Intervalni / Najhitrejši. **Privzeto je `all`** (ena datoteka na ustrezno vrsto izvoza na kamero). Preskočene kamere (npr. RGB, izključene iz merjenja svetilnosti/odbojnosti) so navedene z `Skipped: SN:<serial> (<reason>)`; vrednost DAQ, uporabljena za odbojnost, je shranjena skupaj z njimi in navedena z `DAQ: <path>`. Glej [Načini zajemanja, snemalniki in ponovna obdelava brez povezave](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | Snemanje kombiniranega indeksnega pogleda v realnem času v video/GIF (za namene spremljanja; zahteva odprt kombinirani tok). |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | Serija surovih Bayerjevih posnetkov z visoko hitrostjo snemanja (za namene analize; ponovna obdelava v načinu brez povezave). |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Ponovna obdelava shranjene serije surovih posnetkov v kalibrirane videoposnetke. |

##### Možnosti `array-connect`

| Oznaka | Privzeto | Opis |
| --- | --- | --- |
| `--serials SN1,SN2,…` | samodejno odkrivanje vseh kamer LATTICE (potrebnih je ≥2) | Prva serijska številka je MASTER. Če je ta opcija izpuščena, se iskanje omeji na modele LATTICE (`TRI032*`) in poveže vse. |
| `--line {Line0,Line2,Line3}` | `Line2` | Sinhronizacijska linija GPIO. |
| `--target-fps F` | samodejno | Hitrost sprožanja glavne kamere. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | samodejno | Prepis izbire stopnje. |
| `--wire-ceiling-mbps MB_PER_S` | samodejno zaznano | **Trajna pasovna širina gostitelja v MB/s — vrednost, od katere je odvisna celotna dodelitev polja.** Zmanjšajte ga, ko niz poroča o poškodovanih okvirih GVSP: samodejna vrednost izhaja iz oglaševane hitrosti povezave mrežne kartice, ki precenjuje USB-adapterje, ozke PCIe-pasove in zasedene skupne strukture. Shranjeno je v bloku zajema niza projekta, zato se ob ponovnem odprtju / CLI / SDK ponovno vzpostavi. Glejte [Stanje matrike](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | auto | Strojno združevanje. |
| `--no-recommend` | off | Preskoči korak analize omrežja. |
| `--no-ptp` | off | Onemogoči PTP (časovni žigi med kamerami nato **niso** primerljivi). |

### Smart-AE / Smart-Capture

LATTICE-nizi izvajajo neprekinjeno avtomatsko ostrenje (AE) v ozadju takoj, ko soso povezani, vendar traja nekaj časa, da se novo usmerjena scena stabilizira. `array-capture --smart` je **pripravljena rešitev**: počaka, da se AE stabilizira na vseh kamerah v nizu, nato pa sproži zajem. Uporabite ga, ko med sejo spremenite sceno.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Pravilo stabilizacije je privzeto konzervativno: 5-sekundni časovni limit, 1,5-sekundno okno stabilnosti, toleranca razpona osvetlitve ±5 %. Nastavite prek »SDK« (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`), če potrebujete drugačno delovanje avtomatizacije.

### Ravni izvoza posnetkov (privzeta vrednost `all`)

Od te različice naprej `lattice capture`, `lattice multi-capture` in `lattice array-capture` **privzeto nastavljene na `--processing all`** – ena shranjena datoteka na vrsto izvoza, ki velja za vsako kamero, kar ustreza delovanju funkcije »Capture All« v grafičnem vmesniku. Ravni so:

| Raven | Izhod | Velja za |
| --- | --- | --- |
| `raw` | Enokanalni Bayer (črno-bele kamere: en pas) neposredno iz senzorja. | Vse kamere. |
| `debayered` | 3-kanalni BGR demosaik (črno-bele kamere: 1-kanalna siva lestvica). | Vse kamere. |
| `radiance` | float32 W/m²/sr/nm prek celotne radiometrične verige. | Samo multispektralne (M3C/M3M) — **preskočeno za kamere s RGB -filtrske kamere**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), pripravljeno za Pix4D-ready. | Samo multispektralno in **samo, če je vezan DAQ + je kamera kalibrirana**; v nasprotnem primeru se preskoči. |
| `preview` / `display` | Celotna veriga predogleda v grafičnem vmesniku (CCM + WB + gama v skladu s profilom kamere). `lattice capture` poimenuje to `preview`; `array-capture`/`multi-capture` uporabljata `display`. | Vse kamere. |

Če prenesete eno samo stopnjo, se shrani le ta (`--processing debayered`). Ko zahtevate `all`, se ravni, ki ne veljajo za dano kamero, preskočijo (in se poročajo), ne pa da se pojavi napaka — nepovezana ali nekalibrirana kamera še vedno dobi `raw` / `debayered` / `preview`.

Za vsak okvir odbojnosti se dejansko uporabljena vrednost navzdol usmerjenega merjenja DAQ zapiše v **`.daq`** ob sliki (tako da se zajem lahko kasneje ponovno obdela) in se poroča v vrstici `DAQ:`.

### Kako izgleda mapa z zajemi

Vsaka vrsta izvoza se shrani v svojo **lastno podmapomapi** pod `-o`, tako da se pri večstopenjskem zajemu vrste nikoli ne mešajo:

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>` je časovni žig zajema, `<serial>` pa serijska številka kamere, tako da si ena sinhronizirana skupina deli
časovni žig med kamerami. **Upoštevajte eno asimetrijo:** raven `display` je shranjena v mapi
z imenom `preview/`, medtem ko same datoteke ohranjajo `_display` – mapa in pripona se razlikujeta
samo za to raven. Neznane ravni se shranijo v mapo z njihovim imenom, in če se podmapo
ne moreustvariti, se datoteka zapiše v izhodni koren namesto da bi se izgubila.

**Ponovna obdelava mape »captures«:**usmerite `chloros-cli process` na**koren mape »captures«**
(`output/`). `process` običajno uvozi le mapo, ki jo imenujete, vendar če ta mapa ne vsebuje
slik in ima podmap, se samodejno spusti v globino – tako se podmappodmape in
koren `.daq` se zajamejo v enem zamahu. Vsaka raven zajema se uvozi kot ena slika, pri čemer
so druge ravni na voljo kot načini, namesto kot ena slika na raven.

Deluje tudi neposredno poimenovanje **podmape na ravni** (npr. `output/raw/`). Pri tem se korenska
mapa `.daq` izpusti, zato kopirajte ali usmerite odčitek DAQ ob strani, ko ponovno izpeljujete radiometrični
produkt iz `raw/` – sicer se časovni žig ne bo ujemal z ničemer.

**Obdelava se vedno začne pri `raw`.** Znotraj vsakega zajema je surovi okvir vir za obdelovalno verigo;
`debayered`, `radiance`, `reflectance` in `preview` se pojavijo kot načini prikaza, vendar se nikoli ne vrnejo
nazaj skozi cevovod. Ponovna obdelava izpeljanega izdelka bi ponovno uporabila vinjeto, CCM in
matematične izračune sevanja, ki so že vgrajeni v njegove piksle, zato Chloros raje zavrne kot da bi
dvojnoobdelave. Dve posledici, ki jih je vredno poznati:

- Renderji `index/` in `composite/` **nikoli** niso obdelani. So izhodi, ne zajemi —
  render LUT-a »NDVI« nima smiselne interpretacije sevanja.
- Mapa zajemov, izvožena **brez** datoteke `raw` (npr. `array-capture --processing reflectance`), nima
  veljavnega vira v delovnem toku. Ti zajemi se uvozijo in prikažejo normalno, vendar `process` jih preskoči
  in to tudi sporoči:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Če resnično potrebujete prenesti izpeljani izdelek – sejo vozlišča, zajeto z
  vklopljenim `demosaic`, ali starejšo mapo –, `--input-level {raw,debayered,processed}` prisili vstopno
  točko in prekliče preskok. Ta zastavica je namerno vgrajena varnostna rešitev; `auto` (privzeta nastavitev)
  nikoli ne obdela zajema, ki nima surovih podatkov.

### Preskočeni zajemi v nizih z mešanimi filtri

Ko v enem nizu mešate kamere z eno barvo (RGB) in multispektralne kamere, `array-capture --processing radiance` (ali `reflectance`) shrani multispektralne posnetke in **preskoči** kamere z eno barvo (RGB) — sevanje na posamezni Bayerjevi pikseli ni pomembno za širokopasovni senzor. CLI izpiše vsako shranjeno datoteko (z njeno stopnjo izvoza), vsak zapisani `.daq` in vsak preskok izrecno, zato število datotek ni presenetljivo:

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Oznake razlogov za preskok sledijo vzorcu `<level>-not-applicable-to-rgb-cam`. Odbojnost se lahko preskoči tudi z `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)` ter z `dls-uncalibrated-band-<nm>`, če pas večinoma leži zunaj radiometrično kalibriranega območja svetlobnega senzorja DAQ (~374–974 nm) — med dostavljenimi artikli je to le F988, katerega podprta pot je delovni tok z reflektančnim panelom.

Uporabite `--processing debayered` (ali `display`), da vključite vse kamere ne glede na vrsto filtra, ali privzeto `all`, da v enem zamahu pridobite vse ustrezne ravni za vsako kamero.

---

## Načini zajemanja, snemalniki in ponovna obdelava brez povezave

Vsi delujejo na **trajnem nizu** (najprej zaženite `array-connect`). Odražajo ploščo zajema v grafičnem vmesniku.

### Načini `array-capture`

`array-capture` je en sam ukaz s štirimi načini zaklopa ter nizom stikal za izvoz:

| Način | Oznaka | Delovanje |
| --- | --- | --- |
| **Enkratno** *(privzeto)* | (ni) | Ena sinhronizirana skupina zajemov, nato izhod. |
| **Neprekinjeno** | `--continuous` | Zaporedni prehodi, dokler se ne izvede `Ctrl+C`, `--count N` ali `--duration S`. |
| **Interval** | `--interval S` | En prehod vsakih `S` sekund (merjeno od začetka vsakega prehoda), iste meje. |
| **Najhitrejši** | `--fastest` | Samo surovi podatki + dodeljena vrednost DAQ + sestavljeni indeks; preskoči izračune svetilnosti/odbojnosti/prikaza, da se slika prikaže hitreje. Vključuje `--processing raw --force-daq`. Shranjene `.daq` kasneje ponovno obdelajte v kalibrirane izdelke. |

Preklopniki izvoza (kombinirajte s katerim koli načinom; vsi uporabljajo grafični vmesnik/končno točko SDK):

| Oznaka | Učinek |
| --- | --- |
| `--processing LEVEL` | Enotna raven izvoza ali `all` (privzeto). |
| `--levels L1,L2,…` | Izrecna podmnožica vrst izvoza (npr. `raw,radiance,reflectance`); **prevladuje nad `--processing`**. |
| `--aligned` / `--no-aligned` | Izvoz vsakega člana, ki ni v surovi obliki, se prilagodi [profilu poravnave](#alignment) niza (so-registriran). Surovi podatki ostanejo neizravnani, vendar nosijo transformacijo v metapodatkih. Če niz nima profila, se vrne na neizravnano stanje (z opozorilom). |
| `--index` / `--no-index` | Shrani / preskoči prekrivni sloj indeksa vegetacije za vsako kamero, kjer je ta konfiguriran. Privzeto: ga prikaži. |
| `--force-daq` | Shrani dodeljeno odčitano vrednost DAQ/DLS kot stranski datotek `.daq`, tudi če nobena izbrana raven tega ne potrebuje (npr. zajem samo surovih podatkov), tako da se lahko posnetki naknadno ponovno obdelajo v odbojnost/indeks offline. |
| `--smart` | Pred sprožitvijo počakaj, da se AE ustalijo na vseh kamerah (glej [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | Kompresija pikslov »TIFF«. `deflate` (privzeto) = zlib L1 brez izgube + horizontalni prediktor, ~4,1 MB na posnetek v polni ločljivosti; `none` = nekomprimirano, ~5× hitrejše zapisovanje pri ~6,3 MB na posnetek — uporabite za največjo trajno hitrost, če to dopušča disk. Obe možnosti sta brez izgube in se ob uvozu berejo enako. |

> **TIFF z enim zapisom + model s trajno hitrostjo.**Posnetki se zapišejo v**enem**prehodu datoteke TIFF, ki vsebuje piksle + XMP + IFD0 Make/Model (izmerjeno na polni ločljivosti Mono12: 36 ms stisnjeno / 6,5 ms nestisnjeno, v primerjavi s ~148 ms za staro ponovno pisanje z-ExifTool); edino preostalo delo ExifTool (dopolnjevanje EXIF pod-IFD) poteka v asinhronem ozadju, posnetek pa je končan in pripravljen za uvoz, tudi če se to nikoli ne izvede. Upoštevajte, da stiskanje DEFLATE zadržuje GIL »Python«, zato se stisnjena zapisovanja**ne**izvajajo vzporedno v niti za zapisovanje posameznih kamer — neprekinjeno zajemanje v polni ločljivosti z 8 kamerami pri hitrosti senzorja (~10,4 sličic na sekundo) zahteva `--compression none`**in** disk razreda NVMe (~500 MB/s neprekinjenega zapisovanja). Isti parameter je na `POST /api/camera/array/capture` na voljo kot `compression`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — video/GIF s kombiniranim indeksom (za namene spremljanja)

Snema vse, kar prikazuje **pogled v živo s kombiniranim indeksom**, na `.avi` (in po želji tudi na `.gif`). Ker črpa iz živega kompozitnega signala, mora biti kombinirani tok odprt (npr. matrika se predogleduje v grafičnem vmesniku), da se posnamejo posamezni okvirji. Napredek preverja vsake 2 s in se ustavi na `--duration`, `Ctrl+C` ali ko se snemalnik samodejno zaključi.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| Oznaka | Privzeto | Opis |
| --- | --- | --- |
| `--array-id ID` | samo niz | Ciljni niz (izpustite, če je priključen le en). |
| `-o, --output DIR` | `output` | Izhodni imenik (lokalno v ozadju). |
| `--fps F` | `10` | Hitrost snemanja. |
| `--duration S` | do Ctrl+C | Samodejni zaustavitev po `S` sekundah. |
| `--gif` | izklopljeno | Zapiši tudi animirani GIF. |
| `--gif-only` | izklopljeno | Shrani samo GIF (brez `.avi`). |

### `array-burst` — surovi Bayerjev niz posnetkov z visoko hitrostjo (za analizo)

Neposredno bere iz sinhroniziranega skupinskega pomnilnika zanke zajemanja — **ni potrebna kalibracijska veriga, ni potrebno orodje exiftool, ni potreben predogled v živo** — tako da deluje pripolni hitrosti zajema. Zapiše surove slike + manifest za vsako sliko + en `.daq` za vsak posamezen DLS-odčitek pod `<output>/bursts/<base>/`. Ponovno obdelajte offline (naslednji ukaz) ali posredujte `--build`, da se to opravi takoj ob ustavitvi.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Oznaka | Privzeto | Opis |
| --- | --- | --- |
| `--array-id ID` | samo matrika | Ciljna matrika. |
| `-o, --output DIR` | `output` | Izhodni imenik (izhodni podatki se shranijo v `<DIR>/bursts/<base>/`). |
| `--duration S` | dokler ne pritisnete Ctrl+C | Samodejni zaustavitev po `S` sekundah. |
| `--max-frames N` | neomejeno | Samodejni zaustavitev po `N` surovih okvirjih. |
| `--build` | izklopljeno | Po zaustavitvi takoj ponovno obdelaj serijo (enako kot `array-build-video`). |
| `--products …` | `combined:index` | Z `--build`: katere videoposnetke sestaviti (glej spodaj). |
| `--fps F` | `10` | Pri `--build`: število sličic na sekundo (fps) izhodnega videa. |
| `--save-tiffs` | izklopljeno | Z `--build`: shrani tudi kalibrirane TIFF-datoteke za vsak posamezni okvir. |
| `--gif` | izklopljeno | Z `--build`: zapiše tudi animirane GIF-datoteke. |

### `array-build-video` — ponovna obdelava shranjene serije posnetkov v načinu brez povezave

Vsak surovi posnetek časovno uskladi z najbližjo shranjeno vrednostjo `.daq` in ga preusmeri skozi **isto verigo svetilnosti / odbojnosti / indeksa kot uvozni proces**, pri čemer se renderira en ali več videoposnetkov.

`--products` je seznam elementov `kind:level`, ločenih z vejico, kjer `kind` ∈ `per_cam` | `combined` ter `level` ∈ `radiance` | `reflectance` | `index`. Samostojni `level` (brez `kind:`) je privzeto nastavljen na `per_cam`. Privzeta vrednost je `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Oznaka | Privzeto | Opis |
| --- | --- | --- |
| `--burst-dir DIR` | (obvezno) | Pot do mape za serijske posnetke (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | Seznam `kind:level`, kot zgoraj. |
| `--fps F` | `10` | Število sličic na sekundo (fps) izhodnega videa. |
| `--save-tiffs` | izklopljeno | Poleg videoposnetkov shranite tudi kalibrirane TIFF-datoteke za vsako posamezno sličico. |
| `--gif` | izklopljeno | Shrani tudi animirane GIF-datoteke. |

> **Izberite ustrezen snemalnik.** `array-record` je *za namene spremljanja* — zajema kompozitni signal v živo, kot-prikazano in potrebuje odprt tok. `array-burst` → `array-build-video` je *primeren za analizo* — shranjuje surove podatke senzorja s polno hitrostjo in naknadno rekonstruira kalibrirane videoposnetke sevanja/odbojnosti/indeksa, pri čemer ni potreben vpogled v živo.

### Mono (M3M) enopasovne kamere

Serija **M3M**je mono različica Bayerjeve serije**M3C**: en ozkopasovni interferenčni filter na kamero (`M3M-<lens>-F<wavelength>`, npr. `M3M-L87-F685`), zato senzor zagotavlja**en sam pas v sivih odtenkih** brez Bayerjevega mozaika. Ni treba izvajati demozajevanja, ni treba ločevati medkanalnega prekrivanja in ni treba nastavljati beline — celoten barvni proces od zajema do prikaza (RGB) preprosto ne velja.

Kaj to pomeni za CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**zaznavajo monokromatsko kamero in**preskočijo z enovrstičnim sporočilom**, namesto da bi vsiljevali nesmiselne nastavitve. V isti seji še vedno delujejo normalno z RGB /Bayer M3C kamero.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** še vedno delujejo — radiacija in odbojnost sta *na pas* radiometrični zemljevidi in so za en pas popolnoma dobro definirani. Mono posnetki vsebujejo **identitetno** matriko odziva senzorja (brez 3×3 ločevanja), zato se kalibracijski izračun izvede brez sprememb.
- **Ena sama mono kamera ne more ustvariti vegetacijskega indeksa.**NDVI / NDRE /itd. potrebujejo vsaj dva pasova (npr. Red + NIR). Da bi iz mono opreme pridobili indeks, usmerite**več** M3M kamer na različne valovne dolžine, jih poravnajte v en večpasovni niz in izračunajte indeks *za to*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` simboli morajo **natančno** ujemati z imeni kanalov v prednastavitvi (upoštevajte velike in male črke; NDVI so z malimi črkami `red`,`nir` — glejte `--list-presets`), ime pasu pa mora ustrezati imenu pasu v poravnanem sklopu (v načinu brez povezave se sprejema indekse pasov, ki se začnejo z 0, npr. `--channel red=0 --channel nir=1`).

Razlikovalni znak v celotnem sklopu je token `M3M` v nizu modela (nikoli se ne pojavi v nizu `M3C`), ki se v grafičnem vmesniku (GUI)/SDKju prikaže kot `is_mono`.

---

## Nastavitev in optimizacija omrežne kartice gostitelja (nizi LATTICE)

Kamere LATTICE prenašajo GVSP prek ethernetnega adapterja gostitelja, zato sta pri nizih z več kamerami **gonilnik**adapterja in**velikost sprejemnega**velikost sprejemnega obroča** prav tako pomembna kot hitrost povezave. Napačne nastavitve se prikažejo kot vrata `FRAMES WILL DROP` / `Reduce ROI to enable` v oknu »Nastavitve niza« (in kot `lattice network-analysis` / `analyze_array_network()` v »SDK«), tudi če so same kamere v dobrem stanju.

### USB 10GbE adapterji — Realtek RTL8157 („Realtek USB 10GbE Family Controller“)

| Element | Zahtevana vrednost | Zakaj je pomembno |
| --- | --- | --- |
| **Različica gonilnika**|**≥ v10.67 (januar 2026)**, INF `rtump64x64sta.inf` | Stari gonilnik iz leta**2016**(v10.65, `rtump64x64.inf`) nepravilno obravnava izklop in napake s**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**ob izklopu/ponovnem zagonu/prehodu v stanje mirovanja. Prehod se zatakne (časovna omejitev ~5 min), uporabnik prisilno izklopi računalnik, ponavljajoče se nepravilno izklapljanje pa**poškoduje WMI-repozitorij**(PowerShell/orodja začnejo odpovedovati z napako `Invalid class`) in**zablokirajo USB-sklad** ob naslednjem zagonu (mrežna kartica se ne vklopi; USB-naprave prenehajo z navajanjem). Posodobite s spletne strani realtek.com (ali pri proizvajalcu ključa), preden se zanesete na pravilne ponovne zagone. |
| **Prejemni pomnilniki**— ključna beseda `ReceiveBufferLen` |**256**(največja vrednost gonilnika) | Prejemni obroč mrežne kartice. Privzeta vrednost gonilnika**32**pušča na voljo le ~0,26 MB uporabnega obroča — kar je premalo za serijsko snemanje z več kamerami — zato nadzorna plošča poroča o napaki `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` in blokira povezave. Pri vrednosti**256**je obroč velik (**~13,5 MB, izmerjeno na laboratorijskem gostitelju z 10GbE**), kar omogoča sprejemnemu cevovodu resnično rezervo za serijske prenose GVSP z več kamerami. (Ali se določena konfiguracija dejansko *poveže*, se odloči na podlagi dveh preverjanj — preverjanja sprejemljivosti, ki upošteva **izpraznjevanje**, in preverjanja**skupne prekomerne zasedenosti** — ne pa na podlagi surovega primerjanja serijskega prenosa z obročem; glej [Model fps in porivov za Array](#array-fps--burst-model).) |
| **Prejemni URB-ji**— ključna beseda `PendingReceives` |**64** (maks.) | Bloki zahtevkov USB v prenosu; povečajte skupaj s prejemnimi pomnilniki za absorpcijo porivov. |
| **Jumbo Frame** — ključna beseda `*JumboPacket` | **9014** | Potrebno za GVSP-pakete velikosti 9000 bajtov (6× manj paketov/okvir kot pri 1500). |

> ⚠️ **Posodobitev gonilnika mrežne kartice ponastavi te napredne lastnosti na privzete vrednosti.**Po posodobitvi ali zamenjavi gonilnika adapterja**ponovno uporabite** `ReceiveBufferLen=256` in `PendingReceives=64`, sicer se bo plošča ponovno blokirala, čeprav se »v strojni opremi ni nič spremenilo«. To je glavni vzrok, da se naprava, ki je prej delovala, nenadoma ne more več povezati.

Uporabite iz **privilegiranega** PowerShell-a (vstavite ime svojega adapterja, npr. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` zajema USB 10GbE adapterje.** Sedaj zazna tip adapterja in nastavi pravilno ključno besedo za sprejemni obroč: `*ReceiveBuffers`→2048 za PCIe mrežne kartice (Intel I219 itd.) ali `ReceiveBufferLen`→256 + `PendingReceives`→64 za Realtek **USB** 10GbE krmilnik (ki ne razkriva `*ReceiveBuffers`). Ciljne vrednosti so omejene na največjo vrednost, ki jo poroča posamezni gonilnik (`NumericParameterMaxValue`), zato se nikoli ne zapiše vrednost zunaj dovoljenega območja. Zagnajte ga v terminalu z **povečanimi pravicami**; kot pri vseh nastavitvah, ki temeljijo na registru, sprememba začne veljati šele po ponovnem zagonu adapterja ali ponovnem zagonu sistema. Zgornji ročni ukazi `Set-NetAdapterAdvancedProperty` ostajajo dobra alternativa — veljajo takoj (ponovno povežejo adapter) brez ponovnega zagona.

### Osnove omrežja (vse povezave LATTICE)

- **Naslavljanje:** link-local `169.254.0.0/16` (GigE Vision LLA). Gostitelj uporablja statični naslov `169.254.x.x/16`; kamere in DAQ-E si sami dodelijo naslove v istem obsegu. DHCP/prehod ni potreben.
- **Velikost paketa:**prednostno jumbo (9000), vendar naj jo avtomatsko preizkušanje poišče — pri vsaki povezavi ponovno izmeri in prek GVSP-preizkusa že presega 1500-bajtno ICMP-omejitev kamere, zato se nastavi na jumbo, kjerkoli jo kabel dejansko prenese. Fiksno nastavite `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` le, če veste več kot preizkus, in dajte prednost nastavitvi na ukaz pred trajno: fiksna nastavitev preskoči preizkus, zato če pot dejansko ne more prenesti 9000,**vsak** zajem poteče v časovno omejitev z `SC_ERR_TIMEOUT -1011` (glej [Okoljske spremenljivke](#environment-variables)).
- **Obseg RX obroča se prilagaja z `ReceiveBufferLen`:**pri privzeti vrednosti `32` je uporaben obroč ~0,26 MB (premajhen za kakršno koli serijo posnetkov z več kamerami); pri največji vrednosti `256` je velik (~13,5 MB, izmerjeno na laboratorijskem gostitelju 10GbE), kar zagotavlja resnično rezervo. Ali se konfiguracija poveže, se nato odloči na podlagi preverjanja sprejemljivosti, ki upošteva izčrpavanje,**in** na podlagi spodnjega preverjanja agregatne prekomerne zasedenosti — ne pa na podlagi surovega primerjanja serijskega snemanja z obročem.

### Model fps in serijskega snemanja v Arrayu

Kako razumeti okno »Nastavitve Array« (in `lattice analyze-array` / `analyze_array_network` v SDK):

- **Burst se sešteva po posameznih kamerah v dejanskem pikselnem formatu vsake kamere.**Mono**M3M**kamere prenašajo**Mono12 (2 B/px)**;**M3C**kamere z Bayerjevim filtrom prenašajo 8- ali 12-bitne podatke (TRI032S tiho oddaja BayerRG12, tudi če je zahtevan BayerRG8). Tako je okvir v polni ločljivosti s 4 kamerami**~12,6 MB, če so vse 8-bitne, z tremi 12-bitnimi mono kamerami pa ~25 MB**. Projekcija določi format vsake kamere na podlagi njenega modela (identitetni predpomnilnik), zato serija posnetkov ustreza dejanskemu prenosu po kablu — ne gre za enotno predpostavko o formatu BayerRG8.
- **USB-ethernetni adapter je omejen na 200 MB/s ne glede na navedbe na tipski ploščici.** Tabela učinkovitosti, ki pretvori hitrost povezave v trajno vrednost, izhaja iz standarda PCIe; USB-mrežna kartica oglašuje svojo *Ethernet* hitrost povezave, vendar je omejen z USB-vodilom in njegovim gonilnikom. USB 10GbE ključ je nekoč dosegal ~1063 MB/s »trajne« hitrosti — številka, ki ni bila nikoli preverjena —, pri čemer je posledično napačno uravnavanje poškodovalo 6–18 % okvirjev, kljub temu da je še vedno poročal o ustrezni ciljni vrednosti fps. Mrežne kartice, priključene prek USB-ja, so zdaj absolutno omejene na **200 MB/s** (omejitev predstavlja vodilo, zato se ne prilagaja navedbam na nalepki; USB 1 GbE adapter dosega ~80 MB/s in to omejitev ne zadeva). `wire_ceiling_source` v zapisu zmogljivosti to navaja z besedami, `nic_is_usb` pa to označi. V obeh primerih lahko to prepisate z `--wire-ceiling-mbps`.
- **Dopustnost upošteva odtok, ne pa celotnega izbruha v primerjavi z obročem.** Sočasni izbruh mora ustrezati le *prehodnemu zaostanku* = `max(0, Σ per-cam arrival − host drain) × emit_window`, ne pa celotnemu izbruhu. Na hitrem gostitelju / počasni-kamero (**PCIe**10G gostitelj + 4× 1 GbE kamere: prihod ≈ 320 MB/s, odvod ≈ 1063 MB/s) gostitelj prazni hitreje, kot se kamere polnijo, zaostanek ≈ 0, zato**dovoljuje**oddajanje v polni ločljivosti, čeprav 25 MB izbruh presega 13,5 MB obroč. Če iste štiri kamere priključite na**USB**10GbE adapter, je odvod 200 MB/s, ne 1063 — prihod ga prehiti, izguba pa se kaže kot poškodovani okvirji in ne kot nižja hitrost snemanja. Na gostitelju 1 GbE spodnja meja DLThr kamer 31,25 MB/s povzroči, da prihod prehiti odvod → pravilno**blokira** (za *te* vrste blokade zmanjšajte ROI ali uporabite združevanje ≥ 2). Dovoljenje je eno od **dveh** povezovalnih vrat – drugo je spodaj navedeni pregled skupne prekomerne naročnine.
- **Predvidena hitrost sličic na sekundo (fps) je konzervativna zgornja meja za serijsko pridobivanje.**Zank za pridobivanje na gostitelju trenutno**serijsko**(~eno okno oddaje na kamero), zato je cikel omejen z `max(readout+emit, N × emit)`, pri čemer je oddaja na kamero omejena na**povezavo za dostop**kamere (1 GbE ≈ 80 MB/s), ne pa na gostiteljev uplink. Za 4-kamerni v polni ločljivosti s 12-bitno matriko znaša**~2,8 fps**, kar ustreza izmerjenim ~2,7–3,0. fps je namerno**neodvisna od osvetlitve**, zato se v temnih prizorih dejanska vrednost lahko rahlo spusti pod zgornjo mejo, ko se osvetlitev podaljša. Serijsko pridobivanje podatkov je dejanski omejevalnik sličic na sekundo; vzporedno obdelovanje bi zgornjo mejo dvignilo proti hitrosti enega izhoda.
- **Skupna prekomerna zasedenost je trdna ovira za povezavo.**Dodelitev pasovne širine na kamero je omejena na**8 MB/s**(`ARRAY_PER_CAM_FLOOR_BPS`), tako da lahko, ko se spodnja meja doseže, skupno povpraševanje (`per_cam × N`) preseže**zgornjo mejo omrežja, varno pred kolizijami**(`sustained × sim_emit_factor`). Praktične zgornje meje pri polni ločljivosti na 1 GbE:**6 kamer pri 1500 MTU, 9 z jumbo okvirji**. Ta zgornja meja je odvisna izključno od zmogljivosti omrežja in spodnje meje – je**neodvisna od velikosti okvirja**, zato**združevanje in manjši ROI NE pomagata** (zmanjšujeta število bajtov na *okvir*, ne pa števila bajtov na *sekundo*, ki jih določa GevSCPD); edini ukrepi so manjše število kamer, uporaba jumbo okvirjev od začetka do konca ali hitrejša mrežna kartica. Simptom bi bila izguba paketov GVSP, ne pa postopno zmanjšanje števila sličic na sekundo (fps), zato `analyze-array` nastavi vrednosti dosegljivih sličic na sekundo na nič in izpiše `**OVER-SUBSCRIBED**` ter `array-connect` s fiksno ločljivostjo **zavrne povezavo** (v nasprotnem primeru »walk-down« okvirje združi v manjše enote, kar te vrste blokade prav tako ne odpravi). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` znižuje stopnjo zavrnitve na glasno opozorilo za delo na testni opremi — glej [Okoljske spremenljivke](#environment-variables).

### Stanje niza — kateri podsistem izgublja okvirje

`GET /api/camera/array/<array_id>/capability` povezanega niza vsebuje aktivni
blok `health`, ki se ponovno ocenjuje v drsečem **10-sekundnem** oknu. Izgubo okvirjev razdeljuje
razdeljuje na dva vzroka, ki zahtevata nasprotna popravila, namesto da bi poročal o eni »nepopolni«
stopnji, ki ne navaja nobenega od njiju:

| Polje | Kaj pomeni | Kateri podsistem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (na serijsko številko) | Okvir je **prispel, a je bil strukturno poškodovan**— izguba paketov GVSP. |**Omrežje**: zmogljivost povezave, hitrost prenosa, obroč NIC RX, MTU |
| `never_arrived_rate_pct` (po serijski številki) | Okvir **sploh ni prišel**— kamera ni sprožila ali pa iz nje ni nič prišlo. |**Sprožilec / sinhronizacija**: kabel M8, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Najslabša stopnja kamere za vsako od teh. | — |
| `per_cam_rate_pct` | Skupna stopnja nepopolnosti na kamero (oba vzroka skupaj). | — |
| `stable_for_seconds` | Kako dolgo je vsaka kamera ostala pod 0,01 %. | — |

Nad 5 % backend zabeleži vrstico `[array-health <id>] WARN`, v kateri navede razlog za prehod — ob
prvem prekoračitvi, ob spremembi stopnje resnosti, enkrat na minuto, dokler traja, in enkrat, ko
se stanje popravi. Okvarjena polovica izpiše `[gvsp-corrupt <SN>]` ob prvem pojavu na kamero in
razlogu, nato pa zbirni zapis vsake 60 s. Vsaka ocena se še vedno shrani v dnevniški datoteki strežnika;
številčniki se pomikajo pri vsakem pomnilniku, ne glede na od tega, kaj se natisne.

Isti zapis poroča o številu, na katerem visi celotna dodelitev:

| Polje | Kaj pomeni |
| --- | --- |
| `wire_ceiling_mbps` | Trenutni trajni proračun gostitelja za pasovno širino, MB/s. |
| `wire_ceiling_source` | Od kod izvira ta številka, v besedah — npr. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` ali `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, ko ga nastavi `--wire-ceiling-mbps` (ali polje **Wire Budget** ) to nastavi. |
| `nic_is_usb` | `true` za USB-ethernetni adapter — glej zgornjo omejitev 200 MB/s. |

**Razlaga:** vrednost `gvsp_corrupt_rate_pct`, ki ni enaka nič, pri vrednosti `never_arrived_rate_pct` = 0
pomeni, da sta sprožitev in sinhronizacija kabla brezhibni, 100 % izgube pa je na omrežni
poti — znižajte vrednost `--wire-ceiling-mbps` in ponovno vzpostavite povezavo. Nasprotni vzorec kaže na
sinhronizacijski kabel ali sprožilno linijo.

> **`--target-fps` ni vzvod za poškodovane okvirje.** Hitrost GevSCPD se zapiše
> enkrat ob vzpostavitvi povezave, zato znižanje hitrosti sprožilca spremeni delovni cikel, ne pa
> hitrosti sočasnega oddajanja paketov. Izmerjeno 5-kratno zmanjšanje zahtev ni prineslo izboljšanja;
> znižanje zgornje meje žice s 240 na 200 MB/s je pri isti opremi zmanjšalo delež4 %
> poškodovanih na 0,00 %.

> **Samodejno zmanjšanje med prenosom ni na voljo v firmware-u TRI032S.** Delujoči niz
> tega ne more sam popraviti; prekinite povezavo in se ponovno povežite, da lahko izbirnik časa povezave
> ponovno načrtuje z novo zgornjo mejo.

### Simptom → rešitev

| Simptom (Nastavitve niza / povezava / `analyze_array_network`) | Vzrok | Rešitev |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` se ponastavi na 32 (običajno po posodobitvi gonilnika) | Nastavite `ReceiveBufferLen`→256, `PendingReceives`→64; ponovno odprite nadzorno ploščo (ponovno zaženite strežnik, če je v predpomnilniku shranil staro velikost obroča) |
| Zagon/izklop se zatakne; kasneje `Invalid class` napake WMI, mrežna kartica se ne vklopi, manjkajo USB-ključi | Stari gonilnik Realtek USB 10GbE iz leta 2016 → BSOD `0x9F` → prisilno izklop- | Posodobite gonilnik adapterja na različico ≥ v10.67 (2026), nato ponovno uporabite zgornje nastavitve sprejemnega obroča |
| Povezava uspe, vendar vrne ločljivost, ki je nižja od izvorne | Smart-prep samodejno skrči okvir, da se prilega kablu | Nadgradite povezavo / sprejmite skrčitev / `--force-tier slip-emit-and-capture` |
| Niz poroča o ustrezni ciljni hitrosti sličic na sekundo (fps), vendar prenaša le del te vrednosti; `health.gvsp_corrupt_rate_pct` ni enak nič, `never_arrived_rate_pct` 0 | Predvideni pasovni proračun gostitelja precenjuje dejansko zmogljivost (tipično pri USB-Ethernetnem adapterju, ozkem PCIe-pasu ali skupni strukturi) | Ponovno se povežite z nižjo vrednostjo `--wire-ceiling-mbps` in ponovno preverite blok delovanja. **Ne** `--target-fps` — hitrost GevSCPD je ob povezavi fiksna |
| Kamere manjkajo v objavljenih skupinah; `health.never_arrived_rate_pct` ni enak nič, `gvsp_corrupt_rate_pct` 0 | Pot sprožilca/sinhronizacije — kamere se ne sprožajo, ni težava z omrežjem | Preverite sinhronizacijski kabel M8 in `--line`; preverite, ali so vsi člani vklopljeni (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` v `analyze-array`, ali zavrnjena povezava z določeno ločljivostjo (`array over-subscribes the wire`) | Skupna zahteva na kamero (minimalno 8 MB/s × N kamer) presega zgornjo mejo žice, varno pred kolizijami — 6 kamer v polni ločljivosti na 1 GbE pri 1500 MTU, 9 z jumbo okvirji | Manj kamer, jumbo okvirji od začetka do konca ali hitrejša mrežna kartica. **ROI/binning NE bo pomagal** (zgornja meja je neodvisna od velikosti okvirja). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` prevlada v testnem okolju (sprejme izgubo paketov) |

---

## `chloros-cli daq`

ukazi za spektralni senzor. Dve razreda:
- **`pool-*`**— »thin« HTTP odjemalci, ki senzor upravljajo prek trajnega sklopa backenda.**To je podprta pot in edina, ki je prisotna v priloženem CLI.** Backend je lastnik prenosa, zato si grafični vmesnik (GUI), skripti CLI in SDK delijo en aktivni ročaj, namesto da bi se borili za serijsko vrata.
- **Vse ostalo**(`test`, `record`, `live`, `stream`, `connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`, `serve`, `ws`, `udp`, `mqtt`, `reflectance`, `login`, `logout`, `status`) — neposreden dostop do strojne opreme, ki je zaradi popolnosti dokumentiran spodaj. Za te je potreben paket `daq` Python, ki**ni vključen v noben dostavljen artefakt**: kompilirani CLI ga izključuje (`scripts/Build-CLI.ps1` nastavi `--nofollow-import-to=daq`, transporti `pyserial` / `bleak` / `zeroconf` z njim), prav tako pa ga ne vsebuje niti paket PyPI SDK. Delujejo le iz izpisanega izvornega koda, zato jih obravnavajte kot MAPIR -notranjo razvojno pot in ne kot nekaj, kar bi uporabiti.
- **`discover` / `list`** pa se nahajata nekje vmes: gre za ukaze za neposredno upravljanje strojne opreme iz izvoza izvorne kode, vendar se v končni različici zatečejo k `pool-discover`, skeniranje pa opravi ozadje. Tako skeniranje deluje povsod — kar je pomembno, saj je to edini način za pridobitev BLE MAC-a naprave DAQ-M.

> **`chloros-cli daq --help`** (in `-h` / `help`) navaja podukaze `pool-*` — pomoč je namerno usmerjena na odjemalca bazena, tako da odraža ukaze, ki se dejansko izvajajo. Če v končni različici podukaz za neposredno delo s strojno opremo v dostavljeni različici, se program zapre z izrecno napako, ki navaja manjkajoči paket in vas usmerja nazaj na `pool-*`; nič ne odpove tiho. (`discover` / `list` sta izjema — preusmerita na `pool-discover` in preprosto delujeta.)
>
> **Vse, kar stranka potrebuje, je dostopno prek `pool-*`** — povezovanje, pretakanje, snemanje kalibriranih datotek `.daq`in zamenjajte profile kondenzatorjev. DAQ je mogoče upravljati tudi iz Python s pomočjo `chloros_sdk.connect_daq_sensor()`, ki uporablja isto združeno pot.

### Potek prvega povezovanja senzorja DAQ

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### `pool-*`eferenca 

| Podukaz | Namen |
| --- | --- |
| `daq pool-connect` (smart-detect) | Odpre senzor v skladišču na strežniku. |
| `daq pool-connect --port PORT` | DAQ-U na določenem serijskem vmesniku. |
| `daq pool-connect --ble` | DAQ-M prek BLE, samodejno iskanje MAC. |
| `daq pool-connect --mac MAC` | DAQ-M prek BLE na znanem naslovu MAC (vključuje `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E prek omrežja Ethernet na znanem gostitelju. |
| `daq pool-connect --eth` | DAQ-E prek omrežja Ethernet, gostitelj samodejno odkrit (mDNS + ARP kot rezerva; deluje iz praznega predpomnilnika ARP na Windows in Linux). |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | Nastavi integracijsko okno / stanje AE. |
| `daq pool-connect --no-stream` | Vzpostavi povezavo, vendar še ne začni s prenosom podatkov (nadaljuj s `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Profil korekcije omejitve. Privzeta vrednost na strežniški strani je `sunshine_cosine`. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Preglej vsak transport za senzorje, s katerimi se lahko povežeš, brez vzpostavitve povezave. **Tako poiščete BLE MAC naslov DAQ-M naprave.** `daq discover` / `daq list` se v serijskih različicah samodejno usmerita sem. Senzorji, ki so že odprti v zbirki, niso navedeni — povezan DAQ-M preneha z oglaševanjem — zato za te uporabite `pool-list`. |
| `daq pool-list` | Prikaži vse senzorje v bazenu na strežniku. |
| `daq pool-disconnect --sensor-id ID [--all]` | Sprosti. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Najnovejši okvirji spektra N. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Nadaljuj / zaustavi pretakanje. |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | Začni / ustavi snemanje .daq. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Zamenjaj profil korekcije kapacitivnosti med izvajanjem. |

### Neposredni podukazi za strojno opremo (samo v izvorni kodi — niso vključeni v končne zgradbe)

> Navedeni so zaradi popolnosti. Za njihovo delovanje je potreben paket `daq` Python ter `pyserial` / `bleak` / `zeroconf`, od katerih nobena ni vključena v kompilirani CLI ali na PyPI SDK — delujejo le iz izvorne kode MAPIR. **Če uporabljate izdano različico Chloros, namesto tega uporabite zgornje ukaze `pool-*`**; ti zajemajo povezovanje, pretakanje, snemanje in izbiro omejitve.

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

Odprite, povežite se z in upravljajte shranjeni projekt Chloros (mapa z `cameras.json` + `sensors.json` + `project.json`). Vse poteka prek ozadja, zato grafični vmesnik in CLI prikazujeta enako stanje strojne opreme.

### Referenca podukazov

| Podukaz | Namen |
| --- | --- |
| `project open PATH` | Izpiše seznam naprav v projektu (kamere, matrice, senzorji). |
| `project devices PATH [--reconnect]` | Prikaže seznam ali ponovno izvede iskanje. |
| `project connect PATH [--cameras-only] [--sensors-only]` | Poveže vsako shranjeno kamero / niz / senzor. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Enkratni posnetek iz imenovane kamere ali niza. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Serija N posnetkov iz imenovane kamere ali niza (`-n/--count` privzeto 5; `-i/--interval` sekunde med posnetki, privzeto 0). Serije posnetkov iz niza odstranijo podvojene ponavljajoče se sinhronizirane skupine (nadzor zastarelosti), tako da niz z delnim ciklom ne more vrniti N kopij enega posnetka; izpiše rezultate za vsako iteracijo. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | Prenosna disk prek naloge v ozadju. `--poll-interval` = število sekund med poizvedbami `/stats` (privzeto 2,0). |
| `project sensor read PATH NAME [--json]` | Najnovejši spektralni okvir. |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | Snemanje .daq. |
| `project run PATH RECIPE.yaml` | Izvedba recepta za zajem YAML/JSON recept za zajem. `--dry-run` preveri brez izvedbe. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Izračunaj poravnavo za niz — glej [tabelo zastavic spodaj](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Izpiši trenutni profil poravnave. |
| `project align clear PATH NAME` | Izbriši profil iz predpomnilnika. |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | Premakni transformacijo enega od podrejenih procesov. |
| `project align export PATH NAME --to FILE` | Shrani profil v datoteko JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Naloži shranjen profil. |

#### `project align calibrate` Možnosti

| Oznaka | Privzeto | Opis |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Metoda poravnave. **Te pisave se razlikujejo od `lattice align-calibrate`**, ki uporablja kratke oblike `orb` / `akaze` / `phase`; ti dve ukazi nista zamenljivi pri tem zastavku. |
| `--model {translation, rigid, affine, homography}` | `affine` | Transformiraj model, da se prilega. |
| `--frames N` | `1` | Sinhronizirane posnetke okvirjev s povprečjem. |
| `--reference SN` | glavna | Referenčna serijska številka kamere; vsi ostali člani so deformirani na njo. |
| `--max-features N` | `5000` | Omejitev števila značilnosti ORB. |
| `--ratio-threshold F` | `0.75` | Loweov test razmerja. |
| `--ransac-threshold-px F` | `3.0` | Prag notranjih točk RANSAC. |
| `--min-matches N` | `15` | **Kakovostni** — zavrni rešitev, če je število ujemanj z notranjimi točkami manjše od tega. |
| `--max-reproj-err-px F` | `4.0` | **Kakovostni prag** — zavrni rešitev, če je napaka RMS pri ponovni projekciji večja od te vrednosti. |
| `--checkerboard RxC` | — | Geometrija plošče za `--method checkerboard`, npr. `9x6`. |
| `--name PROFILE` | prazno | Ime profila, vgrajeno v shranjeni JSON. **Ne gre za ime matrike** — to je pozicijski `NAME`. |

Ti dve kakovostni preizkušnji sta razlog, da lahko kalibracija uspešno reši problem, a kljub temu
zavrne shranjevanje: profil, ki ne izpolni ene od njiju, bi tiho napačno registriral vsak
kasnejši zajem, zato se ga zavrne, namesto da bi se shranil.

### Primeri

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### DSL receptov

`project run RECIPE.yaml` sprejema datoteko YAML ali JSON, ki opisuje zaporedje dejanj:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Podprta dejanja: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. Dejanje `burst` sprejme `name` (obvezno), `count` (privzeto 5), `interval` (v sekundah, privzeto 0), `output`, `format`, in `settings` (enaka oblika nastavitev za posamezno kamero kot pri `apply`); zaporedni posnetki v nizu uporabljajo isti nadzorni mehanizem za sveže sinhronizirano skupino kot `project burst`.

Zagnajte ga:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## Okoljske spremenljivke

| Spremenljivka | Učinek |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Prepisuje backend URL (privzeto `http://127.0.0.1:5000`) — **upošteva se le pri družinah ukazov `lattice`, `project` in `daq pool-*`).** Osnovne ukaze (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) priključijo `http://127.0.0.1:<port>` in prezri to spremenljivko (literal IPv4 zaobide kazen »Windows« `localhost`→`::1` ~2 s na zahtevo), zato vedno ciljajo na lokalni računalnik. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1` zniža zavrnitev povezave zaradi prekomernega števila povezav v nizu (skupno povpraševanje na kamero &gt; zgornja meja varne povezave brez kolizij z `pin_resolution`) na glasno opozorilo in nadaljevanje, pri čemer sprejema izgubo paketov GVSP. Samo za uporabo v laboratorijskih pogojih — glej [Model fps in burst za niz](#array-fps--burst-model). |
| `CHLOROS_CLI_MODE` | Nastavi ga sam »CLI«; naroča backendu, naj omogoči vzporedno obdelavo. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` preskoči GVSP-preizkus za nadomestno povezavo (samo rezultati ICMP). **To izklopi jumbo, ne le zmanjša število vpisov v dnevnik** — kamera odgovarja na DF-pingove le do 1500 na vsaki poti, zato je ta preizkus edina stvar, ki lahko zazna jumbo. Prihrani ~1 s na kamero na povezavo; stane ~1,45× zgornjo mejo pasovne širine, če bi omrežje *lahko* prenašalo jumbo. Stran SDK opozori, ko to nastavite. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Fiksira velikost paketa GVSP na N bajtov; popolnoma preskoči preizkušanje. Prednost dajte nastavitvi po posameznih ukazih (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) namesto trajni nastavitvi: fiksirana velikost se ne prilagaja več omrežju pred njo, fiksiranje vrednosti 9000 na poti, ki ne more prenašati jumbo paketov, pa povzroči, da **vsak** zajem poteče v časovno omejitev z napako `SC_ERR_TIMEOUT -1011`. |
| `TMPDIR` (Linux) | Prepisuje imenik za izpisovanje datotek Nuitka onefile. CLI samodejno uporabi `/mnt/ssd/tmp`, če je prisoten. |

---

## Izhodne kode

| Koda | Pomen |
| --- | --- |
| `0` | Uspešno. |
| `1` | Splošna napaka (večina napak podukazov). |
| `2` | Napaka v argumentu. |
| `130` | Prekinjeno s tipko Ctrl+C. |

---

## Nasveti za odpravljanje težav

- **»Potrebna prijava«** → Na tem računalniku enkrat zaženite `chloros-cli login EMAIL PASSWORD`.
- **»Backend ni dosegljiv«** → Zaženite namizno aplikacijo Chloros ali neposredno zaženite binarno datoteko backenda (`chloros-backend`) ali pa preverite `CHLOROS_BACKEND_URL`, če gre za oddaljeno povezavo.
- **`lattice` ukazi se ne izvedejo z napako »LATTICE camera drivers not found«** → Arena SDK runtime ni nameščen; CLI vsebuje `win32api` v paketu na Windows vendar je C-izvedbeno okolje del namestitvenega programa z grafičnim vmesnikom.
- **V oknu »Array connect« / »Array Settings« se prikaže sporočilo »FRAMES WILL DROP« ali »Reduce ROI to enable«** → Obroč sprejemanja gostiteljske mrežne kartice je premajhen (običajno se po posodobitvi gonilnika mrežne kartice ponastavi na 32). Glejte [Nastavitev in optimizacija gostiteljske mrežne kartice](#host-nic-setup--tuning-lattice-arrays) — nastavite `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Računalnik se ob ponovnem zagonu/izklopu zatakne, nato WMI `Invalid class` / omrežna kartica se nevklopiti / manjkajo USB-pogoni** → Zastarel gonilnik USB 10GbE adapterja povzroča `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Posodobite gonilnik adapterja — glejte [Nastavitev in optimizacija omrežne kartice gostitelja](#host-nic-setup--tuning-lattice-arrays).
- **Opozorilo o zamenjavi na Jetsonu** → Dodajte zamenjavo, podprto s datotekami; v datoteki »CLI« so natisnjeni natančni ukazi `fallocate` / `swapon`.
- **Manjkajo neposredni ukazi DAQ** → Pričakovano: priloženi `chloros-cli` namerno izključuje paket `daq`, zato je prisoten le `pool-*` (tudi na PyPI SDK ga ni). Uporabite `pool-*`, ki prek backenda upravlja isti senzor, ali pa `chloros_sdk.connect_daq_sensor()` s spletne strani Python.

---

## Glejte tudi

- [Python Referenca za SDK](sdk-reference.md) — programski ekvivalent vseh ukazov CLI.
- [Vodnik za senzorje DAQ](../daq/README.md) — ožičenje in kalibracija za posamezne senzorje.
- Spletna dokumentacija: `https://mapir.gitbook.io/chloros/cli`

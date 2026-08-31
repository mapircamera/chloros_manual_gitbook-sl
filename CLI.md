# CLI : Ukazna vrstica

> **Celoten priročnik:**[CLI Priročnik](reference/cli-reference.md) opisuje**vsak parameter vsakega podukaza** in je optimiziran za AI-pomočnike — vnesite njegov URL v svojega pomočnika in zaprosite za delujoč ukaz: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Nasvet za orodja z umetno inteligenco:** vsaka stran tega priročnika je na voljo v surovem formatu Markdown, če k njenemu URL dodate `.md` (npr. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), `https://mapir.gitbook.io/chloros/llms.txt` pa indeksira celoten priročnik za uporabo v velikih jezikovnih modelih (LLM).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->


## Kaj je »CLI
«

`chloros-cli` je uporabniški vmesnik za ukazno vrstico, ki temelji na istem obdelovalnem jedru, kot ga uporablja namizna aplikacijaChloros
. Gre za tanekHTTP
odjemalec, ki deluje prek strežniškega delaChloros
(lokalni strežnik na `127.0.0.1:5000`) — večina ukazov samodejno zažene strežniški del, zato skriptu zadostuje en sam klic `chloros-cli process …`.

Deluje na **Windows
10/11 (x64)**in**Linux
(x86_64 ter NVIDIA Jetson arm64 na JetPack 6)**, v katerem koli terminalu, brez potrebe po grafičnem vmesniku. Preverite svojo namestitev s:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

Pregled družin ukazov:

* **Obdelava in račun** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 jezikov — glej [Podprti jeziki](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (samo zaLinux
/Jetson)
* **Delujoča strojna oprema** — `lattice` (upravljanje kamere LATTICE, več kot 45 podukazov), `daq pool-*` (svetlobni senzorji DAQ), `time-sync` (PTP)
* **Avtomatizacija** — `project` (izvajanje shranjenega projektaChloros
brez grafičnega vmesnika, vključno z recepti zajemanja v formatu YAML)

Globalne možnosti, ki jih je vredno poznati: `--port N` (vmesnik strežnika, privzeto `5000`), `-v/--verbose`, `--restart` (prisilni ponovni zagon backenda), `--backend-exe PATH`. Celoten seznam najdete v [CLI
Referenca](reference/cli-reference.md).

***

## Namestitev

CLI
**je vključen v namestitveni programChloros** za vse platforme — ni ločenega prenosaCLI
. Namestitveni program prenesite s strani [Prenos](download.md).

###Windows


Namestitveni program namestiCLI
v:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

in to mapo doda v vaš sistem `PATH` — po namestitvi **odprite nov terminal**, da se posodobi `PATH`. Namestitveni program v korensko mapo namestitve shrani tudi skripte za zagon (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) ter**Chloros
CLI
** povezavo v meniju Start, od katerih vsaka odpre terminal z `chloros-cli`, pripravljenim za uporabo.

###Linux


Namestite `.deb` za vašo arhitekturo:

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

S tem namestite `chloros-cli` do `/usr/bin/chloros-cli` (že na `PATH`) ter backend na `/usr/lib/chloros/chloros-backend`, skupaj z izvedbenim okoljem ArenaSDK
, potrebnim za kamere LATTICE. Za podrobnosti glej [NamestitevLinux
](linux/linux-installation.md).

### Preverjanje

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Prijava in licenciranje

CLI
(inPython
SDK
) dostop zahteva **plačljiv načrtChloros
+**— vsi plačljivi načrti ga vključujejo; brezplačni načrt ga ne vključuje. Omejitev se izvaja**na strani strežnika** s strani backenda, ne s strani izvedljive datotekeCLI
: klic, ko uporabnik ni prijavljen, se zavrne s kodo napake `401 AUTH_REQUIRED`, klic v prijavljenem stanju na brezplačnem paketu pa z napako `403 PLAN_UPGRADE_REQUIRED`, ne glede na to, ali prihaja iz `chloros-cli`,SDK
ali ročno izdelanega odjemalcaHTTP
. Nadgradite na [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Prijavite se **enkrat na računalnik**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->


{% hint style="warning" %}
**Gesla s posebnimi znaki**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` se v lupini poškoduje (CLI
to zazna ob napaki 401 in samodejno ponovi poskus, vendar se težavi v celoti izognete z uporabo enojnih narekovajev).
{% endhint %}

Sejna je shranjena v predpomnilniku v `~/.chloros/user_session.json` in deluje brez internetne povezave še v obdobju odloga plačila (30 dni za mesečne naročnine, do izteka veljavnosti za letne). `chloros-cli status` deluje tudi brez plačanega načrta, zato je razlog za zavrnitev vedno viden.

{% hint style="danger" %}
**Načrtujete brezglavo delo? Najprej se prijavite.**Ukaz za zagon strežniškega procesa (`process`, `status`, `export-status`, …) brez**shranjene seje**v predpomnilniku ne ponesreči takoj – namesto tega se prikaže interaktivni ukazni vrstici `Email:` / `Password:` na stdin. Avtomatizirana cron naloga ali korak CI se bo zato**zastal v čakanju na vnos**. Pred načrtovanjem česarkoli enkrat zaženite `chloros-cli login EMAIL 'PASSWORD'` na računalniku.
{% endhint %}

***

## Vaš prvi izvedbeni cikel

Usmerite `process` v mapo z zajetimi podatki — samodejno zaznaSurvey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` ali njihovo kombinacijo:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Napredek se prenaša v realnem času po posameznih niti procesa (zaznavanje, analiza, obdelava, izvoz), uspešno izvajanje pa se zaključi s poročilom o številu zapisnih slikovnih izdelkov (`Image products written: N`).



<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### Kam se shranijo izhodni podatki

`process` zapisuje v **projektno mapo**, ne v vašo vhodno mapo:

* Če ni `-o`: projekt se ustvari v vaši privzeti projektni mapi (skupni z grafičnim vmesnikom; upravljajte jo z `get-project-folder` / `set-project-folder`, nadomestna možnost `~/Chloros Projects`), poimenovana z `-n/--project-name` ali časovnim žigom (`YYYYMMDD_HHMMSS`), če je izpuščena.
* Pri `-o PATH`: ta mapa **je** projektna mapa. Če že vsebuje `project.json`, se namesto prepisovanja ustvari sestrska mapa s končnico `_1`/`_2`…

Znotraj projekta so izdelki razvrščeni **po kameri, nato pa po formatu datoteke**:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Mapa kamere je `LATT-<sensor>-<lens>-F<filter>` za LATTICE (kar ustreza EXIF-u posnetka `Model`) in `<model>_<filter>` (npr. `Survey3N_RGN`) zaSurvey3
. Mapa formata sledi `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` ali `tiff32` za `TIFF (32-bit, Percent)`.

{% hint style="info" %}
**Vsak izvožen izdelek ohrani ime izvorne datoteke.**Izvoz radianca iz `capture_..._raw.tif` se še vedno imenuje `capture_..._raw.tif` — le da se nahaja v mapi `tiff32/Radiance_Images/`.**Izdelek identificira mapa, ne ime datoteke**, zato uporabite glob za mapo, ne za končnico `*radiance*`.
{% endhint %}

### Možnosti, ki jih boste dejansko uporabljali

| Oznaka | Privzeto | Kaj počne |
| --- | --- | --- |
| `-o, --output PATH` | privzeta mapa projekta | Mesto mape projekta (glej zgoraj). |
| `-n, --project-name NAME` | časovni žig | Ime projekta. |
| `--format FMT` | `TIFF (16-bit)` | Ena od naslednjih možnosti: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | noben | Indeksi vegetacije za izvoz (glej [Indeksi vegetacije](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = nevronski debayer, počasnejši, najvišja kakovost (Chloros
+, grafična kartica NVIDIA). |
| `--vignette / --no-vignette` | vklopljeno | Popravek vinjetiranja. |
| `--reflectance / --no-reflectance` | vklopljeno | Kalibracija odbojnosti; za LATTICE je to tudi stikalo za produkt odbojnosti. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Prisilna nastavitev vstopne točke procesnega toka za datoteke TIFF sistema LATTICE. |

Za vse ostalo — nastavitev zaznavanja ciljev, PPK, zatiče osvetlitve, zastavice poravnave matrike — glejte [poglavje `process` v referenčnem priročniku za programCLI
](reference/cli-reference.md).

***

## Izbiranje, kaj izvoziti (izdelki LATTICE)

Obdelava LATTICE se v enem prehodu razširi na **vse ustrezne izdelke**. Štiri stikala za posamezne izdelke so**privzeto vklopljena**; uporabite obrazec `--no-`, da eno izklopite:

| Stikalo | Izdelek |
| --- | --- |
| `--debayered` | Linearna demozajka → `Debayered_Images/` |
| `--preview` | Predogled zaslona (balanca beline + gama; raztezanje lažnih barv za multispektralne slike) → `Preview_Images/` |
| `--radiance` | sevanje v formatu float32, W/m²/sr/nm → `Radiance_Images/` (vedno `tiff32/`) |
| `--reflectance` | uint16 odbojnost, pripravljeno za Pix4D → `Reflectance_Calibrated_Images/` |

RGB
glavne kamere vedno oddajajo le podatke brez Bayesovega filtra + predogled — sevanje/odbojnost po posameznih pasovih za širokopasovni senzor nima pomena, zato ti stikala zanje ne delujejo.Survey3
`.raw` prezre stikala in sledi standardni poti odbojnosti/cilja.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (privzeto `auto`) izbere referenco odbojnosti: `auto` ustvari [kalibracijski cilj](calibration-targets.md) znotraj okvirja, ki izpolnjuje zahteve za nadzor kakovosti absolutno referenco in se vrne na delitno razmerje svetlobnega senzorja DAQ za navzdol usmerjeno svetlobo (ρ = π·L/E), kadar cilj ni prisoten; `target` je strog (brez nadomestitve DAQ); `daq` se zanaša na DAQ. Z `--target-reflectance-dir` je mogoče zagotoviti skeniranje ciljev, izmerjenih na enoto.

{% hint style="info" %}
**Branje pikslov odbojnosti:**DN, ki pomeni ρ = 1,0, je**na vir** — Datoteke LATTICE v XMP vpišejo oznako `Chloros:PixelScale=32768`; datotekeSurvey3
uporabljajo vrednost 65535 (in ne vsebujejo oznak `Chloros:*`). Preberite oznako in delite z njo, namesto da predpostavljate konstanto. Podrobnosti in en nameren mejni primer brez merila so v [CLI
Referenca](reference/cli-reference.md).
{% endhint %}

**Obdelava se vedno začne z `raw`.** Izpeljani izdelki (izvozi brez debayeringa/svetilnosti/odbojnosti) se nikoli ne vračajo skozi procesno verigo — njihov ponovni uvoz in obdelava bi pomenila dvojno uporabo kalibracijskih izračunov, zato jihChloros
preskoči in to tudi navede. `--input-level` je namerno vgrajena rešitev za primer, ko resnično potrebujete prisiliti začetno točko.

***

## Ko obdelava ni uspešna

Od različice 1.2.0 naprej se `process` jasno ne uspe, namesto da bi se »uspešno« zaključil brez prikazanih rezultatov:

* Izvajanje, ki je **zahtevalo izdelke, a ni zapisalo nobenega**— le `project.json` in `calibration_data.json` — izpiše `Processing finished but wrote no image products.` in**se zaključi z vrednostjo, ki ni nič**, tako da ga skripti lahko zaznajo. Običajni vzroki: vhodna mapa ni bila prepoznana kot zajem (preverite razpored in `--input-level`) ali pa noben od zahtevanih produktov ni bil uporaben za te kamere (npr. zahteva po sevanju/odbojnosti iz kamer, ki podpirajo leRGB
).
* **Namerno izvajanje samo z metapodatki** (vsi izdelki izklopljeni, brez `--indices`) je še vedno uspešno — prazen izhodni slikovni zapis je v tem primeru pravilen rezultat.
* Ponovno zaženite z `--verbose` in v dnevniku strežnika preverite vrstice `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, ki pojasnjujejo preskoke po posameznih kamerah.

Kode izhoda: `0` uspeh · `1` splošna napaka · `2` napaka v argumentu · `130` prekinjeno s Ctrl+C.

***

## Indeksi vegetacije

`--indices` izvedite z enim ali več imeni prednastavitev; vsak indeks se shrani v svojo mapo `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

22 vnaprej nastavljenih imen, ki jih `process --indices` sprejema:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**Obstajajo trije seznami indeksov — ne zamenjujte jih.**Spustni meni »Nastavitve projekta« v grafičnem uporabniškem vmesniku vsebuje 27 formul (dodaja `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` – teh pet je na voljo le v grafičnem vmesniku in**ne** veljajo za `--indices`). Ukaz `lattice index --preset` v realnem času/offline uporablja lasten ločen seznam 22 prednastavitev. Formule in matematični izračuni za pasove so dokumentirani v [Formulah multispektralnih indeksov](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Svetlobni senzorji DAQ: Kratek pregled

Družina `daq pool-*` upravlja spektralne senzorje DAQ podjetjMAPIR
(DAQ-U prek USB, DAQ-M prek BLE, DAQ-E prek Etherneta) prek trajnega sklada v ozadju — grafični vmesnik (GUI),CLI
inSDK
si delijo eno aktivno ročico. **`pool-*` je podprta pot DAQ v priloženemCLI
**; drugi podukazi `daq`, na katere se morda sklicujete, soMAPIR
– notranja površina, namenjena izključno virom, ki se zaključi z izrecno napako, ki vas usmerja na `pool-*`.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record` brez `--duration` teče do `pool-record --stop`; privzeti izhodni imenik je `~/Documents/DAQ Live View/` **na računalniku backenda**. Profil korekcije omejitve se izbere ob vzpostavitvi povezave (`--cap-id`, privzeta vrednost strežnika `sunshine_cosine`) in ga je mogoče v realnem času zamenjati s `pool-set-cap` — profili omejitve in kalibrirano območje senzorja so obravnavani v poglavjih o DAQ v tem priročniku.

{% hint style="warning" %}
**DAQ-E na gostitelju z več mrežnimi karticami:** prvo samodejno odkrivanje `pool-connect --eth` po zagonu lahko spodleti, tudi če je senzor v dobrem stanju. `--eth-host <ip-or-hostname>` je zanesljiva oblika — uporabite jo, kadar odkrivanje ne prinese rezultatov.
{% endhint %}

***

## Kamere LATTICE, PTP in avtomatizacija projektov

Družina `lattice` (več kot 45 podukazov) pokriva celotno delovanje kamer LATTICE: odkrivanje, posamezne posnetke, trajne sinhronizirane nize s tokom povezovanja »smart-prep« v grafičnem vmesniku, predogled v živo v brskalniku, poravnavo, indeksno matematiko in diagnostiko gostiteljske mrežne kartice. Primer:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Poleg tega: `chloros-cli time-sync` poroča o glavnem PTP-strežniku (grandmaster), ki ga poganja gostiteljChloros
(kamere LATTICE in senzorji DAQ-E so mu podrejeni za časovne oznake med napravami), `chloros-cli project` pa odpre shranjeni projekt »Chloros
« in brez grafičnega vmesnika upravlja s kamerami, matričnimi senzorji in senzorji – vključno s skriptiranimi recepti za zajem v formatu YAML.

Te tri družine (`lattice`, `project`, `daq pool-*`) so tudi edine, ki upoštevajo `CHLOROS_BACKEND_URL` za upravljanje **oddaljenega** backenda; osnovni ukazi so vedno usmerjeni na lokalni računalnik.

Podrobna navodila so na voljo v poglavjih o LATTICE v tem priročniku; vsi parametri so navedeni v [CLI
Referenca](reference/cli-reference.md).

***

## Odpravljanje težav: Top 5

| Simptom | Rešitev |
| --- | --- |
| `Login required` ali načrtovana naloga se zatakne pri pozivu `Email:` | Na tem računalniku enkrat zaženite `chloros-cli login EMAIL 'PASSWORD'` — ukazi brez shranjene seje se izvajajo interaktivno, namesto da bi se takoj prekinili. |
| `backend unreachable` | Zaženite namizno aplikacijoChloros
ali neposredno zaženite binarno datoteko strežnika (`chloros-backend`). Če usmerite `lattice`/`project`/`daq pool-*` na oddaljeni backend, preverite `CHLOROS_BACKEND_URL`. |
| Povezava z nizom je blokirana: `FRAMES WILL DROP` / `Reduce ROI to enable` | Obroč sprejemanja omrežne kartice gostitelja je bil ponastavljen na privzete vrednosti — to je najpogostejši vzrok, da se sistem, ki je prej deloval, ne želi več povezati, običajno po posodobitvi gonilnika omrežne kartice. Zaženite `chloros-cli lattice network --fix` iz **terminala s povečanimi pravicami** (ali nastavite `ReceiveBufferLen=256`, `PendingReceives=64`); glejte poglavje *Nastavitev in optimizacija omrežne kartice gostitelja* v referenčnem priročniku. |
| Podukaz `daq` se zapre: »zahteva celoten paket daq…« | Pričakovano pri dobavljenih različicah — kompilirana različicaCLI
vsebuje le družino ukazov `daq pool-*`, ki zajema povezovanje, prenos podatkov, snemanje in izbiro datotek CAP. Uporabite `pool-*` (ali `chloros_sdk.connect_daq_sensor()` izPython
). |
| Jetson pred obdelavo velikih map prikaže opozorilo o zamenjavi | Dodajte zamenjavo, podprto s datotekami —CLI
izpiše natančne ukaze `fallocate`/`swapon`, ki jih je treba izvesti. |

***

## Pomoč

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Vsak parameter, vsak podukaz:** [CLI
Referenca](reference/cli-reference.md)
* **Ekvivalent vPython
:** [Python
SDK
](api-python-sdk.md) in [SDK
Referenca](reference/sdk-reference.md)
* **Podpora:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)

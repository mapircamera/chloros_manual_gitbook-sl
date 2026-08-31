# Chloros Python SDK Referenca

**Različica:**

1.2.0**Ustvarjeno:**

29. 7. 2026 ob 19:19 ·**Revidirano:**

30. 8. 2026**Paket:** `chloros-sdk` (PyPI)**Ciljna publika:** Optimizirano za uporabo v velikih jezikovnih modelih (LLM); berljivo za ljudi.**Obseg:** Vse javne razrede, funkcije in pomožne funkcije, ki jih ponuja `import chloros_sdk`, s primeri, ki jih je mogoče kopirati in prilepiti ter ki zajemajo obdelavo slik, krmiljenje ene kamere, sinhronizirane nize, DAQ-senzorje in avtomatizacijo projektov.

Če vas zanimajo le najpomembnejše informacije, preskočite na:
- [Namestitev in hiter začetek](#namestitev)
- [Smart-Connect za LATTICE-matrice](#smart-connect-for-lattice-cameras)
- [Sejne nastavitve DAQ-senzorjev](#daq-sensor-sessions)
- [Avtomatizacija projektov](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## Arhitektura v 60 sekundah

SDK je tanek sloj Python nad ozadjem Chloros (isti strežnik Flask, ki ga uporabljata namizni grafični vmesnik in CLI). Za avtomatizacijo uvozite `chloros_sdk` in pokličete metode visoke ravni; v ozadju se vsak klic pretvori v zahtevo HTTP do lokalnega backenda na vratih 5000 — `http://127.0.0.1:5000/api/...` (namerno ne `localhost`, ki se najprej razreši v `::1` na Windows in stane ~2 s na zahtevo pri backendu, ki podpira le IPv4). Backend je lastnik sklopa strojne opreme — kamere, senzorji DAQ, profili poravnave, okvirni pomnilniki — zato lahko skripti SDK delujejo vzporedno z grafičnim vmesnikom (GUI), ne da bi se morali boriti za serijske vrata ali pasovno širino omrežne kartice.

Uporabljali boste tri vmesnike:

1. **`ChlorosLocal` + brezplačne funkcije** (`process_folder`, `process_lattice_capture`) – Potek obdelave slik. Z enim klicem Python izvedite kalibracijo, debayer in izvoz indeksa za celotno mapo.
2. ****Smart-connect ročaji** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Odprite trajno sejo v ozadju za delujočo strojno opremo. Enak potek »pametne priprave« kot v grafičnem vmesniku: preverjanje omrežja, samodejna izbira ravni, PTP, nastavitev AE, konfiguracija sprožilca GPIO.
3. **`ChlorosProject` / `open_project`** — Naložite shranjen projekt (mapo z datotekami `cameras.json` + `sensors.json` + `project.json`), vse hkrati poveže in omogoči zajem z imenovanimi ročaji.

Površini 1 in 2 **samodejno zaženeta lokalni backend** če ta še ne posluša (isti priloženi izvedljivi datotek, ki ga sproži GUI/CLI) — tako da skript deluje iz novega terminala, ne da bi morali najprej zagnati backend. Za izklop te funkcije prenesite `auto_start_backend=False` (npr. ko kaže na oddaljeni backend, ki se nikoli ne zažene). Glej [Samodejni zagon backenda](#backend-auto-start). Surface 3 se obnaša drugače: `open_project()` ne sprejema parametra `auto_start_backend`, `connect_all()` pa nikoli ne zažene backenda — enkrat preveri `http://127.0.0.1:5000` in, če se nič ne odzove, tiho preklopi na neposredno (brez backendabrez backenda) `lattice_sdk`. Le `proj.process()` in `stream(..., overlays=True)` leno vzpostavita `ChlorosLocal()` (ki se samodejno zažene).

Vse tri so zaščitene z avtentifikacijo: enkrat zaženite `chloros-cli login` na računalniku ali se prijavite prek namiznega grafičnega vmesnika. Klici SDK brez veljavne seje sprožijo napako `ChlorosAuthenticationError`.

Zahteve:
- Python 3.7+ (kot navaja paket; razvit/preizkušen na različici 3.10)
- Lokalno nameščen Chloros Desktop (binarna datoteka za ozadje je vključena v namestitveni program)
- Aktivna prijava v Chloros+. Najnižja raven za SDK/CLI je stopnja **Copper**ali višja (Copper / Bronze / Silver / Gold); brezplačna stopnja**Iron**nima dostopa do SDK/CLI. To se uveljavlja**na strežniški strani**: vsaka zahteva z oznako SDK/CLImora vsebovati tako aktivno sejo kot tudi plačani načrt, sicer strežnik vrne `403` z `error_code: PLAN_UPGRADE_REQUIRED` (ki se prikaže kot `ChlorosLicenseError` prek `ChlorosLocal` in kot `ChlorosConnectError` prek pomožnih funkcij `connect_*`). Odjavljeni uporabnik namesto tega prejme `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — ti dve sta različni, ker ponovno zagnanje `chloros-cli login` popravi prvo, druge pa ne more popraviti.
- Offline uporaba je podprta v okviru obdobja odloga naročniškega paketa: stopnja se prebere iz predpomnilnika za preverjanje veljavnosti na strežniku (5 min) ali iz podpisanega, licenčni predpomnilnik, vezan na računalnik (30 dni za mesečne načrte, do izteka naročnine za letne). Ko to obdobje odloga poteče, se načrt spremeni v brezplačnega in dostop do SDK/CLI dostop se ustavi, dokler računalnik ne vzpostavi povezave s strežnikom. `chloros-cli status` (`GET /api/license-status`) ostane dostopen v brezplačnem nivoju, zato je razlog viden – to je edina pot SDK/CLI, ki je izvzet iz omejitve stopnje.
- Windows 10/11 64-bitni, **Ubuntu 22.04 LTS ali novejši**ali Jetson (JetPack 6). Ubuntu 20.04**ni** podprt: odvisnosti `.deb` izhajajo iz tega, s čimer se povezuje backend, vključno z `libc6 (>= 2.34)`, Focal pa vsebuje glibc 2.31.

---

## Namestitev

Python SDK je tanek sloj Python nad backendom Chloros. Za vse, kar presega nekaj delovnih tokov, namenjenih izključno zajemanju podatkov (DAQ) , potrebujete **lokalno nameščen namizni paket Chloros** (namestitveni program Windows ali Linux `.deb`) — ta namreč zagotavlja binarno datoteko backenda, izvedbeno okolje Arena SDK za kamere LATTICE ter kalibracijske pakete.

Najnovejše datoteke za prenos: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Korak 1 — Namestite paket platforme Chloros

#### Windows (.exe)

1. Prenesite `Chloros-Setup-x.y.z.exe` s strani za prenos.
2. Zaženite namestitveni program in sledite navodilom čarovnika. Privzeta namestitvena pot je `C:\Program Files\MAPIR\Chloros\`.
3. Vsaj enkrat zaženite Chloros in se prijavite s svojim računom Chloros+.

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### Korak 2 — Namestite Python SDK

**Namestitveni program Chloros vsebuje ustrezen paket SDK.** Vsak namestitveni program Windows in datoteka .deb Linux na disk namesti `chloros_sdk-X.Y.Z-py3-none-any.whl`, ki se natančno ujema z različico GUI / CLI / različico backenda. Ni vam treba spremljati PyPI, da bi ostali sinhronizirani.

#### Windows

Namestitveni program samodejno zažene `pip install` na priloženem paketu wheel z uporabo vašega sistemskega Python (priporočljiv je zagon `py.exe`, v nasprotnem primeru pa se zateče k `python -m pip`). Ni potrebno nobeno ukrepanje — `import chloros_sdk` deluje v vašem Python okolju po uspešni namestitvi. Če Python na računalniku, namestitveni program ta korak tiho preskoči, grafični vmesnik in CLI pa še naprej delujeta.

#### Linux (.deb)

Datoteka .deb namesti kolo v `/usr/lib/chloros/sdk/`. `postinst` izpiše natančno ukazno vrstico — distribucije po PEP 668 privzeto zavračajo globalno pisanje v pip, privzeto ne dovoljujejo globalnega pisanja v pip, zato ne izvajamo samodejne namestitve:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Za namestitve Jetson v izoliranem omrežju je to popolnoma brez povezave – wheel je že na disku.

#### Javni PyPI

Za gostitelje, ki uporabljajo izključno pip (brez nameščenega namiznega paketa Chloros; delovni tokovi z oddaljenim backendom ali izključno za DAQ):

```bash
pip install chloros-sdk
```

PyPI se posodablja ob izdaji novih različic namestitvenih zgradb, zato objavljen wheel ustreza najnovejši stabilni izdaji. Razvojne zgradbe (npr. `1.1.4.dev1`) se dostavljajo le prek priloženega namestitvenega wheela.

#### Preverjanje

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Chloros+ je potrebna naročnina.** Vsi klici SDK zahtevajo aktivno prijavo Chloros+. Izvedite `chloros-cli login user@example.com 'YourPassword'` enkrat na vsak računalnik; pristopni podatki se shranijo v `~/.chloros/`.

### Ali potrebujem paket za namizje?

Sam paket pip **ni** dovolj za večino delovnih tokov. Tukaj je navedeno, kaj potrebuje vsaka SDK površina:

| Površina SDK | Potrebuje paket Desktop? | Zakaj |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Da** | Samodejni zagon binarnega programa v ozadju na `/usr/lib/chloros/chloros-backend` (Linux) ali `C:\Program Files\MAPIR\Chloros\…` (Windows). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Da**(lokalno)**/ Ne**(oddaljeno) | Potrebni so čisti odjemalci HTTP prek strežnika. Lokalni backend → potreben namizni paket. Oddaljeni backend → `backend_url=`**prek tunela** (glej Način oddaljenega backenda — priloženi backendi se vežejo le na zanko). |
| `ChlorosProject` / `open_project` | **Da** | Pogoni shranjenih projektov prek backenda. |
| Neposredni razredi LATTICE (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Da** | Potrebujejo nativno izvedbeno okolje Arena SDK, ki je priloženo v namiznem paketu. `CAMERA_AVAILABLE` je sicer `False` ob uvozu. |
| Razredi za neposredno zajemanje podatkov (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **Ne** | Čisti Python prek pyserial/bleak/zeroconf. Okolje, ki temelji izključno na pip, lahko upravlja z DAQ-ji od začetka do konca. |

### Način oddaljenega strežnika (gostitelj, ki uporablja izključno pip, prek tunela)

> **Priloženi strežnik ni dosegljiv prek LAN-omrežja.** Produkcijske
> različice se vežejo izključno na loopback (obe družini loopback) in trdno zavračajo
> edini način, ki ni loopback (`CHLOROS_CLOUD_MODE`), zato
> `backend_url="http://<lan-ip>:5000"` **ne more delovati z nameščenim
> Chloros** — ta vzorec je vedno deloval le z backendom iz vira/razvoja
> backend. Če želite upravljati backend na drugem računalniku, sami preusmerite njegovo
> loopback vrata in usmerite SDK na tunel:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

Gostitelji brez zaslona / CI / robotika lahko en računalnik s polno namestitvijo namizja ohranijo kot »Chloros strežnik«, drugje pa uporabljajo `pip install chloros-sdk` — vendar je prenos med njimi zgoraj navedeni tunel, ki ga uredi uporabnik, ne pa neposredna povezava v lokalnem omrežju URL.

> **Znana omejitev — `ChlorosLocal` ne podpira izključno pip.** `ChlorosLocal(backend_url=BACKEND)` trenutno v svojem konstruktorju razreši lokalno binarno datoteko backenda *preden* preveri URL in sproži `ChlorosBackendError` (»Chloros backend ni najden…«), če ni nameščen noben namizni paket — celo če je oddaljeni backend dosegljiv. Le površina smart-connect zgoraj (`connect_camera` / `connect_array` / `connect_daq_sensor`, skupaj z `analyze_array_network` in pomočniki `list_*` / `discover_*`) delujejo z gostitelja, ki uporablja izključno pip.

### Delovni tok izključno za DAQ (gostitelj, ki podpira le pip)

Če potrebujete le senzorje DAQ in ne uporabljate kamer LATTICE ali obdelave slik, je paket pip samostojen:

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

Za neposredno delo z DAQ na stroju ni potreben noben backend, noben .deb in nobeno prijavljanje v Chloros+.

---

## Hitri začetek

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Glavni indeks API

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## Obdelava slik — `ChlorosLocal`

Glavni razred za obdelavo podatkov. Ob prvi uporabi zažene backend, ustvari in konfigurira projekte, spremlja napredek ter vrne povzetke po zaključku izvajanja.

### Konstruktor

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Metode

| Metoda | Opis |
| --- | --- |
| `create_project(project_name, camera=None)` | Ustvari nov projekt (po želji s predlogo kamere, kot je `"Survey3N_RGN"`). |
| `import_images(folder_path, recursive=False)` | Uvozi slike v formatih RAW/TIF/JPG/DNG **in posnetke svetlobnega-senzorjev**. Vrne `count` (slike) in `scan_count` (posnetke). Opozori le, če mapa ne vsebuje niti enega niti drugega. |
| `export_light_sensor(daq=True, csv=True)` | Zapiše kalibrirane `.daq` + `.csv` za vsak posnetek svetlobnega senzorja v projektu v `<project>/Light Sensor/`. Glej [Posnetki svetlobnega senzorja](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Nastavite parametre obdelave. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Zaženite obdelovalno pot. Vrne `{"status": "complete", "async": False}`, in ključ `summary`, če ga backend zagotovi — glej [Povzetek po izvedbi in nasveti](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | Preveri stanje backenda. |
| `logout()` | Izbriši shranjene poverilnice iz predpomnilnika. |
| `shutdown_backend()` | Ustavite backend (če ga je zagnal SDK). |
| `discover_cameras()` | Odkrijte kamere LATTICE **prek backenda te instance** (`/api/camera/discover`). Vrne seznam slovarjev (`serial`, `model`, `ip`, …) — enake oblike, kot jih vidi GUI/CLI. Prazen seznam, če ni najdenih kamer ali če je backend nedosegljiv. |
| `camera_capture(output_dir, format="tiff", **settings)` | Zajame en sam okvir**prek backenda**(samodejni zagon prek tega ročaja), tako da dobi enako pripravo kot GUI/CLI (privzeto 12-bitno, ponovna uporaba sklada, vgrajeni kalibracijski metapodatki). Cilj določite z `serial=` ali `device_index=`; prenesite `exposure`/`gain`/`pixel_format`/`preset` kot `**settings`. Vrne slovar metapodatkov stare različice (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Prikaz predoglednih okvirjev, sestavljenih s prekrivanjem, iz združene kamere — tanek MJPEG-odjemalec prek poti `/api/camera/<serial>/stream-annotated` v ozadju (zebra / mreža / križec / histogram / peaking / točka, narisana na strežniški strani). `decode=True` generira BGR-nize; `False` generira surove JPEG bajte. Dostopno tudi po posameznih projektih kot `ChlorosProject.stream(overlays=True)`. |

Uporabite kot upravitelja konteksta za zagotovljeno čiščenje:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Zapisi svetlobnih senzorjev — kalibrirani `.daq` + `.csv`

DAQ-U / DAQ-M / DAQ-E je mogoče snemati **brez** kalibracijskega paketa. To je
tisto, kar javni [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) počnejo privzeto: zapisujejo surove vrednosti senzorjev in označijo
datoteko, tako da Chloros najprej pridobi tovarniško kalibracijo tega senzorja **po serijski številki** – najprej iz lokalnega predpomnilnika,
nato iz oblaka MAPIR – in jo uporabi ob uvozu.

Chloros rezultat zapiše nazaj kot dva izdelka na posnetek, pod
`<project>/Light Sensor/`:

| Izdelek | Kaj je |
| --- | --- |
| `<name>_calibrated.daq` | Arhiv, ki ga je mogoče ponovno obdelati — ista shema kot pri posnetku v realnem času, zdaj pa navaja paket, ki ga je ustvaril. Ponovni uvoz **ne** kalibrira. |
| `<name>_calibrated.csv` | Spektralna obsevanost v W/m²/nm na lastni valovni mreži senzorja, ena vrstica na odčitek, skupaj s fotometričnimi stolpci (skupna moč, fotopični/skotopični luks, PPFD in njegova razdelitev na modro/zeleno/rdeče, vrhunska valovna dolžina). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Samo senzorji brez svežnja (DAQ-A).** Surovi spektralni odčitki senzorja — *ne* sevanje. Glej spodaj. |

`process()` izvede ta izvoz kot eno od svojih faz. Za to **ni** potrebna slika:
samostojno letenje svetlobnega senzorja je vrhunski delovni tok, takšen projekt pa po svoji naravi nima nobene
slike.

**Zapisi DAQ-A se izvozijo kot surove vrednosti.** Družina DAQ-A je starejša od sistema svežnjev po serijskih številkah
in nima svežnja, ki bi ga bilo treba pridobiti — namesto tega se kalibrira na terenu glede na
reflektančni cilj, zato ga nikoli ni potrebovala. Ti zapisi se izvozijo
pod korenom `_raw` namesto `_calibrated`: gre za drugačno ime datoteke namesto zastavice
znotraj datoteke, saj mora oznaka ostati nespremenjena tudi ob pošiljanju po e-pošti kot golo ime. Glava
`.csv` navaja `raw spectral sensor counts (NOT irradiance)` in opozarja, da so
vrednosti primerljive **znotraj** datoteke – točno za to jih uporabljakalibracija na podlagi cilja
jih uporablja — in ne med senzorji. Fotometrični stolpci, odvisni od moči (skupna moč,
fotopični/skotopični luks, PPFD), se vrnejo kot **NULL** namesto da bi bili integrirani iz števcev.

DAQ-U / DAQ-M / DAQ-E, katerega sveženj preprosto ni bilo mogoče pridobiti, se še vedno **preskoči**,
ne zapiše v surovi obliki: tam sveženj obstaja in »ponovna povezava in ponovna obdelava« sta resnična nasveta.

Starejši posnetki **v1.01 / v1.02** (te piše DAQ-A-SD) ne vsebujejo epohe za vsak odčitek,
temveč le čas pisanja datoteke. Program za usklajevanje slike↔navzdolnega sevanja jih še vedno zavrača — usklajevanje
s časom zapisa bi bil nevidno napačen — vendar jih izvoznik prebere, in
CSV natisne `clock=daq_created_on`, tako da izdelek navaja, na kateri uri deluje.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Posnetek, katerega kalibracijskega paketa ni mogoče pridobiti (offline ali senzor brez
kalibracije v datoteki), se poroča pod `skipped` **z navedbo razloga**. Nikoli se ne
zapiše kot »kalibrirana« datoteka z neobdelanimi števili — povežite se z internetom in
ponovno zaženite postopek, da se izvoz zaključi.

### Povratni klici o napredku

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Povzetek po izvedbi in nasveti

Ob zaključku `process()` pridobi `GET /api/processing-summary` in priloži vsebino kot `result["summary"]`. Prenos poteka po načelu »najboljšega možnega prizadevanja« in nikoli ne blokira uspešnega vrnitve – če povzetek ni na voljo, se `process()` vrne k osnovni obliki `{"status": "complete", "async": False}`. Vsak vnos v `summary["hints"]` – celotni stavki s predlaganim popravkom, npr. zakaj je izvedba prinesla ničelni izhod — se prav tako ponovno izpiše kot Python `UserWarning`, tako da so 0izhodi brez izhoda se samodiagnosticirajo, tudi če slovarja nikoli ne pregledate:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` je strojno berljiva polovica:

| Ključ | Kaj šteje |
| --- | --- |
| `models` | Skupine kamer v izvedbi. |
| `images_in_groups` | Izvorne slike v teh skupinah. |
| `targets_found` | Zaznani cilji odbojnosti. |
| `images_calibrated` | Slike, ki jih je izvedba kalibrirala. |
| `exported_files` | **Datoteke s slikovnimi izdelki, ki jih je izvozil izvedbeni cikel.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | Zapisi svetlobnih senzorjev, namerno šteti ločeno — izhajajo iz druge faze in obstajajo tudi za izvedbe brez slik, zato bi njihovo vključevanje povzročilo, da bi se izvedba, namenjena izključno zbiranju podatkov (DAQ), kazala, kot da je izvozila slike. |

Poleg njih: `summary["output_dirs"]` (vsak imenik, v katerega je bilo zapisano),
`summary["light_sensor_export"]`, `summary["stopped"]` (velja, če je uporabnik prekinil
izvedbo, zato se delni izračuni ne obravnavajo kot zaključena izvedba z nezadostno produkcijo) ter
`summary["groups"]` (razčlenitev po skupinah).

`exported_files` se v cevovodu zabeleži **med zapisovanjem**, ne pa naknadno s skeniranjem
slikovnih objektov projekta. Paralelne strategije in strategije GPU ustvarjajo svoje lastne slikovne
objekte (v delovnih podprocesih za poti GPU), zato je staro skeniranje poročalo o
`0 file(s) written` za vsak tak izvedbeni cikel in nato izdalo namig o ničelnih izvozih – pri izvedbenih ciklih,
kjer je vse delovalo. Če pišete skripte na podlagi te številke, zdaj zdrav vzporeden izvedbeni cikel
poroča o številu, ki ni nič.

Poročila o preskočenih svetlobnih senzorjih navajajo dejanski razlog, ki ga je bralnik ugotovil za vsako datoteko –
neberljiva shema, manjkajoči sveženj, napaka pri pisanju –, **brez podvajanja**, tako da se dvajset datotek,
preskočenih zaradi enega vzroka, prikaže kot en vzrok namesto dvajsetih ponovitev istega vzroka.

> **`process()` se ne sproži, kadar izvedba ne ustvari nobenih slik.** To je edini primer, kjer se SDK in
> CLI namerno razlikujeta: `chloros-cli process` obravnava »zahtevani so bili izhodi, vendar ni bil
> zapisan noben« kot napako in se zaključi z vrednostjo, ki ni nič, medtem ko se SDK normalno zaključi in poroča o
> stanje prek `summary` / namigov. Če se vaš procesni tok ustavi ob praznem izvajanju, to preverite
> sami – preglejte `summary` (ali preštejte datoteke v projektni mapi), namesto da se zanašate na
> odsotnost izjeme. Običajni vzroki so vhodna mapa, ki ni bila prepoznana kot
> zajem, in izdelki, ki so bili preskočeni, ker niso ustrezali prisotnim kameram (npr. sevanjska moč iz kamer, ki podpirajo le RGB
>).

### Funkcije za lažjo uporabo

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### Podprte vrednosti

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Radiometrični izhod (multispektralni postopek LATTICE)

Raven izvoza multispektralnih podatkov (M3C/M3M) v poti obdelave `process` — `reflectance` (privzeto), `radiance`, `sensor-response` ali `all` (vsak ustrezen način za vsako sliko) — se nanaša na nastavitev obdelave **»Radiometrični izhod&quot;**. `configure()` ima za to namensko ključno besedo:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Napredna rešitev — pisanje ključa projekta `"Radiometric output"` prek `custom_settings` — še vedno deluje, vendar ne pozabite, da nadomesti celoten blok nastavitev (glejte opozorilo spodaj):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (privzeta vrednost) deli sevanje kamere z **DAQ-ovim navzdolnim tokom, usklajenim s časovnim žigom**, ki se samodejno izračuna iz posnetega `.daq` (DAQ-U/M/E)**ali izvirnega `.csv` za DAQ-M**, najdenega skupaj s slikovnim gradivom; vsak paket kalibracij za posamezno kamero ali DAQ, ki lokalno manjka, se ob prvi uporabi**samodejno prenese iz AWS**. CLI to prikaže v skladu spreklopnikov tipa izdelka na `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> `custom_settings` **nadomesti** celoten blok izračunanih nastavitev (po zasnovi preskoči druge ključne besede in preverjanje veljavnosti v `configure()`). Ko ga uporabljate, vključite vse ključe `Project Settings`, ki so vam pomembni, kot v zgornjem primeru.

---

## Smart-Connect za kamere LATTICE

Trajne seje v ozadju za strojno opremo v realnem času. Uporabljajo se iste končne točke kot v grafičnem vmesniku, zato je delovanje enako v SDK / CLI / grafičnem vmesniku.

### Ena kamera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

#### Podpis `connect_camera()`

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

#### `CameraSession` Metode

| Metoda | Opis |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Prebere vozlišča GenICam; vrne `{nodes, errors, enums, device}`. |
| `set_settings(**kwargs)` | Zapiše vozlišča po prijaznem imenu (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | Posname **en** posnetek. Vrne seznam z enim elementom, ki vsebuje slovarje metapodatkov posnetka. (Zasnovanje/snemanje več posnetkov je bilo odstranjeno — če potrebujete serijo, v zanki pokličite `capture()`.) |
| `disconnect()` | Sprostitev iz skladišča. Brez učinka, če smo se priključili na že odprto sejo. |

`capture()` nadzor izvoza (isti model kot niz + GUI):

- `processing` / `levels` — `processing="all"` shrani vse ustrezne vrste izvoza; `levels=["raw","radiance"]` shrani le tiste (prevladuje nad `processing`). Oba izpustite za privzeto nastavitev v ozadju.
- `force_daq=True` — shrani dodeljeno vrednost DAQ/DLS kot stranski datotek `.daq` tudi pri zajemu, ki vsebuje le surove podatke, tako da se okvir lahko kasneje ponovno obdela v odbojnost/indeks. Brez učinka, če ni povezan noben DAQ.

### Sinhronizirana matrika — `ArraySession` (Smart-Prep)

`connect_array` je **priporočena izhodiščna točka** za konfiguracije z več kamerami. V ozadju izvaja celoten potek Smart-Prep prek grafičnega vmesnika:

1. **Analiza omrežja** (`/api/camera/array/recommend`) — poišče največjo velikost okvirja, ki ustreza stopnji sim-emit brez izgube okvirjev.
2. **Samodejna izbira stopnje** — `sim-capture-sim-emit`, če omrežna povezava to omogoča; sicer `sim-capture-ftd-stagger` ali `slip-emit-and-capture`.
3. **Samodejno zmanjšanje**— tiho zmanjša velikost okvirja / poveča združevanje, kadar povezava ne more vzdrževati zahtevane ločljivosti.**Ta varnostna mreža ne pokriva agregatnega prekomernega naročanja**: preveč kamer za povezavo ni mogoče popraviti z zmanjšanjem okvirjev — glej [Prekomerno naročanje](#over-subscription-the-per-cam-floor).
4. **PTP omogočen** privzeto — časovni žigi med kamerami so primerljivi z natančnostjo v mikrosekundah.
5. **Samodejna izbira formata pikslov za vsako kamero** — kamere RGB → `BayerRG8`, multispec → `BayerRG12`.
6. **Zagon avtomatske ekspozicije (AE)** — posname trenutno stanje avtomatske ekspozicije vsake kamere, tako da povezava med delovanjem ne ponastavi osvetlitve.
7. **Konfiguracija sprožilca GPIO** — `connect_array` aktivira vse kamere (`TriggerMode=On`, `TriggerSource=Line2`), tako da impulz glavne kamere prek kabla M8 poganja podrejene kamere. To je: ena sama kamera, odprta z `LatticeCamera`, namesto tega deluje samostojno.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### `connect_array()` Podpis

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`force_tier` vrednosti:
- `"sim-capture-sim-emit"` — resnično sočasno (vse kamere se sprožijo na istem robu takta).
- `"sim-capture-ftd-stagger"` — prilagodljivo časovno zamikanje (kamere oddajajo v rahlo zamaknjenih časih, tako da se paketi po vodniku zaporedno prenašajo).
- `"slip-emit-and-capture"` — zaporedno zajemanje po posameznih kamerah (brez časovne sinhronizacije; edina možnost, kadar nobena velikost okvirja ne ustreza simultanemu načinu).

`wire_ceiling_mbps` prevlada nad **trajnim proračunom gostitelja za pasovno širino** v MB/s — to je edina
številka, od katere je odvisna celotna dodelitev polja. Pustite nastavitev na `None`, da se uporabi samodejno zaznana
vrednost. Zmanjšajte jo, ko niz poroča o okvirjih z napako GVSP: samodejna vrednost izhaja
iz oglaševane hitrosti povezave mrežne kartice, ki precenjuje USB-adapterje, ozke PCIe-pasove in
zasedene skupne strukture — ta precenitev se kaže kot okvirji z napako namesto kot
vidno počasne povezave. Vrednost se shrani v bloku zajema polja projekta, zato jo
ponovno odprtje ali kasnejša nastavitev `connect_array` obnovi kot katero koli drugo nastavitev polja.
Glej [Stanje matrike](#array-health--which-subsystem-is-losing-frames).

#### Prekomerna dodelitev (spodnja meja na kamero)

Sim-emit pacing vsaki kameri dodeli delež proračuna za povezavo, varno pred kolizijami, z najnižjo mejo **8 MB/s na kamero**(`per_cam_floor_bps`). Ko `N × floor` preseže zgornjo mejo, varno pred kolizijami, matrika**prekomerno dodeli pasovno širino**— način okvare je izguba paketov GVSP, ne nižja hitrost okvirjev — in ni rešitve, ki bi temeljila na velikosti okvirja:**združevanje in ROI zmanjšujeta število bajtov na okvir, ne pa število bajtov na sekundo**, ki jih primerja agregatna preveritev. Praktične zgornje meje pri polni ločljivosti na gostitelju 1 GbE:**6 kamer pri 1500 MTU, 9 z jumbo okvirji** (`max_cams_collision_safe` v odgovoru analize poroča o zgornji meji za vašo povezavo). Rešitve: manj kamer, jumbo okvirji od začetka do konca ali hitrejša mrežna kartica.

- Odgovori `analyze_array_network()` in `/api/camera/array/connect` vsebujejo `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` in `per_cam_floor_bps`. Če je `oversubscribed` resničen, projekcija **polja fps izniči** (`achievable_fps_max` / `fps_bright` / `fps_dark`), namesto da bi poročala o zavajajoči hitrosti, ki je sicer počasna, a deluje.
- `POST /api/camera/array/connect` sprejema parameter telesa `pin_resolution` (**izključno za HTTP — ni ključni argument za SDK**; `connect_array` ga ne razkriva). Zafiksiranje odstrani varnostno mrežo postopnega zmanjševanja, zato je prekomerno zasedena povezava z nastavitvijo `pin_resolution`**odločno zavrnjena** z napako, ki navaja vse možne rešitve. Brez fiksiranja se povezava nadaljuje s postopkom zmanjševanja, vendar opozori, da zmanjšanje ne more izčistiti agregata.
- Izhod v sili za laboratorijsko delo: nastavite `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` v okolju backenda, da znižate raven zavrnitve na glasno opozorilo – povezavo vzpostavite kljub temu in sprejmete izgubo paketov.

#### Stanje matrike — kateri podsistem izgublja okvirje

`GET /api/camera/array/<array_id>/capability` vsebuje aktivni blok `health` na
priključenem nizu, ki se ponovno ocenjuje v tekočem **10-sekundnem** oknu. Izgubo okvirjev razčleni
na dva vzroka, ki zahtevata nasprotna popravka, namesto ene »nepopolne« stopnje, ki
ne navaja nobenega od njiju:

| Polje | Kaj pomeni | Kateri podsistem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (na serijsko vrsto) | Okvir **je prispel in je bil strukturno poškodovan**— izguba paketov GVSP. |**Omrežje**: zmogljivost povezave, hitrost prenosa, obroč NIC RX, MTU |
| `never_arrived_rate_pct` (na serijsko številko) | Okvir **sploh ni prispel**— kamera ni sprožila ali pa ni nič zapustilo. |**Sprožilec / sinhronizacija**: kabel M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Najslabša hitrost kamere za vsako. | — |
| `per_cam_rate_pct` | Skupna stopnja nepopolnosti na kamero (oba vzroka skupaj). | — |
| `stable_for_seconds` | Kako dolgo je vsaka kamera ostala pod 0,01 %. | — |

Poleg `health` isti zapis navaja tudi številko, na kateri visi celotna dodelitev:

| Polje | Kaj pomeni |
| --- | --- |
| `wire_ceiling_mbps` | Trenutni trajni proračun za pasovno širino gostitelja, MB/s. |
| `wire_ceiling_source` | Izvor te vrednosti, v besedah — npr. `USB-capped 200 MB/s (was theoretical 1062; …)` ali `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, ko ga je nastavil `wire_ceiling_mbps=`. |
| `nic_is_usb` | `true` za USB-ethernetni adapter. |

Za to končno točko ni ovitka SDK — preberite ga neposredno:

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Branje:** vrednost `gvsp_corrupt_rate_pct`, ki ni enaka nič, z vrednostjo `never_arrived_rate_pct` enako 0 pomeni, da
da sta sprožitev in sinhronizacija kabla brezhibni, 100 % izgube pa je na omrežni poti — znižajte
`wire_ceiling_mbps` in se ponovno povežite. Nasprotni vzorec kaže na sinhronizacijski kabel ali
sprožilno linijo.

> **`target_fps` ni vzrok za poškodovane okvirje.** Hitrost GevSCPD se zapiše enkrat ob
> vzpostavitvi povezave, zato znižanje hitrosti sprožilca spremeni delovni cikel, ne pa
> hitrosti sočasnega oddajanja. Izmerjeno 5-kratno zmanjšanje zahtev ni prineslo izboljšanja, medtem ko
> je znižanje zgornje meje povezave s 240 na 200 MB/s pri isti napravi zmanjšalo delež poškodovanih okvirjev s 10,4 % na
> 0,00 %.

> **Samodejno zmanjšanje sredi prenosa ni na voljo v firmware-u TRI032S.** Delujoča matrica tega ne more
> sama popraviti; odklopite in ponovno priklopite, da bo izbirnik časa povezave ponovno načrtoval glede na
> novo zgornjo mejo.

**USB-Ethernetni adapter je s sondo omejen na 200 MB/s**, ne glede na njegovo
oznako: tabela učinkovitosti, ki pretvori hitrost povezave v trajno vrednost, je
izpeljana iz PCIe, USB-mrežna kartica pa oglašuje svojo hitrost ethernetne povezave, pri čemer je omejena z
USB-vodilom in njegovim gonilnikom. Omejitev je absolutna, ne relativna – USB 1 GbE adapter
dosega ~80 MB/s in to omejitev ne vpliva nanj.

#### `ArraySession` Metode

| Metoda | Opis |
| --- | --- |
| `status(timeout=10.0)` | V živo `{fps, ptp, frame_count, last_error, …}`. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | Ena sinhronizirana skupina zajemov. Vrne `CaptureResult` (seznam slovarjev okvirjev + `.skipped`). Nadzor izvoza spodaj. |
| `capture(..., smart=True)` | **Pametno zajemanje** — počaka, da se AE stabilizira na vseh kamerah, nato pa sproži zajem. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Najhitrejše zajemanje: samo surovi podatki + dodeljena vrednost DAQ (+ prosti kombinirani indeks). Odraža gumb »Najhitrejše zajemanje« v grafičnem vmesniku. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Enkratno / neprekinjeno / intervalno zajemanje v enem omejenem ciklu. Vrne `list[CaptureResult]`.**Zahteva `count` in/ali `duration_s`** da se zaključi (SDK nima Ctrl+C). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Začni snemati pogled kombiniranega indeksa v živo v video/GIF → `RecorderHandle`. En kompozitni snemalnik na matriko. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Začni serijsko snemanje surovih Bayerjevih podatkov z visoko hitrostjo snemanja → `RecorderHandle`. Ponovno obdelaj v načinu brez povezave s programom `build_video()`. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Offline ponovna obdelava shranjene serije surovih posnetkov v kalibrirane videoposnetke. Blokira do konca (`wait=True`) in vrne `{outputs, errors, combined}`. |
| `build_video_status(job_id, timeout=15.0)` | Preveri stanje naloge za izdelavo v offline načinu: `{running, result, error, burst_dir}`. |
| `disconnect()` | Sprosti celoten niz. |

`capture()` nadzor izvoza (isti končni točki, kot ju uporabljata GUI/CLI):

- `processing` / `levels` — `processing="all"` (ali `levels=["raw","radiance",…]`) shrani vsako ustrezno vrsto izvoza za vsako kamero; ena sama vrednost `processing` shrani le to raven.
- `aligned=True` — vsak ne-surov izvoz člana poravna s [profilom poravnave](#array-alignment) (so-registriran); surovi podatki ostanejo neporavnani, vendar nosijo transformacijo v metapodatkih. Če niz nima profila, se vrne na neporavnano stanje (z opozorilom, ki se prikaže v `alignment` rezultata).
- `render_index=False` — preskoči prekrivni sloj indeksa vegetacije za posamezno kamero; privzeto ga prikaže tam, kjer je konfiguriran.
- `force_daq=True` — shrani dodeljeno vrednost DAQ/DLS kot stranski datotek `.daq`, tudi če nobena izbrana raven tega ne potrebuje.

**TIFF stiskanje (HTTP-samo gumb):**`ArraySession.capture()` ne pošilja ključa `compression`, zato velja privzeta nastavitev backenda — `POST /api/camera/array/capture` bere parameter `compression` parametra telesa, `"deflate"` privzeto (brezizgubna stiskanje zlib L1 + horizontalni prediktor, ~4,1 MB na sliko v polni ločljivosti). `"none"` piše nekomprimirano (~6,3 MB/sliko) s**~5× hitrejšim zapisovanjem** — oba sta brez izgube in se ob uvozu berejo enako. SDK za to ne razkriva nobenega kwarga; izhod v sili sta `chloros-cli lattice array-capture --compression none` ali surovi HTTP. DEFLATE prav tako zadržuje GIL Python, zato se stisnjeno zapisovanje ne izvaja vzporedno v niti za zapisovanje posameznih kamer — za neprekinjeno zajemanje v polni ločljivosti z 8 kamerami pri hitrosti senzorja je potreben `compression: "none"`. Podrobnosti: [CLI Referenca → zajem v nizu](cli-reference.md).**Prepisovanje izvoza po posameznih elementih (samo HTTP):**isti končni točki sprejema tudi `exclude_serials` (seznam — izključitev članov iz shranjenega niza; niz se še vedno sproži kot ena sinhronizirana skupina, izključeni člani pa se vrnejo v `excluded`), `serial_levels` (`{serial: [level tokens]}` prepisovanja na ravni posamezne kamere), in `serial_index` (`{serial: bool}` prepisovanje indeksnih prekrivanj na kamero). To so parametri telesa, ki so enakovredni tistim v grafičnem vmesniku,**še niso pa SDK ključni argumenti**; člani, ki niso prisotni v zemljevidih, se vrnejo na vrednosti `levels` / `render_index`, ki veljajo za celoten niz.

##### Pregled preskočenih kamer — `CaptureResult.skipped`

`ArraySession.capture()` vrne `CaptureResult`, ki je podrazred `list`: iteriraj, indeksiraj, `len()`-aj — vsi obstoječi vzorci še naprej delujejo. Nova koda lahko pregleda atribut `.skipped`, da ugotovi, katere kamere so bile izključene in zakaj. Najpogostejši primer so kamere RGB v matrici z mešanimi filtri, ko zahtevate `processing="radiance"` ali `"reflectance"` — sevanje na posamezni Bayerjevi celici je za širokopasovni senzor brez pomena, zato backend te kamere preskoči, namesto da bi ustvarjal nesmiselne podatke.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Znakovni razlogi sledijo vzorcu `<level>-not-applicable-to-rgb-cam` (en vnos na preskočeno raven, vsak vsebuje `level`). Preskoki, specifični za odbojnost, so `reflectance-skipped-no-fresh-dls` (ni na voljo nove meritve navzdol usmerjenega sevanja), `reflectance-skipped-bound-daq-unavailable (…)` (vezanega DAQ-ja ni bilo mogoče doseči) in `dls-uncalibrated-band-<nm>` — pas se večinoma nahaja zunaj radiometrično kalibriranega območja svetlobnega senzorja DAQ (~374–974 nm), zato je absolutna delitev odbojnosti na podlagi DAQ zavrnjena in se okvir glasno preklopi na odziv senzorja. Med serijskimi izdelki to sproži le F988; podprta pot te kamere je delovni tok z odbojnostno ploščo.

Ravni `processing`:

| Raven | Izhod |
| --- | --- |
| `"raw"` | Enokanalni Bayer (mono kamere: en pas) neposredno iz senzorja. |
| `"debayered"` *(privzeto SDK)* | 3-kanalni BGR prek bilinearnega demosaika (črno-bele kamere: 1-kanalna siva lestvica). |
| `"radiance"` | float32 W/m²/sr/nm prek celotne radiometrične verige. Samo multispektralne — kamere RGB so preskočene. |
| `"reflectance"` | uint16 0..32768 (združljivo s Pix4D); zahteva povezavo z DAQ v realnem času za absolutno referenco. Samo multispektralno. |
| `"display"` | Celotna veriga, ki ustreza predogledu v grafičnem vmesniku (CCM + WB + gama v skladu s profilom kamere). |
| `"all"` | **Ena datoteka na ustrezno raven** za vsako kamero (v skladu s privzetim nastavitvam GUI »Capture All« / CLI privzeta nastavitev). Vrnjena datoteka `CaptureResult` nato vsebuje en slovar okvirja na `(cam, level)`, z ravnjo v vsakem slovarju; neuporabne ravni se pojavijo v `.skipped`. Odčitek DAQ, uporabljen za kateri koli okvir odbojnosti, se shrani kot stranski datotek `.daq`. |

> **Opomba — privzeta nastavitev se razlikuje od CLI.** Privzeta vrednost za `ArraySession.capture()` je `processing="debayered"`; privzeta vrednost za ukaz `chloros-cli lattice array-capture` je `processing="all"`. Izrecno prenesite `processing="all"` iz SDK, da se odraža večstopenjsko shranjevanje CLI/GUI.

### Načini zajemanja in snemalniki

Površina matrike odraža panel zajemanja v grafičnem vmesniku: načini zaklopa Single / neprekinjen / intervalni / najhitrejši način zaklopa, poleg tega pa še dva snemalnika (kompozitni video v živo in surovi zaporedni posnetki → ponovna obdelava offline).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**je zanka »Neprekinjeno/Intervalno« za SDK. Ker ni `Ctrl+C`, ki bi jo prekinil iz skripta,**morate** posredovati `count` in/ali `duration_s` (zastane, ko je dosežen kateri koli od njiju). `interval_s` se meri od začetka vsakega prehoda (v skladu z grafičnim vmesnikom). Preostali kwargi se neposredno posredujejo v `capture()`.
- **`record`** je *za namene spremljanja*: zajema kombinirani indeks v realnem času, kot se prikazuje, zato mora biti kombinirani tok odprt, da se okvirji lahko shranijo. En snemalnik kompozita na matriko (izvzame izjemo, če eden že teče).
- **`burst` → `build_video`** je *za analizo*: `burst` zapiše surove slike + manifest za vsako sliko + en `.daq` za vsako posamezno DLS-odčitavanje pod `<output>/bursts/<base>/` pri polni hitrosti zajemne zanke (brez verige, brez exiftool, brez živega predogleda). `build_video` časovno uskladi vsak posnetek z najbližjim `.daq` in ponovno izvede verigo svetilnosti/odbojnosti/indeksa uvoznega procesa. `products` je seznam `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (privzeto: kombinirani indeks). `burst().stop()` prav tako samodejno sproži izgradnjo kombiniranegaindeks, ki se v končnem rezultatu vrne kot `build_job`.

#### `RecorderHandle`

Vrnejo ga `ArraySession.record()` in `ArraySession.burst()`. Uporabite ga kot upravitelja konteksta za samodejno ustavitev ob izhodu iz obsega ali ga upravljajte ročno.

| Član | Opis |
| --- | --- |
| `job_id` | ID naloge v ozadju (str). |
| `kind` | `"composite"` (iz `record`) ali `"raw"` (iz `burst`). |
| `start_stats` | Slovar, ki ga vrne klic `start`. |
| `result` | `None` med izvajanjem; končni slovar rezultatov ob ustavitvi. |
| `stats(timeout=10.0)` | Statistični podatki o delovanju v realnem času (zapisani okvirji, doseženi fps, pretečen čas). |
| `stop(timeout=60.0)` | Ustavi snemalnik; vrne in shrani končni rezultat v predpomnilnik. Idempotentno (drugi klic vrne rezultat iz predpomnilnika). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### Priključitev na že povezan niz — `attach_array`

Če je niz že vzpostavljen (ga je odprl grafični vmesnik ali ga je prej poklicala seansa SDK seja poklicala `connect_array`), uporabite `attach_array`, da pridobite ročaj do njega, namesto da se ponovno povežete. `connect_array` v takšni situaciji vedno vrne napako »Kamera  je<sn> že v nizu<id>«, saj pošiljanje zahtevka POST za `/array/connect` za člana v bazenu ni idempotentno; `attach_array` prebere `/api/camera/array/list` in primerja po array_id ali serijske številke.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Vzorec: skripti SDK, ki delujejo-najemnikom z namiznim grafičnim vmesnikom, naj najprej poskusijo z `attach_array` in se vrnejo na `connect_array`, če v skupini še ni nobenega niza.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Pomembno — izhod iz context-managerja DEJANSKO prekine povezavo.**`ArraySession.disconnect()` vedno pošlje zahtevo POST na `/array/disconnect`; ni varovala »attached-not-owned«, kot je to pri `CameraSession` / `DAQSensorSession`. Če si delite prostor z grafičnim vmesnikom (GUI) in ne želite razgraditi matrike ob izhodu iz obsega,**ne uporabljajte bloka `with`** — ročaj shranite v običajno spremenljivko in preskočite eksplicitni `disconnect()`:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Pomočnik za analizo omrežja

Koristno pred odprtjem matrike — predvideva, ali bodo vaše predlagane nastavitve ustrezale:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

`status` je eden izmed `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (v nasprotnem primeru `error`). `auto_capped_fps` pomeni, da zahtevana ločljivost ustreza RX-obroču le pri omejeni hitrosti sprožanja — ohranite ločljivost in prenesite `target_fps=result["recommended"]["recommended_target_fps"]` na `connect_array` (glej [Primer 6](#6-preverjanje-zmogljivosti-pred-priključitvijo-niza-4-kamer)).

**Kako razumeti projekcijo** (enak model kot v oknu »Nastavitve niza« v grafičnem vmesniku):

- **Serija posnetkov (`frame_bytes_total`) se sešteva po posameznih kamerah v dejanskem formatu pikslov vsake kamere.**Mono**M3M**kamere prenašajo Mono12 (2 B/px) ne glede na vrednost `pixel_format`, ki jo posredujete, zato je slika v polni ločljivosti s 4 kamerami**~25 MB** pri treh mono kamerah, ne pa ~12,6 MB, kot bi to izhajalo iz predpostavke, da so vsi 8-bitni. Backend prepozna format vsake kamere na podlagi njenega modela.
- **Prepustnost (`burst_fits_nic_ring`) upošteva praznjenje**, ne pa razmerja med celotnim izbruhom in obročem: sim-emit je primeren, ko gostitelj prazni sprejemni obroč hitreje, kot ga kamere polnijo. 10G gostitelj + 1 GbE kamere**dopuščajo** polno ločljivost, tudi če burst presega obroč; 1 GbE gostitelj blokira (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` je konzervativna zgornja meja za serijsko pridobivanje** — `max(readout+emit, N×emit)` z oddajo na kamero, omejeno na 1 GbE povezavo kamere, neodvisno od osvetlitve. Npr. ~2,8 sličic na sekundo za niz 4 kamer s polno ločljivostjo in 12-bitno globino barv (kar ustreza izmerjenim ~2,7–3,0). Celoten model: [CLI Referenca → Model sličic na sekundo in serijskega zajema za niz](cli-reference.md#array-fps--burst-model).
- **Prekomerna dodelitev (`oversubscribed: true`) pomeni, da spodnja meja N × na kamero presega zgornjo mejo, varno pred kolizijami** — polja fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) kažejo vrednost 0, avtomatsko zmanjšanje/združevanje pa tega ne more odpraviti (zmanjšuje število bajtov na okvir, ne pa števila bajtov na sekundo). Rešitve so manj kamer, jumbo okvirji ali hitrejša mrežna kartica; `max_cams_collision_safe` poroča o zgornji meji (6 kamer s polno ločljivostjo na 1 GbE @ 1500 MTU, 9 z jumbo okvirji). Odgovor vsebuje tudi `aggregate_demand_bps`, `collision_safe_ceiling_bps` in `per_cam_floor_bps` (8 MB/s). Glej [Prekomerno naročanje](#over-subscription-the-per-cam-floor).

### Odkrivanje in prikaz seznama

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

Nizi LATTICE v ozadju izvajajo neprekinjeno avtomatsko ostrenje (AE), takoj ko so povezani, vendar pa novo usmerjena scena potrebuje nekaj časa, da se stabilizira. **Smart-capture** je priročna rešitev: preveri osvetlitev vsake kamere, počaka, dokler se niz v celotnem oknu ne stabilizira, nato pa sproži zajem. To je enakovredno grafičnemu vmesniku: gumb »smart« capture v namizni aplikaciji pokliče isto končno točko v ozadju.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Pri upravljanju prek `ChlorosProject` (naslednji oddelek), imate na voljo več nastavitev:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Pravilo pametne avtomatske ekspozicije (smart-AE) je privzeto konzervativno. Za zahtevno radiometrično delo zategnite `exposure_tolerance_pct`; razširite ga za hitro spreminjajoče se prizore, kjer želite le »dovolj približno“.

---

## Seje senzorjev DAQ

Trajni bazen v ozadju za spektralne senzorje (DAQ-U prek USB, DAQ-M prek BLE, DAQ-E prek Etherneta). Odraža površino kamere: pametno zaznavanje, ponovna uporaba bazena, idempotentno priključevanje.

### Pametno zaznavanje (brez konfiguracije)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Prednost: Ethernet → BLE → USB. Posredujte kateri koli eksplicitni namig, da se transport fiksira.

### Fiksiran transport

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### Metode `DAQSensorSession`

| Metoda | Opis |
| --- | --- |
| `status(timeout=10.0)` | Povzetek vnosa v zbirko (stanje pretakanja/snemanja, valovno območje, kalibracijski SHA, čas integracije, frame_avg, stanje AE). |
| `latest(n=1, timeout=10.0)` | Vrne do N najboljslike spektra. |
| `stream_start()` / `stream_stop()` | Nadaljuj / zaustavi pretakanje (ročaj ostane odprt). |
| `record_start(output_dir=None, device_name=None)` | Začni snemanje datoteke .daq. Vrne pot do datoteke. Zavrne se za DAQ-U/M brez AWS-kalibracijskega paketa (DAQ-E je izvzet). |
| `record_stop()` | Ustavi snemanje. Vrne`{path, rows}`. |
| `disconnect()` | Sprostitev iz skladišča. Brez učinka za priključene ročaje, ki niso v lasti. |

> **Profili za popravek kapice (`cap_id`) niso gumb SDK.** `connect_daq_sensor()` / `DAQSensorSession` ne razkrivata nobenega parametra `cap_id` ali metode `set_cap`. Izberite profil korekcije omejitve flote prek CLI (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) ali prek poti `/api/daq` HTTP (`/api/daq/connect` in `/api/daq/<id>/cap-id` sprejemata `cap_id`).

### Odkrivanje — iskanje naslova za povezavo

`discover_daq_sensors()` pregleduje USB / BLE / ETH za senzorje, ki jih *bi lahko* odprli. To je DAQ-jev ustreznik `discover_lattice_cameras()` in edini način za pridobitev **DAQ-M&#x27;BLE MAC** — DAQ-E ima ime gostitelja, DAQ-U pa COM-vrata, MAC-naslov pa ni natisnjen na napravi niti ga operacijski sistem ne navaja.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Polje | Opis |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM-vrata / BLE MAC / ime gostitelja — prenesi v `connect_daq_sensor` kot `port=` / `mac=` / `eth_host=`. |
| `display` | Berljiva oznaka. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` ali `None` za vrata, ki jih skeniranje ne more identificirati (serijske USB-prilagodnike ni mogoče razlikovati brez sonde, zato se neznani elementi prikažejo namesto da bi bili skriti). |
| `extra` | Podrobnosti po posameznem prenosu (oglaševano ime BLE, proizvajalec USB, DAQ-E ip/fw/…). Prazne vrednosti se izpustijo. |

| Parameter | Privzeto | Opis |
| --- | --- | --- |
| `transports` | vse tri | Zaporedje (ali niz v formatu CSV), ki omejuje skeniranje. Vredno je to podati, če veste, kaj želite — BLE je počasnejši del. |
| `scan_timeout` | 5 | Okno skeniranja po posameznem prenosu v sekundah; backend omeji vrednost na 1–20. |
| `timeout` | 60.0 | HTTP zgornja meja za celoten klic (kot drugje v SDK). |
| `auto_start_backend` | `True` | Zaženi lokalno ozadje, če še nobeno ne teče. Nikoli se ne zažene za oddaljeni `backend_url`. |

> **Senzorji, ki so v bazenu že odprti, se ne prikažejo.** Povezana BLE-periferija preneha z oglaševanjem, odprtega COM-vrata pa ni mogoče preveriti, zato iskanje prikaže le tisto, kar je *na voljo za povezavo*. Takoj po povezavi je pričakovati prazen rezultat — za tisto, kar že imate, uporabite `list_daq_sensors()`. Prevozi, katerih skeniranje se ne more izvesti (ni nameščenih bleak / zeroconf), se preskočijo, namesto da bi sprožili napako, zato naprava brez Bluetootha še vedno dobi odgovore za USB in ETH.

### Seznam

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Soobstoj z grafičnim vmesnikom / CLI

Če ima grafični vmesnik že odprt senzor, klic `connect_daq_sensor(port="COM3")` iz Python vrne ročaj, označen z `already_connected=True`. `disconnect()` seje je nato brez učinka, zato vaš skript SDK ob izhodu iz programa ne odtrga senzorja iz grafičnega vmesnika.

### Razredi za neposredno upravljanje strojne opreme (brez ozadja)

`daq_sdk` je ponovno izvožen s strani `chloros_sdk`, zato lahko senzorje upravljate od začetka do konca znotraj procesa tudi brez ozadja:

> **Razpoložljivost:**`daq_sdk` je priložen namestitvi Chloros za namizne računalnike,**ne** s paketom PyPI — `pip install chloros-sdk` vam zagotavlja `lattice_sdk`, vendar izpušča `chloros_sdk.DAQ_AVAILABLE == False`. Preverite to oznako, preden uporabite te razrede; na gostitelju, ki uporablja izključno pip, senzor upravljajte prek [`connect_daq_sensor()`](#daq-sensor-sessions), ki ne potrebuje lokalnih transportnih knjižnic.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

Raje uporabite pot »smart-connect« (`connect_daq_sensor`), če želite deljeno lastništvo z grafičnim uporabniškim vmesnikom; neposredne razrede uporabite za skripte brez grafičnega vmesnika, ki imajo senzor v izključni lasti.

---

## Avtomatizacija projektov — `ChlorosProject`

Shranjeni projekt Chloros je mapa, ki vsebuje `cameras.json` + `sensors.json` + `project.json`. `open_project` naloži manifest, `connect_all` pa vsako shranjeno napravo poveže z internetom z njenimi shranjenimi nastavitvami — v enakem stanju strojne opreme, kot bi ga ustvaril grafični uporabniški vmesnik.

### Minimalni primer

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

Ali kot upravitelj konteksta:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### Metode `ChlorosProject`

| Metoda | Opis |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Odkrije in poveže vsako shranjeno napravo. Vrne poročilo o povezavi po razredih. Uporabi zagnan backend, če ta posluša na `127.0.0.1:5000`; v nasprotnem primeru se tiho preklopi na neposredno (brez backenda) `lattice_sdk` upravljanje naprav — nikoli ne ustvari backenda. |
| `disconnect_all()` | Prekini vse. |
| `capture_all(output_dir=".")` | En okvir iz vsake kamere + niz + spekter iz vsakega senzorja. |
| `stream(camera, overlays=False, fps=10.0)` | Generator, ki ustvarja BGR `numpy` okvirje iz imenovane kamere (ali niza). `overlays=False` je neposredna zanka zajemanja `lattice_sdk` (nizi dajejo slovarje `{serial: frame}`). `overlays=True` usmerja prek `ChlorosLocal.camera_stream()` → MJPEG-podatkovni tok `/api/camera/<serial>/stream-annotated` zadnjega dela, pri čemer se shranjeni blok kamere `ui.overlay` posreduje kot parametri poizvedbe. Zahteva način zadnjega dela in **samostojno kamero**: kamera v neposrednem načinu sproži `RuntimeError` (zadnji del ne more prevzeti kamere, ki pripada temu procesu) , niz pa sproži `NotImplementedError` (prekriva kompozit po kameri — pretaka element po imenu). Enkratni ekvivalent: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Izvedi poravnavo za vsak trenutno priključen niz. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Izvedi kalibracijski/indeksni postopek na slikah projekta (zavije `ChlorosLocal.process`; ti štirje so **edini** sprejeti ključni argumenti — `indices=` itd. sprožijo napako `TypeError`; indekse nastavite prek `ChlorosLocal.configure()`). Lenično ustvari `ChlorosLocal()`, ki samodejno zažene backend. |

Atributi:
- `proj.cameras` — `Dict[str, CameraHandle]` indeksiran po imenu IN serijski številki.
- `proj.arrays` — `Dict[str, ArrayHandle]`, indeksiran po imenu in array_id.
- `proj.sensors` — `Dict[str, SensorHandle]`, indeksiran po imenu in slot_id.
- `proj.config` — `project.json["config"]` slovar.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Ravni obdelave.** `capture()`, `grab()` in `frame_stream()` vsi uporabljajo isti token `processing`
, veriga pa je kumulativna — vsaka stopnja izvaja vse, kar je nad njo:

| Stopnja | Izhod | Opombe |
| --- | --- | --- |
| `raw` | 1-kanalni Bayer, nativni za senzor | Brez demosaika. Prekrivni sloji na tej stopnji niso na voljo. |
| `debayered` | 3-kanalni BGR (**privzeto**) | Bilinearna demosaika. Edina raven, ki deluje brez načina backend. |
| `radiance` | float32, W/m²/sr/nm | Celotna radiometrična veriga: demosaik + 3×3 ločevanje (multispec) + DSNU + flat-field + NIST lestvica, pri čemer sta osvetlitev × ojačanje izločena, tako da so vrednosti absolutne. |
| `reflectance` | uint16, 32768 = 1,0 | Sevanje, deljeno z navzdolnim obsevanjem (ρ = π·L/E). Potrebna je odčitka DLS/DAQ — glej opombo spodaj. |
| `display` | 8-bitni sRGB-podobni | GUI-ekvivalentno upodabljanje: CCM + bela bilanca + gama prek aktivnega barvnega profila kamere. |

Vse razen `debayered` zahteva način backend; kamera v neposrednem načinu sproži
`NotImplementedError`. `reflectance` potrebuje uporabno odčitano vrednost navzdolnega toka — končna točka okvirja samodejno potegne
zbrano DAQ v režo DLS kamere, vendar brez vezanega DAQ-ja veriga zavrne
izhod odbojnosti in pošteno označi znižanje v vrnjenih metapodatkih, namesto da bi tiho
vrnila slabši izdelek.

> **Lestvica DN odbojnosti — je ne vpisujte trdno.** Odbojnost LATTICE uporablja `32768` = ρ 1,0 in označi
> XMP `Chloros:PixelScale=32768`; odbojnost Survey3 uporablja `65535` = ρ 1,0 in ne vsebuje
> oznak `Chloros:*`. Preberite oznako in delite z njo. Opredeljena je v domeni uint16, zato ostane
> `32768` za vsak format, ki spreminja merilo (16-bitni TIFF, 8-bitni PNG/JPG, 32-bitni odstotek) — najprej normalizirajte
> shranjeni podatkovni tip nazaj na uint16 (×257 iz 8-bitnega, ×65535 iz float). Edina izjema:
> zajem iz 8-bitnega vira, zapisan kot 8-bitni TIFF, je *odrezan*, ne pa prilagojen, zato ga nobena lestvica ne opisuje
> — Chloros v tem primeru v celoti izpusti `PixelScale` in MicaSense-tuple. Manjkajočo
> oznako v datoteki odbojnosti LATTICE obravnavajte kot »brez veljavne lestvice«, ne kot privzeto.

> **EXIF se prenese v izvoz.** `process()` kopira blok GPS izvirnega posnetka
> **in njegov ExifIFD** prenese na vsak izdelek, zato izvozi vsebujejo `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` in `CameraSerialNumber` ter
> georeferenciranje. `FocalLength` je tisto, na podlagi česar Pix4D izračuna razdaljo med vzorci na tleh – brez njega
> rekonstrukcija pade v povsem napačno merilo (v izmerjenem primeru se je lokacija velikosti 411 m
> spremenila v lokacijo velikosti 47,8 km). Kopija namerno ni `-all:all`: strukturne oznake IFD0 prekinejo
> izhod LATTICE, `ExifImageWidth`/`Height` pa sta izključena, ker opisujeta
> zajem vira in ne izvožene rastrske slike.

Podoznake v fazi zajema (veljajo za radiometrične ravni — `radiance`, `reflectance`, `display`):

| Oznaka | Privzeto | Pomen |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + flat-field + 3x3 unmix + radiometrična lestvica NIST. |
| `apply_white_balance` | `True` | WB LUT. DLS-upošteva, če je DAQ povezan s kamero. |
| `apply_index` | `False` | Ocena indeksa vegetacije. |
| `index_expression` | `None` | Prepis formule. Če ni prazno → samodejno omogoči indeks. |
| `annotated` | `False` | Prekrivanje okraskov grafičnega vmesnika (zebra/mreža/izpostavitev). Ni na voljo za `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Vrsta vrnjenega vrednost je `CapturePathMap`, ne `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` je `Dict[str, Union[str, List[str]]]`: enostopenjski
> `processing` vsaki serijski številki dodeli eno pot, medtem ko večstopenjska (`"all"` ali
> eksplicitni seznam `levels`) pa mu dodeli **urejen seznam** vseh izdelkov, shranjenih za to
> kamero. Kombinirana kompozicija v živo, če se ta predvaja v živo, prispe pod dodatnim
> ključem `"combined"` in ne pod serijsko številko. Koda, ki predpostavlja `str`, se pokvari pri
> obliki seznama, ne da bi kateri koli preverjevalec tipov ugovarjal – opomba je navajala `Dict[str, str]`
> še nekaj časa po izdaji seznama, zato ta alias obstaja. Normalizirajte
> obliko, če želite ravno obliko:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Poravnava niza

`ArrayHandle` razkriva celotno površino poravnave. Profil je privzeto veljaven le za sejo — za trajno shranjevanje izrecno pokličite `export_alignment()`.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Poravnava ob povezovanju

`connect_all(align=...)` lahko samodejno poravna vsak niz ob :

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Če ni drugače določeno, se uporabi `project.json["config"]["auto_align_on_connect"]`.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## Neposredna strojna oprema (brez backenda)

Če želite popolno neodvisnost od backenda (CI, roboti brez grafičnega vmesnika, vgrajeni sistemi), neposredno uvozite `lattice_sdk` in `daq_sdk` — oba sta ponovno izvožena s strani `chloros_sdk`. Opozorilo glede `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`: `lattice_sdk` je v paketu PyPI (vendar potrebuje prisotnost izvedbenega okolja Arena SDK), medtem ko je `daq_sdk` na voljo le v namestitvi za namizne računalnike.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Prednastavitve in sprožilec

Tri od štirih prednastavitev so **free-run**: kamera neprekinjeno osvetljuje,
`capture()` vrne naslednji kader. `triggered` je izjema – kamero
pripravi na signal na liniji 2, zato ne zajame ničesar, dokler ta ne prispe.

| Prednastavitev | Sprožilec | Uporabite, ko |
| --- | --- | --- |
| `default` | prosto delovanje | splošna raba |
| `high_speed` | prosto delovanje | 8-bitno, omejitev 60 fps, kratka osvetlitev |
| `high_quality` | prost tek | 12-bitno, brez omejitve fps — običajna izbira za fotografije |
| `triggered` | **vklopljeno, linija 2** | kamera je priključena na sinhronizacijski kabel M8 in jo sproži nekaj drugega |

Če izberete `triggered` (ali sami nastavite `trigger_mode="On"` sami), pri čemer nič
ne aktivira linije 2, bo pri vsakem `capture()` prišlo do časovne omejitve — kar je pravilno, saj ste
kamero, naj počaka. SDK to pojasni, ko se to zgodi; glejte
[SC_ERR_TIMEOUT med zajemanjem](#direct-hardware-backend-free).

> **Opomba — »GVSP probe« / sporočila `SC_ERR_TIMEOUT -1011` ob povezavi niso napake.**&gt; Ob vzpostavitvi povezave SDK poskuša dogovoriti**jumbo okvirje** (9000-bajtni GVSP-paketi) za večjo prepustnost. Na neposredni povezavi med omrežnimi karticami (npr. lokalni naslov `169.254.x.x`) omrežje običajno ne morene more prenašati jumbo okvirjev, zato ta preizkus poteče in v dnevnik zabeleži vrstice, kot so:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> To je **predvideni nadomestni postopek**: SDK samodejno preklopi na standardne pakete velikosti 1500 bajtovte pakete, kamera pa se še naprej normalno povezuje (vrstice `[chunk-enable …]`, ki sledijo, so del običajnega zaporedja povezovanja). Snemanje še vedno deluje.
>
> To preizkušnjo lahko preskočite, vendar **ni zgolj za utišanje dnevnika — izklopi jumbo okvirje.** Kamera odgovarja na pingove »Don&#x27;t-Fragment« le do 1500 bajtov, ne glede na to, kako dobro deluje vaše omrežje, zato sam ping test nikoli ne more odkriti jumbo okvirjev; to preizkušanje je edino, kar to lahko stori. Če jo onemogočite, bo kamera v vsakem omrežju za vedno pošiljala standardne pakete velikosti 1500 bajtov:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> To se splača le v omrežju, za katero *veste*, da ne podpira jumbo paketov, kjer to prihrani približno sekundo časa povezovanja na kamero. Ker gre za resnično kompromisno rešitev in ne le kozmetično spremembo, SDK to zdaj tudi navaja, ko ga uporabite:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Pustite ga pri miru, razen če imate razlog.** Če ostane omogočen, se pri vsaki povezavi ponovno izmeri omrežje, ki ga dejansko imate: priključite se na stikalo, ki podpira jumbo pakete, in naslednja povezava bo sama prepoznala jumbo pakete, brez kakršne koli konfiguracije in brez ponovnega zagona.
>
> Če *želite* jumbo prepustnost, omogočite jumbo od začetka do konca (MTU mrežne kartice 9000 + stikalo, ki jih prepušča), ali pa jo fiksirajte z `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`, ko veste, da povezava to podpira — čeprav je bolje uporabiti `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` za posamezne ukaze namesto trajne nastavitve, saj fiksirana velikost preskoči preizkus in prepreči prilagajanje omrežju pred njo. **Vsaka** naprava na poti mora podpirati jumbo pakete — vključno z vsakim PoE razdelilnikom ali injektorjem, kar je običajni razlog, da sicer jumbo-združljiva konfiguracija ne more prenašati teh paketov.

> **`SC_ERR_TIMEOUT -1011` med `capture()` / `grab*()` je drugačen problem – ta je prava napaka.**&gt; Zgornja opomba se nanaša le na `-1011`, ki ga je zabeležila**sonda za vzpostavitev povezave****. Enaka napaka, ki se pojavi pri**zajemanju**, pomeni, da je kamera pravilno povezana, vendar ne pošilja nobenih slik:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Razkriva jo kamera, katere *nadzorni* kanal deluje pravilno — odkrivanje deluje, nastavitve in zapisi `[chunk-enable …]` so uspešni — medtem ko *vsak* posnetek preseže časovno omejitev.
>
> **Običajni vzrok je, da je kamera nastavljena na sprožilec prek strojne opreme.** Pri `trigger_mode="On"` in `trigger_source="Line2"` kamera ne oddaja ničesar, dokler na sinhronizacijskem kablu M8 ne pride do električnega impulza. Če nimate kabla, ki bi vodil to linijo, vsak poskus zajema čaka v nedogled. Kamera ni pokvarjena in omrežje deluje pravilno – počne natanko to, kar ji je bilo naročeno.
>
> `CameraSettings()` ter `default` / `high_speed` / `high_quality` so nastavljeni na samostojno delovanje, pri čemer zajem, ki med aktiviranjem preseže časovno omejitev, prikaže pojasnilo namesto samega `-1011`. `PRESETS["triggered"]` po zasnovi aktivira Line2.
>
> Če želite katero koli kamero prisiliti v samostojno delovanje:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Če se s kodo `trigger_mode="Off"` še vedno izteče časovni limit, kamera dejansko ne pošilja podatkov – pošljite nam dnevnik in kodo `ip link show`.

#### Barvni profili (RGB predogled v živo) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` izbere barvni profil zaslona za **predogled v živo** na kamerah RGB kamerah (multispec kamere to nastavitev prezrejo):

| Profil | Pomen |
| --- | --- |
| `raw` | Popolnoma preskoči radiometrično verigo. |
| `linear` | DSNU + flat + WB, brez CCM, brez gama. |
| `natural` | Linearni + izmerjeni CCM + sRGB gama, samo s poceni zaključkom (izglajevanje barvne globine + desaturacija svetlih delov) — realistična privzeta nastavitev. |
| `enhanced` | `natural` plus popolna obdelava s parnostjo vozlišč (odstranjevanje barvnih robov, živahnost, lokalni kontrast CLAHE). Bogatejši videz ob približno **dvojnih stroških obdelave na posamezni sliki**, kar pomeni nižjo hitrost prikazovanja v živo. |
| `custom_temp` | `natural`, vendar je WB fiksiran na `custom_cct_k` Kelvin (DLS se ne upošteva; omejen na 2000–10000 K na strani backendastrani). |

Profil je **samo za predogled v živo**, ki omogoča hitrost in izbiro videza: shranjeni posnetki vedno dobijo polno, bogato končno obdelavo ne glede na izbrani profil, zato izbira `natural` za pridobitev časa okvirja ne zniža kakovosti tistega, kar se shrani na disk. Neznani profil poveča `ValueError`; ko je backend chloros dosegljiv, se se sprememba pošlje tudi vanj, tako da jo naslednji predogledni okvir upošteva (uporabniki direct-SDK brez backenda še vedno dobijo spremembo nastavitev).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Mono (M3M) kamere in `Calibration`

Mono **M3M** kamera (`M3M-<lens>-F<wavelength>`) je enopasovna: ena ravnina s sivimi odtenki, brez Bayerjevega mozaika, brez 3×3 matrike spektralnega prekrivanja. `Calibration` jo prepozna in prikaže zastavico `is_mono`. Odbojnost se še vedno uporablja kotpasovni radiometrični zemljevid (razmešavanje je identitetna matrika), vendar večpasovna matematika na eni kameri prinaša smiselne rezultate namesto nesmiselnih:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Za izračun indeksa vegetacije iz mono opreme združite več kamer M3M z različnimi valovnimi dolžinami v poravnani večpasovni niz (glej [Poravnava nizaignment](#array-alignment)) in izračunajte indeks za celoten niz namesto za eno kamero.

DAQ v neposrednem načinu:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **`apply_sensor_settings` sprejete ključe**— natančno `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E; opuščeno v korist `cap_id`), `filter_model` (DAQ-M) ter `cap_id` (vse vrste DAQ; `None`/`""`/`"none"` = goli senzor, brez korekcije pokrova). Neznani ključi se**tiho prezrejo** — npr. `{"integration_time": 64}` ne stori ničesar (mora biti `integration_time_ms`). Vrne `{"applied": [...], "errors": {...}}` in nikoli ne sproži izjeme.

`chloros_sdk` ponovno izvozi le jedrno površino, uporabljeno zgoraj. Celotna javna `daq_sdk` API (22 imen) dodaja naslednje — uvozite jih neposredno iz `daq_sdk`:

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## Izjeme

Ujemite osnovni razred za obravnavo »vsega, kar je šlo narobe«:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` in `ChlorosConfigurationError` sta izvožena na najvišji ravni skupaj z ostalimi; uvoziti ju je mogoče tudi iz `chloros_sdk.exceptions`X, kot je prikazano.

Hierarhija:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Primeri od začetka do konca

### 1. Obdelava mape s prilagojenim kazalnikom napredka

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Matrika LATTICE v realnem času → Odbojnost + referenca DAQ

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Kampanja zajemanja, usmerjena v projekt

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Tok sličic iz več kamer → NumPy-cevovod

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Skript za zajem brez grafičnega vmesnika neposredno na strojni opremi (brez strežniškega dela)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Preverjanje zmogljivosti pred priključitvijo niza 4 kamer

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Enakovreden recept za zajem (čisti Python)

Recept DSL v CLI ima neposredni ekvivalent v Python:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Samodejni zagon strežnika

Vstopne točke za pametno povezovanje — `connect_camera`, `connect_array`, `connect_daq_sensor` in `discover_lattice_cameras` — so »tanki« odjemalci HTTP, ki predpostavljajo, da backend prisluhne na naslovu `127.0.0.1:5000` (privzeti naslov backenda v smart-connect površine) posluša. Če je grafični vmesnik ali CLI že zagnan, eden od njiju že deluje. Pri zagonu iz samega skripta pa morda ni – zato te funkcije **samodejno zaženejo priloženo binarno datoteko strežnika** (brez okna, na enak način kot to počne `ChlorosLocal`) pred prvim klicem, nato pa počakajo do `backend_startup_timeout`, da se ta zažene.

Pravila:

- **Ustvari se vedno le lokalni URL.** `backend_url`, ki kaže na `localhost` / `127.0.0.1` / `[::1]`, je primeren; za vsak drug gostitelj se domneva, da je računalnik nekoga drugegain se nikoli ne ustvari.
- **Zadnji del ostane v teku za ponovno uporabo** (enako kot pri CLI) — ob zaključku skripta ne pride do samodejnega zaustavitve. Ponovni zagon skripta ponovno uporabi aktivni backend.
- **Izključite se z `auto_start_backend=False`** pri katerem koli od teh klicov (npr. če ste navedli oddaljeni backend ali če sami upravljate z življenjskim ciklom backenda).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Če priloženega izvedljivega datoteka ni mogoče najti ali zagnati, naslednji klic HTTP sproži izvedljivo, **na platformo prilagojeno** napako `ChlorosConnectError` namesto navadne napake– pri Windows vas usmeri na namizno aplikacijo ali ukaz `chloros-cli`; pri Linux (brez grafičnega vmesnika) vas usmeri na ukaz `chloros-cli` ali `.deb`.

---

## Okolje in glave

SDK označi vsak klic HTTP z oznako `X-Chloros-Client: sdk`. Backend uporablja pravila licenciranja SDK/CLI (potrebna je prijava **in** plačan načrt Chloros+) namesto poti brezplačnega nivoja v grafičnem vmesniku. To se nastavi samodejno ob uvozu – ničesar vam ni treba storiti.

`http://localhost` in `http://127.0.0.1` se zaznavata kot lokalni backend. Klici na druge strežnike (npr. vaša lastna analitična storitev) ostanejo nespremenjeni.

Zamenjajte backend URL s predajo `backend_url=` (ali `api_url=` na `ChlorosLocal`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(`backend_url` brez zankovanja doseže le backend source/dev – priloženi backendi se vežejo le na zankovanje; glejte Način oddaljenega backenda za vzorec tunela.)

---

## Različice in združljivost

- Različica SDK je na voljo kot `chloros_sdk.__version__`.
- SDK svoje delovanje prilagaja različici priloženega backenda. Kombinacija starejšega SDK z novejšim backendom običajno deluje (naprej združljive končne točke), vendar lahko kombiniranje novejšega SDK s starejšim backendom povzroči napake `404` na novih končnih točkah — posodobite namizno aplikacijo, da bo ustrezala.
- Vmesnik Smart Connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) ter končna točka za analizo omrežja vračata stabilne sheme JSON; nova polja so dodatna.

---

## Nasveti za odpravljanje težav

- **`ChlorosAuthenticationError: Login required`** → Na tem računalniku enkrat zaženite `chloros-cli login EMAIL PASSWORD` ali se prijavite prek namizne aplikacije Chloros.
- **`ChlorosConnectError: No Chloros backend is running …`** → Funkcija Smart-Connect samodejno zažene lokalni backend, zato se to sporočilo prikaže le, če priložene izvedljive datoteke ni mogoče najti/zažeti (npr. gostitelj, ki podpira le pip in nima namiznega paketa). Sporočilo je odvisno od platforme: na Windows odprite namizno aplikacijo ali zaženite kateri koli `chloros-cli` ukaz; na Linux zaženite ukaz `chloros-cli` (grafični vmesnik ne obstaja) ali namestite `.deb`. Pri oddaljenem strežniku prenesite `backend_url=` (in `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** pri uvozu → `lattice_sdk` se ni uspel naložiti (običajno niso nameščene Arena SDK DLL-datoteke za izvajanje). Površina brez kamere še vedno deluje.
- **Povezava niza vrne ločljivost, manjšo od izvorne**→ Funkcija »smart-prep« v ozadju samodejno pomanjša velikost okvirja, da se prilega žici. Uporabite `analyze_array_network()`, da ugotovite vzrok, nato pa nadgradite povezavo, sprejmite pomanjšanje ali pa za zaporedno zajemanje uporabite `force_tier="slip-emit-and-capture"`. Varnostna mreža za zmanjšanje**ne** pokriva skupne prekomerne naročnine (`oversubscribed: true`, polja fps 0): preveč kamer za vodilo ni mogoče popraviti z združevanjem v skupine/ROI — zmanjšajte število kamer, omogočite jumbo okvirje ali preklopite na hitrejšo mrežno kartico (glejte [Prekomerna naročnina](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` poroča, da je sprejemni obroč omrežne kartice zelo majhen (~0,26 MB) / povezovalna vrata z napako »FRAMES WILL DROP«** → Sprejemni obroč gostiteljske omrežne kartice je na privzeti vrednosti (pogosto se po posodobitvi gonilnika omrežne kartice ponastavi na 32). Na Realtek USB 10GbE adapterju nastavite `ReceiveBufferLen=256` in `PendingReceives=64` (povišano), nato ponovno zaženite backend, da ponovno prebere obroč. Celoten postopek: [CLI Referenca → Nastavitev in optimizacija gostiteljske mrežne kartice](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Gostitelj se ob ponovnem zagonu/izklopu zatakne, kasneje pa se pojavijo napake WMI `Invalid class` / mrežna kartica se ne vklopi** → Zastarel gonilnik USB 10GbE povzroča napako `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Posodobite gonilnik adapterja na najnovejšo različico (≥ 2026) in ponovno uporabite nastavitve sprejemnega obroča. Glejte [CLI Referenca → Nastavitev in optimizacija omrežne kartice gostitelja](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Zavrnjena odbojnost** → Za odbojnost v absolutni lestvici mora biti kamera (ali niz kamer) povezana z aktivnim DAQ-om. Povežite prek grafičnega vmesnika ali uporabite `processing="radiance"` (W/m²/sr/nm), ki ne zahteva sparjenega senzorja.
- **Zajem s `smart=True` traja dlje, kot je pričakovano** → Konvergenca AE je odvisna od dinamike prizora; če želite hitrejši (manj stabilen) sprožilec, povečajte vrednost `exposure_tolerance_pct` ali skrajšajte `stability_window_s`.

---

## Glej tudi

- [CLI Referenca](cli-reference.md) — vsak podukaz CLI odraža klic SDK.
- [Vodnik po senzorjih DAQ](../daq/README.md) — pravila o ožičenju, kalibraciji in snemanju, specifična za posamezne senzorje.
- Spletna dokumentacija: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>

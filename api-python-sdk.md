# API: Python SDK

{% hint style="info" %}
**Iščete celoten API?** Ta stran je praktični vodič. Vse javne razrede, metode, natančne podpise in primere, ki jih lahko preprosto kopirate in prilepite, najdete v [referenčnem priročniku za SDK](reference/sdk-reference.md), ki je optimiziran za AI-pomočnike.**Delate z AI-pomočnikom?** Prilepite ta URL v klepet, da bo imel na voljo celoten, aktualen Chloros 1.3.0 API:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Vsaka stran tega priročnika je na voljo kot surovi markdown pod imenom z malimi črkami + `.md`, celoten priročnik pa je indeksiran na `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` na PyPI) omogoča izvajanje vseh funkcij namizne aplikacije iz Pythona: obdelavo slik v paketih, nadzor kamere LATTICE in matrike v realnem času, seje z zbranimi podatki svetlobnih senzorjev ter avtomatizacijo shranjenih projektov. Gre za tanek sloj nad istim lokalnim backendom, ki ga uporabljata GUI in CLI (HTTP na `127.0.0.1:5000`), zato je delovanje enako na vseh treh vmesnikih.

## Namestitev

SDK je priložen namiznemu paketu Chloros, ki zagotavlja tudi backend za obdelavo in izvedbene okolje za strojno opremo ter je vedno usklajen z njegovo različico.

**Korak 1 — Namestite Chloros.** Windows: zaženite namestitveni program za namizje (privzeta pot `C:\Program Files\Chloros\`) s strani [Prenos](download.md). Linux: namestite paket `.deb` z `sudo apt install ./chloros_1.3.0_<arch>.deb` ([Namestitev v Linuxu](linux/linux-installation.md)).**Korak 2 — Preverite SDK** (Python 3.7+). Običajno ni treba nič namestiti:

* **Windows:** namestitveni program namesti priloženo kolo SDK v vaš sistemski Python (če Python ni nameščen, ta korak preskoči).
* **Linux:** `.deb` namesti SDK za sistemski `python3` v `/usr/lib/python3/dist-packages`. Paket priporoča naslednje zahteve: `python3-numpy` in `python3-requests`; `apt install` jih vključuje, `dpkg -i` pa ne.
* **Virtualno okolje** ne vidi sistemskih paketov; v njem namestite priloženo kolo: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` v sistemu Linux ali `chloros_sdk-*.whl` v mapi `sdk` v namestitveni mapi sistema Windows (privzeto `C:\Program Files\Chloros\sdk\`).

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

Na računalniku brez paketa Chloros, katerega SDK poganja backend na drugem računalniku (glejte [Kaj prinaša namestitev samo s pipom](#what-a-pip-only-install-gets)), ga namestite iz PyPI: `pip install chloros-sdk`. PyPI se posodablja ob izdaji novih različic, zato ustreza najnovejši stabilni različici.

**Korak 3 — Prijavite se enkrat na vsakem računalniku:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Pristopni podatki se shranijo v `~/.chloros/` (na obeh platformah). V sistemu Windowsse lahko enakovredno prijavite prek zavihka »Uporabnik« <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> v namizni aplikaciji. SDK zahteva plačljiv načrt Chloros+ — glejte [Zahteve glede licence](#license-requirement) spodaj.

| Zahteva | Podrobnosti |
| --- | --- |
| **Namestitev Chloros** | Windows: namestitveni program za namizje; Linux: paket `.deb` (vsebuje binarno datoteko strežnika in SDK) |
| **Python** | 3.7 ali višja različica (razvit/preizkušen na 3.10) |
| **Operacijski sistem** | Windows 10/11 64-bitni, Ubuntu 22.04 LTS ali novejši, ali NVIDIA Jetson (JetPack 6) |
| **Licenca** | Aktivna prijava v Chloros+, kateri koli plačljivi paket (Copper ali višji) |

## Uspeh v 60 sekundah

Z enim klicem ustvarite projekt, uvozite mapo, konfigurirate obdelavo in zaženete potek obdelave – pri čemer se backend samodejno zažene, če še ne teče:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

(V sistemu Linux uporabite Linuxove poti: `/home/user/drone_images/flight001`. SDK deluje enako na obeh platformah.)

Obdelujete mapo z zajetimi podatki LATTICE? Uporabite LATTICE-prijazen ovitek – ta uporabi prave privzete nastavitve (brez zaznavanja ciljnega panela, standardni debayer):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` – popoln nadzor nad potekom obdelave

Za vse, kar presega enovrstični ukaz, uporabite `ChlorosLocal`. Uporablja backend, ki že teče (iz namizne aplikacije), ali pa ob prvi uporabi zažene nameščenega (`auto_start_backend=True`; v sistemu Windows iz mape `backend` v namestitveni mapi Chloros, privzeto pa `C:\Program Files\Chloros\backend\`), ustvarja in konfigurira projekte, spremlja napredek ter po zaključku izvede povzetek.

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

{% hint style="info" %}
Ohranite privzeto ime `http://127.0.0.1:5000`, namesto da ga nadomestite z `localhost` — v sistemu Windows se `localhost` najprej razreši v `::1`, kar pri strežniku, ki podpira le IPv4, traja približno 2 sekundi na zahtevo.
{% endhint %}

Uporabite ga kot upravitelja konteksta. Izhod iz bloka prekine uporabo strežnika s strani te instance, ne pa samega strežnika: ta ostane v teku za naslednji skript in se samodejno ustavi po 30 minutah neuporabe. `shutdown_backend()` ustavi to instanco, ko se ta zažene.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` sprejema naslednje ključne besede: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` in `custom_settings`. Glavne vrednosti:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Nastavitve, specifične za LATTICE (`input_level`, `radiometric_output`, družina `array_alignment*`), so dokumentirane s svojimi popolnimi tabelami vrednosti v [Referenca SDK](reference/sdk-reference.md#supported-values).

### Spremljanje napredka

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Branje povzetka po izvedbi — in zaznavanje praznih izvedb

Ob zaključku `process()` priloži povzetek obdelave backenda kot `result["summary"]`. Vsak vnos v `summary["hints"]` je celoten stavek, ki pojasnjuje vse pomembno — na primer, zakaj je izvedba prinesla ničelni izhod — vsak namig pa se ponovno izpiše tudi kot Pythonov `UserWarning`, tako da se prazne izvedbe samodiagnosticirajo, tudi če slovarja nikoli ne pregledate:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` se ne sproži, kadar izvedba ne ustvari nobenih slik.** To je edino področje, na katerem se SDK in CLI namerno razlikujeta: `chloros-cli process` obravnava »zahtevani so bili izdelki, vendar ni bil zapisani noben« kot napako in se zaključi z izhodno vrednostjo, ki ni nič, medtem ko se SDK vrne normalno in o tem poroča prek `summary` / namigov. Če naj se vaš potek ob praznem izvajanju ustavi, to preverite sami – preglejte `summary` (ali preštejte datoteke v projektni mapi), namesto da se zanašate na izjemo.
{% endhint %}

## Smart Connect — aktivna strojna oprema

Trije pomočniki odprejo trajne seje v skladišču strojne opreme strežnika — istem skladišču, ki ga uporabljajo CLI, drugi skripti in zavihki kamere v grafičnem vmesniku, tako da skripti SDK sobivajo z njimi, ne da bi se borili za serijske vmesnike ali pasovno širino omrežja. Vsi trije samodejno zaženejo lokalni strežnik, če ta še ne teče. Plošča DAQ v grafičnem vmesniku ni del sklopa: zaprite svetlobni senzor v grafičnem vmesniku, preden ga odprete iz skripta.

### Ena kamera LATTICE — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**Relativne poti se nanašajo na mesto, kjer se izvaja vaš skript (spremenjeno v različici 1.3.0).** Pri lokalnem backendu je relativna pot `output_dir` (vključno s privzeto potjo `"output"`) — ostale poti pa odpre backend (`burst_dir`, `profile_path`, `save_path`, `mask_path`) — se razrešijo glede na delovno mapo vašega skripta, zato `cam.capture("output/")` piše v `<current folder>/output/`. V različici 1.2.0 jih je nameščen backend razrešil v svoji lastni namestitveni mapi in se je pojavila napaka »Dostop je zavrnjen«. Pri oddaljenem backendu se poti pošljejo takšne, kot so vnesene.
{% endhint %}

### Sinhronizirani niz — `connect_array`

`connect_array` je priporočena vstopna točka za sisteme z več kamerami. Izvaja isti potek pametne priprave kot grafični vmesnik: analiza omrežja, samodejna izbira sinhronizacijskega nivoja, časovna sinhronizacija PTP, izbira formata pikslov za vsako kamero, nastavitev avtomatske osvetlitve (AE) in aktiviranje sprožilca GPIO. **Prva serijska številka je glavna** (sproži impulz strojne sprožilne naprave); ostale so podrejene.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Dodajte `smart=True` v kateri koli zajemni niz, da pred sprožitvijo počakate, da se samodejna osvetlitev ustalijo na vseh kamerah.

Novost v različici 1.3.0: `ArraySession` vam sinhronizirane okvirje posreduje tudi neposredno, brez zapisovanja datotek:

* `arr.grab()` vrne eno sliko na kamero, vse iz enega sprožilca, v obliki numpy-matric (`{serial: array}`); `.group_id` je skupni sprožilec, `.errors` pa imenuje vsako kamero, ki je bila izpuščena.
* `arr.frame_stream(fps=5.0)` vrne en tak nabor na vsak tick, za obdelavo v realnem času v vašem lastnem potoku.
* `arr.continuous("output/")` izvaja lastno neprekinjeno zajemanje backenda (način »Continuous« v grafičnem vmesniku) in vrne `ContinuousHandle` z `status()`, `wait()` in `stop()` — uporabite ga namesto zajemne zanke, kadar morate vsak sprožilec shraniti z visoko frekvenco.

Za načine zajemanja (enkratno / neprekinjeno / intervalno / najhitrejše), snemalnike, zajem v videoposnetek in poravnavo matrik glejte [Referenco SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Senzor svetlobe DAQ — `connect_daq_sensor`

Brez argumentov `connect_daq_sensor()` samodejno zazna prenosni protokol (prednost: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Vsak okvir vsebuje 135-točkovni `spectrum` (W/m²/nm po kalibraciji), zastavico `is_saturated` ter CIE `x`, `y`, `z`. Če želite določiti posamezen senzor ali način prenosa, posredujte en eksplicitni namig:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Upoštevajte, da profili za korekcijo kapice (`cap_id`) **niso** nastavljivi prek SDK-ja — namesto tega jih izberite prek `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Shranjeni projekti — `open_project`

Shranjeni projekt Chloros ohrani povezano strojno opremo (`cameras.json` + `sensors.json` skupaj z `project.json`), `chloros_sdk.open_project(path)` pa lahko hkrati ponovno poveže svoje kamere, matrike in svetlobne senzorje ter sproži zajeme po imenu naprave. Svetlobni senzorji se ponovno povežejo prek zagnanega strežniškega dela Chloros, skupaj s svojimi shranjenimi nastavitvami in zmogljivostjo. Glej [Avtomatizacija projekta](reference/sdk-reference.md#project-automation--chlorosproject) v referenčnem gradivu.

## Kaj prinaša namestitev, ki vključuje le pip

Preden uporabite strojne površine, preverite oznake razpoložljivosti na ravni modulov:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Na gostitelju, ki ima **samo** `pip install chloros-sdk` in nima namiznega paketa Chloros:

* `ChlorosLocal`, `process_folder` in `process_lattice_capture` **ne** delujejo — potrebujejo binarno datoteko strežniškega dela, ki je priložena v namestitvenem programu za namizje.
* Pomočniki za pametno povezovanje (`connect_camera`, `connect_array`, `connect_daq_sensor`) so čisti HTTP-odjemalci, zato delujejo v povezavi z backendom na drugem računalniku — vendar so priloženi backendi vezani le na zanko, zato morate vrata sami preusmeriti (npr. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) ter `backend_url="http://127.0.0.1:5000"` posredovati skupaj z `auto_start_backend=False`. Glejte [Način oddaljenega strežnika](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Razredi LATTICE za neposredno delovanje s strojno opremo (`LatticeCamera`, `CameraPool`, …) se uvozijo, vendar potrebujejo izvedbeno okolje Arena SDK iz namiznega paketa — brez njega je `CAMERA_AVAILABLE` enak `False`.
* Senzorji DAQ se upravljajo izključno prek (tuneliranega) backenda — `connect_daq_sensor()` ali projektnega `proj.sensors`; backend odpre senzor na svojem računalniku.

## AI pomočniki — `chloros-mcp`

SDK vključuje `chloros-mcp`, strežnik MCP, ki omogoča programom Claude Desktop, Claude Code, Cursor in drugim odjemalcem MCP upravljanje sistema Chloros: obdelavo slik ter upravljanje kamer LATTICE, matrik in svetlobnih senzorjev DAQ. Njegova orodja pokrivajo funkcionalnost tega SDK-ja. Namesti se skupaj z SDK-jem, zato je za Claude Code nastavitev enostavna:

```bash
claude mcp add chloros -- chloros-mcp
```

Drugi odjemalci: [Uporaba Chlorosa z AI-pomočniki](ai-assistants.md). Možnosti in vsa orodja: [MCP-strežnik (AI-pomočniki)](reference/sdk-reference.md#mcp-server-ai-assistants).

## Zahteve glede licence

Za dostop do SDK-ja je potrebna aktivna prijava v Chloros+ na kateri koli plačljivi stopnji — **Copper ali višji**(Copper / Bronze / Silver / Gold); brezplačni paket Iron nima dostopa do SDK/CLI. Preverjanje se izvaja**na strežniški strani**: vsaka zahteva SDK mora vsebovati tako aktivno sejo kot plačani paket, sicer backend vrne `403` / `PLAN_UPGRADE_REQUIRED` (ki ju pomožni programi `ChlorosLocal` in `connect_*` povzročita kot `ChlorosLicenseError` oziroma `ChlorosConnectError`). Odjavljeni klicatelj namesto tega prejme `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — ponovno zagon `chloros-cli login` odpravi prvi primer, drugega pa ne.

Uporaba brez internetne povezave deluje v okviru odloga plačila načrta: stopnja se prebere iz predpomnilnika za preverjanje strežnika (5 minut) ali iz predpomnilnika podpisanih, na napravo vezanih licenc (30 dni za mesečne načrte; do izteka naročnine za letne). Ko se obdobje odloga izteče, se načrt spremeni v brezplačnega, dostop do SDK-ja pa se ustavi, dokler računalnik ne vzpostavi povezave s strežnikom. `chloros-cli status` ostane dostopen v brezplačni stopnji, zato je razlog vedno viden. Glejte [Prijava v Chloros+](chloros+-login.md).

## Izjeme

Ujemite osnovni razred za obravnavo »vseh napak v Chloros«:

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

Vse izjeme v poti (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) izhajajo iz `ChlorosError`. Ena past: `ChlorosConnectError` — sproži ga le `connect_camera` / `connect_array` / `connect_daq_sensor` — izhaja iz navadnega `Exception`, **ne** pa iz `ChlorosError`, zato ga `except ChlorosError` ne bo zaznal. Celotna hierarhija je v [referenčnem priročniku SDK](reference/sdk-reference.md#exceptions).

## Glej tudi

* [Referenčni priročnik SDK](reference/sdk-reference.md) — celotna površina API-ja, optimizirana za AI-pomočnike.
* [Referenca CLI](reference/cli-reference.md) — vsak podukaz CLI odraža klic SDK.
* [Uporaba Chlorosa z AI-pomočniki](ai-assistants.md) — navodila za nastavitev in uporabo `chloros-mcp`.
* [Prenos](download.md) — namestitveni programi za Windows in Linux.

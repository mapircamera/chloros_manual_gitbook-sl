# API : Python SDK

{% hint style="info" %}
**Iščete celoten API?** Ta stran je praktični vodič. Vsi javni razredi, metode, natančni podpisi in primeri, ki jih lahko preprosto kopirate in prilepite, so v [SDK referenčnem priročniku](reference/sdk-reference.md), ki je optimiziran za AI-pomočnike.**Delate z AI-asistentom?** Prilepite ta URL v klepet, da bo imel na voljo celoten, aktualen Chloros 1.2.0 API:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Vsaka stran tega priročnika je na voljo kot surovi markdown pod imenom v malih črkah + `.md`, celoten priročnik pa je indeksiran na `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` na PyPI) omogoča vse, kar lahko naredi namizna aplikacija iz Python: obdelavo slik v serijah, nadzor kamere LATTICE v realnem času in nadzor matrice, seje z zvočnim senzorjem DAQ ter avtomatizacijo shranjenih projektov. Gre za tanek sloj nad istim lokalnim ozadjem, ki ga uporabljata grafični vmesnik in CLI (HTTP na `127.0.0.1:5000`), zato je delovanje enako na vseh treh vmesnikih.

## Namestitev

Namestitev poteka v dveh korakih: najprej namestite namizni paket Chloros (ki zagotavlja ozadje za obdelavo in izvedbene okolje za strojno opremo), nato pa paket Python.

**Korak 1 — Namestite Chloros.** Windows: zaženite namestitveni program za namizje (privzeta pot `C:\Program Files\MAPIR\Chloros\`) s strani [Prenos](download.md). Linux: namestite paket `.deb` ([Namestitev Linux](linux/linux-installation.md)).**Korak 2 — Namestite SDK** (Python 3.7+):

```bash
pip install chloros-sdk
```

Morda sploh ne boste potrebovali pip: vsak namestitveni program vsebuje ustrezen SDK wheel. Namestitveni program Windows ga samodejno namesti v vaš sistem Python; namestitveni program Linux `.deb` ga namesti v `/usr/lib/chloros/sdk/` in izpiše natančen ukaz `pip install --user`. PyPI se posodablja ob izdaji novih različic, zato `pip install chloros-sdk` ustreza najnovejši stabilni različici.

**Korak 3 — Prijavite se enkrat na vsakem računalniku:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Pristojnosti so shranjene v predpomnilniku v `~/.chloros/` (na obeh platformah). V Windows se lahko enakovredno prijavite prek zavihka »Uporabnik« (User) v namizni aplikaciji <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">. Za SDK je potreben plačljiv načrt Chloros+ — glejte [Zahteve glede licence](#license-requirement) spodaj.

| Zahteva | Podrobnosti |
| --- | --- |
| **Chloros nameščen** | Windows: namestitveni program za namizje; Linux: paket `.deb` (vsebuje binarno datoteko strežnika) |
| **Python** | 3.7 ali višja (razvit/preizkušen na 3.10) |
| **Operacijski sistem** | Windows 10/11 64-bitni, Ubuntu 22.04 LTS ali novejši, ali NVIDIA Jetson (JetPack 6) |
| **Licenca** | Aktivna prijava v Chloros+, kateri koli plačljivi paket (Copper ali višji) |

## Uspeh v 60 sekundah

Z enim klicem ustvarite projekt, uvozite mapo, konfigurirate obdelavo in zaženete potek obdelave – pri čemer se samodejno zažene ozadje, če še ne teče:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(Na Linux uporabite poti Linux: `/home/user/drone_images/flight001`. SDK deluje enako na obeh platformah.)

Obdelujete mapo zajemov LATTICE? Uporabite ovitek, prilagojen za LATTICE — uporabi prave privzete nastavitve (brez zaznavanja ciljnega panela, standardni debayer):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — popoln nadzor nad delovnim tokom

Za vse, kar presega eno vrstico, uporabite `ChlorosLocal`. Ob prvi uporabi zažene backend (`auto_start_backend=True`), ustvari in konfigurira projekte, spremlja napredek ter po končanem izvajanju vrne povzetek.

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
Ohranite privzeto `http://127.0.0.1:5000`, namesto da ga nadomestite z `localhost` — pri Windows se se `localhost` najprej razreši v `::1` in stane približno 2 sekundi na zahtevo pri strežniški podlagi, ki podpira le IPv4.
{% endhint %}

Uporabite ga kot upravitelja konteksta za zagotovljeno čiščenje:

```python
import chloros_sdk

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

`configure()` sprejema naslednje ključne besede: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` in `custom_settings`. Glavne vrednosti:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Gumbki, značilni za LATTICE (`input_level`, `radiometric_output`, družina `array_alignment*`) so dokumentirani s svojimi popolnimi tabelami vrednosti v [SDK Reference](reference/sdk-reference.md#supported-values).

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

Ob zaključku `process()` priloži povzetek obdelave zadnjega dela kot `result["summary"]`. Vsak vnos v `summary["hints"]` je celoten stavek, ki pojasnjuje vse pomembno — na primer, zakaj je izvedba prinesla ničelni izhod — vsak namig pa se ponovno izpiše tudi kot Python `UserWarning`, tako da se prazne izvedbe samodiagnosticirajo, tudi če slovarja nikoli ne pregledate:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` se ne sproži, kadar izvedba ne ustvari nobenih slik.** To je edini primer, kjer se SDK in CLI namerno razlikujeta: `chloros-cli process` obravnava »zahtevani so bili izhodi, vendar ni bil zapisani noben« kot napako in se zaključi z vrednostjo, ki ni nič, medtem ko se SDK normalno zaključi in o tem stanju poroča prek `summary` / namigov. Če naj se vaš potek ob praznem izvajanju ustavi, to sami preverite — preglejte `summary` (ali preštejte datoteke v projektni mapi), namesto da se zanašate na izjemo.
{% endhint %}

## Smart Connect — aktivna strojna oprema

Trije pomočniki odprejo trajne seje v sklopu strojne opreme v ozadju – istem sklopu, ki ga uporablja grafični vmesnik, zato skripti SDK sobivajo z namizno aplikacijo, ne da bi se borili za serijske vmesnike ali pasovno širino omrežja. Vsi trije samodejno zaženejo lokalno ozadje, če nobeno ne teče.

### Ena kamera LATTICE — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### Sinhronizirana skupina — `connect_array`

`connect_array` je priporočena izhodiščna točka za sisteme z več kamerami. Izvaja isti potek pametne priprave kot grafični uporabniški vmesnik: omrežno analizo, samodejni izbor sinhronizacijskega nivoja, časovno sinhronizacijo PTP, izbiro formata pikslov za vsako kamero posebej, nastavitev avtomatske osvetlitve (AE) in aktivacijo sprožilca GPIO. **Prva serijska številka je glavna** (sproži impulz strojne sprožilne naprave); ostale so podrejene.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Dodajte `smart=True` kateri koli snemalni matrici, da se pred sprožitvijo počaka, da se avtomatska osvetlitev ustalijo na vseh kamerah. Za načine zajema (enojni / neprekinjen / intervalni / najhitrejši), snemalnike, zajem v seriji v video in poravnavo matrike glejte [Referenco SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Senzor svetlobe DAQ — `connect_daq_sensor`

Brez argumentov `connect_daq_sensor()` samodejno zazna prenosni protokol (prednost: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Vsak okvir vsebuje 135-točkovno vrednost `spectrum` (W/m²/nm po kalibraciji), zastavico `is_saturated` ter CIE `x`, `y`, `z`. Če želite določiti določen senzor ali način prenosa – kar je zanesljiva izbira na gostiteljih z več omrežnimi vmesniki, kjer samodejno odkrivanje prek omrežja Ethernet pri prvem poskusu morda ne zazna delujočega DAQ-E – posredujte en eksplicitni namig:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Upoštevajte, da profili za popravljanje omejitve (`cap_id`) **niso** nastavitev SDK – namesto tega jih izberite prek `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Shranjeni projekti — `open_project`

Shranjeni projekt Chloros ohrani povezano strojno opremo (`cameras.json` + `sensors.json` skupaj z `project.json`), `chloros_sdk.open_project(path)` pa lahko vse naenkrat ponovno poveže in zajema podatke po imenih naprav. Glejte [Avtomatizacija projektov](reference/sdk-reference.md#project-automation--chlorosproject) v referenčnem gradivu.

## Kaj prinaša namestitev samo prek pip

Preden uporabite strojne površine, preverite oznake razpoložljivosti na ravni modulov:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Na gostitelju, ki ima **samo** `pip install chloros-sdk` in nima namiznega paketa Chloros:

* `ChlorosLocal`, `process_folder` in `process_lattice_capture` **ne** delujejo — potrebujejo binarno datoteko backenda, ki je priložena v namestitvenem programu za namizje.
* Pomožni programi za pametno povezovanje (`connect_camera`, `connect_array`, `connect_daq_sensor`) so čisti odjemalci HTTP, zato delujejo z backendom na drugem računalniku — vendar so priloženi backendi vezani le na zanko nazaj, zato morate vrata posredovati sami (npr. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) ter `backend_url="http://127.0.0.1:5000"` posredovati skupaj z `auto_start_backend=False`. Glejte [Način oddaljenega strežnika](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Razredi LATTICE za neposredno delovanje s strojno opremo (`LatticeCamera`, `CameraPool`, …) se uvozijo, vendar potrebujejo Arena SDK runtime iz namiznega paketa — brez njega je `CAMERA_AVAILABLE` enak `False`.
* `daq_sdk` (razredi za neposredno zajemanje podatkov) je priložen namestitvi za namizne računalnike, ne pa paketu PyPI, zato je `DAQ_AVAILABLE` na gostitelju, ki uporablja izključno pip, enak `False` — namesto tega upravljajte senzorje DAQ prek `connect_daq_sensor()` v povezavi z (tuneliranim) backendom.

## Zahteve glede licence

Za dostop do SDK je potrebna aktivna prijava v Chloros+ na kateri koli plačljivi stopnji — **Copper ali višji**(Copper / Bronze / Silver / Gold); brezplačni paket Iron nima dostopa do SDK/CLI. Izvrševanje poteka**na strežniški strani**: vsaka zahteva SDK mora vsebovati tako aktivno sejo kot plačani paket, sicer backend vrne `403` / `PLAN_UPGRADE_REQUIRED` (ki ga `ChlorosLocal` sproži kot `ChlorosLicenseError`, pomožni programi pa kot `ChlorosConnectError`). Odjavljeni klicatelj namesto tega prejme `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — ponovno zagnanje `chloros-cli login` odpravi prvi primer, drugega pa ne.

Uporaba brez internetne povezave deluje v okviru obdobja odloga načrta: stopnja se prebere iz predpomnilnika za preverjanje strežnika (5 minut) ali iz predpomnilnika podpisane, na napravo vezane licence (30 dni za mesečne načrte; do izteka naročnine za letne). Ko se obdobje odloga izteče, se načrt spremeni v brezplačnega, dostop SDK pa se ustavi, dokler se računalnik ne poveže s strežnikom. `chloros-cli status` ostane dostopen v brezplačni stopnji, zato je razlog vedno viden. Glejte [Chloros+ Prijava](chloros+-login.md).

## Izjeme

Ujemite osnovni razred za obravnavo »vsega, kar je šlo narobe Chloros«:

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

Vse izjeme v poti (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) izhajajo iz `ChlorosError`. Ena past: `ChlorosConnectError` — sproži ga le `connect_camera` / `connect_array` / `connect_daq_sensor` — izhaja iz navadnega `Exception`, **ne** pa iz `ChlorosError`, zato ga `except ChlorosError` ne bo zaznal. Celotna hierarhija je v [Referenca za SDK](reference/sdk-reference.md#exceptions).

## Glej tudi

* [Referenca za SDK](reference/sdk-reference.md) — celotna površina API, optimizirana za AI-asistente.
* [CLI Reference](reference/cli-reference.md) — vsak CLI podukaz odraža SDK klic.
* [Prenos](download.md) — namestitveni programi za Windows in Linux.

# Namestitev Linux

Chloros se za Linux distribuira kot paketi `.deb`, ki namestijo CLI in strežnik v ozadju. Python SDK je ločen paket pip (vključen tudi v `.deb` kot wheel, ki ustreza različici).

Imena datotek paketov vsebujejo različico in arhitekturo: `chloros_1.2.0_amd64.deb` za x86_64 in `chloros_1.2.0_arm64_jp6.deb` za JetPack 6 Jetson. V spodnjih ukazih nadomestite z datoteko, ki ste jo dejansko prenesli.

***

## Linux amd64 (x86_64)

### Sistemski zahtevi

| Zahteva | Minimalna | Priporočena |
| --- | --- | --- |
| **Distribucija** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Procesor** | x86_64 (Intel/AMD) | Intel Core i7 ali boljši |
| **Pomnilnik (RAM)** | 8 GB | 16 GB ali več |
| **Grafična kartica** | Ni potrebna (obdelava prek procesorja) | NVIDIA GPU z 4 GB ali več VRAM (12 GB ali več omogoča `GPU_PARALLEL`, 7 GB ali več preprečuje uporabo Texture Aware na poti z eno sliko) |
| **Prostor za shranjevanje** | 2 GB prostega prostora | SSD z 10 GB+ prostega prostora |
| **Python** | Python 3.7+ (za SDK) | Python 3.10+ |

> **Ubuntu 20.04 in Debian 11 nista podprta.** Seznam odvisnosti za `.deb` je
> izpeljan iz tega, s čimer se dejansko povezuje ozadje Chloros, kar vključuje
> `libc6 (>= 2.34)`. Focal in bullseye vsebujeta glibc 2.31, zato `apt`
> namestitev takoj zavrne, namesto da bi dopustil, da bi kasneje med izvajanjem prišlo do napake.

### Namestitev

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
`dpkg -i` ne reši odvisnosti. Če poroča o manjkajočih paketih, `sudo apt-get install -f` (ali `sudo apt --fix-broken install`) dokonča namestitev — to je običajen potek, ne pa napaka.
{% endhint %}

Preverite namestitev:



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```***

## Linux arm64 (NVIDIA Jetson)

### Sistemski zahtevi

| Zahteva | Minimalna | Priporočena |
| --- | --- | --- |
| **Platforma** | NVIDIA Jetson z JetPack 6 | Jetson Orin NX 16 GB ali AGX Orin |
| **JetPack** | JetPack 6.x | Najnovejši JetPack 6 |
| **Pomnilnik (RAM)** | 8 GB (skupno za GPU/CPU) | 16 GB+ skupno (12 GB+ je prag za vzporedne delavce GPU) |
| **Shranjevanje** | 2 GB prostega prostora | NVMe SSD z 10 GB+ prostega prostora |
| **Python** | Python 3.7+ (za SDK) | Python 3.10+ |

### Namestitev

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Enaka struktura kot pri amd64 `.deb`, z različico CUDA, prilagojeno za Jetson Orin / Orin NX / Orin Nano. Za informacije o delovanju pomnilnika, toplotnih lastnostih in uporabi v praksi za Jetson glejte [NVIDIA Jetson Guide](nvidia-jetson-guide.md).

***

## Namestitev Python in SDK (vse različice Linux)

SDK je čisti Python HTTP odjemalec za backend, zato isti paket deluje na amd64 in arm64. Dva vira:**Iz PyPI** — objavljena stabilna različica:

```bash
pip install chloros-sdk
```

**Iz priloženega paketa wheel** — zagotovo ustreza CLI/backendu, ki ste ga pravkar namestili (uporabite to, če je vaš `.deb` novejši od tistega na PyPI):

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**Distribucije PEP 668** (Ubuntu 23.10+, Debian 12+) ne dopuščajo namestitev prek pip na ravni celotnega sistema. Uporabite `pip install --user …`, virtualno okolje ali `sudo pip install --break-system-packages …`. Namestitveni program paketa nikoli samodejno ne namesti SDK v vaš sistem Python — ta odločitev je prepuščena vam.
{% endhint %}

Izbirne dodatne funkcije:

| Dodatek | Ukaz | Dodaja |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` za prenos napredka v realnem času |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` za prenos prek BLE (DAQ-M) |

Preverite SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
`.deb` namesti Chloros, CLI in backend. Python in SDK komunicirata s tem backendom prek lokalnega HTTP in API (`http://127.0.0.1:5000`) ter ga po potrebi samodejno zažene. Vedno uporabljajte dobesedni naslov IPv4 namesto `localhost` — `localhost` se lahko razreši v `::1` in stane približno dve sekundi na zahtevo.
{% endhint %}

***

## Prva nastavitev

### 1. Prijava

Za dostop do naslovov CLI in SDK je potrebna plačljiva stopnja Chloros+ (**Copper** ali višji), kar se uveljavlja na strani strežnika: izpisani klicatelj dobi `401 AUTH_REQUIRED`, klicatelj z brezplačnim paketom (Iron) pa `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Pristojnosti so shranjene v predpomnilniku v `~/.chloros/user_session.json`.

{% hint style="warning" %}
**Po vsaki namestitvi ali nadgradnji se morate ponovno prijaviti.** Skript `prerm` paketa namerno izbriše `~/.chloros/user_session.json` in shranjeno licenco za vsakega uporabnika na računalniku, tako da nova različica vedno ponovno preveri veljavnost licence, namesto da bi se zanašala na zastarel predpomnilnik.
{% endhint %}

### 2. Preverite stanje svoje licence

```bash
chloros-cli status
```

`chloros-cli status` deluje na vseh ravneh (vključno z brezplačno), zato lahko vedno ugotovite, zakaj je dostop na voljo ali ni.

### 3. Izvedite sistemsko diagnostiko

```bash
chloros-cli selftest
```

Sedem preverjanj se izvede po vrstnem redu, ukaz pa se zaključi z vrednostjo, ki ni nič, če katero koli od njih ni uspešno:

| # | Preverjanje | Kaj dokazuje |
| --- | --- | --- |
| 1 | **Različica** | CLI poroča o svoji različici (`v1.2.0`). |
| 2 | **Vrata na voljo** | Vrata 5000 so prosta *ali* na njih že odgovarja delujoč strežnik Chloros (kar se šteje kot uspešno). |
| 3 | **Zagon strežnika** | Izvede se izvedljiva datoteka strežnika. |
| 4 | **Test API (`/api/test`)** | Backend odgovori `status: ok`. |
| 5 | **Podatki o sistemu** | Izpiše `GPU: <name>, CUDA: <bool>, PyTorch: <version>` iz `/api/system-info`. |
| 6 | **Modeli za odstranjevanje šuma** | Najde modele `*.pth.enc` (na Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + odstranjevalnik šuma**| Funkcija »Texture Aware« je dejansko uporabna — potrebuje CUDA**in** vsaj eno datoteko modela. |

Izvajanje se konča z `N/7 checks passed`, pri čemer so morebitne napake navedene po imenu.

### 4. Obdelava prvega niza podatkov

```bash
chloros-cli process ~/datasets/flight001
```

***

## Datoteke in mape

### Na uporabnika

Chloros hrani svoje poverilnice in konfiguracijo CLI v enem samem medplatformnem imeniku, **`~/.chloros/`** (na Windows, `%USERPROFILE%\.chloros\`). Dva predpomnilnika, specifična za Linux, pa sledita konvencijam XDG — ta upoštevata `XDG_CONFIG_HOME` / `XDG_CACHE_HOME`, če sta nastavljena.

| Pot | Namen |
| --- | --- |
| `~/.chloros/user_session.json` | Predpomnilnik prijavne seje, ki ga zapiše `chloros-cli login` (izbriše se ob vsaki namestitvi/posodobitvi paketa) |
| `~/.chloros/working_directory.txt` | Prevladujoča privzeta mapa projekta (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Jezikovna nastavitev CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Nastavitev jezika, ki se deli z grafičnim vmesnikom Windows — v tem primeru ima `language` prednost pred `cli_language.json` |
| `~/.chloros/update_cache.json` | Enourni predpomnilnik za preverjanje posodobitev ob zagonu Linux/Jetson |
| `~/.chloros/backend.log` | Dnevnik zadnjega dela sistema ob zagonu zadnjega dela s strani CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | V predpomnilniku shranjeni kalibracijski paketi LATTICE za posamezne kamere, razvrščeni po serijski številki in hash vrednosti paketa |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Izbirne uporabniške prekrivke za profile popravkov omejitve DAQ |
| `~/.config/chloros/system_config.json` | V predpomnilniku shranjen profil strojne opreme iz dinamičnega prilagajanja računalniških zmogljivosti (Dynamic Compute Adaptation) — izbrišite ga, da prisilite novo zaznavanje strojne opreme |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Dnevniki strežnika backenda, ena datoteka na zagon |
| `~/Chloros Projects/` | Privzeta mapa projekta, če ni nastavljeno nobeno prekrivanje |

### Na ravni sistema

| Pot | Namen |
| --- | --- |
| `/usr/bin/chloros-cli` | Skript ovijalca — nastavi `LD_LIBRARY_PATH` za priložene izvorne knjižnice, nato pa izvede dejansko binarno datoteko |
| `/usr/bin/chloros-backend` | Skript ovijalca — enako, plus `CHLOROS_PRODUCTION=1`, da se avtentifikacijski mehanizem ozadja nikoli ne more tiho onemogočiti |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Kompilirane izvedljive datoteke |
| `/usr/lib/chloros/arena_runtime/` | Arena SDK runtime, ki ga zahtevajo kamere LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Šifrirani modeli za odstranjevanje šuma, ki jih uporablja Texture Aware debayer |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK kolo, ki ustreza točno tej različici |
| `/usr/lib/chloros/exiftool` | Priloženo orodje exiftool (simbolična povezava do `/usr/local/bin/exiftool` le, če sistemski exiftool ne obstaja) |
| `/etc/chloros/update.conf` | Konfiguracija kanala za posodobitve, ki jo bere `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Nastavi `net.ipv4.ip_unprivileged_port_start = 319`, da lahko backend veže PTP-vrata brez pravic root |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Usmeri dinamični nalagalnik na `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Prijavljenemu uporabniku dodeli dostop do serijskega mostu DAQ-U USB (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend.service` | Vključi storitev v ozadju, ki je vedno vklopljena (nameščena, **ni omogočena**) |
| `/usr/share/applications/chloros-cli.desktop` | Vnos v meniju aplikacije »Chloros CLI«, ki odpre terminal |

## Mesto izvedljive datoteke strežniškega dela

CLI in SDK samodejno zaznavata strežniški del:

| Komponenta | Pot |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` |

Pot backenda lahko prepisujete z zastavico `--backend-exe` CLI ali s parametrom konstruktorja `backend_exe` SDK, vmesnik pa s `--port` (privzeto `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` usmerja družine ukazov **`lattice`**,**`project`**in**`daq pool-*`** na oddaljenem strežniku. Osnovni ukazi (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) to namerno prezrejo in vedno ciljajo na `http://127.0.0.1:<port>`.
{% endhint %}

***

## Kamere LATTICE in svetlobni senzorji DAQ na Linux

Vse družine ukazov za delo z živo strojno opremo delujejo na Linux (amd64 in Jetson):

* **`chloros-cli lattice`** — odkrivanje, povezovanje, konfiguriranje in zajemanje s kamer LATTICE ter sinhroniziranih nizov. `.deb` združuje potrebno izvedbeno okolje Arena SDK in ga registrira pri dinamičnem nalagalniku.
* **`chloros-cli daq pool-*`** — povezovanje svetlobnih senzorjev DAQ-U/M/E prek bazena backendov, pretakanje kalibriranih spektrov in snemanje datotek `.daq`. Kompilirani CLI vsebuje le družino `pool-*`: `pool-connect`, `pool-disconnect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`.
* **`chloros-cli project`** — brez uporabe grafičnega vmesnika zaženite shranjen projekt (njegove kamere, senzorje in nastavitve obdelave).
* **`chloros-cli time-sync`** — pregled PTP-grandmastera, na katerem teče Chloros-backend za kamere LATTICE in senzorje DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` je potreben za `pool-latest`, `pool-stream`, `pool-record` in `pool-set-cap`; `pool-list` prikazuje identifikacijske številke, ki so trenutno v zbirki.

{% hint style="info" %}
**Za prvo povezavo DAQ-E na računalniku z več omrežnimi vmesniki raje uporabite `--eth-host`.** Samodejno odkrivanje preišče mDNS in lahko spregleda vmesnik senzorja zaradi praznega ARP-predpomnilnika, zato lahko prva povezava `pool-connect --eth` po zagonu računalnika ne uspe, čeprav je senzor popolnoma delujoč. Če navedete IP-naslov ali ime gostitelja senzorja, se odkrivanje v celoti preskoči.
{% endhint %}

**Serijska dovoljenja za DAQ-U** upravlja nameščeno pravilo udev (`uaccess` + skupina `dialout`). Če senzor, ki je bil že priključen, ostane nedostopen, ponovno naložite pravila ali ga ponovno priključite:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Za celoten nabor ukazov glejte [referenco CLI](../CLI.md).

### Vedno vklopljen PTP za gostitelje brez zaslona

Ob prvi namestitvi se ustvari enota systemd `chloros-backend.service`, vendar **ni omogočena**. Na Jetsonu brez zaslona ali strežniku, kjer mora sinhronizacija časa PTP neprekinjeno delovati za senzorje DAQ-E in kamere LATTICE, jo omogočite:

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

Brez tega PTP deluje le, dokler teče backend Chloros — to pomeni med aktivno sejo CLI/SDK.

Naprava poveže backend z `127.0.0.1:5000` (nastavitve okolja `CHLOROS_HOST` / `CHLOROS_PORT` znotraj naprave; prepisati z `sudo systemctl edit chloros-backend.service`) in ga v primeru napake ponovno zažene po 5 sekundah.

**Kako PTP pridobi svoja vrata.** PTP uporablja UDP 319/320, ki sta oba pod običajno spodnjo mejo 1024 za privilegirana vrata. Paket `postinst` zapiše `/etc/sysctl.d/60-chloros-ptp.conf` z `net.ipv4.ip_unprivileged_port_start = 319`, kar omogoča, da jih backend veže med izvajanjem kot vaš uporabnik. Kot dodatno varnostno ukrepanje uporabi tudi `setcap cap_net_bind_service,cap_net_raw=+ep` za binarno datoteko backenda – zato je `libcap2-bin` deklarirana odvisnost paketa.***

## Primeri skriptov v Bashu

{% hint style="info" %}
**Izhodne kode, primerne za skripte.**`chloros-cli process` ob uspešnem zaključku izpiše `0`,**ob neuspehu pa vrednost, ki ni nič – vključno z izvajanjem, ki je zahtevalo slikovne produkte, a jih ni zapisalo** (izpiše `Processing finished but wrote no image products.` ter navede ime projektne mape in običajne vzroke). Uspešni izvajanji poročata, koliko slikovnih izdelkov je bilo shranjenih (`Image products written: N`). Kode izhoda: `0` uspeh, `1` neuspeh, `2` napaka v argumentu, `130` prekinjeno.
{% endhint %}

### Obdelava več naborov podatkov

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Obdelava s prilagojenimi nastavitvami

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

Veljavnih vrednosti `--format` je natanko štiri, vsebujejo pa presledke – vedno jih vpišite v narekovajih:

| Vrednost `--format` | Izhodna mapa |
| --- | --- |
| `TIFF (16-bit)` *(privzeto)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` sprejema `standard` (privzeto) ali `texture-aware` (Chloros+).

### Avtomatizirana obdelava s Cronom

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Primer Python SDK

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Odpravljanje težav

### CLI ni bil najden po namestitvi

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Dostop zavrnjen

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### »setcap ni uspel« med namestitvijo

`.deb` uporabi `cap_net_bind_service` za `/usr/lib/chloros/chloros-backend`, da lahko brez root dostopa poveže PTP vrata 319/320. Če je ob namestitvi manjkala `libcap2-bin`, se klic preskoči. Namestite jo in ponovno namestite paket:

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### PTP se ne zažene / ne more povezati vrat 319

Preverite, ali je bila spodnja meja za vrata brez privilegijev znižana, in jo ponovno uporabite za trenutni zagon, če ni bila:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Nato preverite glavni strežnik:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### »Gonilniki kamer LATTICE niso bili najdeni«

Izvedbeno okolje Arena SDK se ne razreši. Preverite, ali je konfiguracija nalagalnika, ki jo zapiše paket, prisotna in posodobljena:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
ls /usr/lib/chloros/arena_runtime | head
```

### Zaganjanje backenda ni uspelo

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

Dnevniki backenda za neuspešen zagon so v `~/.cache/chloros/logs/`.

### CUDA ni zaznan

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

`chloros-cli selftest` poroča o istem v eni vrstici: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Manjkajoče skupne knjižnice

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### Počasen zagon na sistemih s pomnilniško kartico SD

Kompilirane izvedljive datoteke se ob vsakem zagonu samodejno razpakirajo v začasni imenik. Če obstaja `/mnt/ssd/tmp`, ga Chloros samodejno uporabi; v nasprotnem primeru nastavite `TMPDIR` na hiter datotečni sistem:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Posodobitev Chloros na Linux

Ukaz `update` je na voljo le za Linux/Jetson. Preveri različico, objavljeno v kanalu za posodobitve, nastavljenem v `/etc/chloros/update.conf`, in ponudi prenos ter namestitev ustrezne različice `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Na Linux/Jetson CLI ob vsakem zagonu izvede tudi neblokirajoči pregled posodobitev (rezultat se shrani v predpomnilnik za eno uro v `~/.chloros/update_cache.json`) in izpiše `Update available: vX.Y.Z`, če obstaja novejša različica. Vaše nastavitve in projekti ostanejo ohranjeni po posodobitvi; po njej se boste morali ponovno prijaviti.

## Odstranjevanje

```bash
sudo apt remove chloros
```

Odstranitev ustavi `chloros-backend.service`, ponovno vzpostavi privzeto spodnjo mejo za neprivilegirana vrata (1024), odstrani simbolično povezavo do priloženega orodja exiftool in konfiguracijo zaganjalnika Arena ter izbriše shranjena pooblastila. Vaši projekti in datoteke `~/.chloros/` ostanejo nedotaknjeni.

***

## Naslednji koraki

* [Vodnik za NVIDIA Jetson](nvidia-jetson-guide.md) — optimizacija in namestitev, specifična za Jetson
* [CLI : Ukazna vrstica](../CLI.md) — vodnik CLI
* [API : Python SDK](../api-python-sdk.md) — vodnik SDK
* [CLI Referenca](../reference/cli-reference.md) in [SDK Referenca](../reference/sdk-reference.md) — izčrpni seznami ukazov/API za različico 1.2.0
* [Dinamična prilagoditev računalniške zmogljivosti](../processing-architecture/dynamic-compute-adaptation.md) — kako se Chloros prilagaja vaši strojni opremi

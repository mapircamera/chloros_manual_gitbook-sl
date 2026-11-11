# Namestitev v sistemu Linux

Chloros se za sistem Linux distribuira v obliki paketov `.deb`, ki namestijo CLI, strežnik v ozadju in Python SDK (za sistem `python3` ter kolo, ki ustreza različici, za virtualne okolja).

Imena datotek paketov vsebujejo različico in arhitekturo: `chloros_1.3.0_amd64.deb` za x86_64, in `chloros_1.3.0_arm64_jp6.deb` za arm64 (NVIDIA Jetson, JetPack 6). V spodnjih ukazih vstavite ime datoteke, ki ste jo dejansko prenesli.

{% hint style="info" %}
**Novost v različici 1.3.0: programi so dostavljeni v razpakirani obliki.** Backend in CLI sta nameščena kot mapi, `/usr/lib/chloros/backend/` in `/usr/lib/chloros/cli/`, namesto kot posamezne samorazpakirajoče se datoteke. Pri vsakem zagonu se ne razpakirata več v začasno mapo, zato se zagnata veliko hitreje (na x86_64 je backend prej porabil približno 100 sekund za razpakiranje ob vsakem zagonu) in ne potrebujeta več več GB prostega prostora v mapi `/tmp`. Tudi amd64 `.deb` je zdaj približno pol manjši kot prej.
{% endhint %}

***

## Linux amd64 (x86_64)

### Sistemski zahtevi

| Zahteva | Minimalna | Priporočena |
| --- | --- | --- |
| **Distribucija** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Procesor** | x86_64 (Intel/AMD) | Intel Core i7 ali boljši |
| **Pomnilnik (RAM)** | 8 GB | 16 GB ali več |
| **Grafična kartica** | Ni potrebna (obdelava v procesorju) | NVIDIA GPU z 4 GB+ VRAM (12 GB+ odklene `GPU_PARALLEL`, 7 GB+ ohranja Texture Aware izklopljeno na poti z eno sliko) |
| **Prostor za shranjevanje** | 2 GB prostega prostora | SSD z 10 GB+ prostega prostora |
| **Python** | Python 3.7+ (za SDK) | Python 3.10+ |

> **Ubuntu 20.04 in Debian 11 nista podprta.** Seznam odvisnosti `.deb` je
> izpeljan iz tega, s čimer se backend Chloros dejansko povezuje, kar vključuje
> `libc6`, ki je novejši od glibc 2.31, ki ga vključujeta različici focal in bullseye, zato `apt`
> namestitev takoj zavrne, namesto da bi dopustil, da bi kasneje med izvajanjem prišlo do napake.

### Namestitev

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**Uporabite `apt install ./…`, ne `dpkg -i`.** `apt` razreši odvisnosti paketa in namesti tudi tisto, kar priporoča: zahteve Python SDK, `python3-numpy` in `python3-requests`. `dpkg -i` ne namesti nobenega od njiju. CLI in backend sta samostojna in delujeta brez njih, vendar jih `import chloros_sdk` potrebuje; izpis namestitve navaja vse, kar manjka. Ohranite `./`, da bo `apt` namestil datoteko, namesto da bi iskal paket z istim imenom. Paket deklarira tudi `libgl1`, ki ga naloži priloženi OpenCV, vendar ga v minimalni strežniški sliki morda ni.
{% endhint %}

Preverite namestitev:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### Sistemski zahtevi

| Zahteva | Minimalna | Priporočena |
| --- | --- | --- |
| **Platforma** | NVIDIA Jetson z JetPack 6 | Jetson Orin NX 16 GB ali AGX Orin |
| **JetPack** | JetPack 6.x | Najnovejši JetPack 6 |
| **Pomnilnik (RAM)** | 8 GB (skupno za GPU/CPU) | 16 GB+ skupno (12 GB+ je prag za vzporedno obdelavo) |
| **Shranjevanje** | 2 GB prostega prostora | NVMe SSD z 10 GB+ prostega prostora |
| **Python** | Python 3.7+ (za SDK) | Python 3.10+ |

### Namestitev

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Enaka postavitev kot pri amd64 `.deb` (vključuje Python SDK, sprejemne pomnilnike in strežniške storitve), za Jetson Orin / Orin NX / Orin Nano na **JetPack 6** (L4T R36). Njegov PyTorch je CUDA-izgradnja JetPack 6 za Orin, zato obdelava na GPU, vključno z Texture Aware debayerjem, poteka na GPU-ju Jetsona. Paket sam ne vsebuje CUDA: uporablja knjižnici CUDA in cuDNN, ki ju namesti JetPack 6 (`sudo apt install nvidia-jetpack` na sliki, ki ju nima). Kjer ni na voljo grafičnega procesorja s podporo za CUDA, obdelava poteka na procesorju, `--debayer texture-aware` pa uporablja standardni debayer (backend v dnevniku zabeleži eno vrstico `[DEBAYER] … needs a CUDA GPU …`, v kateri pojasni razlog). Za informacije o delovanju Jetsona v zvezi s pomnilnikom, toplotnimi lastnostmi in uporabo v terenskih pogojih glejte [NVIDIA Jetson Guide](nvidia-jetson-guide.md).***

## Namestitev Python SDK (vsi Linux)**`.deb` samodejno namesti SDK.** Oba paketa (amd64 in Jetson) ga namestita za sistem `python3`, zato `import chloros_sdk` deluje takoj po namestitvi, brez pip in brez omrežja. Njegovi paketi (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` ter nalagalniki projektov) se nahajajo v `/usr/lib/python3/dist-packages`, ki pripada paketu: nadgradnja jih nadomesti, odstranitev paketa pa jih odstrani. Različica se vedno ujema z namestljenim CLI-jem in backendom.

Lastne zahteve SDK-ja, `python3-numpy` in `python3-requests`, so **priporočene, niso pa obvezne** s strani paketa: `sudo apt install ./chloros_<version>_<arch>.deb` jih namesti, `dpkg -i` pa ne. Izpis namestitve poroča o različici SDK-ja, ki jo `python3` uvozi, o vseh manjkajočih zahtevah ter vse `chloros_sdk` drugje na poti `python3`, ki prekriva kopijo paketa (na primer tisto, ki je bila prej nameščena z `pip install --user`).**V virtualnem okolju**, ki ne vidi sistemskih paketov, vanj namestite priloženo kolo (povsem brez povezave):

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**Iz PyPI** — samo za računalnik, na katerem ni nameščen paket Chloros, katerega SDK poganja backend na drugem računalniku (glejte [Način oddaljenega backenda](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

Izbirne dodatne možnosti za namestitev s pip (v virtualnem okolju ali prek PyPI):

| Dodatek | Ukaz | Dodaja |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` za prenos napredka v realnem času |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` za prenos prek BLE (DAQ-M) |

Preverite SDK:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Pythonov SDK komunicira z ozadjem, ki ga `.deb` namesti prek lokalnega HTTP API-ja (`http://127.0.0.1:5000`). `ChlorosLocal` in pomožni programi za povezovanje ga samodejno zaženejo, ko je to potrebno; `connect_all()` shranjenega projekta tega nikoli ne stori, zato na računalniku brez grafičnega vmesnika zagnajte backend kot [storitev](#always-on-ptp-for-headless-hosts). Vedno uporabljajte dobesedni naslov IPv4 namesto `localhost` — `localhost` se lahko razreši v `::1` in stane približno dve sekundi na zahtevo.
{% endhint %}

***

## Prva nastavitev

### 1. Prijava

Za dostop do CLI-ja in SDK-ja je potrebna plačljiva stopnja Chloros+ (**Copper** ali višja), kar se uveljavlja na strani strežnika: odjavljeni klicatelj dobi naslov `401 AUTH_REQUIRED`, klicatelj z brezplačno stopnjo (Iron) pa naslov `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Pristojnosti so shranjene v predpomnilniku v `~/.chloros/user_session.json`.

{% hint style="info" %}
**Pri nadgradnjah se vaše prijavne podatke ohranijo; pri odstranitvi se izbrišejo.** Če namestite novejšo različico `.deb` nad obstoječo namestitvijo (tudi prek `chloros-cli update`), se ohranita `~/.chloros/user_session.json` in shranjena licenca, tako da naloge CLI/SDK brez grafičnega vmesnika še naprej delujejo. Odstranitev paketa (`sudo apt remove chloros`) izbriše shranjene prijavne podatke in licenčni predpomnilnik za vsakega uporabnika na računalniku, zato je za novo namestitev ponovno potrebna različica `chloros-cli login`.
{% endhint %}

### 2. Preverite stanje svoje licence

```bash
chloros-cli status
```

`chloros-cli status` deluje na vseh ravneh (vključno z brezplačno), zato lahko vedno ugotovite, zakaj je dostop na voljo ali ni.

### 3. Zaženite sistemsko diagnostiko

```bash
chloros-cli selftest
```

Sedem preverjanj se izvede po vrstnem redu, ukaz pa se zaključi z izhodno vrednostjo, ki ni nič, če katero koli od njih ni uspešno:

| # | Preverjanje | Kaj dokazuje |
| --- | --- | --- |
| 1 | **Različica** | Ukazna vrstica (CLI) poroča o svoji različici (`v1.3.0`). |
| 2 | **Vrata na voljo** | Vrata 5000 so prosta *ali* se na njih že odziva delujoč backend Chlorosa (kar šteje kot uspešno). |
| 3 | **Zagon backenda** | Binarna datoteka backenda se zažene. |
| 4 | **Test API-ja (`/api/test`)** | Backend odgovori z `status: ok`. |
| 5 | **Podatki o sistemu** | Izpiše `GPU: <name>, CUDA: <bool>, PyTorch: <version>` iz `/api/system-info`. |
| 6 | **Modeli za odstranjevanje šuma** | Najde modele `*.pth.enc` (v sistemu Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + odstranjevalnik šuma**| Texture Aware je dejansko uporaben — potrebuje CUDA**in** vsaj eno datoteko modela. |

Izvajanje se konča z `N/7 checks passed`, pri čemer so navedene vse napake po imenu.

### 4. Obdelajte svoj prvi nabor podatkov

```bash
chloros-cli process ~/datasets/flight001
```

***

## Datoteke in imeniki

### Na uporabnika

Chloros shranjuje svoje poverilnice in konfiguracijo CLI v enem samem medplatformnem imeniku, **`~/.chloros/`** (v sistemu Windows `%USERPROFILE%\.chloros\`). Dva predpomnilnika, specifična za Linux, pa sledita konvencijam XDG – ta upoštevata `XDG_CONFIG_HOME` / `XDG_CACHE_HOME`, če sta nastavljena.

| Pot | Namen |
| --- | --- |
| `~/.chloros/user_session.json` | Predpomnilnik prijavne seje, ki ga zapiše `chloros-cli login` (ohrani se tudi ob posodobitvah paketa; izbriše se ob odstranitvi paketa) |
| `~/.chloros/working_directory.txt` | Privzeta nadomestitev mape projekta (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Nastavitev jezika za CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Nastavitev jezika, ki se deli z grafičnim vmesnikom Windows — vrednost `language` ima tukaj prednost pred vrednostjo `cli_language.json` |
| `~/.chloros/update_cache.json` | Enourni predpomnilnik za preverjanje posodobitev ob zagonu sistema Linux/Jetson |
| `~/.chloros/backend.log` | Dnevnik strežnika, ko je bil strežnik zagnan prek CLI-ja |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | V predpomnilniku shranjeni kalibracijski paketi LATTICE za posamezne kamere, razvrščeni po serijski številki in hash vrednosti paketa |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Izbirne uporabniške prekrivke za profile korekcije omejitve DAQ |
| `~/.config/chloros/system_config.json` | V predpomnilniku shranjen profil strojne opreme iz dinamičnega prilagajanja računalniške zmogljivosti (Dynamic Compute Adaptation) — izbrišite ga, da prisilite novo zaznavanje strojne opreme |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Dnevniki strežnika backenda, ena datoteka na zagon |
| `~/Documents/MAPIR/Chloros Projects/` | Privzeta mapa projekta, če ni nastavljeno nobeno prepisovanje |

### Na ravni sistema

| Pot | Namen |
| --- | --- |
| `/usr/bin/chloros-cli` | Skript ovijalca — nastavi `LD_LIBRARY_PATH` za priložene izvorne knjižnice (in `TMPDIR` na `/mnt/ssd/tmp`, če ta mapa obstaja), nato pa zažene dejanski izvedljivi datotek |
| `/usr/bin/chloros-backend` | Skript ovijalca — enako za backend, ki ga označi kot nameščeno produkcijsko različico |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | Kompilirani CLI in backend, nameščena v razpakirani obliki (novost v različici 1.3.0); programa sta `cli/chloros-cli` in `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Dvovrstični zagonski skripti, ki zaženejo zgoraj navedene dejanske programe — shranjeni so na teh poteh, ker jih SDK, CLI in starejši skripti iščejo tukaj |
| `/usr/lib/chloros/arena_runtime/` | Izvedbeno okolje Arena SDK, ki ga zahtevajo kamere LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Šifrirani modeli za odstranjevanje šuma, ki jih uporablja Texture Aware debayer |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (in `lattice_sdk`, `lattice_cam`, `arena_api`, nalagalniki projektov) | Python SDK, nameščen za sistem `python3` in v lasti paketa |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK wheel, ki ustreza točno tej različici, za virtualno okolje |
| `/usr/lib/chloros/exiftool` | Priloženi zaganjalnik ExifTool (simbolično povezan z `/usr/local/bin/exiftool` le, če sistemski exiftool ne obstaja). Od različice 1.3.0 naprej skript ExifTool vsebuje lastno knjižnico Perl v `/usr/lib/chloros/exiftool-dist/`, zato deluje brez sistemskega paketa `libimage-exiftool-perl` |
| `/etc/chloros/update.conf` | Konfiguracija kanala za posodobitve, ki jo bere `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Nastavi `net.ipv4.ip_unprivileged_port_start = 319`, da lahko backend veže PTP-vrata brez pravic root |
| `/etc/sysctl.d/60-chloros-gige.conf` | Poveča sprejemne pomnilnike vtičnic jedra (`net.core.rmem_max`, `net.core.rmem_default`) na 32 MB za tokove kamere LATTICE in jih uporabi ob namestitvi (že nastavljena višja vrednost ostane nespremenjena). Pri privzeti nastavitvi jedra se paketi kamere v jedru zavržejo, zaradi česar niz izgubi okvirje in hitrost snemanja; `chloros-cli lattice network` poroča o stanju pomnilnikov, ko se njihova zmogljivost zmanjša. Glej [Linuxovi gostitelji — pomnilniki za sprejem vtičnic](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Usmeri dinamični nalagalnik na `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Prijavljenemu uporabniku dodeli dostop do serijskega mostu DAQ-U USB (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | Storitev »opt-in« za vedno vklopljeno ozadje, ki ozadje izvaja kot uporabnik, ki ga imenujete (nameščena, **ni omogočena**) — tista, ki jo je treba omogočiti, glejte [Always-On PTP](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | Ista storitev, ki izvaja ozadje kot **root**(nameščena,**ni omogočena**), ohranjena zato, da računalnik, na katerem je omogočena, še naprej deluje |
| `/var/log/chloros/` | Podrobni dnevnik storitve root, ki ga lahko bere skupina `adm` (izbrisan ob odstranitvi paketa) |
| `/usr/share/applications/chloros-cli.desktop` | Vnos v meniju aplikacij »Chloros CLI«, ki odpre terminal |

**Požarni zid (novost v različici 1.3.0).** Če sta ob namestitvi aktivna `ufw` ali `firewalld`, paket odpre UDP-vrata `5002` (podatki DAQ-E), `5353` (odkrivanje mDNS), `319` in `320` (PTP), ter ta pravila odstrani ob odstranitvi paketa. Če je požarni zid nameščen, vendar ni aktiven, ne doda ničesar in izpiše vrata, ki jih je treba dovoliti, če ga kasneje vklopite. Ne odpre nobenega TCP-vrata: posodobitev firmware-a DAQ-E, pri kateri se senzor prek protokola TCP ponovno poveže s tem računalnikom, da bi zbral sliko, se ne uspešno zaključi z napako `… device never connected back on TCP <port> …`, če aktivni požarni zid blokira to povezavo.

## Mesto izvedljive datoteke strežnika

CLI in SDK samodejno zaznavata strežnik:

| Komponenta | Pot |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` (zagon za `/usr/lib/chloros/backend/chloros-backend`) |

Pot do backenda lahko prepisate z zastavico CLI-ja `--backend-exe` ali s parametrom konstruktorja SDK-ja `backend_exe`, vrata pa s `--port` (privzeto `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` usmerja družine ukazov **`lattice`**,**`project`**in**`daq pool-*`** na oddaljeni strežniški platformi. Osnovni ukazi (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) ga namerno prezrejo in vedno ciljajo na `http://127.0.0.1:<port>`.
{% endhint %}

***

## Kamere LATTICE in svetlobni senzorji DAQ v sistemu Linux

Vse družine ukazov live-hardware delujejo v sistemu Linux (amd64 in Jetson):

* **`chloros-cli lattice`** — odkrivanje, povezovanje, konfiguriranje in zajemanje s kamerami LATTICE ter sinhroniziranimi nizi. `.deb` združuje potrebno izvedbeno okolje Arena SDK, ki ga potrebujejo, in ga registrira pri dinamičnem nalagalniku.
* **`chloros-cli daq pool-*`** — povezovanje svetlobnih senzorjev DAQ-U/M/E prek bazena strežnikov, pretakanje kalibriranih spektrov, snemanje datotek `.daq` in zamenjava profilov pokrovčkov ter za DAQ-E spreminjanje njegovih omrežnih nastavitev, ponovni zagon in posodobitev njegove programske opreme. `chloros-cli daq discover` išče senzorje, ki jih lahko priključite, `chloros-cli daq --help` pa prikaže seznam podukazov `pool-*`.
* **`chloros-cli project`** — brez zaslona zažene shranjen projekt (njegove kamere, senzorje in nastavitve obdelave).
* **`chloros-cli time-sync`** — pregleda glavni PTP-strežnik, na katerem teče backend Chloros za kamere LATTICE in senzorje DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` je potreben za `pool-latest`, `pool-stream`, `pool-record` in `pool-set-cap`; `pool-list` prikaže ID-je, ki so trenutno v zbirki.

{% hint style="info" %}
**`--eth-host` preskoči odkrivanje.** `pool-connect --eth` najde DAQ-E prek mDNS, z nadomestnim ARP-jem, ki deluje iz praznega ARP-predpomnilnika; s posredovanjem IP-naslova ali imena gostitelja senzorja s `--eth-host` se z njim poveže neposredno.
{% endhint %}

**Serijska dovoljenja za DAQ-U** se obdelujejo z nameščenim pravilom udev (`uaccess` + skupina `dialout`). Če senzor, ki je bil že priključen, ostane nedostopen, ponovno naložite pravila ali ga ponovno priključite:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Za celoten nabor ukazov glejte [referenco CLI](../CLI.md).

### Vedno vklopljen PTP za gostitelje brez zaslona

Paket namesti dve storitvi systemd za backend, ki nista omogočeni. Na Jetsonu brez zaslona ali strežniku, kjer mora sinhronizacija časa PTP neprekinjeno delovati za senzorje DAQ-E in kamere LATTICE, ali kjer skripti SDK povezujejo shranjene projekte (`connect_all()` nikoli ne zažene ozadja), omogočite storitev **kot vaš uporabnik**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` izvaja backend kot ta uporabnik: zajemi, ki jih zapiše, so njegovi, podrobni dnevnik je v njegovem `~/.cache/chloros/logs`, uporablja pa njegov `chloros-cli login` (`~/.chloros`), zato se najprej enkrat prijavite kot ta uporabnik. Pri namestitvi paketa se izpiše ta ukaz z vnesenim vašim uporabniškim imenom.

Brez storitve se backend (in z njim tudi PTP grandmaster) izvaja le, dokler ga nekaj ne zažene – ukaz `chloros-cli` ali skript SDK.

Enota poveže backend z nastavitvami okolja `127.0.0.1:5000` (nastavitve okolja `CHLOROS_HOST` / `CHLOROS_PORT` znotraj enote; prepisujte z `sudo systemctl edit chloros-backend@.service`) in ga ob napaki ponovno zažene po 5 sekundah.

**Storitev root.**`chloros-backend.service` izvaja backend kot**root**. Še vedno je nameščen, tako da računalnik, na katerem je omogočen, še naprej deluje, vendar vse, kar zapiše ozadje z uporabniškim računom root, pripada rootu, zato uporabnik ne more izbrisati svojih zajetih podatkov; njegov dnevnik je v `/var/log/chloros`, ki ga lahko bere skupina `adm`. Za preklop na svoj uporabniški račun:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**Backend storitve je skupen in ga odjemalec nikoli ne ustavi.** Skripti `chloros-cli` in SDK ga uporabljajo takoj, ko ga najdejo: `process` teče na njem in naloži svoj projekt namesto zadnjega. Kjer bi CLI sicer ponovno zagnal backend (`--restart` ali `process`, katerega uvoz ni našel ničesar), pa poimenuje enoto, izpiše `sudo systemctl restart <unit>` in zapre `1`, saj bi backend, ustavljen na ta način, ostal neaktiven. Ponovno ga zaženite sami s `sudo systemctl restart chloros-backend@$USER.service`. Zadnji del storitve se v mirovanju nikoli ne ustavi sam, zato ga `chloros-cli backend stop` zavrne in navede ukaz `systemctl stop`.**Kako PTP pridobi svoja vrata.** PTP uporablja vrata UDP 319/320, ki sta oba pod običajno spodnjo mejo 1024 za privilegirana vrata. `postinst` iz paketa zapiše `/etc/sysctl.d/60-chloros-ptp.conf` z `net.ipv4.ip_unprivileged_port_start = 319`, kar omogoča ozadju, da jih veže, medtem ko teče kot vaš uporabnik. Od različice 1.3.0 paket ne nastavi nobenih datotečnih zmogljivosti (`setcap`) na backendu, `libcap2-bin` pa ni več odvisnost: na razpakiranem backendu bi datotečna zmogljivost preprečila njegov zagon, zato je ne dodajajte sami.***

## Primeri skriptov v Bashu

{% hint style="info" %}
**Izhodne kode, primerne za skripte.**`chloros-cli process` zaključi delovanje z kodo `0` ob uspešnem izvajanju in**z vrednostjo, ki ni nič, ob neuspehu — vključno z izvajanjem, ki je zahtevalo slikovne izdelke, a jih ni zapisalo** (izpiše `Processing finished but wrote no image products.` ter navede ime projektne mape in običajne vzroke). Uspešna izvajanja poročajo, koliko slikovnih izdelkov je bilo zapisano (`Image products written: N`, plus `(plus M clamp masks)`, če so bile zapisane maske). Izhodne kode: `0` uspeh, `1` neuspeh, `2` napaka v argumentu, `130` prekinjeno.
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
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

Veljavnih vrednosti `--format` je natanko štiri, vsebujejo pa presledke – vedno jih vpišite v narekovajih:

| Vrednost `--format` | Izhodna mapa |
| --- | --- |
| vrednost `TIFF (16-bit)` *(privzeto)* | `tiff16` |
| vrednost `TIFF (8-bit)` | `tiff8` |
| vrednost `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` sprejema `standard` (privzeto) ali `texture-aware` (Chloros+).

### Avtomatizirana obdelava s Cronom

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Primer SDK-ja za Python

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
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
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### Backend se ne zažene po dodajanju datotečnih zmogljivosti

Ne izvajajte `setcap` na backendu. Od različice 1.3.0 paket ne nastavlja nobenih datotečnih zmogljivosti, vrata PTP 319/320 pa izhajajo iz nastavitve `sysctl`, opisane v [Kako PTP pridobi svoja vrata](#always-on-ptp-for-headless-hosts). Zmožnost datoteke postavi razpakirano ozadje v način varnega izvajanja, v katerem ne more najti lastnih knjižnic, zato se ne zažene. Če ste jo dodali, jo odstranite:

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### PTP se ne zažene / ne more vzpostaviti povezave na vratih 319

Preverite, ali je bila spodnja meja za neprivilegirana vrata znižana, in jo ponovno uporabite za trenutni zagon, če še ni bila:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Nato preverite glavni strežnik:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### `lattice` Ukazi se ne izvedejo z napako »LATTICE camera drivers not found«

Backend ni mogel naložiti izvedbenega okolja kamere LATTICE, s katerim je bil nameščen. Vsak ukaz `lattice` se izvaja na backendu; sam CLI ne vsebuje gonilnikov za kamere. Ponovno namestite Chloros (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`): `.deb` namesti izvedbeno okolje poleg ozadja v `/usr/lib/chloros/arena_runtime` in ga registrira pri dinamičnem nalagalniku. Preverite to registracijo:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
ls /usr/lib/chloros/arena_runtime | head
```

### Zagonska napaka ozadja

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

Dnevniki backenda za neuspešen zagon so v datoteki `~/.cache/chloros/logs/`. Če vrata 5000 pripadajo storitvi backenda (`systemctl status chloros-backend@$USER.service`), storitev ponovno zaženite, namesto da ustavite njen proces.

### CUDA ni bila zaznana

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

`chloros-cli selftest` poroča o istem v eni vrstici: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Manjkajoče skupne knjižnice

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### Počasen zagon na sistemih s pomnilniško kartico SD

Od različice 1.3.0 naprej se strežnik in CLI namestita v razpakirani obliki, zato se ob vsakem zagonu ne razpakirata več v začasni imenik. Začasne datoteke se samodejno shranijo v `/mnt/ssd/tmp`, če ta mapa obstaja; v nasprotnem primeru jih lahko usmerite v hiter datotečni sistem:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Posodabljanje Chlorosa na Linuxu

Ukaz `update` je na voljo le za Linux/Jetson. Preveri različico, objavljeno v kanalu za posodobitve, nastavljenem v `/etc/chloros/update.conf`, in ponudi prenos ter namestitev ustrezne različice `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

V sistemu Linuxu/Jetsonu CLI ob vsakem zagonu izvede tudi neblokirajočo preverjanje posodobitev (rezultat se shrani v predpomnilnik za eno uro v `~/.chloros/update_cache.json`) ter izpiše `Update available: vX.Y.Z`, če obstaja novejša različica. Vaše nastavitve, projekti in prijavni podatki ostanejo ohranjeni po posodobitvi.

Nadgradnja paketa med namestitvijo zaustavi vse strežniške storitve, zato tekoči skript izgubi povezavo s kamerami in senzorji, nato pa ponovno zažene vsako od njih, ki je omogočena (tista, ki je bila ročno zagnana, a nikoli omogočena, ostane zaustavljena, tako kot po ponovnem zagonu sistema).

## Odstranjevanje

```bash
sudo apt remove chloros
```

Odstranitev ustavi in onemogoči vse strežniške storitve, ponovno vzpostavi privzeto spodnjo mejo za neprivilegirana vrata (1024), odstrani datoteko z nastavitvami sprejemnega pomnilnika (vrednosti, ki so trenutno v uporabi, ostanejo do ponovnega zagona sistema), simbolično povezavo do priloženega orodja exiftool, konfiguracijo zaganjalnika Arena, pravila požarnega zidu, ki jih je dodal paket, ter Python SDK, ki ga je namestil, ter izbriše shranjena pooblastila iz predpomnilnika. Vaši projekti in datoteke `~/.chloros/` ostanejo nedotaknjeni. `sudo apt purge chloros` prav tako izbriše `/var/log/chloros`.

***

## Naslednji koraki

* [Vodnik za NVIDIA Jetson](nvidia-jetson-guide.md) — Optimizacija in namestitev, specifična za Jetson
* [CLI: ukazna vrstica](../CLI.md) — vodnik po CLI
* [API: Python SDK](../api-python-sdk.md) — vodnik po SDK
* [Referenca CLI](../reference/cli-reference.md) in [Referenca SDK](../reference/sdk-reference.md) — izčrpni seznami ukazov/API za različico 1.3.0
* [Dinamična prilagoditev računalniške zmogljivosti](../processing-architecture/dynamic-compute-adaptation.md) — kako se Chlorosprilagaja vaši strojni opremi

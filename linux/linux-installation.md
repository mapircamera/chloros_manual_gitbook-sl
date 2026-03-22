# Namestitev Linux

Chloros se za Linux distribuira v obliki paketov `.deb`, ki namestijo CLI in backend. Python SDK se namesti ločeno prek pip.

***

## Linux amd64 (x86_64)

### Sistemski zahtevi

| Zahteva | Minimalna | Priporočena |
| --- | --- | --- |
| **Distribucija** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **Procesor** | x86_64 (Intel/AMD) | Intel Core i7 ali boljši |
| **Pomnilnik (RAM)** | 8 GB | 16 GB ali več |
| **Grafična kartica** | Ni potrebna (obdelava v procesorju) | NVIDIA GPU z 4 GB+ VRAM |
| **Shranjevanje** | 2 GB prostega prostora | SSD z 10 GB+ prostega prostora |
| **Python** | Python 3.7+ (za SDK) | Python 3.10+ |

### Namestitev

Prenesite paket `.deb` in namestite:

```bash
sudo dpkg -i chloros-amd64.deb
```

Preverite namestitev:

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### Sistemski zahtevi

| Zahteva | Minimalna | Priporočena |
| --- | --- | --- |
| **Platforma** | NVIDIA Jetson z JetPack 6 | Jetson Orin NX 16 GB ali AGX Orin |
| **JetPack** | JetPack 6.x | Najnovejši JetPack 6 |
| **Pomnilnik (RAM)** | 8 GB (skupni GPU/CPU) | 16 GB+ skupni |
| **Shranjevanje** | 2 GB prostega prostora | NVMe SSD z 10 GB+ prostega prostora |
| **Python** | Python 3.7+ (za SDK) | Python 3.10+ |

### Namestitev

Prenesite paket JetPack 6 `.deb` in namestite:

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

Preverite namestitev:

```bash
chloros-cli --version
```

Za podrobne informacije o nastavitvi Jetsona, vključno z upravljanjem toplote in namestitvijo na terenu, glejte [NVIDIA Jetson Guide](nvidia-jetson-guide.md).

***

## Python SDK Namestitev (vse Linux)

Python SDK se namesti ločeno prek pip in deluje tako na amd64 kot na arm64:

```bash
pip install chloros-sdk
```

Za vključitev izbirne podpore za pretakanje napredka:

```bash
pip install chloros-sdk[progress]
```

Preverite SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Paket `.deb` namesti Chloros CLI in backend. Python SDK je ločen paket pip, ki komunicira z backendom prek lokalnega HTTP API.
{% endhint %}

***

## Konfiguracijski imeniki

Chloros na Linux sledi [Specifikaciji osnovnega imenika XDG](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html):

| Namen | Linux Pot | Windows Enakovredno |
| --- | --- | --- |
| **Konfiguracija** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **Podatki / Projekti** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **Predpomnilnik / poverilnice** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## Mesta izvedljivih datotek strežnika

Paket `.deb` namesti strežnik na standardno mesto. Paketa CLI in SDK samodejno zaznavata pot do ozadja:

| Način namestitve | Pot do ozadja |
| --- | --- |
| Paket `.deb` | `/usr/lib/chloros/chloros-backend` |
| Ročno / po meri | `/opt/mapir/chloros/backend/chloros-backend` |

Pot do backenda lahko prepisate z zastavico `--backend-exe` CLI ali s parametrom konstruktorja `backend_exe` SDK.

***

## Prva namestitev

### 1. Aktivirajte svojo licenco

Za dostop do CLI in SDK je potrebna licenca Chloros+:

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. Preverite stanje svoje licence

```bash
chloros-cli status
```

### 3. Obdelajte svoj prvi niz podatkov

```bash
chloros-cli process ~/datasets/flight001
```

### 4. Zaženite sistemsko diagnostiko

Preverite, ali je vaš sistem pravilno konfiguriran:

```bash
chloros-cli selftest
```

To izvede 7 diagnostičnih pregledov, vključno z različico, zagonom ozadja, povezljivostjo API in razpoložljivostjo CUDA/GPU.

***

## Primeri skriptov Bash

### Obdelava več podatkovnih nizov

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### Obdelava s prilagojenimi nastavitvami

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### Avtomatizirana obdelava s Cron

Dodajte v svoj crontab (`crontab -e`), da se novi nizi podatkov obdelujejo samodejno:

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK Primer

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

### CLI ni najden po namestitvi

Če `chloros-cli` ni najden po namestitvi paketa `.deb`:

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### Dostop zavrnjen

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### Zagon ozadja ni uspel

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

### CUDA ni zaznan

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### Manjkajoče skupne knjižnice

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## Posodabljanje Chloros na Linux

Uporabite vgrajeni ukaz za posodabljanje, da preverite in namestite posodobitve:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## Naslednji koraki

* [Vodnik za NVIDIA Jetson](nvidia-jetson-guide.md) — Optimizacija in namestitev, specifična za Jetson
* [CLI : Ukazna vrstica](../CLI.md) — Celoten referenčni priročnik za ukaze CLI
* [API : Python SDK](../api-python-sdk.md) — Celoten pregled SDK
* [Dinamično prilagajanje računalniških zmogljivosti](../processing-architecture/dynamic-compute-adaptation.md) — Kako se Chloros prilagaja vaši strojni opremi

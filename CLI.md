# CLI: Ukazna vrstica

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** omogoča zmogljiv dostop prek ukazne vrstice do motorja za obdelavo slik Chloros, kar omogoča avtomatizacijo, pisanje skriptov in brezglavo delovanje za vaše delovne tokove obdelave slik.

### Ključne značilnosti

* 🚀 **Avtomatizacija** – skriptno obdelovanje več podatkovnih nizov v serijah
* 🔗 **Integracija** – vgradnja v obstoječe delovne tokove in poteke
* 💻 **Delovanje brez grafičnega vmesnika** – zagon brez grafičnega vmesnika
* 🌍 **Večjezičnost** – podpora za 38 jezikov
* ⚡ **Vzporedna obdelava** – [Dinamična prilagoditev računalniške moči](processing-architecture/dynamic-compute-adaptation.md) se samodejno optimizira za vašo strojno opremo

### Zahteve

| Zahteva          | Podrobnosti                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Operacijski sistem** | Windows 10/11 (64-bit), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Licenca**          | Chloros+ ([potreben plačljiv načrt](https://cloud.mapir.camera/pricing)) |
| **Pomnilnik**           | Najmanj 8 GB RAM (priporočeno 16 GB)                                  |
| **Internet**         | Potreben za aktivacijo licence                                     |
| **Prostor na disku**       | Odvisno od velikosti projekta                                              |

{% hint style="warning" %}
**Zahteve glede licence**: CLI zahteva plačano naročnino Chloros+. Standardni (brezplačni) načrti nimajo dostopa do CLI. Za nadgradnjo obiščite [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).
{% endhint %}

## Hitri začetek

### Namestitev

#### Windows

CLI je samodejno vključen v namestitveni program Chloros:

1. Prenesite in zaženite **Chloros Installer.exe**

2. Izpolnite namestitvenega čarovnika
3. CLI je nameščen v: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
Namestitveni program samodejno doda `chloros-cli` v sistemsko pot PATH. Po namestitvi ponovno zaženite terminal.
{% endhint %}

#### Linux

Namestite paket `.deb` za vašo arhitekturo:

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Za podrobna navodila o nastavitvi Linux glejte [Namestitev Linux](linux/linux-installation.md).

### Prva namestitev

Pred uporabo CLI aktivirajte svojo licenco Chloros+:

**Windows:**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### Osnovna uporaba

Obdelajte mapo s privzetimi nastavitvami:

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## Referenca ukazov

### Splošna sintaksa

```
chloros-cli [global-options] <command> [command-options]
```

***

## Ukazi

### `process` - Obdelava slik

Obdelava slik v mapi s kalibracijo.

**Sintaksa:**

```bash
chloros-cli process <input-folder> [options]
```

**Primeri:**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### Možnosti ukaza za obdelavo

| Možnost                | Tip    | Privzeto        | Opis                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Pot    | _Obvezno_     | Mapa, ki vsebuje multispektralne slike RAW/JPG                                         |
| `-o, --output`        | Pot    | Enako kot vhod  | Izhodna mapa za obdelane slike                                                     |
| `-n, --project-name`  | Niz  | Samodejno ustvarjeno | Ime projekta po meri                                                                    |
| `--vignette`          | Oznaka    | Omogočeno        | Omogoči popravek vinjetiranja                                                             |
| `--no-vignette`       | Oznaka    | -              | Onemogoči popravek vinjetiranja                                                            |
| `--reflectance`       | Oznaka    | Omogočeno        | Omogoči kalibracijo odbojnosti                                                         |
| `--no-reflectance`    | Oznaka    | -              | Onemogoči kalibracijo odbojnosti                                                        |
| `--ppk`               | Oznaka    | Onemogočeno       | Uporabi PPK popravke iz podatkov svetlobnega senzorja .daq                                      |
| `--format`            | Izbira  | TIFF (16-bit)  | Izhodni format: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Celo število | Samodejno           | Najmanjša velikost cilja v pikslatih za zaznavanje kalibracijskega polja                          |
| `--target-clustering` | Celo število | Samodejno           | Prag združevanja ciljev (0–100)                                                    |
| `--debayer`           | Izbirno  | `standard`     | Metoda debayeringa: `standard` ali `texture-aware` (samo Chloros+)                          |
| `--target`, `--targets` | Oznaka  | Onemogočeno       | Iskanje kalibracijskih ciljev samo v podmapah »target« ali »targets« (pospeši obdelavo) |
| `--indices`           | Seznam    | Noben           | Indeksi vegetacije za izračun (npr. `--indices NDVI NDRE GNDVI`)                    |
| `--exposure-pin-1`    | Niz  | Noben           | Zakleni osvetlitev za model kamere (Pin 1)                                                 |
| `--exposure-pin-2`    | Niz  | Noben           | Zaklepanje osvetlitve za model kamere (Pin 2)                                                 |
| `--recal-interval`    | Cel številka | Samodejno           | Interval ponovne kalibracije v sekundah                                                      |
| `--timezone-offset`   | Cel številka | 0              | Časovni zamik v urah                                                               |

***

### `login` - Preverjanje pristnosti računa

Prijavite se s svojimi poverilnicami Chloros+, da omogočite obdelavo CLI.

**Sintaksa:**

```bash
chloros-cli login <email> <password>
```

**Primer:**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Posebni znaki**: Uporabite enojne narekovaje okoli gesel, ki vsebujejo znake, kot so `$`, `!` ali presledke.
{% endhint %}

**Izhod:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` – Izbriši poverilnice

Izbriši shranjene poverilnice in se odjavi iz svojega računa.

**Sintaksa:**

```bash
chloros-cli logout
```

**Primer:**

```bash
chloros-cli logout
```

**Izhod:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**SDK Uporabniki**: Python SDK ponuja tudi programsko metodo `logout()` za izbris poverilnic znotraj skriptov Python. Za podrobnosti glejte [dokumentacijo Python SDK](api-python-sdk.md#logout).
{% endhint %}

***

### `status` – Preverjanje stanja licence

Prikaže trenutno stanje licence in avtentifikacije.

**Sintaksa:**

```bash
chloros-cli status
```

**Primer:**

```bash
chloros-cli status
```

**Izhod:**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` – Preverjanje napredka izvoza

Spremljajte napredek izvoza niti 4 med ali po obdelavi.

**Sintaksa:**

```bash
chloros-cli export-status
```

**Primer:**

```bash
chloros-cli export-status
```

**Primer uporabe:** Ta ukaz pokličite med potekom obdelave, da preverite napredek izvoza.***

### `language` – Upravljanje jezika vmesnika

Preglejte ali spremenite jezik vmesnika CLI.

**Sintaksa:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Primeri:**

```bash
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Podprti jeziki (skupaj 38)

| Koda    | Jezik              | Izvirno ime      |
| ------- | --------------------- | ---------------- |
| `en`    | angleščina               | English          |
| `es`    | španščina               | Español          |
| `pt`    | portugalščina            | Português        |
| `fr`    | francoščina                | Français         |
| `de`    | nemščina                | Deutsch          |
| `it`    | italijanščina              | Italiano         |
| `ja`    | japonščina              | 日本語              |
| `ko`    | korejščina                | 한국어              |
| `zh`    | Kitajščina (poenostavljena)  | 简体中文             |
| `zh-TW` | Kitajščina (tradicionalna) | 繁體中文             |
| `ru`    | Ruščina               | Русский          |
| `nl`    | Nizozemščina                 | Nederlands       |
| `ar`    | Arabščina                | العربية          |
| `pl`    | Poljščina                | Polski           |
| `tr`    | Turščina               | Türkçe           |
| `hi`    | Hindijščina                 | हिंदी            |
| `id`    | Indonezijščina            | Bahasa Indonesia |
| `vi`    | Vietnamščina            | Tiếng Việt       |
| `th`    | Tajski                  | ไทย              |
| `sv`    | Švedski               | Svenska          |
| `da`    | Danski                | Dansk            |
| `no`    | Norveški             | Norsk            |
| `fi`    | finščina               | Suomi            |
| `el`    | grščina                 | Ελληνικά         |
| `cs`    | češčina                 | Čeština          |
| `hu`    | Madžarski             | Magyar           |
| `ro`    | Romunski              | Română           |
| `uk`    | Ukrajinski             | Українська       |
| `pt-BR` | Brazilski portugalščina  | Português Brasileiro |
| `zh-HK` | Kantonski             | 粵語             |
| `ms`    | Malajski                | Bahasa Melayu    |
| `sk`    | Slovaški                | Slovenčina       |
| `bg`    | Bolgarski             | Български        |
| `hr`    | Hrvaški              | Hrvatski         |
| `lt`    | Litovski            | Lietuvių         |
| `lv`    | Latvijščina               | Latviešu         |
| `et`    | Estonščina              | Eesti            |
| `sl`    | Slovene             | Slovenščina      |

{% hint style="success" %}
**Samodejna ohranitev**: Vaša jezikovna nastavitev se shrani v `~/.chloros/cli_language.json` in se ohrani med vsemi sejami.
{% endhint %}

***

### `set-project-folder` - Nastavitev privzete mape projekta

Spremenite lokacijo privzete mape projekta (skupno z GUI na Windows).

**Sintaksa:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Primeri:**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` - Prikaži mapo projekta

Prikaži trenutno lokacijo privzete mape projekta.

**Sintaksa:**

```bash
chloros-cli get-project-folder
```

**Primer:**

```bash
chloros-cli get-project-folder
```

**Izhod:**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` – Ponastavi na privzeto

Ponastavi mapo projekta na privzeto lokacijo.

**Sintaksa:**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` – Izvedba sistemskih diagnostičnih pregledov

Izvedite 7 diagnostičnih pregledov za preverjanje konfiguracije sistema.

**Sintaksa:**

```bash
chloros-cli selftest
```

**Izvedene diagnostične preveritve:**

1. Preverjanje različice
2. Razpoložljivost vrat (5000)
3. Zagon ozadja
4. Preizkus povezljivosti API
5. Informacije o sistemu in zaznavanje grafičnega procesorja
6. Preverjanje modelov za odstranjevanje šuma
7. Preverjanje razpoložljivosti CUDA

{% hint style="info" %}
**Koristno za odpravljanje težav**: Po namestitvi zaženite `selftest`, da preverite, ali je vaš sistem pravilno konfiguriran, zlasti na Linux/Jetson, kjer je morda potrebno preveriti nastavitve grafičnega procesorja in CUDA.
{% endhint %}

***

### `update` – Preverjanje posodobitev (samo Linux)

Preverite in namestite posodobitve CLI na sistemih Linux.

**Sintaksa:**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| Možnost    | Opis                        |
| --------- | ---------------------------------- |
| `--check` | Samo preveri posodobitve, ne namesti |

{% hint style="info" %}
Ta ukaz je na voljo samo v Linux. V Windows se posodobitve dostavijo prek namestitvenega programa.
{% endhint %}

***

## Splošne možnosti

Te možnosti veljajo za vse ukaze:

| Možnost            | Vrsta    | Privzeto       | Opis                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | Pot    | Samodejno zaznano | Pot do izvedljive datoteke strežnika                       |
| `--port`          | Celo število | 5000          | Številka vrat strežnika API                          |
| `--restart`       | Oznaka    | -             | Prisilni ponovni zagon strežnika (ukine obstoječe procese) |
| `--version`       | Zastavica    | -             | Prikaži informacije o različici in zapri                |
| `--help`          | Zastavica    | -             | Prikaži informacije o pomoči in zapri                   |

{% hint style="info" %}
**Samodejno zaznavanje ozadja**: Pot `--backend-exe` se samodejno zazna glede na platformo:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (ročno)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**Primer z globalnimi možnostmi:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux:**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## Priročnik za nastavitve obdelave

### Vzporedna obdelava in dinamična prilagoditev računalniške moči

Chloros 1.1.0 vključuje [dinamično prilagoditev računalniške moči](processing-architecture/dynamic-compute-adaptation.md) — procesni motor **samodejno zazna vašo strojno opremo** in izbere optimalno strategijo:

| Platforma | Strategija | Delavci | Potek | Opombe |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` | Učinkovita raba pomnilnika, serializirana |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` | Sočasna obdelava GPU |
| **Namizni računalnik z 8 GB GPU** | `GPU_SINGLE` | 3 | `tiled_gpu` | Dobra zmogljivost namiznega računalnika |
| **Namizni računalnik z grafično kartico 12 GB+** | `GPU_PARALLEL` | 3–4 | `fused_gpu` | Optimalna zmogljivost namiznega računalnika |
| **Sistem samo s CPU** | `CPU_PARALLEL` | jedra – 1 | `cpu_fallback` | Grafična kartica ni potrebna |

{% hint style="success" %}
**Ročna konfiguracija ni potrebna!** Chloros samodejno zazna vaš CPU, GPU, RAM in (na Jetsonu) toplotne senzorje, nato pa samodejno konfigurira optimalni procesni potek.
{% endhint %}

### Metode debayeringa

| Metoda | CLI Oznaka | Kakovost | Hitrost | Licenca |
| --- | --- | --- | --- | --- |
| **Standardna (hitra, srednja kakovost)** | `--debayer standard` | Dobra | Hitra | Brezplačna / Chloros+ |
| **Texture Aware (počasno, najvišja kakovost)** | `--debayer texture-aware` | Najvišja | Počasno | Samo Chloros+ |

Privzeta metoda debayerja je **Standardna**. Metoda**Texture Aware** uporablja model za odstranjevanje šuma AI/ML za izhod najvišje kakovosti, vendar zahteva licenco Chloros+ in grafično kartico NVIDIA.

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### Popravek vinjetiranja

**Kaj počne:** Popravlja padec svetlobe na robovih slike (temnejši koti, ki so pogosti na fotografijah).

* **Privzeto omogočeno** – Večina uporabnikov naj to pusti omogočeno
* Uporabite `--no-vignette` za onemogočanje

{% hint style="success" %}
**Priporočilo**: Vedno vklopite popravek vinjete, da zagotovite enakomerno svetlost po celotnem okvirju.
{% endhint %}

### Kalibracija odbojnosti

Pretvori surove vrednosti senzorja v standardizirane odstotke odbojnosti z uporabo kalibracijskih plošč.

* **Privzeto vklopljeno** – nujno za analizo vegetacije
* Zahteva kalibracijske ciljne plošče na slikah
* Uporabite `--no-reflectance` za onemogočanje

{% hint style="info" %}
**Zahteve**: Za natančno pretvorbo odbojnosti poskrbite, da so kalibracijske plošče na slikah pravilno osvetljene in vidne.
{% endhint %}

### PPK-popravki

**Kaj počne:** Uporablja poprocesne kinematične popravke z uporabo podatkov dnevnika DAQ-A-SD za izboljšano natančnost GPS.

* **Privzeto onemogočeno**
* Uporabite `--ppk` za vklop
* Zahteva datoteke .daq v projektni mapi iz MAPIR svetlobnega senzorja DAQ-A-SD.

### Izhodni formati

<table><thead><tr><th width="197">Oblika</th><th width="130.20001220703125">Bitna globina</th><th width="116.5999755859375">Velikost datoteke</th><th>Najbolj primerno za</th></tr></thead><tbody><tr><td><strong>TIFF (16-bitni)</strong> ⭐</td><td>16-bitno celo število</td><td>Velika</td><td>GIS analiza, fotogrametrija (priporočeno)</td></tr><tr><td><strong>TIFF (32-bitni, odstotek)</strong></td><td>32-bitno število s plavajočo vejico</td><td>Zelo veliko</td><td>Znanstvena analiza, raziskave</td></tr><tr><td><strong>PNG (8-bitni)</strong></td><td>8-bitno celo število</td><td>Srednje</td><td>Vizualni pregled, deljenje na spletu</td></tr><tr><td><strong>JPG (8-bit)</strong></td><td>8-bitno celo število</td><td>Majhno</td><td>Hiter predogled, stisnjena izhodna datoteka</td></tr></tbody></table>***

## Avtomatizacija in skriptiranje

### Obdelava v seriji v PowerShellu (Windows)

Samodejna obdelava več map z nizi podatkov v Windows:

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Windows skript za obdelavo v paketih (Windows)

Preprost krog za obdelavo v paketih na Windows:

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Obdelava v paketu Bash (Linux)

Obdelava več map z nizi podatkov na Linux:

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### Skript za avtomatizacijo Python (večplatformski)

Napredna avtomatizacija z obdelavo napak (deluje na Windows in Linux):

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## Potek obdelave

### Standardni potek

1. **Vnos**: Mapa, ki vsebuje pare slik RAW/JPG
2. **Odkrivanje**: CLI samodejno skenira za podprte slikovne datoteke
3. **Obdelava**: Vzporedni način se prilagaja vašim jedrom CPU (Chloros+)
4. **Izhod**: Ustvari podmapke po modelih fotoaparatov z obdelanimi slikami

### Primer strukture izhodnih datotek

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### Ocene časa obdelave

Tipični časi obdelave za 100 slik (vsaka 12 MP):

| Platforma | Način | Predvideni čas | Opombe |
| --- | --- | --- | --- |
| **Namizni računalnik z grafično kartico 12 GB+** | `GPU_PARALLEL` | 5–10 min | Najhitrejša možnost |
| **Namizni računalnik z 8 GB GPU** | `GPU_SINGLE` | 10–15 min | Dobra zmogljivost |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 15–25 min | Obdelava na robu omrežja |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 30–60 min | Omejen pomnilnik |
| **Samo CPU** | `CPU_PARALLEL` | 20–40 min | GPU ni potreben |

{% hint style="info" %}
**Nasvet za izboljšanje zmogljivosti**: Čas obdelave se razlikuje glede na število slik, ločljivost, metodo debayeringa in strojno opremo. Debayering z upoštevanjem teksture traja znatno dlje kot standardni. Za podrobnosti glejte [Dinamično prilagajanje računalniške zmogljivosti](processing-architecture/dynamic-compute-adaptation.md).
{% endhint %}

***

## Odpravljanje težav

### CLI ni bilo mogoče najti

**Windows Napaka:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Windows Rešitve:**

1. Preverite mesto namestitve:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Uporabite polno pot, če ni v PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Ročno dodajte v PATH:
   * Odprite Lastnosti sistema → Okoljske spremenljivke
   * Uredite spremenljivko PATH
   * Dodajte: `C:\Program Files\Chloros\resources\cli`
   * Ponovno zaženite terminal

**Linux Napaka:**

```
chloros-cli: command not found
```

**Linux Rešitve:**

1. Preverite namestitev:

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. Ponovno naložite lupino:

```bash
source ~/.bashrc
```

3. Preverite dovoljenja:

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### Backend se ni uspel zagnati**Napaka:**

```

Backend failed to start within 30 seconds
```

**Rešitve:**

1. Preverite, ali backend že teče (ga najprej zaprite)
2. Preverite, ali ga požarni zid ne blokira (Windows) ali preverite razpoložljivost vrat (Linux: `lsof -i :5000`)
3. Poskusite z drugim vratom:

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. Prisilno ponovno zaženite backend:

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. Pri napaki Linux preverite, ali izvedljiva datoteka backenda obstaja:

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### Težave z licenco / avtentifikacijo**Napaka:**

```

Chloros+ license required for CLI access
```

**Rešitve:**

1. Preverite, ali imate aktivno naročnino Chloros+
2. Prijavite se s svojimi poverilnicami:

```bash
chloros-cli login user@example.com 'password'
```

3. Preverite stanje licence:

```bash
chloros-cli status
```

4. Obrnite se na podporo: info@mapir.camera

***

### Slike niso bile najdene**Napaka:**

```

No images found in the specified folder
```

**Rešitve:**

1. Preverite, ali mapa vsebuje podprte formate (.RAW, .TIF, .JPG)
2. Preverite, ali je pot do mape pravilna (za poti z vmesnimi presledki uporabite narekovaje)
3. Preverite, ali imate dovoljenja za branje mape
4. Preverite, ali so končnice datotek pravilne

***

### Obdelava se ustavi ali zamrzne**Rešitve:**

1. Preverite razpoložljivi prostor na disku (poskrbite, da je dovolj prostora za izhodne datoteke)
2. Zaprite druge aplikacije, da sprostite pomnilnik
3. Zmanjšajte število slik (obdelujte v serijah)

***

### Vrata so že v uporabi**Napaka:**

```

Port 5000 is already in use
```

**Rešitve:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux:**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## Pogosta vprašanja

### V: Ali potrebujem licenco za CLI?

**O:**Da! Za CLI je potrebna plačljiva**licenca Chloros+**.

* ❌ Standardni (brezplačni) načrt: CLI onemogočen
* ✅ Chloros+ (plačljivi) načrti: CLI v celoti omogočen

Naročite se na: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### V: Ali lahko uporabljam CLI na strežniku brez grafičnega vmesnika?**O:** Da! CLI deluje popolnoma brez grafičnega vmesnika. To je glavni primer uporabe na Linux.**Windows strežnik:**
* Windows strežnik 2016 ali novejši
* Nameščen Visual C++ Redistributable

**Linux strežnik:**
* Ubuntu 20.04+ / Debian 11+ (amd64) ali JetPack 6 (arm64)
* Namestitev prek paketa `.deb`

**Obe platformi:**
* Najmanj 8 GB RAM (priporočeno 16 GB)
* Enkratna aktivacija licence: `chloros-cli login user@example.com 'password'`

***

### V: Kje se shranijo obdelane slike?**O:**Privzeto se obdelane slike shranijo v**isto mapo kot vhodne** v podmapah po modelih kamer (npr. `Survey3N_RGN/`).

Uporabite možnost `-o`, da določite drugo izhodno mapo:

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### V: Ali lahko obdelujem več map hkrati?**O:** Ne neposredno z enim ukazom, vendar lahko uporabite skripte za zaporedno obdelavo map. Glejte poglavje [Avtomatizacija in skripti](CLI.md#automation--scripting).***

### V: Kako shranim izhod CLI v datoteko dnevnika?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash:**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### V: Kaj se zgodi, če med obdelavo pritisnem Ctrl+C?**O:** CLI bo:

1. Obdelavo ustavil na primeren način
2. Zaprl backend
3. Izstopil s kodo 130

Delno obdelane slike lahko ostanejo v izhodni mapi.

***

### V: Ali lahko avtomatiziram obdelavo CLI?**O:** Seveda! CLI je zasnovan za avtomatizacijo. Glejte [Avtomatizacija in skriptiranje](CLI.md#automation--scripting) za PowerShell (Windows), Batch (Windows), Bash (Linux) in Python (medplatformski).***

### V: Kako preverim različico CLI?**O:**

```bash
chloros-cli --version
```

**Izhod:**

```

Chloros CLI 1.1.0
```

***

## Pomoč

### Pomoč v ukazni vrstici

Oglejte si informacije o pomoči neposredno v CLI:

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Kanali podpore

* **E-pošta**: info@mapir.camera
* **Spletna stran**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Cene**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Celotni primeri

### Primer 1: Osnovna obdelava

Obdelava s privzetimi nastavitvami (vignette, odbojnost):

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### Primer 2: Visokokakovostni znanstveni izhod

32-bitni float TIFF:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### Primer 3: Hitra obdelava predogleda

8-bitni PNG brez kalibracije za hiter pregled:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### Primer 4: Obdelava s PPK-popravki

Uporabite PPK-popravke z odbojnostjo:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### Primer 5: Mesto izpisa po meri

Obdelava v drugo mesto z določenim formatom:

**Windows:**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux:**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### Primer 6: Potek avtentifikacije

Celoten potek avtentifikacije (enak na vseh platformah):

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Primer 7: Uporaba več jezikov

Sprememba jezika vmesnika (enaka na vseh platformah):

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```

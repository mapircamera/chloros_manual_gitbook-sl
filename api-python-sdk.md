# API : Python SDK

**Chloros Python SDK** omogoča programski dostop do procesorja za obdelavo slik Chloros, kar omogoča avtomatizacijo, prilagojene delovne tokove in nemoteno integracijo z vašimi aplikacijami Python in raziskovalnimi procesi.

### Ključne značilnosti

* 🐍 **Native Python** - Čisti, Pythonic API za obdelavo slik
* 🔧 **Popoln dostop do API** - Popoln nadzor nad obdelavo Chloros
* 🚀 **Avtomatizacija** - Ustvarjanje prilagojenih delovnih tokov za obdelavo v serijah
* 🔗 **Integracija** – vgradite Chloros v obstoječe aplikacije Python
* 📊 **Pripravljen za raziskave** – idealen za znanstvene analitične poti
* ⚡ **Vzporedna obdelava** – prilagaja se vašim jedrom CPU (Chloros+)

### Zahteve

| Zahteva          | Podrobnosti                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros Desktop**  | Mora biti nameščen lokalno                                           |
| **Licenca**          | Chloros+ ([potreben plačljiv načrt](https://cloud.mapir.camera/pricing)) |
| **Operacijski sistem** | Windows 10/11 (64-bitni)                                              |
| **Python**           | Python 3.7 ali višji                                                |
| **Pomnilnik**           | Najmanj 8 GB RAM (priporočeno 16 GB)                                  |
| **Internet**         | Potreben za aktivacijo licence                                     |

{% hint style=&quot;warning&quot; %}
**Zahteve za licenco**: Python SDK zahteva plačano naročnino Chloros+ za dostop do API. Standardni (brezplačni) načrti nimajo dostopa do API/SDK. Obiščite [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing), da nadgradite.
{% endhint %}

## Hitri začetek

### Namestitev

Namestite prek pip:

```bash
pip install chloros-sdk
```

{% hint style=&quot;info&quot; %}
**Prva namestitev**: Pred uporabo SDK aktivirajte svojo licenco Chloros+ tako, da odprete Chloros, Chloros (brskalnik) ali Chloros CLI in se prijavite s svojimi poverilnicami. To je treba narediti le enkrat.
{% endhint %}

### Osnovna uporaba

Obdelajte mapo z le nekaj vrsticami:

```python
from chloros_sdk import process_folder

# One-line processing
results = process_folder("C:\\DroneImages\\Flight001")
```

### Popoln nadzor

Za napredne delovne tokove:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## Navodila za namestitev

### Predpogoji

Pred namestitvijo SDK se prepričajte, da imate:

1. **Chloros Desktop** nameščen ([download](download.md))
2. **Python 3.7+** ([python.org](https://www.python.org))
3. **Aktivna licenca Chloros+** ([nadgradnja](https://cloud.mapir.camera/pricing))

### Namestitev prek pip

**Standardna namestitev:**

```bash
pip install chloros-sdk
```

**S podporo za spremljanje napredka:**

```bash
pip install chloros-sdk[progress]
```

**Namestitev za razvoj:**

```bash
pip install chloros-sdk[dev]
```

### Preverjanje namestitve

Preverite, ali je SDK pravilno nameščen:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Prva namestitev

### Aktiviranje licence

SDK uporablja isto licenco kot Chloros, Chloros (brskalnik) in Chloros CLI. Aktivirajte enkrat prek GUI ali CLI:

1. Odprite **Chloros ali Chloros (brskalnik)** in se prijavite na kartici Uporabnik <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> . Ali pa odprite **CLI**.
2. Vnesite svoje poverilnice Chloros+ in se prijavite
3. Licenca se shrani v lokalni predpomnilnik (ohrani se tudi po ponovnem zagonu)

{% namig style=&quot;success&quot; %}
**Enkratna nastavitev**: Po prijavi prek GUI ali CLI, SDK samodejno uporabi shranjeno licenco. Dodatna avtentifikacija ni potrebna!
{% endhint %}

### Preizkus povezave

Preverite, ali se SDK lahko poveže z Chloros:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## API Referenca

### Razred ChlorosLocal

Glavni razred za lokalno obdelavo slik Chloros.

#### Konstruktor

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**Parametri:**

| Parameter                 | Tip | Privzeto                   | Opis                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL lokalnega Chloros backenda          |
| `auto_start_backend`      | bool | `True`                    | Samodejni zagon backenda, če je potrebno |
| `backend_exe`             | str  | `None` (samodejno zaznavanje)      | Pot do izvedljive datoteke backenda            |
| `timeout`                 | int  | `30`                      | Časovni limit zahtevka v sekundah            |
| `backend_startup_timeout` | int  | `60`                      | Časovni limit za zagon backenda (sekunde) |

**Primeri:**

```python
# Default (auto-start backend)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom timeout
chloros = ChlorosLocal(timeout=60)
```

***

### Metode

#### `create_project(project_name, camera=None)`

Ustvari nov projekt Chloros.

**Parametri:**

| Parameter      | Tip | Obvezno | Opis                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Da      | Ime projekta                                     |
| `camera`       | str  | Ne       | Predloga kamere (npr. »Survey3N\_RGN«, »Survey3W\_OCN«) |

**Vrne:** `dict` - Odgovor na ustvarjanje projekta

**Primer:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Uvozi slike iz mape.

**Parametri:**

| Parameter     | Tip     | Obvezno | Opis                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Da      | Pot do mape s slikami         |
| `recursive`   | bool     | Ne       | Iskanje podmap (privzeto: False) |

**Vrne:** `dict` – Rezultati uvoza s številom datotek

**Primer:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Konfigurirajte nastavitve obdelave.

**Parametri:**

| Parameter                 | Tip | Privzeto                 | Opis                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | »Visoka kakovost (hitrejša)« | Metoda Debayer                  |
| `vignette_correction`     | bool | `True`                  | Omogoči popravek vinjete      |
| `reflectance_calibration` | bool | `True`                  | Omogoči kalibracijo odbojnosti  |
| `indices`                 | list | `None`                  | Indeksi vegetacije za izračun |
| `export_format`           | str  | &quot;TIFF (16-bit)&quot;         | Izhodni format                   |
| `ppk`                     | bool | `False`                 | Omogoči PPK popravke          |
| `custom_settings`         | dict | `None`                  | Napredne prilagojene nastavitve        |

**Izvozni formati:**

* `"TIFF (16-bit)"` – priporočljivo za GIS/fotogrametrijo
* `"TIFF (32-bit, Percent)"` – znanstvena analiza
* `"PNG (8-bit)"` – vizualni pregled
* `"JPG (8-bit)"` – stisnjena izhodna datoteka

**Razpoložljivi indeksi:**

NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 in drugi.

**Primer:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

Obdelajte slike projekta.

**Parametri:**

| Parameter           | Tip     | Privzeto      | Opis                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Način obdelave: »vzporedno« ali »zaporedno«   |
| `wait`              | bool     | `True`       | Počakaj na zaključek                       |
| `progress_callback` | callable | `None`       | Funkcija za povratni klic napredka (napredek, msg) |
| `poll_interval`     | float    | `2.0`        | Interval za preverjanje napredka (sekunde)   |

**Vrne:** `dict` – Rezultati obdelave

{% hint style=&quot;warning&quot; %}
**Vzporedni način**: Zahteva licenco Chloros+. Samodejno se prilagaja vašim jedrom CPU (do 16 delavcev).
{% endhint %}

**Primer:**

```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```

***

#### `get_config()`

Pridobi trenutno konfiguracijo projekta.

**Vrne:** `dict` – Trenutna konfiguracija projekta

**Primer:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Pridobi informacije o stanju backenda.

**Vrne:** `dict` - Stanje backenda

**Primer:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
```

***

#### `shutdown_backend()`

Zaustavi backend (če ga je zagnal SDK).

**Primer:**

```python
chloros.shutdown_backend()
```

***

### Priročne funkcije

#### `process_folder(folder_path, **options)`

Enovrstična priročna funkcija za obdelavo mape.

**Parametri:**

| Parameter                 | Tip     | Privzeto         | Opis                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Obvezno        | Pot do mape s slikami     |
| `project_name`            | str      | Samodejno ustvarjeno  | Ime projekta                   |
| `camera`                  | str      | `None`          | Predloga kamere                |
| `indices`                 | list     | `["NDVI"]`      | Indeksi za izračun           |
| `vignette_correction`     | bool     | `True`          | Omogoči popravek vinjete     |
| `reflectance_calibration` | bool     | `True`          | Omogoči kalibracijo odbojnosti |
| `export_format`           | str      | &quot;TIFF (16-bit)&quot; | Izhodni format                  |
| `mode`                    | str      | `"parallel"`    | Način obdelave                |
| `progress_callback`       | callable | `None`          | Povratni klic napredka              |

**Vrne:** `dict` – Rezultati obdelave

**Primer:**

```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```

***

## Podpora za upravitelja konteksta

SDK podpira upravitelje konteksta za samodejno čiščenje:

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***

## Celotni primeri

### Primer 1: Osnovna obdelava

Obdelava mape s privzetimi nastavitvami:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Primer 2: Prilagojeni delovni tok

Popoln nadzor nad obdelovalnim procesom:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### Primer 3: Obdelava več map v paketu

Obdelava več podatkovnih nizov letov:

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***

### Primer 4: Integracija raziskovalnega procesa

Integracija Chloros z analizo podatkov:

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```

***

### Primer 5: Prilagojeno spremljanje napredka

Napredno spremljanje napredka z beleženjem:

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***

### Primer 6: Obravnavanje napak

Zanesljivo obravnavanje napak za uporabo v proizvodnji:

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros Desktop is installed."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```

***

### Primer 7: Orodje za ukazno vrstico

Izdelajte prilagojeno orodje CLI z SDK:

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    
    args = parser.parse_args()
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**Uporaba:**

```bash
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI
```

***

## Obravnavanje izjem

SDK zagotavlja posebne razrede izjem za različne vrste napak:

### Hierarhija izjem

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### Primeri izjem

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros Desktop is installed.")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Napredne teme

### Prilagojena konfiguracija backenda

Uporabite prilagojeno lokacijo ali konfiguracijo backenda:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Neoviranje obdelave

Začnite obdelavo in nadaljujte z drugimi nalogami:

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### Upravljanje pomnilnika

Za velike podatkovne nize obdelujte v paketih:

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***

## Odpravljanje težav

### Backend se ne zažene

**Težava:** SDK ne more zažeti backenda.

**Rešitve:**

1. Preverite, ali je nameščen Chloros Desktop:

```python
import os
backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Preverite, ali Windows požarni zid ne blokira
3. Poskusite z ročno potjo backenda:

```python
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")
```

***

### Licenca ni zaznana

**Težava:** SDK opozarja na manjkajočo licenco

**Rešitve:**

1. Odprite Chloros, Chloros (brskalnik) ali Chloros CLI in se prijavite.
2. Preverite, ali je licenca shranjena v predpomnilniku:

```python
from pathlib import Path
import os

# Check cache location (Windows)
cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
print(f"Cache exists: {cache_path.exists()}")
```

3. Obrnite se na podporo: info@mapir.camera

***

### Napake pri uvozu

**Problem:** `ModuleNotFoundError: No module named 'chloros_sdk'`

**Rešitve:**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### Časovna omejitev obdelave

**Težava:** Časovni limit obdelave

**Rešitve:**

1. Podaljšajte časovni limit:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Obdelujte manjše serije
3. Preverite razpoložljivi prostor na disku
4. Nadzorujte sistemske vire

***

### Vrata so že v uporabi

**Težava:** Vrata 5000 v ozadju so zasedena

**Rešitve:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Ali poiščite in zaprite proces, ki povzroča konflikt:

```powershell
# PowerShell
Get-NetTCPConnection -LocalPort 5000
```

***

## Nasveti za izboljšanje zmogljivosti

### Optimizirajte hitrost obdelave

1. **Uporabite vzporedni način** (zahteva Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Zmanjšajte ločljivost izhoda** (če je to sprejemljivo)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Onemogočite nepotrebne indekse**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Obdelava na SSD** (ne HDD)

***

### Optimizacija pomnilnika

Za velike podatkovne nize:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Obdelava v ozadju

Sprostite Python za druge naloge:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## Primeri integracije

### Integracija Django

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyter Notebook

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***

## Pogosta vprašanja

### V: Ali SDK zahteva internetno povezavo?

**O:** Samo za začetno aktiviranje licence. Po prijavi prek Chloros, Chloros (brskalnik) ali Chloros CLI, se licenca shrani v lokalni predpomnilnik in deluje brez povezave 30 dni.

***

### V: Ali lahko uporabljam SDK na strežniku brez grafičnega vmesnika?

**O:** Da! Zahteve:

* Windows Server 2016 ali novejši
* Chloros nameščen (enkratno)
* Licenca aktivirana na katerem koli računalniku (licenca v predpomnilniku kopirana na strežnik)

***

### V: Kakšna je razlika med Desktop, CLI in SDK?

| Funkcija         | Desktop GUI | CLI ukazna vrstica | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Vmesnik**   | Klikni in izberi | Ukazna vrstica          | Python API  |
| **Najbolj primerno za**    | Vizualno delo | Pisanje skriptov        | Integracija |
| **Avtomatizacija**  | Omejena     | Dobra             | Odlična   |
| **Prilagodljivost** | Osnovna       | Dobra             | Največja     |
| **Licenca**     | Chloros+    | Chloros+         | Chloros+    |

***

### V: Ali lahko distribuiram aplikacije, ustvarjene z SDK?

**O:** Kodo SDK lahko integrirate v svoje aplikacije, vendar:

* Končni uporabniki morajo imeti nameščen Chloros.
* Končni uporabniki morajo imeti aktivne licence Chloros+.
* Komercialna distribucija zahteva licenco OEM.

Za vprašanja v zvezi z licenco OEM se obrnite na info@mapir.camera.

***

### V: Kako posodobim SDK?

```bash
pip install --upgrade chloros-sdk
```

***

### V: Kje se shranijo obdelane slike?

Privzeto v poti projekta:

```
Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### V: Ali lahko obdelujem slike iz skriptov Python, ki se izvajajo po urniku?

**O:** Da! Uporabite Windows Task Scheduler s skriptami Python:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("C:\\Flights\\Today")
```

Načrtujte z Task Schedulerjem, da se izvaja dnevno.

***

### V: Ali SDK podpira async/await?

**O:** Trenutna različica je sinhrona. Za asinhrono delovanje uporabite `wait=False` ali izvedite v ločenem niti:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

## Pomoč

### Dokumentacija

* **API Referenca**: Ta stran

### Kanali podpore

* **E-pošta**: info@mapir.camera
* **Spletna stran**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Cene**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Vzorec kode

Vsi tukaj navedeni primeri so preizkušeni in pripravljeni za uporabo. Kopirajte jih in prilagodite za svoj primer uporabe.

***

## Licenca

**Lastniška programska oprema** – Copyright (c) 2025 MAPIR Inc.

SDK zahteva aktivno naročnino Chloros+. Neodobrena uporaba, distribucija ali sprememba je prepovedana.

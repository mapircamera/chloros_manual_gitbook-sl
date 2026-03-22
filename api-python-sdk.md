# API : Python SDK

**Chloros Python SDK** omogoča programski dostop do motorja za obdelavo slik Chloros, kar omogoča avtomatizacijo, prilagojene delovne tokove in nemoteno integracijo z vašimi aplikacijami Python in raziskovalnimi procesi.

### Ključne značilnosti

* 🐍 **Nativni Python** – Čist, Pythonski API za obdelavo slik
* 🔧 **Popoln dostop do API** – Popoln nadzor nad obdelavo Chloros
* 🚀 **Avtomatizacija** – Ustvarjanje prilagojenih delovnih tokov za obdelavo v paketih
* 🔗 **Integracija** – Vgradnja Chloros v obstoječe aplikacije Python
* 📊 **Pripravljeno za raziskave** – Idealno za znanstvene analitične procese
* ⚡ **Vzporedna obdelava** – prilagaja se številu jeder vašega procesorja (Chloros+)

### Zahteve

| Zahteva          | Podrobnosti                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros nameščen** | Windows: namestitveni program za namizne računalnike; Linux: paket `.deb`                  |
| **Licenca**          | Chloros+ ([potreben plačljiv načrt](https://cloud.mapir.camera/pricing)) |
| **Operacijski sistem** | Windows 10/11 (64-bitni), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Python**           | Python 3.7 ali višja                                                |
| **Pomnilnik**           | Najmanj 8 GB RAM (priporočeno 16 GB)                                  |
| **Internet**         | Potreben za aktivacijo licence                                     |

{% hint style="warning" %}
**Zahteve glede licence**: Python SDK zahteva plačano naročnino Chloros+ za dostop do API. Standardni (brezplačni) načrti nimajo dostopa do API/SDK. Za nadgradnjo obiščite [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).
{% endhint %}

## Hitri začetek

### Namestitev

Namestite prek pip:

```bash
pip install chloros-sdk
```

{% hint style="info" %}
**Prva namestitev**: Pred uporabo SDK aktivirajte svojo licenco Chloros+ tako, da odprete Chloros, Chloros (Brskalnik) ali Chloros CLI in se prijavite s svojimi podatki. To je treba narediti le enkrat. Na Linux (brez grafičnega vmesnika) uporabite: `chloros-cli login user@example.com 'password'`
{% endhint %}

### Osnovna uporaba

Obdelajte mapo z le nekaj vrsticami:

```python
from chloros_sdk import process_folder

# One-line processing (Windows)
results = process_folder("C:\\DroneImages\\Flight001")

# One-line processing (Linux)
results = process_folder("/home/user/drone_images/flight001")
```

{% hint style="info" %}
**Potki za različne platforme**: Primeri kode na tej strani uporabljajo potke v slogu Windows (npr. `C:\\DroneImages\\Flight001`). V sistemu Linux namesto tega uporabite poti v slogu Linux (npr. `/home/user/drone_images/flight001` ali `~/drone_images/flight001`). SDK deluje enako na obeh platformah.
{% endhint %}

### Popoln nadzor

Za napredne delovne tokove:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")  # Windows
# chloros.import_images("/home/user/drone_images/flight001")  # Linux

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

1. **Chloros nameščen** — Windows: Namestitveni program za namizje ([prenos](download.md)); Linux: paket `.deb` ([Namestitev](linux/linux-installation.md))
2. **Python 3.7+** nameščen ([python.org](https://www.python.org))
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

SDK uporablja isto licenco kot Chloros, Chloros (brskalnik) in Chloros CLI. Aktivirajte enkrat prek grafičnega vmesnika ali CLI:**Windows:**Odprite**Chloros ali Chloros (brskalnik)** in se prijavite na zavihku Uporabnik <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> ali uporabite CLI.**Linux:** Uporabite CLI (GUI ni na voljo):

```bash
chloros-cli login user@example.com 'your_password'
```

Licenca je shranjena v lokalnem predpomnilniku in ostane veljavna tudi po ponovnem zagonu računalnika.

{% hint style="success" %}
**Enkratna nastavitev**: Po prijavi prek grafičnega vmesnika ali CLI, SDK samodejno uporabi licenco iz predpomnilnika. Dodatno preverjanje pristnosti ni potrebno!
{% endhint %}

{% hint style="info" %}
**Odjava**: Uporabniki SDK lahko programsko izbrišejo shranjena pooblastila z metodo `logout()`. Glejte [metodo logout()](#logout) v referenčnem priročniku API.
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

## Referenca API

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
| `auto_start_backend`      | bool | `True`                    | Samodejni zagon ozadja, če je potrebno |
| `backend_exe`             | str  | `None` (samodejno zaznavanje)      | Pot do izvedljive datoteke ozadja            |
| `timeout`                 | int  | `30`                      | Časovna omejitev zahtevka v sekundah            |
| `backend_startup_timeout` | int  | `60`                      | Časovna omejitev za zagon ozadja (v sekundah) |

**Primeri:**

```python
# Default (auto-start backend, auto-detect path on Windows and Linux)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path (Windows)
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom backend path (Linux)
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")

# Custom timeout with longer startup (e.g., for Jetson)
chloros = ChlorosLocal(timeout=60, backend_startup_timeout=120)
```

{% hint style="info" %}
**Samodejno zaznavanje med platformami**: SDK samodejno poskusi pravo pot do ozadja za vašo platformo:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (ročno)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

***

### Metode

#### `create_project(project_name, camera=None)`

Ustvarite nov projekt Chloros.

**Parametri:**

| Parameter      | Tip | Obvezno | Opis                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Da      | Ime projekta                                     |
| `camera`       | str  | Ne       | Predloga kamere (npr. »Survey3N\_RGN«, &quot;Survey3W\_OCN&quot;) |

**Vrne:** `dict` - Odgovor na ustvarjanje projekta**Primer:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Uvoz slike iz mape.

**Parametri:**

| Parameter     | Tip     | Obvezno | Opis                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Da      | Pot do mape s slikami         |
| `recursive`   | bool     | Ne       | Iskanje v podmapah (privzeto: False) |

**Vrne:** `dict` - Rezultati uvoza s številom datotek**Primer:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Nastavi nastavitve obdelave.

**Parametri:**

| Parameter                 | Tip | Privzeto                 | Opis                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | &quot;Standard (Hitro, Srednja kakovost)&quot; | Metoda Debayer            |
| `vignette_correction`     | bool | `True`                  | Omogoči popravek vinjetiranja      |
| `reflectance_calibration` | bool | `True`                  | Omogoči kalibracijo odbojnosti  |
| `indices`                 | seznam | `None`                  | Indeksi vegetacije za izračun |
| `export_format`           | str  | &quot;TIFF (16-bit)&quot;         | Izhodni format                   |
| `ppk`                     | bool | `False`                 | Omogoči PPK popravke          |
| `custom_settings`         | dict | `None`                  | Napredne prilagojene nastavitve        |

**Izvozni formati:**

* `"TIFF (16-bit)"` – Priporočeno za GIS/fotogrametrijo
* `"TIFF (32-bit, Percent)"` – Znanstvena analiza
* `"PNG (8-bit)"` – Vizualni pregled
* `"JPG (8-bit)"` – Stisnjen izhod

**Razpoložljivi indeksi:**NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 in drugi.**Primer:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
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
| `mode`              | str      | `"parallel"` | Način obdelave: »parallel« ali »serial«   |
| `wait`              | bool     | `True`       | Počakaj na zaključek                       |
| `progress_callback` | callable | `None`       | Funkcija povratnega klica za napredek (progress, msg) |
| `poll_interval`     | float    | `2.0`        | Interval poizvedovanja za napredek (sekunde)   |

**Vrne:** `dict` - Rezultati obdelave

{% hint style="warning" %}
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

**Vrne:** `dict` - Trenutna konfiguracija projekta**Primer:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Pridobi informacije o stanju ozadja, vključno z napredkom obdelave po posameznih niti.

**Vrne:** `dict` - Stanje ozadja z naslednjo strukturo:

```python
{
    "running": True,
    "url": "http://localhost:5000",
    "processing": {
        "percent": 75.0,
        "phase": "processing"
    },
    "export": {
        "percent": 50.0,
        "phase": "exporting",
        "active": True
    }
}
```

**Primer:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
print(f"Processing: {status['processing']['percent']}%")
print(f"Export: {status['export']['percent']}% - Active: {status['export']['active']}")
```

***

#### `shutdown_backend()`

Zaustavi backend (če je bil zagnan s SDK).

**Primer:**

```python
chloros.shutdown_backend()
```

***

#### `logout()`

Izbriše shranjena pooblastila iz lokalnega sistema.

**Opis:**

Programsko se odjavi z odstranitvijo shranjenih poverilnic za avtentifikacijo. To je koristno za:
* Preklapljanje med različnimi računi Chloros+
* Izbris poverilnic v avtomatiziranih okoljih
* Varnostne namene (npr. odstranitev poverilnic pred odstranitvijo programa)

**Vrne:** `dict` - Rezultat operacije odjave**Primer:**

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Clear cached credentials
result = chloros.logout()
print(f"Logout successful: {result}")

# After logout, login required via GUI/CLI/Browser before next SDK use
```

{% hint style="info" %}
**Potrebna ponovna avtentifikacija**: Po klicu `logout()` se morate ponovno prijaviti prek Chloros, Chloros (brskalnik) ali Chloros CLI, preden uporabite SDK.
{% endhint %}

***

### Funkcije za lažje delo

#### `process_folder(folder_path, **options)`

Enovrstična priročna funkcija za obdelavo mape.

**Parametri:**

| Parameter                 | Tip     | Privzeto         | Opis                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Obvezno        | Pot do mape s slikami     |
| `project_name`            | str      | Samodejno ustvarjeno  | Ime projekta                   |
| `camera`                  | str      | `None`          | Predloga kamere                |
| `indices`                 | seznam     | `["NDVI"]`      | Indeksi za izračun           |
| `vignette_correction`     | bool     | `True`          | Omogoči popravek vinjete     |
| `reflectance_calibration` | bool     | `True`          | Omogoči kalibracijo odbojnosti |
| `export_format`           | str      | &quot;TIFF (16-bit)&quot; | Izhodni format                  |
| `mode`                    | str      | `"parallel"`    | Način obdelave                |
| `progress_callback`       | callable | `None`          | Povratni klic napredka              |

**Vrne:** `dict` - Rezultati obdelave**Primer:**

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

## Podpora upraviteljev konteksta

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

{% hint style="info" %}
**Uporabniki Linux**: Vsi spodnji primeri uporabljajo poti Windows. Pot `C:\\...` zamenjajte s svojimi potmi Linux (npr. `/home/user/...` ali `~/...`). Vse funkcionalnosti SDK so enake na vseh platformah.
{% endhint %}

### Primer 1: Osnovna obdelava

Obdelajte mapo s privzetimi nastavitvami:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Primer 2: Prilagojeni potek dela

Popoln nadzor nad potekom obdelave:

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
    debayer="Standard (Fast, Medium Quality)",
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

Obdelava več naborov podatkov o letih:

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

### Primer 4: Integracija v raziskovalni proces

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

Zanesljivo obravnavanje napak za uporabo v produkciji:

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
        return False, f"Backend error: {e}. Ensure Chloros is installed (Windows installer or Linux .deb package)."
    
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

### Primer 7: Upravljanje računa in odjava

Programsko upravljanje poverilnic:

```python
from chloros_sdk import ChlorosLocal

def switch_account():
    """Clear credentials to switch to a different account"""
    try:
        chloros = ChlorosLocal()
        
        # Clear current credentials
        result = chloros.logout()
        print("✓ Credentials cleared successfully")
        print("Please log in with new account via Chloros, Chloros (Browser), or CLI")
        
        return True
    
    except Exception as e:
        print(f"✗ Logout failed: {e}")
        return False

def secure_cleanup():
    """Remove credentials for security purposes"""
    try:
        chloros = ChlorosLocal()
        chloros.logout()
        print("✓ Credentials removed for security")
        
    except Exception as e:
        print(f"Warning: Cleanup error: {e}")

# Switch accounts
if switch_account():
    print("\nRe-authenticate via Chloros GUI/CLI/Browser before next SDK use")

# Or perform secure cleanup
# secure_cleanup()
```

***

### Primer 8: Orodje za ukazno vrstico

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
    parser.add_argument('--logout', action='store_true',
                       help='Clear cached credentials before processing')
    
    args = parser.parse_args()
    
    # Handle logout if requested
    if args.logout:
        from chloros_sdk import ChlorosLocal
        chloros = ChlorosLocal()
        chloros.logout()
        print("Credentials cleared. Please re-login via Chloros GUI/CLI/Browser.")
        return 0
    
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
# Process multiple folders
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI

# Clear cached credentials
python my_processor.py --logout
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
    print("Backend failed to start. Ensure Chloros is installed (Windows installer or Linux .deb package).")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Napredne teme

### Konfiguracija prilagojenega ozadja

Uporabite prilagojeno lokacijo ali konfiguracijo ozadja:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Nezablokirajoča obdelava

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

Za velike nize podatkov obdelujte v serijah:

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

**Težava:** SDK ne more zagnati backenda**Rešitve:**

1. Preverite, ali je Chloros nameščen:

```python
import os
import platform

# Auto-detect backend path
if platform.system() == "Windows":
    backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
else:
    backend_path = "/usr/lib/chloros/chloros-backend"

print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Preverite požarni zid (Windows) ali razpoložljivost vrat (Linux: `lsof -i :5000`)
3. Poskusite z ročno potjo do backenda:

```python
# Windows
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")

# Linux
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")
```

***

### Licenca ni zaznana**Problem:** SDK opozarja na manjkajočo licenco**Rešitve:**

1. Odprite Chloros, Chloros (brskalnik) ali Chloros CLI in se prijavite.
2. Preverite, ali je licenca shranjena v predpomnilniku:

```python
from pathlib import Path
import os
import platform

# Check cache location
if platform.system() == "Windows":
    cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
else:
    cache_path = Path.home() / '.cache' / 'chloros'

print(f"Cache exists: {cache_path.exists()}")
```

3. Če imate težave s poverilnicami, izbrišite shranjene poverilnice in se ponovno prijavite:

```python
from chloros_sdk import ChlorosLocal

# Clear cached credentials
chloros = ChlorosLocal()
chloros.logout()

# Then login again via Chloros, Chloros (Browser), or Chloros CLI
```

4. Obrnite se na podporo: info@mapir.camera

***

### Napake pri uvozu**Težava:** `ModuleNotFoundError: No module named 'chloros_sdk'`**Rešitve:**

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

### Časovna omejitev obdelave**Težava:** Obdelava se prekine zaradi časovne omejitve**Rešitve:**

1. Podaljšajte časovno omejitev:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Obdelujte manjše serije
3. Preverite razpoložljivi prostor na disku
4. Nadzorujte sistemske vire

***

### Vrata so že v uporabi**Težava:** Vrata strežnika 5000 so zasedena**Rešitve:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Ali poiščite in zaprite proces, ki povzroča konflikt:

```powershell
# Windows PowerShell
Get-NetTCPConnection -LocalPort 5000
```

```bash
# Linux
lsof -i :5000
kill $(lsof -t -i :5000)
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

4. **Obdelujte na SSD** (ne na HDD)***

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

### Integracija z Djangom

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

**O:** Samo za začetno aktivacijo licence. Po prijavi prek Chloros, Chloros (brskalnik) ali Chloros CLI se licenca shrani v lokalni predpomnilnik in deluje brez povezave 30 dni.***

### V: Ali lahko uporabljam SDK na strežniku brez grafičnega vmesnika?**O:** Da! SDK deluje brez grafičnega vmesnika na strežnikih Windows in Linux.**Linux (priporočeno za brez grafičnega vmesnika):**
* Namestite prek paketa `.deb`
* Aktivirajte licenco: `chloros-cli login user@example.com 'password'`

**Strežnik Windows:**
* Strežnik Windows 2016 ali novejši
* Nameščen Chloros (enkratno)
* Licenca aktivirana prek CLI ali na katerem koli računalniku

***

### V: Kakšna je razlika med Desktop, CLI in SDK?

| Funkcija         | Desktop GUI | CLI ukazna vrstica | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Vmesnik**   | Točka-klik | Ukazna vrstica          | Python API  |
| **Najbolj primerno za**    | Vizualno delo | Skriptiranje        | Integracija |
| **Avtomatizacija**  | Omejena     | Dobra             | Odlična   |
| **Prilagodljivost** | Osnovna       | Dobra             | Največja     |
| **Licenca**     | Chloros+    | Chloros+         | Chloros+    |***

### V: Ali lahko distribuiram aplikacije, zgrajene z SDK?**O:** Kodo SDK lahko vključite v svoje aplikacije, vendar:

* Končni uporabniki morajo imeti nameščeno Chloros
* Končni uporabniki morajo imeti aktivne licence Chloros+
* Komercialna distribucija zahteva licenco OEM

Za poizvedbe glede licenc OEM se obrnite na info@mapir.camera.

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

### V: Ali lahko obdelujem slike iz skriptov Python, ki se izvajajo po urniku?**O:** Da! Uporabite načrtovalnik operacijskega sistema s skripti Python:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("/data/flights/today")  # Linux
# results = process_folder("C:\\Flights\\Today")  # Windows
```

**Windows:** Načrtujte prek načrtovalnika opravil, da se izvaja dnevno.**Linux:** Načrtujte prek cron:

```cron
# Run at 2 AM daily
0 2 * ** /usr/bin/python3 /home/user/scheduled_processing.py >> /var/log/chloros.log 2>&1
```

***

### V: Ali SDK podpira async/await?**O:** Trenutna različica je sinhrona. Za asinhrono delovanje uporabite `wait=False` ali zagnajte v ločenem niti:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

### V: Kako preklopim med različnimi računi Chloros+?**O:** Uporabite metodo `logout()` za izbris shranjenih poverilnic, nato se ponovno prijavite z novim računom:

```python
from chloros_sdk import ChlorosLocal

# Clear current credentials
chloros = ChlorosLocal()
chloros.logout()

# Re-login via Chloros, Chloros (Browser), or Chloros CLI with new account
```

Po odjavi se avtentificirajte z novim računom prek grafičnega vmesnika, brskalnika ali CLI, preden ponovno uporabite SDK.

***

## Pomoč

### Dokumentacija

* **API Referenca**: Ta stran

### Kanali podpore

* **E-pošta**: info@mapir.camera
* **Spletna stran**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Cene**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Primer kode

Vsi tukaj navedeni primeri so preizkušeni in pripravljeni za uporabo v produkcijskem okolju. Kopirajte jih in prilagodite za svoj primer uporabe.

***

## Licenca**Lastniška programska oprema** – Avtorske pravice (c) 2025 MAPIR Inc.

SDK zahteva aktivno naročnino Chloros+. Neodobrena uporaba, distribucija ali spreminjanje je prepovedano.

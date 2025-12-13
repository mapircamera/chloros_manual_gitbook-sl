# CLI: ukazna vrstica

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** omogoča zmogljiv dostop prek ukazne vrstice do procesorja za obdelavo slik Chloros, kar omogoča avtomatizacijo, skriptiranje in brezglavo delovanje za vaše delovne tokove obdelave slik.

### Ključne značilnosti

* 🚀 **Avtomatizacija** – skriptno obdelovanje več podatkovnih nizov
* 🔗 **Integracija** – vključitev v obstoječe delovne tokove in poti
* 💻 **Delovanje brez grafičnega vmesnika** – delovanje brez grafičnega vmesnika
* 🌍 **Večjezičnost** – podpora za 38 jezikov
* ⚡ **Vzporedna obdelava** – dinamično se prilagaja vašemu CPU (do 16 vzporednih delavcev)

### Zahteve

| Zahteva          | Podrobnosti                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Operacijski sistem** | Windows 10/11 (64-bit)                                              |
| **Licenca**          | Chloros+ ([potreben plačljiv načrt](https://cloud.mapir.camera/pricing)) |
| **Pomnilnik**           | Najmanj 8 GB RAM (priporočeno 16 GB)                                  |
| **Internet**         | Potreben za aktivacijo licence                                     |
| **Prostor na disku**       | Odvisno od velikosti projekta                                              |

{% hint style=&quot;warning&quot; %}
**Zahteve za licenco**: CLI zahteva plačljivo naročnino Chloros+. Standardni (brezplačni) načrti nimajo dostopa do CLI. Za nadgradnjo obiščite [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).
{% endhint %}

## Hitri začetek

### Namestitev

CLI je samodejno vključen v namestitveni program Chloros:

1. Prenesite in zaženite **Chloros Installer.exe**.
2. Izvedite namestitvenega čarovnika.
3. CLI je nameščen v: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style=&quot;success&quot; %}
Namestitveni program samodejno doda `chloros-cli` v sistemsko pot PATH. Po namestitvi ponovno zaženite terminal.
{% endhint %}

### Prva namestitev

Preden začnete uporabljati CLI, aktivirajte svojo licenco Chloros+:

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### Osnovna uporaba

Obdelajte mapo s privzetimi nastavitvami:

```powershell
chloros-cli process "C:\Images\Dataset001"
```

***

## Referenca ukazov

### Splošna sintaksa

```
chloros-cli [global-options] <command> [command-options]
```

***

## Ukazi

### `process` – Obdelava slik

Obdelava slik v mapi s kalibracijo.

**Sintaksa:**

```bash
chloros-cli process <input-folder> [options]
```

**Primer:**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### Možnosti ukazov za obdelavo

| Možnost                | Tip    | Privzeto        | Opis                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Pot    | _Obvezno_     | Mapa, ki vsebuje večspektralne slike RAW/JPG                                         |
| `-o, --output`        | Pot    | Enako kot vhod  | Izhodna mapa za obdelane slike                                                     |
| `-n, --project-name`  | Niz  | Samodejno ustvarjen | Ime projekta po meri                                                                    |
| `--vignette`          | Oznaka    | Omogočeno        | Omogoči popravek vinjete                                                             |
| `--no-vignette`       | Oznaka    | -              | Onemogoči popravek vinjete                                                            |
| `--reflectance`       | Oznaka    | Omogočeno        | Omogoči kalibracijo odbojnosti                                                         |
| `--no-reflectance`    | Oznaka    | -              | Onemogoči kalibracijo odbojnosti                                                        |
| `--ppk`               | Oznaka    | Onemogočeno       | Uporabi popravke PPK iz podatkov svetlobnega senzorja .daq                                      |
| `--format`            | Izbirno  | TIFF (16-bit)  | Izhodni format: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Celo število | Samodejno           | Minimalna ciljna velikost v slikovnih točkah za zaznavanje kalibracijskega panela                          |
| `--target-clustering` | Celo število | Samodejno           | Prag združevanja ciljev (0–100)                                                    |
| `--exposure-pin-1`    | Niz  | Nič           | Zaklepanje osvetlitve za model kamere (Pin 1)                                                 |
| `--exposure-pin-2`    | Niz  | Nič           | Zaklepanje osvetlitve za model kamere (Pin 2)                                                 |
| `--recal-interval`    | Celo število | Samodejno           | Interval ponovne kalibracije v sekundah                                                      |
| `--timezone-offset`   | Celo število | 0              | Časovni zamik v urah                                                               |

***

### `login` – Preverjanje pristnosti računa

Prijavite se s svojimi poverilnicami Chloros+, da omogočite obdelavo CLI.

**Sintaksa:**

```bash
chloros-cli login <email> <password>
```

**Primer:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Posebni znaki**: Uporabite enojne narekovaje okoli gesel, ki vsebujejo znake, kot so `$`, `!` ali presledki.
{% endhint %}

**Izhod:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` – Izbriši poverilnice

Izbriše shranjene poverilnice in se odjavi iz vašega računa.

**Sintaksa:**

```bash
chloros-cli logout
```

**Primer:**

```powershell
chloros-cli logout
```

**Izhod:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

***

### `status` – Preveri stanje licence

Prikaži trenutno stanje licence in avtentifikacije.

**Sintaksa:**

```bash
chloros-cli status
```

**Primer:**

```powershell
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

### `export-status` – Preveri napredek izvoza

Spremlja napredek izvoza niti 4 med ali po obdelavi.

**Sintaksa:**

```bash
chloros-cli export-status
```

**Primer:**

```powershell
chloros-cli export-status
```

**Primer uporabe:** Pokličite ta ukaz med obdelavo, da preverite napredek izvoza.

***

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

```powershell
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
| `en`    | angleščina               | angleščina          |
| `es`    | španščina               | španščina          |
| `pt`    | portugalščina            | portugalščina        |
| `fr`    | francoščina                | francoščina         |
| `de`    | Nemščina                | Deutsch          |
| `it`    | Italijanščina               | Italiano         |
| `ja`    | Japonščina              | 日本語              |
| `ko`    | Korejščina                | 한국어              |
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
| `fi`    | Finski               | Suomi            |
| `el`    | Grški                 | Ελληνικά         |
| `cs`    | Češki                 | Čeština          |
| `hu`    | Madžarski             | Magyar           |
| `ro`    | Romunski              | Română           |
| `uk`    | Ukrajinski             | Українська       |
| `pt-BR` | Brazilski portugalski  | Português Brasileiro |
| `zh-HK` | kantonščina             | 粵語             |
| `ms`    | malajščina                 | Bahasa Melayu    |
| `sk`    | slovaščina                | Slovenčina       |
| `bg`    | Bolgarščina             | Български        |
| `hr`    | Hrvaščina              | Hrvatski         |
| `lt`    | Litovščina            | Lietuvių         |
| `lv`    | Latvijščina               | Latviešu         |
| `et`    | Estonščina              | Eesti            |
| `sl`    | Slovenščina             | Slovenščina      |

{% hint style=&quot;success&quot; %}
**Samodejna vztrajnost**: Vaša jezikovna nastavitev je shranjena v `~/.chloros/cli_language.json` in ostane nespremenjena v vseh sejah.
{% endhint %}

***

### `set-project-folder` – Nastavitev privzete mape projekta

Spremenite lokacijo privzete mape projekta (skupna z GUI).

**Sintaksa:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Primer:**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder` – Prikaži mapo projekta

Prikaži trenutno privzeto lokacijo mape projekta.

**Sintaksa:**

```bash
chloros-cli get-project-folder
```

**Primer:**

```powershell
chloros-cli get-project-folder
```

**Izhod:**

```
ℹ Current project folder: C:\Projects\2025
```

***

### `reset-project-folder` – Ponastavi na privzeto

Ponastavi projektno mapo na privzeto lokacijo.

**Sintaksa:**

```bash
chloros-cli reset-project-folder
```

***

## Globalne možnosti

Te možnosti veljajo za vse ukaze:

| Možnost          | Tip    | Privzeto       | Opis                                      |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | Pot    | Samodejno zaznan | Pot do izvedljive datoteke backenda                       |
| `--port`        | Celo število | 5000          | Številka vrat backenda API                          |
| `--restart`     | Oznaka    | -             | Prisilni ponovni zagon backenda (ukine obstoječe procese) |
| `--version`     | Oznaka    | -             | Prikaži informacije o različici in zapri                |
| `--help`        | Oznaka    | -             | Prikaži informacije o pomoči in zapri                   |

**Primer z globalnimi možnostmi:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## Vodnik po nastavitvah obdelave

### Vzporedna obdelava

Chloros+ CLI **samodejno prilagaja** vzporedno obdelavo zmogljivostim vašega računalnika:

**Kako deluje:**

* Zazna jedra CPU in RAM
* Dodeli delavce: **2× jedra CPU** (uporablja hipernitnost)
* **Največ: 16 vzporednih delavcev** (zaradi stabilnosti)

**Sistemski nivoji:**

| Tip sistema   | CPU        | RAM      | Delavci  | Zmogljivost     |
| ------------- | ---------- | -------- | -------- | --------------- |
| **Visokokakovostni**  | 16+ jeder  | 32+ GB   | Do 16 | Največja hitrost   |
| **Srednji razred** | 8–15 jeder | 16–31 GB | 8–16     | Odlična hitrost |
| **Nizki razred**   | 4–7 jeder  | 8–15 GB  | 4–8      | Dobra hitrost      |

{% hint style=&quot;success&quot; %}
**Avtomatska optimizacija**: CLI samodejno zazna specifikacije vašega sistema in konfigurira optimalno vzporedno obdelavo. Ročna konfiguracija ni potrebna!
{% endhint %}

### Metode debayerja

CLI uporablja **visoko kakovost (hitrejši)** kot privzeti in priporočeni algoritem debayerja:

| Metoda                      | Kakovost | Hitrost | Opis                                 |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **Visoka kakovost (hitrejša)** ⭐ | ⭐⭐⭐⭐    | ⚡⚡⚡   | Algoritem, ki upošteva robove (privzeto, priporočeno) |

### Popravek vinjete

**Kaj počne:** Popravi padec svetlobe na robovih slike (temnejši koti, ki so pogosti na slikah iz kamere).

* **Privzeto omogočeno** – večina uporabnikov naj to možnost pusti omogočeno.
* Za onemogočanje uporabite `--no-vignette`.

{% hint style=&quot;success&quot; %}
**Priporočilo**: Vedno omogočite popravek vinjete, da zagotovite enakomerno svetlost po celotnem okviru.
{% endhint %}

### Kalibracija odbojnosti

Pretvori surove vrednosti senzorja v standardizirane odstotke odbojnosti z uporabo kalibracijskih plošč.

* **Privzeto omogočeno** – bistveno za analizo vegetacije.
* Zahteva kalibracijske ciljne plošče v slikah.
* Za onemogočanje uporabite `--no-reflectance`.

{% hint style=&quot;info&quot; %}
**Zahteve**: Za natančno pretvorbo odbojnosti se prepričajte, da so kalibracijske plošče pravilno osvetljene in vidne na vaših slikah.
{% endhint %}

### PPK popravki

**Kaj počne:** Uporablja popravke kinematične obdelave po obdelavi z uporabo podatkov dnevnika DAQ-A-SD za izboljšano natančnost GPS.

* **Privzeto onemogočeno**
* Za omogočanje uporabite `--ppk`
* Zahteva datoteke .daq v projektni mapi iz MAPIR DAQ-A-SD svetlobnega senzorja.

### Izhodni formati

<table><thead><tr><th width="197">Oblika</th><th width="130.20001220703125">Bitna globina</th><th width="116.5999755859375">Velikost datoteke</th><th>Najbolj primerno za</th></tr></thead><tbody><tr><td><strong>TIFF (16-bitni)</strong> ⭐</td><td>16-bitno celo število</td><td>Velika</td><td>GIS analiza, fotogrametrija (priporočeno)</td></tr><tr><td><strong>TIFF (32-bitni, odstotek)</strong></td><td>32-bitni plavajoči</td><td>Zelo velika</td><td>Znanstvena analiza, raziskave</td></tr><tr><td><strong>PNG (8-bitni)</strong></td><td>8-bitno celo število</td><td>Srednje</td><td>Vizualni pregled, spletno deljenje</td></tr><tr><td><strong>JPG (8-bitni)</strong></td><td>8-bitno celo število</td><td>Majhno</td><td>Hiter predogled, stisnjena izhodna datoteka</td></tr></tbody></table>***

## Avtomatizacija in skriptiranje

### PowerShell Batch Processing

Samodejna obdelava več map z nizi podatkov:

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

### Windows Batch Script

Preprost krog za obdelavo v paketih:

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

### Python skript za avtomatizacijo

Napredna avtomatizacija z obravnavanjem napak:

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

## Delovni tok obdelave

### Standardni delovni tok

1. **Vnos**: Mapa, ki vsebuje pare slik RAW/JPG
2. **Odkrivanje**: CLI samodejno skenira podprte slikovne datoteke
3. **Obdelava**: Paralelni način se prilagaja vašim jedrom CPU (Chloros+)
4. **Izhod**: Ustvari podmapo za model kamere z obdelanimi slikami

### Primer strukture izhoda

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

| Način              | Čas      | Strojna oprema                                     |
| ----------------- | --------- | -------------------------------------------- |
| **Vzporedni način** | 5–10 min  | i7/Ryzen 7, 16 GB RAM, SSD (do 16 delavcev) |
| **Vzporedni način** | 10–15 min | i5/Ryzen 5, 8 GB RAM, HDD (do 8 delavcev)   |

{% hint style=&quot;info&quot; %}
**Nasvet za izboljšanje zmogljivosti**: Čas obdelave se razlikuje glede na število slik, ločljivost in specifikacije računalnika.
{% endhint %}

***

## Odpravljanje težav

### CLI ni najden

**Napaka:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Rešitve:**

1. Preverite mesto namestitve:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Če ni v PATH, uporabite polno pot:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Ročno dodajte v PATH:
   * Odprite Lastnosti sistema → Okoljske spremenljivke
   * Uredite spremenljivko PATH
   * Dodajte: `C:\Program Files\Chloros\resources\cli`
   * Ponovno zaženite terminal.

***

### Backend se ni uspel zagnati.

**Napaka:**

```
Backend failed to start within 30 seconds
```

**Rešitve:**

1. Preverite, ali backend že teče (najprej ga zaprite).
2. Preverite, ali Windows požarni zid ne blokira.
3. Poskusite z drugim vratom:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. Prisilno znova zaženite backend:

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### Težave z licenco/avtentifikacijo

**Napaka:**

```
Chloros+ license required for CLI access
```

**Rešitve:**

1. Preverite, ali imate aktivno naročnino Chloros+.
2. Prijavite se s svojimi poverilnicami:

```powershell
chloros-cli login user@example.com 'password'
```

3. Preverite stanje licence:

```powershell
chloros-cli status
```

4. Obrnite se na podporo: info@mapir.camera

***

### Ni najdenih slik

**Napaka:**

```
No images found in the specified folder
```

**Rešitve:**

1. Preverite, ali mapa vsebuje podprte formate (.RAW, .TIF, .JPG).
2. Preverite, ali je pot do mape pravilna (za poti z presledki uporabite narekovaje).
3. Preverite, ali imate pravice za branje mape.
4. Preverite, ali so končnice datotek pravilne.

***

### Obdelava se ustavi ali zamrzne

**Rešitve:**

1. Preverite prosti prostor na disku (preverite, ali je dovolj prostora za izhodne datoteke).
2. Zaprite druge aplikacije, da sprostite pomnilnik.
3. Zmanjšajte število slik (obdelujte v serijah).

***

### Vrata so že v uporabi

**Napaka:**

```
Port 5000 is already in use
```

**Rešitev:**

Določite drugo vrata:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## Pogosta vprašanja

### V: Potrebujem licenco za CLI?

**O:** Da! CLI zahteva plačljivo **Chloros+ licenco**.

* ❌ Standardni (brezplačni) načrt: CLI onemogočen
* ✅ Chloros+ (plačljivi) načrti: CLI v celoti omogočen

Naročite se na: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### V: Ali lahko uporabljam CLI na strežniku brez grafičnega vmesnika?

**O:** Da! CLI deluje popolnoma brez grafičnega vmesnika. Zahteve:

* Windows Server 2016 ali novejši
* Nameščen Visual C++ Redistributable
* Zadostna količina pomnilnika RAM (najmanj 8 GB, priporočeno 16 GB)
* Enkratna aktivacija licence GUI na katerem koli računalniku

***

### V: Kje se shranijo obdelane slike?

**O:** Privzeto se obdelane slike shranijo v **isti mapi kot vhodne** v podmapah modelov kamer (npr. `Survey3N_RGN/`).

Uporabite možnost `-o`, da določite drugo izhodno mapo:

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### V: Ali lahko obdelam več map hkrati?

**O:** Ne neposredno z enim ukazom, vendar lahko uporabite skripte za zaporedno obdelavo map. Glejte poglavje [Avtomatizacija in skripte](CLI.md#automation--scripting).

***

### V: Kako shranim izhod CLI v dnevniško datoteko?

**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Paketno:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

***

### V: Kaj se zgodi, če med obdelavo pritisnem Ctrl+C?

**O:** CLI bo:

1. Prenehalo z obdelavo
2. Ustavil backend
3. Zaprl se s kodo 130

Delno obdelane slike lahko ostanejo v izhodni mapi.

***

### V: Ali lahko avtomatiziram obdelavo CLI?

**O:** Seveda! CLI je zasnovan za avtomatizacijo. Glejte [Avtomatizacija in skriptiranje](CLI.md#automation--scripting) za primere PowerShell, Batch in Python.

***

### V: Kako preverim različico CLI?

**O:**

```powershell
chloros-cli --version
```

**Izhod:**

```
Chloros CLI 1.0.2
```

***

## Pomoč

### Pomoč v ukazni vrstici

Pomoč si lahko ogledate neposredno v CLI:

```powershell
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
* **Cene**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

## Celotni primeri

### Primer 1: Osnovna obdelava

Obdelava s privzetimi nastavitvami (vinjeta, odbojnost):

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### Primer 2: Visokokakovostni znanstveni izhod

32-bitni plavajoči TIFF:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### Primer 3: Hitra obdelava predogleda

8-bitni PNG brez kalibracije za hiter pregled:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### Primer 4: Obdelava s PPK-popravki

Uporaba PPK-popravkov z odbojnostjo:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### Primer 5: Prilagojena lokacija izhoda

Obdelava na drugem disku v določenem formatu:

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### Primer 6: Potek avtentifikacije

Zaključite potek avtentifikacije:

```powershell
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
chloros-cli process "C:\Datasets\Field_A"

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Primer 7: Uporaba več jezikov

Spremenite jezik vmesnika:

```powershell
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
chloros-cli process "C:\Vuelos\Campo_A"

# Change back to English
chloros-cli language en
```

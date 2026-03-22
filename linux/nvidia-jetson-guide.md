# Priročnik za NVIDIA Jetson

Chloros na platformi NVIDIA Jetson omogoča obdelavo multispektralnih slik na robu omrežja – na terenu, na brezpilotnih letalih (UAV) in v oddaljenih napravah. Chloros samodejno prepozna vaš model Jetson in optimizira strategijo obdelave za vašo strojno opremo.

***

## Podprti modeli Jetson

| Model                | RAM            | Strategija obdelave                                   | Priporočena uporaba                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB skupnega | `GPU_PARALLEL` (4 delavci)                            | Največja zmogljivost, veliki nizi podatkov                      |
| **Jetson Orin NX**   | 8–16 GB skupnega pomnilnika  | `GPU_PARALLEL` (3 delavci, 16 GB) / `GPU_SINGLE` (8 GB) | Priporočilo za uporabo v zraku in na terenu |
| **Jetson Orin Nano** | 8 GB skupnega pomnilnika     | `GPU_SINGLE` (1 delavec)                               | Osnovni računalniški sistem na robu omrežja                                 |
| **Jetson Nano**      | 4–8 GB skupno   | `GPU_SINGLE` (1 delavec)                               | Vstopni model z omejenim pomnilnikom                          |

{% hint style="info" %}
**Starejši modeli Jetson** (TX2, TX1, Xavier NX) morda niso podprti. Zmogljivost se bo razlikovala glede na razpoložljivi pomnilnik GPU in zmogljivosti CUDA.
{% endhint %}

***

## Zahteve

* **JetPack 6.x** (priporočena najnovejša različica)
* **NVIDIA CUDA** (vključena v JetPack)
* **Chloros+ licenca** (potrebna za dostop do CLI/SDK)

## Namestitev

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

Za splošne podrobnosti o namestitvi Linux glejte [Namestitev Linux](linux-installation.md).

***

## Dinamična prilagoditev računalniških zmogljivosti na Jetsonu

Chloros samodejno zazna vaš model Jetson in izbere optimalno strategijo obdelave. **Ročno nastavljanje ni potrebno.**

### Kako deluje

Ob zagonu Chloros profilira vaš sistem:

1. **Zazna model Jetson** prek `/proc/device-tree/model`
2. **Prebere razpoložljivi GPU/skupni pomnilnik**

3.**Izbere strategijo obdelave** (`GPU_PARALLEL`, `GPU_SINGLE` ali `CPU_PARALLEL`)
4. Samodejno **nastavi število delavcev, vrsto poti in dodelitev pomnilnika**

### Delovanje po posameznih modelih

| Model Jetson                | Strategija       | Delavci | Potek                       | Sočasnost |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8 GB**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (pomnilniško učinkovit) | Serializirano  |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | Serializirano  |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | Serializirano  |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 3       | `fused_gpu` (polna pot GPU)    | Sočasno  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | Sočasno  |

{% hint style="success" %}
**Jetson Orin NX 16 GB** je idealna rešitev za namestitev na robu omrežja – uporablja strategijo `GPU_PARALLEL` s 3 sočasnimi delavci, kar omogoča resnično vzporedno obdelavo GPU v kompaktni obliki.
{% endhint %}

Ključna razlika med platformami je **pomnilnik**. Jetson Nano z 8 GB skupnega pomnilnika mora slike obdelovati po eno naenkrat z uporabo pomnilniško učinkovitega pristopa s ploščicami, medtem ko lahko Orin NX z 16 GB hkrati obdeluje 3 slike prek GPU z uporabo združenega potoka z višjo prepustnostjo.

Za celoten referenčni priročnik o prilagajanju računalniške moči glejte [Dinamično prilagajanje računalniške moči](../processing-architecture/dynamic-compute-adaptation.md).

***

## Upravljanje toplote

Naprave Jetson imajo omejeno toplotno rezervo, zlasti v zaprtih ali letalskih namestitvah. Chloros vključuje samodejno spremljanje temperature in omejevanje zmogljivosti:

| Temperatura         | Ukrep                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normalno delovanje — polna hitrost obdelave          |
| **70 °C** (Opozorilo)  | Samodejno zmanjšanje velikosti serije                   |
| **80 °C** (Kritično) | Agresivno omejevanje — manjša sočasnost         |
| **90 °C** (Izklop) | Popolna zaustavitev obdelave GPU — potrebno hlajenje |

{% hint style="warning" %}
**Zagotovite ustrezno prezračevanje in odvod toplote** za trajno obdelavo, zlasti v zaprtih ohišjih na terenu ali v letalskih sistemih. Toplotno omejevanje bo zmanjšalo zmogljivost obdelave, da se zaščiti strojna oprema.
{% endhint %}

***

## Upravljanje pomnilnika

Naprave Jetson uporabljajo **enotni pomnilnik** — GPU in CPU si delita isti fizični RAM. To pomeni, da navedeni VRAM (npr. 15,3 GB na Orin NX 16 GB) ni namenski pomnilnik GPU; deli si ga z operacijskim sistemom in drugimi procesi.

### Priporočila za zamenjavo

Za velike podatkovne nize ali obdelavo Texture Aware debayer lahko Chloros priporoči ustvarjanje prostora za zamenjavo:

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Ocene pomnilnika na sliko:**

* Standardni debayer: ~10 MB na sliko
* Texture Aware debayer: ~15 MB na sliko

Chloros samodejno izračuna potrebni pomnilnik na podlagi velikosti vašega niza podatkov in vas opozori, če je priporočljiv swap.

### Nadomestna rešitev za OOM (Out of Memory)

Če se med obdelavo zazna stanje pomanjkanja pomnilnika:

1. Chloros samodejno zmanjša število delavcev GPU
2. Preklopi iz poteka `fused_gpu` na potek `tiled_gpu` (bolj učinkovit z vidika pomnilnika)
3. Nadaljuje obdelavo z zmanjšano prepustnostjo, namesto da se sesuje

***

## Uporaba na terenu

### Upoštevanje porabe energije

| Model Jetson     | Tipična poraba energije | Opombe                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5–10 W              | USB-C ali cilindrični priključek    |
| Jetson Orin Nano | 7–15 W              | DC-vtičnica          |
| Jetson Orin NX   | 10–25 W             | DC-vtičnica          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD ali DC-vtičnica |

Načrtujte porabo energije za neprekinjeno obdelavo – največja poraba energije se pojavi med GPU-intenzivnim Thread 3 (Processing).

### Priporočila za shranjevanje

* **NVMe SSD** se močno priporoča za namestitve arm64
* SD kartice so prepočasne za obdelavo — uporabljajte jih le kot zagonski medij
* Načrtujte 2–3-kratno velikost surovih slikovnih podatkov za obdelano izhodno datoteko

### Delovanje brez zaslona prek SSH

Chloros CLI je idealen za namestitve Jetson brez zaslona:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

### Avtomatizirana obdelava s systemd

Ustvarite storitev systemd za avtomatizirano obdelavo:

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

Povežite jo s časovnikom systemd za načrtovano obdelavo:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## Primeri poteka dela

### Osnovna obdelava Jetson

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI
```

### Python SDK na Jetsonu

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Obdelava več letov v seriji

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Priporočeni sistemi Jetson za uporabo na terenu

Za uporabo na terenu in v zraku upoštevajte naslednje možnosti nosilnih plošč Jetson Orin NX 16 GB:

* **V zraku/droni**: Sistemi z odpornostjo proti vibracijam (MIL-STD), lahki (pod 300 g), pasivno hlajenje
* **Robustna uporaba na terenu**: Vodotesna ohišja IP67/IP69K s povezljivostjo kamere PoE GigE
* **Minimalno/proračunsko**: Razvojni kompleti z dodatnimi ohišji

Za konkretna priporočila glede strojne opreme za vaš scenarij uporabe se obrnite na [MAPIR podporo](https://www.mapir.camera/community/contact).

***

## Naslednji koraki

* [Linux Namestitev](linux-installation.md) — Splošni podatki o namestitvi Linux
* [Dinamična prilagoditev računalniške zmogljivosti](../processing-architecture/dynamic-compute-adaptation.md) — Celoten pregled strategij računalniške zmogljivosti
* [Obdelovalni potek](../processing-architecture/processing-pipeline.md) — Razumevanje 4-nitnega poteka
* [CLI : Ukazna vrstica](../CLI.md) — Celoten referenčni priročnik za CLI
* [API : Python SDK](../api-python-sdk.md) — Celoten referenčni priročnik za SDK

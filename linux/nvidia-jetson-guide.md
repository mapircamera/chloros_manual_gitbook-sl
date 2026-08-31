# Priročnik za NVIDIA Jetson

Chloros na platformi NVIDIA Jetson omogoča obdelavo multispektralnih slik na robu omrežja – na terenu, na brezpilotnih letalih (UAV) in v oddaljenih napravah. Chloros 1.2.0 ob zagonu zazna vaš model Jetson in optimizira strategijo obdelave za najdeno strojno opremo. **Ročno nastavljanje ni potrebno.**

***

## Podprti modeli Jetson

| Model                | RAM            | Strategija obdelave                                     | Priporočena uporaba                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB skupnega pomnilnika | `GPU_PARALLEL` (2 delavca)                              | Največja zmogljivost, veliki nizi podatkov                      |
| **Jetson Orin NX**   | 8–16 GB skupnega pomnilnika  | `GPU_PARALLEL` (2 delavca, 16 GB) / `GPU_SINGLE` (8 GB)   | Glavno priporočilo za uporabo v zraku in na terenu |
| **Jetson Orin Nano** | 8 GB skupnega pomnilnika     | `GPU_SINGLE` (1 delovni proces, sekvenčno)                     | Osnovna raven računalništva na robu omrežja                                 |

{% hint style="info" %}
Paket Linux arm64 zahteva **JetPack 6**, ki je na voljo za družino Jetson Orin. Starejši modeli (Nano, TX2, Xavier NX) ne morejo izvajati JetPack 6 in jih trenutni paket ne podpira.
{% endhint %}

***

## Zahteve

* **JetPack 6.x** (priporočena najnovejša različica)
* **NVIDIA CUDA** (vključena v JetPack)
* **Plačljiv načrt Chloros+** — stopnja Copper ali višja (potrebna za ves dostop do CLI/SDK; uveljavljena na strani strežnika)

## Namestitev

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.2.0"

# Install Python SDK (optional) — the bundled wheel always matches this build
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl

# Run system diagnostics
chloros-cli selftest
```

Za splošne podrobnosti o namestitvi Linux, lokacijah datotek in odpravljanju težav glejte [Namestitev Linux](linux-installation.md).

{% hint style="info" %}
**Izkopirajte razpakirani imenik na hiter pomnilnik.** Kompilirane izvedljive datoteke se ob vsakem zagonu same razpakirajo v začasni imenik — kar je s pomnilniške kartice SD izredno počasno. Chloros samodejno uporabi `/mnt/ssd/tmp`, če ta obstaja; v nasprotnem primeru nastavite `TMPDIR` na pot na vašem NVMe (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Dinamična prilagoditev računalniške zmogljivosti na Jetsonu

### Kako deluje

Ob zagonu Chloros profilira vaš sistem:

1. **Zazna model Jetson** prek `/proc/device-tree/model`
2. **Prebere razpoložljivi skupni pomnilnik GPU/CPU** (Jetson uporablja enoten pomnilnik)
3. **Izbere strategijo obdelave** (`GPU_PARALLEL`, `GPU_SINGLE` ali `CPU_PARALLEL`)
4. **Samodejno nastavi število delavcev, vrsto poti in dodelitev pomnilnika**Odločitev temelji na**skupnem skupnem pomnilniku RAM**, ne na imenu modela:

* **Manj kot 12 GB skupnega pomnilnika RAM**(vsi Jetsoni z 8 GB): `GPU_SINGLE` z**1 delavcem – namerno zaporedno obdelovanje**. Pomnilnika je premalo za sočasno delovanje delavcev, zato se slike obdelujejo po ena naenkrat. Na napravah Jetson z**8 GB ali manj** nit 3 v celoti preskoči skupino delavcev in svoje delo za posamezno sliko izvaja znotraj procesa.
* **12 GB ali več**(Orin NX 16 GB, AGX Orin): enotni pomnilnik ustreza zahtevam za `GPU_PARALLEL`, vendar je število delavcev**na Jetsonu omejeno na 2** — grafični procesor (GPU), pomnilnik RAM delovnih procesov in njihovi konteksti CUDA za vsakega delavca črpajo iz istega skupnega sklopa, zato več delavcev povečuje tveganje za napake zaradi pomanjkanja pomnilnika.

Avtomatsko izbiro lahko prepisate s spremenljivko okolja `CHLOROS_STRATEGY` — glejte [Dinamično prilagajanje računalniške moči](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Delovanje po posameznih modelih

| Model Jetson                | Strategija       | Delavci | Izvedba                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Zaporedna zanka znotraj procesa (`tiled_gpu` ob pomanjkanju pomnilnika) |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 1       | Zaporedna zanka znotraj procesa                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Vzporedni delovni procesi, pot `fused_gpu`  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2       | Sočasni delovni procesi, pot `fused_gpu`  |

Ključna razlika med platformami je **pomnilnik**. Jetson z 8 GB mora ob visoki obremenitvi slike obdelovati po eno naenkrat z uporabo pomnilniško učinkovitega pristopa s ploščicami, medtem ko lahko Orin z 16 GB in več prek GPU hkrati obdeluje 2 sliki z uporabo združenega potoka z višjo prepustnostjo.

### Proračun GPU na model

Vsak model Jetson ima tudi profil strojne opreme, ki omejuje, koliko prostora iz skupnega sklada lahko porabi obdelava, ter prilagaja velikosti serij:

| Model | Zgornja meja proračuna GPU | Multiplikator velikosti serije | Rezervirano za sistem/zaslon |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70 % | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75 % | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80 % | ×1,5 | 4,0 GB |

Profil se prilagodi glede na zaznani RAM: pri napravi Jetson, ki poroča o **16 GB ali več**, se množitelj velikosti serije poveča za ×1,2. Osnovna velikost serije pred uporabo množiteljev je 8 slik.

Za celoten referenčni dokument o prilagajanju računskih zmogljivosti glejte [Dinamično prilagajanje računskih zmogljivosti](../processing-architecture/dynamic-compute-adaptation.md).

***

## Omejitev frekvence GPU za Texture Aware na modelih Nano in Orin Nano

Algoritem Texture Aware za odstranjevanje pikslov izvaja inferenco nevronskega omrežja na GPU, kar lahko pri modelih Jetson z nizko porabo energije (razred 10–15 W) pri polni hitrosti takta GPU sproži **opozorila o prevelikem toku**. Pred obdelavo s Texture Aware na**Jetson Nano ali Orin Nano**Chloros preveri najvišjo frekvenco GPU-ja in jo omeji na**510 MHz** (510000000), če je trenutno višja:

* Če lahko CLI zapiše v sysfs vozlišče frekvence grafičnega procesorja, se omejitev **uporabi samodejno** in prikaže se potrditev.
* Če ne (potrebuje root), CLI izpiše natančen ukaz `sudo` za ročno uporabo omejitve, počaka trenutek, da ga lahko preberete, nato pa nadaljuje — obdelava še vedno poteka, vendar se lahko prikažejo opozorila o prevelikem toku.

Če želite omejitev nastaviti sami pred obdelavo:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Modeli z večjo močjo (Orin NX 25 W, AGX Orin 60 W) delujejo pri polni hitrosti grafičnega procesorja; omejitev ni nastavljena. Standardni debayer na nobenem modelu nikoli ne sproži omejitve.

{% hint style="info" %}
**Texture Aware na Jetsonu vedno obdeluje eno sliko naenkrat.** Vsak delavec bi potreboval svoj CUDA-kontekst (~1 GB) ter svojo kopijo modela za odstranjevanje šuma, česar enotni pomnilnik ne more zagotoviti – zato je na Jetsonu pot »Texture Aware« pritrjena na enega samega delavca, pri čemer je dostop do grafičnega procesorja serializiran. Pričakujte, da bo »Texture Aware« na katerem koli Jetsonu opazno počasnejši od »Standard«.
{% endhint %}

***

## Upravljanje toplote

Naprave Jetson imajo omejeno toplotno rezervo, zlasti v zaprtih prostorih ali pri uporabi v zraku. Chloros spremlja temperaturo SoC-ja in samodejno omejuje velikost serij:

| Temperatura         | Ukrep                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normalno delovanje — polna hitrost obdelave          |
| **70 °C** (Opozorilo) | Velikost serije se postopoma zmanjšuje (100 % → 50 % med 70 °C in 80 °C) |
| **80 °C** (kritično) | Agresivno omejevanje (50 % → 0 % med 80 °C in 90 °C) |
| **90 °C** (izklop) | Popolna zaustavitev obdelave GPU — potrebno hlajenje |

{% hint style="warning" %}
**Zagotovite ustrezno prezračevanje in odvod toplote** za neprekinjeno delovanje, zlasti v zaprtih terenskih ohišjih ali letalskih sistemih. Toplotno omejevanje bo zmanjšalo zmogljivost obdelave, da se zaščiti strojna oprema.
{% endhint %}

***

## Upravljanje pomnilnika

Naprave Jetson uporabljajo **enotni pomnilnik** — grafični procesor (GPU) in centralni procesor (CPU) si delita isti fizični pomnilnik RAM. Navedeni VRAM (npr. ~15,3 GB na napravi Orin NX 16 GB) ni namenski pomnilnik grafičnega procesorja; gre za isti pomnilnik RAM, ki ga uporabljata operacijski sistem in vsi drugi procesi.

### Opozorilo glede izmenjave pomnilnika in priporočila

Pred obdelavo na napravi Jetson program CLI prešteje RAW-slike v vaši vhodni mapi (`.tif`, `.tiff`, `.raw`, `.dng` — predogledi v formatu JPG se ne štejejo), oceni največjo količino pomnilnika, ki jo bo obdelava potrebovala, in **opozori pred zagonom**, če je verjetno, da RAM in zamenjalni pomnilnik ne bosta zadostovala. Opozorilo ima naslov `LOW MEMORY WARNING - Jetson Detected`, prikaže število slik, RAM, trenutni swap in ocenjeno največjo porabo, nato pa poda natančne ukaze `fallocate` / `chmod` / `mkswap` / `swapon`, prilagojene velikosti vašega projekta (nikoli manjše od 8 GB). Program se za nekaj sekund ustavi, da se sporočilo ne izgubi v pomiku nazaj, nato pa se obdelava nadaljuje.**Ocene pomnilnika, ki jih uporablja opozorilo:**

| Način odstranjevanja pik | Osnova | Na sliko |
| --- | --- | --- |
| Standardni | ~1,5 GB | ~10 MB |
| Texture Aware | ~2,5 GB (model + izvedbeni čas Python) | ~15 MB |

Opozorilo se sproži, ko ocenjena največja poraba presega RAM + zamenjevalni prostor minus 1 GB varnostne rezerve, pri čemer se upošteva le zamenjevalni prostor, **podprt z datotekami** – nastavitev, ki uporablja izključno zram, bo še vedno označena.

Če želite zamenjavo dodati ročno (primer: 8 GB):



<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->

```bash
# Check current memory and swap
free -h

# Create a swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```### Obravnavanje OOM (Out of Memory)

Med obdelavo Chloros spremlja pomnilnik grafičnega procesorja in namesto da se sesuje, gladko zmanjša zmogljivost:

1. Ko izkoriščenost pomnilnika GPU preseže **85 %**, se velikosti serij preventivno zmanjšajo
2. Če se kljub temu pojavi dogodek pomanjkanja pomnilnika, se velikost serije **prepolovi** in se ob vsakem naslednjem OOM ponovno prepolovi; vsaka naslednja uspešna serija to kazen zmanjša za eno stopnjo
3. Pri trajnem obremenjevanju se potek obdelave preusmeri s poti `fused_gpu` na pot `tiled_gpu`, ki je bolj učinkovita z vidika porabe pomnilnika, v skrajnem primeru pa na obdelavo s CPU

***

## Uporaba v praksi

### Upoštevanje porabe energije

| Model Jetson     | Tipična poraba energije | Opombe                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7–15 W              | DC-vtičnica          |
| Jetson Orin NX   | 10–25 W             | DC-vtičnica          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD ali DC-vtičnica |

Načrtujte porabo energije za neprekinjeno obdelavo — največja poraba energije se pojavi med nalogo Thread 3 (Processing), ki intenzivno obremenjuje grafični procesor (GPU).

### Priporočila glede shranjevanja

* **NVMe SSD** se močno priporoča za namestitve arm64
* SD-kartice so prepočasne za obdelavo — uporabljajte jih le kot zagonski nosilec
* Za obdelane izhodne podatke predvidite 2–3-kratno velikost surovih slikovnih podatkov

### Delovanje brez zaslona prek SSH

Chloros CLI je idealen za namestitve Jetson brez zaslona:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Vedno vklopljen backend za sinhronizacijo časa LATTICE / DAQ-E

Če vaš Jetson brez grafičnega vmesnika krmili kamere LATTICE ali svetlobne senzorje DAQ-E, omogočite storitev systemd v ozadju, da bo glavni PTP-strežnik deloval neprekinjeno (enota je nameščena, vendar privzeto ni omogočena):

```bash
sudo systemctl enable --now chloros-backend.service
chloros-cli time-sync status
```

Za podrobnosti, vključno s tem, kako paket omogoča vezavo PTP-vrat 319/320 brez pravic root, glejte [Linux Namestitev](linux-installation.md#always-on-ptp-for-headless-hosts).

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

`chloros-cli process` se zaključi z vrednostjo, ki ni nič, kadar izvedba, ki je zahtevala izdelke, ne zapiše nobenih slik, zato je stanje napake v systemd pomembno za spremljanje.

Za načrtovano obdelavo jo povežite s časovnikom systemd:

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

## Primeri delovnih tokov

### Osnovna obdelava na Jetsonu

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format "TIFF (32-bit, Percent)" \
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
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Priporočeni sistemi Jetson za uporabo na terenu

Za uporabo na terenu in v zraku upoštevajte naslednje možnosti nosilnih plošč Jetson Orin NX 16 GB:

* **V zraku/droni**: sistemi z odpornostjo proti vibracijam (MIL-STD), lahki (manj kot 300 g), pasivno hlajenje
* **Robustna uporaba na terenu**: vodotesna ohišja s stopnjo zaščite IP67/IP69K in povezljivostjo kamere PoE GigE
* **Minimalna/proračunska rešitev**: razvojni kompleti z dodatnimi ohišji

Obrnite se na [MAPIR podporo](https://www.mapir.camera/community/contact) za konkretna priporočila glede strojne opreme za vaš scenarij namestitve.

***

## Naslednji koraki

* [Linux Namestitev](linux-installation.md) — Splošni podatki o namestitvi Linux
* [Dinamična prilagoditev računalniške zmogljivosti](../processing-architecture/dynamic-compute-adaptation.md) — Celoten pregled strategij računalniške zmogljivosti
* [Obdelovalni potek](../processing-architecture/processing-pipeline.md) — Razumevanje 4-nitnega poteka
* [CLI : Ukazna vrstica](../CLI.md) — Priročnik za CLI
* [API : Python SDK](../api-python-sdk.md) — Priročnik za SDK
* [CLI Referenca](../reference/cli-reference.md) in [SDK Referenca](../reference/sdk-reference.md) — Izčrpen seznam ukazov/API za različico 1.2.0

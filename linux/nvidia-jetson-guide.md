# Priročnik za NVIDIA Jetson Program

Chloros
na platformi NVIDIA Jetson omogoča obdelavo multispektralnih slik na robu omrežja – na terenu, na brezpilotnih letalih (UAV) in v oddaljenih napravah. Chloros
1.3.0 ob zagonu prepozna vaš model Jetson in optimizira strategijo obdelave glede na ugotovljeno strojno opremo. **Ročno nastavljanje ni potrebno.**
{% hint style="info" %}
**Paket Jetson 1.3.0 obdeluje podatke na grafičnem procesorju (GPU) naprave Jetson.** `chloros_1.3.0_arm64_jp6.deb` je zasnovan za JetPack 6 in uporablja CUDA-različico PyTorch iz JetPack 6, zato obdelava, vključno z algoritmom Texture Aware debayer, poteka na grafičnem procesorju (GPU) naprave Orin. Paket sam ne vsebuje CUDA: uporablja knjižnici CUDA in cuDNN, ki ju namesti JetPack 6 (`sudo apt install nvidia-jetpack` na sliki, ki ju nima). Kadar ni na voljo grafičnega procesorja s podporo CUDA, obdelava poteka na procesorju, Texture Aware pa uporablja standardni debayer.
{% endhint %}

***

## Podprti modeli Jetson

| Model                | RAM            | Strategija obdelave                                     | Priporočena uporaba                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB skupnega pomnilnika | `GPU_PARALLEL` (2 delavca)                              | Največja zmogljivost, veliki nizi podatkov                      |
| **Jetson Orin NX**   | 8–16 GB skupnega pomnilnika  | `GPU_PARALLEL` (2 delavca, 16 GB) / `GPU_SINGLE` (8 GB)   | Glavno priporočilo za uporabo na terenu              |
| **Jetson Orin Nano** | 8 GB skupnega pomnilnika     | `GPU_SINGLE` (1 delovni proces, zaporedno)                     | Osnovna raven računalništva na robu omrežja                                 |

{% hint style="info" %}
Paket Linux
arm64 zahteva **JetPack 6**, ki je na voljo za družino Jetson Orin. Starejši modeli (Nano, TX2, Xavier NX) ne morejo izvajati JetPack 6 in jih trenutni paket ne podpira.
{% endhint %}

***

## Zahteve

* **JetPack 6.x** (priporočena najnovejša različica)
* **NVIDIA CUDA in cuDNN** iz JetPack 6 (`sudo apt install nvidia-jetpack`, če jih vaša slika ne vsebuje)
* **Plačljiv načrt Chloros
+** — stopnja Copper ali višja (potrebna za ves dostop do CLI
/SDK
; uveljavljena na strani strežnika)

## Namestitev

```bash
# Install the arm64 .deb package (JetPack 6) -- apt, not dpkg -i, so the
# Python SDK's requirements (python3-numpy, python3-requests) come too
sudo apt install ./chloros_1.3.0_arm64_jp6.deb

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.3.0"
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # the .deb installs the SDK

# Run system diagnostics
chloros-cli selftest
```

`.deb` namesti Python
ov SDK
za sistem `python3`, ki ustreza tej različici. Le v virtualno okolje (virtualenv) je treba namestiti priloženi wheel: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` (popolnoma brez povezave). Za splošne podrobnosti o namestitvi v sistemu Linux
, lokacijah datotek in odpravljanju težav glejte [Namestitev v sistemu Linux
](linux-installation.md).

{% hint style="info" %}
**Pri zagonu ni več razpakiranja.** Od različice 1.3.0 naprej se backend in CLI
namestita nerazpakirana v mapo `/usr/lib/chloros/`, zato se ob vsakem zagonu ne razpakirata več v začasno mapo. Začasne datoteke se še vedno samodejno shranjujejo v `/mnt/ssd/tmp`, če ta mapa obstaja; v nasprotnem primeru lahko `TMPDIR` nastavite na pot na vašem NVMe (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Dinamična prilagoditev računalniške zmogljivosti na Jetsonu

### Kako deluje

Ob zagonu Chloros
ustvari profil vašega sistema:

1. **Zazna model Jetson** prek `/proc/device-tree/model`
2. **Prebere razpoložljivi skupni pomnilnik GPU/CPU** (Jetson uporablja enoten pomnilnik)
3. **Vpraša PyTorch, ali lahko uporabi CUDA** — če sta nameščena CUDA in cuDNN iz JetPack 6, lahko
4. **Samodejno izbere strategijo obdelave** (`GPU_PARALLEL`, `GPU_SINGLE` ali `CPU_PARALLEL`) in število delavcev

Odločitev temelji na **skupnem deljenem pomnilniku RAM**, ne na imenu modela:

* **Pri skupnem pomnilniku RAM manj kot 12 GB**(vsi Jetsoni z 8 GB):**1 delavec — namerno zaporedna obdelava**. Pomnilnika je premalo za sočasno delovanje več delavcev, zato se slike obdelujejo po ena naenkrat. Na napravah Jetson z**8 GB ali manj** niti 3 v celoti preskoči skupino delavcev in svoje delo za posamezno sliko izvaja znotraj procesa. V dnevniku strežnika je ta strategija poimenovana `GPU_SINGLE`.
* **12 GB ali več**(Orin NX 16 GB, AGX Orin): združen pomnilnik izpolnjuje pogoje za `GPU_PARALLEL`, vendar je število delavcev**na Jetsonu omejeno na 2** — grafični procesor (GPU), pomnilnik RAM delovnih procesov in njihovi konteksti CUDA za vsakega delavca črpajo iz istega skupnega sklopa, zato več delavcev povečuje tveganje za napake zaradi pomanjkanja pomnilnika.
* **Ni uporabnega CUDA** (manjkajo CUDA-knjižnice JetPacka): `CPU_PARALLEL` pri 12 GB ali več, spodaj po ena slika naenkrat.

Avtomatsko izbiro lahko prepisate s spremenljivko okolja `CHLOROS_STRATEGY` — glejte [Dinamično prilagajanje računalniške moči](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Obnašanje po posameznih modelih

| Model Jetson                | Strategija       | Delavci | Izvedba                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Zaporedna zanka znotraj procesa (`tiled_gpu` ob pomanjkanju pomnilnika) |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 1       | Zaporedna zanka znotraj procesa                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Vzporedni delovni procesi, pot `fused_gpu`  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2       | Sočasni delovni procesi, pot `fused_gpu`  |

Ključna razlika med platformami je **pomnilnik**. Jetson z 8 GB mora ob visoki obremenitvi slike obdelovati po eno naenkrat z uporabo pomnilniško učinkovitega pristopa s ploščicami, medtem ko lahko Orin z 16 GB ali več hkrati obdeluje 2 sliki prek grafičnega procesorja z uporabo združenega potoka z višjo prepustnostjo.

### Proračun GPU na model

Vsak model Jetson ima tudi profil strojne opreme, ki omejuje, koliko prostora iz skupnega sklada lahko porabi obdelava, ter prilagaja velikost serij:

| Model | Zgornja meja proračuna GPU | Multiplikator velikosti serije | Rezervirano za sistem/zaslon |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70 % | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75 % | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80 % | ×1,5 | 4,0 GB |

Profil se prilagodi glede na zaznan RAM: pri napravi Jetson, ki poroča o **16 GB ali več**, se množitelj velikosti serije poveča za ×1,2. Osnovna velikost serije pred uporabo množiteljev je 8 slik.

Za celoten referenčni dokument o prilagajanju računske zmogljivosti glejte [Dinamično prilagajanje računske zmogljivosti](../processing-architecture/dynamic-compute-adaptation.md).

***

## Omejitev frekvence GPU za Texture Aware na modelih Nano in Orin Nano

Algoritem Texture Aware za odstranjevanje pikslov izvaja sklepanje nevronskih mrež na GPU, kar lahko pri modelih Jetson z nizko porabo energije (razred 10–15 W) pri polni hitrosti takta GPU sproži **opozorila o prevelikem toku**. Ko v**Jetson Nano ali Orin Nano**, CLI
preveri najvišjo frekvenco GPU-ja in jo omeji na **510 MHz** (510000000), če je trenutno višja:

* Če lahko CLI
zapiše v sysfs vozlišče frekvence grafičnega procesorja, se omejitev **uporabi samodejno** in prikaže se potrditev.
* Če ne (potrebuje pravice root), CLI
izpiše natančen ukaz `sudo` za ročno uporabo omejitve, počaka trenutek, da ga lahko preberete, nato pa nadaljuje — obdelava še vedno poteka, vendar se lahko prikažejo opozorila o prevelikem toku.

Če želite omejitev nastaviti sami pred obdelavo:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Modeli z večjo močjo (Orin NX 25 W, AGX Orin 60 W) delujejo pri polni hitrosti grafičnega procesorja; omejitev se ne uporablja. Standardni debayer na nobenem modelu nikoli ne sproži omejitve.

{% hint style="info" %}
**Texture Aware na Jetsonu vedno obdeluje eno sliko naenkrat.** Vsak delavec bi potreboval svoj CUDA-kontekst (~1 GB) ter svojo kopijo modela za odstranjevanje šuma, česar enotni pomnilnik ne more zagotoviti – zato je na Jetsonu pot Texture Aware vezana na enega samega delavca, pri čemer je dostop do grafičnega procesorja (GPU) serializiran. Pričakujte, da bo Texture Aware na katerem koli Jetsonu znatno počasnejši od Standard. Texture Aware se uporablja za slike Survey3
: zajemi LATTICE vedno uporabljajo Standardni debayer. Brez uporabnega CUDA GPU-ja se izvede Standardni način, backend pa zabeleži eno vrstico `[DEBAYER] … needs a CUDA GPU …`, ki pojasni razlog (prikazano z `chloros-cli --verbose`).
{% endhint %}

***

## Upravljanje toplote

Naprave Jetson imajo omejeno toplotno rezervo, zlasti v zaprtih prostorih ali pri uporabi v zraku. Program Chloros
spremlja temperaturo SoC-ja in samodejno omejuje velikost serij:

| Temperatura         | Ukrep                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normalno delovanje — polna hitrost obdelave          |
| **70 °C** (Opozorilo)  | Velikost serije se postopoma zmanjšuje (100 % → 50 % med 70 °C in 80 °C) |
| **80 °C** (kritično) | Agresivno omejevanje (50 % → 0 % med 80 °C in 90 °C) |
| **90 °C** (izklop) | Popolna zaustavitev obdelave GPU — potrebno hlajenje |

{% hint style="warning" %}
**Zagotovite ustrezno prezračevanje in odvod toplote** za neprekinjeno delovanje, zlasti v zaprtih terenskih ohišjih ali letalskih sistemih. Toplotno omejevanje bo zmanjšalo zmogljivost obdelave, da se zaščiti strojna oprema.
{% endhint %}

***

## Upravljanje pomnilnika

Naprave Jetson uporabljajo **enotni pomnilnik** — grafična kartica (GPU) in procesor (CPU) si delita isti fizični pomnilnik RAM. Navedeni VRAM (npr. ~15,3 GB na Orin NX 16 GB) ni namenski pomnilnik grafične kartice; gre za isti pomnilnik RAM, ki ga uporabljajo operacijski sistem in vsi drugi procesi.

### Opozorilo in priporočila glede zamenjave

Pred obdelavo na napravi Jetson CLI
prešteje RAW-slike v vaši vhodni mapi (`.tif`, `.tiff`, `.raw`, `.dng` — predogledi v formatu JPG se ne štejejo), oceni največjo količino pomnilnika, ki jo bo izvajanje potrebovalo, in **opozori pred začetkom**, če je verjetno, da RAM + swap ne bosta zadostovala. Opozorilo ima naslov `LOW MEMORY WARNING - Jetson Detected`, izpiše število vaših slik, RAM, trenutni swap in ocenjeno največjo porabo, nato pa poda natančne ukaze `fallocate` / `chmod` / `mkswap` / `swapon`, prilagojene velikosti vaš projekt (nikoli manjši od 8 GB). Za nekaj sekund se ustavi, da se sporočilo ne izgubi v zgodovini pomikanja, nato pa se obdelava nadaljuje.**Ocene pomnilnika, ki jih uporablja opozorilo:**

| Način Debayer | Osnovno | Na sliko |
| --- | --- | --- |
| Standardni | ~1,5 GB | ~10 MB |
| Texture Aware | ~2,5 GB (model + Pythonovo izvajališče) | ~15 MB |

Opozorilo se sproži, ko ocenjena največja poraba presega RAM + zamenjavo minus 1 GB varnostne rezerve, pri čemer se upošteva le zamenjava, **podprta s datotekami** — nastavitev, ki uporablja izključno zram, bo še vedno označena.

Za ročno dodajanje izmenjalnega prostora (primer: 8 GB):

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
```

<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->


### Obravnavanje OOM (Out of Memory)

Med obdelavo program Chloros
spremlja pomnilnik grafičnega procesorja (GPU) in namesto da bi se sesul, postopoma zmanjša zmogljivost:

1. Ko izkoriščenost pomnilnika GPU preseže **85 %**, se velikosti serij preventivno zmanjšajo
2. Če se kljub temu pojavi dogodek pomanjkanja pomnilnika, se velikost serije **prepolovi**, pri vsakem naslednjem OOM pa se ponovno prepolovi; vsaka naslednja uspešna serija to kazen zniža za eno stopnjo
3. Pri trajnem pritisku se potek obdelave preusmeri s poti `fused_gpu` na pot `tiled_gpu`, ki je bolj učinkovita z vidika porabe pomnilnika, v skrajnem primeru pa na obdelavo v CPU

Ker si GPU deli RAM-a naprave Jetson, RAM skupaj z izmenjevalnim prostorom še vedno omejuje izvajanje – zgornje opozorilo o izmenjevalnem prostoru je preverjanje, na katerega je treba paziti.

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
* SD-kartice so prepočasne za obdelavo — uporabljajte jih le kot zagonski medij
* Za obdelane izhodne podatke predvidite 2–3-kratno velikost surovih slikovnih podatkov

### Delovanje brez grafičnega vmesnika prek SSH


Chloros
CLI
je idealen za namestitve Jetsona brez grafičnega vmesnika:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Vedno vklopljen backend za sinhronizacijo časa LATTICE / DAQ-E

Če vaš Jetson brez zaslona upravlja kamere LATTICE ali svetlobne senzorje DAQ-E, omogočite storitev systemd v ozadju **kot vaš uporabnik**, da bo PTP grandmaster deloval neprekinjeno in da bodo skripti SDK
našli delujoče ozadje (enota je nameščena, vendar privzeto ni omogočena):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
chloros-cli time-sync status
```

Backend nato deluje kot vi: posnetki, ki jih zapiše, so vaši, uporablja pa vaš `chloros-cli login`. Za podrobnosti, vključno z uporabniškim imenom root `chloros-backend.service` in načinom, kako paket omogoča vezavo PTP-vrat 319/320 brez pravic root, glejte [Namestitev v sistemu Linux
](linux-installation.md#always-on-ptp-for-headless-hosts).

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

Za načrtovano obdelavo ga združite s časovnikom systemd:

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

### Osnovna obdelava na Jetsonu

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI GNDVI OSAVI
```

### Python
SDK
na Jetsonu

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "GNDVI", "OSAVI"],
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
        --indices NDVI OSAVI
    echo "Completed $name"
done
```

***

## Priporočeni sistemi Jetson za uporabo na terenu

Za uporabo na terenu in v zraku upoštevajte naslednje možnosti nosilnih plošč Jetson Orin NX 16 GB:

* **V zraku/droni**: Sistemi z odpornostjo proti vibracijam (MIL-STD), lahki (manj kot 300 g), s pasivnim hlajenjem
* **Zahtevne terenske razmere**: vodotesna ohišja s stopnjo zaščite IP67/IP69K in povezljivostjo s kamero PoE GigE
* **Minimalna/proračunska rešitev**: razvojni kompleti z dodatnimi ohišji

Za konkretna priporočila glede strojne opreme za vaš scenarij uporabe se obrnite na [podporo MAPIR
](https://www.mapir.camera/community/contact).

***

## Naslednji koraki

* [Namestitev Linux
a](linux-installation.md) — Splošni podatki o namestitvi Linux
a
* [Dinamična prilagoditev računalniške zmogljivosti](../processing-architecture/dynamic-compute-adaptation.md) — Celoten pregled strategij računalniške zmogljivosti
* [Obdelovalni potek](../processing-architecture/processing-pipeline.md) — Razumevanje 4-nitnega poteka
* [CLI: ukazna vrstica](../CLI.md) — Priročnik za CLI

* [API
: Python
SDK
](../api-python-sdk.md) — Priročnik za SDK

* [Referenca CLI
](../reference/cli-reference.md) in [Referenca SDK
](../reference/sdk-reference.md) — Izčrpni seznami ukazov/API
za različico 1.3.0

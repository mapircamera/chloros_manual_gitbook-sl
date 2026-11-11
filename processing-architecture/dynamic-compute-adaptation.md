# Dinamično prilagajanje računalniške zmogljivosti

Chloros
1.3.0 uporablja zaznavanje strojne opreme in samodejno izbiro strategije obdelave. Obdelovalni motor se prilagaja vaši strojni opremi – od Jetson Orin Nano do delovne postaje z več grafičnimi procesorji – brez kakršne koli ročne konfiguracije.

***

## Kako deluje

Ko se Chloros
zažene, naredi profil vašega sistema:

1. **Zazna operacijski sistem** – Windows
ali Linux
2. **Prepozna jedra procesorja in skupno količino pomnilnika RAM**

3.**Zazna prisotnost grafičnega procesorja** – zmogljivost NVIDIA CUDA, VRAM, model
4. **Prepozna model Jetson** (če je to ustrezno) – prek `/proc/device-tree/model`
5. **Preveri temperaturne senzorje** (Jetson) – za obdelavo, ki upošteva temperaturo
6. **Izbere strategijo računanja** – na podlagi vse zaznane strojne opreme
7. **Samodejno konfigurira število delavcev, vrsto poteka in dodelitev pomnilnika**

Zaznan profil se za tekoče sejo shrani v predpomnilnik v pomnilniku in na disku, tako da se poznejša izvajanja začnejo hitreje:

| Platforma | Profil v predpomnilniku |
| --- | --- |
| **Linux
/ Jetson** | `~/.config/chloros/system_config.json` (upošteva `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Izbrišite to datoteko, da prisilite novo zaznavanje — to je koristno po dodajanju grafičnega procesorja (GPU) ali več pomnilnika RAM. Chloros
samodejno ponovno zazna tudi takrat, ko je predpomnilnik zapisala nezdružljiva starejša različica.

***

## Strategije izračuna

Chloros
izbere eno od treh strategij izračuna glede na vašo strojno opremo:

| Strategija | Izbrana, ko | Delavci | Izvajalec | Potek |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| CUDA grafična kartica z**12 GB+ VRAM**(na združenem pomnilniku Jetson, zahteva tudi skupaj 12 GB+ deljenega RAM-a) | `min(4, ⌊(VRAM − 0.5GB) × 0.9 ÷ 3.5GB⌋)`, najmanj 2 —**omejeno na 2 na Jetsonu** | Delovni procesi (ločeni konteksti CUDA) | `fused_gpu` |
| **`GPU_SINGLE`**| CUDA GPU z**2–12 GB VRAM**| 3 (prekrivanje I/O; dostop do GPU je serializiran s semaforjem).**1 (zaporedno) na napravah Jetson z manj kot 12 GB RAM** | Delovni procesi (ločeni konteksti CUDA); zaporedno znotraj procesa na napravah Jetson z manj RAM | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Brez CUDA GPU ali z manj kot 2 GB VRAM | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Praktični primeri formule za število delavcev `GPU_PARALLEL`: 12 GB VRAM → 2 delavca, 16 GB → 3 delavca, 20 GB+ → 4 delavca. Vsakemu delavcu je dodeljenih 3,5 GB, kar ustreza izmerjeni potrebi naloge Texture Aware, v okviru 90 % preostalega VRAM-a po odbitku 0,5 GB rezerve.

{% hint style="info" %}
**Strategije za GPU zahtevajo PyTorch, ki podpira CUDA.** Naprava, katere PyTorch nima uporabne CUDA-naprave, dobi `CPU_PARALLEL`, ne glede na njen GPU. Paket Jetson 1.3.0 uporablja CUDA-različico PyTorcha iz JetPack 6, zato je Jetson primeren, ko so nameščene CUDA- in cuDNN-knjižnice iz JetPacka. Jetson z manj kot 12 GB RAM-a vedno obdeluje eno sliko naenkrat, med obdelavo (zabeleženo kot `GPU_SINGLE`).
{% endhint %}

Strategije GPU izvajajo skupino delovnih procesov, od katerih je vsak ločen proces operacijskega sistema z lastnim CUDA-kontekstom, ki jih Chloros
zažene s ponovnim zagonom lastnega nameščenega backend programa (ne prek Python
ovega `multiprocessing`, katerega zagon delovnih procesov ne deluje znotraj pakirane aplikacije). Strategija CPU uporablja nabor niti (Python
ov `ThreadPoolExecutor`). Chloros
ne uporablja nobenega distribuiranega okvira tretjih ponudnikov (kot je Ray).

### Vrste poti obdelave

* **`fused_gpu`** — Pot obdelave, ki v celoti poteka na GPU. Operacije debayeringa, korekcije in indeksiranja potekajo na GPU v enem samem združenem prehodu. Največja prepustnost, zahteva največ VRAM-a.
* **`tiled_gpu`** — Pot na GPU, ki učinkovito izkorišča pomnilnik. Slike obdeluje v ploščicah, da se prilagodijo omejenemu pomnilniku GPU. Nižja prepustnost, vendar deluje na napravah z omejenim pomnilnikom.
* **`cpu_fallback`** — Obdelava izključno na CPU z uporabo večnitne vzporednosti. Uporablja se, kadar ni na voljo grafičnega procesorja NVIDIA, ter kot zadnja možnost, če obe poti na grafičnem procesorju odpovesta.

Veriga nadomestnih rešitev v času izvajanja je vedno `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Ročno prepisovanje strategije

Nastavite spremenljivko okolja `CHLOROS_STRATEGY`, da prisilite uporabo določene strategije – to je strokovna rešitev za primer, ko samodejno zaznavanje izbere nekaj, kar ni primerno za vašo situacijo (na primer, da grafični procesor ostane prost za drugo delo):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

Pri iskanju spremenljivke se ne upošteva velike in male črke; vse, kar ni eno od treh imen, se prezre, samodejno zaznavanje pa poteka normalno. Tudi ob ročnem preklopu Chloros
še vedno sam izbere število delavcev:

| Preklop | Uporabljeno število delavcev |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

Prepis `GPU_SINGLE` ali `GPU_PARALLEL` na računalniku brez uporabne naprave CUDA se zavrne z vrstico `[STRATEGY]` v dnevniku, izvedba pa uporabi `CPU_PARALLEL`.

Priporočljivo je, da to nastavite za posamezne ukaze in ne trajno, da se običajne izvedbe še naprej samodejno prilagajajo.

***

## Obnašanje, specifično za platformo

| Platforma | Strategija | Delavci | Potek | Opombe |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (zaporedno) | Način z učinkovito rabo pomnilnika, po ena slika naenkrat |
| **Jetson Orin NX 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (zaporedno) | Skupni pomnilnik pod 12 GB zahteva zaporedno obdelavo |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (sočasno) | Priporočena robna naprava — omejeno na 2 delavca za Jetson |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (sočasno) | Največja zmogljivost na robu omrežja (tudi pri Jetsonu omejeno na 2 delavca) |
| **Namizni računalnik z 8 GB GPU** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 delavci si prekrivajo vhodno-izhodne operacije, medtem ko semafor serializira dostop do grafičnega procesorja |
| **Namizni računalnik z grafično kartico 12 GB+** | `GPU_PARALLEL` | 2–4 | `fused_gpu` (sočasno) | Optimalna zmogljivost namiznega računalnika: 12 GB → 2 delavca, 16 GB → 3, 20 GB+ → 4 |
| **Sistem, ki uporablja samo CPU** | `CPU_PARALLEL` | fizična jedra − 1 (min. 2) | `cpu_fallback` | GPU ni potreben, uporablja zbirko niti |

{% hint style="info" %}
**Enotni pomnilnik Jetson**: Naprave Jetson si delijo pomnilnik grafičnega procesorja in procesorja. Naprava Jetson Orin NX s 16 GB poroča o ~15,3 GB VRAM-a, vendar gre za isti fizični RAM, ki ga uporabljajo operacijski sistem in procesi procesorja. Zato so naprave Jetson z 16 GB in več primerne za `GPU_PARALLEL`, podobno kot namizni grafični procesorji z 12 GB in več, vendar so omejene na 2 delavca – grafični procesor, delavni procesi in njihovi konteksti CUDA za vsakega delavca črpajo iz istega skupnega sklopa.
{% endhint %}

### Proračun grafičnega procesorja glede na VRAM (diskretni grafični procesorji)

Na gostiteljih x86_64 z diskretnim grafičnim procesorjem NVIDIA zaznan VRAM določa tudi, koliko zmogljivosti kartice se lahko zahteva in kako velike lahko postanejo serije:

| Zaznan VRAM | Zgornja meja proračuna za grafično kartico | Multiplikator velikosti serije |
| --- | --- | --- |
| **8 GB+** | 90 % | ×2,0 |
| **6–8 GB** | 85 % | ×1,75 |
| **3,5–6 GB** | 80 % | ×1,5 |
| **2–3,5 GB** | 75 % | ×1,25 |
| **Manj kot 2 GB** | 70 % | ×1,0 |

Diskretne grafične kartice za sistem rezervirajo le 0,5 GB, saj ne delijo sistemskega pomnilnika RAM. Profili Jetson rezervirajo veliko več in imajo nižjo zgornjo mejo — glej [NVIDIA Jetson Guide](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Dinamična dodelitev pomnilnika grafične kartice

Chloros
uporablja [4-nitni procesni potek](processing-pipeline.md):

* **Nit 1** (Zaznavanje) — nalaganje slike, razčlenjevanje EXIF, zaznavanje cilja
* **Nit 2** (Kalibracija) — Izračun kalibracije odbojnosti
* **Nit 3** (Obdelava) — GPU debayer, popravljanje vinjetiranja, izračun indeksa
* **Nit 4** (Izvoz) — Pisanje datotek, vgrajevanje metapodatkov

Niti 1, 2 in 4 porabljajo malo zmogljivosti grafičnega procesorja; Nit 3 pa je tista, ki porabi največ. Ko se prejšnje niti v poti zaključijo, se njihov proračun za GPU **porazdeli med preostale aktivne niti**, tako da nit 3 s potekom izvajanja postopoma dobiva več pomnilnika.

### Faze dodeljevanja

| Faza | Aktivne niti | Porazdelitev pomnilnika GPU |
| --- | --- | --- |
| **Zgodnja** | 1, 2, 3, 4 | Razdeljen med vse niti, večina pa gre niti 3 |
| **Srednje-zgodnja** | 2, 3, 4 | Delež niti 1 se prerazporedi |
| **Srednje-pozno** | 3, 4 | Deleži niti 1 in 2 se dodelijo nitema 3 in 4 |
| **Pozno** | 3 ali 4 | Zadnja aktivna nit dobi največjo dodelitev |

Številke urejata dve pravili:

* Niti, ki je **edina** aktivna, se dodeli največja dodelitev iz njenega profila.
* Ko je aktivnih več kot ena *zahtevna* naloga GPU-ja, se osnovna dodelitev vsake zahtevne naloge razdeli mednje (nikoli pod nastavljenim minimumom).

Vrednost, ki se dejansko uporabi med izvajanjem, je **nižja** od dodelitve profila platforme in trenutnega priporočila iz monitorja pomnilnika GPU, zato ima obremenjena kartica vedno prednost pred optimističnim profilom.***

## Obdelava, ki upošteva teksturo

Debayer »Texture Aware« (**samo Chloros
+** — `--debayer texture-aware`) izvaja model za odstranjevanje šuma z umetno inteligenco/strojnim učenjem, ki potrebuje približno 1,75 GB VRAM v FP16 na kopijo, zato porabi veliko več pomnilnika GPU kot standardna metoda:

* Sistemi z **manj kot 7 GB VRAM**obdelujejo Texture Aware v**sinhronem ciklu, po eno sliko naenkrat** — več kopij modela se ne more zmestiti, delavniški bazen pa bi le povečal tekmovanje za dostop
* Sistemi z **7 GB ali več VRAM** lahko Texture Aware obdelujejo sočasno, čeprav z manj delavci v primerjavi s Standardno metodo
* Na **Jetsonu** je Texture Aware vedno dodeljen enemu delavcu
* Če **ni uporabnega CUDA grafičnega procesorja**, se izvede standardni debayer, backend pa zabeleži eno vrstico `[DEBAYER] … needs a CUDA GPU …`, ki pojasni razlog
* Na Jetsonih z nizko porabo energije (Nano, Orin Nano) CLI
, ko zahtevate »Texture Aware«, omeji frekvenco grafičnega procesorja – glejte [NVIDIA Jetson Guide](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)

***

## Upravljanje toplote (Jetson)

Naprave Jetson imajo toplotne omejitve, zlasti pri namestitvah v zaprtih prostorih ali v zraku. Chloros
spremlja vgrajene temperaturne senzorje naprave Jetson in samodejno prilagaja velikost serij:

| Temperatura | Odziv |
| --- | --- |
| **&lt; 70 °C** | Normalno delovanje – polna hitrost |
| **70 °C** (Opozorilo) | Velikost serije se postopoma zmanjšuje (100 % → 50 % med 70 °C in 80 °C) |
| **80 °C** (kritično) | Agresivno omejevanje zmogljivosti (50 % → 0 % med 80 °C in 90 °C) |
| **90 °C** (izklop) | Popolna zaustavitev obdelave GPU |

Na namiznih sistemih z ustreznim hlajenjem se toplotno omejevanje redko sproži.

***

## Obravnavanje pritiska na pomnilnik

Program Chloros
med obdelavo neprekinjeno spremlja pomnilnik grafičnega procesorja in reagira na treh ravneh.

**Določanje velikosti serije.** Serija se začne z 8 slikami, pomnoženimi z multiplikatorjem platforme iz zgornjih tabel. Chloros
nato preveri prosti VRAM, 20 % tega rezervira za lastne potrebe PyTorcha in predpostavlja približno 100 MB pomnilnika grafičnega procesorja na sliko z ločljivostjo 12 MP – serija je tista, ki je manjša, bodisi omejitev, izpeljana iz pomnilnika, bodisi osnovna vrednost platforme. Nikoli ne pade pod 1.

**Preventivno zmanjšanje.**Pri**izkoriščenosti VRAM nad 85 %** se velikosti serij zmanjšajo, še preden pride do napake.**Zmanjšanje dodelitve na nit.** Ko se trenutna izkoriščenost poveča, se proračun GPU za vsako nit zmanjša: ×0,75 pri izkoriščenosti nad 80 %, ×0,5 pri izkoriščenosti nad 90 %. Pragi monitorja so 70 % (konzervativni), 85 % (normalna meja delovanja) in 95 % (tveganje za pomanjkanje pomnilnika).**Umaknitev in obnovitev ob pomanjkanju pomnilnika.** Če kljub vsemu pride do pomanjkanja pomnilnika:

* se velikost serije **prepolovi** in se ob vsakem zaporednem dogodku pomanjkanja pomnilnika ponovno prepolovi – vsaka naslednja uspešna serija to kazen zmanjša za eno stopnjo
* se dodelitve aktivnim nitim zmanjšajo na 70 % njihove trenutne vrednosti, dodeljevalnik pa preklopi na konzervativno strategijo, ki se po nizu uspešnih dodelitev spet sprosti
* pod hudim pritiskom se potek obdelave umakne s `fused_gpu` na `tiled_gpu`, v skrajnem primeru pa na `cpu_fallback`

**Pomnilnik gostitelja (Jetson).** Pred obdelavo CLI
oceni največjo porabo pomnilnika gostitelja na podlagi števila slik in načina debayeringa ter opozori, če je verjetno, da RAM skupaj z izmenjavo, podprto s datotekami, ne bo zadostoval, pri čemer izpiše natančne ukaze za dodajanje izmenjave — glej [NVIDIA Jetson Guide](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).

***

## Spremljanje prilagajanja računalniške zmogljivosti

### Sistemsko diagnosticiranje

`chloros-cli selftest` je najhitrejši način za potrditev tega, kar vidi računalniški sloj:

```bash
chloros-cli selftest
```

Njegovih 7 preverjanj zajema različico, razpoložljivost vrat, zagon backenda, `/api/test`, sistemske informacije, prisotnost modela za odstranjevanje šuma ter pripravljenost CUDA in odstranjevalca šuma. Preverjanje 5 neposredno izpiše vrstico o strojni opremi:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

Preverjanje 7 izpiše `CUDA: <bool>, Denoiser: <bool>` — oba morata biti resnična, da je Texture Aware sploh uporaben.

### Dnevniki backenda

Strategija in število delavcev se izbereta znotraj backenda na začetku vsakega izvajanja — ni nobenega CLI
-pasice, ki bi ju prikazovala. Ko se nekaj obnaša nepričakovano (preklop na GPU pot, OOM, odšumevalnik, ki se ne naloži), se to prikaže v dnevniku backenda za tisto sejo:

| Platforma | Mesto dnevnika |
| --- | --- |
| **Linux
/ Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (ena datoteka na zagon) |
| **Linux
, backend zagnan prek CLI** | prav tako `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Napredek v realnem času

Med izvajanjem CLI
prikazuje napredek v realnem času za vsak niti (Detecting, Analyzing, Processing, Exporting), ki se prenaša prek Server-Sent Events — to je praktičen pokazatelj, ali je nit 3 ozko grlo. Glej [Obdelovalni potek](processing-pipeline.md).

***

## Naslednji koraki

* [Obdelovalni potek](processing-pipeline.md) — Razumevanje arhitekture poteka s 4 niti
* [Vodnik za NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Namestitev in optimizacija, specifična za Jetson
* [CLI
: ukazna vrstica](../CLI.md) — Vodnik po ukazni vrstici
* [CLI
Reference](../reference/cli-reference.md) — Izčrpen seznam ukazov za različico 1.3.0

# Dinamično prilagajanje računalniške zmogljivosti

Chloros 1.2.0 uporablja zaznavanje strojne opreme in samodejno izbiro strategije obdelave. Obdelovalni motor se prilagaja vaši strojni opremi – od Jetson Orin Nano do delovne postaje z več grafičnimi procesorji – brez kakršne koli ročne konfiguracije.

***

## Kako deluje

Ko se Chloros zažene, opravi profiliranje vašega sistema:

1. **Zazna operacijski sistem** — Windows ali Linux
2. **Prepozna jedra procesorja in skupno količino pomnilnika RAM**

3.**Zazna prisotnost grafičnega procesorja** — zmogljivost NVIDIA CUDA, VRAM, model
4. **Prepozna model Jetson** (če je primerno) — prek `/proc/device-tree/model`
5. **Preveri toplotne senzorje** (Jetson) — za obdelavo, ki upošteva temperaturo
6. **Izbere strategijo računanja** — na podlagi vse zaznane strojne opreme
7. **Samodejno konfigurira število delavcev, vrsto poteka in dodelitev pomnilnika**

Zaznani profil se za tekoče sejo shrani v predpomnilnik in na disk, tako da se poznejši zagoni začnejo hitreje:

| Platforma | Shranjeni profil |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (upošteva `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Izbrišite to datoteko, da prisilite novo zaznavanje — to je koristno po dodajanju grafične kartice ali več pomnilnika RAM. Chloros samodejno ponovno zazna tudi takrat, ko je predpomnilnik zapisala nezdružljiva starejša različica.

***

## Strategije računanja

Chloros izbere eno od treh strategij računanja glede na vašo strojno opremo:

| Strategija | Izbrana, ko | Delavci | Izvajalec | Potek |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| CUDA grafična kartica, ki poroča o**12 GB+ VRAM**(na združenem pomnilniku Jetson, zahteva tudi skupno 12 GB+ deljenega pomnilnika RAM) | `min(4, VRAM ÷ 4GB)`, najmanj 2 —**omejeno na 2 na Jetsonu** | `ProcessPoolExecutor` (spawn) | `fused_gpu` |
| **`GPU_SINGLE`**| CUDA GPU z**2–12 GB VRAM**| 3 (prekrivanje I/O; dostop do GPU je serializiran s semaforjem).**1 (zaporedno) na napravah Jetson z manj kot 12 GB RAM** | `ProcessPoolExecutor` (spawn); zaporedno znotraj procesa na Jetsonih z manj RAM-a | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Brez CUDA GPU ali z manj kot 2 GB VRAM | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Praktični primeri formule za število delavcev `GPU_PARALLEL`: 12 GB VRAM → 3 delavci, 16 GB+ → 4 delavci, kateri koli Jetson → 2 delavca.

Vzporednost je implementirana s standardnim `concurrent.futures` iz Python: strategije GPU uporabljajo `ProcessPoolExecutor` z metodo zagona **spawn** (vsak delavec je ločen proces z lastnim CUDA-kontekstom – `fork` bi kopiral že inicializirano CUDA-stanje in poškodoval podrejene procese), strategija za CPU pa uporablja `ThreadPoolExecutor`. Chloros ne uporablja nobenega distribuiranega okvira tretjih ponudnikov (kot je Ray).

### Vrste poti obdelave

* **`fused_gpu`** — Pot obdelave v celoti na GPU. Operacije debayeringa, korekcije in indeksiranja potekajo na GPU v enem samem združenem prehodu. Največja prepustnost, zahteva največ VRAM-a.
* **`tiled_gpu`** — Pot obdelave na GPU, ki je učinkovita z vidika pomnilnika. Slike obdeluje v ploščicah, da se prilagodijo omejenemu pomnilniku GPU. Manjša prepustnost, vendar deluje na napravah z omejenim pomnilnikom.
* **`cpu_fallback`** — Obdelava izključno na CPU z uporabo večnitne vzporednosti. Uporablja se, kadar ni na voljo grafične kartice NVIDIA, ter kot zadnja možnost, če obe poti na grafični kartici odpovesta.

Veriga nadomestnih rešitev v času izvajanja je vedno `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Ročno prepisovanje strategije

Nastavite spremenljivko okolja `CHLOROS_STRATEGY`, da prisilite uporabo določene strategije — to je strokovna rešitev za primer, ko samodejno zaznavanje izbere nekaj, kar ni primerno za vašo situacijo (na primer, da bi grafični procesor ostal prost za drugo delo):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

Pri iskanju spremenljivke se ne upošteva velike in male črke; vse, kar ni eno od treh imen, se prezre, samodejno zaznavanje pa poteka normalno. Tudi ob preklopu Chloros še vedno izbere število delavcev za vas:

| Preklop | Uporabljeno število delavcev |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

Priporočljivo je, da to nastavite za posamezne ukaze in ne trajno, tako da se običajno izvajanje še naprej samodejno prilagaja.

***

## Obnašanje, specifično za posamezne platforme

| Platforma | Strategija | Delavci | Potek | Opombe |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (zaporedno) | Način z učinkovito rabo pomnilnika, po ena slika naenkrat |
| **Jetson Orin NX 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (zaporedno) | Skupni pomnilnik pod 12 GB zahteva zaporedno obdelavo |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (sočasno) | Priporočena robna naprava – omejitev Jetsona na 2 delavca |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (sočasno) | Največja zmogljivost na robu omrežja (tudi pri Jetsonu omejeno na 2 delavca) |
| **Namizni računalnik z 8 GB GPU** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 delavci si prekrivajo vhodno-izhodne operacije, medtem ko semafor serializira dostop do GPU |
| **Namizni računalnik z grafično kartico 12 GB+** | `GPU_PARALLEL` | 3–4 | `fused_gpu` (sočasno) | Optimalna zmogljivost namiznega računalnika: 12 GB → 3 delavci, 16 GB+ → 4 |
| **Sistem, ki uporablja izključno CPU** | `CPU_PARALLEL` | fizična jedra − 1 (min. 2) | `cpu_fallback` | GPU ni potreben, uporablja sklop niti |

{% hint style="info" %}
**Enotni pomnilnik Jetson**: Naprave Jetson si delijo pomnilnik grafičnega procesorja in procesorja. Jetson Orin NX 16 GB poroča o ~15,3 GB VRAM-a, vendar gre za isti fizični RAM, ki ga uporabljajo operacijski sistem in procesi CPU. Zato so Jetsoni z 16 GB+ primerni za `GPU_PARALLEL`, podobno kot namizni GPU z 12 GB+, vendar so omejene na 2 delavca – grafična kartica, delavni procesi in njihovi CUDA-konteksti na delavca črpajo iz istega skupnega skladišča.
{% endhint %}

### Proračun grafične kartice glede na VRAM (diskretne grafične kartice)

Na gostiteljih x86_64 z diskretno grafično kartico NVIDIA zaznan VRAM prav tako določa, koliko zmogljivosti kartice se lahko porabi in kako velike se lahko povečajo serije:

| Zaznan VRAM | Zgornja meja proračuna za grafično kartico | Množitelj velikosti serije |
| --- | --- | --- |
| **8 GB+** | 90 % | ×2,0 |
| **6–8 GB** | 85 % | ×1,75 |
| **3,5–6 GB** | 80 % | ×1,5 |
| **2–3,5 GB** | 75 % | ×1,25 |
| **Manj kot 2 GB** | 70 % | ×1,0 |

Diskretne grafične kartice za sistem rezervirajo le 0,5 GB, saj ne delijo sistemskega pomnilnika RAM. Profili Jetson rezervirajo veliko več in imajo nižjo zgornjo mejo — glejte [NVIDIA Jetson Guide](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Dinamična dodelitev pomnilnika grafičnega procesorja

Chloros uporablja [4-nitni procesni potek](processing-pipeline.md):

* **Nit 1** (Zaznavanje) — nalaganje slike, razčlenjevanje EXIF-podatkov, zaznavanje cilja
* **Nit 2** (Kalibracija) — izračun kalibracije odbojnosti
* **Nit 3** (Obdelava) — GPU debayer, popravljanje vinjetiranja, izračun indeksa
* **Nit 4** (izvoz) — zapisovanje datotek, vgrajevanje metapodatkov

Niti 1, 2 in 4 porabijo malo zmogljivosti grafičnega procesorja; nit 3 pa je tista, ki jo porabi največ. Ko se prejšnji niti v procesni verigi zaključijo, se njihov proračun za GPU **porazdeli med preostale aktivne niti**, tako da nit 3 med izvajanjem postopoma dobi vedno več pomnilnika.

### Faze dodeljevanja

| Faza | Aktivne niti | Porazdelitev pomnilnika GPU |
| --- | --- | --- |
| **Zgodnja** | 1, 2, 3, 4 | Razdeljen med vse niti, večina pa gre niti 3 |
| **Srednje-zgodnja** | 2, 3, 4 | Delež niti 1 se prerazporedi |
| **Srednje-pozno** | 3, 4 | Deleži niti 1 in 2 se dodelita niti 3 in 4 |
| **Pozno** | 3 ali 4 | Zadnja aktivna nit dobi svojo največjo dodelitev |

Številke urejata dve pravili:

* Niti, ki je **edina** aktivna, se dodeli največja dodelitev iz njenega profila.
* Ko je aktivnih več kot ena *zahtevna* naloga GPU-ja, se osnovna dodelitev vsake zahtevne naloge razdeli mednje (nikoli pod nastavljenim minimumom).

Vrednost, ki se dejansko uporabi med izvajanjem, je **nižja** od dodelitve profila platforme in trenutnega priporočila iz monitorja pomnilnika GPU, zato ima obremenjena kartica vedno prednost pred optimističnim profilom.***

## Obdelava, ki upošteva teksturo

Debayer, ki upošteva teksture (**samo Chloros+** — `--debayer texture-aware`) izvaja model za odstranjevanje šuma z umetno inteligenco/strojnim učenjem, ki potrebuje približno 1,75 GB VRAM v FP16 na kopijo, zato porabi veliko več pomnilnika GPU kot standardna metoda:

* Sistemi z **manj kot 7 GB VRAM**obdelujejo Texture Aware v**sinhronem ciklu, po eno sliko naenkrat** — več kopij modela se ne more zmestiti, delavni bazen pa bi le povečal tekmovanje za vir
* Sistemi z **7 GB ali več VRAM** lahko Texture Aware obdelujejo sočasno, čeprav z manj delavci v primerjavi s standardno metodo
* Na **Jetsonu** je Texture Aware vedno dodeljen enemu delavcu, na modelih z nizko porabo energije (Nano, Orin Nano) pa samodejno uporabi tudi omejitev frekvence grafičnega procesorja – glej [NVIDIA Jetson Guide](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)***

## Upravljanje temperature (Jetson)

Naprave Jetson imajo omejitve glede temperature, zlasti pri namestitvah v zaprtih prostorih ali v zraku. Chloros spremlja vgrajene temperaturne senzorje naprave Jetson in samodejno prilagaja velikost serij:

| Temperatura | Odziv |
| --- | --- |
| **&lt; 70 °C** | Normalno delovanje – polna hitrost |
| **70 °C** (Opozorilo) | Velikost serije se postopoma zmanjšuje (100 % → 50 % med 70 °C in 80 °C) |
| **80 °C** (kritično) | Agresivno omejevanje zmogljivosti (50 % → 0 % med 80 °C in 90 °C) |
| **90 °C** (Izklop) | Popolna zaustavitev obdelave s GPU |

Na namiznih sistemih z ustreznim hlajenjem se termično omejevanje redko sproži.

***

## Obravnavanje pritiska na pomnilnik

Chloros med obdelavo neprekinjeno spremlja pomnilnik GPU in se odziva na treh ravneh.

**Velikost serije.** Serija se začne z 8 slikami, pomnoženimi z multiplikatorjem platforme iz zgornjih tabel. Chloros nato preveri prosti VRAM, 20 % tega rezervira za lastne potrebe PyTorcha, in predpostavi približno 100 MB pomnilnika GPU na sliko z ločljivostjo 12 MP – serija je tista, ki je manjša, bodisi omejitev, izpeljana iz pomnilnika, bodisi osnovna vrednost platforme. Nikoli ne pade pod 1.**Preventivno zmanjšanje.**Pri**izkoriščenosti VRAM nad 85 %** se velikosti serij zmanjšajo, preden pride do kakršne koli napake.**Zmanjšanje dodelitve na nit.** Ko se trenutna izkoriščenost poveča, se proračun GPU za vsako nit zmanjša: ×0,75 nad 80 % izkoriščenosti, ×0,5 nad 90 %. Pragi monitorja so 70 % (konzervativni), 85 % (normalna meja delovanja) in 95 % (tveganje za OOM).**OOM umik in obnovitev.** Če kljub vsemu pride do dogodka pomanjkanja pomnilnika:

* se velikost serije **zmanjša za polovico**, pri vsakem naslednjem pomanjkanju pomnilnika pa še enkrat za polovico – vsaka naslednja uspešna serija to kazen zmanjša za eno stopnjo
* se dodelitve aktivnim nitim zmanjšajo na 70 % njihove trenutne vrednosti, dodeljevalnik pa preklopi na konzervativno strategijo, ki se po nizu uspešnih dodelitev spet sprosti
* pod močnim pritiskom se potek obdelave umakne iz `fused_gpu` na `tiled_gpu`, v skrajnem primeru pa na `cpu_fallback`

**Pomnilnik gostitelja (Jetson).** Pred obdelavo CLI oceni največjo porabo pomnilnika gostitelja na podlagi števila slik in načina debayera ter opozori, če je verjetno, da pomnilnik RAM skupaj z zamenjavo, podprto s datotekami, ne bo zadostoval, pri čemer izpiše natančne ukaze za dodajanje zamenjave – glejte [Priročnik za NVIDIA Jetson](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).***

## Spremljanje prilagajanja računalniške zmogljivosti

### Sistemsko diagnostiko

`chloros-cli selftest` je najhitrejši način za preverjanje, kaj vidi računalniški sloj:

```bash
chloros-cli selftest
```

Njegovih 7 preverjanj zajema različico, razpoložljivost vrat, zagon backenda, `/api/test`, sistemske informacije, prisotnost modela za odstranjevanje šuma ter pripravljenost CUDA in odstranjevalca šuma. Preverjanje 5 neposredno izpiše vrstico s podatki o strojni opremi:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

Preverjanje 7 izpiše `CUDA: <bool>, Denoiser: <bool>` — oba morata biti resnična, da je Texture Aware sploh uporaben.

### Dnevniki ozadja

Strategija in število delavcev se izbereta znotraj ozadja na začetku vsakega izvajanja — ni nobenega napisa CLI, ki bi ju napovedoval. Ko se nekaj obnaša nepričakovano (prehod na rezervno pot GPU, OOM, denoiser, ki se ne naloži), se to prikaže v dnevniku backenda za tisto sejo:

| Platforma | Mesto dnevnika |
| --- | --- |
| **Linux / Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (ena datoteka na zagon) |
| **Linux, CLI – zagnani backend** | tudi `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Napredek v realnem času

Med izvajanjem CLI prikazuje napredek v realnem času za vsak niti (zaznavanje, analiziranje, obdelava, izvoz), ki se prenaša prek dogodkov, poslanih s strežnika (Server-Sent Events) — to je praktičen pokazatelj, ali je nit 3 ozko grlo. Glej [Obdelovalni potek](processing-pipeline.md).

***

## Naslednji koraki

* [Obdelovalni potek](processing-pipeline.md) — Razumevanje arhitekture poteka s 4 niti
* [Vodnik za NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Namestitev in optimizacija, specifična za Jetson
* [CLI : Ukazna vrstica](../CLI.md) — Priročnik za CLI
* [CLI Referenca](../reference/cli-reference.md) — Izčrpen seznam ukazov za različico 1.2.0

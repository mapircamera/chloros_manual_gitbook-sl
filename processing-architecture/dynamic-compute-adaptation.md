# Dinamično prilagajanje računalniške zmogljivosti

Chloros 1.1.0 uvaja inteligentno prepoznavanje strojne opreme in samodejno izbiro strategije obdelave. Obdelovalni motor se prilagaja vaši strojni opremi – od Jetson Nano do delovne postaje z več grafičnimi karticami – brez kakršne koli ročne konfiguracije.

***

## Kako deluje

Ko se Chloros zažene, samodejno profilira vaš sistem:

1. **Zazna operacijski sistem** – Windows ali Linux
2. **Prepozna jedra CPU in skupni RAM**

3.**Zazna prisotnost GPU** — zmogljivost NVIDIA CUDA, VRAM, model
4. **Prepozna model Jetson** (če je primerno) — prek `/proc/device-tree/model`
5. **Preveri toplotne senzorje** (Jetson) — za obdelavo, ki upošteva temperaturo
6. **Izbere optimalno strategijo računanja** — na podlagi vse zaznane strojne opreme
7. **Samodejno konfigurira število delavcev, vrsto poteka in dodelitev pomnilnika**Rezultat se shrani v predpomnilnik, tako da se naslednja izvedba začne hitreje. Če se strojna oprema spremeni (npr. dodan je grafični procesor), Chloros ob naslednjem zagonu ponovno ustvari profil.***

## Strategije računanja

Chloros izbere eno od treh strategij računanja na podlagi vaše strojne opreme:

| Strategija | Potreben GPU | Delavci | Potek | Najbolj primerno za |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`** | Da (12 GB+ VRAM ali 16 GB+ skupno) | 3–4 | `fused_gpu` | Namizni grafični procesorji z 12 GB+, Jetson Orin NX 16 GB, AGX Orin |
| **`GPU_SINGLE`** | Da (&lt; 12 GB VRAM) | 1–3 | `tiled_gpu` | Grafične kartice osnovnega razreda, Jetson Nano, Orin Nano |
| **`CPU_PARALLEL`** | Ne | jedra – 1 | `cpu_fallback` | Sistemi brez grafičnega procesorja NVIDIA |

### Vrste poti

* **`fused_gpu`** — Celotna pot obdelave grafičnega procesorja. Vse operacije debayeringa, korekcije in indeksiranja potekajo na grafičnem procesorju v enem samem združenem prehodu. Najvišja prepustnost, vendar zahteva več VRAM-a.
* **`tiled_gpu`** — Pot grafičnega procesorja, ki je učinkovita z vidika pomnilnika. Obdeluje slike v ploščicah, da se prilagodijo omejenemu pomnilniku grafičnega procesorja. Nižja prepustnost, vendar deluje na napravah z omejenim pomnilnikom.
* **`cpu_fallback`** — Obdelava izključno na CPU z uporabo večnitne vzporednosti. Uporablja se, kadar ni na voljo grafičnega procesorja NVIDIA.***

## Delovanje, specifično za platformo

| Platforma | Strategija | Delavci | Potek | Opombe |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (serializirano) | Način z učinkovito rabo pomnilnika, obdeluje eno sliko naenkrat |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` (sočasno) | Priporočena robna naprava — resnično vzporedno obdelovanje GPU |
| **Jetson AGX Orin 64 GB** | `GPU_PARALLEL` | 4 | `fused_gpu` (sočasno) | Največja zmogljivost na robu omrežja |
| **Namizni računalnik z 8 GB GPU** | `GPU_SINGLE` | 3 | `tiled_gpu` | Dobra zmogljivost namiznega računalnika z pomnilniško učinkovitimi ploščicami |
| **Namizni računalnik z grafično kartico 12 GB+** | `GPU_PARALLEL` | 3–4 | `fused_gpu` | Optimalna zmogljivost namiznega računalnika |
| **Sistem samo s CPU** | `CPU_PARALLEL` | jedra – 1 | `cpu_fallback` | GPU ni potreben, uporablja ThreadPool |

{% hint style="info" %}
**Enotni pomnilnik Jetson**: Naprave Jetson si delijo pomnilnik grafičnega procesorja in procesorja. Naprava Jetson Orin NX 16 GB poroča o ~15,3 GB VRAM-a, vendar gre za isti fizični RAM, ki ga uporabljajo operacijski sistem in procesi procesorja. Chloros to upošteva pri nastavitvi pragov za dodeljevanje pomnilnika.
{% endhint %}

***

## Dinamična dodelitev pomnilnika GPU

Chloros uporablja [4-nitni procesni potek](processing-pipeline.md):

* **Nit 1** (Zaznavanje) — nalaganje slike, razčlenjevanje EXIF, zaznavanje cilja
* **Nit 2** (Kalibracija) — izračun kalibracije odbojnosti
* **Nit 3** (Obdelava) — GPU debayer, popravek vinjetiranja, izračun indeksa
* **Nit 4** (Izvoz) — Pisanje datotek, vgrajevanje metapodatkov

Ko prejšnje niti poteka zaključijo svoje delo (npr. ko so bile zaznane vse slike), se njihova dodelitev pomnilnika GPU sprosti in **ponovno razporedi na preostale aktivne niti**. To pomeni, da niz 3 (faza, ki intenzivno uporablja GPU) s potekom procesa postopoma dobiva več pomnilnika, kar izboljša zmogljivost za najbolj računsko intenzivno delo.

### Faze dodeljevanja

| Faza | Aktivni nizi | Porazdelitev pomnilnika GPU |
| --- | --- | --- |
| **Zgodnja** | 1, 2, 3, 4 | Razdeljeno med vse nize |
| **Zgodnje-srednje** | 2, 3, 4 | Pomnilnik niti 1 se prerazporedi |
| **Srednje-pozno** | 3, 4 | Pomnilnik niti 1+2 gre k 3+4 |
| **Pozno** | 3 ali 4 | Največji pomnilnik za preostalo nit |***

## Obdelava Texture Aware

Metoda Texture Aware debayer (samo Chloros+) zaradi modela za odstranjevanje šuma AI/ML porabi znatno več pomnilnika GPU kot standardna metoda:

* Sistemi z **&lt; 7 GB VRAM** so v načinu Texture Aware prisiljeni v sinhrono obdelovalno zanko (ena slika naenkrat)
* Sistemi z **7 GB+ VRAM** lahko obdelujejo Texture Aware sočasno, čeprav z manj delavci v primerjavi s Standard***

## Upravljanje toplote (Jetson)

Naprave Jetson imajo toplotne omejitve, zlasti v zaprtih ali letalskih namestitvah. Chloros spremlja temperature GPU in CPU ter samodejno prilagaja obdelavo:

| Temperatura | Odziv |
| --- | --- |
| **&lt; 70 °C** | Normalno delovanje — polna hitrost |
| **70 °C** (Opozorilo) | Zmanjšanje velikosti serije |
| **80 °C** (Kritično) | Agresivno omejevanje — zmanjšanje sočasnosti in števila delavcev |
| **90 °C** (Izklop) | Popolna zaustavitev obdelave GPU |

Za spremljanje temperature se na platformah Jetson uporablja `tegrastats`. Na namiznih sistemih z ustreznim hlajenjem se termično omejevanje redko sproži.

***

## Obravnavanje pritiska na pomnilnik

Chloros med obdelavo spremlja obremenitev sistemskega pomnilnika:

* **Prag pomnilnika**: 85-odstotna izkoriščenost sproži konzervativno delovanje
* **Zmanjšanje OOM**: Če pride do dogodka pomanjkanja pomnilnika, se dodelitev zmanjša za 25 % (multiplikator 0,75x)
* **Preklop poteka**: Pri hudem pritisku na pomnilnik se cevovod samodejno preklopi iz `fused_gpu` na `tiled_gpu`
* **Priporočila za zamenjavo**: Na Jetsonu vas Chloros opozori, če je prostora za zamenjavo premalo za velikost vašega niza podatkov***

## Spremljanje prilagajanja računalniške moči

### Izpis stanja CLI

Ko se obdelava začne, CLI prikaže zaznan profil strojne opreme:

```

Chloros CLI 1.1.0
Platform: Linux aarch64 (Jetson Orin NX 16GB)
Strategy: GPU_PARALLEL | Workers: 3 | Pipeline: fused_gpu
CUDA: Available | GPU Memory: 15.3 GB (shared)
```

### Diagnostika sistema

Za ogled celotnega profila strojne opreme in preverjanje zmogljivosti računalniškega procesiranja zaženite `chloros-cli selftest`:

```bash
chloros-cli selftest
```

To preveri razpoložljivost CUDA, pomnilnik GPU, modele za odstranjevanje šuma in povezljivost z ozadjem.

***

## Naslednji koraki

* [Obdelovalni potek](processing-pipeline.md) — Razumevanje arhitekture poteka s 4 nitmi
* [Vodnik za NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Razporeditev in optimizacija, specifična za Jetson
* [CLI : Ukazna vrstica](../CLI.md) — Celoten CLI referenčni priročnik

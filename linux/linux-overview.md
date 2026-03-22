# Linux Pregled

Chloros 1.1.0 prinaša vgrajeno podporo za **CLI**in**Python SDK**, kar omogoča obdelavo večspektralnih slik brez grafičnega vmesnika na delovnih postajah, strežnikih in napravah NVIDIA Jetson.

{% hint style="info" %}
**Na Linux ni grafičnega vmesnika.** Grafični vmesnik Chloros Desktop je na voljo le na Windows. Uporabniki Linux komunicirajo z Chloros prek [CLI](../CLI.md) in [Python SDK](../api-python-sdk.md).
{% endhint %}

***

## Matrika podpore platform

| Funkcija | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Namizni GUI** | Da | N/A | Ne | Ne |
| **CLI** | Da | Da | Da | Da |
| **Python SDK** | Da | Da | Da | Da |
| **Pospeševanje z grafičnim procesorjem (CUDA)** | Da | Da | Da | Da (JetPack 6) |
| **Debayer z upoštevanjem tekstur** | Da (Chloros+) | Da (Chloros+) | Da (Chloros+) | Da (Chloros+) |
| **Dinamična prilagoditev računanja** | Da | Da | Da | Da |***

## Podprte arhitekture

| Arhitektura | Opis | Način namestitve |
| --- | --- | --- |
| **amd64 (x86_64)** | Standardni namizni/strežniški procesorji (Intel, AMD) | Paket `.deb` |
| **arm64 (aarch64)** | Procesorji na osnovi ARM, predvsem NVIDIA Jetson | Paket `.deb` (JetPack 6) |

## Podprte distribucije Linux

* **Ubuntu 20.04+** (amd64)
* **Debian 11+** (amd64)
* **NVIDIA JetPack 6** (arm64 — platforme Jetson)***

## Kaj dobijo uporabniki Linux

* **Chloros CLI** — Celoten vmesnik ukazne vrstice za obdelavo v paketih, avtomatizacijo in pisanje skript
* **Chloros Python SDK** — Programski vmesnik Python (`pip install chloros-sdk`) za integracijo v raziskovalne poti in prilagojena orodja
* **Pospešitev z GPU** — Obdelava, pospešena s CUDA, na grafičnih procesorjih NVIDIA (namizni in Jetson)
* **Dinamična prilagoditev računalniških zmogljivosti** — Samodejno zaznavanje strojne opreme in optimizacija strategije obdelave
* **Vse funkcije obdelave** — Enak multispektralni procesni tok kot Windows (kalibracija, popravljanje vinjetiranja, indeksi vegetacije, vsi izvozni formati)
* **Funkcije Chloros+** — večnitna obdelava, Texture Aware debayer, prilagojeni indeksi (z licenco Chloros+)

## Kaj uporabniki Linux ne dobijo

* **Namizni grafični vmesnik** — Ni grafičnega vmesnika; vsa interakcija poteka prek CLI ali Python SDK
* **Pregledovalnik slik** — Ni interaktivnega pregledovalnika slik, pogleda v mreži ali označevalcev na zemljevidu
* **Vizualno upravljanje projektov** — Projekti se upravljajo prek ukazov CLI in klicov SDK***

## Začetek dela z Linux

1. **Namestite Chloros** — Glejte [Namestitev Linux](linux-installation.md) za namestitev paketa `.deb`
2. **Namestite Python SDK** (neobvezno) — `pip install chloros-sdk`
3. **Aktivirajte svojo licenco** — `chloros-cli login your@email.com 'password'`
4. **Obdelajte svoj prvi niz podatkov** — `chloros-cli process ~/datasets/flight001`

Uporabniki NVIDIA Jetson naj si ogledajo poseben [vodnik za NVIDIA Jetson](nvidia-jetson-guide.md) za nastavitev in optimizacijo, prilagojeno tej platformi.

***

## Naslednji koraki

* [Namestitev Linux](linux-installation.md) — Podrobna navodila za namestitev za amd64 in arm64
* [Vodnik za NVIDIA Jetson](nvidia-jetson-guide.md) — Nastavitev, upravljanje toplote in namestitev v terenskem okolju, specifično za Jetson
* [CLI : Ukazna vrstica](../CLI.md) — Celoten referenčni priročnik za CLI
* [API : Python SDK](../api-python-sdk.md) — Celoten referenčni priročnik za SDK
* [Dinamična prilagoditev računalniških zmogljivosti](../processing-architecture/dynamic-compute-adaptation.md) — Kako se Chloros prilagaja vaši strojni opremi

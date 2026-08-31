# Linux – Pregled

Chloros 1.2.0 zagotavlja vgrajeno podporo Linux za **CLI**in**Python SDK** — obdelavo multispektralnih slik brez zaslona ter nadzor kamere LATTICE v realnem času in svetlobnega senzorja DAQ — na delovnih postajah, strežnikih in napravah NVIDIA Jetson na robu omrežja.

{% hint style="info" %}
**Na Linux ni namiznega grafičnega vmesnika.**Namizni grafični vmesnik Chloros je na voljo izključno na Windows. Uporabniki Linux komunicirajo z Chloros prek [CLI](../CLI.md) in [Python SDK](../api-python-sdk.md). `.deb` v vaš meni aplikacij res doda vnos**Chloros CLI** v meniju vaše aplikacije – preprosto odpre emulator terminala, ki izvaja `chloros-cli`.
{% endhint %}

***

## Matrika podpore platform

| Funkcija | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Namizni grafični vmesnik** | Da | N/A | Ne | Ne |
| **CLI** (`chloros-cli`) | Da | Da | Da | Da |
| **Python SDK** (`chloros-sdk`) | Da | Da | Da | Da |
| **Procesna veriga za obdelavo slik** | Da | Da | Da | Da |
| **Upravljanje kamer LATTICE (v živo)** | Da (zavihek »Kamere«) | Da (`chloros-cli lattice`, SDK) | Da | Da |
| **Senzorji svetlobe DAQ (v živo)** | Da (zavihek Senzorji svetlobe) | Da (`chloros-cli daq pool-*`, SDK) | Da | Da |
| **Sinhronizacija časa PTP (gostitelj je glavni strežnik)** | Da | Da (`chloros-cli time-sync`) | Da | Da |
| **Pospeševanje z GPU (CUDA)** | Da | Da | Da | Da (JetPack 6) |
| **Debayer z upoštevanjem tekstur** | Da (Chloros+) | Da (Chloros+) | Da (Chloros+) | Da (Chloros+) |
| **Dinamična prilagoditev računalniških zmogljivosti** | Da | Da | Da | Da |
| **Backend kot sistemska storitev** (`chloros-backend.service`) | Ne | Ne | Da (po izbiri) | Da (po izbiri) |
| **Posodobitveni program na mestu** (`chloros-cli update`) | Ne (zaženite namestitveni program) | Ne (zaženite namestitveni program) | Da | Da |***

## Podprte arhitekture

| Arhitektura | Opis | Paket |
| --- | --- | --- |
| **amd64 (x86_64)** | Standardni namizni/strežniški procesorji (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | Procesorji ARM — družina NVIDIA Jetson Orin | `chloros_<version>_arm64_jp6.deb` (različica JetPack 6) |

## Podprte distribucije Linux

* **Ubuntu 22.04 LTS ali novejša različica** (amd64)
* **Debian 12 ali novejša različica** (amd64)
* **NVIDIA JetPack 6** (arm64 — platforme Jetson Orin)***

## Kaj dobijo uporabniki Linux

* **Chloros CLI** — popoln vmesnik ukazne vrstice za obdelavo v paketih, avtomatizacijo in pisanje skript
* **Chloros Python SDK** — programski vmesnik Python za raziskovalne poteke in prilagojena orodja (namestljiv iz PyPI, vključen pa je tudi v `.deb` kot wheel, ki ustreza različici)
* **Upravljanje kamer LATTICE** — odkrivanje, povezovanje, konfiguriranje in zajemanje s kamerami LATTICE ter sinhroniziranimi sklopi več kamer prek `chloros-cli lattice` in SDK; `.deb` vključuje izvedbeno okolje Arena SDK, ki ga kamere potrebujejo
* **Upravljanje svetlobnih senzorjev DAQ** — priključite senzorje DAQ-U/M/E, pretakajte kalibrirane spektre in snemajte datoteke `.daq` prek `chloros-cli daq pool-*` in SDK
* **Sinhronizacija časa PTP** — zadnji del Chloros upravlja glavni PTP-strežnik, na katerega so kamere LATTICE in senzorji DAQ-E sinhronizirani; preverite ga z `chloros-cli time-sync`, in ga brez grafičnega vmesnika ohranjajte v delovanju s sistemsko enoto `chloros-backend.service` (glejte [Namestitev Linux](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Avtomatizacija projektov** — shranjene projekte izvajajte v brezuporabniški način s pomočjo `chloros-cli project` in `open_project` v SDK
* **Pospeševanje z grafičnim procesorjem (GPU)** — obdelava s pospeševanjem CUDA na grafičnih procesorjih NVIDIA (namiznih računalnikih in Jetson)
* **Dinamična prilagoditev računalniške moči** — samodejno zaznavanje strojne opreme in izbira strategije obdelave, s prepisom `CHLOROS_STRATEGY` kot izhodom v sili za strokovnjake
* **Vse funkcije obdelave** — isti potek obdelave kot pri Windows: kalibracija, popravljanje vinjetiranja, indeksi vegetacije in vsi formati izvoza
* **Funkcije Chloros+** — večnitna (potekna) obdelava, Texture Aware debayer in prilagojeni indeksi, s plačanim načrtom Chloros+

## Kaj uporabniki Linux ne dobijo

* **Namizni grafični vmesnik** — ni grafičnega vmesnika; vsa interakcija poteka prek CLI ali Python SDK
* **Pregledovalnik slik** — ni interaktivnega pregledovalnika slik, pogleda v mreži ali označevalcev na zemljevidu
* **Vizualno upravljanje projektov** — projekti se ustvarjajo in vodijo prek ukazov CLI ter klicev SDK (sama strojna oprema — kamere, senzorji, zajem — ostaja v celoti nadzorovana prek terminala)***

## Zahteve glede licence

Za dostop do CLI in SDK je potrebna **plačljiva stopnja Chloros+ — Copper ali višja**(Copper, Bronze, Silver, Gold). Brezplačna stopnja**Iron** nima dostopa do CLI/SDK. Ta omejitev se izvaja na strežniški strani, ne le v CLI:

| Situacija | Odgovor strežnika |
| --- | --- |
| Ni prijavljen | `401` z `error_code: AUTH_REQUIRED` |
| Prijavljen na brezplačni stopnji Iron | `403` z `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` deluje na kateri koli stopnji – to je edina pot, ki je izvzeta iz prepreke –, zato je razlog za zavrnitev vedno viden.

***

## Začetek dela z Linux

1. **Namestite Chloros** — glejte [Namestitev Linux](linux-installation.md) za namestitev `.deb`
2. **Preverite** — `chloros-cli --version` natisne `Chloros CLI 1.2.0`; `chloros-cli selftest` izvede 7-stopenjsko diagnostiko
3. **Namestite Python in SDK** (neobvezno) — `pip install chloros-sdk`
4. **Prijavite se** — `chloros-cli login your@email.com 'your-password'` (enkrat na napravo in ponovno po vsaki nadgradnji paketa)
5. **Obdelajte svoj prvi nabor podatkov** — `chloros-cli process ~/datasets/flight001`

Za NVIDIA Jetson si oglejte namenski [Vodnik za NVIDIA Jetson](nvidia-jetson-guide.md) za nastavitev, specifično za platformo, toplotno obnašanje in namestitev v praksi.

***

## Naslednji koraki

* [Namestitev Linux](linux-installation.md) — podrobna namestitev, lokacije datotek in odpravljanje težav za amd64 in arm64
* [Vodnik za NVIDIA Jetson](nvidia-jetson-guide.md) — nastavitve, specifične za Jetson, delovanje pomnilnika in toplotno obnašanje ter namestitev v praksi
* [CLI : Ukazna vrstica](../CLI.md) — vodnik za CLI
* [API : Python SDK](../api-python-sdk.md) — priročnik SDK
* [CLI Referenca](../reference/cli-reference.md) in [SDK Referenca](../reference/sdk-reference.md) — izčrpni seznami ukazov/API za različico 1.2.0
* [Dinamična prilagoditev računalniške zmogljivosti](../processing-architecture/dynamic-compute-adaptation.md) — kako se Chloros prilagaja vaši strojni opremi

{% hint style="info" %}
**Programsko branje tega priročnika.** Vsaka stran je na voljo tudi kot surovi Markdown na svoji strani URL ter `.md` (na primer `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), indeks celotnega priročnika pa je objavljen na [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}

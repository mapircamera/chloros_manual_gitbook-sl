# Pregled sistema Linux

Chloros 1.3.0 zagotavlja nativno podporo za sistem Linux za **CLI**in**Python SDK** – obdelavo multispektralnih slik brez grafičnega vmesnika ter nadzor kamer LATTICE in svetlobnih senzorjev DAQ v realnem času – na delovnih postajah, strežnikih in napravah NVIDIA Jetson za obdelavo podatkov na robu omrežja.

{% hint style="info" %}
**Na Linuxu ni namiznega grafičnega vmesnika.**Namizni grafični vmesnik Chloros je na voljo izključno za Windows. Uporabniki Linuxa komunicirajo s Chlorosom prek [CLI](../CLI.md) in [Python SDK](../api-python-sdk.md). `.deb` v vaš meni aplikacij doda vnos**Chloros CLI** — preprosto odpre emulator terminala, na katerem teče `chloros-cli`.
{% endhint %}

***

## Matrika podpore platform

| Funkcija | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/ --- | --- | --- | --- |
| **Namizni GUI** | Da | N/A | Ne | Ne |
| **CLI** (`chloros-cli`) | Da | Da | Da | Da |
| **Python SDK** (`chloros-sdk`) | Da | Da | Da | Da |
| **Procesna veriga za obdelavo slik** | Da | Da | Da | Da |
| **Upravljanje kamere LATTICE (v živo)** | Da (zavihek »Kamere«) | Da (`chloros-cli lattice`, SDK) | Da | Da |
| **Svetlobni senzorji DAQ (v živo)** | Da (zavihek Svetlobni senzorji) | Da (`chloros-cli daq pool-*`, SDK) | Da | Da |
| **Sinhronizacija časa PTP (gostitelj je glavni strežnik)** | Da | Da (`chloros-cli time-sync`) | Da | Da |
| **Pospeševanje z GPU (CUDA)** | Da | Da | Da | Da (JetPack 6) |
| **Texture Aware debayer** (potreben je grafični procesor s podporo CUDA) | Da (Chloros+) | Da (Chloros+) | Da (Chloros+) | Da (Chloros+) |
| **Dinamična prilagoditev računalniških zmogljivosti** | Da | Da | Da | Da |
| **Backend kot sistemska storitev** (`chloros-backend@<user>.service`) | Ne | Ne | Da (po izbiri) | Da (po izbiri) |
| **Posodobitveni program na mestu** (`chloros-cli update`) | Ne (zaženite namestitveni program) | Ne (zaženite namestitveni program) | Da | Da |***

## Podprte arhitekture

| Arhitektura | Opis | Paket |
| --- | --- | --- |
| **amd64 (x86_64)** | Standardni namizni/strežniški procesorji (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | Procesorji ARM — družina NVIDIA Jetson Orin (JetPack 6) | `chloros_<version>_arm64_jp6.deb` |

## Podprte distribucije Linuxa

* **Ubuntu 22.04 LTS ali novejši** (amd64)
* **Debian 12 ali novejši** (amd64)
* **NVIDIA JetPack 6** (arm64 — platforme Jetson Orin)***

## Kaj dobijo uporabniki Linuxa

* **Chloros CLI** — popoln vmesnik ukazne vrstice za obdelavo v paketih, avtomatizacijo in pisanje skript
* **Chloros Python SDK** — programski vmesnik v jeziku Python za raziskovalne poteke in prilagojena orodja, ki ga namesti `.deb` za sistem `python3` (kolo z ustrezno različico za virtualenvs je v `/usr/lib/chloros/sdk/`; PyPI je namenjen računalnikom brez paketa Chloros)
* **Upravljanje kamer LATTICE** — odkrivanje, povezovanje, konfiguriranje in zajemanje s kamerami LATTICE ter sinhroniziranimi nizi več kamer prek `chloros-cli lattice` in SDK-ja; `.deb` vključuje izvedbeno okolje Arena SDK, ki ga kamere potrebujejo
* **Upravljanje svetlobnih senzorjev DAQ** — priključite senzorje DAQ-U/M/E, pretakajte kalibrirane spektre in snemajte datoteke `.daq` prek `chloros-cli daq pool-*` in SDK-ja
* **Sinhronizacija časa PTP** — backend Chloros izvaja glavni PTP-strežnik, na katerega so kamere LATTICE in senzorji DAQ-E sinhronizirani; preverite ga z `chloros-cli time-sync` in ga ohranjajte v brezuporabniškem načinu z enoto systemd `chloros-backend@<user>.service` (glejte [Namestitev v Linuxu](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Avtomatizacija projektov** — shranjene projekte izvajajte brez uporabniškega vmesnika s pomočjo `chloros-cli project` in `open_project` iz SDK-ja
* **Pospeševanje z GPU** — obdelava s pospeševanjem CUDA na grafičnih procesorjih NVIDIA (namizni grafični procesorji na amd64 in grafični procesor Jetsona z JetPack 6)
* **Dinamična prilagoditev računanja** — samodejno zaznavanje strojne opreme in izbira strategije obdelave, s prepisom `CHLOROS_STRATEGY` kot izhodom v sili za strokovnjake
* **Vse funkcije obdelave** — isti potek obdelave kot v sistemu Windows: kalibracija, popravljanje vinjetiranja, indeksi vegetacije in vsi formati izvoza
* **Funkcije Chloros+** — večnitna (pipelinirana) obdelava, Texture Aware debayer (na grafičnem procesorju CUDA) in prilagojeni indeksi, s plačanim načrtom Chloros+

## Kaj uporabniki Linuxa ne dobijo

* **Namizni grafični vmesnik** — ni grafičnega vmesnika; vsa interakcija poteka prek CLI-ja ali Python SDK-ja
* **Pregledovalnik slik** — ni interaktivnega pregledovalnika slik, pogleda v mreži ali označevalcev na zemljevidu
* **Vizualno upravljanje projektov** — projekti se ustvarjajo in upravljajo prek ukazov CLI in klicev SDK (sama strojna oprema — kamere, senzorji, zajem — ostaja v celoti nadzorovana iz terminala)***

## Zahteve glede licence

Dostop do CLI in SDK zahteva **plačljiv paket Chloros+ — Copper ali višji**(Copper, Bronze, Silver, Gold). Brezplačni paket**Iron** nima dostopa do CLI/SDK. Ta omejitev se izvaja na strani strežnika, ne le prek CLI:

| Situacija | Odgovor strežnika |
| --- | --- |
| Niste prijavljeni | `401` z `error_code: AUTH_REQUIRED` |
| Prijavljen na brezplačni stopnji Iron | `403` z `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` deluje na kateri koli stopnji — to je edina pot, ki je izvzeta iz prepreke — zato je razlog za zavrnitev vedno viden.

***

## Začetek dela na Linuxu

1. **Namestite Chloros** — glejte [Namestitev na Linuxu](linux-installation.md) za namestitev `.deb`
2. **Preverite** — `chloros-cli --version` izpiše `Chloros CLI 1.3.0`; `chloros-cli selftest` izvede 7-stopenjsko diagnostiko
3. **Preverite Python SDK** — `.deb` ga namesti: `python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"` (v virtualnem okolju virtualenv, `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`)
4. **Prijavite se** — `chloros-cli login your@email.com 'your-password'` (enkrat na računalnik; nadgradnje paketa ohranijo prijavne podatke, odstranitev paketa pa jih izbriše)
5. **Obdelajte svoj prvi nabor podatkov** — `chloros-cli process ~/datasets/flight001`

Za NVIDIA Jetson si oglejte namenski [Vodnik za NVIDIA Jetson](nvidia-jetson-guide.md) za nastavitev, specifično za platformo, toplotno obnašanje in namestitev v praksi.

***

## Naslednji koraki

* [Namestitev v sistemu Linux](linux-installation.md) — podrobna namestitev, lokacije datotek in odpravljanje težav za arhitekturi amd64 in arm64
* [Vodnik za NVIDIA Jetson](nvidia-jetson-guide.md) — nastavitev, specifična za Jetson, delovanje pomnilnika in toplotno obnašanje ter uvajanje v praksi
* [CLI: ukazna vrstica](../CLI.md) — vodnik po ukazni vrstici
* [API: Python SDK](../api-python-sdk.md) — vodnik po SDK
* [Referenca CLI](../reference/cli-reference.md) in [Referenca SDK](../reference/sdk-reference.md) — izčrpni seznami ukazov/API za različico 1.3.0
* [Dinamično prilagajanje računalniške moči](../processing-architecture/dynamic-compute-adaptation.md) — kako se Chloros prilagaja vaši strojni opremi

{% hint style="info" %}
**Programsko branje tega priročnika.** Vsaka stran je na voljo tudi kot surovi Markdown na lastnem URL-ju, ki vsebuje `.md` (na primer `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), indeks celotnega priročnika pa je objavljen na [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}

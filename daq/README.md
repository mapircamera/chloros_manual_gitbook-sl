# Svetlobni senzorji DAQ

> **Iščete informacije o strojni opremi?**Sami senzorji – modeli, način pritrditve, pokrovčki, vmesniki, napajanje in aplikacija SCANNER – so opisani v**[uporabniškem priročniku za DAQ](https://mapir.gitbook.io/daq)**. To poglavje obravnava njihovo uporabo v programu Chloros. Svetlobni senzorji**DAQ** podjetja

MAPIR merijo okoljsko svetlobo kot radiometrično kalibrirane spektre. V programu Chloros imajo dve vlogi:

* **Samostojni spektralni instrument** — grafi spektra v realnem času, kolorimetrični podatki in posnetki `.daq`, vse iz [zavihek Svetlobni senzorji](gui.md), [CLI](cli-quick-start.md) ali Python SDK.
* **Vir navzdol usmerjene svetilnosti za odbojnost** — med obdelavo Chloros interpolira vaše odčitke `.daq` na časovni žig osvetlitve vsakega posnetka in uporabi izmerjeno navzdolno svetlobo za pretvorbo svetilnosti kamere v odbojnost (`--reflectance-source daq`), za kalibrirane pasove pa ni potreben panel v prizoru.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Trije modeli, en format podatkov

| Model | Prenos | Odkrivanje |
| --- | --- | --- |
| **DAQ-U** | USB (serijski) | skeniranje serijskega vmesnika |
| **DAQ-M** | Bluetooth Low Energy | skeniranje BLE po imenu |
| **DAQ-E** | Ethernet (IPv4, napajanje prek PoE) | mDNS `_daq-e._tcp` (ime gostitelja `daq-e-<id>.local`) |**DAQ-E-S (novost v različici 1.3.0).** DAQ-E s fiksnim difuzorjem in senzorjem naklona. V Chlorosu je naveden kot `DAQ-E-S` in deluje popolnoma enako kot DAQ-E, razen da ne upošteva korekcije pokrova in vsaka odčitana vrednost vsebuje podatke o naklonu, zvrtanju in nagibu naprave. Glej [DAQ-E-S: fiksni difuzor in nagib](caps-and-range.md#daq-e-s-fixed-diffuser-and-tilt).

Vsi trije uporabljajo isti komunikacijski protokol in zagotavljajo identične podatke:

* **135-točkovni spekter od 340 do 1010 nm v korakih po 5 nm**, skupaj s tristimulskimi vrednostmi CIE XYZ, v vsakem okviru.
* **Radiometrično kalibrirana spektralna obsevanost v W/m²/nm** — paket tovarniške kalibracije vsake enote (skupaj z njenim aktivnim profilom korekcije pokrova) se uporabi, preden podatki pridejo do vas.
* Enak **format snemanja `.daq`** (datoteka SQLite). Nadaljnja obdelava je enaka ne glede na to, kateri prenosni protokol je ustvaril datoteko.

Prenosni sklopi (serijski USB, BLE, mDNS/zeroconf) so vključeni v ozadje programa Chloros — ni treba namestiti ničesar, da bi komunicirali s katerim koli od treh modelov prek grafičnega vmesnika ali ukazov `pool-*` v CLI.

***

## Kalibrirano območje: poročano 340–1010 nm, kalibrirano ~374–974 nm

Senzor poroča o celotni mreži 340–1010 nm, vendar se radiometrični dobiček, sledljiv do NIST, razteza približno od **374–974 nm**. Chloros zavrne delitev po absolutni odbojnosti za kateri koli pas kamere, pri katerem je manj kot polovica spektralne teže znotraj tega kalibriranega območja; preskočen pas se poroča z razlogom za preskok `dls-uncalibrated-band-<nm>`.

Med trenutno dobavljenimi artikli filtrov LATTICE je prizadet le **F988**:

Odbojnost F988 je kalibrirana z uporabo plošče za merjenje odbojnosti v sceni: pas leži zunaj kalibriranega obsega svetlobnega senzorja DAQ, zato Chloros uporabi vaš najnovejši posnetek plošče in ga ohrani med posameznimi meritvami plošče.

Če se zajem modela F988 obdeluje zgolj z razpoložljivimi podatki DAQ, Chloros zavrne odbojnost, ki temelji na DAQ, za ta pas z razlogom za preskok `dls-uncalibrated-band-988` — [delovni tok z odbojno ploščo](../calibration-targets.md) je podprta pot za F988.

***

## ID-ji senzorjev

Vsak DAQ poroča o stabilni identifikacijski številki senzorja. Njena oblika se razlikuje glede na model:

| Model | Oblika ID | Primer |
| --- | --- | --- |
| DAQ-U | 5-oktetna s vezaji | `CB-7C-A8-2E-5F` |
| DAQ-M | 5-oktetna z vezaji | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

ID senzorja je:

* vtisnjen v vsako datoteko `.daq`, ki jo senzor zabeleži,
* ključ, ki ga Chloros uporablja za pridobivanje tovarniškega kalibracijskega paketa te enote,
* vrednost, ki jo posredujete `--sensor-id` v ukazih CLI, in
* za DAQ-E pa tudi njegovo mDNS-ime gostitelja (`daq-e-def330.local`) — vrednost, ki jo sprejme `--eth-host`.

***

## Tovarniška kalibracija in oblak

Vsaka enota DAQ je posamično tovarniško kalibrirana z radiometrično verigo, sledljivo do NIST, podjetje Chloros pa v vsako enoto naloži kalibracijski paket, ki je vezan na ID senzorja. Poročilo o kalibraciji za posamezno enoto (PDF) je mogoče prenesti iz nastavitev senzorja v [zavičku Svetlobni senzorji](gui.md).

{% hint style="warning" %}
**Za kalibracijo naprav DAQ-U in DAQ-M je potreben dostop do oblaka.**Noben od modelov ne shranjuje podatkov v pomnilniku naprave: njihovi tovarniški kalibracijski paketi so shranjeni v oblaku podjetjMAPIR in se pridobijo na podlagi identifikacijske številke senzorja (nato se shranijo v lokalni predpomnilnik). Chloros potrebuje internetno povezavo, da lahko posreduje kalibrirane podatke v W/m²/nm iz DAQ-U ali DAQ-M.**DAQ-E je izjema** — kalibracijo hrani na sami napravi.
{% endhint %}
<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

***

## Kje se shranijo posnetki

| Površina | Privzeta destinacija `.daq` |
| --- | --- |
| GUI — zavihek Svetlobni senzorji | `<project folder>/light_sensor/` (zaključeni posnetki se samodejno dodajo v projekt) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` na računalniku, na katerem teče backend (relativna pot `-o` se razreši glede na mapo, v kateri ste izvedli ukaz, če je strežnik na istem računalniku) |

Vsako ime datoteke `.daq` vsebuje ID senzorja in časovni žig.

***

## V tem poglavju

* [**Zavihek DAQ v programu Chloros**](gui.md) — popoln vodnik po grafičnem vmesniku: priključitev posameznih modelov, nastavitve za posamezne senzorje, spektralni grafi, kolorimetrični podatki v realnem času, odbojnost z dvema senzorjema in snemanje.
* [**Hitri začetek z CLI (pool-\*)**](cli-quick-start.md) — upravljanje senzorjev DAQ iz `chloros-cli daq pool-*`, podprta pot ukazne vrstice.
* [**Profilni omejevalniki in kalibrirano spektralno območje**](caps-and-range.md) — kateri omejevalniki so na voljo za posamezne modele, kako jih deklarirati ter podrobnosti o kalibriranem spektralnem območju.
* [**Snemanje in format .daq**](recording.md) — format SQLite `.daq` in poteki snemanja.
* [**Mrežno povezovanje DAQ-E in sinhronizacija časa**](ethernet-ptp.md) — načini prenosa DAQ-E in sinhronizacija časa PTP.
* [**Delovni postopki za odbojnost**](reflectance.md) — uporaba podatkov DAQ o navzdolnem sevanju za izračun odbojnosti.
* Za popolno dokumentacijo na ravni zastavic glejte [Referenco CLI](../reference/cli-reference.md) (poglavje `chloros-cli daq`) in [Referenco SDK](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), ki sta napisani tako, da ju lahko neposredno uporabljajo AI-pomočniki.

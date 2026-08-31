# Svetlobni senzorji DAQ

> **Iščete informacije o strojni opremi?**Sami senzorji – modeli, način pritrditve, pokrovi, vtičnice, napajanje in aplikacija SCANNER – so opisani v**[uporabniškem priročniku za DAQ](https://mapir.gitbook.io/daq)**. To poglavje obravnava njihovo uporabo od Chloros naprej.

Svetlobni senzorji **DAQ** iz MAPIR merijo okoljsko svetlobo kot radiometrično kalibrirane spektre. V Chloros imajo dve vlogi:

* **Samostojni spektralni instrument** — grafi spektra v realnem času, kolorimetrični podatki in posnetki `.daq`, vse to iz [zavička Svetlobni senzorji](gui.md), [CLI](cli-quick-start.md) ali Python SDK.
* **Vir navzdol usmerjene svetilnosti za odbojnost** — med obdelavo Chloros interpolira vaše odčitke `.daq` na časovni žig osvetlitve vsakega posnetkačasovni žig posnetka in uporabi izmerjeno navzdolno svetlobo za pretvorbo svetilnosti kamere v odbojnost (`--reflectance-source daq`), za kalibrirane pasove ni potreben panel v prizoru.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Trije modeli, en format podatkov

| Model | Prenos | Odkrivanje |
| --- | --- | --- |
| **DAQ-U** | USB (serijski) | skeniranje serijskega vmesnika |
| **DAQ-M** | Bluetooth Low Energy | skeniranje BLE po imenu |
| **DAQ-E** | Ethernet (IPv4, napajanje prek PoE) | mDNS `_daq-e._tcp` (ime gostitelja `daq-e-<id>.local`) |

Vsi trije uporabljajo isti komunikacijski protokol in posredujejo enake podatke:

* **135-točkovni spekter od 340 do 1010 nm v korakih po 5 nm**, skupaj s tristimulskimi vrednostmi CIE XYZ, v vsakem okviru.
* **Radiometrično kalibrirana spektralna obsevanost v W/m²/nm** — tovarniški kalibracijski paket vsake enote (skupaj z njenim aktivnim profilom korekcije pokrova) se uporabi, preden podatki pridejo do vas.
* Enak **format snemanja `.daq`** (datoteka SQLite). Nadaljnja obdelava je enaka ne glede na to, kateri prenosni kanal je ustvaril datoteko.

Prenosni sklopi (serijski USB, BLE, mDNS/zeroconf) so vključeni v ozadje Chloros — ni treba namestiti ničesar, da bi komunicirali s katerim koli od treh modelov prek grafičnega vmesnika ali ukazov CLI v `pool-*`.

***

## Kalibrirano območje: poročano 340–1010 nm, kalibrirano ~374–974 nm

Senzor poroča o celotni mreži 340–1010 nm, vendar se radiometrični dobiček, sledljiv po standardih NIST, razteza približno od **374–974 nm**. Chloros zavrne delitev absolutne odbojnosti za kateri koli pas kamere, pri katerem je manj kot polovica spektralne teže znotraj tega kalibriranega obsega; preskočen pas se poroča z razlogom za preskok `dls-uncalibrated-band-<nm>`.

Med dostavljenimi artikli filtrov LATTICE je prizadet le **F988**:

Odbojnost F988 se kalibrira z uporabo plošče za merjenje odbojnosti v sceni: pas leži zunaj kalibriranega območja svetlobnega senzorja DAQ, zato Chloros uporabi vaš najnovejši posnetek plošče in ga ohrani med posameznimi meritvami plošče.

Če se zajem modela F988 obdeluje zgolj z razpoložljivimi podatki DAQ, Chloros zavrne odbojnost na podlagi DAQ za ta pas z razlogom za preskok `dls-uncalibrated-band-988` — [delovni tok z odbojnostno ploščo](../calibration-targets.md) je podprta pot za F988.

***

## ID-ji senzorjev

Vsak DAQ poroča o stabilnem ID-ju senzorja. Njegova oblika se razlikuje glede na model:

| Model | Oblika ID-ja | Primer |
| --- | --- | --- |
| DAQ-U | 5 oktetov s vezaji | `CB-7C-A8-2E-5F` |
| DAQ-M | 5-oktetna z vezaji | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

ID senzorja je:

* vtisnjen v vsako datoteko `.daq`, ki jo zabeleži,
* ključ, ki ga Chloros uporablja za pridobitev tovarniškega kalibracijskega paketa te enote,
* vrednost, ki jo posredujete `--sensor-id` v ukazih CLI `pool-*`, ter
* za DAQ-E pa tudi njegovo ime gostitelja mDNS (`daq-e-def330.local`) — vrednost, ki jo sprejme `--eth-host`.

***

## Tovarniška kalibracija in oblak

Vsaka enota DAQ je tovarniško kalibrirana posamično z radiometrično verigo, sledljivo do NIST-a, Chloros pa v vsako enoto naloži kalibracijski paket, ki je vezan na ID senzorja. Poročilo o kalibraciji za posamezno enoto (PDF) je mogoče prenesti iz nastavitev senzorja v [zavihtku Svetlobni senzorji](gui.md).

{% hint style="warning" %}
**DAQ-U in DAQ-M za kalibracijo potrebujeta dostop do oblaka.**Noben od modelov ne shranjuje ničesar na napravi: njuni tovarniški kalibracijski paketi se nahajajo v oblaku MAPIR in se pridobijo na podlagi identifikacijske številke senzorja (nato se shranijo v lokalni predpomnilnik). Chloros potrebuje internetno povezavo za prenos kalibriranih podatkov W/m²/nm iz DAQ-U ali DAQ-M.**DAQ-E je izjema** – kalibracijo hrani na sami napravi.

<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

{% endhint %}***

## Kam se shranjujejo posnetki

| Površina | Privzeta destinacija za `.daq` |
| --- | --- |
| GUI — zavihek Svetlobni senzorji | `<project folder>/light_sensor/` (končani zapisi se samodejno dodajo v projekt) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` na računalniku, na katerem teče strežniški del |

Vsako ime datoteke `.daq` vsebuje ID senzorja in časovni žig.

***

## V tem poglavju

* [**Zavihek DAQ v Chloros**](gui.md) — popoln vodnik po grafičnem vmesniku: priključitev vsakega modela, nastavitve za posamezne senzorje, spektralni grafikoni, kolorimetrični podatki v realnem času, odbojnost z dvema senzorjema in snemanje.
* [**CLI Hitri začetek (pool-\*)**](cli-quick-start.md) — upravljanje senzorjev DAQ iz `chloros-cli daq pool-*`, podprta pot ukazne vrstice.
* [**Profil omejitev in kalibrirano spektralno območje**](caps-and-range.md) — katere omejitve obstajajo za posamezne modele, kako jih deklarirati ter podrobnosti o kalibriranem spektralnem območju.
* [**Snemanje in format .daq**](recording.md) — format SQLite `.daq` in poteki snemanja.
* [**Mrežno povezovanje DAQ-E in sinhronizacija časa**](ethernet-ptp.md) — načini prenosa DAQ-E in sinhronizacija časa PTP.
* [**Delovni postopki za odbojnost**](reflectance.md) — uporaba podatkov o navzdolnjem sevanju iz DAQ za izračun odbojnosti.
* Za popolno dokumentacijo na ravni zastavic glejte [CLI Referenca](../reference/cli-reference.md) (poglavje `chloros-cli daq`) in [referenco SDK](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), ki sta napisani tako, da ju lahko neposredno uporabljajo AI-pomočniki.

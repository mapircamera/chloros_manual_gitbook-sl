---
metaLinks: {}
---

# Uvod

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros

je programska aplikacija podjetja [MAPIR

](https://www.mapir.camera) za obdelavo multispektralnih posnetkov, nadzor v realnem času strojne opremeMAPIR

in beleženje podatkov senzorjev.Chloros

1.2.0 podpira celotno družino izdelkovMAPIR

:

* **kamereSurvey3** — obdelujejo posnetke v formatu RAW+JPG v kalibrirane karte odbojnosti in indeksov vegetacije. Glejte [Podprte kamere](supported-cameras.md).
* **kamere LATTICE** — v živo priključite GigE-module za multispektralne kamere, posamično ali kot sinhronizirane nize več kamer: predogled, zajem in obdelava v kalibrirane izdelke sevanja in odbojnosti. Glejte [poglavje o LATTICE](lattice/README.md).
* **Svetlobni senzorji DAQ** — spektralni senzorji DAQ-U (USB), DAQ-M (Bluetooth) in DAQ-E (Ethernet): kalibrirani spektri v realnem času, posnetki `.daq` in osvetlitev od zgoraj za obdelavo odbojnosti. Glejte [poglavje o DAQ](daq/README.md).

{% hint style="success" %}
**Novosti vChloros

1.2.0**: nadzor kamere in matrike LATTICE v realnem času, integracija svetlobnih senzorjev DAQ, načini zajemanja in snemalniki, celoten radiometrični obdelovalni proces LATTICE, avtomatizacija projektov prekCLI

/SDK

ter še veliko več. Oglejte si seznam novosti spodaj in [Prenesite](download.md) za seznam sprememb.
{% endhint %}

{% hint style="info" %}
**UporabljateChloros

z umetno inteligenco?** Ta priročnik je namenjen prav temu. Usmerite svojega pomočnika na:

* `https://mapir.gitbook.io/chloros/llms.txt` — strojno berljiv indeks vsake strani.
* Katera koli stran v surovem formatu Markdown — k njenemu naslovuURL

dodajte `.md` (npr. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI

Referenca](reference/cli-reference.md) in [SDK

Referenca](reference/sdk-reference.md) — popolne referenčne strani z natančnimi vrednostmi, napisane za uporabo v modelih LLM.

Primer navodila: *»Preberi https://mapir.gitbook.io/chloros/reference/cli-reference.md,, nato napiši skript, ki se prijavi in obdela mapo ~/flights/flight_001 v GeoTIFF-datoteke z odbojnostjo +NDVI

.«*

Celoten vodnik: [UporabaChloros

z AI-pomočniki](ai-assistants.md).
{% endhint %}

***

## Novosti v programuChloros

1.2.0

* **Upravljanje kamere v živo — nov zavihek »Kamere«.** Povežite kamere LATTICE posamično ali kot sinhronizirane skupine več kamer (časovna sinhronizacija PTP, zajem, sprožen s strojno opremo), s prekrivnimi sliki v živo, histogrami po pasovih, pametno samodejno osvetlitvijo, kalkulatorjem indeksa v živo in posodobitvami programske opreme kamere znotraj aplikacije.
* **Svetlobni senzorji — nov zavihek »Light Sensors«.** Priključite senzorje DAQ-U (USB), DAQ-M (Bluetooth) in DAQ-E (Ethernet); si oglejte kalibrirane spektre v živo (W/m²/nm), shranite datoteke `.daq` v svoj projekt, izberite profile korekcije kapice in posodobite firmware DAQ-E prek omrežja.
* **Načini zajemanja in snemalniki.** Enkratno / neprekinjeno / intervalno zajemanje ter način »Fastest Capture« (najhitrejše zajemanje) izključno za surove podatke; izbira po posameznem projektu, katere kamere in vrste izvoza ustvari »Capture All«; matrični snemalniki za indeksne videoposnetke za spremljanje in surove serije posnetkov za analizo z izdelavo videoposnetkov v načinu brez povezave.
* **Obdelovalni tok LATTICE.** Uvozite mape zajemov LATTICE in vsak surovi okvir razčlenite v izdelke brez bayersovega filtra, predogled, sevanje float32 (W/m²/sr/nm) in odbojnost s stikali za vsak izdelek posebej. Odbojnost lahko izhaja iz kalibracijskega cilja znotraj okvirja ali iz navzdol usmerjenega DAQ-signala; pri izvozih se uporabi poravnava matrike; manjkajoča tovarniška kalibracija se samodejno prenese na podlagi serijske številke kamere.
* **Projekti si zapomnijo strojno opremo.** Priključene kamere in svetlobni senzorji se shranijo skupaj s projektom (`cameras.json` / `sensors.json`) in se ob ponovnem odprtju projekta ponovno povežejo s shranjenimi nastavitvami. Glejte [GUI: Projekti](projects.md).
* **Izboljšave v pregledovalniku slik.** Odčitavanje pikslov/indeksa kurzorja s pravilnim prilagajanjem odbojnosti za vsako datoteko, histogrami plasti, drsnik za združevanje GSD, načini mreže »na sprožilec« / »na kamero«, pogledi izdelkov LATTICE ter izvoz indeksa/LUT v peskovnik na disk.
* **Znatno razširjene funkcije »CLI

« in »SDK

«.** Nove družine ukazov `lattice`, `daq pool-*`, `project` in `time-sync`; nove možnosti `process` (`--input-level`, stikala za posamezne izdelke, `--reflectance-source`, zastavice za poravnavo niza);SDK

ročaji za pametno povezovanje (`connect_camera` / `connect_array` / `connect_daq_sensor`), ki samodejno zaženejo ozadje; avtomatizacija `open_project()`; kolo »SDK

« je vključeno v namestitvene programe in objavljeno na PyPI kot `chloros-sdk`.
* **Pojasnjena semantika napak.** Izvedba `chloros-cli process`, ki je zahtevala izdelke, a ni zapisala nobenega, zdaj jasno poroča o napaki in se zaključi z izhodno vrednostjo, ki ni nič; uspešne izvedbe poročajo, koliko slikovnih izdelkov so zapisale.
* **Nova postavitev izhodnih podatkov.** Proizvodi se shranijo v mape `<project>/<camera>/<format>/<Product>_Images/` in ohranijo ime izvorne datoteke — mape, ne končnica imena datoteke, identificira proizvod. Glej [Formati izhodnih slik](output-image-formats.md).
* **Več vhodnih podatkov, načrtov in jezikov.** Podpora za vhodne podatke `.dng`; vseh 38 jezikov vmesnika je v celoti na voljo; omejitve strojne opreme po načrtu z brezplačno (brez prijave) uporabo do 4 kamer in 2 svetlobnih senzorjev.
* **Zanesljivost.** Funkcija »Stop Processing« se pravilno zaključi s podrobnim povzetkom izvedbe, projekti z več kamerami izvozijo podatke vseh kamer, nadgradnje namestitvenega programa pa vas ne odjavijo več.***

Chloros

je na voljo v 3 različicah:

##Chloros

: namizna aplikacija z grafičnim uporabniškim vmesnikom

Samostojno okno z vsemi funkcijami, vključno z zavihkoma »Kamere v živo« in »Svetlobni senzorji«. _Samo za Windows._

## [Chloros

CLI

: vmesnik ukazne vrstice](CLI.md)

Obdelava v paketih prek ukazne vrstice ter ukazi v realnem času `lattice`, `daq pool-*`, `project` in `time-sync`. Idealno za avtomatizacijo, pisanje skript in delovanje brez grafičnega vmesnika. Na voljo na **Windows

,Linux

amd64 inLinux

arm64 (NVIDIA Jetson)**. _Za dostop do CLI je potrebna plačljiva stopnjaChloros

+._

## [Chloros

API

:Python

SDK

](api-python-sdk.md)

Programski vmesnikPython

za avtomatizacijo in prilagojene delovne tokove: obdelava celotnega procesa, sej v živo s kamero/nizom, sej z DAQ-senzorji in avtomatizacija shranjenih projektov. Namesti se s paketom desktop/CLI

, objavljen pa je tudi kot `pip install chloros-sdk`. _Za dostop do API-ja je potrebna plačljiva stopnjaChloros

+._

***

## Podprte platforme

| Platforma | GUI |CLI

|Python

SDK

|
| --- | --- | --- | --- |
| **Windows

10/11 (x64)** | Da | Da | Da |
| **Linux

amd64 (x86_64)** | Ne | Da | Da |
| **Linux

arm64 (NVIDIA Jetson)** | Ne | Da | Da |

Navodila za namestitev v sistemuLinux

najdete v razdelku [Linux

in robno računalništvo](linux/linux-overview.md).

***

## Začnite v treh korakih

1. **Namestitev** — prenesite in zaženite namestitveni program za svojo platformo. Glejte [Prenos](download.md).
2. **Prijava (neobvezno za GUI)** — GUI brezplačno obdeluje slike tudi brez računa. S [Chloros

+ prijavo](chloros+-login.md) odklenete vzporedno obdelavo, pospeševanje z GPU, višje omejitve naprav ter dostop doCLI

/SDK

.
3. **Ustvarite svoj prvi projekt** — odpriteChloros

, ustvarite [nov projekt](projects.md), [dodajte svoje slike](processing-images-gui/adding-files-to-a-project.md) in [začnite z obdelavo](processing-images-gui/starting-the-processing.md). Če želite namesto tega upravljati z dejansko strojno opremo, odprite zavihek »Kamere« ali »Svetlobni senzorji« — glejte [GUI: Navigacija](navigation.md).

***

##Chloros

+

Čeprav je program »Chloros

« za večino nalog brezplačen, boste morda ugotovili, da potrebujete več. V tem primeru vam lahko koristi plačljiva licenca za »Chloros

+«. Z licenco »Chloros

+« lahko odklenete nove funkcije, kot so:

* **Večnitna obdelava**: znatno pospešite obdelavo slik za večje projekte s hkratno obdelavo slik skozi obdelovalno verigo.
* **Pospeševanje z grafičnim procesorjem (CUDA)**: izkoristite današnje možnosti večjega pomnilnika grafičnega procesorja za dodatno pospešitev procesne verige obdelave slik. Za najboljše rezultate priporočamo 4 GB ali več pomnilnika VRAM.
* **Dostop doChloros

+**[**CLI**](CLI.md): z ukazom »Chloros

« iz ukazne vrstice lahko avtomatizirate in integrirate program v svojo lastno programsko opremo. Na voljo na vseh plačljivih ravneh; uveljavljeno na strežniški strani.
* **Chloros

+**[**API**](api-python-sdk.md) **Dostop:** za programsko upravljanje zaženiteChloros

+ izPython

, kar omogoča nemoteno integracijo z vašimi raziskovalnimi postopki, delovnimi tokovi za analizo podatkov in prilagojenimi aplikacijami. Na voljo na vseh plačljivih ravneh; izvršuje se na strežniški strani.
* **Višje omejitve strojne opreme**: hkrati priključite več kamer in svetlobnih senzorjev. Brez prijave grafični uporabniški vmesnik (GUI) omogoča priključitev do 4 kamer in 2 svetlobnih senzorjev DAQ; plačljivi paketi povečajo obe omejitvi:

| Paket | Kamere | Svetlobni senzorji DAQ |
| --- | --- | --- |
| Iron (brezplačen, brez prijave) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Uporaba več naprav**: vsaka licencaChloros

+ omogoča registracijo 2 ali več naprav. Za upravljanje registriranih naprav uporabite svoj račun v oblakuMAPIR

. Podporo za več naprav lahko dodate z nadgradnjo licenceChloros

+.
* **Napredna metoda debayeringa, ki upošteva teksturo:** visokokakovosten debayering, ki upošteva robove, v kombinaciji z modelom za odstranjevanje šuma na podlagi umetne inteligence (AI) in strojnega učenja (ML), ki odstrani skoraj ves šum pri debayeringu.
* **Prilagojene formule za multispektralne indekse:** vnesite prilagojene multispektralne indekse v rastrske kalkulatorjeChloros

, tako za obdelavo kot za okolje za ogled slik.
* **OblakLinux

in robno računalništvo (Edge Computing):** zagnajte storitev »Chloros

« na platformahLinux

x86\_64 in ARM64, vključno z NVIDIA Jetson, za obdelavo na terenu in na robu omrežja. Oglejte si [Pregled storitveLinux

](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Cene in prijava</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.2.0 build so the banner prints "Chloros CLI 1.2.0". -->

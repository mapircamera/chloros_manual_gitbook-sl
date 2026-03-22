---
metaLinks: {}
---

# Uvod

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros je programska aplikacija podjetja [MAPIR](https://www.mapir.camera) za obdelavo slik in drugih podatkov senzorjev.

***{% hint style="success" %}**Novosti v Chloros 1.1.0**: Podpora za Linux (amd64 in arm64), NVIDIA Jetson edge computing, Dynamic Compute Adaptation, 4-nitni procesni potok, nove CLI ukazi in možnosti. Za celoten seznam sprememb glej [Prenos](download.md).
{% endhint %}

Chloros je na voljo v 3 načinih uporabe:

## Chloros: Namizna aplikacija GUI

Samostojno ločeno okno z vsemi funkcijami. _Samo za Windows._

## [Chloros CLI: Vmesnik ukazne vrstice](CLI.md)

Paketna obdelava v ukazni vrstici. Idealno za avtomatizacijo, pisanje skriptov in delovanje brez grafičnega vmesnika. Na voljo na **Windows, Linux amd64 in Linux arm64 (NVIDIA Jetson)**. _Za dostop do CLI je potrebna licenca Chloros+._

## [Chloros API: Python SDK](api-python-sdk.md)

Programski vmesnik Python za avtomatizacijo in prilagojene delovne tokove. Idealen za raziskovalne poti, integracijo z obstoječimi aplikacijami Python in razvoj prilagojenih orodij. Na voljo na **vseh platformah** prek `pip install chloros-sdk`. _Za dostop do API je potrebna licenca Chloros+._***

## Podprte platforme

| Platforma | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11** | Da | Da | Da |
| **Linux amd64 (x86_64)** | Ne | Da | Da |
| **Linux arm64 (NVIDIA Jetson)** | Ne | Da | Da |

Za navodila za namestitev Linux glejte poglavje [Linux &amp; Edge Computing](linux/linux-overview.md).

***

## Chloros+

Čeprav je Chloros za večino nalog brezplačen, boste morda ugotovili, da potrebujete več. V tem primeru vam lahko koristi plačljiva licenca za Chloros+. Z licenco Chloros+ lahko odklenete nove funkcije, kot so:

* **Večnitno obdelovanje**: znatno pospešite obdelavo slik za večje projekte s hkratno obdelavo slik skozi procesno verigo.
* **Pospešitev GPU (CUDA)**: izkoristite današnje možnosti večjega pomnilnika GPU za dodatno pospešitev procesne verige obdelave slik. Za najboljše rezultate priporočamo 4 GB ali več VRAM-a.
* **Chloros+**[**CLI**](CLI.md)**Dostop**: zaženite Chloros+ iz ukazne vrstice, da ga avtomatizirate in integrirate v svojo lastno programsko opremo.
* **Chloros+**[**API**](api-python-sdk.md)**Dostop:** za programsko upravljanje zaženite Chloros+ iz Python, kar omogoča nemoteno integracijo z vašimi raziskovalnimi potmi, delovnimi tokovi za analizo podatkov in prilagojenimi aplikacijami.
* **Uporaba na več napravah**: vsaka licenca Chloros+ omogoča registracijo 2 ali več naprav. Za upravljanje registriranih naprav uporabite svoj račun MAPIR Cloud. Dodajte podporo za več naprav z nadgradnjo licence Chloros+.
* **Napredna metoda debayeringa, ki upošteva teksturo:** visokokakovosten debayering, ki upošteva robove, v kombinaciji z modelom za odstranjevanje šuma AI/ML, ki odstrani skoraj ves šum debayeringa. 
* **Prilagojene formule za multispektralne indekse:** vnesite prilagojene multispektralne indekse v rastrske kalkulatorje Chloros, tako za obdelavo kot za peskovnik za pregledovanje slik.
* **Linux in robno računalništvo:** zaženite Chloros na platformah Linux x86\_64 in ARM64, vključno z NVIDIA Jetson, za obdelavo na terenu in na robu. Glejte [Pregled Linux](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Cene in prijava</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>

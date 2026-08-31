---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibracijski tarči

MAPIR ponuja različne kalibracijske tarče za široko paleto uporab. Kompaktna tarča T4-R50, prikazana spodaj, vsebuje 4 plošče, katerih svetlobna odbojnost je bila izmerjena v območju od 250 do 2.500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Difuzne referenčne tarče T4 imajo naslednje krivulje odbojnosti, [podatke lahko prenesete tukaj](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR Odbojnost T4 :: 250–2.500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR Odbojnost T4 :: 400–1.000 nm</p></figcaption></figure>Referenčni cilji T4P z difuznim odbojem imajo naslednje krivulje odbojnosti, [podatke lahko prenesete tukaj](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P odbojnost :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR Odbojnost T4P :: 400–1000 nm</p></figcaption></figure>Če si ogledate graf odbojnosti, lahko vidite, da so vrednosti prikazane kot valovna dolžina (os x) v odnosu do odstotka odbojnosti (os y). Ko zajamemo sliko kalibracijskega cilja, vzpostavimo povezavo med vrednostjo pikslov in odstotkom odbojnosti znotraj spektra, na katerega je občutljiv vsak od senzorskih pasov kamere.

To pomeni, da lahko pri vsaki sliki, ki jo posnamete z našimi kamerami, uporabite fotografijo naših ciljev odbojnosti, kot sta [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) ali [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), za kalibracijo slik glede na odbojnost. Po kalibraciji je vsaka slikovna točka na sliki enaka odstotku odbojnosti.

Za **Survey3** izhodih, če kalibrirane slike v formatu Chloros izvozite kot običajne JPG ali TIFF, se odstotek odbojnosti izračuna tako, da vrednost piksla delite z bitno globino slikovnega formata. Torej za JPG delite z 255, za TIFF pa z 65.535. Lahko izberete tudi izhod v formatu PERCENT v datoteki Chloros, pri čemer bo vrednost vsakega piksla v območju od 0,0 do 1,0 (od 0 % do 100 % odbojnosti). Upoštevajte, da nekatere programske aplikacije za obdelavo slik ne podpirajo slik v odstotkih (s plavajočo vejico), poleg tega pa so te slike glede na velikost pomnilnika zelo obsežne.

{% hint style="info" %}
**Odbojnost LATTICE uporablja drugačno lestvico pikslov.** Odbojnost LATTICE se shranjuje z DN 32768 = 100 % odbojnosti (ne 65535), vsaka datoteka pa vsebuje oznako XMP `Chloros:PixelScale`, ki navaja njeno lestvico. Preberite oznako in delite z njo, namesto da predpostavljate konstantno vrednost — glejte [Izhodne slike](output-image-formats.md).
{% endhint %}

## Kalibracijske tarče s kamerami LATTICE

Pri kamerah LATTICE je kalibracijski cilj za odbojnost **neobvezen**: Chloros lahko namesto tega odbojnost nanaša na navzdolno sevanje, izmerjeno z DAQ-senzorjem svetlobe (ρ = π·L/E). Referenca se izbere z nastavitvijo vira odbojnosti (Nastavitve projekta v grafičnem vmesniku; `--reflectance-source` v CLI; `reflectance_source` v SDK):

| Vrednost | Obnašanje |
| --- | --- |
| `auto` *(privzeto)* | Cilj v okviru, ki je uspešno prestal kontrolo kakovosti (QA), je **absolutna referenca**; če cilj ni prisoten ali če kontrola kakovosti ni uspešna, se Chloros vrne na delitveno vrednost navzdolnega toka DAQ. |
| `target` | Izključno cilj – brez nadomestitve s strani DAQ. |
| `daq` | DAQ ima prednost – meritev navzdolnega toka je vedno referenca. |

Dodatno obnašanje ciljev za LATTICE:

* **Geometrije ciljev** — podprti so paneli, označeni z ArUco, paneli s fiksnim ROI in trakovni cilji; geometrija izhaja iz konfiguracije ciljev projekta.
* **Podatki o ciljih, izmerjeni na enoto** — `--target-reflectance-dir DIR` kaže na imenik skenov odbojnosti ciljev, izmerjenih na enoto (`<serial>.csv`, iskanih po serijski številki/QR enote cilja). V primeru neuspeha se Chloros zateče k nominalnim spektrom T3/T4P.
* **Časovno sidranje** — zaznan cilj kalibrira okvirje okoli sebe in se ohrani med opazovanji cilja.

Celotna semantika zastavic in primeri so v [CLI Referenca](reference/cli-reference.md) (glej »Stikala za izvoz po posameznih izdelkih«).

### F988

»Odbojnost F988 se kalibrira z uporabo plošče za merjenje odbojnosti v prizoru: pas leži zunaj kalibriranega območja svetlobnega senzorja DAQ, zato Chloros uporabi vaš najnovejši posnetek plošče in ga ohrani med opazovanji plošče.«

Če se F988 izvaja s kalibracijo, ki temelji izključno na DAQ, Chloros zavrne odbojnost za ta pas, ki temelji na DAQ, in navede razlog (razlog za preskok `dls-uncalibrated-band-988`); podprt je potek dela s ploščo.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>

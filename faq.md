---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Pogosta vprašanja

<details>

<summary>Ali lahko z napravo Chloros obdelujem slike iz kamer, ki niso znamke MAPIR?</summary>

Ne, program Chloros podpira le obdelavo slik iz kamer MAPIR — družin Survey3 in LATTICE. Za več informacij si oglejte seznam [podprtih modelov kamer](supported-cameras.md). Obdelavo posnetkov drugih kamer ponujamo v oblaku MAPIR, celoten seznam si oglejte [tukaj](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Ali Chloros podpira kamere LATTICE?</summary>

Da. Chloros 1.2.0 podpira kamere LATTICE M3C in M3M od začetka do konca: **nadzor v živo**— odkrivanje, povezovanje, predogled in zajem prek zavihka »Kamere« v grafičnem uporabniškem vmesniku, `chloros-cli lattice` ali Python SDK, vključno s sinhroniziranimi nizi več kamer s časovno sinhronizacijo PTP — ter**popolno radiometrično obdelavo** posnetkov (surovi podatki → odstranitev bayera → sevanje → odbojnost → indeks). Glejte [Podprte kamere](supported-cameras.md) in [LATTICE vodnik](lattice/README.md).

</details>

<details>

<summary>Ali lahko kalibriram svoje slike za odbojnost brez kalibracijskega cilja?</summary>

**Survey3:** Ne. Brez slike kalibracijskega cilja, posnete ob istem času kot slike brez cilja, ne boste mogli povezati vrednosti pikslov slike z znanim odstotkom odbojnosti. Če ne vključite tudi dnevnika iz svetlobnega senzorja MAPIR, se spekter okoljske svetlobe ne bo izmeril, rezultati odbojnosti pa ne bodo natančni.**LATTICE:** Da. Odbojnost se lahko nanaša na navzdol usmerjeno sevanje, izmerjeno s svetlobnim senzorjem DAQ namesto s panelom (ρ = π·L/E). Ko je v okvirju prisoten cilj, ki izpolnjuje zahteve za kakovost (QA), ta postane privzeto absolutna referenca (`--reflectance-source auto`). Ena izjema: »Odbojnost F988 se kalibrira z uporabo plošče za odbojnost v prizoru: pas leži zunaj kalibriranega območja svetlobnega senzorja DAQ, zato Chloros uporabi vaš najnovejši posnetek plošče in ga ohrani med posameznimi opazovanji plošče.« Glejte [Kalibracijske tarče](calibration-targets.md).

</details>

<details>

<summary>Ali potrebujem svetlobni senzor DAQ?</summary>

Ne za sevanje: izvozi sevanja LATTICE izhajajo iz tovarniške radiometrične kalibracije vsake kamere in ne potrebujejo niti senzorja DAQ niti cilja. Za **odbojnost**potrebujete referenco za okolno svetlobo – bodisi meritev navzdol usmerjenega sevanja senzorja svetlobe DAQ bodisi kalibracijski cilj znotraj kadra. Senzor DAQ vam omogoča izračun kalibrirane odbojnosti**brez namestitve kakršnih koli plošč v prizor**. Posnete datoteke `.daq` se samodejno uskladijo z vašimi posnetki na podlagi časovnega žiga. Glejte [kalibracijske tarče](calibration-targets.md) in [referenco CLI](reference/cli-reference.md).

</details>

<details>

<summary>Ali lahko Chloros uporabljam z umetno inteligenco (Claude, ChatGPT itd.)?</summary>

Da — ta priročnik in datoteki CLI/SDK sta zasnovani prav za to:

* Celoten kazalo priročnika je na voljo na strani `https://mapir.gitbook.io/chloros/llms.txt`, da lahko AI-pomočniki najdejo vsako stran.
* Surovi markdown vsake strani je na voljo na strani z malimi črkami URL, ki ji je dodan `.md` (na primer `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI Referenca](reference/cli-reference.md) in [sklicu SDK](reference/sdk-reference.md) sta napisana za uporabo v modelih LLM: natančne oznake, privzete vrednosti, semantika izhoda in ukazi, ki jih je mogoče kopirati in prilepiti.

V [AI Assistants](ai-assistants.md) si oglejte, kako svojega pomočnika usmeriti na Chloros.

</details>

<details>

<summary>Kam se shranijo moje obdelane izhodne datoteke?</summary>

Izdelki se shranijo v projektno mapo, razvrščeni po kameri in nato po datotečnem formatu:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **mapa-kamere** — `LATT-<sensor>-<lens>-F<filter>` za LATTICE, `<model>_<filter>` (npr. `Survey3N_RGN`) za Survey3
* **mapa po formatu** — `tiff16`, `tiff8`, `png8`, `jpg8` ali `tiff32`
* **mape izdelkov** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (vedno pod `tiff32`), `<INDEX>_Index_Images/`**Izvožene datoteke ohranijo ime izvorne datoteke — izdelek se prepozna po mapi, ne po končnici imena datoteke.**Pri CLI se projektna mapa ustvari poleg vhodne mape, razen če prenesete `-o`. Upoštevajte, da izvedba `chloros-cli process`, ki je zahtevala izdelke, a ni zapisala nobenega, izpiše `Processing finished but wrote no image products.` in**se zaključi z vrednostjo, ki ni nič**, tako da jo skripti lahko zaznajo. Glejte [Formati izhodnih slik](output-image-formats.md) in [Referenca za CLI](reference/cli-reference.md).

</details>

<details>

<summary>Ali lahko slike uredim pred obdelavo v Chloros?</summary>

Ne. Chloros predpostavlja, da vhodni podatki niso bili spremenjeni. Ne spreminjajte imen datotek.

</details>

<details>

<summary>Ali lahko kamere MAPIR in Survey3 nastavim na samodejno osvetlitev in slike obdelam v Chloros?</summary>

Ne. Nizi slik Survey3 morajo imeti fiksno/zaklenjeno osvetlitev, torej brez samodejne hitrosti zaklopa ali samodejne ISO. Vse slike istega modela kamere morajo imeti enako hitrost zaklopa in ISO (osvetlitev).

Kamere LATTICE nimajo te omejitve: Chloros samodejno prilagaja osvetlitev v realnem času (Smart AE), pri vsakem zajemu pa se zabeležita dejansko uporabljena osvetlitev in ojačanje, kar upošteva radiometrični proces.

</details>

<details>

<summary>Ali lahko Chloros obdeluje ali analizira ortomozaične slike?</summary>

Ne. Podprte so le posamezne slike kamere MAPIR, ne pa sestavljenih slik, kot je ortomozaična karta.

</details>

<details>

<summary>Kako lahko pospešim korak zaznavanja ciljev v programu Chloros?</summary>

Če v tabeli brskalnika datotek v desnem stolpcu vnaprej izberete ciljne slike, bo program Chloros iskal kalibracijske cilje le v teh slikah, kar bo znatno pospešilo obdelavo.

</details>

<details>

<summary>Če bom svoje slike naložil v <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">oblak MAPIR Cloud,</a> ali jih moram pred nalaganjem obdelati v programu Chloros?</summary>

Če nameravate slike naložiti na našo spletno platformo za obdelavo [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), jih pred nalaganjem ne urejajte. Cloud bo opravil enako obdelavo in še več.

</details>

<details>

<summary>Ali bo MAPIR kdaj podpiral funkcijo X? Res si želim, da bi MAPIR ponujal X.</summary>

Vedno nas zanima vaše mnenje o naših izdelkih. Če opazite kakšno težavo z našimi izdelki ali imate predlog, kako bi jih lahko izboljšali, nas prosimo [KONTAKTIRAJTE](https://www.mapir.camera/community/contact), da nam posredujete svoje mnenje. Večina našega raziskovalno-razvojnega dela temelji na upoštevanju najpomembnejših potreb naših strank.

</details>

<details>

<summary>Ali je Chloros na voljo za Linux?</summary>

Da! Chloros 1.2.0 podpira Linux amd64 (x86_64) in arm64 (NVIDIA Jetson JetPack 6) prek paketov `.deb`. CLI in Python SDK sta v celoti podprta na Linux, vključno z nadzorom kamere LATTICE v realnem času in senzorja DAQ. Za Linux ni grafičnega uporabniškega vmesnika — vsa interakcija poteka prek [CLI](CLI.md) ali [Python SDK](api-python-sdk.md). Za podrobnosti glejte [Linux Pregled](linux/linux-overview.md).

</details>

<details>

<summary>Ali lahko Chloros zaženem na platformi NVIDIA Jetson?</summary>

Da! Chloros podpira platforme NVIDIA Jetson, vključno z Jetson Nano, Orin Nano, Orin NX in AGX Orin, na katerih teče JetPack 6. Chloros samodejno prepozna vaš model Jetson in optimizira strategijo obdelave. Za navodila za nastavitev in namestitev si oglejte [Vodnik za NVIDIA Jetson](linux/nvidia-jetson-guide.md).

</details>

<details>

<summary>Ali se Chloros samodejno optimizira za mojo strojno opremo?</summary>

Da! Chloros vključuje [dinamično prilagajanje računalniških zmogljivosti](processing-architecture/dynamic-compute-adaptation.md), ki samodejno zazna vaš CPU, GPU, RAM in (na Jetsonu) toplotne senzorje. Nato izbere optimalno strategijo obdelave — od `GPU_PARALLEL` na sistemih z veliko pomnilnika do `GPU_SINGLE` na napravah z omejenimi zmogljivostmi ter `CPU_PARALLEL` na sistemih brez grafičnega procesorja NVIDIA. Ročna konfiguracija ni potrebna.

</details>

<details>

<summary>Kaj je 4-nitni procesni potok?</summary>

Chloros uporablja 4-nitno cevno arhitekturo za uporabnike Chloros+: Nit 1 (zaznavanje) naloži slike in zazna kalibracijske cilje, nit 2 (kalibracija) izračuna kalibracijo odbojnosti, nit 3 (obdelava) izvede GPU-pospešeno odstranjevanje bayerskih vzorcev in izračun indeksa, nit 4 (izvoz) pa zapiše izhodne datoteke. Za maksimalno prepustnost lahko več slik hkrati teče v različnih niti. Podrobnosti si oglejte v [Obdelovalni potek](processing-architecture/processing-pipeline.md).

</details>

<details>

<summary>Kako izvedem diagnostiko na svoji namestitvi Chloros?</summary>

Uporabite ukaz `selftest` za izvedbo 7-stopenjskega preizkusa delovanja: različica, razpoložljivost vrat, zagon backenda, povezljivost API (`/api/test`), informacije o sistemu (`/api/system-info` — GPU/CUDA/PyTorch), prisotnost modela za odstranjevanje šuma ter pripravljenost CUDA in odstranjevalca šuma:

```bash
chloros-cli selftest
```

To je še posebej koristno na sistemih Linux/Jetson za preverjanje nastavitev grafičnega procesorja in CUDA.

</details>

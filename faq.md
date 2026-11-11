---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Pogosta vprašanja

<details>

<summary>Ali lahko z Chlorosom obdelujem slike iz kamer, ki niso znamke MAPIR?</summary>

Ne, Chloros podpira le obdelavo slik iz kamer MAPIR — družin Survey3 in LATTICE. Za več informacij si oglejte seznam [podprtih modelov kamer](supported-cameras.md). Obdelavo slik iz drugih kamer ponujamo v storitvi MAPIR Cloud, celoten seznam pa si oglejte [tukaj](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Ali program Chloros podpira kamere LATTICE?</summary>

Da. Chloros 1.3.0 v celoti podpira kamere LATTICE M3C in M3M: **nadzor v realnem času**— odkrivanje, povezovanje, predogled in zajem prek zavihka »Kamere« v grafičnem uporabniškem vmesniku (GUI), `chloros-cli lattice`, ali prek Python SDK, vključno s sinhroniziranimi nizi več kamer s časovno sinhronizacijo PTP — ter**popolno radiometrično obdelavo** posnetkov (surovi podatki → odstranjevanje bayera → sevanje → odbojnost → indeks). Glejte [Podprte kamere](supported-cameras.md) in [vodnik za LATTICE](lattice/README.md).

</details>

<details>

<summary>Ali lahko kalibriram svoje slike za odbojnost brez kalibracijskega cilja?</summary>

**Survey3

:** Ne. Brez slike kalibracijskega cilja, posnete ob istem času kot slike brez cilja, ne boste mogli povezati vrednosti pikslov slike z znanim odstotkom odbojnosti. Če ne vključite tudi dnevnika svetlobnega senzorja MAPIR

, se spekter okoljske svetlobe ne bo izmeril, rezultati odbojnosti pa ne bodo natančni.

**LATTICE:** Da. Odbojnost se lahko nanaša na navzdolno sevanje, izmerjeno s svetlobnim senzorjem DAQ namesto s panelom (ρ = π·L/E). Ko je v okviru prisoten cilj, ki je uspešno prestal kontrolo kakovosti (QA), ta postane privzeto absolutna referenca (`--reflectance-source auto`), razen če je bil sprejet z oznako, v tem primeru pa se cilj in verige DAQ merijo med seboj glede na izmerjeno oceno. Ena izjema: »Odbojnost F988 se kalibrira z uporabo plošče za odbojnost v prizoru: pas leži zunaj kalibriranega območja svetlobnega senzorja DAQ, zato Chloros

uporabi vaš najnovejši posnetek plošče in ga ohrani med posameznimi opazovanji plošče.« Glejte [Kalibracijske tarče](calibration-targets.md).

</details>

<details>

<summary>Ali potrebujem svetlobni senzor DAQ?</summary>

Ne za sevanje: izvozi sevanja LATTICE izhajajo iz tovarniške radiometrične kalibracije vsake kamere in ne potrebujejo niti senzorja DAQ niti cilja. Za **odbojnost**potrebujete referenco za okolno svetlobo – bodisi meritev navzdol usmerjenega svetlobnega toka senzorja DAQ bodisi kalibracijski cilj v kadru. S senzorjem DAQ lahko pridobite kalibrirano odbojnost**brez namestitve kakršnih koli plošč v prizor**. Posnete datoteke `.daq` se samodejno uskladijo z vašimi slikami na podlagi časovnega žiga. Glejte [Kalibracijske tarče](calibration-targets.md) in [Referenco CLI

](reference/cli-reference.md).

</details>

<details>

<summary>Ali lahko uporabljam Chlorosa z AI-pomočnikom (Claude, ChatGPT itd.)?</summary>

Da — ta priročnik ter CLI in SDK sta zasnovana prav za to:

* Celoten kazalo priročnika je na voljo na naslovu `https://mapir.gitbook.io/chloros/llms.txt`, da lahko AI-pomočniki odkrijejo vsako stran.
* Surovi markdown vsake strani je na voljo na njenem URL-ju v malih črkah, ki mu je dodan `.md` (na primer `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [Referenca za CLI](reference/cli-reference.md) in [Referenca za SDK](reference/sdk-reference.md) sta napisani za uporabo v velikih jezikovnih modelih (LLM): natančni zastavki, privzete vrednosti, semantika izhodnih ukazov in ukazi, ki jih je mogoče kopirati in prilepiti.

V [AI Assistants](ai-assistants.md) si oglejte, kako svojega pomočnika usmeriti na Chloros.

</details>

<details>

<summary>Kam se shranijo moje obdelane izhodne datoteke?</summary>

Izdelki se shranijo v projektno mapo, razvrščeni po kameri in nato po datotečnem formatu:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **mapa-kamere** — `LATT-<sensor>-<lens>-F<filter>` za LATTICE, `<model>_<filter>` (npr. `Survey3N_RGN`) za Survey3
* **mapa-formata** — `tiff16`, `tiff8`, `png8`, `jpg8` ali `tiff32`
* **mape izdelkov** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (vedno pod `tiff32`), `<INDEX>_Index_Images/`
* **`clamp_masks/`** — samo za multispektralne podatke LATTICE: maske na ravni posameznih pik poleg map z izdelki, glej spodaj**Izvožene datoteke ohranijo ime izvorne datoteke — izdelek se prepozna po mapi, ne po končnici imena datoteke.**Pri uporabi CLI se projektna mapa ustvari v vaši privzeti projektni mapi (`Documents/MAPIR/Chloros Projects`, če je niste spremenili), razen če prenesete `-o`; relativna pot `-o` se upošteva glede na mapo, iz katere izvajate ukaz. Upoštevajte, da izvedba `chloros-cli process`, ki je zahtevala izdelke, a ni zapisala nobenega, izpiše `Processing finished but wrote no image products.` in**se zaključi z vrednostjo, ki ni nič**, tako da jo skripti lahko zaznajo. Glejte [Formati izhodnih slik](output-image-formats.md) in [Referenca CLI](reference/cli-reference.md).

</details>

<details>

<summary>Kaj so mape clamp_masks in datoteke *_clampmask.tif?</summary>

Novost v različici 1.3.0. Produkti sevanja in odbojnosti iz multispektralne kamere LATTICE M3C se obdelujejo z razmešavanjem pasov 3×3, in kadar se v pasu pojavi negativna vrednost, program Chlorosto vrednost omeji na nič. Produkt z vsaj enim odrezanim pikslom dobi enopasovno masko `uint8`, ki označuje, kateri piksli in pasovi so bili odrezani (bit 0 = R-kanal produkta, bit 1 = G, bit 2 = B). Maske se shranijo v mapo `clamp_masks/` **poleg** map izdelkov — na primer `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif` — XMP-oznaka izdelka `Chloros:UnmixClampMask` pa poimenuje njegovo masko (ali navaja `none`).

To niso slike prizora: Chlorosov lastni uvoz jih preskoči, in ker nikoli niso shranjene znotraj mape izdelka, lahko mapo izdelka predate programu Pix4D ali Metashape takšno, kot je. Glejte [maske Clamp](output-image-formats.md#clamp-masks).

</details>

<details>

<summary>Zakaj Chloros pravi: »Mapa projekta je za ta računalnik pregloboka«?</summary>

To velja le zWindows. V Windows so dolge poti privzeto izklopljene, zato nobena pot do datoteke ne sme biti daljša od 260 znakov. Chlorosovi izdelki se nahajajo do približno 130 znakov pod projektno mapo, zato Chloros pred obdelavo zavrne globljo projektno mapo – namesto da bi se program zažgal in ne zapisal ničesar, kot se je lahko zgodilo pred različico 1.3.0. Sporočilo navaja najdaljšo projektno mapo, ki ustreza.

To lahko odpravite na dva načina:

* Uporabite krajšo projektno mapo (**Delovni imenik** v nastavitvah projekta ali `-o` / `chloros-cli set-project-folder` za CLI).
* Vklopite dolge poti v sistemu Windows in ponovno zaženite Chloros. To stori možnost **Omogoči dolge poti v sistemu Windows** v namestitvenem programu Chloros (privzeto označena); ponovno zaženite namestitveni program, če ste jo odkljukali.

Glejte [Začetek obdelave](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

</details>

<details>

<summary>Ali lahko pred obdelavo v programu Chloros uredim svoje slike?</summary>

Ne. Program Chloros predpostavlja, da vhodni podatki niso bili spremenjeni. Ne spreminjajte imen datotek.

</details>

<details>

<summary>Ali lahko kamere MAPIRSurvey3 nastavim na samodejno osvetlitev in slike obdelam v programu Chloros?</summary>

Ne. Nizi slik Survey3 morajo imeti fiksno/zaklenjeno osvetlitev, torej brez samodejne hitrosti zaklopa ali samodejnega ISO. Vse slike istega modela kamere morajo imeti enako hitrost zaklopa in ISO (osvetlitev).

Kamere LATTICE te omejitve nimajo: Chlorosu v realnem času nadzira njihovo osvetlitev (Smart AE), vsaka posnetka pa zabeleži dejansko uporabljeno osvetlitev in ojačanje, kar upošteva radiometrični proces.

</details>

<details>

<summary>Ali lahko program Chlorosobdeluje ali analizira ortomozaične slike?</summary>

Ne. Podprte so le posamezne slike kamerMAPIR, ne pa sestavljenih slik, kot je ortomozaična karta.

</details>

<details>

<summary>Kako lahko pospešim korak zaznavanja ciljev v programChloros?</summary>

Če v tabeli brskalnika datotek v desnem stolpcu vnaprej izberete ciljne slike, bo to Chlorosu naročilo, naj kalibracijske cilje išče le v teh slikah, kar bo znatno pospešilo obdelavo.

</details>

<details>

<summary>Če bom svoje slike naložil v <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> ali jih moram pred nalaganjem obdelati v Chlorosu?</summary>

Če nameravate slike naložiti na našo spletno platformo za obdelavo [MAPIR

Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), jih pred nalaganjem ne urejajte. Cloud bo opravil enako obdelavo in še več.

</details>

<details>

<summary>Ali bo MAPIR kdaj podpiral funkcijo X? Res si želim, da bi MAPIR ponujal X.</summary>

Vedno nas zanimajo povratne informacije o naših izdelkih. Če opazite težavo z našimi izdelki ali imate predlog, kako jih lahko izboljšamo, nas prosimo [KONTAKTIRAJTE](https://www.mapir.camera/community/contact), da nam posredujete svoje mnenje. Večina našega raziskovalno-razvojnega dela temelji na upoštevanju najpomembnejših potreb naših strank.

</details>

<details>

<summary>Ali je Chloros na voljo za Linux?</summary>

Da! Chloros 1.3.0 podpira Linux amd64 (x86_64) in arm64 (NVIDIA Jetson JetPack 6) prek paketov `.deb`. CLI in Python SDK sta v celoti podprta v sistemu Linux, vključno z nadzorom kamere LATTICE v realnem času in senzorjev DAQ. Za sistem Linux ni grafičnega uporabniškega vmesnika (GUI) – vsa interakcija poteka prek [CLI](CLI.md) ali [Python SDK](api-python-sdk.md). Za podrobnosti glejte [Pregled Linuxa](linux/linux-overview.md).

</details>

<details>

<summary>Ali lahko zagnam sistem Chloros na platformi NVIDIA Jetson?</summary>

Da! Chloros podpira **družino** NVIDIA Jetson Orin — Orin Nano, Orin NX in AGX Orin — z nameščenim JetPack 6. Starejši modeli (Jetson Nano, TX2, Xavier NX) ne podpirajo JetPack 6 in jih trenutni paket ne podpira. Chloros samodejno zazna vaš model Jetson in optimizira strategijo obdelave. Paket za Jetson uporablja CUDA-različico PyTorcha za JetPack 6, zato obdelava poteka na grafičnem procesorju (GPU) naprave Jetson. Navodila za nastavitev in namestitev najdete v [Vodniku za NVIDIA Jetson](linux/nvidia-jetson-guide.md).

</details>

<details>

<summary>Ali se Chloros samodejno optimizira za mojo strojno opremo?</summary>

Da! Chloros vključuje [dinamično prilagajanje računalniške moči](processing-architecture/dynamic-compute-adaptation.md), ki samodejno zazna vaš CPU, GPU, RAM in (na Jetsonu) toplotne senzorje. Nato izbere optimalno strategijo obdelave — od `GPU_PARALLEL` za grafične procesorje z velikim pomnilnikom do `GPU_SINGLE` za manjše grafične procesorje ter `CPU_PARALLEL` za sisteme brez grafičnega procesorja CUDA, ki ga lahko uporablja PyTorch. Ročna konfiguracija ni potrebna.

</details>

<details>

<summary>Kaj je 4-nitni obdelovalni potok?</summary>

Chloros
uporablja 4-nitno arhitekturo s potekom obdelave za uporabnike Chloros
+: Nit 1 (Zaznavanje) naloži slike in zazna kalibracijske cilje, Nit 2 (Kalibracija) izračuna kalibracijo odbojnosti, Nit 3 (Obdelava) izvede GPU-pospešeno odstranjevanje bayerjevega vzorca in izračun indeksa, Nit 4 (Izvoz) pa zapiše izhodne datoteke. Za maksimalno zmogljivost lahko več slik hkrati poteka v različnih niti. Za podrobnosti glejte [Obdelovalni potek](processing-architecture/processing-pipeline.md).

</details>

<details>

<summary>Kako izvedem diagnostiko na svoji namestitvi Chloros?</summary>

Uporabite ukaz `selftest` za izvedbo 7-stopenjskega preizkusa delovanja: različica, razpoložljivost vrat, zagon ozadja, povezljivost API-ja (`/api/test`), informacije o sistemu (`/api/system-info` — GPU/CUDA/PyTorch), prisotnost modela za odstranjevanje šuma ter pripravljenost CUDA in odstranjevalca šuma:

```bash
chloros-cli selftest
```

To je še posebej koristno v sistemih Linux za preverjanje nastavitev GPU in CUDA. Na napravi Jetson `CUDA: False` običajno pomeni, da manjkajo knjižnice CUDA in cuDNN za JetPack 6: namestite jih z ukazom `sudo apt install nvidia-jetpack`.

</details>

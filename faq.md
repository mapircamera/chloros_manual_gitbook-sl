---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Pogosta vprašanja

<details>

<summary>Ali lahko s programom Chloros obdelujem slike iz kamer, ki niso znamke MAPIR?</summary>

Ne, Chloros podpira le obdelavo slik iz kamer MAPIR. Za več informacij si oglejte seznam [podprtih modelov kamer](supported-cameras.md). Obdelavo slik iz drugih kamer ponujamo v oblaku MAPIR, celoten seznam pa si oglejte [tukaj](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Ali lahko kalibriram slike za odbojnost brez kalibracijskega cilja?</summary>

Ne. Brez slike kalibracijskega cilja, posnete ob zajemanju slik brez cilja, ne boste mogli povezati vrednosti pikslov slike z znanim odstotkom odbojnosti. Če ne vključite tudi dnevnika svetlobnega senzorja MAPIR, se spektrum okoljske svetlobe ne bo izmeril, rezultati odbojnosti pa ne bodo natančni.

</details>

<details>

<summary>Ali lahko slike uredim pred obdelavo v Chloros?</summary>

Ne. Chloros predpostavlja, da vhodni podatki niso bili spremenjeni. Ne spreminjajte imen datotek.

</details>

<details>

<summary>Ali lahko nastavim kamere MAPIR in Survey3 na avtomatsko osvetlitev in slike obdelam v programu Chloros?</summary>

Ne. Nizi slik Survey3 morajo imeti fiksno/zaklenjeno osvetlitev, torej brez samodejne hitrosti zaklopa ali samodejnega ISO. Vse slike istega modela kamere morajo imeti enako hitrost zaklopa in ISO (osvetlitev).

</details>

<details>

<summary>Ali lahko Chloros obdeluje ali analizira ortomosaične slike?</summary>

Ne. Podprte so le posamezne slike kamere MAPIR, ne pa sestavljene slike, kot je ortomosaikna karta.

</details>

<details>

<summary>Kako lahko pospešim korak zaznavanja ciljev v Chloros?</summary>

V tabeli brskalnika datotek predizberite ciljne slike v desnem stolpcu, s čimer boste Chloros naročili, naj kalibracijske cilje išče le v teh slikah, kar bo znatno pospešilo obdelavo.

</details>

<details>

<summary>Če bom svoje slike naložil v <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> ali jih moram pred nalaganjem obdelati v Chloros?</summary>

Če nameravate slike naložiti na našo spletno platformo za obdelavo [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), jih pred nalaganjem ne urejajte. Cloud bo opravil enako obdelavo in še več.

</details>

<details>

<summary>Ali bo MAPIR kdaj podpiral funkcijo X? Resnično si želim, da bi MAPIR ponujal X.</summary>

Vedno nas zanimajo povratne informacije o naših izdelkih. Če opazite težavo z našimi izdelki ali imate predlog, kako jih lahko izboljšamo, nas prosim [KONTAKTIRAJTE](https://www.mapir.camera/community/contact) in nam sporočite svoje mnenje. Večina našega raziskovalno-razvojnega dela temelji na poslušanju največjih potreb naših strank.

</details>

<details>

<summary>Ali je Chloros na voljo za Linux?</summary>

Da! Chloros 1.1.0 podpira Linux amd64 (x86_64) in arm64 (NVIDIA Jetson JetPack 6) prek paketov `.deb`. CLI in Python SDK sta v celoti podprta na Linux. Za Linux ni grafičnega vmesnika – vsa interakcija poteka prek [CLI](CLI.md) ali [Python SDK](api-python-sdk.md). Za podrobnosti glejte [Linux Pregled](linux/linux-overview.md).

</details>

<details>

<summary>Ali lahko Chloros uporabljam na NVIDIA Jetson?</summary>

Da! Chloros 1.1.0 podpira platforme NVIDIA Jetson, vključno z Jetson Nano, Orin Nano, Orin NX in AGX Orin, na katerih teče JetPack 6. Chloros samodejno zazna vaš model Jetson in optimizira strategijo obdelave. Za navodila za nastavitev in namestitev si oglejte [Vodnik za NVIDIA Jetson](linux/nvidia-jetson-guide.md).

</details>

<details>

<summary>Ali se Chloros samodejno optimizira za mojo strojno opremo?</summary>

Da! Chloros 1.1.0 vključuje [dinamično prilagajanje računalniških zmogljivosti](processing-architecture/dynamic-compute-adaptation.md), ki samodejno zazna vaš CPU, GPU, RAM in (na Jetsonu) toplotne senzorje. Nato izbere optimalno strategijo obdelave – od `GPU_PARALLEL` na sistemih z veliko pomnilnika do `GPU_SINGLE` na napravah z omejenimi zmogljivostmi ter `CPU_PARALLEL` na sistemih brez grafičnega procesorja NVIDIA. Ročna konfiguracija ni potrebna.

</details>

<details>

<summary>Kaj je 4-nitni procesni potok?</summary>

Chloros 1.1.0 uporablja 4-nitno arhitekturo s potekom za uporabnike Chloros+: Nit 1 (zaznavanje) naloži slike in zazna kalibracijske cilje, nit 2 (kalibracija) izračuna kalibracijo odbojnosti, nit 3 (obdelava) izvede GPU-pospešeno odstranjevanje bayerjevega vzorca in izračun indeksa, nit 4 (izvoz) pa zapiše izhodne datoteke. Za maksimalno prepustnost je mogoče več slik hkrati obdelovati v različnih niti. Podrobnosti najdete v [Obdelovalni poti](processing-architecture/processing-pipeline.md).

</details>

<details>

<summary>Kako izvedem diagnostiko na svoji namestitvi Chloros?</summary>

Uporabite ukaz `selftest` za izvedbo 7 sistemskih diagnostik, vključno s preverjanjem različice, razpoložljivostjo vrat, zagonom backenda, povezljivostjo API, informacijami o sistemu, modeli za odstranjevanje šuma in razpoložljivostjo CUDA:

```bash
chloros-cli selftest
```

To je še posebej koristno na sistemih Linux/Jetson za preverjanje nastavitev GPU in CUDA.

</details>

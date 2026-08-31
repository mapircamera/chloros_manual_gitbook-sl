# Delovni postopki za odbojnost

Svetlobni senzor DAQ pretvori radiometrične slike v odbojnost. Obstajata dva različna delovna postopka:

1. **En senzor** — en DAQ meri navzdol usmerjeno sevanje, medtem ko kamera zajema sliko, Chloros pa sevanje kamere deli s to referenčno vrednostjo.
2. **Dva senzorja** — dva senzorja DAQ, eden usmerjen v nebo, drugi pa v objekt, ustvarjata spektralno krivuljo odbojnosti v realnem času brez udeležbe kamere.

## En senzor + kamera (referenca navzdol usmerjenega sevanja)

DAQ deluje kot senzor navzdol usmerjenega svetlobnega sevanja (DLS): kamera meri sevalno moč navzgor **L**(W/m²/sr/nm), DAQ meri sevanje navzdol**E** (W/m²/nm), Chloros pa izračuna odbojnost po pasovih kot:

> ρ = π · L / E

Odčitek DAQ je vedno **časovno usklajen z osvetlitvijo** — zato DAQ in kamere uporabljajo skupno uro, sinhronizirano po protokolu PTP (glej [Mrežno povezovanje in sinhronizacija časa DAQ-E](ethernet-ptp.md)). Za delo na prostem si nadenite sončno kosinusno kapico in jo pravilno deklarirajte; deklaracija kapice neposredno prilagaja vrednost E (glej [Profil kapic in kalibrirano območje](caps-and-range.md)). Za kvantitativno delo upoštevajte značilnost instrumenta: kvantitativna obsevanost izhaja iz povprečja odčitkov v obdobju vsaj 15 sekund.

### Snemanje v realnem času

Povežite DAQ s kamero v zavihku »Kamere«: v nastavitvenem oknu vsake kamere je spustni meni **Svetlobni senzor**, ki navaja vse povezane DAQ-je (DAQ-U/M/E) iz zavihka »Svetlobni senzorji«; pri sinhronizirani matrici se izbira svetlobnega senzorja za celotno matrico prenese na vsakega člana (posamezne kamere lahko še vedno prevzamejo nadzor). Ko je senzor povezan, se njegovi spektri posredujejo v DLS-režo kamere, izvozi odbojnosti pa se delijo z ujemajočo se odčitano vrednostjo.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->

Dve značilnosti, ki jih je vredno poznati:

* **Če ni povezan noben DAQ → se odbojnost zavrne, ne pa ponaredi.** Chloros zavrne produkt odbojnosti in zabeleži razlog za preskok, namesto da bi tiho vrnil slabši produkt.
* **Uporabljena vrednost se ohrani.** Za vsak okvir odbojnosti se dejansko uporabljena vrednost DAQ zapiše kot stranski zapis `.daq` poleg posnetka, tako da je mogoče zajem kasneje ponovno obdelati ([Snemanje in format .daq](recording.md)).

### Obdelava posnetih slik

Za obdelavo po letu med sejo posnemite datoteko `.daq` in jo shranite skupaj s slikami – obdelovalni proces samodejno razreši navzdolno sevanje, ki se ujema s časovnim žigom, pri prvi uporabi pa iz oblaka MAPIR pridobi morebitne manjkajoče tovarniške kalibracije. Posnetki grafičnega vmesnika se ob zaključku samodejno dodajo v odprt projekt.

Referenčno odbojnost je mogoče izbrati ob obdelavi – `--reflectance-source` na `chloros-cli process` ali nastavitev vira odbojnosti v nastavitvah projekta v grafičnem vmesniku:

| Vrednost | Delovanje |
| --- | --- |
| `auto` (privzeto) | Kalibracijski cilj znotraj okvirja, ki je uspešno prestal kontrolo kakovosti, je absolutna referenca; DAQ-spuščanje (ρ = π·L/E) je nadomestna rešitev |
| `daq` | Veljavnost DAQ |
| `target` | Strog cilj znotraj okvira; brez nadomestitve z DAQ |

Glejte [Kalibracijske tarče](../calibration-targets.md) za delovne tokove tarč in [poglavje LATTICE](../lattice/README.md) ter [referenco CLI](../reference/cli-reference.md) za celoten procesni potek. Pri branju izvoženih pikslov odbojnosti uporabite označeno lestvico (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3: 65535) – glejte [Formati izhodnih slik](../output-image-formats.md).

### Pasovi zunaj kalibriranega območja DAQ-ja

Radiometrično kalibrirano območje DAQ-ja je ~374–974 nm. Chloros zavrne odbojnost, ki temelji na DAQ, za kateri koli pas kamere, pri katerem je manj kot polovica spektralne teže znotraj tega obsega, in poroča o razlogu za preskok `dls-uncalibrated-band-<nm>`. Med dobavljenimi artikli to vpliva le na model F988: odbojnost modela F988 se kalibrira z uporabo plošče za merjenje odbojnosti v prizoru: pas leži zunaj kalibriranega območja svetlobnega senzorja DAQ, zato Chloros uporabi vaš najnovejši posnetek plošče in ga ohrani med posameznimi meritvami plošče. Če kamera F988 deluje izključno v načinu DAQ, Chloros zavrne odbojnost, temelječo na DAQ, za ta pas z razlogom za preskok `dls-uncalibrated-band-988` – podprt je delovni tok s ploščo.

## Dvojni senzor (okoljska svetloba + objekt)

Dva senzorja DAQ – kateri koli par, ne glede na način prenosa – zagotavljata spektrum odbojnosti v realnem času brez kamere: en senzor je usmerjen proti nebu (**vir okoljske svetlobe**), drugi pa proti predmetu (**skener predmeta**), Chloros pa izračuna vrednost za vsako valovno dolžino:

> R(λ) = objekt(λ) / okolje(λ)

(nič, če je okoljska svetloba ≤ 0).

### V grafičnem uporabniškem vmesniku

Ko sta oba senzorja priključena v zavihku Svetlobni senzorji, odprite prekrivni okno za dodajanje senzorja (gumb »+« na ploščici grafa v pogledu mreže) in izberite **Združi okolico + objekt**. V spustnih menijih »Ambient Light Source« in »Object Scanner« izberite oba senzorja ter kliknite »Create«. Skupina se prikaže kot samostojen grafikon in kot vrstica v stranskem pasu z zeleno oznako**REF**.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->

Pod grafom odbojnosti (pogled v obliki seznama) **tabela indeksov vegetacije** v realnem času izračuna indekse iz krivulje z uporabo središč pasov pri modri 450 / zeleni 550 / rdeči 670 / NIR 800 nm. Indeksi, ki temeljijo na razmerjih in izničijo absolutno lestvico (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR), so vedno prikazani; indeksi, ki zahtevajo absolutno odbojnost (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) se prikažejo le, če sta oba senzorja kalibrirana glede na moč.

### Navidezna proti relativna – pravilo označevanja

Chloros označi izhod dvojnega senzorja glede na to, kar lahko senzorski par dejansko navaja:

| Senzorski par | Oznaka |
| --- | --- |
| Oba senzorja kalibrirana — naložen tovarniški paket | **Navidezna odbojnost** |
| Če je kateri koli senzor nekalibriran | **Relativna odbojnost** |

Vsi trije modeli so radiometrični: ko je naložen tovarniški kalibracijski paket senzorja, so njegovi spektri izraženi v absolutnih vrednostih W/m²/nm, zato par kalibriranih senzorjev daje razmerje do absolutne navidezne odbojnosti — prevoz tega ne določa. Če senzor še vedno prenaša surove števne vrednosti (paket ni dosegljiv), se rezultat razvrsti v relativno krivuljo (spektralna oblika ostaja veljavna). Oba senzorja morata imeti pravilno deklarirane omejitve ([Profil omejitev in kalibrirano območje](caps-and-range.md)).

### Iz Python

V združenem vmesniku SDK ni namenske funkcije za delo z dvema senzorjema: odprite dve seji z `chloros_sdk.connect_daq_sensor()` in sami izračunajte razmerje med njunima spektroma `latest()`, pri čemer uporabite enako konvencijo označevanja. (Orodje za snemanje z dvema senzorjema obstaja tudi na notranji površini z neposredno strojno opremo MAPIR, navedeno v [Referenca CLI](../reference/cli-reference.md) – ni del priloženega CLI; zgoraj opisani potek dela v grafičnem vmesniku je podprta pot v realnem času.)

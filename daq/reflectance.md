# Delovni postopki za odbojnost

Svetlobni senzor DAQ pretvori radiometrične posnetke v odbojnost. Obstajata dva različna delovna postopka:

1. **En senzor** — en DAQ meri navzdol usmerjeno sevanje, medtem ko kamera snema, program Chloros
pa sevanje kamere deli s to referenčno vrednostjo.
2. **Dva senzorja** — dva senzorja DAQ, eden usmerjen v nebo, drugi pa v objekt, ustvarjata spektralno krivuljo odbojnosti v realnem času brez uporabe kamere.

## En senzor + kamera (referenca navzdol usmerjenega sevanja)

DAQ deluje kot senzor navzdol usmerjenega svetlobnega sevanja (DLS): kamera meri sevalno moč navzgor **L**(W/m²/sr/nm), DAQ meri sevanje navzdol**E** (W/m²/nm), program Chloros
pa izračuna odbojnost po posameznih pasovih po formuli:

> ρ = π · L / E

Odčitek DAQ je vedno **časovno usklajen z osvetlitvijo** — zato si DAQ in kamere delijo uro, ki deluje po standardu PTP (glej [Mrežno povezovanje in sinhronizacija časa DAQ-E](ethernet-ptp.md)). Za delo na prostem namestite sončno kosinusno kapico in jo pravilno deklarirajte; deklaracija kapice neposredno prilagaja vrednost E (glej [Cap Profiles &amp; Calibrated Range](caps-and-range.md)). Za kvantitativno delo upoštevajte značilnosti instrumenta: kvantitativna obsevanost izhaja iz povprečja odčitkov v obdobju vsaj 15 s.

### Snemanje v živo

Povežite DAQ s kamero v zavihku »Kamere«: v nastavitvenem oknu vsake kamere je spustni meni **Svetlobni senzor**, ki navaja vse povezane DAQ-je (DAQ-U/M/E) iz zavihka »Svetlobni senzorji«; pri sinhronizirani matrici se izbira svetlobnega senzorja za celotno matrico prenese na vsakega člana (posamezne kamere lahko še vedno prevzamejo nadzor). Ko je senzor povezan, se njegovi spektri prenašajo v DLS-režo kamere, izvozi odbojnosti pa se delijo z ujemajočo se odčitano vrednostjo. V nizu član z lastnim senzorjem uporablja lastno odčitano vrednost – za svoje pasove v kompozitni sliki v živo, svoje izvozne podatke in svojo samodejno osvetlitev –, ostali člani pa uporabljajo vrednosti niza.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->


Dve lastnosti, ki jih je vredno poznati:

* **Če ni vezave z DAQ → se odbojnost zavrne, ne pa ponaredi.** Program Chloros
zavrne izračun odbojnosti in zabeleži razlog za preskok, namesto da bi tiho vrnil manjši izračun.
* **Uporabljena vrednost se ohrani.** Za vsak okvir odbojnosti se dejansko uporabljena vrednost DAQ zapiše kot datoteka `.daq` v mapi `daq/` zajetih podatkov, tako da se zajem lahko kasneje ponovno obdela ([Snemanje in format .daq](recording.md)). Datoteka `.daq.cameras.json` poleg nje navaja kamere, ki jih opisuje, ponovna obdelava pa jo uskladi le s slikami teh kamer.

### Obdelava posnetih slik

Za obdelavo po letu med sejo posnemite datoteko `.daq` in jo shranite skupaj s slikovnim gradivom – obdelovalni proces samodejno razreši časovno usklajene posnetke navzdol usmerjenega žarka ter ob prvi uporabi pridobi morebitne manjkajoče tovarniške kalibracije iz oblaka MAPIR
. Posnetki iz grafičnega vmesnika se ob zaključku samodejno dodajo odprtemu projektu.

Referenčno odbojnost je mogoče izbrati ob obdelavi – `--reflectance-source` na `chloros-cli process` ali nastavitev vira odbojnosti v nastavitvah projekta v grafičnem uporabniškem vmesniku:

| Vrednost | Delovanje |
| --- | --- |
| `auto` (privzeto) | Cilj kalibracije znotraj okvirja, ki je uspešno prestal kontrolo kakovosti (QA), je absolutna referenca; kalibracija DAQ (ρ = π·L/E) je nadomestna rešitev. Cilj, ki opravi kontrolo kakovosti z oznako (izpadel panel ali ne delujoč pas), ni samodejno prednosten: verige »cilj«, »cilj+DAQ« in »samo DAQ« se preizkušajo v vrstnem redu glede na njihovo izmerjeno oceno na izločenih panelih. |
| `daq` | DAQ je avtoritativen |
| `target` | Strog cilj znotraj okvirja; brez nadomestitve z DAQ |

Glejte [Kalibracijske tarče](../calibration-targets.md) za delovne tokove tarč ter [poglavje LATTICE](../lattice/README.md) ter [referenco CLI
](../reference/cli-reference.md) za celoten procesni potek. Pri branju izvoženih pikslov odbojnosti uporabite označeno lestvico (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3
: 65535) — glejte [Formati izhodnih slik](../output-image-formats.md).

### Pasovi zunaj kalibriranega območja DAQ-ja

Radiometrično kalibrirano območje DAQ-ja je ~374–974 nm. Program Chloros
zavrne odbojnost, pridobljeno prek DAQ-ja, za kateri koli pas kamere, pri katerem je manj kot polovica spektralne teže znotraj tega območja, in pri tem navede razlog za preskok `dls-uncalibrated-band-<nm>`. Med trenutno dostopnimi artikli to vpliva le na model F988: odbojnost modela F988 se kalibrira z uporabo plošče za merjenje odbojnosti v prizoru; pas leži zunaj kalibriranega območja svetlobnega senzorja DAQ, zato Chloros
uporabi vaš najnovejši posnetek plošče in ga ohrani med posameznimi meritvami plošče. Če kamera F988 deluje izključno v načinu DAQ, Chloros
zavrne odbojnost, ki temelji na DAQ, za ta pas z razlogom za preskok `dls-uncalibrated-band-988` – podprt je delovni tok s ploščo.

## Dvojni senzor (okoljska svetloba + objekt)

Dva senzorja DAQ – kateri koli par, ne glede na način prenosa – zagotavljata spektrum odbojnosti v realnem času brez kamere: en senzor je usmerjen proti nebu (**vir okoljske svetlobe**), drugi pa proti predmetu (**skener predmeta**), pri čemer program Chloros
izračuna vrednost za vsako valovno dolžino:

> R(λ) = objekt(λ) / okolje(λ)

(nič, če je okoljska svetloba ≤ 0).

### V grafičnem uporabniškem vmesniku

Ko sta oba senzorja priključena v zavihku Svetlobni senzorji, odprite prekrivni okvir za dodajanje senzorja (gumb »+« na ploščici grafa v pogledu mreže) in izberite **Kombiniraj okolico + objekt**. V spustnih menijih**Vir okoljske svetlobe**in**Skener objekta**izberite oba senzorja ter kliknite**Ustvari**. Skupina se prikaže kot samostojen graf in kot vrstica v stranskem pasu z zeleno oznako**REF**.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->


Pod grafom odbojnosti (pogled v obliki seznama) **tabela indeksov vegetacije** v realnem času izračuna indekse iz krivulje z uporabo središč pasov pri modri 450 / zeleni 550 / rdeči 670 / NIR
800 nm. Indeksi, ki temeljijo na razmerjih in izničijo absolutno lestvico (NDVI
, GNDVI
, ENDVI, WDRVI
, GRVI
, CVI
, GCI
, MSR), so vedno prikazani; indeksi, ki potrebujejo absolutno odbojnost (EVI
, SAVI
, OSAVI
, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI
, NLI, MNLI, FCI, GEMI
), se prikažejo le, če sta oba senzorja kalibrirana glede na moč.

### Navidezna proti relativni – pravilo označevanja

Chloros
označuje izhod dvojnega senzorja glede na to, kar lahko senzorski par dejansko zagotovi:

| Senzorski par | Oznaka |
| --- | --- |
| Oba senzorja kalibrirana – naložen tovarniški paket | **Navidezna odbojnost** |
| Če je kateri koli senzor nekalibriran | **Relativna odbojnost** |

Vsi trije modeli so radiometrični: ko je naložen tovarniški kalibracijski paket senzorja, so njegovi spektri izraženi v absolutnih vrednostih W/m²/nm, zato par kalibriranih senzorjev daje razmerje do absolutne navidezne odbojnosti – transport tega ne določa. Če senzor še vedno pošilja surove številske vrednosti (paket ni dosegljiv), se rezultat razvrsti kot relativna krivulja (spektralna oblika ostane veljavna). Oba senzorja morata imeti pravilno deklarirane omejitve ([Profil omejitev in kalibrirano območje](caps-and-range.md)).

### Iz Python
a

V združenem vmesniku SDK
ni namenske funkcije za delo z dvema senzorjema: odprite dve seji z `chloros_sdk.connect_daq_sensor()` in sami izračunajte razmerje med njunima spektroma `latest()`, pri čemer uporabite enako konvencijo označevanja. (Orodje za snemanje z dvema senzorjema obstaja tudi v MAPIR
-jevem notranjem vmesniku za neposredno upravljanje strojne opreme, ki je zaradi popolnosti navedeno v [CLI
Reference](../reference/cli-reference.md) — ni del priloženega CLI
; zgoraj navedeni potek dela v grafičnem vmesniku je podprta pot v živo.)

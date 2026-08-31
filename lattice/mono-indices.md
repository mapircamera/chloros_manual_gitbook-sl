# Monokromatske kamere in indeksi vegetacije

## Ena kamera = en pas

Kamera **M3M**je monokromatska različica kamere Bayer**M3C**: monokromatski senzor IMX265, za katerim je nameščen en ozkopasovni interferenčni filter. Ime pasu je navedeno v imenu modela — `M3M-<lens>-F<wavelength>`, npr. `M3M-L87-F685` (prikazano v Chloros kot `LATT-M3M-L87-F685`). Senzor zagotavlja**en sam pas v sivih odtenkih** brez Bayerjevega mozaika: ni ničesar, kar bi bilo treba demozajati, ni medkanalnega prekrivanja, ki bi ga bilo treba ločiti, in ni treba nastavljati belega ravnovesja.

Posledice, ki jih je vredno poznati, preden načrtujete monokromatski sistem:

* **Sevanje in odbojnost sta v celoti definirana za vsak pas.**Gre za radiometrične karte po pasovih, zato ena kamera M3M proizvaja kalibrirano sevanje float32 (W/m²/sr/nm) in odbojnost uint16 (`32768` = ρ 1,0) točno tako, kot to počne pas M3C. Mono posnetki vsebujejo**identitetno** matriko odziva senzorja — razmešavanje 3×3 ni potrebno niti se ne uporablja.
* **Ena sama mono kamera ne more ustvariti indeksa vegetacije.** NDVI, NDRE in podobni indeksi potrebujejo vsaj dva pasova. Za izračun indeksov iz mono opreme je treba združiti več kamer M3M — glejte spodaj.
* Kamere M3M prenašajo **Mono12** (12-bitno, 2 bajta/piksel po kablu), kar je pomembno za [načrtovanje pasovne širine matrike](arrays.md#bandwidth-the-rules-of-thumb).

## Kaj Chloros preskoči za mono – in kako vas o tem obvesti

Stopnje barvnega procesa preprosto ne veljajo za enopasovni senzor. Chloros **jih preskoči z enovrstičnim sporočilom**, namesto da bi prikazal napako, in jih še vedno normalno izvaja za katero koli kamero M3C (Bayer) v isti seji:

| Faza | Obnašanje v načinu Mono (M3M) | Obnašanje v načinu M3C |
| --- | --- | --- |
| Demosaic / debayer | Preskočeno — izvoz `debayered` je enokanalna slika v sivih odtenkih. | 3-kanalni demosaic. |
| Belina (`lattice white-balance`) | Preskočeno z enovrstičnim sporočilom. | Deluje normalno. |
| Barvni profil (`lattice color-profile`) | Preskočeno z enovrstičnim sporočilom. | Deluje normalno. |
| Nasičenost/kontrast (`lattice color`) | Preskočeno z enovrstičnim sporočilom. | Deluje normalno. |
| Ločevanje spektralnega prekrivanja | Identiteta (brez matrike 3×3). | Uporabljena matrika 3×3 za vsako kamero posebej. |
| Sijaj / odbojnost | **Deluje** — po posameznem pasu, popolnoma kalibrirano. | Deluje po posameznem pasu. |

GUI uporablja enako omejitev: za monokamero okno nastavitev za vsako kamero skrije vrstice, ki so namenjene izključno RGB (bela ravnovesje, gama, barvni profil, nasičenost, kontrast, razdelitev kanalov), histogram v realnem času pa je zaklenjen na eno samo krivuljo **MONO**. Razlikovalni dejavnik v celotnem nizu je token `M3M` v nizu modela, ki se v grafičnem vmesniku prikaže kot SDK.

## Indeksi potrebujejo ≥ 2 pasove: poravnava → zlaganje → indeksiranje

Delovni tok indeksiranja mono slik je vedno enak in obsega tri korake:

1. **Poravnava** — usmerite več M3M-kamer na različne valovne dolžine (npr. F650 »Red« in F850 »NIR«), povežite jih v [mrežo več kamer](arrays.md) in pustite, da Chloros izračuna deformacijo za sočasno registracijo med kamerami.
2. **Zložba** — poravnani okvirji postanejo ena večpasovna slika (vsaka kamera prispeva eno poimenovano pasovno območje).
3. **Indeks** — izračunajte indeksno formulo za pasovna območja zložbe, po želji pa jo upodobite prek LUT-a.

V grafičnem uporabniškem vmesniku (GUI) ta celotna veriga predstavlja način prikazovanja niza **Combined Cameras**: sestavljena slika v živo je že poravnana, indeksni kalkulator niza (spodaj) pa definira formulo, po kateri se upodablja. Posnete izvozne datoteke je mogoče deformirati v isto poravnavo z možnostjo zajema**Aligned**.

## Kalkulator indeksa

Kalkulator indeksa ustvari indeksni izraz, ki ga uporabljata prikaz v živo in izvozi indeksa po posameznih kamerah. Gre za eno skupno površino, ki se odpre z dveh mest v stranskem meniju zavihka »Kamere«:

* **Po posamezni kameri**— Živi predogled → zobnik**Indeks** (samo kamere Bayer RGN/OCN/NGB; samostojna monokamera nima nadzora nad indeksom, saj en pas ne more tvoriti indeksa).
* **Po matrici**— nastavitve matrice → Živi predogled → ikona zobnika**Indeks**. To je pot za monokamere: seznam pasov zajema**vse kamere v matrici**, zato monokamera s svojim parom tukaj prispeva oba pasova.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Njegovi elementi za nadzor, od zgoraj navzdol:

* **Čipi pasov** („Pasovi — kliknite za dodajanje v izraz“) — en gumb za vsak razpoložljiv pas, označen z imenom barve + valovno dolžino v nm (podvojena imena barv se razlikujejo, npr. „Barva 850“). S klikom vstavite žeton pasu na mesto kazalca. Pasovi iz kamer, ki ne morejo proizvajati sevanja po posameznih pasovih (RGB/FRGB), so izločeni.
* **Gumbi za operatorje in funkcije** — `+ - * / ( ) ^ ,` in `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Polje za izraze** — prosto vnesena formula; zaznamek prikazuje klasično obliko NDVI v obliki `(NIR - Red) / (NIR + Red)`. Predogled zgoraj, ki je samo za branje in razdeljen na tokene, prikazuje pasovne čipe, številke in zastavice neznanih tokenov.
* **Pasica veljavnosti**— siva »Prazno — indeks se ne bo uporabil«; zeleni »Veljaven izraz«; rdeči z natančno napako pri razčlenitvi (neznan pas, dvoumen pas, ki ga zajemajo več kamer, manjkajoči oklepaj, …); ali rumeni, kadar je izraz veljaven, vendar**konstanten** (npr. `X/X`, ali imenovalec NDVI, vnesen kot `−` namesto `+`) — konstanta celoten okvir preslika v eno barvo.
* Ločeno rumeno opozorilo se prikaže, če je uporabljeni izraz pravilen, vendar je **živi okvir enoten** (ravna ali nasičena scena) – zazna se zmanjšanje histograma.
* Vnosi **Uporabi LUT**(privzeto vklopljeno; izklopljeno = raztegnitev sive lestvice),**Raven**2/3/5/7 stopenj (privzeto 7 stopenj) ter**Min / Max**obdajajo drsnik gradienta. Privzeta vrednost za Min je**0,2**— to poveča barvni prehod v obseg, relevanten za vegetacijo, medtem ko vrednosti pod tem preidejo v sivo lestvico; nastavite Min na −1 za celoten indeksni obseg (gumb**Ponastavi** ponovno vzpostavi obseg −1…+1). Privzeta vrednost za Max je 1.
* **Histogram v realnem času** porazdelitve indeksa — palice, skalirane s kvadratnim korenom, jantarne črte 2. in 98. percentila, bela sredinska črta ter odčitki izven območja („◀ N% &lt; lo“ / „hi &lt; N% ▶“), ki ob vrednosti nad 1 % postanejo jantarne kot znak, da je treba razširiti okno Min/Max.
* **Uporabi**potrdi izraz v živo; prilagoditve LUT se uporabijo v realnem času brez pritiska na gumb »Uporabi«. Izrazi so namerno**veljavni le za tekočo sejo** — med sejami se ne ohranijo.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->

## Pot CLI

Ista veriga poravnava → sklad → indeks, ki jo je mogoče v celoti programirati:

```bash
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` preslikuje simbole prednastavitve na imena pasov v skladu. Dve pravili vam prihranita neuspešno izvedbo:

* **Pri simbolih se upošteva velika in mala črka** in se morajo natančno ujemati z imeni kanalov prednastavitve — prednastavitve uporabljajo male črke (NDVI so `red`,`nir`; preverite `--list-presets`). `--channel red=Red_660` deluje; `--channel RED=660` ne uspe in prikaže napako `channel_map missing entries`.
* Stran pasu mora imenovati pas v poravnanem skladu (`lattice align-info --profile align.json` jih navaja). Način brez povezave sprejema tudi indekse pasov, ki se začnejo z 0, npr. `--channel red=0 --channel nir=1`.

`lattice index` deluje tudi popolnoma v načinu brez povezave na podlagi shranjenega poravnanega večpasovnega TIFF:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Prednastavitve indeksov

`lattice index --preset` (in [Index/LUT sandbox](../image-viewer-gui/index-lut-sandbox.md) na zavihku Slika, ki uporablja isti pogon) vključuje teh **22 prednastavitev**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Za formule in simbole kanalov posameznih prednastavitev zaženite `chloros-cli lattice index --list-presets`, za razpoložljive barvne prehode pa `--list-gradients`. Pri uporabi lastnih formul uporabite `--formula EXPR` z enako sintakso kot pri kalkulatorju indeksov. Upoštevajte, da je ta seznam prednastavitev specifičen za indeksni motor LATTICE – spustni meni »Obdelava« v nastavitvah projekta za uvožene slike vsebuje drugačen seznam (glejte [Formule multispektralnih indeksov](../project-settings/multispectral-index-formulas.md)).

Celoten nabor zastavic (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, drsniki za poravnavo in popačenje za `--live`, in še več) je dokumentiran v [CLI Referenca § Indeks / Matematika vegetacije](../reference/cli-reference.md#index--vegetation-maths); ekvivalenti za SDK so v [SDK Referenca](../reference/sdk-reference.md).

## Zajemanje indeksnih izdelkov iz enotnega niza

Ko je niz povezan in je uporabljen indeksni izraz, `array-capture` (ali GUI **Zajemi vse**) shrani ravni izvoza za vsako kamero *in* indeksno upodabljanje — `--index`/`--no-index` to vklopi na CLI, privzeto pa zajame vse ustrezne ravni. Prispevek enokamere k vsaki skupini zajemov je njen en pas na ravneh surovih podatkov/debayeringa (siva lestvica)/sevanja/odbojnosti, skupaj s skupnim sestavljenim indeksom, kadar niz deluje v kombiniranem načinu. Glejte [Mreže z več kamerami § Zajemanje](arrays.md#capturing-monitoring-vs-analysis).

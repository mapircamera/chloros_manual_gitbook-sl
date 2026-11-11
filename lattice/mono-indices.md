# Monokromatske kamere in indeksi vegetacije

## Ena kamera = en pas

Kamera **M3M**je monokromatska različica kamere Bayer**M3C**: monokromatski senzor IMX265, za katerim je nameščen en ozkopasovni interferenčni filter. Ime pasu je navedeno v imenu modela — `M3M-<lens>-F<wavelength>`, npr. `M3M-L87-F685` (v programu Chloros se prikaže kot `LATT-M3M-L87-F685`). Senzor zagotavlja **en sam pas sivo lestvice** brez Bayerjevega mozaika: ni ničesar, kar bi bilo treba demosaikirati, ni medkanalnega prekrivanja, ki bi ga bilo treba ločiti, in ni treba nastavljati beline.

Posledice, ki jih je vredno poznati, preden načrtujete monokromatski sistem:

* **Sevanje in odbojnost sta v celoti definirana za vsak pas.**Gre za radiometrične karte po posameznih pasovih, zato ena kamera M3M proizvaja kalibrirano sevanje v formatu float32 (W/m²/sr/nm) in odbojnost v formatu uint16 (`32768` = ρ 1,0), točno tako kot pas M3C. Mono posnetki vsebujejo**identitetno** matriko odziva senzorja — razmešavanje 3×3 ni potrebno niti se ne uporablja.
* **Ena sama mono kamera ne more izračunati indeksa vegetacije.** Za indekse, kot so NDVI, NDRE in podobni, sta potrebna vsaj dva pasova. Za izračun indeksov iz mono opreme je treba kombinirati več kamer M3M — glejte spodaj.
* Kamere M3M prenašajo **Mono12** (12-bitni: 2 bajta/piksel po kablu ali 1,5 bajta/piksel, če matrika uporablja [zgoščeni 12-bitni prenos](arrays.md#packed-12-bit-transport)), kar je pomembno za [načrtovanje pasovne širine matrike](arrays.md#bandwidth-how-many-cameras-fit).

## Kaj Chloros preskoči pri mono načinu — in kako vas o tem obvesti

Stopnje barvnega procesa preprosto ne veljajo za enopasovni senzor. Chloros **jih preskoči z enovrstičnim sporočilom**, namesto da bi prikazal napako, in jih še vedno normalno izvaja za katero koli kamero M3C (Bayer) v isti seji:

| Faza | Obnašanje v načinu Mono (M3M) | Obnašanje v načinu M3C |
| --- | --- | --- |
| Demosaic / debayer | Preskočeno — izvozna raven `debayered` je enokanalna slika v sivih odtenkih. | 3-kanalni demosaic. |
| Belina (`lattice white-balance`) | Preskočeno z enovrstičnim sporočilom. | Deluje normalno. |
| Barvni profil (`lattice color-profile`) | Preskočeno z enovrstičnim sporočilom. | Deluje normalno. |
| Nasičenost/kontrast (`lattice color`) | Preskočeno z enovrstičnim sporočilom. | Deluje normalno. |
| Ločevanje spektralnega prekrivanja | Identiteta (brez matrike 3×3). | Uporabljena matrika 3×3 za vsako kamero posebej. |
| Sijaj / odbojnost | **Deluje** — po posameznih pasovih, popolnoma kalibrirano. | Deluje po posameznih pasovih. |

GUI uporablja enako omejevanje: za monokamero okno nastavitev za vsako kamero skrije vrstice, namenjene izključno RGB (belo ravnovesje, gama, barvni profil, nasičenost, kontrast, razdelitev kanalov), živ histogram pa je zaklenjen na eno samo **MONO** krivuljo. Razlikovalni element v celotnem sklopu je token `M3M` v nizu modela, ki se v grafičnem vmesniku/SDK-ju prikaže kot `is_mono`.

## Indeksi potrebujejo ≥ 2 pasova: poravnava → sklop → indeks

Delovni tok mono indeksa vedno obsega iste tri korake:

1. **Poravnava** — usmerite več kamer M3M na različne valovne dolžine (npr. F650 »Red « in F850 »NIR «), povežite jih v [niz več kamer](arrays.md) ter pustite, da program Chloros izračuna popravke za usklajevanje med kamerami.
2. **Zlaganje** — poravnani posnetki postanejo ena večpasovna slika (vsaka kamera prispeva en poimenovan pas).
3. **Indeks** — izračunajte indeksno formulo za pasove sklada, po želji pa jo upodabljajte prek LUT-a.

V grafičnem uporabniškem vmesniku je celotna veriga prikazana v načinu **Kombinirane kamere**: sestavljena slika v živo je že poravnana, indeksni kalkulator niza (spodaj) pa določa formulo, po kateri se upodablja. Zajete izvoze je mogoče deformirati v isto poravnavo z možnostjo zajema**Aligned**.

## Kalkulator indeksa

Kalkulator indeksa ustvari indeksni izraz, ki ga uporabljata v živo prikaz in izvozi indeksa po posameznih kamerah. Gre za eno skupno površino, ki se odpre iz dveh mest v stranskem meniju zavihka »Kamere«:

* **Po kameri**— Živi predogled → zobnik**Indeks** (samo kamere RGN/OCN/NGB Bayer; samostojna mono kamera nima nadzora nad indeksom, ker en pas ne more tvoriti indeksa).
* **Po matrici**— nastavitve matrice → Živi predogled → zobnik**Indeks**. To je pot za enobarvne kamere: seznam pasov zajema**vse kamere v matrici**, zato enobarvni par tukaj prispeva svoja dva pasova.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Njegovi elementi za nadzor, od zgoraj navzdol:

* **Čipi pasov** („Pasovi — kliknite za dodajanje v izraz“) — en gumb na razpoložljiv pas, označen z imenom barve + valovno dolžino v nm (podvojena imena barv se razlikujejo, npr. „Barva 850“). S klikom se vstavijo oznake pasov na mesto kazalca. Pasovi iz kamer, ki ne morejo proizvajati sevanja po posameznih pasovih (RGB/FRGB), so izločeni.
* **Gumbi za operacije in funkcije** — `+ - * / ( ) ^ ,` in `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Polje za vnos izraza** — prosto vnesena formula; v mestu za vnos je prikazana klasična oblika NDVI: `(NIR - Red) / (NIR + Red)`. Nad njim je pregled v obliki tokenov, ki je samo za branje in prikazuje pasovne čipe, številke ter zastavice za neznane tokene.
* **Pasica veljavnosti**— siva »Prazno — indeks se ne bo uporabil«; zelena »Veljaven izraz«; rdeča s konkretno napako pri razčlenjevanju (neznani pas, dvoumen pas, ki ga zajemajo več kamer, manjkajoči oklepaj, …); ali rumena, kadar je izraz veljaven, vendar**konstanta** (npr. `X/X` ali imenovalec NDVI, vnesen kot `−` namesto `+`) — konstanta celoten okvir preslika v eno barvo.
* Ločeno rumeno opozorilo se prikaže, če je uporabljeni izraz v redu, vendar je **živi okvir enoten** (ravna ali nasičena scena) — zaznan je sesutje histograma.
* Vnosi **Uporabi LUT**(privzeto vklopljeno; izklopljeno = raztezek sive lestvice),**Raven**2/3/5/7 stopenj (privzeto 7 stopenj) ter**Min / Max**ob barvnem prehodu. Privzeta vrednost za Min je**0,2**— to poveča barvni prehod v območje, relevantno za vegetacijo, medtem ko vrednosti pod tem preidejo v sivo lestvico; nastavite Min na −1 za celotno območje indeksa (gumb**Ponastavi** ponovno vzpostavi območje od −1 do +1). Privzeta vrednost za Max je 1.
* **Živi histogram** porazdelitve indeksa — palice, skalirane s kvadratnim korenom, jantarne črte za 2. in 98. percentil, bela črta za mediano ter odčitki izven območja („◀ N % < lo“ / „hi < N % ▶“), ki se nad 1 % obarvajo v jantarno barvo kot znak, da je treba razširiti okno Min/Max.
* **Uporabi** potrdi izraz v živi tok; prilagoditve LUT se uporabijo v realnem času brez pritiska na gumb Uporabi. Če je projekt odprt, se izraz in nastavitve LUT shranijo skupaj s projektom ter obnovijo ob ponovnem odprtju; brez projekta veljajo le za tekočo sejo.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->



## Pot v CLI



Ista veriga poravnava → zlaganje → indeksiranje, ki jo je mogoče v celoti programirati:

```bash
chloros-cli lattice align-calibrate --serials SN_RED,SN_NIR --save align.json
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`align-calibrate` uporablja povezano matrico, ki vsebuje kamere, ali pa jih sam odpre za eno izvedbo. `index --live` vedno sam odpre kamere ([začasni niz](../reference/cli-reference.md#multi-camera-transient-sessions)) in nima `--serials`: odpre vsako kamero v omrežju. Zato sprostite vse kamere, ki jih drži Chloros, preden — GUI, `cam-connect`, `array-connect` (`chloros-cli lattice array-disconnect --all`): ponovno odpiranje zadržane kamere lahko ustavi tok njenega imetnika ali povzroči sesutje strežnika. Kamera, ki ne prenaša nobenega okvirja, povzroči neuspeh izvajanja, zato se ne zapiše niti zaloga niti indeks.

`--channel` preslikava simbole prednastavitve na imena pasov v zalogi. Dve pravili vam preprečita neuspešno izvedbo:

* **Pri simbolih se upošteva velika in mala črka** in morajo natančno ustrezati imenom kanalov prednastavitve — prednastavitve uporabljajo male črke (za NDVI so to `red`, `nir`; preverite `--list-presets`). `--channel red=Red_660` deluje; `--channel RED=660` se ne izvede in prikaže napako `channel_map missing entries`.
* Na strani pasov je treba navesti ime pasu v poravnanem nizu (`lattice align-info --profile align.json` jih navaja). Način brez povezave sprejema tudi indekse pasov, ki se začnejo z 0, npr. `--channel red=0 --channel nir=1`.

`lattice index` deluje tudi v načinu brez povezave na shranjeni poravnani večpasovni datoteki TIFF. Vsak obrazec teče na strežniku, zato mora ta teči in biti prijavljen na plačljivi stopnji Chloros+; relativni `--input` ali `--output` se nanaša na mapo, iz katere izvajate ukaz:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Prednastavitve indeksa

`lattice index --preset` (in [Index/LUT sandbox](../image-viewer-gui/index-lut-sandbox.md) na zavihku »Image«, ki uporablja isti pogon) vključuje naslednjih **22 prednastavitev**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Za formule in simbole kanalov posameznih prednastavitev zaženite `chloros-cli lattice index --list-presets`, za razpoložljive barvne prehode pa `--list-gradients`. Pri lastnih formulah uporabite `--formula EXPR` z enako sintakso kot pri kalkulatorju indeksov. Upoštevajte, da je ta seznam prednastavitev specifičen za indeksni motor LATTICE — spustni meni za obdelavo v nastavitvah projekta za uvožene slike vsebuje drugačen seznam (glejte [Formule multispektralnih indeksov](../project-settings/multispectral-index-formulas.md)).

Celoten nabor zastavic (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, drsniki za poravnavo in deformacijo za `--live` ter drugo) je dokumentiran v [Referenca CLI § Indeksi / Vegetačna matematika](../reference/cli-reference.md#index--vegetation-maths); SDK-ekvivalenti so v [SDK Reference](../reference/sdk-reference.md).

## Zajemanje indeksnih izdelkov iz mono matrike

Ko je matrika priključena in je uporabljen indeksni izraz, `array-capture` (ali GUI **Zajemi vse**) shrani ravni izvoza za vsako kamero *in* indeksno upodabljanje — `--index`/`--no-index` to vklopi v CLI, privzeto pa zajame vse ustrezne ravni. Prispevek mono kamere k vsaki skupini zajemov je njen en pas na ravneh surovih podatkov/debayeringa (siva lestvica)/sevanja/odbojnosti, skupaj s skupnim kompozitom kombiniranega indeksa, kadar niz deluje v kombiniranem načinu. Glej [Nizi z več kamerami § Snemanje](arrays.md#capturing-monitoring-vs-analysis).

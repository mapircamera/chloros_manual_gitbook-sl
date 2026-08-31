# Nastavitve in načini zajemanja

Zajemanje v zavihku »Kamere« poteka prek enega rdečega gumba **Zajemi vse**in enega okna**Nastavitve zajemanja**, ki določa, kaj ta gumb sproži: katere kamere sodelujejo, katere vrste izvoza shranjuje vsaka kamera ter ali se zaklop sproži enkrat, neprekinjeno ali v intervalih. Ta stran opisuje celoten potek – konfiguracijo, sam zajem, kam se datoteke shranijo na disku in kako jih kasneje ponovno obdelati v kalibrirane izdelke. Nadzor nad kamerami in nizom je na voljo v [Nastavitvah kamere](camera-settings.md).

{% hint style="info" %}
**Za zajem je potreben odprt projekt.** Možnosti »Zajemi vse« in ikona zobnika »Nastavitve zajema« sta onemogočeni, dokler ni odprt projekt („Ustvarite ali odprite projekt za shranjevanje posnetkov“). Vsak posnetek se shrani v projektni mapi v `captures/`.
{% endhint %}

## Okno »Nastavitve zajema«

Odprite ga s **zobnikom ob možnosti »Zajemi vse«**na stranskem seznamu kamer ali z gumbom**»Odpri nastavitve zajema…«** na dnu katerega koli okna z nastavitvami za posamezno kamero. Naslov je »Nastavitve zajema« z gumbom ← za vrnitev.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->

Vaše izbire tukaj – vključene kamere, potrditvena polja po tipu in način zajema – se shranijo **po posameznem projektu** in se obnovijo, ko ga ponovno odprete.

### Načini zajemanja

Trije gumbi za izbiro načina na vrhu okna:

| Način | Kaj počne | Podnastavitve (privzete) |
| --- | --- | --- |
| **Enkratno** *(privzeto)* | En zajem na vseh izbranih kamerah. | — |
| **Neprekinjeno**| Zaporedni posnetki, dokler ni izpolnjen pogoj za ustavitev. | Ustavitev po**število posnetkov** (privzeto 1) *ali* **trajanje zajema** (privzeto 10 s; enote: sekunde / minute / ure / dnevi). |
| **Interval**(časovni preskok) | Serije posnetkov po časovniku. |**Posnetki / interval**(privzeto 1) ·**Vsake**N enote (privzeto 5 s) ·**Za** N enote (privzeto 1 m). |

V načinu »Neprekinjeno« ali »Interval« se gumb »Posnemi vse« med delovanjem spremeni v gumb **»Ustavi (N)«**, ki šteje posnetke, ko se ti shranijo.

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->

### Izbiranje kamer in vrst izvoza

Pomožno besedilo v oknu to povzema: izberite, katere kamere in vrste izvoza bo ustvaril »Zajemi vse« — privzeto je vse vklopljeno, izbire pa se shranijo skupaj s tem projektom.

* Gumbi **Izberi vse / Ne izberi nič** hkrati preklopijo potrditvena polja za vključitev vsake kamere.
* **Stikala za izbiro vrste izvoza v paketu**(dve vrsti gumbov):**Vse surove / Vse brez debayeringa / Vse predoglede / Vse sevanje / Vse odbojnost / Vse indeks**. Vsaka možnost ima tri stanja: zelena ✓ = vklopljena za vsako kamero, ki jo podpira, rumena – = vklopljena za nekatere, siva = nobena. Preklopnik je onemogočen, če nobena priključena kamera ne podpira te vrste. Vsi so sivi, ko je vklopljena funkcija »Fastest Capture«.
* **Vrstice po posameznih kamerah**: potrditveno polje za vključitev ter razširljiv (▸/▾) seznam ustreznih tipov izvoza za to kamero s posameznimi potrditvenimi polji. Vrstica prikazuje število vključenih, na primer »4/6«.

### Vrste izvoza in katere kamere jih podpirajo

Obstaja šest vrst izvoza: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. V vrstici vsake kamere se prikažejo le tiste, ki so zanjo ustrezne:

| Vrsta izvoza | Vsebina | RGB (FRGB) | Bayerjev multispektralni (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Bayerjev mozaik (mono: en sam pas) neposredno iz senzorja | ✓ | ✓ | ✓ |
| **Debayered** | Linearni demosaik (mono: 1-kanalna sivinska lestvica) | ✓ | ✓ | ✓ |
| **Predogled** | Celotna veriga prikaza (belina + gama v skladu s profilom kamere; multispektralno: raztezek lažnih barv) | ✓ | ✓ | ✓ |
| **Sijaj** | float32 W/m²/sr/nm prek celotne radiometrične verige | — (ni na voljo) | ✓ | ✓ |
| **Odbojnost** | uint16 ρ (32768 = 1,0) | — (ni na voljo) | ✓ — prikazano le, če ima kamera svetlobni senzor DAQ (lastnega ali podedovanega iz niza) | enako kot pri multispektralnem |
| **Indeks** | upodabljanje vegetacijskega indeksa (LUT) | — | ✓ — zahteva omogočen, neprazen indeksni izraz na kameri in ni na voljo članom kombiniranega niza (niz ima en skupni indeks) | — (indeks potrebuje ≥2 pasove; glej [Monokamere in vegetacijski indeksi](mono-indices.md)) |

Sevanje in odbojnost nista nikoli na voljo za kamere RGB — sevanje po posameznih Bayerjevih pikah nima pomena za širokopasovni fotometrični senzor.

### Najhitrejše zajemanje

Preklopnik **⚡ Najhitrejše zajemanje — samo surovi podatki**(oranžen, ko je vklopljen) prevlada nad vsemi izbirami izvoza in nastavi**samo surove podatke** — poleg tega pa še brezplačno sestavljeno sliko s kombiniranim indeksom za matrice — tako da se posnetek shrani čim hitreje: izračuni sevanja/odbojnosti/prikaza se ob zajemanju v celoti preskočijo.

{% hint style="info" %}
**`.daq` se še vedno shrani.** Ko je dodeljen svetlobni senzor, funkcija »Najhitrejše zajemanje« še vedno zapiše vrednost navzdol usmerjenega sevanja DAQ poleg surovih slik – tako je mogoče kasneje s ponovno obdelavo ustvariti vse izdelke na podlagi sevanja, odbojnosti in indeksa (glej [Ponovna obdelava zajetih podatkov](#re-processing-captures-into-calibrated-products)). Program Fastest Capture tudi ne vpliva na vaše izbire v potrditvenih poljih: če ga izklopite, se te izbire ponovno prikažejo.
{% endhint %}

### Nadzorni elementi za posamezne matrice

Vsaka priključena matrica ima v oknu svojo skupinsko kartico:

* **Potrditveno polje »Vključi«** (tri stanja za posamezne člane) ter ime matrice z njenim načinom prikazovanja: »(združeno | ločeno)«.
* Polje za potrditev **Poravnano**(privzeto**vklopljeno**): izvozi elementov se prilagodijo profilu poravnave niza, tako da so izvozi po vseh kamerah poravnani na ravni pikslov. Surovi podatki ostanejo neizkrivljeni, vendar v svojih metapodatkih nosijo transformacijo. (Sam profil se izračuna v [oknu nastavitev niza](camera-settings.md#alignment-co-registration-combined-only).)
* Vrstice kamer članic so vgrajene znotraj kartice.

Kartica matrike gosti tudi dva snemalnika. Razumite ju kot **spremljanje proti analizi**:

| Snemalnik | Stopnja | Kaj snema |
| --- | --- | --- |
| **● Snemanje indeksnega videa / ■ Zaustavitev snemanja** *(samo pri kombiniranih nizih)* | **Spremljanje** | Sestavljeni indeksni posnetek v živo v formatu videa s 10 sličicami na sekundo — 8-bitni, ločljivost predogleda, vgrajena LUT. Zahteva odprt projekt in prenos v živo. Med snemanjem prikazuje posamezne slike + pretečen čas. |
| **⦿ Snemanje surove serije / ■ Ustavitev surove serije** *(kateri koli niz)* | **Analiza**| Surovi okvirji Bayerja s hitrostjo zajema v živo (brez obdelave) ter manifest za vsak okvir in odčitki `.daq`, shranjeni v `captures/bursts/`. Po seriji posnetkov se prikaže gumb**Ustvari video**: serijo posnetkov ponovno obdela v načinu brez povezave v kalibriran video — kombinirani indeks in/ali sevanje / odbojnost / indeks za vsako kamero posebej — ter po želji tudi datoteke TIFF. Ustvarjanje kombiniranega indeksa se samodejno začne, ko ustavite serijsko snemanje. |##

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

Postopek

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

»Capture All« (Zajemi vse) V stranskem seznamu kamer pritisnite **Capture All**:

1. Vsaka vključena, vidna in nezaustavljena kamera zajame posnetke z izbranimi tipi izvoza. **Nizi se sprožijo kot en sinhroniziran sprožilec** (ena sinhronizirana skupina za vse člane — glej [Nizi z več kamerami](arrays.md)); samostojne kamere snemajo posamično.
2. Skrite (oči) ali zaustavljene kamere se preskočijo. Niz je v celoti blokiran šele takrat, ko so *vsi* njegovi člani skriti ali zaustavljeni.
3. Kadarkoli je dodeljen svetlobni senzor, se ustrezna vrednost DAQ-odčitka navzdolnega sevanja shrani kot datoteka `.daq` skupaj s slikovnim gradivom — tudi pri zajemih, ki so izključno v surovi obliki — tako da je mogoče radiometrične produkte vedno izpeljati kasneje.
4. Gumb prikazuje napredek v realnem času — »Snemanje… končano/skupaj« — v načinu »Continuous« (neprekinjeno) ali »Interval« (intervalno) pa se spremeni v **Stop (N)**. Vsaka posamezna snemalna naloga ima časovno omejitev 300 s.
5. Ko se prelet konča, se v obvestilnem oknu prikaže **»Shranjenih N datotek«**ali**»Shranjenih N, F ni uspelo«**, skupaj z »(S skrito/zaustavljeno/preskočeno)«, če so bile kamere preskočene.

## Kje se shranijo zajeti podatki

Zajeti podatki se shranijo v odprtem projektu pod imenom `<project>/captures/`. Vsaka vrsta izvoza se shrani v **lastno podmapo**, tako da se pri večstopenjskem zajemanju vrste nikoli ne mešajo:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── *.daq          the downwelling reading matched to the capture
```

* `<ts>` je časovni žig zajema, `<serial>` pa serijska številka kamere. Samostojni posnetki so poimenovani `capture_<ts>_SN<serial>_<level>`; posnetki matrike iz enega sinhroniziranega sprožilca so poimenovani `sync_<ts>_SN<serial>_<level>` in **imajo enoten časovni žig za vse kamere v skupini** (pripona ravni se izpusti, če kamera shrani le eno raven).
* **Ena asimetrija, ki jo je treba poznati:** raven prikaza je shranjena v mapi z imenom `preview/`, medtem ko datoteke v imenu ohranijo `_display` — mapa in končnica se razlikujeta le za to raven.
* Neznane ravni se shranijo v mapo z lastnim imenom; če podmapo ni mogoče ustvariti, se datoteka zapiše v korensko mapo posnetkov, namesto da bi se izgubila.
* TIFF-datoteke posnetkov so privzeto stisnjene brez izgube kakovosti (DEFLATE) in vsebujejo vse metapodatke o kalibraciji in obdelavi **znotraj datoteke XMP** — posnetki so samopisni, brez stranskih datotek, razen datoteke `.daq`.

To je ista struktura, ki jo `chloros-cli lattice capture` / `array-capture` zapisujeta v svojo mapo `-o` — dokumentirano v [CLI Referenca § Kako izgleda mapa z zajetimi posnetki](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, a .daq file at the root, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Ponovna obdelava zajetih posnetkov v kalibrirane izdelke

Posnete surove slike in shranjeni `.daq` so vse, kar potrebuje obdelovalni proces — zato je »Fastest Capture« varen za dejansko delo.

* **GUI**: dodajte mapo z zajetimi posnetki v projekt ([Dodajanje datotek v projekt](../processing-images-gui/adding-files-to-a-project.md)) in obdelajte kot običajno.
* **CLI**: usmerite `process` na**korenno mapo posnetkov**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` običajno uvozi le mapo, ki jo imenujete, vendar če ta mapa ne vsebuje slik in ima podmap, se samodejno spusti v globino — tako se podmap na tej ravni in datoteke v korenski mapi `.daq` zajamejo v enem koraku. Vsak zajem se uvozi kot **ena slika**, pri čemer so druge ravni priložene kot načini prikaza, ne pa kot ena slika na raven.

Neposredno poimenovanje podmape ravni (npr. `…/captures/raw/`) prav tako deluje, vendar se pri tem izpustijo korenske datoteke `.daq` – te kopirajte hkrati, ko ponovno izpeljujete radiometrični produkt iz `raw/`, sicer se časovni žig ne bo ujemal z ničemer.

{% hint style="warning" %}
**Obdelava se vedno začne z `raw`.**Znotraj vsakega zajema je surovi okvir vir za obdelavo; `debayered`, `radiance`, `reflectance` in `preview` so na voljo kot načini prikaza, vendar se nikoli ne vrnejo v obdelovalno verigo — ponovna obdelava izpeljanega izdelka bi ponovno uporabila matematiko vinjetiranja, barv in svetlosti, ki je že vgrajena v njegove piksle, zato se Chloros zavrne, namesto da bi se izvedla dvojna obdelava. Renderji `index/` in `composite/` se sploh ne obdelujejo (gre za izhodne datoteke, ne za zajete datoteke). Mapa »captures« se shrani**brez** uvoza surovih podatkov in se normalno prikaže, vendar jo `process` preskoči in to tudi sporoči; `--input-level {raw,debayered,processed}` je namerno vgrajena varnostna rešitev, ki prisili vstopno točko. Glejte [CLI Referenco](../reference/cli-reference.md#what-a-captures-folder-looks-like) za natančna sporočila o preskoku.
{% endhint %}

Še dve lastnosti, ki jih je vredno poznati pri pisanju skriptov za ponovno obdelavo:

* Izvedba `chloros-cli process`, ki je zahtevala izdelke, vendar **ni zapisala nobenih slikovnih izdelkov, se očitno ne uspe in se zaključi z vrednostjo, ki ni nič** — nikoli ne boste dobili tihe prazne izvedbe. Uspešne izvedbe poročajo o številu izdelkov. (Namerno izvajanje, ki zajema samo metapodatke, še vedno šteje kot uspeh.)
* Ponovno uvoženi obdelani izvozi nikoli ne zasedajo prostora za surove podatke zajema — izvirni surovi podatki vedno ostanejo vir v poti obdelave.

## Ekvivalenti CLI

Vse na tej strani je mogoče upravljati brez grafičnega vmesnika. Načini zajema v grafičnem vmesniku se neposredno ujemajo z `chloros-cli lattice array-capture`:

| GUI | CLI |
| --- | --- |
| Enkratno | `chloros-cli lattice array-capture` |
| Neprekinjeno | `array-capture --continuous [--count N] [--duration S]` |
| Intervalno | `array-capture --interval S [--duration S]` |
| Najhitrejše zajemanje | `array-capture --fastest` |
| Poravnano potrditveno polje | `--aligned / --no-aligned` |
| Polja za izbiro vrste izvoza | `--processing LEVEL` ali `--levels L1,L2,…` (privzeto `all`) |
| Snemanje indeksnega videa | `chloros-cli lattice array-record` |
| Snemanje surove serije / Ustvarjanje videa | `chloros-cli lattice array-burst` / `array-build-video` |

Celotne tabele zastavic, možnost zajema s pametnim avtomatskim ostrenjem (`--smart`) ter model s stalno hitrostjo so opisani v [CLI Referenca § Načini zajema, snemalniki in ponovna obdelava v načinu brez povezave](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

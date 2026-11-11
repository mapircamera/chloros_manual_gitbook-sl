# Nastavitve in načini zajemanja

Zajemanje v zavihku »Kamere« poteka prek enega rdečega gumba **Zajemi vse**in enega okna**Nastavitve zajemanja**, ki določa, kaj ta gumb sproži: katere kamere sodelujejo, katere vrste izvoza shranjuje vsaka kamera ter ali se zaklop sproži enkrat, neprekinjeno ali v določenih intervalih. Ta stran opisuje celoten potek – konfiguracijo, sam zajem, kam se datoteke shranijo na disku in kako jih kasneje ponovno obdelati v kalibrirane izdelke. Nadzorne plošče za kamere in matrice so na strani [Nastavitve kamere](camera-settings.md).

{% hint style="info" %}
**Za zajem je potreben odprt projekt.** Možnosti »Posnemi vse« in ikona zobnika »Nastavitve zajema« sta onemogočeni, dokler ni odprt projekt („Ustvarite ali odprite projekt za shranjevanje posnetkov“). Vsak posnetek se shrani v projektni mapi v `captures/`.
{% endhint %}

## Okno »Nastavitve zajema«

Odprite ga z **zobčkom ob možnosti »Zajemi vse«**na stranskem seznamu kamer ali s gumbom**»Odpri nastavitve zajema…«** na dnu katerega koli okna z nastavitvami za posamezno kamero. Naslov okna je »Nastavitve zajema« z gumbom ← za vrnitev.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->



Vaše izbire tukaj – vključene kamere, potrditvena polja po tipu in način zajema – se shranijo **po posameznem projektu** in se obnovijo, ko ga ponovno odprete.

### Načini zajema

Trije gumbi za način na vrhu okna:

| Način | Kaj počne | Podnastavitve (privzete) |
| --- | --- | --- |
| **Enkratno** *(privzeto)* | En posnetek z vsemi izbranimi kamerami. | — |
| **Neprekinjeno**| Zaporedni posnetki, dokler ni izpolnjen pogoj za ustavitev. | Zaustavitev po**število posnetkov** (privzeto 1) *ali* **trajanje zajema** (privzeto 10 s; enote: sekunde / minute / ure / dnevi). |
| **Interval**(časovni preskok) | Serije posnetkov po časovniku. |**Število posnetkov / interval**(privzeto 1) ·**Vsake**N enot (privzeto 5 s) ·**Za** N enot (privzeto 1 m). |

V načinu Neprekinjeno ali Interval se gumb Posnemi vse med delovanjem spremeni v gumb **Zaustavi (N)**, ki šteje posnetke, ko se pojavijo. Poln pogon ustavi izvajanje (glej [Potek »Capture All«](#the-capture-all-flow)).**Neprekinjeno delovanje na nizih poteka znotraj backenda (novost v različici 1.3.0).** Ko vsaka kamera na seznamu pripada nizu (samostojne kamere so skrite ali izključene), način »Continuous« ne pošilja več ene zahteve na zajem: backend Chloros sam izvaja zanko zajema, pri čemer zapiše eno sinhronizirano skupino, medtem ko izbira naslednjo, gumb pa preverja število zajemov. Če so na seznamu samostojne kamere, Chloros zajema po posameznih prehodih, kot prej.

{% hint style="warning" %}
**Hitro zajemanje v sistemu Windows: izključite backend Chloros iz skeniranja programa Microsoft Defender.** Skeniranje v realnem času vsake datoteke, ki jo zapiše zajem z visoko hitrostjo, v testih MAPIR porabi približno 30 % hitrosti zajema (na stotine datotek na sekundo pri majhnem območju zanimanja). Namestitveni program za Windows ponuja možnost **»Izključi backend Chloros a iz skeniranja Microsoft Defenderja (precej hitrejše zajemanje s kamero; priporočeno)«**, ki je privzeto označena, izključitev pa se ob odstranitvi programa ponovno vključi. Če ste to polje odkljukali, ponovno zaženite namestitveni program z označeno možnostjo ali dodajte izključitev procesa v Defenderju za `chloros-backend.exe` (v mapi `backend` v namestitveni mapi Chloros). Ko Defender med neprekinjenim zajemanjem niza uporabi eno ali več jeder procesorja, backend Chloros to enkrat zabeleži v svojem dnevniku, skupaj z istima dvema rešitvama.
{% endhint %}

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->




### Izbiranje kamer in vrst izvoza

Pomočno besedilo v oknu to povzema: izberite, katere kamere in vrste izvoza bo ustvaril »Capture All« – privzeto je vse vklopljeno, izbire pa se shranijo skupaj s tem projektom.

* Gumbi **Izberi vse / Ne izberi nič** hkrati preklopijo potrditvena polja za vključitev vsake kamere.
* **Preklopniki za vrste izvoza v paketu**(dve vrstici gumbov):**Vse Raw / Vse Debayered / Vse Predogled / Vse Radiance / Vse Reflectance / Vse Index**. Vsaka možnost je označena s tremi barvami: zelena ✓ = vklopljena za vsako kamero, ki jo podpira, rumena – = vklopljena za nekatere, siva = nobena. Stikalo je onemogočeno, če nobena priključena kamera ne podpira te vrste. Vse so sive, ko je vklopljena funkcija »Fastest Capture«.
* **Vrstice po posameznih kamerah**: potrditveno polje za vključitev ter razširljiv (▸/▾) seznam ustreznih vrst izvoza za to kamero s posameznimi potrditvenimi polji. V vrstici je prikazano število vklopljenih, na primer »4/6«.

### Vrste izvoza in katere kamere jih podpirajo

Obstaja šest vrst izvoza: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. V vrstici vsake kamere se prikažejo le tiste, ki so zanjo ustrezne:

| Vrsta izvoza | Vsebina | RGB (FRGB) | Bayerjev multispektralni (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Bayerjev mozaik (mono: en sam pas) neposredno iz senzorja | ✓ | ✓ | ✓ |
| **Debayered** | Linearni demosaik: temni odmik (DSNU) iz kalibracijskega paketa je odstranjen iz surovega podatka, nato sledi demosaik z upoštevanjem robov — brez korekcije linearnosti, izravnave polja ali ločevanja barv. Shranjeno kot 3-kanalni RGB TIFF (mono: 1-kanalna siva lestvica) | ✓ | ✓ | ✓ |
| **Predogled** | Celotna veriga prikazovanja (belina + gama v skladu s profilom kamere, skupaj z nastavitvijo »Highlights« RGB-kamere; multispektralno: raztezek v lažnih barvah) | ✓ | ✓ | ✓ |
| **Sijaj** | float32 W/m²/sr/nm prek celotne radiometrične verige | — (ni na voljo) | ✓ | ✓ |
| **Odbojnost** | uint16 ρ (32768 = 1,0) | — (ni na voljo) | ✓ — prikazano le, če ima kamera svetlobni senzor DAQ (lastnega ali podedovanega iz matrike) | enako kot pri multispektralnem |
| **Indeks** | upodabljanje vegetacijskega indeksa (LUT) | — | ✓ — zahteva omogočen, neprazen indeksni izraz na kameri in ni na voljo članom kombiniranega niza (niz ima en skupni indeks) | — (indeks potrebuje ≥2 pasove; glej [Mono kamere in vegetacijski indeksi](mono-indices.md)) |

Sevanje in odbojnost nista nikoli na voljo za kamere RGB — sevanje po Bayerjevem vzorcu ni pomembno za širokopasovni fotometrični senzor.

### Najhitrejše zajemanje

Preklopnik **⚡ Najhitrejše zajemanje — samo surovi podatki**(oranžen, ko je vklopljen) prevlada nad vsemi izbirami izvoza in nastavi**samo surove podatke** — poleg tega pa še brezplačno sestavljeno sliko s kombiniranim indeksom za matrike — tako da se posnetek shrani čim hitreje: izračuni sevanja/odbojnosti/prikaza se ob zajemanju v celoti preskočijo.

{% hint style="info" %}
**`.daq` se še vedno shrani.** Ko je dodeljen svetlobni senzor, »Najhitrejše zajemanje« še vedno zapiše odčitek DAQ navzdol usmerjenega sevanja (v mapo `daq/` posnetkov) — tako se lahko produkti sevanja, odbojnosti in indeksa kasneje ustvarijo s ponovno obdelavo (glej [Ponovna obdelava zajemov](#re-processing-captures-into-calibrated-products)). Program Fastest Capture tudi ne vpliva na vaše izbire v potrditvenih poljih: če ga izklopite, se te izbire ponovno prikažejo.
{% endhint %}

### Nadzorni elementi za posamezne nize

Vsak priključen niz ima v oknu svojo skupinsko kartico:

* **Potrditveno polje »Vključi«** (tri stanja za posamezne člane) in ime niza z načinom prikaza: »(združeno | ločeno)«.
* Polje **Poravnano**(privzeto**vklopljeno**): izvozi elementov se prilagodijo profilu poravnave niza, tako da so izvozi med kamerami poravnani na ravni pikslov. Surovi podatki ostanejo neizkrivljeni, vendar v svojih metapodatkih nosijo transformacijo. (Sam profil se izračuna v [oknu nastavitev niza](camera-settings.md#alignment-co-registration).) Datoteke članov niza se shranijo v izvorni orientaciji vsake kamere, pri čemer se morebitne nastavitve zrcaljenja/vrtenja zabeležijo v datoteki, namesto da bi se uporabile.
* Vrstice kamer, ki so člani niza, so vgrajene znotraj kartice.

Kartica niza vsebuje tudi dva snemalnika. Razumite ju kot **spremljanje proti analizi**:

| Snemalnik | Stopnja | Kaj snema |
| --- | --- | --- |
| **● Snemanje indeksnega videa / ■ Zaustavitev snemanja** *(samo pri kombiniranih nizih)* | **Spremljanje** | Sestavljeni indeksni posnetek v živo v obliki videa s 10 sličicami na sekundo — 8-bitni, ločljivost predogleda, vgrajena LUT. Potreben je odprt projekt in prenos v živo. Med snemanjem prikazuje posamezne slike in pretečen čas. |
| **⦿ Snemanje surovih zaporednih posnetkov / ■ Ustavitev surovega zaporedja** *(kateri koli niz)* | **Analiza**| Surovi okvirji Bayerja s hitrostjo zajemanja v živo (brez obdelave) ter manifest za vsak okvir in odčitki `.daq`, shranjeni v `captures/bursts/`. Po seriji posnetkov se prikaže gumb**Ustvari video**: serijo posnetkov ponovno obdela v načinu brez povezave v kalibriran video — kombinirani indeks in/ali svetilnost / odbojnost / indeks po posamezni kameri — skupaj z izbirnimi datotekami TIFF. Ustvarjanje kombiniranega indeksa se samodejno začne, ko ustavite serijo posnetkov. |

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

## Potek

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

»Capture All«

V stranskem seznamu kamer pritisnite **Capture All**:

1. Vsaka vključena, vidna in nezaustavljena kamera zajema z izbranimi tipi izvoza. **Nizi se sprožijo kot en sinhroniziran sprožilec** (ena sinhronizirana skupina za vse člane — glej [Nizi z več kamerami](arrays.md)); samostojne kamere zajemajo posamično.
2. Skrite (z očesom označene) ali zaustavljene kamere se preskočijo. Niz je v celoti blokiran šele takrat, ko so *vsi* njegovi člani skriti ali zaustavljeni.
3. Kadarkoli je dodeljen svetlobni senzor, se ustrezna vrednost merjenja DAQ za padajoče sevanje shrani kot datoteka `.daq` v mapi posnetkov `daq/` — tudi za posnetke, ki so izključno v surovi obliki — tako da je mogoče radiometrične produkte vedno izpeljati kasneje.
4. Gumb prikazuje napredek v realnem času – »Zajem… končano/skupaj« – v načinu »Neprekinjeno/Interval« pa se spremeni v **Stop (N)**. Vsaka postavka zajema ima časovno omejitev 300 s.
5. Ko se prehod konča, se v poročilu o rezultatih prikaže **»Shranjenih N datotek«**ali**»Shranjenih N, F ni uspelo«**, skupaj z »(S skrito/zaustavljeno/preskočeno)«, če so bile kamere preskočene. Če se pogon zapolni, se izvajanje ustavi in prikaže sporočilo**»Ustavljeno: pogon poln. Shranjenih N, F ni uspelo«** – vsaka datoteka, ki je ni bilo mogoče zapisati, se šteje kot neuspešna.

## Kje se shranijo zajeti podatki

Zajeti podatki se shranijo v odprtem projektu pod imenom `<project>/captures/`. Vsaka vrsta izvoza se shrani v **lastno podmapo**, zato se pri večstopenjskem zajemanju vrste nikoli ne mešajo:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_clampmask.tif, reflectance/…, index/… — unmix clamp masks,
│                  one subfolder per product folder, only when a band was clamped
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── daq/           the downwelling reading(s) matched to each capture (.daq), each with a
                   .daq.cameras.json naming the cameras it describes
```

* `<ts>` je časovni žig zajema, `<serial>` pa serijska številka kamere. Samostojni posnetki so poimenovani `capture_<ts>_SN<serial>_<level>`; posnetki niza iz enega sinhroniziranega sprožilca so poimenovani `sync_<ts>_SN<serial>_<level>` in **imajo enoten časovni žig za vse kamere v skupini** (pripona ravni se izpusti, če kamera shrani le eno raven).
* **Ena asimetrija, ki jo je treba poznati:** raven prikaza se shrani v mapo z imenom `preview/`, medtem ko datoteke v imenu ohranijo `_display` — mapa in pripona se razlikujeta le za to raven.
* Neznane ravni se shranijo v mapo z njihovim imenom; če podmapo ni mogoče ustvariti, se datoteka zapiše v korensko mapo posnetkov, namesto da bi se izgubila.
* TIFF-datoteke posnetkov so privzeto stisnjene brez izgube (DEFLATE) in vsebujejo svoje popolne metapodatke o kalibraciji in obdelavi **znotraj XMP-datoteke** — zajeti podatki so samopisni, brez dodatnih datotek razen branja `.daq` (in njegovega `.daq.cameras.json`).
* **Maske omejevanja (novost v različici 1.3.0).**Kadar je bilo pri spektralnem ločevanju 3×3 treba pas omejiti na 0 na vsaj enem pikselu, dobijo produkti sevanja, odbojnosti in indeksa masko na piksel, `clamp_masks/<product folder>/<product name>_clampmask.tif` (bit 0 = pas v R-kanalu produkta, bit 1 = G, bit 2 = B); XMP-oznaka produkta `Chloros:UnmixClampMask` kaže nanjo. Maske se nahajajo**poleg** map s produkti, nikoli znotraj njih, zato je mogoče mapo s produkti, kot je `reflectance/`, v celoti predati programski opremi za fotogrametrijo. Lastni uvozni program Chloros maske preskoči.
* **Orientacija (1.3.0).** Produkti samostojne kamere, kot so »Preview«, »Radiance«, »Reflectance« in »Index«, se zavrtijo glede na nastavitev »Mirror/Rotation«; datoteki »Raw« in »Debayered« ostanejo takšni, kot jih je senzor prebral. Vse datoteke v nizu ostanejo v izvirni obliki. Vsaka datoteka shrani nastavitev v svojem XMP, zato se pri ponovni obdelavi ta uporabi na enak način.
* **Kateri kameri pripada `.daq` (1.3.0).** `.daq` posnetka se ob obdelavi ujemajo le s slikami iz kamer, ki jih imenuje `.daq.cameras.json`. Samostojna kamera brez lastnega svetlobnega senzorja zato ne uporablja več odčitkov druge kamere – ji dodajte senzor ali ga vnesite v zavihku Svetlobni senzorji. Zapis svetlobnega senzorja (in kateri koli `.daq` brez te datoteke, vključno s starejšimi zajemi) še vedno služi vsaki kameri.

To je ista struktura, ki jo datoteki `chloros-cli lattice capture` / `array-capture` zapišeta v svoj imenik `-o` — dokumentirano v [Referenca CLI § Kako izgleda mapa zajemov](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, the daq/ and clamp_masks/ folders, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Ponovna obdelava zajetih posnetkov v kalibrirane izdelke

Zajeti surovi okvirji skupaj s shranjenim `.daq` so vse, kar potrebuje obdelovalni proces – zato je »Fastest Capture« varen za dejansko delo.

* **GUI**: dodajte mapo z zajetimi posnetki v projekt ([Dodajanje datotek v projekt](../processing-images-gui/adding-files-to-a-project.md)) in obdelajte kot običajno.
* **CLI**: usmerite `process` na **koren mapo z zajetimi posnetki**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` običajno uvozi le mapo, ki jo imenujete, vendar če ta mapa ne vsebuje slik in ima podmap, se samodejno spusti v globino — tako se podmap na vseh ravneh in mapa `daq/` zajamejo v enem koraku. Vsak zajem se uvozi kot **ena slika**, pri čemer so druge ravni priložene kot načini pregleda, ne pa kot ena slika na raven.

Neposredno poimenovanje podmap ravni (npr. `…/captures/raw/`) prav tako deluje, vendar pri tem ostane mapa `daq/` izpuščena – kopirajte jo ob strani, koizpeljujete radiometrični produkt iz mape `raw/`, sicer se časovni žig ne bo mogel uskladiti z ničemer.

{% hint style="warning" %}
**Obdelava se vedno začne z mape `raw`.** Znotraj vsakega zajema je surovi okvir vir za obdelovalno verigo; `debayered`, `radiance`, `reflectance`, in `preview` se pojavijo kot načini prikaza, vendar se nikoli ne vrnejo skozi procesno verigo — ponovna obdelava izpeljanega izdelka bi ponovno uporabila vignetiranje, barvo in matematične izračune svetilnosti, ki so že vgrajeni v njegove piksle, zato Chloros to zavrne, namesto da bi izvedel dvojno obdelavo. Renderji `index/` in `composite/` se sploh ne obdelujejo (so izhodi, ne zajemi). Mapa »captures« se shrani **brez** uvoza surovih podatkov in se normalno prikaže, vendar jo `process` preskoči in to tudi sporoči; `--input-level {raw,debayered,processed}` je namerno vgrajena izhodna pot, ki prisili vstopno točko. Za natančna sporočila o preskočitvi glejte [Referenco CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like).
{% endhint %}

Še dve lastnosti, ki jih je vredno poznati pri pisanju skriptov za ponovno obdelavo:

* Izvedba `chloros-cli process`, ki je zahtevala izdelke, a ni zapisala **nobenega slikovnega izdelka, se glasno ne uspe in zaključi z izhodno vrednostjo, ki ni nič** — nikoli ne boste dobili tihe prazne izvedbe. Uspešne izvedbe poročajo o številu izdelkov. (Namerno izvajanje, ki zajema le metapodatke, še vedno šteje kot uspeh.)
* Ponovno uvoženi obdelani izvozi nikoli ne zasedajo prostora za surove podatke zajema — izvirni surovi podatki vedno ostanejo vir v poti obdelave.

## Ekvivalenti v CLI



Vse na tej strani je mogoče upravljati brez grafičnega vmesnika. Načini zajema v grafičnem vmesniku (GUI) se neposredno ujemajo z `chloros-cli lattice array-capture`:

| GUI | CLI |
| --- | --- |
| Enkratno | `chloros-cli lattice array-capture` |
| Neprekinjeno | `array-capture --continuous [--count N] [--duration S]` |
| Intervalno | `array-capture --interval S [--duration S]` |
| Najhitrejše zajemanje | `array-capture --fastest` |
| Polje za poravnavo | `--aligned / --no-aligned` |
| Polja za izbiro vrste izvoza | `--processing LEVEL` ali `--levels L1,L2,…` (privzeto `all`) |
| Snemanje indeksnega videa | `chloros-cli lattice array-record` |
| Snemanje surovih serij / Ustvarjanje videa | `chloros-cli lattice array-burst` / `array-build-video` |

Celotne tabele zastavic, možnost zajema s pametnim avtomatskim ostrenjem (`--smart`) in model s stalno hitrostjo so opisani v [Referenca CLI § Načini zajema, snemalniki in ponovna obdelava brez povezave](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

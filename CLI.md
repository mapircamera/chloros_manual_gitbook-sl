# CLI: ukazna vrstica

> **Celoten priročnik:**[Priročnik za CLI](reference/cli-reference.md) opisuje**vsak parameter vsakega podukaza** in je optimiziran za AI-pomočnike — vnesite njegov URL v svojega pomočnika in zaprosite za delujoč ukaz: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Nasvet za orodja z umetno inteligenco:** vsaka stran tega priročnika je na voljo v surovem formatu Markdown, če k njenemu URL-ju dodate `.md` (npr. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), `https://mapir.gitbook.io/chloros/llms.txt` pa indeksira celoten priročnik za uporabo v velikih jezikovnih modelih (LLM).

> **Želite, da vaš pomočnik sam zažene Chloros?** `chloros-mcp`, strežnik MCP, nameščen s Pythonovim SDK-jem, deluje s Claude Desktop, Claude Code, Cursor in drugimi MCP-odjemalci: glejte [Uporaba Chlorosa z AI-pomočniki](ai-assistants.md).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.3.0 build so the version line reads "Chloros CLI 1.3.0" -->


## Kaj je CLI


`chloros-cli` je vmesnik ukazne vrstice za isti procesni motor, ki ga uporablja namizna aplikacija Chloros
. Gre za preprost HTTP
-odjemalec, ki komunicira z Chloros
-zadnjim delom (lokalnim strežnikom na `127.0.0.1:5000`) — večina ukazov samodejno zažene zadnji del, zato skript potrebuje le en klic `chloros-cli process …`. Backend nato ostane zagnan za naslednji ukaz: ustavi se ob `chloros-cli logout` (razen če je aplikacija odprta ali če poteka izvajanje ali gradnja), ob `chloros-cli backend stop` ali samodejno po 30 minutah brez odjemalca, izvajanja ali povezane naprave. `chloros-cli backend status` to prikaže.

Deluje na **Windows
u 10/11 (x64)**in**Linux
u (x86_64 ter NVIDIA Jetson arm64 na JetPack 6)**, v katerem koli terminalu, brez potrebe po grafičnem vmesniku. Preverite namestitev z:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

Pregled skupin ukazov:

* **Obdelava in račun** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 jezikov — glej [Podprti jeziki](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (samo Linux
/Jetson)
* **Delujoča strojna oprema** — `lattice` (upravljanje kamere LATTICE, več kot 45 podukazov), `daq pool-*` (svetlobni senzorji DAQ), `time-sync` (PTP)
* **Avtomatizacija** — `project` (brez grafičnega vmesnika za zagon shranjenega projekta Chloros
, vključno z recepti za zajem v formatu YAML)

Globalne možnosti, ki jih je vredno poznati: `--port N` (vrata backenda, privzeto `5000`), `-v/--verbose`, `--restart` (prisilni ponovni zagon backenda), `--backend-exe PATH`. Celoten seznam najdete v [referenčnem priročniku za CLI
](reference/cli-reference.md).

***

## Namestitev

CLI
**je vključen v namestitveni program Chloros** na vseh platformah – ni ločenega prenosa CLI
. Namestitveni program prenesite s strani [Prenos](download.md).

### Windows


Namestitveni program namesti CLI
v:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

in enkrat doda to mapo v vaš sistem `PATH` (posodobitev odstrani tudi podvojene vnose, ki so jih pustili prejšnji namestitveni programi) — po namestitvi **odprite nov terminal**, da se zazna posodobljeni `PATH`. Namestitveni program v korensko mapo namestitve shrani tudi skripte za zagon (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) ter**Chloros
CLI
** v meniju Start, ki vsak odpre terminal z `chloros-cli`, pripravljenim za uporabo.

### Linux


Namestite `.deb` za svojo arhitekturo s pomočjo `apt` (ne `dpkg -i`, ki preskoči priporočene zahteve Python
SDK
):

```bash
# Linux x86_64
sudo apt install ./chloros_1.3.0_amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
```

S tem se namesti `chloros-cli` do `/usr/bin/chloros-cli` (že na `PATH`), backend in programe CLI
, razpakirane v `/usr/lib/chloros/backend/` in `/usr/lib/chloros/cli/` (`/usr/lib/chloros/chloros-backend` je zaganjalnik, ki zažene backend), izvedbeno okolje Arena SDK
, ki ga backend potrebuje za kamere LATTICE, ter Python
SDK
za sistem `python3`. Za podrobnosti glejte [Namestitev v sistemu Linux
](linux/linux-installation.md).

### Preverjanje

```bash
chloros-cli --version    # "Chloros CLI 1.3.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Prijava in licenciranje

Za dostop do CLI
(in Python
SDK
) je potreben **plačljiv načrt Chloros
+**— ta je na voljo v vseh plačljivih stopnjah; v brezplačni stopnji pa ni. Omejitev se izvaja**na strani strežnika** s strani backenda, ne pa s strani izvedljive datoteke CLI
: klic, ko niste prijavljeni, se zavrne s kodom `401 AUTH_REQUIRED`, klic v prijavljenem stanju na brezplačnem paketu pa z napako `403 PLAN_UPGRADE_REQUIRED`, ne glede na to, ali prihaja iz `chloros-cli`, SDK
-ja ali ročno izdelanega HTTP
-odjemalca. Nadgradite na [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Prijavite se **enkrat na računalnik**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.3.0 build showing the license tier line -->


{% hint style="warning" %}
**Gesla s posebnimi znaki**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` se v lupini poškoduje (CLI
to zazna ob napaki 401 in samodejno ponovi poskus, vendar se tej težavi v celoti izognete z uporabo enojnih narekovajev).
{% endhint %}

Sej je shranjena v predpomnilniku pod `~/.chloros/user_session.json` in deluje v načinu brez povezave še v obdobju odloga plačila (30 dni za mesečne naročnine, do izteka veljavnosti za letne). `chloros-cli status` deluje tudi brez plačanega načrta, zato je razlog za zavrnitev vedno viden.

{% hint style="danger" %}
**Načrtujete delo brez grafičnega vmesnika? Najprej se prijavite.**Ukazi za zagon strežniških procesov (`process`, `status`, `export-status`, …) brez**shranjene seje**v predpomnilniku ne preneha delovati takoj – namesto tega se prikaže interaktivna ukazna vrstica `Email:` / `Password:` na stdin. Avtomatizirana cron naloga ali korak CI se bo zato**zastal v čakanju na vnos**. Pred načrtovanjem česarkoli enkrat zaženite `chloros-cli login EMAIL 'PASSWORD'` na računalniku.
{% endhint %}

***

## Vaš prvi izvedbeni cikel

Usmerite `process` v mapo z zajetimi podatki — samodejno zazna Survey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` ali kombinacijo:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Napredek se prenaša v živo po posameznih niti cevovoda (zaznavanje, analiza, obdelava, izvoz), uspešno izvajanje pa se zaključi s poročilom o številu zapisnih slikovnih izdelkov: `Image products written: N`, čemur sledi `(plus M clamp masks)`, če je izvedba zapisala tudi [maske omejitve](output-image-formats.md#clamp-masks) (te se ne štejejo kot izdelki).

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N (plus M clamp masks)" summary line -->


Če je backend Chloros
že v teku (na primer v namizni aplikaciji), ga `process` uporabi in v njem odpre svoj projekt. Če je aplikacija odprta na projektu, ki ga je sama odprla, to zavrne: izvedite postopek iz aplikacije ali zaprite ta projekt. Če se prejšnji zagon še vedno zaključuje, počaka do 15 sekund, preden zavrne. `--restart` prisili zagon novega backenda. V sistemu Linux
se backend, ki ga izvaja storitev systemd, nikoli ne zagnati na novo: `process` ga uporabi takšnega, kakršen je, in tam, kjer bi ga sicer zagnal na novo (`--restart` ali uvoz, ki ni našel ničesar), izpiše `sudo systemctl restart <unit>` in se zaključi `1`.

### Kam se shranijo izhodni podatki

`process` piše v **projektno mapo**, ne v vašo mapo z vhodnimi podatki:

* Če ni `-o`: projekt se ustvari v vaši privzeti projektni mapi (skupni z grafičnim vmesnikom; upravljajte jo z `get-project-folder` / `set-project-folder`; če ni nastavljena, `Documents/MAPIR/Chloros Projects` v vaši uporabniški mapi), poimenovan z `-n/--project-name` ali časovnim žigom (`YYYYMMDD_HHMMSS`), če je izpuščen.
* Pri `-o PATH`: ta mapa **je** projektna mapa. Če že vsebuje `project.json`, se namesto prepisovanja ustvari `_1`/`_2`… namesto prepisovanja. Relativna pot `PATH` se določi glede na mapo, iz katere zaženete `chloros-cli`.

{% hint style="info" %}
**Relativne poti (spremenjeno v različici 1.3.0).** Vsaka pot, ki jo odpre backend — `-o`/`--output` pri katerem koli ukazu (vključno s privzetim `output` za ukaze zajemanja), `--input`, `--burst-dir`, `--profile`, `--save` in `--mask` — se razreši glede na mapo, iz katere zaženete `chloros-cli`, tako da `-o captures/` piše v `<current folder>/captures/`. V različici 1.2.0 jih je strežnik razrešil v svoji lastni namestitveni mapi, kar je povzročilo napako »Dostop je zavrnjen«. Pri oddaljenem strežniku (`CHLOROS_BACKEND_URL`) se poti pošljejo takšne, kot so vnesene, in se razrešijo na tem računalniku.
{% endhint %}

{% hint style="warning" %}
**Windows
: imenik projekta naj bo kratek, razen če so omogočene dolge poti.** Če so v Windows
u dolge poti onemogočene (privzeta nastavitev Windows
a), nobena pot do datoteke ne sme biti daljša od 260 znakov, izdelki pa se nahajajo do približno 130 znakov pod imenikom projekta. `process` to preveri pred zagonom: pregloboka projektna mapa se zavrne z napako **»Projektna mapa je pregloboka za ta računalnik«**, najdaljša projektna mapa, ki ustreza, in navodila za vklop dolgih poti – nič se ne obdela. Uporabite krajšo različico `-o` (ali `set-project-folder`) ali vklopite dolge poti v sistemu Windows
(možnost namestitvenega programa **Omogoči dolge poti v sistemu Windows**, ki je privzeto izbrana) in ponovno zaženite program Chloros
.
{% endhint %}

Znotraj projekta so izdelki razvrščeni **po kameri, nato pa po formatu datoteke**:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Mapa kamere je `LATT-<sensor>-<lens>-F<filter>` za LATTICE (kar ustreza EXIF-podatkom posnetka `Model`) in `<model>_<filter>` (npr. `Survey3N_RGN`) za Survey3
. Mape po formatu sledijo `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` ali `tiff32` za `TIFF (32-bit, Percent)`.

Pri multispektralni kameri M3C se v mapi formata poleg map s produkti lahko pojavi tudi mapa `clamp_masks/` (na primer `tiff16/clamp_masks/Reflectance_Calibrated_Images/` in `tiff32/clamp_masks/Radiance_Images/`). V njej so shranjene maske sevanja in odbojnosti na piksel za produkte, pri katerih je razmešavanje pasov omejilo negativno vrednost na nič — glej [Maske omejitve](output-image-formats.md#clamp-masks). Nobena mapa z izdelki nikoli ne vsebuje maske.

{% hint style="info" %}
**Vsak izvožen produkt ohrani ime izvorne datoteke.**Izvoz sevanja iz `capture_..._raw.tif` se še vedno imenuje `capture_..._raw.tif` — le da se nahaja v `tiff32/Radiance_Images/`.**Mapa identificira izdelek, ne pa ime datoteke**, zato uporabite glob za mapo, ne pa za končnico `*radiance*`.
{% endhint %}

### Možnosti, ki jih boste dejansko uporabljali

| Oznaka | Privzeto | Kaj počne |
| --- | --- | --- |
| `-o, --output PATH` | privzeta mapa projekta | Mesto mape projekta (glej zgoraj). |
| `-n, --project-name NAME` | časovni žig | Ime projekta. |
| `--format FMT` | `TIFF (16-bit)` | Ena od naslednjih: `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros
ne povečuje ločljivosti: na vseh 8-bitnih slikah se globlji format zapiše kot `TIFF (8-bit)`, kar je navedeno enkrat. |
| `--indices NAME [NAME ...]` | noben | Indeksi vegetacije za izvoz (glej [Indeksi vegetacije](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = nevronski debayer, počasnejši, najvišja kakovost (Chloros
+, potrebuje CUDA GPU: brez njega se uporabi `standard`; zavrnjeno za posnetke LATTICE, ki vedno dobijo `standard`). |
| `--vignette / --no-vignette` | vklopljeno | Popravek vinjetiranja. |
| `--reflectance / --no-reflectance` | vklopljeno | Kalibracija odbojnosti; za LATTICE je to tudi stikalo za produkt odbojnosti. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Prisilna nastavitev vstopne točke procesne verige za TIFF-datoteke LATTICE. |

Za vse ostalo — nastavitev zaznavanja ciljev, PPK, zatiči osvetlitve, zastavice poravnave matrik — glejte [poglavje `process` v referenčnem priročniku za CLI
](reference/cli-reference.md).

***

## Izbiranje, kaj izvoziti (izdelki LATTICE)

Obdelava LATTICE se v enem prehodu razširi na **vse ustrezne izdelke**. Štiri stikala za posamezne izdelke so**privzeto vklopljena**; uporabite obrazec `--no-`, da eno izklopite:

| Stikalo | Izdelek |
| --- | --- |
| `--debayered` | Linearni demosaic → `Debayered_Images/`: odstranjen temni odmik kalibracijskega paketa, nato demosaik z upoštevanjem robov, brez korekcije linearnosti, izravnave polja ali ločevanja barv — ista datoteka, kot jo zajem shrani na svoji ravni `debayered`. Ohranjena v izvorni orientaciji senzorja (zabeležena v XMP). |
| `--preview` | Predogled na zaslonu (balanca beline + gama; raztezek v lažnih barvah za multispektralne posnetke) → `Preview_Images/` |
| `--radiance` | float32 sevanje, W/m²/sr/nm → `Radiance_Images/` (vedno `tiff32/`) |
| `--reflectance` | uint16 odbojnost, pripravljeno za Pix4D → `Reflectance_Calibrated_Images/` |

RGB
-glavne kamere vedno oddajajo le podatke po debayeringu + predogled — sevanje/odbojnost po posameznih pasovih za širokopasovni senzor nima pomena, zato ti preklopniki zanje ne delujejo. Survey3
`.raw` prezre preklopnike in sledi standardni poti odbojnosti/cilja.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (privzeto `auto`) izbere referenco odbojnosti: `auto` določi [kalibracijsko tarčo](calibration-targets.md) znotraj okvirja, ki je uspešno prestala kontrolo kakovosti (QA), za absolutno referenco in se v primeru odsotnosti tarče zateče k razdelitvi navzdol usmerjenega svetlobnega toka senzorja DAQ (ρ = π·L/E), kadar cilj ni prisoten (cilj, ki opravi kontrolo kakovosti z označbo, tekmuje z DAQ glede na izmerjeno oceno); `target` je strog (brez zamenjave z DAQ); `daq` je avtoritativen za DAQ. Skeniranja ciljev, izmerjena na enoto, se lahko zagotovijo z `--target-reflectance-dir`.

{% hint style="info" %}
**Branje pik odbojnosti:**DN, ki pomeni ρ = 1,0, je**na vir** — datoteke LATTICE v XMP vstavijo oznako `Chloros:PixelScale=32768`; datoteke Survey3
uporabljajo vrednost 65535 in ne vsebujejo oznake `Chloros:PixelScale` (vsebujejo pa druge oznake obdelave `Chloros:*`). Preberite oznako in delite z njo, namesto da predpostavljate konstanto. Podrobnosti in en nameren mejni primer brez merila so v [Referenca CLI
](reference/cli-reference.md).
{% endhint %}

**Obdelava se vedno začne z `raw`.** Izpeljani izdelki (izvozi brez debayeringa/svetilnosti/odbojnosti) se nikoli ne vračajo skozi obdelovalno verigo — njihov ponovni uvoz in obdelava bi pomenila dvojno uporabo kalibracijskih izračunov, zato jih Chloros
preskoči in to tudi navede. `--input-level` je namerno vgrajena izhodna pot, kadar resnično potrebujete prisilno določiti začetno točko.

***

## Ko obdelava ne uspe

Od različice 1.2.0 naprej `process` glasno poroča o napaki, namesto da bi »uspel« brez prikazanih rezultatov:

* Izvedba, ki je **zahtevala izdelke, a ni zapisala nobenega**— le `project.json` in `calibration_data.json` — izpiše `Processing finished but wrote no image products.` in**se zaključi z vrednostjo, ki ni nič**, tako da ga skripti lahko zaznajo. Običajni vzroki: vhodna mapa ni bila prepoznana kot zajem (preverite razpored in `--input-level`) ali pa noben od zahtevanih produktov ni bil uporaben za te kamere (npr. zahteva po sevanju/odbojnosti iz kamer, ki podpirajo le RGB
).
* **Namerno izvajanje samo z metapodatki** (vsi izdelki izklopljeni, brez `--indices`) je še vedno uspešno — prazen izhodni slikovni zapis je v tem primeru pravilen rezultat.
* Ponovno zaženite z `--verbose` in v dnevniku strežnika preverite vrstice `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, ki pojasnjujejo preskoke po posameznih kamerah.
* V sistemu Windows
, kjer so dolge poti onemogočene, se projektna mapa, ki je pregloboka za omejitev poti na 260 znakov, zavrne **pred** začetkom obdelave, pri čemer se uporabi najdaljša projektna mapa, ki ustreza omejitvi (glej [Kje se shranijo izhodni podatki](#where-the-outputs-land)).
* Ukazi za zajem (`lattice capture`, `lattice array-capture` in druge družine ukazov za zajem) navedejo vsako datoteko, ki je niso mogli zapisati, in zaključijo z `1`; ponavljajoči se zajemi se ustavijo, ko je pogon poln.

Kode izhoda: `0` uspeh · `1` splošna napaka · `2` napaka v argumentu · `130` prekinjeno s Ctrl+C.

***

## Indeksi vegetacije

Program `--indices` zaženite z enim ali več prednastavljenimi imeni; vsak indeks se shrani v svojo mapo `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI GNDVI OSAVI
```

22 vnaprej nastavljenih imen, ki jih sprejema `process --indices`:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

Projektu se doda ime, ki v njem še ni navedeno. Ime, ki ni vnaprej nastavljeno ali ga pasovi kamer ne morejo ustvariti, se preskoči z opozorilom, v katerem je navedeno. Če CLI
-je ne more prebrati trenutnega indeksnega seznama projekta, seznama ne spremeni in namesto zamenjave izda opozorilo.

{% hint style="warning" %}
**Obstajajo trije indeksni seznami – ne zamenjujte jih.**Spustni meni »Nastavitve projekta« v grafičnem vmesniku vsebuje 27 formul (dodaja `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` – teh pet je na voljo le v grafičnem vmesniku in**ne** velja za `--indices`). Ukaz `lattice index --preset` za delo v realnem času/offline uporablja svoj ločen seznam 22 prednastavitev. Formule in matematični izračuni za pasove so dokumentirani v [Formulah za multispektralne indekse](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Svetlobni senzorji DAQ: Kratek pregled

Družina `daq pool-*` upravlja spektralne senzorje DAQ podjetjMAPIR
(DAQ-U prek USB, DAQ-M prek BLE, DAQ-E prek Etherneta) prek trajnega sklopa v ozadju — grafični vmesnik (GUI), vmesnik ukazne vrstice (CLI
) in razvojni komplet (SDK
) si delijo eno aktivno ročajno. **`pool-*` je podprta pot DAQ v priloženem CLI
**; drugi podukazi `daq`, na katere se morda sklicujete, so le notranja površina MAPIR
, namenjena izključno virom, in se zaključijo z izrecno napako, ki vas usmerja k `pool-*`.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record` brez `--duration` teče do `pool-record --stop`; privzeti izhodni imenik je `~/Documents/DAQ Live View/` **na računalniku backenda**. Profil korekcije kapacitivnosti se izbere ob vzpostavitvi povezave (`--cap-id`, privzeto v backendu `sunshine_cosine`) in ga je mogoče med delovanjem zamenjati s `pool-set-cap` — profili omejitve in kalibrirano območje senzorja so obravnavani v poglavjih o DAQ v tem priročniku.

{% hint style="warning" %}
**DAQ-E na gostitelju z več mrežnimi karticami:** prvo samodejno odkrivanje `pool-connect --eth` po zagonu lahko spodleti, tudi če je senzor v dobrem stanju. `--eth-host <ip-or-hostname>` je zanesljiva oblika — uporabite jo, kadar odkrivanje ne prinese rezultatov.
{% endhint %}

***

## Kamere LATTICE, PTP in avtomatizacija projektov

Družina `lattice` (več kot 45 podukazov) pokriva delo s kamerami LATTICE od začetka do konca: odkrivanje, posamezne zajeme, trajne sinhronizirane matrike s tokom povezovanja »smart-prep« v grafičnem vmesniku, predogled v živo v brskalniku, poravnavo, indeksno matematiko in diagnostiko gostitelja-mrežne kartice. Primer:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Poleg tega: `chloros-cli time-sync` poroča o glavnem PTP-strežniku, ki ga izvaja gostitelj Chloros
(kamere LATTICE in senzorji DAQ-E so mu podrejeni za časovne oznake med napravami), `chloros-cli project` pa odpre shranjen projekt Chloros
in brez grafičnega vmesnika upravlja njegove kamere, matrike in senzorje – vključno s skriptiranimi recepti za zajem v formatu YAML.

Te tri družine (`lattice`, `project`, `daq pool-*`) so tudi edine, ki podpirajo `CHLOROS_BACKEND_URL` za upravljanje **oddaljenega** backenda; osnovni ukazi so vedno usmerjeni na lokalni računalnik.

Celotni navodili so na voljo v poglavjih o LATTICE v tem priročniku; vsi argumenti so navedeni v [Referenca CLI
](reference/cli-reference.md).

***

## Odpravljanje težav: Top 5

| Simptom | Rešitev |
| --- | --- |
| `Login required` ali načrtovana naloga se zatakne pri pozivu `Email:` | Na tem računalniku enkrat zaženite `chloros-cli login EMAIL 'PASSWORD'` — ukazi brez shranjene seje se izvajajo interaktivno, namesto da bi se takoj prekinili. |
| `backend unreachable` | Zaženite namizno aplikacijo Chloros
ali sami zaženite backend: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` v sistemu Windows
(mapa `backend` v vaši namestitveni mapi; ni v `PATH`), `chloros-backend` v sistemu Linux
. `chloros-cli backend status` prikazuje, ali kateri od njih teče; `lattice`/`project`/`daq pool-*` ga nikoli ne zaženejo. Če jih usmerite na oddaljeni backend, preverite `CHLOROS_BACKEND_URL`. |
| Povezava z nizom blokirana: `FRAMES WILL DROP` / `Reduce ROI to enable` | Prejemni obroč omrežne kartice gostitelja je ponastavljen na privzete vrednosti — to je najpogostejši vzrok, da se naprava, ki je prej delovala, ne želi več povezati, običajno po posodobitvi gonilnika omrežne kartice. Zaženite `chloros-cli lattice network --fix` iz terminala z **povečanimi pravicami** (ali nastavite `ReceiveBufferLen=256`, `PendingReceives=64`); glejte referenčni dokument *Nastavitev in optimizacija omrežne kartice gostitelja*. |
| Podukaz `daq` se zapre: »zahteva celoten DAQ paket…« | Uporabite ukaze `daq pool-*`, ki prek backenda upravljajo isti senzor: povezovanje, pretakanje, snemanje, izbira omejitve ter za DAQ-E njegove omrežne nastavitve, ponovni zagon in posodobitve firmware-a (ali `chloros_sdk.connect_daq_sensor()` iz Python
a). |
| Jetson pred velikimi mapami prikaže opozorilo o zamenjavi | Dodajte zamenjavo, podprto s datotekami — CLI
prikaže natančne ukaze `fallocate`/`swapon`, ki jih je treba izvesti. |

***

## Pomoč

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Vsak zastavici, vsak podukaz:** [Referenca za CLI
](reference/cli-reference.md)
* **Python
ov ekvivalent:** [Python
SDK
](api-python-sdk.md) in [Referenca za SDK
](reference/sdk-reference.md)
* **Podpora:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)

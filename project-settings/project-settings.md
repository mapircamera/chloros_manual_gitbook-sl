# Nastavitve projekta

Stranski pas »Nastavitve projekta« (<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

) v programu Chloros omogoča konfiguracijo vseh vidikov obdelave slik, zaznavanja kalibracijskih ciljev, izračunov multispektralnih indeksov ter možnosti izvoza za vaš projekt. Te nastavitve se shranijo skupaj z vašim projektom in jih je mogoče shraniti kot predloge za ponovno uporabo v več projektih.

## Dostop do nastavitev projekta

Za dostop do nastavitev projekta:

1. Odprite projekt v programu Chloros
2. V levem stranskem meniju kliknite zavihek **Nastavitve projekta**<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">


3. V oknu z nastavitvami se prikažejo vse razpoložljive možnosti konfiguracije, razvrščene po kategorijah

<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->

{% hint style="info" %}
**Nastavitve, ki so odvisne od drugih nastavitev, so sive.** Kadar nadrejeno stikalo onemogoča določeno nastavitev (na primer, če odkljukate *Kalibracija odbojnosti / bela barva*, je *Izvoz odbojnosti* nemogoč), je odvisni nadzorni element onemogočen, v njegovem orodnem namigu pa je navedeno stikalo, ki ga je treba spremeniti.
{% endhint %}

***

## Prikaz

### Ločljivost sličic

* **Vrsta**: Izbirni seznam
* **Možnosti**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Privzeto**: Privzeto (512 px)
* **Opis**: Ločljivost (najdaljša stranica, v slikovnih točkah), pri kateri se prikazujejo sličice v mreži slik. Višje vrednosti so pri povečanju videti ostrejše, vendar se nalagajo počasneje in porabijo več pomnilnika. Polna ločljivost ustreza originalni velikosti slike.
* **Opomba**: Samo za prikaz — to nikoli ne vpliva na obdelavo ali izvožene datoteke.***

## Zaznavanje ciljev

Te nastavitve uravnavajo, kako Chloros zazna in obdeluje kalibracijske cilje na vaših slikah. Obe sta aktivni le, ko je omogočena **kalibracija odbojnosti / beline** (v nasprotnem primeru sta sive, saj se zaznavanje ciljev v celoti preskoči).

### Najmanjša površina kalibracijskega vzorca (px)

* **Vrsta**: Število
* **Območje**: od 0 do 10.000 pikslov
* **Privzeto**: 25 pikslov
* **Opis**: Določa najmanjšo površino (v pikslah), ki je potrebna, da se zaznano območje šteje za veljaven vzorec kalibracijske tarče. Manjše vrednosti bodo zaznale manjše tarče, vendar lahko povečajo število lažnih pozitivnih rezultatov. Večje vrednosti zahtevajo večja in jasnejša območja tarč za zaznavanje.
* **Kdaj prilagoditi**:
  * Povečajte vrednost, če se pojavljajo lažna zaznavanja na majhnih artefaktih na sliki
  * Zmanjšajte vrednost, če se vaši kalibracijski cilji na slikah zdijo majhni in se ne zaznavajo

### Minimalno združevanje ciljev (0–100)

* **Vrsta**: Število
* **Območje**: od 0 do 100
* **Privzeto**: 60
* **Opis**: Uravnava prag združevanja za razvrščanje podobno obarvanih območij v skupine pri zaznavanju kalibracijskih ciljev. Višje vrednosti zahtevajo, da se v skupino združijo bolj podobne barve, kar povzroči bolj konzervativno zaznavanje ciljev. Nižje vrednosti dopuščajo večje barvne razlike znotraj skupine ciljev.
* **Kdaj prilagoditi**:
  * Povečajte vrednost, če se kalibracijski cilji razdelijo na več zaznav
  * Zmanjšajte vrednost, če se kalibracijski cilji z barvnimi odstopanji ne zaznavajo v celoti

***

## Obdelava

Te nastavitve določajo, kako Chloros obdeluje in kalibrira vaše slike.

### Popravljanje vinjetiranja

* **Vrsta**: Potrditveno polje
* **Privzeto**: Omogočeno (potrjeno)
* **Opis**: Uporabi popravek vinjetiranja za kompenzacijo potemnitve objektiva na robovih slik. Vinjetiranje je pogost optični pojav, pri katerem so koti in robovi slike zaradi lastnosti objektiva temnejši od sredine.
* **Stranski učinek**: Ta stikalo izbere tudi, kateri *nekalibrirani nadomestni izdelek* se zapiše ob izvedbi (glej spodaj).

### Kalibracija odbojnosti / bela bilanca

* **Vrsta**: Potrditveno polje
* **Privzeto**: Omogočeno (označeno)
* **Opis**: Omogoča kalibracijo odbojnosti — na podlagi zaznanih kalibracijskih ciljev v okviru in/ali podatkov o navzdol usmerjenem svetlobnem toku senzorja DAQ, odvisno od kamere in tega, kar je na voljo. To normalizira vrednosti odbojnosti v vašem nizu podatkov in zagotavlja dosledne meritve ne glede na svetlobne razmere.
* **Ko je onemogočeno**: Zaznavanje ciljev se v celoti preskoči in**nobena kamera ne more ustvariti izdelka odbojnosti** — niti Survey3, ki temelji na ciljih, niti LATTICE, ki temelji na DAQ. Odvisne nastavitve (*Izvoz odbojnosti*, *Minimalni interval ponovne kalibracije* in pragovi za zaznavanje ciljev) so sive barve.

### Nekalibrirani nadomestni produkti: Izvoz odziva senzorja / Izvoz popravljene vinjete

* **Vrsta**: Dve potrditveni polji
* **Privzete nastavitve**: Obe sta omogočeni (potrjeni)
* **Opis**: Ko okvirja ni mogoče kalibrirati glede na odbojnost (ni bil najden kalibracijski cilj ali je kalibracija odbojnosti izklopljena), se namesto tega zapiše kot *nekalibriran nadomestni izdelek*. **Za vsak model kamere obstaja natanko en od dveh nadomestnih izdelkov na posnetek**, izbran s stikalom *Popravek vinjete*:
  * Popravek vinjetiranja **vklopljen**→ `Vignette_Corrected_Images/` (uravnava ga**Izvoz s popravljenim vinjetiranjem**)
  * Popravek vinjetiranja **izklopljen**→ `Sensor_Response_Images/` (ureja ga nastavitev**Izvoz odziva senzorja**)
* Nadomestni izdelek, ki ni v uporabi, je sivo obarvan. Če odkljukate tistega, ki je v uporabi, se ta datoteka sploh ne zapiše.

### Izdelki za izvoz LATTICE

Pri projektih, ki vsebujejo posnetke LATTICE, se vsak uvožen okvir LATTICE v enem samem obdelovalnem ciklu razširi na vse omogočene **in ustrezne**izdelke. Razširjanje nadzorujejo štiri potrditvena polja (vsa so privzeto**vklopljena**):

| Nastavitev | Izhodna mapa | Kaj izvozi |
| --- | --- | --- |
| **Izvoz brez bayera** | `Debayered_Images/` | Linearna slika brez bayera. Velja za RGB in multispektralne kamere. |
| **Izvoz predogleda** | `Preview_Images/` | Predogled na zaslonu. RGB = bela bilanca (DAQ-svetlobni vir, če je na voljo, sicer siva skala) + gama; multispektralno = raztezanje v lažnih barvah. |
| **Izvoz sevanja** | `Radiance_Images/` | Spektralno sevanje v obliki Float32 v W/m²/sr/nm. Samo multispektralno (M3C/M3M) — ne velja za matrice RGB. Vedno se zapiše kot 32-bitni TIFF ne glede na nastavitev *Kalibrirani format slike*. |
| **Reflektanca izvoza**| `Reflectance_Calibrated_Images/` | Reflektanca Uint16, skalirana tako, da**32768 = reflektanca 1,0** (označeno kot XMP `Chloros:PixelScale`). Samo za multispektralne slike, zapiše se, kadar ustrezen zapis `.daq` o padajočem sevanju (ali cilj v okviru, ki je uspešno prestal kontrolo kakovosti) pokriva okvir. |

* Glavne kamere RGB oddajajo sliko brez bayersovega filtra + predogled; sevanje/odbojnost se za njih preskočita, ker nista uporabna.
* Bitna globina podatkov brez bayersovega filtra/predogleda sledi nastavitvi *Kalibrirani format slike*; sevanje je vedno float32.
* Te štiri stikala ne vplivajo na obdelavo Survey3.

Iste štiri stikala obstajajo tudi brez imena kot `chloros-cli process --debayered / --preview / --radiance / --reflectance` in kot ustrezni parametri SDK. Nadomestila so staro zastavico `--radiometric-output`, ki ne obstaja več.

{% hint style="warning" %}
**Če izklopite vse ustrezne produkte, bo obdelava spodletela.** Od različice 1.2.0 naprej obdelava, ki je zahtevala produkte, a ni zapisala nobenega slikovnega produkta, poroča o napaki, CLI pa se zaključi z vrednostjo, ki ni nič, namesto da bi poročal o tihem uspehu. Dnevnik navaja izdelek, ki ga ni bilo mogoče zapisati, in razlog za to. Namerno izvedba, namenjena izključno metapodatkom (brez zahtev), je še vedno uspešna.
{% endhint %}

### Vir odbojnosti (nastavitev projekta, nastavljena prek CLI/SDK)

Projekt shranjuje tudi, katero **referenco odbojnosti** uporablja produkt odbojnosti LATTICE. V oknu nastavitev ni posebnega upravljalnika; vrednost je shranjena v konfiguraciji projekta kot `Processing → "Target reflectance source"` in se nastavi s parametrom `chloros-cli process --reflectance-source {auto,target,daq}` ali`reflectance_source`:

* **`auto`** (privzeto): cilj kalibracije v okviru, ki je uspešno prestal kontrolo kakovosti (QA),postane absolutna referenca, pri čemer se v primeru odsotnosti cilja ali neuspešnega QA uporabi nadomestna vrednost DAQ za delitev navzdolnega sevanja (ρ = πL/E).
* **`target`**: stroga odbojnost, ki temelji na cilju — brez nadomestitve s strani DAQ.
* **`daq`**: odbojnost, ki jo določa DAQ; cilji znotraj okvirja se ne uporabljajo kot referenca.

Shranjena vrednost se primerja brez upoštevanja velikosti črk, nekaj različnih zapisov pa se sprejme kot sinonimi: `target`, `target_image`, `empirical` in `empirical_line` vsi pomenijo **cilj**; `daq`, `dls`, `light_sensor` in `sensor` vsi pomenijo**daq**. Vse ostalo — vključno z manjkajočim ključem — se razreši kot**auto**.**Izmerjene** skeniranje ciljev po enotah se poiščejo po serijski številki/QR-kodi ciljne enote, na primer `<serial>.csv`, na treh mestih: v imeniku, navedenem z `--target-reflectance-dir` (shranjeno kot `Processing → "Target reflectance dir"`), v lastni mapi projekta `target_reflectance/` ter v poti, določeni v okoljski spremenljivki `CHLOROS_TARGET_REFLECTANCE_DIR`. Če za to enoto ni na voljo nobenega izmerjenega skena, se namesto tega uporabi nominalna objavljena krivulja za ciljni model.

### Metoda odstranjevanja pik

* **Vrsta**: Izbirni seznam
* **Možnosti**:
  * Standardna (hitra, srednja kakovost)
  * Upošteva teksturo (počasna, najvišja kakovost) \[Chloros+]
* **Privzeto**: Standardno (hitro, srednja kakovost)
* **Opis**: Izbere algoritem demosaicinga, ki se uporablja za pretvorbo surovih podatkov senzorja z vzorcem Bayerja v barvne slike. Metoda »Standardna (hitra, srednja kakovost)« zagotavlja optimalno ravnovesje med hitrostjo obdelave in kakovostjo slike. Metoda »Upošteva teksturo (počasna, najvišja kakovost)« \[Chloros+] uporablja visokokakovostno demosaicing obdelavo, ki upošteva robove, v kombinaciji z modelom za odstranjevanje šuma na podlagi umetne inteligence (AI/ML), ki odstrani skoraj ves šum, nastal pri demosaicingu. Model »Texture Aware« za delovanje potrebuje pomnilnik grafične kartice (VRAM). Za hitrejšo obdelavo priporočamo njegovo uporabo, če imate na voljo več kot 4 GB VRAM.
* **Če je vrstica sploh spustni seznam**: spustni seznam z dvema možnostma se prikaže le, če veljata**obe**pogoji – ste prijavljeni z ustreznim naročniškim paketom Chloros+,**in** projekt ne vsebuje posnetkov LATTICE. V nasprotnem primeru se vrstica prikaže kot navadno besedilo z besedilom `Standard (Fast, Medium Quality)`, brez možnosti izbire.
* **Opomba o LATTICE**: Ni modela »Texture Aware«, usposobljenega za LATTICE, zato postopek za okvirje LATTICE prisili standardno demosaiko, ne glede na shranjeno vrednost. Če dodate mapo LATTICE v projekt, v katerem je bila že izbrana možnost »Texture Aware«, Chloros ponovno nastavi vrednost na »Standard«, namesto da bi v `project.json` pustil zastarelo vrednost.

### Najkrajši interval ponovne kalibracije

* **Vrsta**: Število
* **Območje**: od 0 do 3.600 sekund
* **Privzeto**: 0 sekund
* **Opis**: Določa najkrajši časovni interval (v sekundah) med uporabo kalibracijskih ciljev. Če je nastavljen na 0, bo Chloros uporabil vsak zaznan kalibracijski cilj. Če je nastavljena višja vrednost, bo Chloros uporabil le kalibracijske tarče, ki so med seboj oddaljene vsaj toliko sekund, s čimer se skrajša čas obdelave za nize podatkov s pogostim zajemanjem kalibracijskih tarč.
* **Kdaj prilagoditi**:
  * Nastavite na 0 za največjo natančnost kalibracije, kadar se svetlobne razmere spreminjajo
  * Povečajte vrednost (npr. na 60–300 sekund) za hitrejšo obdelavo, kadar je osvetlitev stalna in imate pogoste slike kalibracijskih ciljev

### Časovni zamik svetlobnega senzorja

* **Vrsta**: Številka
* **Območje**: od -12 do +12 ur
* **Privzeto**: 0 ur
* **Opis**: Določa časovni zamik (v urah od UTC) za časovne oznake podatkov svetlobnega senzorja, ki se uporablja pri usklajevanju-s senzorja s časi zajema slik. Novejši posnetki `.daq` vsebujejo lastne podatke o časovnem pasu, zato je to potrebno predvsem za starejše dnevnike, posnete v lokalnem času.

### Uporabi PPK-popravke

* **Vrsta**: Potrditveno polje
* **Privzeto**: Onemogočeno (nepotrjeno)
* **Opis**: Omogoča uporabo poprocesiranih kinematičnih (PPK) iz MAPIR DAQ snemalnikov, ki vsebujejo GPS (GNSS). Ko je omogočeno, bo Chloros uporabil vse datoteke dnevnika .daq, ki vsebujejo podatke o izpostavljenosti v vašem projektnem imeniku, in na vaše slike uporabil natančne popravke geolokacije.
* **Zahteva**: V vašem projektnem imeniku mora biti prisotna datoteka dnevnika .daq z vnosi o izpostavljenosti
* **Kdaj omogočiti**: Priporočljivo je, da vedno omogočite PPK-popravke, če imate v datoteki dnevnika .daq vnose o povratnih informacijah o izpostavljenosti.

### Izpostavljenost pin 1

* **Vrsta**: Izbirni spustni meni
* **Vidnost**: Vidno le, če je omogočena možnost »Uporabi PPK-popravke« IN so na voljo podatki o osvetlitvi za pin 1
* **Možnosti**:
  * Imena modelov kamer, zaznana v projektu
  * »Ne uporabljaj« – ta pin osvetlitve se prezre
* **Privzeto**: Samodejno izbrano na podlagi konfiguracije projekta
* **Opis**: Pinu 1 za osvetlitev dodeli določeno kamero za časovno sinhronizacijo PPK. Pin za osvetlitev beleži natančen čas sprožitve zaklopa kamere, kar je ključnega pomena za natančno geolokacijo PPK.
* **Način samodejne izbire**:
  * Ena kamera + en pin: Kamera se izbere samodejno
  * Ena kamera + dva pina: Pin 1 se samodejno dodeli kameri
  * Več kamer: Potrebna je ročna izbira

### Izpostavni pin 2

* **Vrsta**: Izbirni seznam
* **Vidnost**: Vidno le, če je omogočena možnost »Uporabi PPK-popravke« IN so na voljo podatki o izpostavljenosti za pin 2
* **Možnosti**:
  * Imena modelov kamer, zaznana v projektu
  * »Ne uporabljaj« – ta priključek za osvetlitev se prezre
* **Privzeto**: Samodejno izbrano na podlagi konfiguracije projekta
* **Opis**: Pri uporabi konfiguracije z dvema kamerama določi določeno kamero za priključek za osvetlitev 2 za časovno sinhronizacijo PPK.
* **Način samodejnega izbiranja**:
  * Ena kamera + en pin: Pin 2 se samodejno nastavi na »Ne uporabljaj«
  * Ena kamera + dva pina: Pin 2 se samodejno nastavi na »Ne uporabljaj«
  * Več kamer: Potrebna je ročna izbira
* **Opomba**: Ista kamera ne more biti hkrati dodeljena priključku 1 in priključku 2.***

## Senzor svetlobe DAQ

Ta razdelek se prikaže v nastavitvah projekta in navaja vse datoteke DAQ za navzdolno merjenje v projektu — posnetke `.daq` ter dnevnike navzdolnega merjenja DAQ-M `.csv`. Posnetki, narejeni v zavihku Svetlobni senzorji, se samodejno dodajo v odprt projekt.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->

Vsaka vrstica prikazuje datoteko, model senzorja in popravek difuzorskega pokrova, ki dejansko velja za to datoteko. Nad vrsticami je en sam nadzorni element za celoten projekt:

### Prepis pokrova (vse datoteke)

* **Vrsta**: Izbirni spustni meni
* **Možnosti**: `Auto` ter profili popravkov pokrovčka, veljavni za vrste senzorjev, prisotne v projektu
* **Privzeto**: Samodejno
* **Shranjeno kot**: `Processing → "DAQ cap id"` (privzeto `auto`)
* **Opis**: `Auto` uporablja omejitev, zabeleženo v posamezni datoteki (če ni zabeleženo nič, se predpostavlja omejitev »Sunshine« — vsi MAPIR DAQ-ji so opremljeni s korektorjem »Sunshine«). Izbiranje določene omejitve prevlada nad**vsako** datoteko z navzdolnjim sevalom v projektu: surove posnetke popravi z njo, posnetke, ki že vsebujejo korekcijski pokrovček, pa ponovno referencira (zabeležena korekcija se razveljavi in uporabi se izbrana).
* **Pomembno**: Izbrana kapica mora ustrezati kapici, ki je bila fizično nameščena med snemanjem. Niti senzor niti programska oprema ne moreta zaznati fizične kapice – neujemanje ID-ja kapice povzroči napačno korekcijo spektrov.

Namerno je na voljo **ena** nastavitev za celoten projekt namesto spustnih menijev za posamezne datoteke: ta nastavitev velja za vsak vir navzdolnega sevanja v projektu.***

## Poravnava nizov

Ta razdelek se prikaže **samo**, če vsaj ena slika v projektu vsebuje transformacijo poravnave med moduli, ki jo matrike LATTICE vtisnejo ob zajemu (oznake XMP `Chloros:Alignment*`). Prikaže, koliko slik vsebuje oznake poravnave, katera kamera je referenčna (oznaka `REF`) ter tabelo števila slik po posameznih kamerah.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Uporabi poravnavo nizov

* **Vrsta**: Potrditveno polje
* **Privzeto**: Omogočeno (potrjeno)
* **Shranjeno kot**: `Processing → "Array alignment"`
* **Opis**: Vsak obdelan izdelek (debayering / predogled / sevanje / odbojnost / indeks) se prilagodi v skupno referenčno geometrijo matrike z uporabo transformacije, vtisnjene ob zajemu. Izklopljeno = izvoz v izvorni geometriji posameznega senzorja.

### Izreži na skupno prekrivanje

* **Vrsta**: Potrditveno polje (aktivno le, če je vklopljena možnost *Uporabi poravnavo matrike*)
* **Privzeto**: Vklopljeno (označeno)
* **Shranjeno kot**: `Processing → "Array alignment crop"`
* **Opis**: Izrez usklajenih izvozov na območje, ki si ga delijo vsi kamerini moduli, tako da ima vsak pas enak odtis. Izklop ohrani celotno senzorsko platno (črna zapolnitev zunaj vira).

### Ponovno vzorčenje

* **Vrsta**: Izbirni spustni seznam (aktivna le, če je vklopljena možnost *Uporabi poravnavo matrike*)
* **Možnosti**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Privzeto**: Bilinearno
* **Shranjeno kot**: `Processing → "Array alignment interpolation"`
* **Opis**: Interpolacija, ki jo uporablja poravnava. Možnost »Najbližja« ohranja natančne izvorne vrednosti (brez mešanja med pikslami) za strogo radiometrično analizo; možnost »Bilinearna« je najboljša za kartiranje in vizualno uporabo.

Enake tri možnosti obstajajo tudi brez predpone kot `chloros-cli process --array-alignment`, `--array-alignment-crop` in `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Indeks

Te nastavitve omogočajo konfiguracijo multispektralnih indeksov za analizo in vizualizacijo.

### Dodaj indeks

* **Vrsta**: Posebni nadzorni panel za konfiguracijo indeksov
* **Opis**: Odpre interaktivni panel, kjer lahko izberete in konfigurirate multispektralne indekse vegetacije (NDVI, NDRE, EVI, itd.), ki se izračunajo med obdelavo slike. Dodate lahko več indeksov, vsak z lastnimi nastavitvami vizualizacije.
* **Razpoložljivi indeksi**: Spustni meni grafičnega vmesnika vsebuje**27** vnaprej določenih formul za multispektralne indekse (za popoln seznam, vključno z imeni, ki jih sprejema tudi CLI/CLI, glejte [Formule za multispektralne indekse](multispectral-index-formulas.md) za popoln seznam, vključno z imeni, ki jih sprejema tudi možnost CLI/SDK `--indices`).
* **Značilnosti**:
  * Izbirajte med vnaprej določenimi formulami indeksov
  * Kanale filtrov vaše kamere povlecite na pasovna mesta formule
  * Konfigurirajte barvne prehode vizualizacije (LUT – iskalne tabele)
  * Nastavite pragovne vrednosti in načine odrezovanja
  * Ustvarite lastne formule indeksov
* **Opomba**: Indeksi se ne izračunajo za enopasovne črno-bele kamere LATTICE M3M – večpasovni indeksi so na enem pasu nedoločeni. To ne vpliva na modele Survey3 in LATTICE M3C.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->

Vsaka dodana indeksna formula se prikaže kot matematični izraz z barvnim krogom za vsak pasovni slot: rdeča = Red, zelena = Green, modra = Blue, oranžna = Orange, cian = Cyan, vijolična = NIR, magenta = RE. Krožec povlecite iz vrstice nad formulo na mesto, da ga povežete; z dvojnim klikom na povezano mesto ga izbrišete. Indeks se izračuna le enkrat, ko ima vsak slot, ki ga formula uporablja, kanal.

### Uporabniške formule (funkcija Chloros+)

* **Vrsta**: Niz definicij uporabniških formul
* **Razpoložljivost**: Zahteva prijavo z ustreznim naročniškim paketom Chloros+.
* **Opis**: Omogoča ustvarjanje in shranjevanje lastnih multispektralnih indeksnih formul z uporabo matematičnih operacij s pasovi. Lastne formule se shranijo skupaj z nastavitvami projekta in jih je mogoče uporabljati enako kot vgrajene indekse.
* **Kako ustvariti**:
  1. V oknu za konfiguracijo indeksa odprite kalkulator za lastne formule
  2. Vnesite formulo z uporabo **simbolov za pasovne reže**, ne pa imen pasov
  3. Formulo shranite z opisnim imenom – nato se prikaže na dnu spustnega menija formul, kjer kanalne kroge vaše kamere povlečete na njene reže, točno tako kot pri vgrajeni prednastavitvi
* **Sintaksa formule**:
  * Reže pasov: `x`, `y`, `z`, `a`, `b`, `c` — šest mest, ki jih s povlekanjem dodelite dejanskim kanalom
  * Operaterji: `+`, `-`, `*`, `/`, `^` in `()` za združevanje v skupine
  * Funkcije: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Zakaj simboli in ne imena glasbenih skupin**: formula, zapisana kot `(y-x)/(y+x)`, deluje na kateri koli kameri, ker preslikava s povleci in spusti določi, ali je `y` 850 nm NIR filtra RGN ali 808 nm NIR filtra OCN. Vgrajeneprednastavitve so shranjene na enak način – glejte [Formule multispektralnih indeksov](multispectral-index-formulas.md) za natančno obliko simbolov vseh 27.
* **Kje delujejo**: prilagojene formule se shranijo skupaj z nastavitvami projekta in jih je mogoče uporabljati v [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) ter pri obdelavi.**Ne** jih sprejema seznam imen CLI/SDK `--indices`, ki razširja le imena 22 vgrajenih prednastavitev.***

## Izvoz

Te nastavitve določajo format in kakovost izvoženih obdelanih slik.

### Format kalibrirane slike

* **Vrsta**: Izbirni seznam
* **Možnosti**:
  * **TIFF (16-bit)** – Nekomprimiran 16-bitni format TIFF
  * **TIFF (32-bitni, odstotek)** – 32-bitni format TIFF s plavajočo vejico, pri katerem so vrednosti odbojnosti izražene v odstotkih
  * **PNG (8-bitni)** – stisnjen 8-bitni format PNG
  * **JPG (8-bitni)** - Stisnjen 8-bitni format JPEG
* **Privzeto**: TIFF (16-bitni)
* **Opis**: Izbere datotečni format za shranjevanje obdelanih in kalibriranih slik. Izvožene datoteke se shranijo v podmapo za posamezni format znotraj mape vsake kamere (`tiff16`, `tiff32`, `png8`, `jpg8`), pri čemer je za vsak izdelek ena mapa `<Product>_Images/`. Izvožene datoteke ohranijo izvirno ime datoteke – izdelek se prepozna po mapi, ne po končnici imena datoteke.
* **Priporočila glede formata**:
  * **TIFF (16-bitni)**: Priporočljiv za znanstvene analize in profesionalne delovne tokove. Ohranja najvišjo kakovost podatkov brez artefaktov stiskanja. Najbolj primeren za multispektralno analizo in nadaljnjo obdelavo v programski opremi GIS.
  * **TIFF (32-bitni, v odstotkih)**: Najbolj primeren za delovne tokove, ki zahtevajo vrednosti odbojnosti v odstotkih (0–100 %). Zagotavlja največjo natančnost za radiometrične meritve.
  * **PNG (8-bit)**: Primeren za ogled na spletu in splošno vizualizacijo. Manjše datoteke z brezizgubno stiskanjem, vendar zmanjšan dinamični razpon.
  * **JPG (8-bit)**: Najmanjše datoteke, najbolj primerne le za predoglede in prikaz na spletu. Uporablja stiskanje z izgubo, ki ni primerno za znanstvene analize.
* **Opomba**: Sijaj LATTICE se vedno izvozi kot 32-bitni float TIFF, ne glede na to nastavitev.***

## Shrani predlogo projekta

Ta funkcija vam omogoča, da shranite trenutne nastavitve projekta kot predlogo za ponovno uporabo.

* **Vrsta**: Vnos besedila + gumb »Shrani«
* **Opis**: Vnesite opisno ime za predlogo nastavitev in kliknite ikono za shranjevanje. Predloga bo shranila vse vaše trenutne nastavitve projekta (azilacijo cilja, možnosti obdelave, indekse, in format izvoza) za enostavno ponovno uporabo v prihodnjih projektih. Predloge so shranjene v mapi `Project Templates/` znotraj mape za shranjevanje vašega projekta, izbrati ali izvoziti pa jih je mogoče tudi iz glavnega menija (*Izberi predlogo* / *Shrani predlogo* / *Izvozi predlogo*).
* **Primeri uporabe**:
  * Ustvarjanje predlog za različne kamere (RGB, multispektralne, NIR)
  * Shranjevanje standardnih konfiguracij za določene vrste pridelkov ali delovnih tokov analize
  * Skupna raba enotnih nastavitev v ekipi
* **Kako uporabljati**:
  1. Konfigurirajte vse želene nastavitve projekta
  2. Vnesite ime predloge (npr. »RedEdge Survey3 NDVI Standard«)
  3. Kliknite ikono za shranjevanje
  4. Predlogo je zdaj mogoče naložiti ob ustvarjanju novih projektov

***

## Mapa za shranjevanje projektov

Ta nastavitev določa, kam se novi projekti privzeto shranjujejo.

* **Vrsta**: Prikaz poti do mape + gumb »Uredi«
* **Privzeto (Windows)**: `C:\Users\[Username]\Chloros Projects`
* **Privzeto (Linux)**: `~/Chloros Projects`
* **Opis**: Prikaže trenutni privzeti imenik, v katerem se ustvarjajo novi projekti Chloros. Kliknite ikono za urejanje, da izberete drug imenik. Prepis se shrani kot ena vrstica besedila v datoteki `~/.chloros/working_directory.txt` — v datoteki Windows, to je `C:\Users\<Username>\.chloros\working_directory.txt`. Če ta datoteka manjka ali navaja pot, ki ne obstaja več, se Chloros vrne k zgornji privzeti nastavitvi. CLI bere in piše isto datoteko, zato se `chloros-cli` in grafični vmesnik vedno strinjata o tem, kje se nahajajo projekti.
* **Predloge projektov** se nahajajo v podmapah `Project Templates/` tega imenika.
* **Kdaj spremeniti**:
  * Nastavite na omrežni pogon za sodelovanje v ekipi
  * Spremenite na pogon z več prostora za shranjevanje za velike nize podatkov
  * Razvrstite projekte po letu, naročniku ali vrsti projekta v različne mape
* **Opomba**: Sprememba te nastavitve vpliva le na NOVE projekte. Obstoječi projekti ostanejo na svojih prvotnih lokacijah.***

## Ohranjanje nastavitev

Projekt Chloros je **mapa**. Vse nastavitve projekta so shranjene v mapi `project.json` znotraj njega; podatki o priključeni strojni opremi se shranijo skupaj z njimi v mapah `cameras.json` in `sensors.json`, zato se ob ponovnem odprtju projekta ponovno povežejo tudi njegove kamere in svetlobni senzorji. Ko ponovno odprete projekt, se vse nastavitve obnovijo točno takšne, kot ste jih zapustili. Shranjene projekte je mogoče upravljati tudi brez grafičnega vmesnika z datoteko `chloros-cli project` ali z datoteko `open_project` iz mape SDK.

### Hierarhija nastavitev

Nastavitve se uporabljajo v naslednjem vrstnem redu:

1. **Sistemski privzeti nastavitvi** – vgrajeni privzeti nastavitvi, ki jih določa Chloros
2. **Nastavitve predloge** – če pri ustvarjanju projekta naložite predlogo
3. **Nastavitve shranjenega projekta** – nastavitve, shranjene skupaj s projektno datoteko
4. **Ročne prilagoditve** – Vse spremembe, ki jih opravite med trenutno sejo

### Nastavitve in obdelava slik

Nastavitve obdelave se preberejo ob zagonu obdelave. Sprememba nastavitve ne vpliva nazaj na izdelke, ki so že na disku – za uporabo novih nastavitev ponovno zaženite obdelavo. Nekatere nastavitve nikoli ne vplivajo na obdelavo:

* Ločljivost sličic (samo za prikaz)
* Shrani predlogo projekta
* Shrani mapo projekta

***

## Referenca konfiguracijskih ključev

Za avtomatizacijo (CLI `--config`, SDK `configure` ali pri neposrednem branju `project.json` ), so to natančni ključi pod `Project Settings`:

| Pot do ključa | Tip | Privzeto |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | številka 0–10000 | `25` |
| `Target Detection → Minimum Target Clustering (0-100)` | številka 0–100 | `60` |
| `Processing → Vignette correction` | bool | `true` |
| `Processing → Reflectance calibration / white balance` | bool | `true` |
| `Processing → Export sensor response` | bool | `true` |
| `Processing → Export vignette corrected` | bool | `true` |
| `Processing → Export debayered` | bool | `true` |
| `Processing → Export preview` | bool | `true` |
| `Processing → Export radiance` | bool | `true` |
| `Processing → Export reflectance` | bool | `true` |
| `Processing → Array alignment` | bool | `true` |
| `Processing → Array alignment crop` | bool | `true` |
| `Processing → Array alignment interpolation` | `"Bilinear" \| "Nearest" \| "Cubic"` | `"Bilinear"` |
| `Processing → Debayer method` | `"Standard (Fast, Medium Quality)" \| "Texture Aware (Slow, Highest Quality)"` | Standard |
| `Processing → Minimum recalibration interval` | število 0–3600 | `0` |
| `Processing → Light sensor timezone offset` | število -12..12 | `0` |
| `Processing → Apply PPK corrections` | bool | `false` |
| `Processing → DAQ cap id` | ID profila omejitve ali `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | seznam konfiguracij indeksov | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

Ključi `Array alignment` se zapišejo ob prvem prikazu odseka »Poravnava matrike« ali ko jih nastavi klic avtomatizacije. Dokler jih ni, potek uporablja iste vrednosti, kot so prikazane zgoraj (`true`, `true`, bilinear), zato se projekt.json brez njih obnaša enako kot tisti z njimi.

### Ključi, shranjeni v `project.json`, brez možnosti nadzora v nadzornem panelu

Ti se nahajajo v isti drevesni strukturi `Project Settings` in jih obdelava prebere, vendar v stranskem pasu ne boste našli widgeta zanje:

| Pot do ključa | Tip | Privzeto | Nastavi |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK `input_level=`. Prepisuje način razlage vhodnih TIFF-datotek v LATTICE; `auto` iz vsake datoteke`Chloros:ProcessingLevel` XMP oznako ter število kanalov. Pri zajemih Survey3 `.raw` se to ignorira. Namerno ni nastavitev v grafičnem vmesniku — nastavitev »auto« je v vseh običajnih primerih pravilna. |
| `Processing → Target reflectance dir` | niz poti | `""` | `chloros-cli process --target-reflectance-dir` ali cilj projekta API |
| `Processing → Target reflectance config` | slovar, indeksiran po serijski številki kamere | `{}` | Registriranje cilja znotraj okvirja (način `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | niz poti | `""` | SDK `process_folder(daq_log_path=…)`. Kaže na posnetek `.daq` posnetek ali mapo z njimi |
| `Target Detection → Minimum calibration target squares` | številka | `4` | Stara privzeta nastavitev; brez nadzora in brez zastavice CLI |
| `UI → Grid thumbnail size` | številka | `160` | Drsnik za povečavo sličic v mreži slik |

Dve nastavitvi pregledovalnika sta shranjeni **na najvišji ravni v `project.json`**, popolnoma zunaj `Project Settings`, saj gre za stanje prikaza in ne za nastavitve obdelave:

| Pot do ključa | Vrsta | Privzeto | Nastavi |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | celo število 1–256 | `1` | Nadzorni element GSD (px) na zavihku slike — glej [Odpiranje slike v polnem zaslonu](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Priporočene prakse

1. **Začnite s privzetimi nastavitvami**: Privzete nastavitve dobro delujejo za večino kamer MAPIR in tipičnih delovnih tokov.
2. **Ustvarite predloge**: Ko optimizirate nastavitve za določen delovni tok ali kamero, jih shranite kot predlogo, da zagotovite doslednost med projekti.
3. **Preizkusite pred celotno obdelavo**: Ko poskušate z novimi nastavitvami, jih preizkusite na majhnem podsklopu slik, preden obdelate celoten nabor podatkov.
4. **Zabeležite svoje nastavitve**: Uporabite opisna imena predlog, ki navajajo sistem kamere, vrsto obdelave in namembno rabo (npr. „Survey3\_RGB\_NDVI\_Kmetijstvo“).
5. **Izbira formata izvoza**: Izberite format izvoza glede na končno uporabo:
   * Znanstvena analiza → TIFF (16-bitni ali 32-bitni)
   * Obdelava v GIS → TIFF (16-bitno)
   * Hitra vizualizacija → PNG (8-bitno)
   * Deljenje na spletu → JPG (8-bitno)

***

Za več informacij o multispektralnih indeksih v Chloros glejte stran [Formule multispektralnih indeksov](multispectral-index-formulas.md).

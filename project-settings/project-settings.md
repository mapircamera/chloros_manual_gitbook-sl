# Nastavitve projekta

Stranski pas z nastavitvami projekta <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

v programu Chloros vam omogoča konfiguracijo vseh vidikov obdelave slik, zaznavanja kalibracijskih ciljev, izračunov multispektralnih indeksov ter možnosti izvoza za vaš projekt. Te nastavitve se shranijo skupaj z vašim projektom in jih lahko shranite kot predloge za ponovno uporabo v več projektih.

## Dostop do nastavitev projekta

Za dostop do nastavitev projekta:

1. Odprite projekt v programu Chloros.
2. V levem stranskem meniju kliknite zavihek **Nastavitve projekta** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">.
3. V oknu z nastavitvami se prikažejo vse razpoložljive možnosti konfiguracije, razvrščene po kategorijah.



<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->

{% hint style="info" %}
**Nastavitve, ki so odvisne od drugih nastavitev, so sive.** Kadar nadrejena nastavitev onemogoča določeno nastavitev (na primer, če odkljukate *Kalibracija odbojnosti / bela barva*, je *Izvoz odbojnosti* onemogočen), je odvisni nadzorni element onemogočen, v njegovem opisu pa je navedeno, katero nastavitev je treba spremeniti.
{% endhint %}

***

## Prikaz

### Ločljivost sličic

* **Vrsta**: Izbirni seznam
* **Možnosti**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Privzeto**: Privzeto (512 px)
* **Opis**: Ločljivost (najdaljša stranica, v pikslah), pri kateri se upodabljajo sličice v mreži slik. Višje vrednosti so pri povečavi videti ostrejše, vendar se nalagajo počasneje in porabijo več pomnilnika. Polna ločljivost ustreza originalni velikosti slike.
* **Opomba**: Samo za prikaz — to nikoli ne vpliva na obdelavo ali izvožene datoteke.***

## Prepoznavanje ciljev

Te nastavitve nadzorujejo, kako program Chloros prepoznava in obdeluje kalibracijske cilje na vaših slikah. Obe sta aktivni le, če je omogočena **Kalibracija odbojnosti / bela bilanca** (v nasprotnem primeru sta siva, saj se prepoznavanje ciljev v celoti preskoči).

### Najmanjša površina kalibracijskega vzorca (px)

* **Vrsta**: Število
* **Območje**: od 0 do 10.000 pikslov
* **Privzeto**: 25 pikslov
* **Opis**: Določa najmanjšo površino (v pikslah), ki je potrebna, da se zaznano območje šteje za veljaven vzorec kalibracijskega cilja. Manjše vrednosti bodo zaznale manjše cilje, vendar lahko povečajo število lažnih pozitivnih rezultatov. Večje vrednosti zahtevajo večja in jasnejša ciljna območja za zaznavanje.
* **Kdaj prilagoditi**:
  * Povečajte vrednost, če se pojavljajo napačne detekcije na majhnih artefaktih na sliki
  * Zmanjšajte vrednost, če se vaši kalibracijski cilji na slikah zdijo majhni in se ne zaznavajo

### Minimalno združevanje ciljev (0–100)

* **Vrsta**: Število
* **Območje**: od 0 do 100
* **Privzeta vrednost**: 60
* **Opis**: Uravnava prag združevanja za razvrščanje podobno obarvanih območij v skupine pri zaznavanju kalibracijskih ciljev. Višje vrednosti zahtevajo, da se v skupino združijo bolj podobne barve, kar povzroči bolj konzervativno zaznavanje ciljev. Nižje vrednosti dopuščajo večje barvne razlike znotraj skupine ciljev.
* **Kdaj prilagoditi**:
  * Povečajte vrednost, če se kalibracijski cilji razdelijo na več zaznav
  * Zmanjšajte vrednost, če se kalibracijski cilji z barvnimi odstopanji ne zaznavajo v celoti

***

## Obdelava

Te nastavitve določajo, kako program Chloros obdeluje in kalibrira vaše slike.

### Popravek vinjetiranja

* **Vrsta**: Potrditveno polje
* **Privzeto**: Omogočeno (potrjeno)
* **Opis**: Uporabi popravek vinjetiranja za kompenzacijo potemnitve objektiva na robovih slik. Vinjetiranje je pogost optični pojav, pri katerem so koti in robovi slike zaradi lastnosti objektiva temnejši od sredine.
* **Stranski učinek**: Ta stikalo izbere tudi, kateri *nekalibrirani nadomestni izdelek* se zapiše ob izvedbi (glej spodaj).

### Kalibracija odbojnosti / bela bilanca

* **Vrsta**: Potrditveno polje
* **Privzeto**: Omogočeno (označeno)
* **Opis**: Omogoča kalibracijo odbojnosti — na podlagi zaznanih kalibracijskih ciljev v okviru in/ali podatkov o navzdol usmerjenem svetlobnem toku senzorja DAQ, odvisno od kamere in razpoložljivih podatkov. To normalizira vrednosti odbojnosti v vašem nizu podatkov in zagotavlja dosledne meritve ne glede na svetlobne razmere.
* **Ko je onemogočeno**: Prepoznavanje ciljev se v celoti preskoči in**nobena kamera ne more ustvariti izdelka odbojnosti** – niti tista, ki temelji na ciljih Survey3, niti tista, ki temelji na DAQ LATTICE. Odvisne nastavitve (*Izvoz odbojnosti*, *Najkrajši interval ponovne kalibracije* in pragovi za prepoznavanje ciljev) so sive.

### Neikalibrirani nadomestni izhodni podatki: Izvoz odziva senzorja / Izvoz popravljenega vinjetiranja

* **Vrsta**: Dve potrditveni polji
* **Privzete nastavitve**: Obe sta omogočeni (potrjeni)
* **Opis**: Kadar okvirja ni mogoče kalibrirati glede na odbojnost (ni bil najden kalibracijski cilj ali je kalibracija odbojnosti izklopljena), se namesto tega zapiše kot *nekalibriran nadomestni izdelek*. **Za vsak model kamere obstaja natanko en nadomestni izdelek na serijo posnetkov**, ki ga izbere stikalo *Popravek vinjetiranja*:
  * Popravek vinjetiranja **vklopljen**→ `Vignette_Corrected_Images/` (uravnava ga**Izvoz s popravkom vinjetiranja**)
  * Popravek vinjetiranja **izklopljen**→ `Sensor_Response_Images/` (uravnava ga**Izvoz odziva senzorja**)
* Nadomestni izhodni format, ki ni v uporabi, je sivo obarvan. Če odkljukate tistega, ki je v uporabi, se ta datoteka sploh ne zapiše.

### Izhodni izdelki LATTICE

Pri projektih, ki vsebujejo posnetke LATTICE, se vsak uvožen okvir LATTICE v enem samem ciklu obdelave razširi v vse omogočene **in ustrezne**izdelke. Razporeditev nadzirajo štiri potrditvena polja (vsa so privzeto**vklopljena**):

| Nastavitev | Izhodna mapa | Kaj izvozi |
| --- | --- | --- |
| **Izvoz brez debayeringa** | `Debayered_Images/` | Linearna slika brez debayeringa: odstranjen je temni odmik kalibracijskega paketa, nato sledi demosaic z upoštevanjem robov, brez korekcije linearnosti, izravnave polja ali ločevanja barv — ista datoteka, kot jo posnetek shrani na ravni brez debayeringa. Ohranjena je izvorna orientacija senzorja (zabeležena v XMP). Velja za RGB- in multispektralne kamere. |
| **Izvoz predogleda** | `Preview_Images/` | Predogled na zaslonu. RGB = bela bilanca (DAQ-svetlobni vir, če je na voljo, sicer siva skala) + gama; multispektralno = raztezek v lažnih barvah. |
| **Izvoz sevanja** | `Radiance_Images/` | Spektralno sevanje v Float32 v W/m²/sr/nm. Samo za multispektralne kamere (M3C/M3M) — ne velja za RGB-izvirnike. Vedno se zapiše kot 32-bitni TIFF, ne glede na nastavitev *Kalibrirani format slike*. |
| **Izvoz odbojnosti**| `Reflectance_Calibrated_Images/` | Odbojnost v Uint16, skalirana tako, da**32768 = odbojnost 1,0** (označeno kot XMP `Chloros:PixelScale`). Samo za multispektralne slike, zapiše se, kadar ustrezen zapis `.daq` za navzdol usmerjeno sevanje (ali cilj v okviru, ki je prestal kontrolo kakovosti) pokriva okvir. |

* Glavne kamere RGB oddajajo podatke po debayeringu + predogled; sevanje/odbojnost se pri njih preskočita, ker nista uporabna.
* Bitna globina podatkov po debayeringu/predogledu sledi nastavitvi *Kalibrirani format slike*; sevanje je vedno float32.
* Te štiri stikala ne vplivajo na obdelavo *Survey3 *.
* Produkti sevanja in odbojnosti multispektralnih kamer M3C lahko prav tako zapišejo **masko omejitve** v mapo `clamp_masks/` poleg map s produkti, pri čemer zabeležijo, kje je razmešavanje pasov omejilo negativno vrednost na nič. Glej [Formati izhodnih slik](../output-image-formats.md#clamp-masks).

Isti štirje stikala obstajajo tudi brez glave kot `chloros-cli process --debayered / --preview / --radiance / --reflectance`. Nadomestila so staro zastavico CLI-ja `--radiometric-output`, ki ne obstaja več.

{% hint style="warning" %}
**Če izklopite vse ustrezne izhodne podatke, se izvedba ne uspe.** Od različice 1.2.0 naprej izvedba obdelave, ki je zahtevala izhodne podatke, a ni zapisala nobenega slikovnega izhodnega podatka, poroča o napaki, CLI pa se zaključi z vrednostjo, ki ni nič, namesto da bi poročal o tihem uspehu. Dnevnik navaja izhodni podatek, ki ga ni mogel zapisati, in razlog za to. Namerno izvedba, ki vključuje le metapodatke (nič ni bilo zahtevano) še vedno velja za uspešno.
{% endhint %}

### Vir odbojnosti (nastavitev projekta, nastavljena prek CLI-ja)

Projekt shranjuje tudi, katero **referenco odbojnosti** uporablja produkt odbojnosti LATTICE. V nadzornem panelu ni posebnega upravljalnika; vrednost je shranjena v konfiguraciji projekta kot `Processing → "Target reflectance source"` in se nastavi z `chloros-cli process --reflectance-source {auto,target,daq}` (SDK-jev `configure()` za to nima ključne besede):

* **`auto`** (privzeto): kalibracijski cilj znotraj okvirja, ki je uspešno prestal kontrolo kakovosti (QA), postane absolutna referenca; če cilja ni ali če QA ni uspešna, se uporabi delitni faktor navzdolnega sevanja DAQ (ρ = πL/E). Cilj, ki uspešno opravi kontrolo kakovosti z oznako (padel panel ali ne delujoč pas), ni samodejno prednosten: verige »cilj«, »cilj+DAQ« in »samo DAQ« se preizkušajo v vrstnem redu glede na njihovo izmerjeno oceno na zadržanih panelih.
* **`target`**: stroga odbojnost, ki jo določa cilj — brez nadomestitve s sistemom DAQ.
* **`daq`**: odbojnost, ki jo določa sistem DAQ; cilji znotraj okvirja se ne uporabljajo kot referenca.

Shranjena vrednost se primerja brez upoštevanja velikosti črk, nekaj različnih zapisov pa se sprejme kot sinonimi: `target`, `target_image`, `empirical` in `empirical_line` vsi pomenijo **cilj**; `daq`, `dls`, `light_sensor` in `sensor` vsi pomenijo**daq**. Vse ostalo – vključno z manjkajočim ključem – se razreši kot**auto**.**Izmerjene** ciljne skeniranje po enotah se poiščejo po serijski številki/QR-kodi ciljne enote, kot je `<serial>.csv`, na treh mestih: v imeniku, navedenem z `--target-reflectance-dir` (shranjenem kot `Processing → "Target reflectance dir"`), v lastni mapi projekta `target_reflectance/`, in v poti, navedeni v okoljski spremenljivki `CHLOROS_TARGET_REFLECTANCE_DIR`. Če za to enoto ne obstaja noben izmerjen sken, se namesto tega uporabi nominalna objavljena krivulja za ciljni model.

### Metoda odstranjevanja pik (Debayer)

* **Vrsta**: Izbirni seznam
* **Možnosti**:
  * Standardna (hitra, srednja kakovost)
  * Upošteva teksturo (počasna, najvišja kakovost) \[Chloros+]
* **Privzeto**: Standardna (hitra, srednja kakovost)
* **Opis**: Izbere algoritem demosaicinga, ki se uporablja za pretvorbo surovih podatkov senzorja z vzorcem Bayerja v polnobarvne slike. Metoda »Standardna (hitra, srednja kakovost)« zagotavlja optimalno ravnovesje med hitrostjo obdelave in kakovostjo slike. Metoda »Upošteva teksturo (počasna, najvišja kakovost)« \[Chloros+] uporablja visokokakovostno demosaicing z upoštevanjem robov v kombinaciji z modelom za odstranjevanje šuma na podlagi umetne inteligence/strojnega učenja, ki odstrani skoraj ves šum pri demosaicingu. Model »Texture Aware« za delovanje potrebuje pomnilnik grafične kartice (VRAM). Priporočamo, da ga uporabite, če imate na voljo več kot 4 GB VRAM-a, kar omogoča hitrejšo obdelavo.
* **Če je vrstica sploh spustni meni**: spustni meni z dvema možnostma se prikaže le, če**oba** pogoja veljata — ste prijavljeni z ustreznim naročniškim paketom Chloros+, **in** projekt ne vsebuje posnetkov LATTICE. V nasprotnem primeru se vrstica prikaže kot navadno besedilo z besedilom `Standard (Fast, Medium Quality)`, brez možnosti izbire.
* **Opomba o LATTICE**: Ni modela »Texture Aware«, usposobljenega za LATTICE, zato procesna veriga za okvirje LATTICE prisili standardno demosaiciranje ne glede na shranjeno vrednost. Če dodate mapo LATTICE v projekt, v katerem je bila že izbrana možnost »Texture Aware«, Chloros ponovno nastavi vrednost na »Standard«, namesto da bi pustil zastarelo vrednost v `project.json`.

### Najkrajši interval ponovne kalibracije

* **Vrsta**: Številka
* **Območje**: od 0 do 3.600 sekund
* **Privzeto**: 0 sekund
* **Opis**: Določa najkrajši časovni interval (v sekundah) med uporabo kalibracijskih ciljev. Če je nastavljen na 0, bo Chloros uporabil vsak zaznan kalibracijski cilj. Če je nastavljen na višjo vrednost, bo Chloros uporabil le kalibracijske cilje, ki so med seboj ločeni vsaj za toliko sekund, s čimer se skrajša čas obdelave za nize podatkov s pogostim zajemanjem kalibracijskih ciljev.
* **Kdaj prilagoditi**:
  * Nastavite na 0 za največjo natančnost kalibracije, kadar se svetlobne razmere spreminjajo
  * Povečajte vrednost (npr. na 60–300 sekund) za hitrejšo obdelavo, kadar je osvetlitev konstantna in imate pogoste slike kalibracijskih ciljev

### Časovni zamik svetlobnega senzorja

* **Vrsta**: Številka
* **Območje**: od -12 do +12 ur
* **Privzeto**: 0 ur
* **Opis**: Določa časovni zamik (v urah od UTC) za časovne oznake podatkov svetlobnega senzorja, ki se uporablja pri usklajevanju dnevnikov svetlobnega senzorja s časi zajema slik. Novejši posnetki `.daq` vsebujejo lastne podatke o časovnem pasu, zato je to potrebno predvsem za starejše dnevnike, posnete v lokalnem času.

### Uporabi PPK-popravke

* **Vrsta**: Potrditveno polje
* **Privzeto**: Onemogočeno (neoznačeno)
* **Opis**: Omogoča uporabo popravkov PPK (Post-Processed Kinematic) iz snemalnikov MAPIR DAQ, ki vsebujejo GPS (GNSS). Ko je ta možnost omogočena, bo Chloros uporabil vse datoteke dnevnikov .daq, ki vsebujejo podatke o izpostavljenosti, v vašem projektnem imeniku in na vaše slike uporabil natančne popravke geografske lokacije.
* **Zahteva**: V vašem projektnem imeniku mora biti prisotna datoteka dnevnika .daq z vnosi o izpostavljenosti
* **Kdaj omogočiti**: Priporočljivo je, da vedno omogočite PPK-popravek, če imate v datoteki dnevnika .daq vnose o povratnih informacijah o izpostavljenosti.

### Izpostavnostni pin 1

* **Vrsta**: Izbirni spustni meni
* **Vidnost**: Vidno le, če je je omogočena možnost »Uporabi PPK-popravke« IN so na voljo podatki o osvetlitvi za zatič 1
* **Možnosti**:
  * Imena modelov kamer, zaznana v projektu
  * »Ne uporabljaj« – ta zatič osvetlitve se prezre
* **Privzeto**: Samodejno izbrano na podlagi konfiguracije projekta
* **Opis**: Pinu 1 za osvetlitev dodeli določeno kamero za časovno sinhronizacijo PPK. Pin za osvetlitev beleži natančen trenutek sprožitve zaklopa kamere, kar je ključnega pomena za natančno geolokacijo PPK.
* **Način samodejne izbire**:
  * Ena kamera + en pin: Kamera se izbere samodejno
  * Ena kamera + dva pina: Pin 1 se samodejno dodeli kameri
  * Več kamer: Potrebna je ročna izbira

### Izpostavni pin 2

* **Vrsta**: Izbirni seznam
* **Vidnost**: Vidno le, če je omogočena možnost »Uporabi PPK-popravke« IN so na voljo podatki o izpostavljenosti za pin 2
* **Možnosti**:
  * Imena modelov kamer, zaznana v projektu
  * »Ne uporabljaj« – ta pin za osvetlitev se prezre
* **Privzeto**: Samodejno izbrano na podlagi konfiguracije projekta
* **Opis**: Pri uporabi konfiguracije z dvema kamerama dodeli določeno kamero pinu 2 za osvetlitev za časovno sinhronizacijo PPK.
* **Način samodejne izbire**:
  * Ena kamera + en pin: Pin 2 se samodejno nastavi na »Ne uporabljaj«
  * Ena kamera + dva pina: Pin 2 se samodejno nastavi na »Ne uporabljaj«
  * Več kamer: Potrebna je ročna izbira
* **Opomba**: Ista kamera ne more biti hkrati dodeljena priključku 1 in priključku 2.***

## Senzor svetlobe DAQ

Ta razdelek se prikaže v nastavitvah projekta in navaja vse datoteke DAQ o padajočem sevanju v projektu — posnetke `.daq` ter dnevnike DAQ-M `.csv`. Zapisi, narejeni v zavihku Svetlobni senzorji, se samodejno dodajo v odprt projekt.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->



Vsaka vrstica prikazuje datoteko, model senzorja in popravek pokrovčka difuzorja, ki dejansko velja za to datoteko. Nad vrsticami je en sam nadzorni element za celoten projekt:

### Prepis pokrovčka (vse datoteke)

* **Vrsta**: Izbirni seznam
* **Možnosti**: `Auto` ter profili korekcije pokrovčka, veljavni za vrste senzorjev, prisotne v projektu
* **Privzeto**: Samodejno
* **Shranjeno kot**: `Processing → "DAQ cap id"` (privzeto `auto`)
* **Opis**: `Auto` uporablja pokrov, zabeležen v posamezni datoteki (če ni zabeležen, se predpostavlja pokrov Sunshine, vendar so DAQ-U, DAQ-M in DAQ-E dobavljeni brez pokrova, program Chloros pa ne more ugotoviti, kateri je nameščen, zato obdelava izda opozorilo o predpostavljeni omejitvi: izberite omejitev, ki je bila nameščena). Izbor določene omejitve prevlada nad **vsemi** datotekami z navzdolnjim tokom v projektu, razen pri DAQ-E-S, katerega difuzor je del njegove kalibracije; projekt, ki vsebuje samo DAQ-E-S, ne prikaže spustnega menija: surove posnetke se popravi z njim, posnetki, ki že vsebujejo pokrovček, pa se ponovno referencirajo (zabeležena korekcija se razveljavi in uporabi se izbrana).
* **Pomembno**: Izbrani pokrovček mora ustrezati pokrovčku, ki je bil fizično nameščen med snemanjem. Niti senzor niti programska oprema ne moreta zaznati fizičnega pokrovčka – neujemanje ID-ja pokrovčka povzroči napačno korekcijo spektrov.

Namesto spustnih menijev za posamezne datoteke je namerno na voljo **en** nadzorni element za celoten projekt: ta nastavitev velja za vsak vir, usmerjen navzdol, v projektu.***

## Poravnava niza

Ta razdelek se prikaže **samo**, če vsaj ena slika v projektu vsebuje transformacijo poravnave med moduli, ki jo matrike LATTICE vtisnejo ob zajemu (oznake XMP `Chloros:Alignment*`). Prikazuje, koliko slik vsebuje oznake poravnave, katera kamera je referenčna (oznaka `REF`) ter tabelo števila slik po posameznih kamerah.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Uporabi poravnavo nizov

* **Vrsta**: Potrditveno polje
* **Privzeto**: Omogočeno (potrjeno)
* **Shranjeno kot**: `Processing → "Array alignment"`
* **Opis**: Vsako obdelano sliko (debayered / predogled / radiance / reflectance / index) prilagodi skupni referenčni geometriji matrike z uporabo transformacije, zabeležene ob zajemu. Izklopljeno = izvoz v izvorni geometriji posameznega senzorja.

### Izreži na skupno prekrivanje

* **Vrsta**: Potrditveno polje (aktivno le, če je vklopljena možnost *Uporabi poravnavo matrike*)
* **Privzeto**: Vklopljeno (označeno)
* **Shranjeno kot**: `Processing → "Array alignment crop"`
* **Opis**: Izrez usklajenih izvozov na območje, ki ga delijo vsi kamerini moduli, tako da ima vsak pas enak odtis. Izklop ohrani celotno senzorsko platno (črna zapolnitev zunaj vira).

### Ponovno vzorčenje

* **Vrsta**: Izbirni spustni meni (aktivno le, če je vklopljena možnost *Uporabi poravnavo matrike*)
* **Možnosti**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Privzeto**: Bilinearno
* **Shranjeno kot**: `Processing → "Array alignment interpolation"`
* **Opis**: Interpolacija, ki jo uporablja izkrivljanje za poravnavo. Najbližja ohranja natančne vrednosti vira (brez mešanja med pikslami) za strogo radiometrično analizo; Bilinearna je najboljša za kartiranje in vizualno uporabo.

Enake tri možnosti obstajajo tudi brez predpone kot `chloros-cli process --array-alignment`, `--array-alignment-crop` in `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Indeks

Te nastavitve vam omogočajo konfiguracijo multispektralnih indeksov za analizo in vizualizacijo.

### Dodaj indeks

* **Vrsta**: Posebni panel za konfiguracijo indeksov
* **Opis**: Odpre interaktiven panel, kjer lahko izberete in konfigurirate multispektralne vegetacijske indekse (NDVI, NDRE, EVI itd.), ki se izračunajo med obdelavo slike. Dodate lahko več indeksov, vsak z lastnimi nastavitvami vizualizacije.
* **Razpoložljivi indeksi**: Spustni meni grafičnega vmesnika vsebuje**27** vnaprej določenih formul za multispektralne indekse (glejte [Formule za multispektralne indekse](multispectral-index-formulas.md) za popoln seznam, vključno z imeni, ki jih sprejema tudi CLI/SDK `--indices`).
* **Značilnosti**:
  * Izbirajte med vnaprej določenimi formulami indeksov
  * Kanale filtrov vaše kamere povlecite na prostore za pasove v formuli
  * Konfigurirajte barvne prehode vizualizacije (LUT – iskalne tabele)
  * Nastavite pragovne vrednosti in načine obrezovanja
  * Ustvarite lastne indeksne formule
* **Opomba**: Indeksi se ne izračunajo za enopasovne črno-bele kamere LATTICE M3M — večpasovni indeksi so na enem pasu nedoločeni. To ne vpliva na kamere Survey3 in LATTICE M3C.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->



Vsak indeks, ki ga dodate, prikaže svojo formulo kot matematični izraz, z barvnim krogom za vsako mesto pasu: rdeča = Red, zelena = Green, modra = Blue, oranžna = Orange, cian = Cyan, vijolična = NIR, magenta = RE. Krožec povlecite iz vrstice nad formulo na mesto, da ga povežete; dvokliknite povezano mesto, da ga izbrišete. Indeks se izračuna le, če ima vsako mesto, ki ga formula uporablja, kanal.

### Lastne formule (funkcija Chloros+)

* **Vrsta**: Niz definicij po meri
* **Razpoložljivost**: Zahteva prijavo z ustreznim naročniškim paketom Chloros+.
* **Opis**: Omogoča ustvarjanje in shranjevanje po meri izdelanih formul za multispektralne indekse z uporabo matematičnih izrazov za pasove. Formule po meri se shranijo skupaj z nastavitvami projekta in jih je mogoče uporabljati enako kot vgrajene indekse.
* **Kako ustvariti**:
  1. V oknu za konfiguracijo indeksa odprite kalkulator za lastne formule
  2. Formulo napišite z uporabo **simbolov pasov in slotov**, ne pa z imeni pasov
  3. Formulo shranite z opisnim imenom – nato se prikaže na dnu spustnega menija formul, kanalne kroge vaše kamere pa povlečete na njene reže točno tako kot pri vgrajeni prednastavitvi
* **Sintaksa formule**:
  * Reže pasov: `x`, `y`, `z`, `a`, `b`, `c` — šest mest, ki jih s povlekanjem dodelite dejanskim kanalom
  * Operaterji: `+`, `-`, `*`, `/`, `^` in `()` za združevanje v skupine
  * Funkcije: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Zakaj simboli in ne imena skupin**: formula, zapisana kot `(y-x)/(y+x)`, deluje na kateri koli kameri, saj preslikava s povleci in spusti določi, ali je `y` 850 nm NIR filtra RGN ali 808 nm NIR filtra OCN. Vgrajene prednastavitve so shranjene na enak način — glejte [Formule multispektralnih indeksov](multispectral-index-formulas.md) za natančno obliko simbolov vseh 27.
* **Kje delujejo**: prilagojene formule se shranijo skupaj z nastavitvami projekta in jih je mogoče uporabljati v [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) ter pri obdelavi. CLI/SDK-ov seznam imen `--indices` jih **ne** sprejema, saj ta seznam razširja le imena 22 vgrajenih prednastavitev.***

## Izvoz

Te nastavitve uravnavajo format in kakovost izvoženih obdelanih slik.

### Format kalibrirane slike

* **Vrsta**: Izbirni seznam
* **Možnosti**:
  * **TIFF (16-bit)** – Nekomprimiran 16-bitni format TIFF


  * **TIFF (8-bit)** – Nekomprimiran 8-bitni format TIFF


  * **TIFF (32-bit, odstotek)** – 32-bitni TIFF s plavajočo vejico, vrednosti odbojnosti pa so izražene v odstotkih
  * **PNG (8-bit)** – Komprimiran 8-bitni format PNG


  * **JPG (8-bitni)** – stisnjen 8-bitni format JPEG


* **Privzeto**: TIFF (16-bitni); projekt, ki v celoti uporablja 8-bitne podatke, prikaže TIFF (8-bitni) in ponuja le 8-bitne formate
* **Opis**: Izbere datotečni format za shranjevanje obdelanih in kalibriranih slik. Izvožene slike se shranijo v podmapo za posamezni format znotraj mape vsake kamere (`tiff16`, `tiff32`, `png8`, `jpg8`), pri čemer je za vsak izdelek ena mapa `<Product>_Images/`. Izvožene datoteke ohranijo izvorno ime datoteke – izdelek se prepozna po mapi, ne po končnici imena datoteke.
* **Priporočila glede formatov**:
  * **TIFF (16-bitni)**: Priporočljiv za znanstvene analize in profesionalne delovne tokove. Ohranja najvišjo kakovost podatkov brez artefaktov stiskanja. Najbolj primeren za multispektralno analizo in nadaljnjo obdelavo v programski opremi GIS.
  * **TIFF (32-bitni, odstotki)**: Najbolj primeren za delovne tokove, ki zahtevajo vrednosti odbojnosti v odstotkih (0–100 %). Zagotavlja največjo natančnost za radiometrične meritve.
  * **PNG (8-bit)**: Primeren za ogled na spletu in splošno vizualizacijo. Manjše datoteke z brezizgubno stiskanjem, vendar zmanjšan dinamični razpon.
  * **JPG (8-bit)**: Najmanjše datoteke, najbolj primerno le za predoglede in prikaz na spletu. Uporablja kompresijo z izgubo, ki ni primerna za znanstvene analize.
* **Opomba**: Radianca LATTICE se vedno izvozi kot 32-bitni TIFF s plavajočo vejico, ne glede na to nastavitev.

***

## Shranjevanje predloge projekta

Ta funkcija vam omogoča, da shranite trenutne nastavitve projekta kot predlogo za ponovno uporabo.

* **Vrsta**: Vnos besedila + gumb »Shrani«
* **Opis**: Vnesite opisno ime za predlogo nastavitev in kliknite ikono za shranjevanje. Predloga bo shranila vse trenutne nastavitve projekta (azilacijo ciljev, možnosti obdelave, indekse, in format izvoza) za enostavno ponovno uporabo v prihodnjih projektih. Predloge so shranjene v mapi `Project Templates/` znotraj mape za shranjevanje vašega projekta, izbrati ali izvoziti pa jih je mogoče tudi iz glavnega menija (*Izberi predlogo* / *Shrani predlogo* / *Izvozi predlogo*).
* **Primeri uporabe**:
  * Ustvarjanje predlog za različne sisteme kamer (RGB, multispektralne, NIR)
  * Shranjevanje standardnih konfiguracij za določene vrste pridelkov ali delovne tokove analize
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
* **Privzeto (za Windows)**: `Documents\MAPIR\Chloros Projects` v mapi »Dokumenti« — običajno `C:\Users\[Username]\Documents\MAPIR\Chloros Projects` ali kjerkoli je OneDrive ali potujoči profil premaknil mapo »Dokumenti«
* **Privzeto (za Linux)**: `~/Documents/MAPIR/Chloros Projects`
* **Opis**: Prikaže trenutno privzeto mapo, v kateri se ustvarjajo novi projekti Chloros. Kliknite ikono za urejanje, da izberete drugo mapo. Sprememba se shrani kot ena vrstica besedila v datoteki `~/.chloros/working_directory.txt` — v sistemu Windows je to datoteka `C:\Users\<Username>\.chloros\working_directory.txt`. Če ta datoteka manjka ali navaja pot, ki ne obstaja več, se Chloros vrne k zgornji privzeti nastavitvi. CLI bere in piše isto datoteko, zato se `chloros-cli` in grafični vmesnik vedno strinjata o tem, kje se nahajajo projekti.
* **Predloge projektov** se nahajajo v podmapah `Project Templates/` tega imenika.
* **Kdaj spremeniti**:
  * Nastavite na omrežni pogon za sodelovanje v ekipi
  * Spremenite na pogon z več prostora za shranjevanje za velike podatkovne nize
  * Razvrstite projekte po letu, naročniku ali vrsti projekta v različne mape
* **Opomba**: Sprememba te nastavitve vpliva le na NOVE projekte. Obstoječi projekti ostanejo na svojih prvotnih lokacijah.
* **Dolžina poti v sistemu Windows**: če niso omogočene dolge poti v sistemu Windows (opcija namestitvenega programa **Omogoči dolge poti v sistemu Windows**), se mapa projekta, ki je globlja od približno 130 znakov, zavrne pred obdelavo, saj bi njeni izhodni podatki presegli omejitev poti v sistemu Windows, ki znaša 260 znakov. Glejte [Začetek obdelave](../processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

***

## Ohranjanje nastavitev

Projekt Chloros je **mapa**. Vse nastavitve projekta so shranjene v mapi `project.json` znotraj nje; podatki o priključeni strojni opremi se shranijo skupaj z njimi v datotekah `cameras.json` in `sensors.json`, tako da se ob ponovnem odprtju projekta ponovno povežejo tudi kamere in svetlobni senzorji. Ko ponovno odprete projekt, se vse nastavitve obnovijo točno takšne, kot ste jih zapustili. Shranjene projekte je mogoče upravljati tudi brez grafičnega vmesnika s pomočjo datoteke `chloros-cli project` ali datoteke `open_project` iz SDK-ja.

### Hierarhija nastavitev

Nastavitve se uporabljajo v naslednjem vrstnem redu:

1. **Sistemski privzeti nastavitvi** – vgrajeni privzeti nastavitvi, ki jih je določil Chloros

2. **Nastavitve predloge** – če pri ustvarjanju projekta naložite predlogo
3. **Nastavitve shranjenega projekta** – nastavitve, shranjene skupaj s projektno datoteko
4. **Ročne prilagoditve** – vse spremembe, ki jih opravite med trenutno sejo

### Nastavitve in obdelava slik

Nastavitve obdelave se preberejo ob zagonu obdelave. Sprememba nastavitve ne vpliva retroaktivno na izdelke, ki so že na disku – za uporabo novih nastavitev je treba obdelavo ponovno zagnati. Nekaj nastavitev sploh ne vpliva na obdelavo:

* Ločljivost sličic slik (samo za prikaz)
* Shrani predlogo projekta
* Shrani mapo projekta

***

## Referenca konfiguracijskih ključev

Za avtomatizacijo (CLI `--config`, SDK `configure` ali neposredno branje `project.json`) so to natančni ključi v datoteki `Project Settings`:

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
| `Processing → Minimum recalibration interval` | število 0-3600 | `0` |
| `Processing → Light sensor timezone offset` | število -12..12 | `0` |
| `Processing → Apply PPK corrections` | bool | `false` |
| `Processing → DAQ cap id` | ID profila omejitve ali `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | seznam konfiguracij indeksov | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (8-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

Ključi `Array alignment` se zapišejo ob prvem prikazu odseka »Poravnava niza« ali ko jih določi klic avtomatizacije. Če niso prisotni, cevovod uporabi iste vrednosti, kot so prikazane zgoraj (`true`, `true`, bilinearni), zato se projekt .json brez njih obnaša enako kot tisti z njimi.

### Ključi, shranjeni v `project.json`, ki jih ni mogoče nadzorovati v nadzornem panelu

Ti se nahajajo v istem drevesu `Project Settings` in jih obdelava prebere, vendar v stranskem pasu ne boste našli widgeta zanje:

| Pot do ključa | Tip | Privzeto | Nastavi |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK `input_level=`. Prepisuje način razlage vhodnih datotek TIFF v LATTICE; `auto` to izpelje iz XMP-oznake vsake datoteke `Chloros:ProcessingLevel` ter števila kanalov. Pri zajemih Survey3 `.raw` se to prezre. Namerno ni nastavitev v grafičnem vmesniku — avtomatska nastavitev je v vseh običajnih primerih pravilna. |
| `Processing → Target reflectance dir` | niz poti | `""` | `chloros-cli process --target-reflectance-dir` ali ciljni API projekta |
| `Processing → Target reflectance config` | slovar, indeksiran po serijski številki kamere | `{}` | Registracija cilja znotraj okvirja (način `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | niz poti | `""` | SDK `process_folder(daq_log_path=…)`. Kaže na posnetek `.daq` ali mapo s posnetki |
| `Target Detection → Minimum calibration target squares` | številka | `4` | Stara privzeta vrednost; brez nadzora in brez zastavice CLI |
| `UI → Grid thumbnail size` | številka | `160` | Lastni drsnik za povečavo sličic v mreži slik |

Dve nastavitvi pregledovalnika sta shranjeni **na najvišji ravni v `project.json`**, popolnoma zunaj `Project Settings`, saj gre za stanje prikaza in ne za nastavitve obdelave:

| Pot do ključa | Tip | Privzeto | Nastavi |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | celo število 1–256 | `1` | Nadzorni element GSD (px) na zavihku slike — glej [Odpiranje slike v polnem zaslonu](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Priporočene prakse

1. **Začnite s privzetimi nastavitvami**: Privzete nastavitve dobro delujejo za večino sistemov kamer MAPIR in tipičnih delovnih tokov.
2. **Ustvarite predloge**: Ko optimizirate nastavitve za določen delovni tok ali kamero, jih shranite kot predlogo, da zagotovite doslednost med projekti.
3. **Preizkusite pred celotno obdelavo**: Ko eksperimentirate z novimi nastavitvami, jih preizkusite na majhnem podnizu slik, preden obdelate celoten nabor podatkov.
4. **Dokumentirajte svoje nastavitve**: Uporabite opisna imena predlog, ki navajajo sistem kamer, vrsto obdelave in namembno uporabo (npr. »Survey3\_RGB\_NDVI\_Agriculture«).
5. **Izbira formata izvoza**: Izberite format izvoza glede na končno uporabo:
   * Znanstvena analiza → TIFF (16-bitni ali 32-bitni)
   * Obdelava v GIS-u → TIFF (16-bitni)
   * Hitra vizualizacija → PNG (8-bitni)
   * Deljenje na spletu → JPG (8-bitni)

***

Za več informacij o multispektralnih indeksih v programu Chloros si oglejte stran [Formule multispektralnih indeksov](multispectral-index-formulas.md).

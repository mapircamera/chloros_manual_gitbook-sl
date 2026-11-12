# Mreža slik

Po uvozu slik v projekt jih boste v glavnem področju videli razporejene v mreži. V tej mreži izberete, **katero različico vsake slike si ogledujete** — gumbi nad njo hkrati preklapljajo vsako sličico med izvirnimi datotekami in posameznimi obdelanimi izdelki.

## Velikost sličic

Za prilagajanje velikosti sličic uporabite drsnik povečave v zgornjem desnem kotu. Drsnik sega od **64 px do 1200 px**.

* **Ctrl + kolesce miške** prav tako spreminja velikost sličic.
* **Ctrl + `+`**/**Ctrl + `=`**in**Ctrl + `−`** spreminjajo velikost za 4 px ob vsakem pritisku. Obseg nastavitev s tipkovnico se na spodnji meji konča pri 64 px, na zgornji meji pa pri velikosti, ki v trenutnem oknu natančno ustreza dvema sličicama na vrstico.
* Velikost, za katero se odločite, se shrani skupaj s projektom (`UI → Grid thumbnail size` v `project.json`, privzeto `160`), tako da se ob ponovnem odprtju projekta ta ponovno vzpostavi.

<figure><img src="../.gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>*Ločljivost* sličic je nastavljiva ločeno od *velikosti* sličic: glejte **Prikaz → Ločljivost sličic slik** v [Nastavitvah projekta](../project-settings/project-settings.md) (privzeto 512 px na daljši strani). Velikost določa, kako velika je ploščica; ločljivost pa določa, koliko podrobnosti se prinese za njeno zapolnitev.

***

## Orodna vrstica mreže

Vrstica gumbov nad mrežo ima do treh skupin, od leve proti desni:

1. **Po sprožilcu / Po kameri** — način združevanja. Prikaže se le pri projektih, ki vsebujejo posnetke LATTICE.
2. **Gumbi za filtriranje kamer** — po en za vsako kamero LATTICE. Prikažejo se le v načinu Po kameri.
3. **Gumbi za izvoz/način prikaza** — kateri izdelek prikazuje vsaka sličica.

Ko je okno preozko za vse gumbe, se skupine z desne proti levi skrčijo v spustne menije ob preletu z miško: najprej se skrčijo gumbi za izvoz/pregled, nato pa gumbi za kamere. Skrčena skupina pusti le en gumb, označen s trenutno aktivno izbiro, ob preletu z miško pa se prikaže celoten nabor. **Gumb »Po sprožilcu« / »Po kameri« se nikoli ne zloži.

<!-- SCREENSHOT-NEEDED: Image grid toolbar of a LATTICE array project at full width, showing all three button groups inline: Per Trigger / Per Camera, three camera filter buttons labelled "LATT-M3M (serial)", and the export/view buttons including TIFF, RAW (Original), RAW (Radiance), RAW (Reflectance). -->

***

## Gumbi za izvoz in ogled

Ti gumbi preklapljajo med različnimi tipi slik v mreži sličic. **Gumb se prikaže takoj, ko obstaja izdelek, ki ga imenuje** — kar za izvorne datoteke pomeni takoj ob uvozu, ne šele po obdelavi. Chloros med potekom obdelave ponovno skenira izdelke projekta, zato se gumbi pojavijo med obdelavo, ko se vsak izdelek začne shranjevati na disk.

### Osnovni gumb

Gumb za izvoz skrajno levo je označen s **tistim, kar ste dejansko uvozili**:

| Kaj ste uvozili | Oznaka gumba |
| --- | --- |
| Survey3 RAW+JPG | `JPG` |
| Posnetki LATTICE s predogledom na zaslonu ob surovem kadru | `PNG` ali `TIFF`, odvisno od tega, kateri predogledi so |
| Posnetki LATTICE, pri katerih je osnovna datoteka **RAW-posnetek** | *ni gumba* — `RAW (Original)` že prikazuje to datoteko |

V mešanem projektu se oznaka prilagodi končnici, ki jo uporablja večina slik.

### Gumbi za izdelke

| Gumb | Prikaže | Kdaj se prikaže |
| --- | --- | --- |
| **Cilji** | Slike z zaznanim kalibracijskim ciljem | Po izvedbi, ki je zaznala cilje |
| **Odbojnost** | Kalibrirane slike odbojnosti | Samo v projektih Survey3 — projekti LATTICE namesto tega uporabljajo `RAW (Reflectance)`, zato mreža nikoli ne prikazuje dveh gumbov za odbojnost |
| **Belina uravnotežena** | Produkt z uravnoteženo belino (kamere RGB) | Po obdelavi |
| **Popravljeno zaradi vinjetiranja** | Nekalibrirana nadomestna slika s popravkom zaradi vinjetiranja | Po izvedbi, pri kateri ni bilo mogoče uporabiti kalibracije odbojnosti in je bila *korekcija vinjetiranja* vklopljena |
| **Odziv senzorja** | Nekalibrirana nadomestna rešitev z odzivom senzorja | Enako, vendar z izklopljeno *korekcijo vinjetiranja* |
| **`RAW (<INDEX> Index)`** | Ena tipka na izračunan indeks | Po izvedbi z nastavljenimi indeksi |
| **`<INDEX> LUT`** | Ena tipka za vsak indeks z barvnim preslikavanjem | Po izvedbi s konfigurirano LUT |
| **`<Index> <Index\|LUT> <NNN>`** | Ena tipka na izvozno izvedbo [Index/LUT Sandbox](index-lut-sandbox.md) | Takoj po končanem izvozu v peskovnik |

### Gumbi ravni LATTICE

Projekti, ki vsebujejo posnetke LATTICE, dodajo te gumbe, označene z imenom ravni namesto z imenom izdelka:

| Gumb | Raven |
| --- | --- |
| **RAW (Original)** | Izvorni surovi okvir, kot je bil uvožen |
| **RAW (Radiance)** | Spektralna sevanja v formatu Float32, W/m²/sr/nm |
| **RAW (Reflectance)** | Odbojnost v formatu uint16, 32768 = ρ 1,0 |

`RAW (Original)` je na voljo takoj ob uvozu — ne potrebuje nobene obdelave. Kadar uvoz LATTICE sploh nima osnovnega gumba (osnovna datoteka vsakega zajema je njegov surovi okvir), se mreža sama premakne na prvi razpoložljivi gumb ravni, tako da je označitev na orodni vrstici usklajena s tem, kar vidite.

Izvozi Chloros z dvema ravnema **nimajo lastnega gumba mreže**:

* **Debayered** — pogled `RAW (Original)` se že prikaže brez bayerjevega filtra, zato bi bil drugi gumb na vizualno identični sliki le motnja. Produkt `RAW (Debayered)` se še vedno zapiše na disk in ga je še vedno mogoče izbrati iz spustnega menija plasti na celotnem zaslonu.
* **Predogled** — na kamerah RGB je predogled registriran kot sloj `White Balanced`, ki ima gumb. Na multispektralnih kamerah je registriran kot `RAW (Preview)` in je dostopen iz spustnega menija slojev za polni zaslon.

{% hint style="info" %}
Ti gumbi ravni se prikažejo le za projekte, ki dejansko vsebujejo okvirje LATTICE. Projekti Survey3 registrirajo nekatera enaka interna imena plasti, zato so gumbi zanje odfiltrirani, tako da mreža Survey3 ohrani svoj znani nabor `JPG / Targets / Reflectance`.
{% endhint %}

S klikom na miniaturno sliko mreže se odpre [Pregledovalnik slik](opening-an-image-full-screen.md) v polnem zaslonu za **isti izdelek, ki ga prikazuje mreža** — če je mreža nastavljena na `Targets`, se s sličico odpre izvožena ciljna slika.

<figure><img src="../.gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: This GIF predates the LATTICE level buttons and the toolbar group separators. Reshoot on a LATTICE project cycling base -> RAW (Original) -> RAW (Radiance) -> RAW (Reflectance) -> an index button, so the new button set and the level names are visible. -->

***

## Razvrščanje projekta LATTICE v skupine: po sprožilcu ali po kameri

Snemanje v nizu ustvari več slik istega trenutka iz različnih kamer. Razvrščanje določa, kako jih mreža razporedi. Oba načina prikazujeta zložljive naslovne vrstice v polni širini; **vsaka skupina se začne razširjena**, Chloros pa si zapomni tiste, ki jih zaprete. Stanje zložljivosti se spremlja ločeno za vsak način, zato zapiranje skupine v načinu »Po kameri« ne zapre ničesar v načinu »Po sprožilcu«.

### Po kameri (privzeto)

Ena skupina na kamerni modul. Zgornja vrstica prikazuje model kamere in serijsko številko (`LATT-M3M — <serial>`) ter število fotografij. Slike znotraj skupine so razvrščene kronološko po dogodku zajema.

V tem načinu orodna vrstica dobi tudi en **gumb za filtriranje kamer na kamero**, označen z `MODEL (SERIAL)`. Vse kamere so na začetku izbrane; s klikom na gumb se izbira te kamere prekliče in njena skupina se odstrani iz mreže. To je hiter način za pregled enega pasu skozi celoten let.

### Po sprožilcu

Ena skupina na posnetek – niz okvirjev, ki so jih vsi moduli posneli ob istem sprožilcu. V glavi so prikazani čas posnetka, število kamer, ki so prispevale k posnetku, ter oznaka za vsak model kamere v skupini. Ploščice znotraj skupine so razvrščene po serijski številki kamere, tako da je isti pas za vsak sprožilec v istem stolpcu.

<!-- SCREENSHOT-NEEDED: Image grid in Per Trigger mode for a 3-camera LATTICE array, showing two consecutive trigger groups with their header bars (chevron, capture timestamp, "3 cameras", and the three model badges) and one group collapsed to show the closed state. -->
Slike, ki niso v formatu LATTICE, v mešanem projektu niso združene v skupine – prikazujejo se kot navadne ploščice za skupinami.

***

## Miniature v mreži sledijo velikosti bloka GSD

Če ste v stranskem meniju zavihka »slika« nastavili velikost bloka **GSD (px)**, se sličice v mreži prikazujejo v isti ločljivosti na tleh – ne le v pogledu na celoten zaslon. Velikost bloka 8 pomeni, da je vsak prikazan piksel povprečje bloka 8 × 8 izvornih pikslov, in sicer povsod v aplikaciji, kjer se slika prikazuje.

Ker je ploščica že na začetku široka le nekaj sto pikslov, grobe velikosti blokov ne povzročajo več vidne razlike v mreži že precej prej, kot se to zgodi v pogledu na celotnem zaslonu: okvir velikosti 4000 px, narisan v ploščico velikosti 160 px, že vsebuje približno 25 izvornih pikslov na prikazan piksel. Za sam nadzorni element glej [Odpiranje slike v polnem zaslonu](opening-an-image-full-screen.md#gsd-block-size).

***

## Povezane strani

* [**Odpiranje slike v polnem zaslonu**](opening-an-image-full-screen.md) — pregledovalnik v polnem zaslonu, vrednosti kazalca in histogram
* [**Sloji slike**](image-layers.md) — spustni meni slojev znotraj pregledovalnika v polnem zaslonu
* [**Indeks/LUT Sandbox**](index-lut-sandbox.md) — ustvarjanje in izvoz vizualizacij indeksa
* [**Nastavitve projekta**](../project-settings/project-settings.md) — stikala za izvoz, ki določajo, kateri izdelki sploh obstajajo

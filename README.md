---
metaLinks: {}
---

# Uvod

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros
je programska aplikacija podjetja [MAPIR
](https://www.mapir.camera) za obdelavo multispektralnih posnetkov, nadzor strojne opreme MAPIR
v realnem času in beleženje podatkov senzorjev. Chloros
1.3.0 podpira celotno družino izdelkov MAPIR
:

* **kamere Survey3** — obdelava posnetkov v formatu RAW+JPG v kalibrirane karte odbojnosti in indeksov vegetacije. Glejte [Podprte kamere](supported-cameras.md).
* **Kamere LATTICE** — v realnem času priključite GigE-multispektralne kamere, posamično ali kot sinhronizirane nize več kamer: predogled, zajem in obdelava v kalibrirane izdelke sevanja in odbojnosti. Glejte [poglavje o LATTICE](lattice/README.md).
* **Svetlobni senzorji DAQ** — spektralni senzorji DAQ-U (USB), DAQ-M (Bluetooth) ter DAQ-E in DAQ-E-S (Ethernet): kalibrirani spektri v realnem času, `.daq` posnetki ter osvetlitev od zgoraj za obdelavo odbojnosti. Glejte [poglavje o DAQ](daq/README.md).

{% hint style="success" %}
**Novosti v Chloros
1.3.0**: AI-pomočniki lahko upravljajo Chloros
prek `chloros-mcp`, enake funkcije pa ponujajo tudi CLI
, SDK
in `chloros-mcp`. Izdelki LATTICE index so izboljšani, polni nizi senzorjev delujejo hitreje in delujejo v vsakem omrežju, backend pa med ukazi neprekinjeno deluje. Oglejte si spodnji seznam novosti in [prenesite](download.md) namestitvene programe.
{% endhint %}

{% hint style="info" %}
**Uporabljate Chloros
s pomočjo AI-asistenta?** Ta priročnik je namenjen prav temu. Asistentu naročite:

* `https://mapir.gitbook.io/chloros/llms.txt` — strojno berljiv indeks vseh strani.
* Katero koli stran v surovem formatu Markdown — dodajte `.md` k njenemu URL
-ju (npr. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [Referenca CLI
](reference/cli-reference.md) in [Referenca SDK
](reference/sdk-reference.md) — popolne referenčne strani z natančnimi vrednostmi, napisane za uporabo v velikih jezikovnih modelih (LLM).

Primer ukaznega niza: *»Preberi https://mapir.gitbook.io/chloros/reference/cli-reference.md,, nato napiši skript, ki se prijavi in obdela mapo ~/flights/flight_001 v GeoTIFF-datoteke z odbojnostjo in NDVI
.«*

Ali pa neposredno zaženite Chloros
: `chloros-mcp`, strežnik MCP, nameščen skupaj z SDK
, deluje s Claude Desktop, Claude Code, Cursor in drugimi odjemalci MCP.

Celoten vodnik: [Uporaba Chloros
a z AI-pomočniki](ai-assistants.md).
{% endhint %}

***

## Novosti v Chloros
u 1.3.0

* **AI-pomočniki lahko upravljajo Chloros
.** `chloros-mcp`, MCP-strežnik, nameščen s Python
SDK
, omogoča programom Claude Desktop, Claude Code, Cursor in drugim MCP-odjemalcem obdelavo slik ter upravljanje kamer LATTICE, matrik in svetlobnih senzorjev DAQ. Glejte [Uporaba Chloros
z AI-pomočniki](ai-assistants.md).
* **CLI
, SDK
in `chloros-mcp` ponujajo enake funkcije.** Zaradi varnosti ostajajo v CLI
le prijava, konfiguracija DAQ-E in neposredni DAQ-strežniki. Nekateri skripti iz različice 1.2.0 zahtevajo spremembe: glejte **Spremembe v različici 1.3.0** na vrhu [Referenčnega priročnika za CLI
](reference/cli-reference.md) in [Referenčnega priročnika za SDK
](reference/sdk-reference.md).
* **Izboljšani indeksni izdelki LATTICE.** NDVI
in drugi indeksni izdelki, izvoženi iz posnetkov LATTICE, so izboljšani v vseh načinih obdelave.
* **Možnosti namestitvenega programa za Windows
.**V razdelku**Sistemski nastavitve**namestitveni program ponuja možnosti**Omogoči dolge poti v Windows
u**in**Izključi backend Chloros
iz skeniranja s programom Microsoft Defender**, ki sta privzeto izbrani. Prva omogoča, da so projektne mape globlje od 260 znakov; druga preprečuje, da bi skeniranje v realnem času upočasnilo zajem kamere (približno 30 % hitrosti zajema pri visokih hitrostih). Namestitveni program zdaj tudi dodaja mapo CLI
v `PATH` le enkrat in odstrani podvojene vnose, ki so jih pustili prejšnji namestitveni programi. Glej [Prenos](download.md).
* **Pregloboke projektne mape se zavrnejo že na začetku.** Če so dolge poti v Windows
u izklopljene (privzeta nastavitev Windows
a), se projektna mapa, ki je pregloboka za omejitev poti na 260 znakov, zavrne še pred zagonom programa, pri čemer se prikaže najdaljša projektna mapa, ki ustreza omejitvi, ter navodila za vklop dolgih poti. Prej takšen zagon ni ustvaril nobenih izdelkov. Če so dolge poti vklopljene, izdelki v zelo globokih mapah zdaj ohranijo svoje metapodatke (oznake GPS in XMP).
* **Omejevalne maske.** Kadar razmešavanje pasov v produktu sevanja ali odbojnosti LATTICE ustvari negativno vrednost pasu, se ta vrednost omeji na nič, maska na piksel v mapi `clamp_masks/` poleg map s produkti pa beleži, kateri piksli in pasovi so bili prizadeti. Produkt poimenuje svojo masko v oznaki `Chloros:UnmixClampMask`. Pogledi v realnem času se zdaj obrezujejo točno tako, kot se obrezujejo pri izvozu. Glejte [Formati izhodnih slik](output-image-formats.md).
* **Izdelki se opisujejo sami.** Indeksni izdelki nosijo svoje lastne oznake obdelave (`Chloros:ProcessingLevel=index`, indeksni izraz in informacijo, ali so bili izračunani iz sevanja ali odbojnosti) namesto oznak izvirnih surovih podatkov. Izdelki Survey3
nosijo oznake obdelave Chloros
v vsakem načinu obdelave. Surove in debayered datoteke LATTICE ohranijo izvorno orientacijo senzorja in svojo orientacijo zabeležijo v XMP; pregledovalnik slik jih obrne le za prikaz.
* **Paketi za Linux
so dobavljeni v razpakirani obliki.** Backend in CLI
sta nameščena kot mape pod `/usr/lib/chloros/`, zato se ob vsakem zagonu ne razpakirata več sama. Na x86_64 backend ne porabi več približno 100 sekund za zagon, velikost `.deb` pa je približno polovica prejšnje. Priloženi ExifTool zdaj deluje na čisti namestitvi Ubuntuja. `.deb` zdaj namesti tudi Python
ov SDK
za sistem `python3` (namestite ga z `sudo apt install ./chloros_1.3.0_<arch>.deb`), poveča sprejemne pomnilnike vtičnic jedra, ki jih potrebujejo tokovi kamere, ter doda storitev backenda, ki teče pod vašim uporabniškim računom. Paket za Jetson, `chloros_1.3.0_arm64_jp6.deb`, je zasnovan za JetPack 6 in obdeluje podatke na grafičnem procesorju (GPU) naprave Jetson z JetPackovo CUDA-različico PyTorcha. V sistemu Linux
vsaka izvožena slika zdaj enkrat doseže disk (zapiše se in označi v pomnilniku, nato pa se kopira na mesto): na Jetsonu, kjer hitrost določa disk, izvedba LATTICE z 288 slikami traja 14 minut namesto 24. Glej [Namestitev v sistemu Linux
](linux/linux-installation.md).
* **OCN
-kamere LATTICE v sončni svetlobi merijo natančneje.** Sijaj, odbojnost in indeksi se po ločitvi pasov hitro popravijo, tako da siva tarča v soncu kaže vrednost, ki je blizu nevtralnega NDVI
. RGN
-kamere ostajajo nespremenjene.
* **Prijava napak znotraj aplikacije.** Ikona napake na vrhu zavihka »Debug Log« (Dnevnik odpravljanja napak) pošlje podjetju MAPIR
opis, do 10 slik, dnevnike in podrobnosti o sistemu. Glej [GUI: Navigacija](navigation.md).
* **LATTICE-matrike.**Matrike privzeto ohranjajo polni senzor, stisnjen 12-bitni prenos (`--packed`, privzeto `auto`) pa poveča hitrost snemanja pri polnem senzorju za približno tretjino; niz deluje v vsakem omrežju s hitrostjo snemanja, ki jo omrežje dopušča (počasna povezava zniža hitrost snemanja, namesto da bi zavrnila niz ali zmanjšala število njegovih posnetkov); niz, katerega kamere so bile izklopljene in ponovno vklopljene, se samodejno obnovi, pri čemer se PTP ponovno vklopi; samodejna osvetlitev v šibki svetlobi ne znižuje več te hitrosti (najprej zapolni časovni interval, nato pa poveča ojačanje); Neprekinjeno zajemanje sledi hitrim nizom; nadzor**Highlights** za RGB
kamere; ter `ArraySession` v SDK
pridobi `grab()`, `frame_stream()` in `continuous()`. Glejte [oddelek LATTICE](lattice/README.md).
* **Stabilnejša samodejna osvetlitev.** Izpostavljenost ostane nespremenjena pri konstantni svetlobi in se stabilizira v nekaj sekundah po spremembi; majhni svetli deli ne temnijo več prizora; pri ročni izpostavljenosti ostane ojačanje nastavljeno na »Auto«; kamera, ki preneha odgovarjati v močnem soncu, pa se ohrani na ravni, ki ustreza njeni svetlobi, ki jo odčita svetlobni senzor ali, če senzor ni priključen, lastni okvirji kamere. Glejte [Nastavitve kamere](lattice/camera-settings.md).
* **Senzorji svetlobe.** **Podpora za DAQ-E-S:** njegov fiksni difuzor ne zahteva popravka pokrova, vsaka odčitana vrednost pa vsebuje podatke o nagibu, vrtenju in naklonu enote. Posnetek navaja, ali je bil pokrov izbran ali predpostavljen, obdelava pa opozori na predpostavljenega: modeli DAQ-U, DAQ-M in DAQ-E se dobavljajo brez pokrova, zato poimenujte tistega, ki je nameščen. Zapisi v realnem času `.daq` ostanejo ohranjeni tudi ob izpadu napajanja (shranite datoteko `.daq-journal` skupaj z njenim `.daq`), DAQ-E, ki preneha pošiljati podatke, pa se ponovno poveže z v zapisu označeno vrzeljo. Projekt, ki vsebuje le posnetke svetlobnih senzorjev, jih prikaže in obdeluje prek aplikacije in CLI
, posnetek, narejen z `chloros_scripts`, pa je naveden kot DAQ, ki ga je ustvaril, z omejitvijo, ki jo lahko nastavite. Vsak posnetek shrani vrednost DAQ, ki jo je uporabil. Glejte [poglavje DAQ](daq/README.md).
* **Obdelava.**Okvirji LATTICE začnejo z obdelavo med zaznavanjem cilja. Referenčne vrednosti cilja in DAQ se določijo za vsak pas posebej. Premaknjen projekt ponovno poveže svoje datoteke. Izvozi 8-bitnega zajema so 8-bitni, v novem formatu**TIFF
(8-bit)**. Survey3
, ki se izvede brez kalibracijskega cilja, se shrani kot nadomestni izdelek (`Vignette_Corrected_Images` ali `Sensor_Response_Images`), ne pa kot odbojnost. Oznake DLS po pasovih za sliko odbojnosti so vedno oznake njenega lastnega okvirja. Funkcija »Texture Aware« je zavrnjena za zajeme LATTICE, ki vedno uporabljajo standardni postopek debayeringa.
* **CLI
in SDK
.** Relativne izhodne poti se zdaj nahajajo v mapi, iz katere izvajate ukaz ali skript (nameščen backend jih je prej reševal v svoji lastni namestitveni mapi, kar je povzročalo napake). Backend deluje med ukazi in skripti ter se samodejno ustavi po 30 minutah neaktivnosti; to upravljata `chloros-cli backend status` in `backend stop`. Ukazi za zajem navedejo imena vseh datotek, ki jih niso mogli zapisati, in se zaključijo s kodo 1, ponavljajoči se zajemi pa se ustavijo, ko se disk zapolni. Uspešno izvajanje `process` poroča o `Image products written: N (plus M clamp masks)`. V sistemu Windows
`ChlorosLocal` iz SDK
-ja / `process_folder` poišče nameščen backend (in uporabi tistega, ki že teče), `read_image_audit_tags` pa deluje iz navadnega `pip install`.

***

## Novosti v Chloros
u 1.2.0

* **Upravljanje kamere v živo — nov zavihek »Kamere«.** Povežite kamere LATTICE po eno ali kot sinhronizirane nize več kamer (časovna sinhronizacija PTP, zajem, sprožen s strojno opremo), s prekrivnimi v živo predogledi, histogrami po pasovih, pametno samodejno osvetlitev, kalkulatorjem indeksa v živo in posodobitvami programske opreme kamere znotraj aplikacije.
* **Svetlobni senzorji — nov zavihek »Svetlobni senzorji«.** Priključite senzorje DAQ-U (USB), DAQ-M (Bluetooth) in DAQ-E (Ethernet); si oglejte kalibrirane spektre v živo (W/m²/nm), shranite datoteke `.daq` v svoj projekt, izberite profile korekcije kapice in posodobite firmware DAQ-E prek omrežja.
* **Načini zajemanja in snemalniki.** Enkratno / neprekinjeno / intervalno zajemanje ter način »Fastest Capture« (najhitrejše zajemanje) izključno za surove podatke; izbira po posameznem projektu, katere kamere in vrste izvoza ustvari »Capture All«; matrični snemalniki za indeksne videoposnetke za spremljanje in surove serije posnetkov za analizo z izdelavo videoposnetkov brez povezave.
* **Obdelovalni tok LATTICE.** Uvozite mape zajemov LATTICE in vsak surovi okvir razčlenite v izdelke brez bayerjevega filtra, predogled, sevanje float32 (W/m²/sr/nm) in odbojnost s stikali za vsak izdelek posebej. Odbojnost lahko izhaja iz kalibracijskega cilja znotraj okvirja ali iz navzdol usmerjenega sevanja DAQ; pri izvozih se uporabi poravnava matrike; manjkajoča tovarniška kalibracija se samodejno prenese na podlagi serijske številke kamere.
* **Projekti si zapomnijo strojno opremo.** Priključene kamere in svetlobni senzorji se shranijo skupaj s projektom (`cameras.json` / `sensors.json`) in se ob ponovnem odprtju projekta ponovno povežejo s shranjenimi nastavitvami. Glejte [GUI: Projekti](projects.md).
* **Izboljšave v pregledovalniku slik.** Odčitavanje pikslov/indeksa kazalca s pravilnim prilagajanjem odbojnosti za vsako datoteko, histogrami plasti, drsnik za združevanje GSD, načini mreže »na sprožilec« / »na kamero«, pogledi izdelkov LATTICE ter izvoz indeksa/LUT v peskovnik na disk.
* **CLI
in SDK
sta bila močno razširjena.** Nove družine ukazov `lattice`, `daq pool-*`, `project` in `time-sync`; nove možnosti `process` (`--input-level`, stikala za posamezne izdelke, `--reflectance-source`, zastavice za poravnavo matrik); povezovalni ročaji SDK
-ja za pametno povezovanje (`connect_camera` / `connect_array` / `connect_daq_sensor`), ki samodejno zaženejo ozadje; avtomatizacija `open_project()`; paket SDK
je priložen namestitvenim programom in objavljen na PyPI kot `chloros-sdk`.
* **Pojasnjena semantika napak.** Izvedba `chloros-cli process`, ki je zahtevala izdelke, a ni zapisala nobenega, zdaj jasno poroča o napaki in se zaključi z izhodno vrednostjo, ki ni nič; uspešne izvedbe poročajo, koliko slikovnih izdelkov so zapisale.
* **Nova razporeditev izhodnih podatkov.** Izdelki se shranijo v mape `<project>/<camera>/<format>/<Product>_Images/` in ohranijo ime izvorne datoteke — izdelek identificira mapa, ne pa končnica imena datoteke. Glej [Formati izhodnih slik](output-image-formats.md).
* **Več vhodnih podatkov, načrtov in jezikov.** Podpora za vhodne podatke `.dng`; vseh 38 jezikov vmesnika je v celoti na voljo; omejitve strojne opreme po načrtu z brezplačno (brez prijave) uporabo do 4 kamer in 2 svetlobnih senzorjev.
* **Zanesljivost.** Funkcija »Stop Processing« se čisto ustavi s poštenim povzetkom izvedbe, projekti z več kamerami izvozijo podatke vsake kamere, nadgradnje namestitvenega programa pa vas ne odjavijo več.***

Program Chloros
je na voljo v 3 uporabniških vmesnikih:

## Chloros
: namizna aplikacija z grafičnim uporabniškim vmesnikom

Samostojno okno z vsemi funkcijami, vključno z zavihkoma »Kamere v živo« in »Svetlobni senzorji«. _Samo za Windows._

## [Chloros
CLI
: vmesnik ukazne vrstice](CLI.md)

Paketna obdelava prek ukazne vrstice ter ukazi v realnem času `lattice`, `daq pool-*`, `project` in `time-sync`. Idealno za avtomatizacijo, pisanje skriptov in delovanje brez grafičnega vmesnika. Na voljo na **Windows
, Linux
amd64 in Linux
arm64 (NVIDIA Jetson)**. _Za dostop do CLI
je potrebna plačljiva stopnja Chloros
+._

## [Chloros
API
: Python
SDK
](api-python-sdk.md)

Programski vmesnik v jeziku Python
za avtomatizacijo in prilagojene delovne tokove: obdelava celotnega procesnega toka, sej v živo s kamero/nizom kamer, sej z DAQ-senzorji ter avtomatizacija shranjenih projektov. Namesti se skupaj s paketom za namizje/CLI in je objavljen tudi kot `pip install chloros-sdk`. _Za dostop do API je potrebna plačljiva stopnja Chloros
+._

***

## Podprte platforme

| Platforma | GUI | CLI
| Python
SDK
|
| --- | --- | --- | --- |
| **Windows
10/11 (x64)** | Da | Da | Da |
| **Linux
amd64 (x86_64)** | Ne | Da | Da |
| **Linux
arm64 (NVIDIA Jetson)** | Ne | Da | Da |

Navodila za namestitev v sistemu Linux
najdete v razdelku [Linux
in robno računalništvo](linux/linux-overview.md).

***

## Začnite v treh korakih

1. **Namestitev** — prenesite in zaženite namestitveni program za svojo platformo. Glejte [Prenos](download.md).
2. **Prijava (neobvezno za grafični vmesnik)** — grafični vmesnik obdeluje slike brezplačno tudi brez računa. S [prijavo v Chloros
+](chloros+-login.md) odklenete vzporedno obdelavo, pospeševanje z GPU, višje omejitve naprav ter dostop do CLI
in SDK
.
3. **Ustvarite svoj prvi projekt** — odprite Chloros
, ustvarite [nov projekt](projects.md), [dodajte svoje slike](processing-images-gui/adding-files-to-a-project.md) in [začnite z obdelavo](processing-images-gui/starting-the-processing.md). Če želite namesto tega upravljati z dejansko strojno opremo, odprite zavihek »Kamere« ali »Svetlobni senzorji« — glejte [GUI: Navigacija](navigation.md).

***

## Chloros


Čeprav je Chloros
za večino nalog brezplačen, boste morda ugotovili, da potrebujete več. V tem primeru vam lahko koristi plačljiva licenca za Chloros
+. Z licenco Chloros
+ lahko odklenete nove funkcije, kot so:

* **Večnitna obdelava**: znatno pospešite obdelavo slik pri večjih projektih s hkratno obdelavo slik skozi obdelovalno verigo.
* **Pospeševanje z GPU (CUDA)**: izkoristite današnje možnosti večjega pomnilnika GPU za dodatno pospešitev obdelovalne verige slik. Za najboljše rezultate priporočamo 4 GB ali več VRAM-a.
* **Chloros
+**[**CLI**](CLI.md) **Dostop**: zaženite Chloros
+ iz ukazne vrstice, da ga avtomatizirate in integrirate v svojo lastno programsko opremo. Na voljo na vseh plačljivih ravneh; uveljavljeno na strani strežnika.
* **Chloros
+**[**API**](api-python-sdk.md) **Dostop:** za programsko upravljanje zaženite Chloros
+ iz Python
a, kar omogoča nemoteno integracijo z vašimi raziskovalnimi procesi, delovnimi tokovi za analizo podatkov in prilagojenimi aplikacijami. Na voljo na vseh plačljivih ravneh; izvršuje se na strežniški strani.
* **Višje omejitve strojne opreme**: hkrati priključite več kamer in svetlobnih senzorjev. Brez prijave grafični vmesnik (GUI) omogoča priključitev do 4 kamer in 2 svetlobnih senzorjev DAQ; plačljivi paketi povečajo obe omejitvi:

| Paket | Kamere | Svetlobni senzorji DAQ |
| --- | --- | --- |
| Iron (brezplačno, brez prijave) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Uporaba več naprav**: vsaka licenca Chloros
+ omogoča registracijo 2 ali več naprav. Za upravljanje registriranih naprav uporabite svoj račun MAPIR
Cloud. Podporo za več naprav lahko dodate z nadgradnjo licence Chloros
+.
* **Napredna metoda debayeringa, ki upošteva teksturo:** visokokakovosten debayering, ki upošteva robove, v kombinaciji z modelom za odstranjevanje šuma na podlagi umetne inteligence (AI) in strojnega učenja (ML), ki odstrani skoraj ves šum pri debayeringu.
* **Prilagojene formule za multispektralne indekse:** vnesite prilagojene multispektralne indekse v rastrske kalkulatorje Chloros
, tako za obdelavo kot za okolje za pregledovanje slik.
* **Linux
in obdelava na robu omrežja:** zagnajte Chloros
na platformah Linux
x86_64 in ARM64, vključno z NVIDIA Jetson, za obdelavo na terenu in na robu omrežja. Oglejte si [Pregled Linux
a](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Cene in prijava za Chloros+</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.3.0 build so the banner prints "Chloros CLI 1.3.0". -->

# Nastavitve kamer

Zavihek **Kamere**je vmesnik za upravljanje v realnem času za kamere LATTICE v programu Chloros: glavno območje za prikaz slike v živo, kjer se vsaka priključena kamera prikaže kot ploščica v realnem času, ter stranski trak, ki se premika med tremi stranmi –**seznamom kamer**,**okno z nastavitvami**(nastavitve za posamezno kamero, niz kamer ali zajem – po ena naenkrat) ter**indeksni kalkulator**. Ta stran opisuje vsak element upravljanja na seznamu kamer, v oknu nastavitev po posamezni kameri in v oknu nastavitev niza. Načini zajemanja, izbira vrste izvoza in potek »Zajemi vse« so opisani na spremljevalni strani [Nastavitve in načini zajemanja](capture.md).

Zavihek »Kamere« se prikaže v stranskem meniju, ko je strežniški del Chloros pripravljen. Vsi spodnji elementi za upravljanje komunicirajo z lokalnim strežniškim delom prek `127.0.0.1:5000`; spremembe se takoj uporabijo za kamero v živo, razen če ni drugače navedeno.

## Vrste kamer, uporabljene na tej strani

Upravljalniki se prikazujejo ali skrijejo glede na to, katera kamera je izbrana. Priročnik ves čas uporablja naslednje izraze:

| Izraz | Pomen | Filtrski kanali |
| --- | --- | --- |
| **Kamera RGB** | LATTICE M3C s filtrom FRGB (model vsebuje `-FRGB`) | Red / Green / Blue |
| **Bayer multispektralna** | LATTICE M3C s FRGN, FOCN ali FNGB | FRGN: Red / Green / NIR · FOCN: Orange / Cyan / NIR · FNGB: NIR / Green / Blue |
| **Mono (M3M)** | LATTICE M3M — en ozkopasovni filter, en kalibriran pas | Enopasovni |
| **Član niza** | Kamera, priključena kot del sinhroniziranega niza (združen ali ločen prikaz) | Glede na filter |

Kamere RGB so obdelane fotometrično (belo ravnovesje, barvni profili, gama); multispektralne in mono kamere so obdelane radiometrično, pri čemer se fotometrični nadzor preskoči. Člani matrice posredujejo nastavitve na ravni toka (format pikslov, ločljivost, združevanje pikslov, sprožilec, hitrost snemanja) posredujejo nizu — te vrstice postanejo v oknu za posamezno kamero samo za branje in se namesto tega premaknejo v okno nastavitev niza.

## Glavno območje

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

prikaza Če ni priključenih kamer, območje prikaza prikazuje uvodno okno **»Povežite kamero, da začnete&quot;**z dvema gumboma:**Poveži kamero**(zelen, odpre pogovorno okno za povezavo posamezne kamere) in**Poveži niz** (modro, odpre pogovorno okno za povezavo niza). Sama pogovorna okna za povezavo so opisana v [Povezovanje kamer](connecting.md); koncepti nizov (sinhronizacija, ravni, pasovna širina) v [Nizi z več kamerami](arrays.md). Ko odprete shranjen projekt, ki vsebuje kamere, se namesto tega prikaže ikona vrtenja z napisom »Ponovno odpiranje N shranjenih kamer…«, medtem ko Chloros obnavlja tokove iz zadnje seje.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Zgornja vrstica

| Upravljalnik | Kaj počne |
| --- | --- |
| **Preklop načina prikaza**| Preklaplja med**mrežnim prikazom**(vse ploščice kot celice) in**seznamskim prikazom** (nizi v polni širini na vrhu, ENA aktivna kamera spodaj). Orodni namigi: »Preklopi na mrežni prikaz« / »Preklopi na seznamski prikaz«. |
| **Zaklep mreže**(ključavnica) | Privzeto**zaklenjeno** — ploščice so fiksirane na mestu. Odkleni, da lahko s povlekanjem preurediš ploščice v katerikoli prostor (vrzeli ostanejo ohranjene). Mreža se samodejno ponovno zaklene vsakič, ko se priključi nova kamera. Nasveti: »Odkleni mrežo (omogoči vlečenje ploščic)« / »Zakleni mrežo (zamrzni ploščice na mestu)«. |
| Drsnik **Povečava vira** | Velikost ploščice, od 60 pikslov do polne širine kontejnerja. Celice ohranjajo razmerje stranic 4:3. Pri širini celice pod 200 pikslov se prekrivni napisi z imenom in številom sličic na sekundo skrijejo, da ploščica ostane čista. |

### Plakete v živo

Vsaka kamera upodablja sestavljeno plaketo v živo; kamera lahko dodatno prikaže tri plakete v sivih odtenkih z **razdelitvijo po kanalih** (glej [Razdelitve po kanalih](#display-overlays-drawn-over-the-live-feed)), matrice pa upodabljajo združeno plaketo. Aktivna ploščica ima obroč izbire v barvi kamere (ali niza).

Ob preletu z miško nad ploščico se prikaže gumb za zapiranje **X**:

* Zapiranje **sestavljene** ploščice, medtem ko njene razdeljene ploščice po kanalih ostanejo vidne, le skrije sestavljeno ploščico.
* Zapiranje **zadnjega vidnega polja samostojne kamere** prekine povezavo s to kamero.
* **Polja z razdelitvijo, ki so del združenega niza, nikoli ne prekinejo povezave** s kamero – le jo skrijejo.

Ko je mreža odklenjena, povlecite katero koli polje na katero koli mesto; razpored se shrani skupaj s projektom.

## Stranski pas – seznam

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

kamer Prva stran stranskega pasu prikazuje seznam vseh povezanih kamer in nizov:

* **Poveži kamero**(zelena) /**Poveži niz** (modra, med skeniranjem prikazuje »Zaznavanje...«). Obe sta onemogočeni, dokler je odprto pogovorno okno za povezovanje.
* **Zajemi vse** (rdeče) — zajame vse kamere na seznamu z izbirnimi tipi izvoza, določenimi v nastavitvah zajemanja. Zahteva odprt projekt. Podrobno opisano v [Nastavitve in načini zajemanja](capture.md).
* **Zobnik nastavitev zajema** (poleg »Zajemi vse«) — odpre [okno »Nastavitve zajema«](capture.md#the-capture-settings-pane). Onemogočeno brez projekta ali med zajemanjem.

### Vrstice kamer

Vsaka vrstica kamere prikazuje barvno označeno obrobo (barva, ki jo je določila kamera), oznako »CAM« — z modro črko **M**(glavna) ali zeleno črko**S** (podrejena) za člane niza — ter prikazno ime. Privzeto ime je `LATTICE-MODEL (serial)`; preimenujte ga v oknu nastavitev za posamezno kamero. Gumbi v vrstici:

| Gumb | Učinek |
| --- | --- |
| **Oko**| Preklop vidnosti. Skrite kamere izginejo iz mreže in so**izključene iz funkcije »Zajemi vse«**. |
| **Zobnik** | Odpre okno nastavitev za posamezno kamero (naslednji odstavek). |
| **Pavza / Predvajanje**| Zamrzne predogled v živo**samo na strani zaslona** — zajemanje v ozadju se nadaljuje. Kamere v pavzi ne morejo zajemati. |
| **X** | Prekini povezavo. Uporabniški vmesnik se posodobi takoj (v najboljšem primeru); sam prekinitveni postopek v ozadju lahko traja 10–30 s. |

### Vrstice v nizu

Vrstica v nizu prikazuje oznako »ARRAY« v barvi niza, ime niza (ki ga je mogoče preimenovati v nastavitvah niza) ter gumb **DAQ · vklop/izklop**—**vklopljeno**, če je nastavljen svetlobni senzor na ravni niza *ali* ima kateri koli član senzor za posamezno kamero; v orodnem namigu je natančno navedeno, kateri senzor kaj prenaša. Kamere članov so navedene z vdolbinami pod njim v svojih vrsticah. Gumbi v vrstici niza: **oko**(skrije/prikaže VSE člane hkrati),**zobnik**(okno nastavitev niza),**X**(odklopi celoten niz).

Stanje svetlobnega senzorja (DLS), ki se uporablja v vrsticah niza in oknu nastavitev niza, ima štiri stanja:**izklopljeno**,**v čakanju**(še ni spektra),**aktivno**(spekter je prispel v zadnjih 3 s) in**zastarelo** — v zadnjih 3 s ni bilo novega spektra, vendar se zadnja vrednost *še vedno uporablja* (vrednosti DAQ na poti zajemanja nikoli ne potečejo).

Samostojne kamere in celotne skupine nizov lahko v stranskem pasu povlečete druga mimo druge, da spremenite vrstni red seznama; posameznih elementov v nizu ni mogoče samostojno premikati.

## Okno nastavitev za posamezno kamero

Odprite ga z **zobnikom** v vrstici kamere. Okno se pomakne čez seznam kamer.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->

**Zgornji del**:**barvni vzorec**(kliknite, da odprete vgrajeni izbirnik barv — nastavi barvo robov stranskega pasu in barvo izbirnega obroča ploščic),**ime**s svinčnikom in gumbom**Preimenuj**(shranjevanje praznega imena vrne privzeto ime `MODEL (serial)`) ter**×** za zaprtje.

### Histogram v realnem času

Na vrhu okna je histogram svetlosti v realnem času, izračunan iz predogleda JPEG pri frekvenci ~8 Hz. Povprečje je tehtano po Bayerjevi metodi — (R+2G+B)/4 — da se ujema z merjenjem avtomatske osvetlitve (AE) same kamere.

* **Prekinjena črta Orange**= ciljna vrednost avtomatske osvetlitve (AE).**Povlecite jo vodoravno, da spremenite ciljno vrednost** — ob sprostitvi se pošlje en ukaz, s povlekanjem pa se način ciljne vrednosti avtomatske osvetlitve (AE) preklopi na ročni.
* **Green neprekinjena črta** = dejanska povprečna svetilnost (kar trenutno zagotavlja avtomatsko osvetljevanje).
* **Gumb RGB** (zgoraj desno): preklaplja med prekrivnimi histogrami po posameznih pasovih, obarvanimi glede na filter kamere (npr. pri FRGN: siva NIR, zelena, rdeča). Na mono (M3M) kamerah je na gumbu napis »MONO« in je onemogočen — mono vedno prikazuje enopasovni histogram svetlobe.
* Oznake na osi X sledijo bitni globini senzorja trenutnega formata pikslov: 0..255, 0..1023, 0..4095 ali 0..65535.

### Vrstice

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

s podatki o kameri | Vrstica | Delovanje |
| --- | --- |
| **Model** | Samo za branje (npr. `LATT-M3C-L87-FRGN`). |
| **Radiometrična kalibracija**| Green**»Aktivna«**oznaka z napisom, ki prikazuje stopnjo kalibracije, hash, datum kalibracije in seznam pasov, naloženih iz kalibracijskega paketa kamere (glej [Tovarniška radiometrična kalibracija](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Skrito za kamere RGB** — te imajo fotometrično kalibracijo beline, ne pa sevanja po posameznih pasovih. |
| **Poročilo o kalibraciji**| Gumb**Prenesi** — odpre PDF-datoteko s certifikatom NIST o kalibraciji kamere za posamezno serijsko številko v pregledovalniku vašega operacijskega sistema. Če certifikat še ni shranjen v predpomnilniku, Chloros namesto tega prikaže namig. |
| **Serijska številka** | Samo za branje. |
| **Firmware**| Prikaže trenutno različico, nato pa poišče razpoložljivo različico za ta model (shranjeno v predpomnilniku po modelu – niz N kamer enkrat preveri strežnik). Stanja: »Preverjanje…« → gumb**»Posodobi na X«**→ »Posodabljanje…« → »Posodobljeno A → B« / »Neuspešno: …« / »Preskočeno: …« / zeleno**»Posodobljeno«**. Opis gumb za posodobitev: »Ponastavitev na tovarniške nastavitve + posodobitev + ponovno programiranje UserSet1. ~2–3 minute; ne prekinjajte povezave.« |
| **IP** | Samo za branje. |
| **Temperatura** | Samo za branje, osvežuje se vsake 3 s. Ob ≥65 °C zasveti oranžno, ob ≥75 °C pa rdeče z opozorilnim znakom ⚠. |
| Polje za potrditev **Kalibracijski cilj** | Omogoča zaznavanje ArUco-cilja odbojnosti s pomočjo validacijske tabele NDVI za vsak panel pod živo sliko (pogled seznama). Velja le za tekočo sejo — vedno se odpre v izklopljenem stanju. |
| Spustni meni **Svetlobni senzor** | Poveže DAQ svetlobni senzor (DAQ-E/M/U, iz seznama na zavihku Svetlobni senzorji) s to kamero za korekcijo osvetlitve z navzdol usmerjeno svetlobo (DLS) in predvidljivo samodejno osvetlitev. Izbira »Nič« prekliče povezavo. Če ni priključenih senzorjev, spustni meni prikaže »(ni priključenih senzorjev — odpri zavihek DAQ)«. Povezava se shrani skupaj s projektom. |

### Osvetlitev in ojačanje

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Vsi številčni vnosi tukaj uporabljajo vrtljive drsnike, ki se pospešijo ob podaljšanem pritisku: dotik = ±1, pritisek &gt;1,5 s = ±10, pridržanje &gt;3 s = ±100. Vrednost se pošlje kameri, ko sprostite prst.

| Nadzor | Obseg / izbire | Privzeto | Velja za | Kaj počne |
| --- | --- | --- | --- | --- |
| **Izpostavljenost (us)**| Najmanjša/največja vrednost kamere v realnem času | Samodejno | Vse | Čas izpostavljenosti v mikrosekundah, s stikalom**Samodejno/Ročno**. Samodejno = neprekinjena samodejna izpostavljenost na strani kamere. |
| **Ojačanje (dB)**| Najmanjša/največja vrednost kamere v realnem času (npr. do 48 dB) | Ročno (izklopljeno) | Vse | Analogno/digitalno ojačanje z lastnim stikalom**Avto/Ročno**. |
| **Ciljna svetlost AE**| 0–255 | 80, način**Avto**| Vse (ureljivo, ko je vklopljena AE ali avtomatska ojačitev) | Svetlost, na katero je usmerjena AE. V načinu**Avto**(privzeto) krmilnik na podlagi histograma sam izbere ciljno vrednost, pri čemer ohranja osvetlitev na 60–75 % maksimalne vrednosti senzorja. Z vnosom vrednosti ali povlečenjem oranžne črte na histogramu se preklopi v**Ročno**. |
| **Izglajevanje AE** | 0,5–40, korak 0,1 | 8,0 | Vse | Dušenje AE. Opomba: »Nižja vrednost = AE reagira hitreje (lahko utripa pri visoki hitrosti snemanja). Višja vrednost = bolj gladko / počasneje.« Vrednosti, ki so precej nižje od privzete, lahko povzročijo utripanje AE in destabilizirajo pretakanje pri visokih hitrostih snemanja; 8,0 je stabilna privzeta vrednost. |
| **Zanimivo območje AE**| Polje za vklop + gumb**Usmeri**| Izklopljeno | Vse | Ko je vklopljeno, AE meri le območje, označeno z zeleno črtkano črto, namesto celotnega okvirja. Gumb**Usmeri** omogoča namestitev z klikom na živo sliko: klik osredotoči območje na 30 % okvirja; klik in povleci pa nariše poljubni pravokotnik (najmanj 5 % × 5 %). Funkcija »Aim« se samodejno izklopi po enem postavitvi. Območje se preslika nazaj v koordinate kamere v skladu z vsako rotacijo/zrcaljenjem, ki ste ga nastavili, in se shrani skupaj s projektom. |
| **Hitrost prilagajanja AE** | 0,1–5, korak 0,1 | 1,0 | Samo za člane skupine | Kako hitro cilj avtomatske AE sledi spremembam svetlosti prizora; pri vrednosti 1,0× se ponovno preveri vsake 2,5 s. |
| **Zaščita svetlih delov** | Stroga (1 %) / Normalna (5 %) / Ohlapna (15 %) | Stroga | Kamere, ki omogočajo to nastavitev | Koliko okvirja se lahko preklopi v belo, preden AE potemni sliko. |

{% hint style="info" %}
**Zahteve glede osvetlitve za multispektralne kamere Bayer (RGN / OCN / NGB):** prizor mora imeti dovolj svetlobe v vseh treh kanalih, sicer kalibracija ne deluje pravilno – ena sama osvetlitev senzorja pokriva vsa tri spektra. Za merjenje svetlobe uporabite svetlobni senzor DAQ ali preklopite na popolnoma monokromatski način (M3M), da vsak pas dobi svojo osvetlitev. Če posnetek to pravilo krši, Chloros to zazna in vas opozori (obvestilo »unmix-clamp«).
{% endhint %}

### Format pik in

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

ločljivost**Člani matrike** prikazujejo vrstici »Current« (format + WxH) in »Binning«, ki sta samo za branje, z opombo »Nastavljeno v nastavitvah matrike« — ponovni zagon toka na enem članu bi prekinil sinhronizacijo, zato se ti parametri upravljajo v [oknu nastavitev matrike](#array-settings-pane).**Samostojne kamere** imajo:

| Nadzor | Možnosti | Kaj počne |
| --- | --- | --- |
| **Pikselni format** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Pikselni format senzorja (bitna globina). |
| **Ločljivost** | Polna / Polovična / Četrtinska | Glede na trenutno združevanje pikslov: Polna = 2048/N × 1536/N za združevanje N×N. |
| **Združevanje pikslov** | 1x1 (ni) / 2x2 / 4x4 | Strojno združevanje N×N — večje vrednosti zmanjšujejo ločljivost, vendar povečajo razmerje signal-šum (SNR) in hitrost snemanja. Sprememba te nastavitve ponovno zažene prenos in ponastavi vse območja zanimanja (ROI) na novo polno vidno polje. |
| **Trenutno** | samo za branje | Dejanska širina (W) in višina (H) ter odmik (x, y), ki veljata. |

### Predogled v živo

Vse v tem razdelku je **namenjeno izključno prikazu**— spreminja tisto, kar vidite v živi sliki, medtem ko shranjeni posnetki ostanejo linearni in nespremenjeni — z eno izjemo:**Vignette** je radiometrična in vpliva tudi na izvoze (opisan spodaj).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->

| Nadzor | Razpon / izbire | Privzeto | Velja za | Kaj počne |
| --- | --- | --- | --- | --- |
| **Ločljivost upodabljanja** | 360p (najhitrejša) / 480p / 720p / 1080p / izvorna ločljivost senzorja (najpočasnejša) | 720p | Vse | Višina, na kateri strežnik izvaja radiometrično verigo predogleda. Nižja vrednost zagotavlja višjo hitrost snemanja brez spremembe vidnega polja. |
| **Indeks**| Potrditveno polje »Omogoči« + zobnik | Izklopljeno | Samo za Bayerjevo multispektralno kamero,**ne** za kamere s kombiniranim nizom | Predogled indeksa žive vegetacije v realnem času. Zobnik odpre skupno okno [Kalkulator indeksa](#index-calculator-pane), v katerem so že vnaprej naloženi naravni pasovi filtra kamere (npr. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Prilagojeni izraz skupaj z LUT (vklopljeno/izklopljeno, privzeta raven 3, privzeta najmanjša vrednost 0,2, privzeta največja vrednost 1) se izračuna za vsak predogledni okvir. Člani kombiniranega niza skrijejo to vrstico — niz ima en skupni indeks. |
| **Bela bilanca** | Izklopljeno / Enkrat / Neprekinjeno + gumb za ponovno zajemanje | Neprekinjeno | Samo RGB | Belina v živo. Gumb za osvežitev ponovno zajame belino iz trenutnega spektra DLS (onemogočeno, če je način izklopljen). |
| **Gama** | Vklopljeno / Izklopljeno | Vklopljeno | Samo RGB | Prikaz gama (γ = 2,2 LUT) v predogledu v živo. Shranjeni posnetki ostanejo linearni. |
| **Odstranjevanje šuma** | Potrditveno polje + jakost 0–100 | Izklopljeno / 50 | Vse (po posamezni kameri, tudi znotraj nizov) | Dvostranski filter v živi predogledu. Višja vrednost = gladkejše, a mehkejše podrobnosti. |
| **Ostrina** | Potrditveno polje + jakost 0–100 | Izklopljeno / 30 | Vse | Maska neostrosti v predogledu v živo, uporabljena kot zadnja. Lahko poveča šum. Samo za predogled. |
| **Vigneta**| Potrditveno polje + jakost 0–100 | Izklopljeno / 0 | Vse | Ročno odstranjevanje preostale vignete (posvetli kotičke), ki se naloži na oceno pametne vignete niza.**Radiometrično — vpliva na v živo prikaz in izvoze**, za razliko od zmanjševanja šuma/ostrine. |
| **Barvni profil** | Raw / Linear / Natural / Enhanced / Custom Temperature | Natural | Samo RGB | Glej spodaj. |
| **Barvna temperatura** | 2000–10000 K, korak 100 | 5500 K | Samo RGB, profil po meri za barvno temperaturo | Priključi belo ravnovesje na fiksno korelirano barvno temperaturo (vhod DLS se ne upošteva). Zadnja izbrana vrednost v kelvinih se shrani tudi ob preklopu med profili. |
| **Nasičenost** | 0–200 (100 = nevtralno) | 100 | Samo RGB | Nasičenost HSV v predogledu v živo. |
| **Kontrast** | 0–200 (100 = nevtralno) | 100 | Samo RGB | Linearni kontrast okoli srednje sive barve v predogledu v realnem času. |
| **Zrcaljenje po vodoravni / Zrcaljenje po navpični osi** | Potrditvena polja | Izklopljeno | Vse | Zrcaljenje predogleda po vodoravni / navpični osi. |
| **Vrtanje**| 0° / 90° / 180° / 270° | 0° | Vse | Vrtanje predogleda. Usmeritev se uporabi na koncu verige predogleda v ozadju —**shranjene posnetke ostanejo v izvorni usmeritvi kamere**, sestavljeni pogledi matrike pa jo ignorirajo. |**Semantika barvnih profilov** (kamere RGB):

* **Raw** — popolnoma preskoči verigo obdelave.
* **Linear** — temni signal + izravnava polja + belina; brez barvne matrike, brez gama.
* **Natural** *(privzeto)* — linearno plus izmerjena matrika barvne korekcije in tonska krivulja, ki se prilagaja sceni.
* **Izboljšano**— Naravno plus živost in lokalni kontrast CLAHE. Dodatni strošek velja**samo za predogled v živo** — shranjeni posnetki vedno dobijo popolno obdelavo ne glede na profil.
* **Prilagojena temperatura** — Naravno z belo ravnovesje, pritrjeno na izbrano vrednost v kelvinih.

{% hint style="warning" %}
Za nastavitve Naravno, Izboljšano in Po meri temperaturo se v oknu prikaže opomba o tonih: posnetki so posvetljeni glede na lastno sceno, zato shranjene *zaslonske* slike niso primerljive med posameznimi posnetki. **Za meritve izvozite sevanje ali odbojnost.**
{% endhint %}

### Prekrivni elementi na zaslonu (risani prek žive slike)

Ti so na voljo le v uporabniškem vmesniku — narišeni so prek videa, nikoli pa ne vplivajo na tok ali posnetke.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Prekrivni element | Upravljalniki | Privzeto | Kaj počne |
| --- | --- | --- | --- |
| **Zebra** | Potrditveno polje + prag 200–255 | Izklopljeno / 250 | Magenta diagonalne črte na odrezanih pikslah. |
| **Križec** | Potrditveno polje | Izklopljeno | Oznaka središča okvirja. |
| **Mreža** | Izklopljeno / 3 × 3 / 9 × 9 | Izklopljeno | Mreža za kompozicijo. |
| **Histogram** | Potrditveno polje + širina 0,10–0,90 okvirja | Izklopljeno / 0,25 | Pas histograma na prenosu. |
| **Vrh ostrenja** | Potrditveno polje + prag 20–200 + barvni vzorec | Izklopljeno / 80 / `#ff5722` | Sobelovo označevanje robov za ostrenje. |
| **Razdelitev kanalov** | »Pokaži razdelitve (Red / Green / NIR)« / gumb »Skrij razdelitve« | Skrito | Dodaja tri neodvisne ploščice s sivimi odtenki po kanalih ob sestavljeni sliki (oznaka gumba sledi filtrskim kanalom kamere). Vsaka razdeljena ploščica je premikljiva in ima barvo robov kamere. Ni na voljo pri monokromatskih kamerah. Shrani se skupaj s projektom. |

### Točkovni merilnik

* Potrditveno polje **Klikni za vzorec**: kliknite na živo sliko, da vzorčite en sam piksel (označi ga križec), ali kliknite in povlecite območje za izračun povprečja pikslov.**Izbriši**izbriše vzorec in križec. Medsebojno izključljivo z načinom**Aim** v AE-ROI.
* Spustni meni **Pokaži**:**Raw (bitna globina)**— izvorne digitalne vrednosti z bitno globino senzorja (npr. 12-bit → 0..4095) — ali**Display (8-bit)** (privzeto). Ko je indeks v živo aktiven, Display namesto tega prikaže izračunano vrednost indeksa (npr. NDVI).
* Na odčitnem panelu so navedene koordinate pikslov, velikost okvirja, format pikslov, bitna globina ter tabela kanalov (Chan / Vrednost / %) z oznakami pasov in valovnimi dolžinami; pari zelene barve po Bayerjevem sistemu so povprečeni; vzorci območij prikazujejo »N px avg«.

Stanje točkovnega merilnika velja le za tekoče sejo.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Prediktivna avtomatska osvetlitev (na podlagi DLS)

Ta razdelek se prikaže le, če je **priključen vsaj en svetlobni senzor DAQ** — reševalec za delovanje potrebuje spektrum v živo, ki pada navzdol.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->

| Nadzor | Obseg | Privzeto | Kaj počne |
| --- | --- | --- | --- |
| **Omogoči** | Potrditveno polje | Vklopljeno (samostojne kamere) | Reševalnik zaprte oblike uporabi spekter DLS skupaj s skalarnimi vrednostmi kalibracijskega paketa kamere, da najsvetlejši pas prinese blizu nasičenja, hkrati pa najtemnejši pas ohrani nad spodnjo mejo SNR — en zapis osvetlitve na rešitev, brez stabilizacijskega cikla. Namenjeno je časovnim posnetkom na sončno energijo, kjer mora biti vsak posnetek pravilno osvetljen. Zadnji del sistema se samodejno vrne na reaktivno avtomatsko osvetlitev (AE), kadar je odčitek DLS zastarel/manjka ali kadar kalibracijski paket ni naložen. |
| **Glajenje (α)** | 0,05–1,0, korak 0,05 | 0,3 | Glajenje zaporednih prediktivnih rešitev (nižja vrednost = bolj gladko). |
| **Odbojnost prizora**| Gumb**Ponovna kalibracija ρ** | — | Ponovno oceni faktor odbojnosti prizora, ki ga uporablja reševalnik. |

{% hint style="info" %}
**Povezava matrike privzeto izklopi prediktivno AE** — za matrike izpostavljenost uravnavata Chloros-ova »pametna AE« ter avtomatska izpostavljenost na strani kamere (z zaščito pred zasičenostjo), medtem ko ocena odbojnosti posamezne scene pri prediktivni AEocena odbojnosti ni zanesljiva v mešanih prizorih. Tukaj jo lahko ponovno vklopite za posamezno kamero, če želite izrecno radiometrično osvetlitev, ki jo poganja DLS.
{% endhint %}

**DAQ-pogojena zgornja meja osvetlitve in AE, vezana na padajoče sevanje.**Neodvisno od zgornjega potrditvenega polja, ko je svetlobni senzor DAQ dodeljen kameri RGB, Chloros izračuna — iz izmerjene absolutne navzdolne obsevanosti — največjo osvetlitev × ojačanje, pri kateri površina s 100-odstotnim odbojem ostane pod pragom izrezanja, in jo uporabi kot**zgornjo mejo**za samodejno osvetlitev. Medtem ko je zgornja meja aktivna, je kamera**vezana na padajočo svetlobo**: deluje v odprtem krogu pri osvetlitvi, izmerjeni na podlagi padajoče svetlobe, z ojačitvijo 0 dB – osvetlitev sledi izmerjeni svetlobi, ne vsebini prizora. Ker zgornja meja lahko le skrajša osvetlitev, sama po sebi ne more povzročiti preklopa. Zgornja meja se samodejno izklopi — in se ponovno vzpostavi običajna avtomatska osvetlitev scene — kadarkoli manjka odčitek DAQ, je zastarel (&gt;30 s) ali temen, ali če se pri vezani osvetlitvi izreže ≥15 % kadra (kar pomeni, da senzor in kamera zaznavata različno osvetljenost). Ni stikala v grafičnem vmesniku; to je standardno obnašanje, kadar je kamera RGB povezana z DAQ.

### Člani matrike za zajem in sprožanje

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

dodatno prikazujejo vrstice **Vloga**(samo za branje; Master v modri barvi / Slave v zeleni barvi),**Sinhronizacijsko linijo**in**Enakovredne enote**.

| Nadzor | Možnosti | Privzeto | Opombe |
| --- | --- | --- | --- |
| **Način sprožilca** | Izklopljeno / Vklopljeno | Vklopljeno | Onemogočeno za člane matrice (matrica upravlja sprožanje). |
| **Vir sprožilca** | Programska oprema / Linija 0 (M8) / Linija 1 / Linija 2 | Linija 0 | Skrito, ko je način sprožilca izklopljen; onemogočeno za člane niza. Linija 0 je optično izoliran zunanji vhod sprožilca M8. |
| **Hitrost snemanja**| Samodejno / Ročno + vrednost | Samodejno |**Samodejno**: omejitev hitrosti snemanja kamere je izklopljena — osvetlitev določa število sličic na sekundo, okence pa prikazuje dejansko trenutno hitrost.**Ročno**: število sličic na sekundo omejite s drsnikom (od 1 do največje vrednosti, omejene s pasovno širino), ki se izračuna na podlagi trenutne dejanske hitrosti. Člani matrike vidijo le za branje namenjeno polje »N sličic na sekundo (v živo)« z opombo »Nastavljeno v nastavitvah matrike«. |

### Omrežje / Prenos

| Vrstica | Delovanje |
| --- | --- |
| **Velikost paketa**| 1500 (standardna) / 9000 (jumbo) — privzeto**jumbo**. |
| **Pretočnost** | Omejitev pretočnosti povezave v MB/s, samo za branje. Backend to ob vsaki povezavi/prekinitvi povezave ponovno uravnoteži med vsemi povezanimi kamerami. |
| **Upravljanje z bufferjem** | Način upravljanja z bufferjem, samo za branje. |

### Snemanje

Okno se konča z gumbom **»Odpri nastavitve snemanja…«**, ki vas preusmeri na [okno z nastavitvami snemanja](capture.md#the-capture-settings-pane) (onemogočen, dokler ni odprt projekt — »Ustvarite ali odprite projekt, da shranite zajeme«). Če je kamera skrita ali zaustavljena, vas namig opozori, da jo pred zajemanjem razkrijete oziroma nadaljujete z delovanjem.

## Okno z nastavitvami niza

Odprete ga z **zobnikom**v vrstici NIZ. Naslov: ime niza s svinčnikom za preimenovanje in**×** za zaprtje. Oddelki spodaj, označeni z *samo v kombiniranem načinu*, se prikažejo le za nize, povezane v kombiniranem načinu prikazovanja.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->

### Sinhronizacija

Vrstice **Master**,**Slaves**in**Sync Line** so samo za branje.

### Senzor okoljske svetlobe

Prikazano za združene in ločene matrike:

* Polje za potrditev **Kalibracijski cilj** — »Zaznaj cilj MAPIR ArUco in preveri NDVI glede na LUT odbojnosti plošče«; upravlja prekrivni cilj in tabelo preverjanja združenega polja.
* Spustni meni **Senzor svetlobe** — poveže en DAQ s celotnim nizom. Izbira se takoj shrani, prenese se v spustni meni »Senzor svetlobe« vsake posamezne kamere v nizu (nastavitve lahko še vedno prepisujete za vsako kamero posebej) in začne posredovati spektre v niz.
* Vrstica **Status** v realnem času: Izklopljeno · »Čakam na prvi spekter…« · »Aktivno — vse kamere v nizu so osvetlitveno popravljene« · »V zadnjih 3 s ni novega spektra — še vedno se uporablja zadnja meritev (ni časovne omejitve za zastarelost)…«.
* Opomba v oknu: »Radiometrična korekcija za celoten niz. Nastavitve za posamezno kamero imajo prednost pred tem.«

### Snemanje — enotne nastavitve senzorja *(samo v kombinaciji)*

Te nastavitve veljajo enotno za vse kamere v nizu (spremembe za posamezno kamero bi prekinile sinhronizacijo). Spremembe se pripravijo in uporabijo skupaj.

<!-- SCREENSHOT-NEEDED: array settings Capture section — Pixel Format, Binning, Resolution preset, the ROI crop W/H/X/Y fields with the "max WxH" hint and Reset button, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons; ideally with the live orange crop-preview box visible on the array tile. -->

| Nadzor | Možnosti / razpon | Kaj počne |
| --- | --- | --- |
| **Format pikslov** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Enotni format senzorja za vse člane. |
| **Združevanje pikselov** | 1x1 / 2x2 / 4x4 | Strojno združevanje pikselov — ohranja celotno vidno polje, hkrati pa poveča razmerje signal-šum (SNR) in hitrost snemanja. Sprememba te nastavitve ponastavi polja ROI na novo celotno vidno polje. |
| **Prednastavljena ločljivost** | Polna / Polovična / Četrtinska | Glede na združevanje pikslov; polja ROI zapolni s centriranim izrezom. |
| **Izrez ROI (px)**| Številčna polja Š / V / X / Y | Izrez senzorja. Širina/višina se poravnata na večkratnike števila 16 (najmanj 64); odmiki se prilagodijo na večkratnike števila 4. Namig »max WxH« prikazuje zgornjo mejo, funkcija**Ponastavi** pa vrne polno vidno polje. Med urejanjem se na ploščici matrike prikaže oranžno okence s predogledom izreza v realnem času (vključno s shemo celotnega senzorja, ko se izrez širi navzven). |
| **Hitrost sprožanja**| Preklop med avtomatskim in ročnim načinom + fps 0,5–10, korak 0,5 |**Avtomatsko**(privzeto): strežnik izračuna hitrost sprožanja na podlagi ločljivosti in pasovne širine — vnos je onemogočen in prikazuje izračunano vrednost.**Ročno**: vrednost se shrani ob kliku na »Uporabi«. |

Opomba v oknu: »Spremembe formata/resolucije za kratek čas ponovno zaženejo vse kamere. Frekvenca sprožitve se uporablja v realnem času.« Gumbov **Uporabi / Prekliči** sta na dnu okna.

### Poravnava (ko-registracija) *(samo v kombinaciji)*

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* Polje **Preostanek**: »RMS x,xx px« — zeleno pod 1 px, rumeno pod 3 px, rdeče v vseh drugih primerih ali če katera koli kamera ni uspela; »brez profila« pred prvim reševanjem.
* Povzetek: »✓ Vse kamere poravnane (N)&quot; / &quot;⚠ p/N kamer poravnanih —  <serial (filter)="">neuspešno&quot; / &quot;Izrez aktiven — Ponovna kalibracija za poravnavo (uporablja celoten senzor)&quot; / »Čakam, da se osvetlitev ustalijo…«.
* Tabela po kamerah: kamera (zadnje 4 številke serijske številke + filter), napaka reprojekcije v px s številom ujemanj („ref“ za glavno kamero) in normalizirana križna korelacijska ocena prekrivanja glede na prag za uspešnost 0,35.
* **Gumb za ponovno kalibracijo poravnave** (pred prvim profilom se prikaže »Kalibriraj poravnavo«) — ponovno izvede so-registracijo na novih okvirjih.
* **Potrditveno polje »Samodejna osvetlitev kamer za poravnavo«** (privzeto potrjeno) — začasno posvetli temne ali ravne kamere (najprej osvetlitev, nato ojačanje), da imajo teksturo za ujemanje, nato pa ponovno vzpostavi avtomatsko osvetlitev.

Kombinirani predogled se ob odprtju samodejno poravna; ponovno kalibrirajte, če se je spremenila ostrina ali globina prizora. Poravnava je **po zasnovi omejena na posamezno sejo** — nikoli se ne shrani v profil, saj je odvisna od trenutne razdalje prizora. Posnetke je še vedno mogoče izvoziti s poravnavo po piksljih (glej [Usklajeni izvozi](capture.md#per-array-controls)).

### Pametna vinjeta

* Polje za potrditev ***Omogoči korekcijo***— uporabi oceno vinjete za vsako kamero posebej v radiometrični verigi (v živo**in** pri izvozih).
* **Kalibriraj iz trenutnega pogleda**– najprej usmeri niz kamer na enotno tarčo (ravno ploščo, steno ali nebo); vsaka kamera se izravna posebej, stanje pa poroča o pridobljeni ravnosti »n/N kamer · −x,x %«.**Počisti** odstrani oceno.
* Natančno nastavite za vsako kamero posebej s pomočjo drsnika **Vignette** za posamezno kamero v [Živem predogledu](#live-preview).

### Živi predogled *(samo združen)** **Indeks**: omogoči potrditveno polje + zobnik — odpre skupni [Kalkulator indeksa](#index-calculator-pane) s pasovi, narisanimi iz**vseh** kamer v skupini. V vrstici za predogled izraza spodaj se prikaže trenutni izraz („Izraz ni nastavljen — odprite kalkulator, da ga ustvarite“), ki se osvežuje vsako sekundo.
* Spustni meni **Ločljivost upodabljanja**(iste prednastavitve kot za posamezno kamero, privzeto 720p): višina toka v živo**in** velikost shranjenega izvoza sestavljenega posnetka. Opomba v oknu: »Predogled + velikost shranjene kompozicije. Slike posameznih kamer se vedno izvozijo v polni ločljivosti.«

### Prikaz plasti *(samo združene)** Potrditveno polje **Omogoči** (privzeto izklopljeno — glavna kamera se prikaže neposredno; vklopljeno = slojna kompozicija).
* Spustni meniji **Prednji plan**/**Zadnji plan**: vsaka posamezna kamera (po imenu) ali**Indeks**. Ko je za Prednji plan izbran Indeks, se v piksljih zunaj minimalne/maksimalne vrednosti LUT prikaže sloj Zadnji plan.

### Razdeljen pogled *(samo v kombinaciji)*

**»Pokaži kamere članic«**— gumb**Razdelj / Skrij kamere članic**, ki dodaja posnetek v živo vsake kamere članice kot ločene ploščice v mreži poleg sestavljenega posnetka. Ploščice berejo obstoječi okvirni pomnilnik niza (brez dodatne povezave s kamero). Samo v pogledu mreže; shranjeno po nizu skupaj s projektom.

### Zmožnosti

Plošča, namenjena le branju, ki se osveži vsakih 5 s:

* **Oznaka stopnje**: »Hkratno zajemanje“ (zelena) · „Hkratno zajemanje (FTD – postopno oddajanje)“ (zelena) · „Postopno zajemanje (100 ms zamik)“ (rumena) · „Konfiguracija je prevelika“ (rdeča).
* **Stanje okvirja**: »x,xx % nepopolno« — zeleno pod 1 %, rumeno pod 5 %, rdeče pri 5 % ali več.
* **Povezovalna črta**: »NIC {mbps} Mbps – trajna {MB/s} MB/s«.

To je trenutna razpoložljiva pasovna širina niza. Za osnovne vrednosti fps in omrežni model – ter kaj spremeniti, ko stopnja preide v rumeno ali rdečo – glejte [Nizi z več kamerami](arrays.md) ter v [CLI Referenca](../reference/cli-reference.md).



<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->## Okno »Index Calculator«

Tretja stran stranskega menija, ki jo delita orodje »Index« za posamezno kamero in orodje »Index« za združeni niz (vsakokrat eno — naslov se glasi »Index Calculator — <camera name="">&quot; ali &quot;Kalkulator indeksa —<array name="">

&quot;). Prejme seznam pasov (naravne pasove filtra kamere ali vse pasove vseh kamer v nizu), trenutni izraz in konfiguracijo LUT (vklopljeno/izklopljeno, raven — privzeto 3, min — privzeto 0,2, max — privzeto 1) ter histogram indeksa v realnem času. S klikom na **Uporabi** se izraz potrdi; spremembe LUT se v realnem času uporabijo v predogledu.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Nastavitve za posamezno kamero v primerjavi z nastavitvami, ki jih upravlja niz

Kratek pregled, kaj se nahaja kje, kadar je kamera članica niza:

| Upravlja niz (v oknu kamere samo za branje) | Še vedno po posamezni kameri znotraj niza |
| --- | --- |
| Format pikslov, ločljivost, združevanje pikselov | Samodejna osvetlitev (osvetlitev, ojačanje, ciljna vrednost, glajenje, ROI) |
| Način/vir sprožitve, hitrost snemanja | Odstranjevanje šuma, ostrina, vinjeta |
| | Usmeritev (zrcaljenje/vrtenje), prekrivni prikazi, točkovni merilnik |
| | Indeks (mreže z ločenim prikazom), vezava svetlobnega senzorja |

Druga medsektorska obnašanja:

* **Kombinirani ali ločeni prikaz** se izbere ob povezavi mreže: kombinirani = ena poravnana sestavljena ploščica (člani prenašajo sliko le prek razdeljenega pogleda); ločeni = vsak član prikaže svojo sinhronizirano ploščico. Kamera nikoli ne prikazuje hkrati samostojnega posnetka in ploščice v nizu.
* **Samodejna ponovna povezava**: ob odprtju shranjenega projekta se obnovijo kamere in nizi ter se vsaka shranjena nastavitev ponovno uporabi v ozadju, preden se tokovi nadaljujejo.
* **Omejevanje zajemanja**: skrite ali zaustavljene kamere so izključene iz funkcije »Capture All«; niz je popolnoma blokiran le, če so VSE kamere skrite/zaustavljene. Glej [Nastavitve in načini zajemanja](capture.md).

## Kako se nastavitve ohranijo

Stanje zavihka »kamere« se shrani **skupaj s projektom**, ne v brskalniku:

* Vsaka odzivna sprememba ustvari posnetek stanja kamer in nizov v projektovem `cameras.json` (z odmevom 500 ms). To vključuje imena in barve kamer, nastavitve osvetlitve/ojačitve/avtomatske osvetlitve (AE), format pikslov/resolucijo/združevanje pikslov, hitrost sprožitve, nastavitve predogleda (resolucija upodabljanja, odstranjevanje šuma, ostrina, vinjeta, barvni profil, nasičenost/kontrast), usmeritev, prekrivni sloji, delitev kanalov, konfiguracija indeksa, nastavitve prediktivne avtomatske osvetlitve (AE), območje avtomatske osvetlitve (ROI), imena matrik, način prikaza, nastavitve zajema matrik (vključno s položajem izreza ROI) ter mrežni blok (povečava posnetka, način prikaza, zaklep mreže, ročno vrstno red ploščic, skrite kamere, zaprte ploščice, aktivna kamera).
* Povezave svetlobnih senzorjev so shranjene v datoteki projekta `sensors.json`.
* Ponovno odprtje projekta ponovno poveže strojno opremo in ponovno uporabi vse te nastavitve.
* **Če ni odprt noben projekt = samo seja**: brez projekta se ob zaprtju Chloros nič ne ohrani.
* Samo za sejo ne glede na projekt: stanje zaustavitve, vzorci točkovnega merilnika svetlobe, potrditveno polje »Kalibracijski cilj« za posamezno kamero (se vedno odpre v izklopljenem stanju) in profil poravnave matrike (po zasnovi se ponovno izračuna za vsako sejo).
* Ena izjema: **Nastavitve zajema** izbire za izvoz in način zajema ostanejo shranjene po projektih v lokalnem pomnilniku aplikacije in ne v `cameras.json` — glej [Nastavitve in načini zajema](capture.md).</array></camera></serial>

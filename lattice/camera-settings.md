# Nastavitve kamer

Zavihek **Kamere** je vmesnik za nadzor v realnem času, ki ga je razvil Chloros

za kamere LATTICE: glavno območje za prikaz v živo, kjer se vsaka priključena kamera prikaže kot ploščica v realnem času, ter stranski pas, ki se premika med tremi stranmi – **seznamom kamer**,**oknom z nastavitvami**(nastavitve za posamezno kamero, niz kamer ali zajem – po ena naenkrat) ter**indeksni kalkulator**. Ta stran opisuje vsak element upravljanja na seznamu kamer, v oknu nastavitev za posamezno kamero in v oknu nastavitev za niz kamer. Načini zajema, izbira vrste izvoza in potek »Capture All« so na spremljevalni strani [Nastavitve in načini zajema](capture.md).

Zavihek »Kamere« se prikaže v stranskem meniju, ko je Chloros

ov backend pripravljen. Vsi spodnji elementi za upravljanje komunicirajo z lokalnim backendom prek `127.0.0.1:5000`; spremembe se takoj upoštevajo v živi sliki kamere, razen če ni drugače navedeno.

## Vrste kamer, uporabljene na tej strani

Upravljalni elementi se prikazujejo ali skrijejo glede na to, katera vrsta kamere je izbrana. Priročnik ves čas uporablja naslednje izraze:

| Izraz | Pomen | Filtrski kanali |
| --- | --- | --- |
| **RGB

-kamera** | LATTICE M3C s filtrom FRGB (model vsebuje `-FRGB`) | Red

/ Green

/ Blue

|
| **Bayerjeva multispektralna** | LATTICE M3C s filtri FRGN, FOCN ali FNGB | FRGN: Red

/ Green

/ NIR

· FOCN: Orange

/ Cyan

/ NIR

· FNGB: NIR

/ Green

/ Blue

|
| **Mono (M3M)** | LATTICE M3M — en ozkopasovni filter, en kalibriran pas | En pas |
| **Član matrike** | Kamera, priključena kot del sinhronizirane matrike (združen ali ločen prikaz) | Glede na svoj filter |

Kamere RGB

so obdelane fotometrično (belina, barvni profili, gama); multispektralne in mono kamere so obdelane radiometrično in preskočijo fotometrične nastavitve. Člani matrike posredujejo nastavitve na ravni toka (format pikslov, ločljivost, združevanje pikslov, sprožilec, hitrost snemanja) nizu — te vrstice postanejo v oknu za posamezno kamero samo za branje in se namesto tega premaknejo v okno z nastavitvami niza.

## Glavno območje prenosa

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->



Če ni priključenih kamer, območje prenosa prikazuje uvodno sporočilo **»Priključite kamero, da začnete«**z dvema gumboma:**Priključi kamero**(zelen, odpre pogovorno okno za povezavo posamezne kamere) in**Poveži niz kamer** (modro, odpre pogovorno okno za povezavo niza kamer). Sama pogovorna okna za povezavo so opisana v [Povezovanje kamer](connecting.md); koncepti niza kamer (sinhronizacija, ravni, pasovna širina) pa v [Nizi več kamer](arrays.md). Ko odprete shranjen projekt, ki vsebuje kamere, se namesto tega prikaže ikona vrtenja z napisom »Ponovno odpiranje N shranjenih kamer…«, medtem ko Chloros

obnavlja tokove iz zadnje seje.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Zgornja vrstica

| Upravljalnik | Kaj počne |
| --- | --- |
| **Preklop načina prikaza**| Preklaplja med**pogledom v mreži**(vse ploščice kot celice) in**pogledom v seznamu**(vrstice v polni širini: vsak združen niz kot ena sestavljena vrstica, vsak član**ločenega** niza kot svoja vrstica z lastnim tokom in tabelo kalibracijskih ciljev, nato pa ENA samostojna kamera – aktivna – spodaj). Opombe: »Preklopi na pogled v mreži« / »Preklopi na pogled v seznamu«. |
| **Zaklep mreže**(ključavnica) | Privzeto**zaklenjeno** – ploščice so fiksirane na mestu. Odkleni, da lahko ploščice s povlekanjem preurediš v katero koli mesto (vrzeli ostanejo ohranjene). Mreža se samodejno ponovno zaklene vsakič, ko se poveže nova kamera. Nasveti: »Odkleni mrežo (omogoči vlečenje ploščic)« / »Zakleni mrežo (zamrzni ploščice na mestu)«. |
| Drsnik **Povečava posnetka** | Velikost ploščice, od 60 pikslov do polne širine okvirja. Celice ohranjajo razmerje stranic 4:3. Če je širina celice manjša od 200 px, se prekrivni napisi z imenom in številom sličic na sekundo skrijejo, da ostane ploščica čista. |

### Ploščice v živo

Vsaka kamera prikaže sestavljeno ploščico v živo; kamera lahko dodatno prikaže tri ploščice v sivih odtenkih, **razdeljene po kanalih** (glej [Razdelitev po kanalih](#display-overlays-drawn-over-the-live-feed)), nizi pa prikazujejo združeno ploščico. Aktivna ploščica ima obroč izbire v barvi kamere (ali niza).

Ob preletu z miško nad ploščico se prikaže gumb za zapiranje **X**:

* Zaprtje **sestavljenega** ploščice, medtem ko ostanejo njene razdelitve po kanalih vidne, le skrije sestavljeno sliko.
* Zaprtje **zadnje vidne ploščice samostojne kamere** prekine povezavo s to kamero.
* **Ploščice z razdelitvami po kanalih, ki so del združenega niza, nikoli ne prekinejo povezave** s kamero — le skrijejo jo.

Ko je mreža odklenjena, povlecite katero koli ploščico na katero koli mesto; razpored se shrani skupaj s projektom.

## Stranski pas – seznam

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

kamer Prva stran stranskega pasu prikazuje seznam vseh povezanih kamer in nizov:

* **Poveži kamero**(zelena) /**Poveži niz** (modra, med skeniranjem prikazuje »Zaznavanje...«). Obe sta onemogočeni, dokler je odprto pogovorno okno za povezovanje.
* **Zajemi vse** (rdeča) — zajame vse kamere na seznamu z izvoznimi tipi, izbranimi v nastavitvah zajemanja. Zahteva odprt projekt. Podrobno dokumentirano v [Nastavitve in načini zajema](capture.md).
* **Zobnik Nastavitve zajema** (poleg »Zajemi vse«) — odpre [okno Nastavitve zajema](capture.md#the-capture-settings-pane). Onemogočeno brez projekta ali med zajemanjem.

### Vrstice kamer

Vsaka vrstica kamere prikazuje barvno označeno obrobo (barva, ki jo je določila kamera), oznako »CAM« — z modro črko **M**(glavna) ali zeleno črko**S** (podrejena) za člane niza — ter prikazno ime. Privzeto ime je `LATTICE-MODEL (serial)`; preimenujte ga v oknu nastavitev za posamezno kamero. Gumbi v vrstici:

| Gumb | Učinek |
| --- | --- |
| **Oko**| Preklop vidnosti. Skrite kamere izginejo iz mreže in so**izključene iz funkcije »Capture All«**. |
| **Zobnik** | Odpre okno z nastavitvami za posamezno kamero (naslednji oddelek). |
| **Pavza / Predvajanje**| Zamrzne predogled v živo**samo na zaslonu** — zajemanje v ozadju se nadaljuje. Kamere v pavzi ne morejo zajemati. |
| **X** | Prekine povezavo. Uporabniški vmesnik se posodobi takoj (v najboljšem primeru); samodejna prekinitev povezave v ozadju pa lahko traja 10–30 s. |

### Vrstice v nizu

Vrstica v nizu prikazuje oznako »ARRAY« v barvi niza, ime niza (ki ga je mogoče preimenovati v nastavitvah niza) in gumb **DAQ · vklop/izklop**—**vklopljen**, kadar je nastavljen svetlobni senzor na ravni niza *ali* kadar ima kateri koli član senzor za posamezno kamero; v njegovem opisu je natančno navedeno, kateri senzor kaj prenaša. Kamere v nizu so navedene z vdolbinami pod njim v svojih vrsticah. Gumbi v vrstici niza: **oko**(skrije/prikaže VSE člane skupaj),**zobnik**(okno nastavitev niza),**X**(odklopi celoten niz).

Stanje svetlobnega senzorja (DLS), ki se uporablja v vrsticah niza in oknu nastavitev niza, ima štiri stanja:**izklopljeno**,**v čakanju**(še ni spektra),**aktiven**(spekter je prispel v zadnjih 3 s) in**zastarel** — v zadnjih 3 s ni novega spektra, vendar se zadnja vrednost *še vedno uporablja* (odčitki DAQ na poti zajemanja nikoli ne potečejo).

Dokler ga ne preuredite, seznam najprej prikazuje matrike, nato samostojne kamere, pri čemer so kamere (in člani vsake matrike) razvrščene po **vrstnem redu kamer**: M3C (Bayer) pred M3M (mono); med M3C, FRGB, FRGN, FOCN, nato FNGB; M3M po valovni dolžini filtra, najprej najnižja. Privzeta razporeditev mreže sledi istemu vrstnemu redu. Samostojne kamere in celotne skupine matrik lahko v stranskem pasu povlečete druga mimo druge, da spremenite vrstni red seznama; elemente matrik ni mogoče povleči neodvisno.

## Okno nastavitev za posamezno kamero

Odprete ga z **zobnikom** v vrstici kamere. Okno se pomakne čez seznam kamer.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->



**Zgornji del**:**barvni vzorec**kamere (kliknite, da odprete vgrajeni izbirnik barv — nastavi barvo robov stranskega pasu in barvo izbirnega obroča ploščic),**ime**s gumbom**Preimenuj**v obliki svinčnika (shranjevanje praznega imena vrne privzeto ime `MODEL (serial)`) in**×** za zaprtje.

### Histogram v realnem času

Na vrhu okna je histogram svetlosti v realnem času, izračunan iz predogleda v formatu JPEG

s frekvenco ~8 Hz. Povprečje je tehtano po Bayerjevi metodi — (R+2G+B)/4 — da se ujema z merjenjem avtomatske osvetlitve (AE) same kamere.

* **OranžnaOrange

a prekinjena črta**= ciljna vrednost AE.**Povlecite jo vodoravno, da spremenite cilj** — ob sprostitvi se pošlje en ukaz, s povlekanjem pa se način ciljne vrednosti AE preklopi na ročni.
* **ZelenaGreen

a neprekinjena črta** = dejanska povprečna svetlost (kar trenutno zagotavlja AE).
* **Gumb RGB** (zgoraj desno): preklaplja med prekrivnimi histogrami po posameznih pasovih, obarvanimi po filtru kamere (npr. pri FRGN: sivo NIR

, zeleno, rdeče). Pri mono (M3M) kamerah je na gumbu napis »MONO« in je onemogočen — mono vedno prikazuje enopasovni histogram svetlosti.
* Oznake na osi X sledijo bitni globini senzorja trenutnega formata pikslov: 0..255, 0..1023, 0..4095 ali 0..65535.

### Vrstice

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

s podatki o kameri

| Vrstica | Delovanje |
| --- | --- |
| **Model** | Samo za branje (npr. `LATT-M3C-L87-FRGN`). |
| **Radiometrična kalibracija** | Green

a **»Aktivna«**oznaka z napisom, ki prikazuje stopnjo kalibracije, hash, datum kalibracije in seznam pasov, naloženih iz kalibracijskega paketa kamere (glej [Tovarniška radiometrična kalibracija](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Skrito za RGB

-kamere** — te imajo fotometrično kalibracijo beline, ne pa sevanja po posameznih pasovih. |
| **Poročilo o kalibraciji**| Gumb**Prenesi** — v pregledovalniku vašega operacijskega sistema odpre PDF-datoteko s certifikatom NIST o kalibraciji kamere po serijski številki. Če certifikat še ni shranjen v predpomnilniku, Chloros

namesto tega prikaže namig. |
| **Serijska številka** | Samo za branje. |
| **Firmware**| Prikaže trenutno različico, nato pa poišče razpoložljivo različico za ta model (shranjeno v predpomnilniku po modelu — niz N kamer enkrat preveri strežnik). Stanj: »Preverjanje…« → gumb**»Posodobi na X«**→ »Posodabljanje…« → »Posodobljeno A → B« / »Neuspešno: …« / »Preskočeno: …« / zeleno**»Posodobljeno«**. Opomba pri gumbu za posodobitev: »Ponastavitev na tovarniške nastavitve + posodobitev + ponovno programiranje UserSet1. ~2–3 minute; ne prekinjajte povezave.« |
| **IP** | Samo za branje. |
| **Temperatura** | Samo za branje, osvežuje se vsake 3 s. Ob ≥65 °C postane oranžna, ob ≥75 °C pa rdeča z opozorilnim simbolom ⚠. |
| **Kalibracijski cilj** | Omogoča zaznavanje cilja odbojnosti ArUco s tabelo za validacijo NDVI

za vsak panel pod živo sliko (pogled seznama) — glej [tabelo kalibracijskih ciljev](#the-calibration-target-table). Na voljo le v seji — vedno se odpre v izklopljenem stanju. |
| Spustni meni **Svetlobni senzor** | Poveže svetlobni senzor DAQ (DAQ-E/M/U, iz seznama na zavihku Svetlobni senzorji) s to kamero za korekcijo osvetlitve z navzdol usmerjeno svetlobo (DLS) in predvidljivo samodejno osvetlitev. »»Nič« izbriše povezavo. Če ni priključenih senzorjev, se v spustnem meniju prikaže »(ni priključenih senzorjev — odpri zavihek DAQ)«. Povezava se shrani skupaj s projektom. |

#### Tabela kalibracijskih tarč

Tabela je **namenjena izključno za preverjanje** („samo za preverjanje — ne spreminja kalibracije“). V naslovu so navedene serijska številka kamere, filter, številka tarče in velikost označevalca v piksljih, vsaka vrstica pa ustreza enemu panelu:

* **Panel**, nato pa en stolpec na pas z vrednostjo panela — Bayerjeva kamera prikazuje svoje tri pasove po imenu in valovni dolžini (npr. FRGN: Red

660 nm / Green

550 nm / NIR

850 nm); **mono (M3M) kamera prikazuje en sam pas** (npr. `F750`) in nima stolpcev NDVI

.
* **Sat** — delež pikslov panela, ki so nasičeni (jantarni od 1 %, rdeči od 5 %).
* **NDVI

Meas**,**NDVI

Theo** (referenčni NDVI

odbojnosti panela) in **Δ**— samo za multispektralne kamere Bayer. Majhna oznaka za »NDVI

Meas« navaja, na podlagi česa je bil izračunan izmerjeni NDVI

:**ρ**(odbojnost — na voljo je odčitek svetlobnega senzorja),**L**(sevanje — ni odčitka svetlobnega senzorja) ali**DN**(surove vrednosti kamere). Le**ρ** NDVI

je primerljiv z NDVI

Theo: za L in DN je stolpec Δ sivo obarvan, kar potrjuje tudi opomba (»Ni odčitka svetlobnega senzorja: NDVI

izhaja iz sevanja, zato Δ ni preverjanje kalibracije« / »NDVI

izhaja iz surovih vrednosti kamere, zato Δ ni preverjanje kalibracije«). Pri ρ je Δ zelen, če je vrednost manjša od 0,02, rumen, če je manjša od 0,05, v vseh drugih primerih pa rdeč.

<!-- SCREENSHOT-NEEDED: a 1.3.0 Calibration Target table under a live feed in list view — a Bayer camera's table showing the band columns, Sat, and "NDVI Meas ρ" / NDVI Theo / Δ; ideally next to a mono camera's single-band table with no NDVI columns. -->

### Osvetlitev in ojačanje

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Vsi številčni vnosi tukaj uporabljajo drsnike, ki se pospešijo ob podaljšanem dotiku: dotik = ±1, podaljšan dotik &gt;1,5 s = ±10, podaljšan dotik &gt;3 s = ±100. Vrednost se pošlje kameri, ko sprostite prst.

| Nadzor | Obseg / možnosti | Privzeto | Velja za | Kaj počne |
| --- | --- | --- | --- | --- |
| **Izpostavljenost (us)**| Najmanjša/največja vrednost kamere v realnem času | Avto | Vse | Čas izpostavljenosti v mikrosekundah, s stikalom**Avto/Ročno**stikalo. Avto = neprekinjena avtomatska osvetlitev s strani kamere. Pri nastavitvi ročne osvetlitve ostane**Ojačanje** na Avto, če je bilo tako nastavljeno, kar omogoča uporabo celotnega obsega ojačanja kamere, dokler ne uredite ojačanja ali ga ne nastavite na Ročno. |
| **Ojačanje (dB)**| Minimalna/maksimalna vrednost kamere v realnem času (npr. do 48 dB) | Ročno (izklopljeno) | Vse | Analogna/digitalna ojačitev z lastnim stikalom**Avto/Ročno**. |
| **Ciljna svetlost AE**| 0–255 | 80, način**Avto**| Vse (ureljivo, ko je vklopljena AE ali avtomatska ojačitev) | Svetlost, na katero se osredotoča AE. V**Avto**(privzeto) krmilnik na podlagi histograma sam izbere ciljno vrednost, pri čemer ohranja osvetlitev na 60–75 % maksimalne vrednosti senzorja. Z vnosom vrednosti ali povlečenjem oranžne črte na histogramu se preklopi na**Ročno**. |
| **Izglajevanje AE** | 0,5–40, korak 0,1 | 8,0 | Vse | Dušenje AE. Opomba: »Nižja vrednost = AE se odziva hitreje (lahko utripa pri visokih fps). Višja vrednost = bolj gladko / počasneje.« Vrednosti, ki so precej nižje od privzete, lahko povzročijo utripanje AE in destabilizirajo pretakanje pri visokih hitrostih snemanja; 8,0 je stabilna privzeta vrednost. |
| **Območje interesa AE**| Polje za vklop + gumb**Usmeri**| Izklopljeno | Vse | Ko je vklopljeno, AE meri le območje, označeno z zelenimi črtkami, namesto celotnega okvirja.**Aim** omogoča namestitev s klikom na živo sliko: s klikom se območje osredotoči na 30 % okvirja; s klikom in vlečenjem se nariše poljuben pravokotnik (najmanj 5 % × 5 %). Funkcija »Aim« se sama izklopi po enem postavitvi. Območje se preslika nazaj v koordinate kamere glede na katerokoli rotacijo/zrcaljenje, ki ste ga nastavili, in se shrani skupaj s projektom. |
| **Hitrost prilagajanja AE** | 0,1–5, korak 0,1 | 1,0 | Samo za člane skupine | Kako hitro cilj avtomatske AE sledi spremembam svetlosti prizora; pri vrednosti 1,0× se ponovno preveri vsake 2,5 s. |
| **Zaščita svetlih delov** | Stroga (1 %) / Normalna (5 %) / Ohlapna (15 %) | Stroga | Kamere, ki omogočajo to nastavitev | Koliko okvirja se lahko preklopi v belo, preden AE potemni sliko. |

{% hint style="info" %}
**Zahteve glede osvetljenosti za multispektralne kamere Bayer (RGN

/ OCN

/ NGB

):** prizor mora imeti dovolj svetlobe v vseh treh kanalih, sicer kalibracija ne deluje pravilno — ena sama osvetlitev senzorja pokriva vsa tri spektra. Za merjenje svetlobe uporabite svetlobni senzor DAQ ali preklopite na popolnoma monokromatski način (M3M), da vsak pas dobi svojo osvetlitev. Če posnetek to zahtevo krši, ga Chloros

zazna in vas opozori (obvestilo o omejitvi razmešavanja). V živo ploščica z razdeljenimi kanali prikaže **»Podatki pasu z nizko zanesljivostjo · N %«**, kadar je unmix omejil ta pas na 25 % ali več njegovih pikslov.
{% endhint %}

{% hint style="info" %}
**Kako avtomatska osvetlitev porablja čas osvetlitve in ojačanje (1.3.0).** Ko je ojačanje nastavljeno na »Auto«, Chloros

ohranja ojačanje na 0 dB, dokler osvetlitev ne doseže najdaljše dovoljene vrednosti (zgornja meja AE, sprožilno obdobje matrike ali zgornja meja osvetlitve DAQ); šele takrat se ojačanje poveča. V matriki ima prednost sprožilna frekvenca: avtomatska osvetlitev zapolni obdobje, ki ga določa ta frekvenca, nato pa poveča ojačanje, tako da temnejša scena ne znižuje več hitrosti sprožanja v avtomatskem načinu (v različici 1.2.0 se je to dogajalo). **Ročna**osvetlitev na elementu matrike, ki traja dlje od obdobja sprožanja, upočasni sprožanje matrike, da se temu prilagodi; avtomatska frekvenca se ponovno vzpostavi, ko se vsi elementi, ki so jo upočasnili, vrnejo na avtomatsko osvetlitev. Če sta**osvetlitev**in**ciljna svetlost AE** nastavljeni na »Avto« in senzor svetlobe DAQ zagotavlja odčitek, se osvetlitev vsake kamere ohrani, ko se avtomatska osvetlitev ustalijo, dokler izmerjena svetloba ostane v okviru približno ±25 %; večja, trajna sprememba svetlobe, ponovni zagon nastavitev (na primer sprememba ROI) ali vaša lastna sprememba osvetlitve jo ponovno sproži.
{% endhint %}

{% hint style="info" %}
**Kako deluje avtomatska osvetlitev (1.3.0).**

* Pri ročni osvetlitvi ostane ojačanje nastavljeno na »Avto«, dokler ga ne spremenite.
* Majhni svetli deli ne potemnijo prizora: odsev do 1 % okvirja se prezre, v panelih znanega kalibracijskega cilja pa do 0,5 % panela.
* Zgornja meja osvetlitve DAQ velja le, dokler je kalibracijski cilj v vidnem polju; če ni vidnega panela, kamera osvetli prizor.
* Osvetlitev ostane stabilna pri konstantni svetlobi in se po večji spremembi ustalijo v nekaj sekundah. Preklip ne omeji več osvetlitve za vedno, sprememba območja zanimanja (ROI) pa ponovno najde ustrezno osvetlitev.
* Ciljna svetlost ne preseže dosega kameredosega, zato vrnitev svetlobe ne povzroči bleščanja v kadru.
* Kamera, katere samodejna osvetlitev preneha odgovarjati v močnem soncu, se ohrani na ravni, ki ustreza svetlobi, v kateri se nahaja, ki jo odčita svetlobni senzor ali, če ta ni priključen, lastni posnetki kamere, in se sprosti, ko se svetloba spremeni. Temni ali zakriti svetlobni senzor tega ne moti.
{% endhint %}

### Format pik in ločljivost

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

**Člani niza** prikazujejo vrstici »Trenutno« (format + ŠxV) in »Združevanje« (samo za branje) z opombo »Nastavljeno v nastavitvah niza« – ponovni zagon toka na enem članu bi prekinil sinhronizacijo, zato se ti upravljajo v [oknu nastavitev niza](#array-settings-pane).**Samostojne kamere** imajo:

| Nadzor | Možnosti | Kaj počne |
| --- | --- | --- |
| **Format pikslov** | 8-bit / 10-bit / 12-bit / 16-bit | Bitna globina senzorja; družina formatov je odvisna od senzorja. Privzeta vrednost je 12-bit, projekt pa shrani nastavljeno vrednost, vključno z 8-bit, in se ponovno poveže z njo. |
| **Ločljivost** | Polna / Polovična / Četrtinska | Glede na trenutno združevanje: Polna = 2048/N × 1536/N za združevanje N×N. |
| **Združevanje pik** | 1x1 (ni) / 2x2 / 4x4 | Strojno združevanje pik N×N — večje vrednosti zmanjšajo ločljivost, vendar povečajo razmerje signal-šum (SNR) in hitrost snemanja. Sprememba te nastavitve ponovno zažene tok podatkov in ponastavi vse območja zanimanja (ROI) na novo polno vidno polje. |
| **Trenutno** | samo za branje | Dejivni WxH in odmik (x, y), ki sta trenutno v veljavi. |

### Predogled v živo

Vse v tem razdelku je **samo na strani prikaza**— spreminja tisto, kar vidite v živi sliki, medtem ko shranjeni posnetki ostanejo linearni in nespremenjeni — z eno izjemo:**Vignette** je radiometrična in vpliva tudi na izvoze (kot je navedeno spodaj).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->



| Nadzor | Obseg / možnosti | Privzeto | Velja za | Kaj počne |
| --- | --- | --- | --- | --- |
| **Ločljivost upodabljanja** | 360p (najhitrejša) / 480p / 720p / 1080p / Izvorna ločljivost senzorja (najpočasnejša) | 720p | Vse | Višina, na kateri backend izvaja radiometrično verigo predogleda. Nižja vrednost zagotavlja višjo hitrost snemanja brez spremembe vidnega polja. |
| **Indeks**| Potrditveno polje »Omogoči« + zobnik | Izklopljeno | Samo za Bayerjev multispektralni senzor,**ne** za senzorje s kombiniranim nizom | Predogled indeksa žive vegetacije. Z ikono zobnika se odpre skupno okno [Index Calculator](#index-calculator-pane), v katerem so že vnaprej naloženi naravni spektralni pasovi kamere (npr. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Po meri določen izraz skupaj z LUT (vklop/izklop, privzeta raven 7 stopenj, privzeta najmanjša vrednost 0,2, privzeta največja vrednost 1) se izračuna za vsak predogledni okvir. Člani kombinirane matrice skrijejo to vrstico – matrica ima en skupni indeks. |
| **Bela bilanca** | Izklopljeno / Enkrat / Neprekinjeno + gumb za ponovno zajemanje | Neprekinjeno | Samo RGB

| Belina v realnem času. Gumb za osvežitev ponovno zajame belo ravnovesje iz trenutnega DLS-spektra (onemogočeno, če je način izklopljen). |
| **Gama** | Vklopljeno / Izklopljeno | Vklopljeno | Samo RGB

| Prikaz gama (γ = 2,2 LUT) v predogledu v živo. Shranjene posnetke ostanejo linearni. |
| **Odstranjevanje šuma** | Potrditveno polje + jakost 0–100 | Izklopljeno / 50 | Vse (po kameri, tudi znotraj nizov) | Dvostranski filter v živi predogledu. Višja vrednost = gladkejši, a mehkejši detajli. |
| **Ostrina** | Potrditveno polje + jakost 0–100 | Izklopljeno / 30 | Vse | Maska neostrosti v živi predogledu, uporabljena kot zadnja. Lahko okrepi šum. Samo v predogledu. |
| **Vignette**| Potrditveno polje + jakost 0–100 | Izklopljeno / 0 | Vse | Ročno odstranjevanje preostale vinjetacije (posvetli kotičke), ki se naloži na oceno pametne vinjetacije niza.**Radiometrično — vpliva na živi prikaz IN izvoze**, za razliko od zmanjševanja šuma/ostrine. |
| **Barvni profil** | Raw / Linear / Natural / Enhanced / Custom Temperature | Natural | Samo RGB

| Glej spodaj. |
| **Barvna temperatura** | 2000–10000 K, korak 100 | 5500 K | Samo RGB

, profil po meri | Belino uravnava na fiksno korelirano barvno temperaturo (vnos DLS se ne upošteva). Zadnja izbrana vrednost v kelvinih se shrani tudi ob preklopu med profili. |
| **Svetli deli** | od −100 do 0, korak 1 | 0 (izklopljeno) | samo RGB

| Novost v različici 1.3.0. Pritemni najsvetlejša, najbelja območja slike (0 = nespremenjeno, −100 = najmočnejše). Velja za predogled v živo **in za posnetke na ravni zaslona (Predogled)**, ki shranijo nastavitev (`Chloros:DisplayHighlights`); izvoz sevanja in odbojnosti ni nikoli prizadet. Ne more obnoviti podatkov senzorja, ki so bili odrezani — za to zmanjšajte osvetlitev. Zebra prekrivni vzorec sledi potemnjeni sliki. Onemogočeno z barvnim profilom Raw (»Ne velja pri barvnem profilu Raw.«). |
| **Nasičenost** | 0–200 (100 = nevtralno) | 100 | Samo RGB

| Nasičenost HSV v živi predogledu. |
| **Kontrast** | 0–200 (100 = nevtralno) | 100 | Samo RGB

| Linearen kontrast okoli srednje sive barve v predogledu v živo. |
| **Zrcaljenje po vodoravni / Zrcaljenje po navpični** | Potrditvena polja | Izklopljeno | Vse | Obračanje predogleda vodoravno / navpično. |
| **Rotacija**| 0° / 90° / 180° / 270° | 0° | Vse | Rotacija predogleda. Usmeritev se uporabi na koncu verige predogleda v ozadju.**Shranjeni posnetki (1.3.0):**izdelki samostojne kamere, kot so Predogled, Sijaj, Odbojnost in Indeks, se obrnejo z Zrcaljenjem/Vrtljajem, medtem ko**Raw in Debayered ohranita izvorno usmeritev kamere**; vsaka datoteka posnetka shrani nastavitev v svojem XMP, zato se pri kasnejši obdelavi surovega posnetka ta uporabi na enak način.**Datoteke v nizu ostanejo v izvorni usmeritvi** (nastavitev je shranjena, vendar se ne uporabi), sestavljeni pogledi niza pa jo ignorirajo. |**Semantika barvnih profilov** (kamere RGB

):

* **Raw** — v celoti preskoči verigo obdelave.
* **Linear** — temni signal + izravnava polja + bela ravnovesja; brez barvne matrike, brez gama.
* **Naravno** *(privzeto)* — linearno plus izmerjena matrika barvne korekcije in tonovna krivulja, prilagojena sceni.
* **Izboljšano**— Naravno plus živost in lokalni kontrast CLAHE. Dodatni strošek velja**samo za predogled v živo** — shranjene posnetke vedno obdela v celoti, ne glede na profil.
* ****Prilagojena temperatura** — Naravno z belo ravnovesjem, fiksiranim na izbrano vrednost v kelvinih.

{% hint style="warning" %}
Za nastavitve »Naravno«, »Izboljšano« in »Prilagojena temperatura« se v oknu prikaže opomba o tonih: posnetki so posvetljeni glede na posamezno sceno, zato shranjene *zaslonske* slike niso primerljive med posameznimi posnetki. **Za meritve izvozite sevanje ali odbojnost.**
{% endhint %}

### Prekrivni elementi na zaslonu (risani prek žive slike)

Ti elementi so na voljo le v uporabniškem vmesniku — so narisani prek videa, nikoli pa ne vplivajo na tok ali posnetke.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Prekrivni element | Nadzorni elementi | Privzeto | Kaj počne |
| --- | --- | --- | --- |
| **Zebra** | Potrditveno polje + prag 200–255 | Izklopljeno / 250 | Magenta diagonalne črte na odrezanih pikslah. |
| **Križec** | Potrditveno polje | Izklopljeno | Oznaka središča okvirja. |
| **Mreža** | Izklopljeno / 3 × 3 / 9 × 9 | Izklopljeno | Mreža za kompozicijo. |
| **Histogram** | Potrditveno polje + širina 0,10–0,90 okvirja | Izklopljeno / 0,25 | Pas histograma v živo. |
| **Focus Peak** | Potrditveno polje + prag 20–200 + barvni vzorec | Izklopljeno / 80 / `#ff5722` | Sobelovo poudarjanje robov za ostrenje. |
| **Channel Splits** | »Pokaži razdelitve (Red

/ Green

/ NIR

)« / gumb »Skrij razdelitve« | Skrito | Dodaja tri neodvisne ploščice v sivih odtenkih za vsak kanal poleg sestavljenega slike (oznaka gumba sledi filtrskim kanalom kamere). Vsaka razdeljena ploščica se lahko premika in ima barvo robov kamere. Ni na voljo pri monokromatskih kamerah. Shrani se skupaj s projektom. |

### Točkovni merilnik

* Polje za potrditev **Klikni za vzorčenje**: kliknite na živo sliko, da vzorčite en sam piksel (označi ga križec), ali kliknite in povlecite območje za izračun povprečja pikslov. Gumb**Izbriši**izbriše vzorec in križec. Medsebojno izključljivo z načinom**Aim** v AE-ROI.
* **Spustni meni**Prikaži**:**Raw (bitna globina)**— izvorne digitalne vrednosti z bitno globino senzorja (npr. 12-bitni → 0..4095) — ali**Prikaz (8-bitni)**(privzeto). Ko je aktiven indeks v realnem času, se v polju**Display** namesto tega prikaže izračunana vrednost indeksa (npr. NDVI

).
* Na odčitnem panelu so navedene koordinate pikslov, velikost okvirja, format pikslov, bitna globina in tabela kanalov (Chan / Value / %) z oznakami pasov in valovnimi dolžinami; pari zelene barve po Bayerjevem sistemu so povprečeni; vzorci območij prikazujejo »N px avg«.

Stanje točkovnega merilnika velja le za tekočo sejo.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Prediktivna avtomatska osvetlitev (na podlagi DLS)

Ta razdelek se prikaže le, če je **priključen vsaj en svetlobni senzor DAQ** — reševalec za delovanje potrebuje spektrum v živo, ki pada navzdol.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->



| Nadzor | Obseg | Privzeto | Kaj počne |
| --- | --- | --- | --- |
| **Omogoči** | Potrditveno polje | Vklopljeno (samostojne kamere) | Reševalnik zaprte oblike uporabi spekter DLS skupaj s skalarnimi vrednostmi kalibracijskega paketa kamere, da najsvetlejši pas prinese blizu nasičenosti, hkrati pa najtemnejši pas ohrani nad spodnjo mejo SNR — en zapis osvetlitve na rešitev, brez stabilizacijskega cikla. Zasnovan za časovni posnetek s sončno energijo, kjer mora biti vsak posnetek pravilno osvetljen. Zadnji del sistema se tiho preklopi na reaktivno avtomatsko osvetlitev (AE), kadar je odčitek DLS zastarel/manjka ali kalibracijski paket ni naložen. |
| **Izglajevanje (α)** | 0,05–1,0, korak 0,05 | 0,3 | Izglajevanje zaporednih prediktivnih rešitev (nižja vrednost = večje izglajevanje). |
| **Odbojnost prizora**| Gumb**Ponovna kalibracija ρ** | — | Ponovno oceni faktor odbojnosti prizora, ki ga uporablja reševalec. |

{% hint style="info" %}
**Povezava matrike privzeto izklopi prediktivno AE** — za matrike izpostavljenost uravnavata pametna AE sistema Chloros

ter avtomatska izpostavljenost na strani kamere (z zaščito pred zasičenostjo), pri čemerocena odbojnosti ni zanesljiva v mešanih prizorih. Tukaj jo lahko ponovno vklopite za posamezno kamero, če želite izrecno radiometrično osvetlitev, ki jo poganja DLS.
{% endhint %}

**Zgornja meja osvetlitve, ki jo poganja DAQ, in AE, vezana na padajoče sevanje.** Ne glede na zgornje potrditveno polje, ko je svetlobni senzor DAQ dodeljen RGB

-kameri, Chloros

izmerjeno absolutno navzdolno sevanje uporabi za izračun največje osvetlitve × ojačitve, pri kateri površina s 100-odstotno odbojnostjo ostane pod pragom izrezanja, in to uporabi kot **zgornjo mejo**pri avtomatski osvetlitvi. Od različice 1.3.0 se zgornja meja uporablja le, dokler je kalibracijski cilj v vidnem polju te kamere (viden v njenem kadru v zadnjih 10 sekundah); če cilj ni v vidnem polju, namesto tega deluje običajna avtomatska osvetlitev kamere z lastno zaščito pred preosvetlitvijo, tako da preostali del prizora ni temnejši, kot je potrebno. Medtem ko je zgornja meja aktivna, kamera deluje**v načinu pritrditve na padajočo svetlobo**: deluje v odprtem krogu pri osvetlitvi, merjeni na padajočo svetlobo, z ojačitvijo 0 dB — osvetlitev sledi izmerjeni svetlobi, ne vsebini prizora. Ker zgornja meja lahko le skrajša osvetlitev, sama po sebi ne more povzročiti izrezovanja. Od različice 1.3.0 se zgornja meja izračuna z lastno ekspozicijsko konstanto kamere iz njenega kalibracijskega paketa (vrednost povprečja flote se uporabi le, če paket te vrednosti ne vsebuje). Zgornja meja se samodejno izklopi – in se ponovno vzpostavi običajna avtomatska ekspozicija scene –, kadar v vidnem polju ni kalibracijskega cilja, manjka odčitek DAQ, je ta zastarel (&gt;30 s), temna ali če se pri fiksirani osvetlitvi izreže ≥15 % slike (kar pomeni, da senzor in kamera zaznavata različno osvetlitev). V grafičnem vmesniku ni stikala za to; to je standardno obnašanje, kadar je RGB

-kamera povezana z DAQ.

### Pridobivanje in sprožitev

Člani<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

matrike dodatno prikazujejo vrstice **Vloga**(samo za branje; Master v modri / Slave v zeleni),**Sinhronizacijska linija**in**Enakovredni**.

| Nadzor | Možnosti | Privzeto | Opombe |
| --- | --- | --- | --- |
| **Način sprožitve** | Izklopljeno / Vklopljeno | Vklopljeno | Onemogočeno za člane matrike (matrika upravlja sprožitev). |
| **Vir sprožilca** | Programska oprema / Linija 0 (M8) / Linija 1 / Linija 2 | Linija 0 | Skrito, ko je Način sprožilca izklopljen; onemogočeno za člane niza. Linija 0 je optično izoliran zunanji vhod za sprožilec M8. |
| **Hitrost snemanja**| Samodejno / Ročno + vrednost | Samodejno |**Samodejno**: omejitev hitrosti snemanja kamere je izklopljena — hitrost snemanja določa osvetlitev, polje pa prikazuje dejansko hitrost v realnem času.**Ročno**: število sličic na sekundo omejite s drsnikom (od 1 do največje vrednosti, omejene s pasovno širino), izhodišče pa je trenutna dejanska frekvenca. Člani matrike vidijo le za branje namenjeno vrednost »N sličic na sekundo (v živo)« z opombo »Nastavljeno v nastavitvah matrike«. |

### Omrežje / Prenos

| Vrstica | Delovanje |
| --- | --- |
| **Velikost paketa**| 1500 (standardna) / 9000 (Jumbo) — privzeto**Jumbo**. |
| **Prepustnost** | Omejitev prepustnosti povezave, ki je samo za branje, v MB/s. Backend to ob vsaki povezavi/prekinitvi povezave ponovno uravnoteži med vsemi povezanimi kamerami. |
| **Upravljanje z bufferjem** | Način upravljanja z bufferjem, ki je samo za branje. |

### Snemanje

Okno se konča z gumbom **»Odpri nastavitve snemanja…«**, ki vas preusmeri na [okno Nastavitve snemanja](capture.md#the-capture-settings-pane) (onemogočeno, dokler ni odprt projekt — »Ustvarite ali odprite projekt za shranjevanje zajemov«). Če je kamera skrita ali zaustavljena, vas namig opozori, da jo pred zajemanjem razkrijete oziroma nadaljujete z delovanjem.

## Okno z nastavitvami niza

Odprete ga z **zobnikom**v vrstici MATRIČE. Naslov: ime matrice s svinčnikom za preimenovanje in**×** za zaprtje. Oddelki spodaj, označeni z *samo kombinirano*, se prikažejo le za matrice, povezane v kombiniranem načinu prikazovanja.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->



### Sinhronizacija

Vrstice **Master**,**Slaves**in**Sync Line** so samo za branje.

### Senzor okoljske svetlobe

Prikazano za združene in ločene nize:

* Spustni meni **Senzor svetlobe**— poveže en DAQ z celotnim nizom. Izbira se takoj shrani, se prenese v spustni meni**Senzor svetlobe** vsake posamezne kamere v nizu (nastavitev lahko še vedno prepisujete za vsako kamero posebej) in začne posredovati spektre v niz.
* Vrstica **Stanje** v realnem času: Izklopljeno · »Čakam na prvi spekter…« · »Aktivno — vse kamere v nizu so osvetlitveno korigirane« · »V zadnjih 3 s ni novega spektra — še vedno se uporablja zadnja vrednost (ni časovne omejitve za zastarele podatke)…«.
* Opomba v oknu: »Radiometrična korekcija za celoten niz. Nastavitve posamezne kamere to prepisujejo.«

### Kalibracijski cilj

Lastna skupina neposredno za senzorjem okoljske svetlobe, tako za združene kot za ločene nize:

* Potrditveno polje **Omogoči** — »Zaznaj cilj MAPIR

ArUco in preveri LUT za NDVI

glede na odbojnost plošče«; upravlja prekrivni cilj in tabele preverjanja za združeno ploščico. Pri **ločenem** nizu vklopi lastno [tabelo kalibracijskega cilja](#the-calibration-target-table) (eno na vrstico kamere v pogledu seznama).

### Zajemanje — enotne nastavitve senzorjev

Prikazano za združene in ločene nize (naslov skupine se glasi »Zajemanje (enotno za vse kamere)«). Te nastavitve veljajo enotno za vsakega člana (spremembe po posameznih članih bi prekinile sinhronizacijo). Urejanja se pripravijo in uporabijo skupaj s **Uporabi**— razen preklopnika**ROI vklopljen / ROI izklopljen**, ki se uporabi takoj.

<!-- SCREENSHOT-NEEDED: array settings Capture section (1.3.0) — Pixel Format, Binning, Resolution preset, the ROI crop row with the "ROI on" toggle, the "Select" button, the "max WxH" hint and Reset button, the W/H/X/Y fields, the full-sensor ROI map below them with the orange crop box and its "W×H @ X,Y" label above the box, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons. -->

| Nadzor | Možnosti / obseg | Kaj počne |
| --- | --- | --- |
| **Pikselni format** | 8-bit / 10-bit / 12-bit / 16-bit | Enotna bitna globina za vse člane; vsaka kamera za svoj senzor uporabi format te globine. |
| **Binning** | 1x1 / 2x2 / 4x4 | Strojno združevanje pik – ohranja celotno vidno polje, hkrati pa poveča razmerje signal-šum (SNR) in hitrost snemanja. S spremembo se polja ROI ponastavijo na novo polno vidno polje. |
| **Prednastavitev ločljivosti** | Polna / Polovična / Četrtinska | Glede na združevanje pik; polja ROI se zapolnijo s centriranim izrezom. |
| **Izrez ROI (px)**| Preklopnik**ROI vklopljeno / ROI izklopljeno**, gumb**Izberi**, polja Š / V / X / Y, zemljevid celotnega senzorja | Izrez senzorja — glej [Urejevalnik ROI](#the-roi-editor). Širina/višina se poravnata na večkratnike števila 16 (najmanj 64); odmiki se poravnajo na večkratnike števila 4. Namig »max WxH« prikazuje zgornjo mejo, gumb**Ponastavi** pa vrne polja na polno vidno polje. Med urejanjem oranžno okvirje za izrez spremlja vsako spremembo polja na zemljevidu in na ploščici matrike. |
| **Hitrost sprožitve**| Preklopnik Avto / Ročno + fps 0,5–10, korak 0,5 |**Avto**(privzeto): strežnik izpelje hitrost sprožanja iz ločljivosti in pasovne širine — vnos je onemogočen in prikazuje izpeljano vrednost.**Ročno**: vrednost se fiksira ob kliku na »Uporabi«. Fiksirana hitrost, ki je za novo velikost okvirja prevelika, se zadrži na najhitrejši hitrosti, ki jo ta velikost dopušča, in se ponovno vzpostavi, ko velikost okvirja to spet omogoča. |

Opomba v oknu: »Spremembe formata/resolucije za kratek čas ponovno zaženejo vse kamere. Hitrost sprožanja se uporablja v realnem času.« Gumboma **Uporabi / Prekliči** sta na dnu okna.

#### Urejevalnik ROI

Novost v različici 1.3.0. Vrstica za izrez ROI in zemljevid pod njo delujeta skupaj:

* **ROI vklopljen / ROI izklopljen**—**takoj**vklopi ali izklopi izrez senzorja (kamere se za kratek čas ponovno zaženejo; ploščica matrike prikazuje »Uporabljanje nastavitev…«).**Izklop**odčita celoten senzor in**si zapomni vaš izrez**— polja ga še naprej prikazujejo;**Vklop** ga ponovno uporabi. Vklop ROI brez nastavljenega izreza uporabi centriran izrez polovične velikosti („Izrez ni bil nastavljen, zato je bil uporabljen centriran izrez polovične velikosti. Prilagodite ga s polji.“).
* **Izberi**— nariše izrez na zemljevidu: povleči po njem ali klikni dva nasprotna kota (Esc prekliče). Pravokotnik se prikaže v poljih Š/V/X/Y, prilagojene zgornjim korakom; nič se ne uporabi, dokler ne pritisnete**Uporabi**, zato ga lahko najprej natančno prilagodite s puščicami v poljih. En pravokotnik na vsak pritisk Izberi.
* **Zemljevid**prikazuje izrez na celotnem senzorju, zato ga lahko spet razširite: »Celoten senzor, v živo«, medtem ko kamere berejo celoten senzor, ali »Posnetek celotnega okvirja iz {čas}. Kamere berejo le izrez«, medtem ko so izrezane. Če je niz doslej prenašal le izrezane slike, se na zemljevidu prikaže »Še ni pogleda celotnega okvirja: ta niz je prenašal le izrezane slike.« s gumbom**Pokaži celoten okvir**, ki izklopi ROI. Oznaka izreza `W×H @ X,Y` se nahaja zunaj okvirja (nad njim ali pod njim, če se okvir dotika zgornjega roba), na zemljevidu in na ploščici v živo.
* **Uporabi**potrdi polja: izrez vklopi ROI; polni okvir (po**Ponastavi**) ga izklopi, pri čemer se prejšnji izrez še vedno pomni; ROI, ki je izklopljen in katerega polja še vedno prikazujejo pomnjen izrez, ostane izklopljen, ko uporabiš drugo spremembo (format, združevanje, hitrost).
* **Na poravnani matrici se okno izreza vsakega člana postavi glede na poravnavo**, tako da vsaka kamera zajame isti del prizora. Matrica, ki je bila izrezana, preden je bila sploh poravnana, se najprej poravna na celotnem senzorju (izrez se odstrani, poravnava se izvede, izrez pa se ponovno vzpostavi z okni za vsako kamero posebej).
* Po spremembi izreza se samodejna osvetlitev ponovno prilagodi novemu kadriranju. Med izrezom lahko osvetlitev posamezne kamere še vedno traja do konca sprožilnega obdobja.
* Stanje vklopa/izklopa ROI in shranjen izrez se shranita skupaj s projektom.

### Poravnava (so-registracija)

Prikazano za **tako združene kot ločene matrike** (v različici 1.2.0 je bilo prikazano le za združene matrike); naslov skupine se glasi »Poravnava«.

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* **Preostanek**: »RMS x,xx px« — zelena barva pri vrednosti pod 1 px, rumena pri vrednosti pod 3 px, rdeča v vseh drugih primerih ali če katera koli kamera ni uspela; »brez profila« pred prvim reševanjem.
* Povzetek: »✓ Vse kamere poravnane (N)« / »⚠ p/N kamer poravnanih —  <serial (filter)="">napaka« / »Izrez aktiven — Ponovna kalibracija za poravnavo (uporablja celoten senzor)« / »Čakam, da se osvetlitev ustalijo…« / »Uporabljam nastavitve…« (kamere se ponovno zaganjajo) / »Niz je prenehal pošiljati slike — ponovno povežite niz«.
* Tabela po posameznih kamerah: kamera (zadnje 4 številke serijske številke + filter), napaka reprojekcije v px s številom ujemanj („ref“ za glavno kamero) ter normalizirana ocena navzkrižne korelacije prekrivanja glede na prag uspešnosti 0,35.
* **Ponovna kalibracija poravnave** gumb (pred prvim profilom se prikaže »Kalibriraj poravnavo«) — ponovno izvede so-registracijo na novih slikah.
* **Potrditveno polje »Samodejna osvetlitev kamer za poravnavo«** (privzeto označeno) — začasno posvetli temne ali ravne kamere (najprej osvetlitev, nato ojačanje), da imajo teksturo za ujemanje, nato pa ponovno vzpostavi avtomatsko osvetlitev.

Chloros samodejno poravna vsak niz po povezavi, v obeh načinih prikaza, ko se samodejna osvetlitev elementov ustalijo (poskuša do 5 minut); tudi združen predogled se poravna ob odprtju. Ločena matrica nima sestavljene ploščice, zato ta oddelek preverja stanje poravnave in sam izvede ponovno kalibracijo. Ponovno kalibrirajte, če se je spremenil fokus ali globina prizora. Poravnava je **po zasnovi omejena na sejo** — nikoli se ne shrani v profil, saj je odvisna od trenutne razdalje prizora. Posnetke je še vedno mogoče izvoziti s poravnavo pikslov (glej [Poravnani izvozi](capture.md#per-array-controls)).

### Pametna vinjeta

* Polje za potrditev **Omogoči korekcijo**— uporabi oceno vinjete za vsako kamero posebej v radiometrični verigi (v živo**in** pri izvozih).
* **Kalibriraj iz trenutnega pogleda**— najprej usmerite niz na enotno tarčo (ravno ploščo, steno ali nebo); vsaka kamera se izravna posebej, stanje pa poroča o izboljšanju ravnosti »n/N kamer · −x,x %«.**Izbriši** odstrani oceno.
* Stanje: »Kalibrirano«, »Ni kalibrirano« ali »Delno kalibrirano — ponovno izvedi«, kadar ocene le nekaterih kamer še veljajo pri njihovem trenutnem kadriranju (na primer po tem, ko je funkcija »Ponovna kalibracija« premaknila okno izreza neke kamere) — ponovno izvedi kalibracijo.
* Natančno nastavite za vsako kamero posebej s pomočjo drsnika **Vignette** v [Živem predogledu](#live-preview).

### Predogled v živo

* Potrditveno polje **Histogram**+ drsnik širine (0,10–0,90 okvirja) — pas histograma v živo na ploščici vsakega člana hkrati (vključno s ploščicami z razdeljenimi kanali), [Prekrivni histogram](#display-overlays-drawn-over-the-live-feed) za celotno skupino. Prikaže se kot vklopljen, če ga imajo že vsi člani. Na**ločeni** skupini je to edina vrstica v skupini.
* *Samo za združene skupine:* **Indeks**: potrditveno polje + zobnik — odpre skupni [kalkulator indeksa](#index-calculator-pane) s pasovi, narisanimi iz**vseh** kamer članov. Vrstica za predogled izraza spodaj prikazuje trenutni izraz („Izraz ni nastavljen — odprite kalkulator, da ga ustvarite“), ki se osvežuje vsako sekundo.
* *Samo v kombinaciji:* spustni meni **Ločljivost upodabljanja**(iste prednastavitve kot za posamezno kamero, privzeto 720p): višina toka v živo**in** velikost shranjenega sestavljenega izvoza. Opomba v oknu: »Predogled + velikost shranjenega sestavljenega izvoza. Slike posameznih kamer se vedno izvozijo v polni ločljivosti.«

### Plasti prikaza *(samo v kombiniranem načinu)** Polje za potrditev **Omogoči** (privzeto izklopljeno — glavna kamera se prikaže neposredno; vklopljeno = sestavljena slika v plasteh).
* **Spustni meniji**Prednji plan**/**Ozadje**: vsaka posamezna kamera (po imenu) ali**Indeks**. Ko je za Prednji plan izbran Indeks, se v pikslah zunaj LUT Min/Max prikaže sloj Ozadje.

### Razdeljen pogled *(samo v kombinaciji)*

**»Prikaži posamezne kamere«**— gumb**Razdelitev / Skrij posamezne kamere**, ki dodaja posnetek v živo vsake posamezne kamere kot ločene ploščice v mreži poleg sestavljenega posnetka. Ploščice berejo obstoječi okvirni pomnilnik niza (brez dodatne povezave s kamero). Samo v pogledu mreže; shrani se po nizu skupaj s projektom.

### Zmožnosti

Plošča, ki je samo za branje in se osvežuje vsakih 5 s:

* **Oznaka stopnje**: »Hkratno zajemanje« (zelena) · »Hkratno zajemanje (FTD – postopno oddajanje)« (zelena) · »Postopno zajemanje (100 ms zamik)« (rumena) · »Prevelika konfiguracija« (rdeča).
* **Stanje okvirja**: »x,xx % nepopolno« — zelena pod 1 %, rumena pod 5 %, rdeča pri 5 % ali več.
* ****Povezovalna linija**: »NIC {mbps} Mbps – trajna {MB/s} MB/s«.

To je trenutni proračun pasovne širine niza. Za osnovne vrednosti fps in omrežni model — ter kaj spremeniti, ko stopnja preide v rumeno ali rdečo — glejte [Nizi z več kamerami](arrays.md) in [Referenco CLI](../reference/cli-reference.md).

<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->

## Okno »Index Calculator«

Tretja stran stranskega menija, ki jo delita nastavitve »Index« za posamezno kamero in nastavitve »Index« za združeno matrico (vsakokrat le ena – naslov se glasi »Index Calculator — <camera name="">« ali »Index Calculator — <array name="">«). Prejme seznam pasov (naravni filtri kamerenaravna pasova kamere ali vsi pasovi vseh članov niza), trenutni izraz in konfiguracijo LUT (vklop/izklop, raven – privzeto 7 stopenj, min – privzeto 0,2, max – privzeto 1) ter histogram indeksa v realnem času. S klikom na **Uporabi** se izraz potrdi; spremembe LUT se v realnem času uporabijo v predogledu.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Nastavitve po posamezni kameri v primerjavi z nastavitvami, ki jih upravlja niz

Kratek pregled, kaj se nahaja kje, ko je kamera članica niza:

| Upravlja niz (v oknu kamere samo za branje) | Še vedno po posamezni kameri znotraj niza |
| --- | --- |
| Format pikslov, ločljivost, združevanje pikslov | Samodejna osvetlitev (osvetlitev, ojačanje, ciljna vrednost, izglajevanje, ROI) |
| Način/vir sprožitve, hitrost snemanja | Odstranjevanje šuma, ostrina, vinjeta |
| | Usmeritev (zrcaljenje/vrtenje), prekrivni prikazi (nastavitve niza → v živi predogled se histogram nastavi na vseh članih hkrati), točkovni merilnik |
| | Indeks (nizi z ločenim prikazom), vezava svetlobnega senzorja |

Druga medsektorska obnašanja:

* **Kombinirani ali ločeni prikaz** se izbere ob povezavi matrice: kombinirani = ena poravnana sestavljena ploščica (člani prenašajo sliko le prek razdeljenega pogleda); ločeni = vsak član prikaže svojo sinhronizirano ploščico. Kamera nikoli ne prikazuje hkrati samostojnega posnetka in ploščice matrice.
* ****Samodejna ponovna povezava**: ob odprtju shranjenega projekta se obnovijo kamere in matrice, vsaka shranjena nastavitev pa se ponovno uporabi v ozadju, preden se tokovi nadaljujejo.
* **Omejevanje zajemanja**: skrite ali zaustavljene kamere so izključene iz funkcije »Zajemi vse«; matrica je popolnoma blokirana le, če so VSE enote skrite/zaustavljene. Glej [Nastavitve in načini zajemanja](capture.md).

## Kako se nastavitve ohranijo

Stanje zavihka kamere se shrani **skupaj s projektom**, ne v brskalniku:

* Vsaka reaktivna sprememba posname trenutno stanje kamer in nizov v datoteko `cameras.json` projekta (z odmevom 500 ms). To vključuje imena in barve kamer, nastavitve osvetlitve/ojačitve/avtomatske osvetlitve (AE), format pikslov/resolucijo/združevanje pikslov, hitrost sprožanja, nastavitve predogleda (resolucija upodabljanja, odstranjevanje šuma, ostrina, vinjeta, barvni profil, svetli deli, nasičenost/kontrast), usmerjenost, prekrivni sloji, delitve kanalov, konfiguracijo indeksa, nastavitve prediktivne avtomatske osvetlitve (AE), območje zanimanja (ROI) za avtomatsko osvetlitev, imena matrik, način prikaza, nastavitve zajema matrik (vključno z izrezom ROI, ali je ROI vklopljen in shranjenim izrezom, ko je izklopljen), »Wire Budget« matrike ter mrežni blok (povečava posnetka, način pregleda, zaklep mreže, ročni vrstni red ploščic, skrite kamere, zaprte ploščice, aktivna kamera).
* Povezave svetlobnih senzorjev se shranijo v datoteki projekta `sensors.json`.
* Ponovno odprtje projekta ponovno poveže strojno opremo in ponovno uporabi vse nastavitve.
* **Če ni odprt noben projekt = samo seja**: brez projekta se ob zaprtju programa Chloros nič ne ohrani.
* Velja le za sejo, ne glede na projekt: stanje zaustavitve, vzorci točkovnega merilnika svetlobe, potrditveno polje »Kalibracijski cilj« za posamezno kamero (vedno odprto v izklopljenem stanju) in profil poravnave matrike (po zasnovi se ponovno izračuna za vsako sejo).
* Ena izjema: izbire za izvoz **Nastavitev zajema** in način zajema se ohranijo po posameznih projektih v lokalnem pomnilniku aplikacije in ne v `cameras.json` — glej [Nastavitve in načini zajema](capture.md).</array></camera></serial>

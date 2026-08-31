# Zavihek DAQ v Chloros

Zavihek DAQ — v stranskem meniju Chloros označen kot **Svetlobni senzorji** — je vmesnik za nadzor v realnem času za [svetlobne senzorje DAQ-U, DAQ-M in DAQ-E](README.md): senzorje lahko priključite prek katerega koli prenosnega protokola, v realnem času spremljate kalibrirane spektre, izračunate odbojnost v realnem času iz para senzorjev in snemate datoteke `.daq` neposredno v svoj projekt.

Zavihek postane dostopen, ko se zaključi zagon strežnika Chloros. Grafi na zavihku se napajajo prek storitve DAQ sistema Chloros prek povezave v realnem času, ki se v primeru prekinitve samodejno ponovno vzpostavi (z odlogom 2–10 s); dokler storitev ni dosegljiva, se v vrstici »Status« senzorja prikaže **No Server**.

Postavitev je sestavljena iz **stranskega pasu senzorjev**(ena vrstica na povezan senzor) in**območja grafikonov** (ena ploščica grafikona na senzor ali skupino).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Povezovanje senzorja

Kliknite **Poveži senzor** na vrhu stranskega pasu. V glavnem območju se odpre pogovorno okno za povezavo (ali kot prekrivni sloj ob dodajanju drugega senzorja – v tem primeru se prikaže gumb Prekliči).

| Nadzor | Delovanje |
| --- | --- |
| **Vrsta naprave** | `DAQ-U (USB)` (privzeto), `DAQ-M (Bluetooth)` ali `DAQ-E (Ethernet)`. Preklop ponovno zažene iskanje za novo izbrani način prenosa. |
| **Vrata / Naprava BLE / Ime gostitelja / IP** | Prikaže seznam odkritih naprav kot `device - description`; samodejno se izbere prvi vnos, ki je prepoznan kot senzor. Med iskanjem se prikažejo `Scanning...` (USB), `Scanning (N)...` z 8-sekundnim odštevanjem (BLE) ali `Discovering ethernet sensors (N)...` z 5-sekundnim odštevanjem (Ethernet). Pri praznih rezultatih se prikaže `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Osveži** | Takoj ponovno skenira izbrani prenos (onemogočeno med skeniranjem BLE/Ethernet). |
| **Poveži** | Omogoči se, ko je izbrana naprava; med vzpostavljanjem povezave se preimenuje v `Connecting...`. |

Iskanje poteka le **medtem ko je na zaslonu prikazan pogovorni okno za povezavo** in se ponavlja vsake 15 sekund le za izbrani prenosni protokol – samo odpiranje zavihka ne sproži iskanja. V primeru napake se v pogovornem oknu prikaže: *»Povezava ni uspela. Poskusite odklopiti in ponovno priključiti senzor, nato pa ponovno kliknite Poveži.«*

Stranski pas se samodejno odpre, ko se poveže vaš prvi senzor.

{% hint style="info" %}
**DAQ-E se ne prikaže?** DAQ-E nima LED-indikatorja stanja — preverite indikator PoE/povezave na stikalu ali vtičnici, v katero je priključen, in po vklopu počakajte nekaj sekund, da se naprava zagoni. Naprava Chloros mora biti v isti domenski skupini (mDNS ne prehaja skozi usmerjevalnike). Na napravi Windows sprejmite poziv požarnega zidu Defender, ko naprava Chloros prvič vzpostavi povezavo s svojimi večvrstnimi vtičnicami (mDNS UDP 5353, podatki DAQ-E UDP 5002, PTP UDP 319/320). Dve enoti DAQ-E v istem LAN-omrežju se odkrijeta ločeno, vsaka pod svojim imenom gostitelja `daq-e-<id>.local`.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Vrsta naprave ponuja DAQ-U (USB), DAQ-M (Bluetooth) in DAQ-E (Ethernet)</figcaption></figure>***

## Stranski pas s senzorji

Vsak priključen senzor dobi eno vrstico (plus ena vrstica na skupino »Ambient+Object«). Vrstice je mogoče preurediti s povlekanjem, njihov vrstni red pa spremeni tudi vrstni red ploščic grafa. Kliknite na vrstico, da ta senzor/skupina postane aktivni graf v seznamu.

| Element | Pomen |
| --- | --- |
| Barvni levi rob | Barva grafa senzorja. |
| Znak prenosa | `DAQ-U` / `DAQ-M` / `DAQ-E` ali zelena oznaka `REF` za skupino odbojnosti Ambient+Object. |
| Ime naprave | Privzeto je serijska številka senzorja (njegova stalna identiteta za kalibracijo, imena datotek `.daq` in ujemanje pri uvozu); po meri določena imena se ohranijo za vsak projekt posebej. |
| **Kalibrirano** (zeleno) | Prikaže se, ko je naložen tovarniški kalibracijski paket senzorja, tj. ko so spektri izraženi v W/m²/nm. |
| Ikona **Na voljo posodobitev** (rumenkasta, samo DAQ-E) | Trenutna različica vgrajene programske opreme je starejša od slike, ki je priložena tej različici Chloros. Med posodobitvijo prikazuje trenutni napredek (`Flashing… N%`, `Restarting sensor…`, nato `Updated X → Y` ali `Failed`). |
| Oko | Vklopi oz. izklopi vidnost tega senzorja na grafu. |
| Zobnik | Odpre modalno okno z nastavitvami za posamezni senzor (spodaj). |
| ✕ (rdeče) | Odklopi senzor ali odstrani skupino »Ambient+Object«. |

Nad vrsticami sta dva gumba:

* **Poveži senzor** — odpre pogovorno okno za povezavo (med postopkom se preimenuje v `Connecting...`).
* **Snemi vse / Ustavi vse**— začne ali ustavi snemanje `.daq` na**vseh**povezanih senzorjih. Zahteva vsaj en senzor**in odprt projekt** (napotek: »Odpri projekt za snemanje«); med snemanjem se obarva rdeče.

V praznem stanju se prikaže »Ni priključenih senzorjev«.

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Nastavitve za posamezen senzor (modalno okno z zobnikom)

Odprete ga z ikono zobnika v vrstici senzorja. Vsebina po vrstnem redu:

* **Vrstice z informacijami** — Tip naprave (DAQ-U/M/E), Povezava (`Serial (USB)` / `Bluetooth` / `Ethernet`), vrata (COM-vrata, naslov BLE ali gostitelj) in serijska številka.
* **Poročilo o kalibraciji: Prenos** — prenese certifikat o kalibraciji te enote, ki je sledljiv do NIST (PDF), in ga odpre v vašem pregledovalniku PDF. Na voljo, ko je serijska številka znana; certifikat se shrani v predpomnilnik ob prvi povezavi.
* **Ime naprave** — kliknite na svinčnik za preimenovanje; nastavljeno ostane za posamezen projekt.
* **Barva črte grafa** — barvni vzorec; nastavljeno ostane za posamezen projekt.
* **Čas integracije (ms)**— drsnik + število,**1–500 ms**, privzeto**32 ms**. Onemogočeno, ko je AE vklopljeno.
* **Povprečje okvirjev**— drsnik + število,**1–50 okvirjev**, privzeto**20**.
* **AE: VKLOPLJENO/IZKLOPLJENO**— stikalo za samodejno osvetlitev;**privzeto VKLOPLJENO** ob povezavi. Izklopite ga, da čas integracije nastavite ročno.
* **Zaustavi prenos / Začni prenos** — zaustavite ali nadaljujte prenos v živo.
* **Snemanje / Zaustavi snemanje** — snemanje `.daq` po posameznih senzorjih (potreben je odprt projekt).
* **Cap** — profil korekcije cap (naslednji oddelek).
* **Vrstice z informacijami v realnem času** — čas integracije (ms), FPS, vzorci, snemanje (rdeče `REC` ali `Off`) in stanje (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Samo za DAQ-E: vrstice omrežja, vgrajene programske opreme in PTP

* **Ime gostitelja / IP** — trenutni naslov enote.
* **Programska oprema** — trenutna različica programske opreme, skupaj s celico za akcijo:<version\>

gumb</version\>

**Posodobi na \<version\>** se prikaže, ko ta različica Chloros vsebuje novejšo sliko programske opreme DAQ-E. Posodobitev se prenese prek omrežja v približno 30 sekundah; senzor se samodejno ponovno zažene in ponovno poveže, prekinjen prenos pa ohrani trenutno firmware nespremenjeno. Napredek se prenaša v realnem času (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`), celica pa prikazuje `Up to date`, ko je posodobitev aktualna.
* **PTP Sync** — trenutno stanje PTP (se vrne na `unknown`). Firmware DAQ-E v1.2.0+ sodeluje v standardu IEEE 1588 PTPv2 kot ura, ki deluje izključno kot podrejeni; backend gostitelja Chloros je glavna ura PTP, vsaka kamera DAQ-E in LATTICE v omrežju LAN pa ji sledi v domeni 0, pri čemer ohranja časovne oznake z odstopanjem približno 1 ms.

Za skupino »Ambient+Object« modalno okno »Gear« prikazuje le senzorje vira skupine, ime naprave in barvo grafične črte.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->

### Izbira pokrovčka

Spustni meni **Pokrovček** pove Chloros, kateri fizični pokrovček je nameščen na difuzorju senzorja, in na vsak spekter uporabi tovarniško izmerjeni korekcijski profil tega pokrovčka. Možnosti so odvisne od modela:

| Model | Možnosti pokrovčkov |
| --- | --- |
| DAQ-U | Noben (gol senzor), FOV 15°, FOV 30°, FOV 45°, FOV 60°, FOV 90°, »Sunshine« (kosinusni korektor) |
| DAQ-M | Brez (gol senzor), Sunshine (kosinusni korektor) |
| DAQ-E | Brez (gol senzor), FOV 15°, FOV 45°, FOV 90°, Sunshine (kosinusni korektor) |

**Privzeta nastavitev za vsak model je »Sunshine« (kosinusni korektor)** — MAPIR vsak DAQ dobavlja z nameščenim pokrovčkom »Sunshine«, kar je standardna konfiguracija za zunanjo uporabo: 180° polkrožni pogled s kosinusno napako ≤ ±4 % do 60° in ≤ ±4,5 % do 70° (nepriporočljivo pri sončni višini pod ~15°), z vgrajenim dušenjem (~12×). Vaša izbira ostane shranjena v projektu.

{% hint style="warning" %}
**Izbira pokrovčka mora ustrezati fizičnemu pokrovčku.**Niti senzor niti programska oprema ne moreta zaznati, kateri pokrovček je nameščen. Izbira vpliva tako na korekcijo v realnem času kot na oznako, vpisano v vsako datoteko `.daq` – zaradi ~12-kratnega slabljenja pokrova Sunshine nedeklarirana zamenjava pokrova povzroči napačno korekcijo spektra za približno ta faktor. (Odstranitev in ponovna namestitev istega pokrova povzroči ponovitev odstopanja v višini približno 1,5 %.) Izberite**None (goli senzor)** samo, če je pokrovček fizično odstranjen; pri DAQ-E se pri izbiri »None« še vedno uporablja tovarniški geometrijski profil za vgrajeni stekleni difuzor – to ni brez učinka – in goli DAQ-E je konfiguracija za laboratorijsko delo, ne pa podprta konfiguracija za uporabo na terenu.
{% endhint %}

{% hint style="info" %}
Nadgradnja iz prejšnjega priročnika: stikalo »Sunshine Diffuser Installed« na strani brskalnika iz različice 1.1.0 je odpravljeno. Upravljanje pokrovčkov zdaj poteka prek profila pokrovčka za vsak senzor posebej, ki se uporablja na strani strežnika.
{% endhint %}

***

## Območje grafikonov

Na fiksni zgornji vrstici sta **stikalo za preklop med seznamom in mrežnim prikazom**ter drsnik**za povečavo grafikona** (velikost ploščic 200–2000 pikslov). Pogled se samodejno preklopi na mrežo, če obstaja več kot ena skupina grafikonov, in nazaj na seznam, če je ena ali manj. Način prikaza in velikost grafikona se ohranjata za vsak projekt posebej.**Spektralni grafikon** za vsak senzor prikazuje:

* **Os X** — Valovna dolžina (nm). Mreža senzorjev obsega 340–1010 nm v korakih po 5 nm (135 točk), za prikaz pa je interpolirana na 1 nm.
* **Os Y** — moč (W/m²), z avtomatsko izbrano predpono SI (m/µ/n) na podlagi vrhovne vrednosti. Spektri so radiometrično kalibrirana spektralna obsevanost (W/m²/nm) na vseh treh nosilcih.
* Mavrična spektralna barvna polnitev pod posamezno krivuljo; več senzorjev na enem grafu se prekriva kot barvne črte z zatemnjenimi polnjenji.
* **Premik miške**— navpični kazalec z valovno dolžino in vrednostjo za posamezni senzor;**povlecite** za povečavo (med povečavo se prikaže gumb za pomanjšanje).
* Gumb **+** (samo v pogledu mreže) za dodajanje senzorja na ta grafikon ali ustvarjanje skupine (spodaj).
* Ime naprave je centrirano na vrhu, medtem ko se prikazuje vrteča ikona, dokler ne prispe prvi okvir.

**Nasičenost** ni označena na samem grafu: nasičen senzor prikazuje rdeče besedilo stanja `SATURATED` in rdečo vrstico `Saturated: Yes` v tabeli podatkov v realnem času. Zmanjšajte čas integracije ali ponovno vklopite AE, da se stanje izniči.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->

***

## Tabela podatkov v realnem času (pogled v obliki seznama)

Pod grafikonom v pogledu v obliki seznama, ki se osvežuje vsakih 500 ms:

* **Vsi modeli**: Vzorec barve svetlobe (sRGB iz CIE XYZ), Zasičenost (Da/Ne), CIE 1931 X/Y/Z, Kromatičnost x/y, CIE u′/v′, CCT (K), CRI (Ra), Dominantna valovna dolžina (nm), vrhunska valovna dolžina (nm), čistost vzbujanja, Duv, CIE L\*/a\*/b\* in Munsell H/V/C.
* **Samo kalibrirani senzorji**(kateri koli izmed DAQ-U / DAQ-M / DAQ-E, ko je naložen paket tovarniške kalibracije — prepoznaven je po zeleni oznaki**Kalibrirano** v vrstici senzorja): Skupna moč (W/m²), fotopični luks (lx), skotopični luks (lx), razmerje S/P, PPFD ter PPFD Red/Green/Blue (µmol/m²/s) ter opične sevanjske jakosti – S-konus, melanopična, rodopična, M-konus, L-konus (vse v W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Skupine odbojnosti (okolje + objekt)

Dva povezana senzorja lahko združite v prikaz odbojnosti v realnem času — brez uporabe kamere:

1. V pogledu mreže kliknite **+**na ploščici grafa in izberite**Združi okolje + objekt**.
2. Izberite senzor **Vir okoljske svetlobe**in senzor**Skener objekta**(dva različna senzorja), nato kliknite**Ustvari**.

Chloros izračuna R(λ) = objekt(λ) / okolje(λ) za vsako valovno dolžino iz dveh tokov v realnem času (0, če je ambientna svetloba ≤ 0). Oznaka skupine sledi kalibracijskemu razredu senzorjev:

* Oba senzorja sta kalibrirana (paket naložen) → **»Navidezna odbojnost«**.
* Če je kateri koli senzor nekalibriran → **»Relativna odbojnost«**.

Skupina se prikaže kot zelena vrstica `REF` v stranskem pasu in v lastnem grafu (mavrična barva, vrednosti ob preletu z miško do 4 decimalnih mest, povečanje s povlekanjem).

Meni **+**ponuja tudi možnost**Dodaj nov senzor** s tremi možnostmi: *Združi nov senzor* (pridruži se temu grafu), *Premakni obstoječi senzor sem* ali *Poglej nov senzor* (lasten graf).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->

### Tabela indeksov vegetacije

V pogledu seznama se pod grafikonom skupine odbojnosti nahaja tabela indeksov vegetacije, izračunana na podlagi trenutne odbojnosti v središčih pasov **modri 450 / zeleni 550 / rdeči 670 / NIR 800 nm** (vrednosti z 4 decimalnimi mesti, `---`, če izračun ni mogoč; za polno ime indeksa premaknite kazalec miške nad ime indeksa):

* **Vedno prikazani** (neodvisni od merila, katera koli kombinacija senzorjev): NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR.
* **Samo če sta oba senzorja kalibrirana glede na moč** (naloženi sta oba paketa): EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

.***

## Snemanje datotek `.daq`

* Za snemanje je potreben **odprt projekt** — sicer sta onemogočena tako možnost »Snemi vse« (stranski trak) kot tudi gumb »Snemi« za posamezni senzor.
* Datoteke se shranijo v mapi **`<project folder>/light_sensor/`**; imena datotek vsebujejo ID senzorja in časovni žig, ime naprave pa se shrani skupaj z zapisom.
* Ko se snemanje ustavi (»Stop«, »Stop All« ali prekinitev povezave med snemanjem), se končana datoteka `.daq` **samodejno doda k odprtemu projektu** — pojavi se na seznamu datotek projekta brez ročnega dodajanja, pripravljen za uporabo kot podatki o padajočem sevanju za [obdelavo odbojnosti](README.md).
* Med snemanjem se v vrsticah v realnem času v oknu nastavitev prikaže rdeč indikator `REC`.

Za kvantitativne vrednosti sevanja izračunajte povprečje podatkov iz najmanj 15 sekund – to je značilnost instrumenta, ne pa napaka.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->

***

## Razporeditve več senzorjev in trajnost projekta

* Združite več senzorjev na enem grafu (skupne osi), ohranite ločene grafe (avtomatska razporeditev mreže), premikajte senzorje med grafi, s povlekanjem spreminjajte vrstni red vrstic/ploščic ter posamezne senzorje skrijte s stikalom v obliki očesa.
* Za vsak projekt se v Chloros ohranijo: imena naprav, barve grafov, velikost grafa, način prikaza ter nastavitve vsakega senzorja (čas integracije, povprečenje okvirjev, stanje AE, izbira omejitve).
* **Ob ponovnem odprtju projekta se senzorji samodejno ponovno povežejo** po naslovu — COM-vrata za DAQ-U, BLE-naprava za DAQ-M, ime gostitelja mDNS za DAQ-E (prepoznano tudi, če se je IP-naslov enote spremenil) — ter ponovno uporabi shranjeni profil pokrovčka, povprečevanje okvirjev, stanje AE in ročni integracijski čas za vsak senzor.***

## Seznanjanje kamere (DLS)

Ni ničesar za seznanjanje. Za razliko od delovnih tokov DLS za drone, ki svetlobni senzor vnaprej povežejo s kamero, Chloros podatke DAQ uskladi s slikami šele naknadno: ob uvozu/obdelavi se odčitki `.daq` interpolirajo na časovni žig osvetlitve vsakega posnetka. Posnemajte s katerim koli priključenim senzorjem (`.daq` se samodejno vključi v projekt), obdelava odbojnosti pa poišče prave odčitke glede na čas — glejte [Svetlobni senzorji DAQ](README.md) za informacije o tem, kako se uporabljajo podatki o svetlobi, ki pada navzdol.</version\>

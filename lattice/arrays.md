# Večkamernih sistemov

**Sistem**LATTICE je sestavljen iz dveh ali več kamer LATTICE, povezanih v eno sinhronizirano enoto. Ena kamera je**glavna**: po skupni sinhronizacijski liniji (privzeto**Line2**) pošlje strojni GPIO sprožilni impulz, tako da vsaka kamera zajame isti trenutek. Chloros dodaja časovno sinhronizacijo PTP, predogled v živo (posamezne ploščice po kameri ali en sam poravnan večpasovni kompozit) ter sinhronizirano zajemanje – vsak zajemni prehod ustvari eno**skupino sličic**, v kateri imajo vse kamere isti časovni žig in ID sličice (na izhodu zajema se prikaže kot `fid:N`).

Mono (M3M) kamere ustvarjajo indekse vegetacije prav z uporabo nizov – ena kamera prispeva en pas, niz pa jih poravna v večpasovni sklop. Glej [Mono kamere in indeksi vegetacije](mono-indices.md).

Obstajajo trije enakovredni načini za povezovanje niza, vsi pa izvajajo isti potek »smart-prep«:

| Vmesnik | Vstopna točka |
| --- | --- |
| GUI | Zavihek »Kamere« → **Poveži niz** (modri gumb) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (prva serijska številka = glavna) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (prva serijska številka = glavni) |

Smart-prep izvede v naslednjem vrstnem redu: preverjanje zmogljivosti omrežja (ICMP DF ping + GVSP preverjanje), izbiro sinhronizacijskega nivoja, samodejno zmanjšanje velikosti okvirja, da se prilagodi povezavi, vklop PTP, samodejno izbiro formata pikslov za vsako kamero, samodejno nastavitev osvetlitve na podlagi shranjenega stanja vsake kamere ter konfiguracijo sprožilca GPIO na Line2.

{% hint style="info" %}
Kamere morajo biti dosegljive prek povezave, preden bo karkoli od tega delovalo – glejte [Povezovanje kamer](connecting.md) za odkrivanje, naslavljanje in prenos kalibracijskih podatkov ob prvi povezavi. Pri sistemih z več kamerami so nastavitve sprejemnega obroča na gostiteljski mrežni karticiso enako pomembne kot hitrost povezave; popolna tabela simptomov in rešitev je v [CLI Referenca § Nastavitev in optimizacija gostiteljske mrežne kartice](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Pogovorno okno »Povezava niza«

Zavihek »Kamere« → **Poveži niz**odpre čarovnika v treh korakih:**Izberi → Način prikaza → Nastavitve**.

### Korak 1 — Izberite glavno in podrejene

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered. Table showing Camera / Serial / IP / Master radio / Slave checkbox columns, with the green "GPIO master detected — selections pre-populated" probe banner visible above the table. -->

kamere Pogovorno okno takoj po odprtju pregleda omrežje („Pregledovanje omrežja...“), nato pa preveri ožičenje sprožilca GPIO („Preverjanje ožičenja GPIO...“). Za sestavo niza potrebujete vsaj **2 kamere**.

Preizkus ožičenja po možnosti vnaprej izpolni izbiro vlog in prikaže eno od treh obvestil:

| Obvestilo | Pomen |
| --- | --- |
| »Zaznana glavna naprava GPIO — izbire so vnaprej izpolnjene« (zeleno) | Preizkus je odkril topologijo sprožilca; potrditvena polja za glavno in podrejeno napravo so že izpolnjena. |
| »Glavna naprava ni zaznana – preverite GPIO kabel« (oranžno) | Nobena kamera ni zaznala sprožilnega impulza; preverite sinhronizacijsko ožičenje. Vloge lahko še vedno izberete ročno. |
| »Ni sinhronizacijskega kabla: {serijske številke}« (oranžno) | Naštete kamere nimajo priključenega sinhronizacijskega kabla. |

Tabela kamer vsebuje stolpce **Kamera / Serijska številka / IP / Glavna (radio) / Podrejena (polje)**:

* Izberite natanko **eno glavno kamero**in**eno ali več podrejenih kamer**. Ponovni klik na radio trenutne glavne kamere to izbiro izbriše.
* Kamere, označene z **„Brez sinhronizacijskega kabla“**, ni mogoče izbrati kot podrejeno kamero — podrejena kamera brez sprožilnega ožičenja bi namreč neskončno dolgo čakala na sinhronizacijski liniji in prenašala mrtvo sliko. Takšno kamero raje priključite kot samostojno kamero.
* Kamere, ki so že priključene samostojno, *niso* onemogočene: povezava v matrico prekine samostojno sejo in kamero ponovno odpre znotraj matrice.

**Naprej: Način prikaza →**se omogoči, ko sta izbrana glavna kamera in vsaj ena podrejena kamera.**Ponovno skeniranje** ponovno zažene iskanje in preverjanje ožičenja.

{% hint style="warning" %}
**Prekliči** je onemogočen, medtem ko poteka skeniranje ali preizkušanje ožičenja – preklic med preizkušanjem ožičenja lahko povzroči sesutje kamere SDK na firmware-u kamere LATTICE. Počakajte, da se vrteči indikator ustavi.
{% endhint %}

### Korak 2 — Način

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

prikaza | Način | Kaj dobite |
| --- | --- |
| **Ločene kamere** | Ena ploščica v živo na kamero, vse se sprožijo skupaj, tako da ostanejo posnetki sinhronizirani. Vsaka kamera ohrani svojo barvo in nastavitve. |
| **Združene kamere** *(privzeto)* | En sam ploščica, ki prikazuje poravnano večpasovno NDVI/index sestavljeno sliko. Kamere si delijo barvo matrike. |

Način prikazovanja spreminja le predstavitev v živo — način zajemanja je v obeh primerih enak.

### Korak 3 — Nastavitve matrike in predvideni izid

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Pin resolution / Trigger Rate controls, right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

Ob vstopu v to sceno Chloros zaprosi strežnik za **priporočilo**in samodejno uporabi kombinacijo ROI + združevanja slikovnih pik, ki ustreza sprejemnemu obroču vaše mrežne kartice (prednost daje združevanju pred izrezovanjem ROI, saj združevanje ohranja celotno vidno polje). Vsaka sprememba, ki jo naredite, ponovno izvede analizo v realnem času in posodobi desni panel**Predvideni izid**.

Levi stolpec – nastavitve:

| Nadzor | Možnosti | Privzeto | Opombe |
| --- | --- | --- | --- |
| **ROI (vidno polje)** | Celotno (2048×1536) / Polovično (1024×768) / Četrtinsko (512×384) | Celotno | Izrez senzorja: polovični/četrtinski izrez na manjše območje pri izvirni gostoti pikslov. |
| **Združevanje pik** | 1× / 2× (vsota 2×2) / 4× (vsota 4×4) | 1× | Strojno združevanje pik: 2×2 = polno vidno polje pri četrtini stroškov žice; 4×4 = polno vidno polje pri 1/16. Skrito, če kamere ne podpirajo združevanja pik. |
| **Slika na vodniku** (odčitavanje) | — | — | Širina×višina po združevanju pik, ki se dejansko pošilja po vodniku, zaokrožena navzdol na večkratnike števila 16 (najmanj 64). |
| **Ločljivost pinov**| potrditveno polje | izklopljeno | Chloros običajno samodejno poveča združevanje ob povezavi, ko predvidena hitrost pade pod**1,5 fps**. Z zaklenitvijo ohranite izbrano velikost okvirja in sprejmete nižjo hitrost — ter spremenite preobremenjeno konfiguracijo v trdno zavrnitev povezave namesto avtomatskega zmanjšanja hitrosti. |
| **Hitrost sprožitve** | 0,5–60 sličic na sekundo, korak 0,1 | prazno = samodejno | Hitrost sprožitve glavne naprave. Pustite prazno, da jo Chloros izračuna sam. |
| **Proračun za povezavo**| 20–2000 MB/s, korak 10 | prazno = avto | Koliko lahko gostitelj dejansko sprejme, v MB/s –**edina številka, od katere je odvisna celotna dodelitev polja.** Samodejno zaznano iz omrežnega adapterja. Zmanjšajte ga, če niz poroča o poškodovanih okvirjih: zaznana vrednost precenjuje zmogljivosti USB-adapterjev in skupnih stikal. Sprememba te vrednosti ponovno zažene projekcijo v realnem času. |

Desni stolpec — **Predvideni izid**:

* **Raven sinhronizacije** — »Hkratno zajemanje« (zelena), »Hkratno zajemanje (FTD – postopno oddajanje)« (zelena), »Postopno zajemanje (100 ms zamik)« (rumena) ali »Konfiguracija je prevelika« (rdeča).
* **Projekcija fps** — prikazana kot razpon („temno → svetlo“), saj je hitrost sinhroniziranega niza omejena z osvetlitvijo najpočasnejše kamere.
* **NIC-linija** — hitrost povezave in trajna prepustnost („NIC {mbps} Mbps · trajna {N} MB/s“).
* **Preverjanje sočasnega oddajanja** — ali lahko sprejemni obroč gostiteljeve mrežne kartice sprejme en sočasen izbruh iz vseh kamer („Sočasno oddajanje: X MB · uporabni obroč NIC: Y MB ✓/✗«).
* **Preverjanje zmogljivosti vodnika** — agregatno povpraševanje v stabilnem stanju v primerjavi z zgornjo mejo vodnika, varno pred kolizijami („Zmogljivost vodnika: {povpraševanje} MB/s, ki ga zahteva {n} kamer · zgornja meja {zgornja meja} MB/s ✓/✗ prekoračena«).
* **»Največje število kamer na tem vodniku: {n} — določeno z minimalno pasovno širino na kamero, zato ga združevanje v skupine ne poveča.«** — prikaže se, ko se približujete (ali presegate) zgornjo mejo števila kamer.
* **»S temi nastavitvami BO PRIŠLO DO IZPADOV OKVIRJEV.«**— rdeče opozorilo z razlogom strežnika, skupaj s seznamom ovir in modrimi**predlogi za odpravo težav** („Da bi ta niz ustrezal omrežju“ / „Da bi omogočili sočasno snemanje“).**Uporabi in poveži** je onemogočeno, dokler ne obstaja napoved, njegova oznaka pa navaja razlog za zavrnitev:

| Oznaka gumba | Pomen | Kaj dejansko pomaga |
| --- | --- | --- |
| „Analizira se...“ | Analiza še poteka. | Počakaj. |
| **»Preveč kamer za to omrežje«**| Niz preobremenjuje omrežje (preverjanje agregata ni uspelo). | Manj kamer, jumbo okvirji od začetka do konca ali hitrejša mrežna kartica.**Manjši ROI NE bo pomagal** — glej spodaj. |
| **»Zmanjšajte ROI, da omogočite«** | Pri teh nastavitvah bi prihajalo do izgube okvirjev (preverjanje burst/ring ni uspelo). | Zmanjšajte ROI, povečajte binning ali popravite sprejemni obroč mrežne kartice. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->

Med povezovanjem se lahko prikaže zeleni **pano za prenos kalibracije** z vrstico napredka za vsako serijsko številko: ko kamero prvič priključite na napravo, Chloros prek GigE prenese njen tovarniški kalibracijski paket velikosti ~3,8 MB iz kamere (približno 70 sekund na kamero). Pri kamerah, ki so shranjene v predpomnilniku, se ta plošča nikoli ne prikaže. Glejte [Povezovanje kamer](connecting.md).

## Pasovna širina: koliko kamer se lahko priključi

Koliko kamer lahko nosi matrika, je odvisno od zmogljivosti omrežja, ne od Chloros, zato so podatki za načrtovanje navedeni v priročniku za strojno opremo: **[Načrtovanje pasovne širine matrike](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Kako Chloros to obravnava: pogovorno okno za povezavo izvede preizkus omrežja, napove dosegljivo hitrost posnetkov in izbere ustrezno stopnjo. Če niz preobremeni povezavo, povezava ne bo vzpostavljena, namesto da bi tiho izgubljal pakete — glejte zgoraj opisano okno z napovedanim izidom.

## Ko se okvirji izgubijo

Kamera lahko manjka v objavljeni skupini iz dveh popolnoma različnih razlogov,
za kateri so potrebne nasprotne rešitve. Chloros jih šteje ločeno, namesto da bi poročal o eni
»nepopolni« številki, ki ne navaja nobenega od razlogov:

| Kaj se je zgodilo | Kaj to pomeni | Kje iskati |
| --- | --- | --- |
| **Poškodovan**— okvir je prispel, a je bil strukturno poškodovan | Izguba paketov GVSP na omrežni poti |**Kapaciteta povezave**, sprejemni obroč mrežne kartice, jumbo okvirji, stikalo |
| **Nikoli ni prispel**— okvir sploh ni prispel | Kamera ni sprožila snemanja ali pa iz nje ni prišlo nič |**Sinhronizacijski kabel M8**, sinhronizacijska linija, ali so vsi člani vklopljeni |

Razdelitev se ponovno ovrednoti vsakih 10 sekund, medtem ko niz prenaša podatke. Nad 5 % se
zabeleži z navedbo obeh številk, vsak poškodovan pomnilnik pa se poroča ob prvem
pojavljanju za vsako kamero, nato pa se podatki združijo enkrat na minuto, da ostane dolgotrajna seja berljiva.

**Poškodovani okvirji z ničelno vrednostjo »nikoli ni prispelo« pomenijo, da sta sprožitev in sinhronizacija prek kabla brezhibni**in da je vsak izgubljen okvir na omrežni poti. Rešitev je zmanjšanje**Wire Budget** in
ponovna povezava.

{% hint style="warning" %}
**Znižanje hitrosti sprožitve ne pomaga pri poškodovanih okvirjih.** Hitrost pošiljanja paketov kamere
se nastavi enkrat, ob vzpostavitvi povezave. Znižanje hitrosti sprožitve spremeni, kako pogosto pride do
serije, ne pa, kako hitro sama serija potuje po kablu. Na izmerjeni napravi s 4 kamerami
5-kratno zmanjšanje hitrosti sprožanja ni spremenilo ničesar, medtem ko je znižanje žičnega proračuna s 240 na
200 MB/s pri isti napravi zmanjšalo delež poškodovanih okvirjev s 10,4 % na nič.
{% endhint %}

Delujoča matrica se ne more sama ponovno načrtovati – prekinite povezavo in jo ponovno vzpostavite, da bo izbirnik časa povezave
lahko deloval v skladu z novim proračunom.

### Mrežni adapterji USB so omejeni na 200 MB/s

USB-ethernetni adapter oglašuje svojo *ethernetno* hitrost povezave, vendar je tisto, kar lahko dejansko
vzdržuje, omejeno z USB-vodilom in njegovim gonilnikom. USB-ključ 10GbE je bil nekoč pripisan
pretoku približno 1000 MB/s – številka, ki je nikoli ni nihče izmeril – in uravnavanje
štirih kamer glede na to fantomsko rezervo je poškodovalo 6–18 % posnetkov, medtem ko je sistem
še vedno poročal o ustrezni ciljni hitrosti snemanja. Adapterji, priključeni prek USB-ja, so zdaj omejeni na
**200 MB/s**. Omejitev je absolutna in ne izražena v odstotkih, saj je omejitev v
samem vodilu: USB-adapter 1 GbE doseže približno 80 MB/s in ga to ne vpliva.

Če je vaš gostitelj resnično hitrejši od te omejitve, povečajte **Wire Budget**, da to odražate.

## Sinhronizacija časa PTP

*Sinhronizacija* okvirjev izhaja iz strojne sprožilne signale; **PTP** (IEEE 1588 PTPv2) zagotavlja primerljive *časovne oznake* na vseh napravah. Privzeto je omogočena ob priključitvi matrice:

* **Gostiteljski backend Chloros deluje kot PTP grandmaster**. Kamere LATTICE in svetlobni senzorji DAQ-E so mu podrejeni v domeni 0, zato so časovni žigi slik in spektri DAQ usklajeni z eno uro (~1 ms).
* `--no-ptp` (CLI) ga onemogoči za delo na delovni mizi — časovni žigi med kamerami takrat **niso** primerljivi.
* Preverite stanje sinhronizacije z CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Zavihek »Kamere« sam po sebi nima indikatorja PTP; tam so na voljo le za branje namenjeni podatki o sinhronizaciji za vsako kamero posebej: **Vloga**(Glavna/Podrejena),**Sinhronizacijska linija** in raven zmogljivosti niza. Stanje PTP za DAQ-E je prikazano v podrobnostih senzorja na zavihku »Light Sensors«.

## Pogled

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows, and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

na niz v realnem času Glavno območje prikaza ponuja dve razporeditvi (preklopite v zgornji vrstici): **pogled v obliki mreže**(vsak kvadrat je celica; z vlečenjem lahko spremenite vrstni red, ko je ključavnica mreže odklenjena) in**pogled v obliki seznama**(nizi v polni širini na vrhu, ena aktivna kamera spodaj). Drsnik**Povečava prenosa** spreminja velikost ploščic; pri širini celice pod 200 pikslov se prekrivni napisi z imenom in številom sličic na sekundo samodejno skrijejo.**Ločen način** prikazuje eno ploščico na kamero. Vsaka ploščica prikazuje:

* ime kamere (zgoraj levo),
* **prikaz sličic na sekundo** (zgoraj desno) — to je *dejanska hitrost zajema* kamere, ki jo poroča strežnik, ne pa hitrost osveževanja predogleda (predogled v živo je omejen na 30 sličic na sekundo ne glede na hitrost zajema),
* statusno piko — zelena (pretakanje) / rumena (nalaganje) / rdeča (napaka),
* **vrtečo ikono zastarelega okvirja**, če v 2 s ni prispel noben nov okvir — to je normalno za ~5 s po vsaki povezavi/prekinitvi, medtem ko strežnik ponovno uravnava porabo pasovne širine med kamerami.**Kombinirani način**prikazuje eno samo sestavljeno ploščico: strežnik izvede debayering, povečanje/pomanjšanje, poravnavo, odstranjevanje šuma, pretvorbo v sevanje po pasovih (plus odbojnost DLS, če je vezan svetlobni senzor), izračuna indeksni izraz matrike, uporabi LUT in rezultat prenese kot MJPEG. Dokler se ne prikaže prvi poravnan okvir, ploščica prikazuje svoj status: »Priprava matrike…«, »Kalibriranje poravnave…«, »Čakam na prvi okvir…« ali — če je porabljen časovni limit za ponovne poskuse samodejne poravnave (~30 s) — »Potrebna poravnava« z gumbom**Kalibriraj poravnavo**.

Koristni podatki o kombiniranem načinu:

* Kompozit je usklajen s sliko **glavne**kamere. Usmerjanje AE-ROI in točkovno merjenje osvetlitve na kompozitu sta natančna za glavno kamero in približna za podrejene kamere; uporabite**Split View** (nastavitve matrike → »Pokaži kamere v sistemu«) za ploščice z natančnostjo na piksel za vsako kamero posebej, ne da bi odpirali dodatne povezave s kamerami.
* **Sloji prikaza**(nastavitve matrike; privzeto izklopljeno) vam omogočajo izbiro prednjega in ozadnega sloja — katero koli kamero v sistemu ali**Indeks**. Če je prednji sloj nastavljen na**Indeks**, piksli zunaj LUT Min/Max prikazujejo ozadni sloj.
* **Ločljivost upodabljanja** (privzeto 720p) nastavi višino živega prenosa *in* velikost izvoza shranjenega kompozita. Slike posameznih kamer se vedno izvozijo v polni ločljivosti.
* Poravnava se izračuna za vsako sejo posebej in se nikoli ne shrani — za RMS-ostanke in gumb »Ponovna kalibracija« si oglejte razdelek o poravnavi v oknu nastavitev matrike.

## Snemanje: spremljanje proti analizi

Površine zajema matrike so jasno razdeljene na **razred za spremljanje**(snemajte, kar vidite) in**razred za analizo** (snemajte surove podatke, kalibrirajte kasneje):

| Delovni tok | Razred | Kaj shrani | GUI | CLI |
| --- | --- | --- | --- | --- |
| **Snemanje**(posnetki) | Analiza | Ena sinhronizirana skupina sličic na prehod; datoteke po posameznih kamerah na vsaki izbrani ravni izvoza (surovi/debayerirani/svetilnost/odbojnost/predogled/indeks) + `.daq` sidecar | Gumb**Zajemi vse** + Nastavitve zajema | `lattice array-capture` |
| **Snemanje indeksnega videa** | Spremljanje | Prikazani kombinirani indeksni kompozit v živo — 8-bitni, ločljivost predogleda, vgrajena LUT; zahteva odprt prenos v živo | ● Snemanje indeksnega videa (kombinirani nizi) | `lattice array-record` |
| **Surovi zaporedni posnetki → sestavi video**| Analiza | Surovi okvirji senzorja pri polni hitrosti zajema + manifest + `.daq`, nato pa rekonstrukcija v načinu brez povezave v kalibrirani video sevanja / odbojnosti / indeksa, časovno usklajen z odčitki DAQ | ⦿ Snemanje surovih zaporednih posnetkov →**Sestavi video** | `lattice array-burst` → `lattice array-build-video` |

Pravilo: če bodo piksli služili za *meritve*, uporabite zajem ali serijo posnetkov (za analizo); če pa želite le *pregledati ali prikazati*, kaj je zaznal senzor, posnemite indeksni video (za spremljanje).

### Nastavitve zajema (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->

Zobnik poleg **Capture All** odpre okno z nastavitvami zajema (zahteva odprt projekt – zajemi se shranijo vanj):

* **Način zajema**:**Enkratno**(en prehod) /**Neprekinjeno**(zaporedno; omejeno s številom zajemov, privzeto 1, ali trajanjem, privzeto 10 s) /**Interval** (časovni preskok: N zajemov vsak X interval za skupno Y, privzeto 1 vsakih 5 s za 1 minuto).
* **Vrste izvoza po kameri**: Raw, Debayered, Radiance, Reflectance, Preview, Index — vse, kar je primerno, je privzeto vklopljeno. Radiance/Reflectance sta skrita za kamere s filtrom RGB;**Reflectance se prikaže le, če ima kamera svetlobni senzor DAQ** (lastnega ali podedovanega iz matrike); za Index je potreben konfiguriran indeksni izraz.
* **Poravnano**(po matrici, privzeto**vklopljeno**): izvoze posameznih elementov prilagodi profilu poravnave matrike, tako da so izvozi registrirani na ravni pikslov. Raw vedno ostane neporavnan, vendar nosi transformacijo v metapodatkih.
* **Najhitrejše zajemanje** (preklop): samo surovi podatki + dodeljena vrednost DAQ + brezplačna sestavljena vrednost kombiniranega indeksa, pri čemer se ob zajemanju preskoči kalibracijski izračun za doseganje največje hitrosti — sevanje/odbojnost/indeks se kasneje ponovno izračunajo iz shranjenega `.daq`.
* Izbire ostanejo v projektu. Skrite ali zaustavljene kamere se preskočijo.

Enakovredna CLI (ista končna točka backenda, ista semantika):

```bash
# One synced group, every applicable export level per camera (the default)
chloros-cli lattice array-capture -o output/

# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# 30-second monitoring clip of the combined index view, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# 5-second analysis-grade raw burst, then build the combined index video
chloros-cli lattice array-burst --duration 5 --build --products combined:index --fps 10 -o capture/
```

Stiskanje za zajeme v TIFF je `deflate` (brez izgube, privzeto) ali `none` — popolne tabele zastavic, razpored mape zajemov in pravila ponovne obdelave so v [CLI Referenca](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Povezovanje svetlobnega senzorja DAQ

Za predoglede z odbojnostjo in osvetlitvijo popravljenimi so potrebni podatki o navzdol usmerjeni svetlobi iz senzorja DAQ (priključenega na zavihku **Svetlobni senzorji**):

* V **vrstici niza**na stranskem pasu se prikaže**gumb »DAQ · vklop/izklop«** — *vklopljen*, ko je nastavljen svetlobni senzor na ravni niza **ali** ima katera koli kamera v nizu svojega; v njegovem opisu je natančno navedeno, kateri senzor napaja katero kamero.
* Nastavite za celoten niz v nastavitvah niza → **Senzor okoljske svetlobe**→ spustni meni**Svetlobni senzor**. Izbira ostane shranjena v projektu, se prenese na vsako kamero v nizu, posamezne kamere pa jo lahko še vedno prepisujejo s svojim senzorjem.
* Statusna vrstica pod njo prikazuje trenutno stanje: **Izklopljeno**→ »Čakam na prvi spekter…« →**»Aktivno — osvetlitev vseh kamer v nizu je popravljena«** → ali, če v zadnjih 3 s ni prispel nov spekter, opozorilo o zastarelem spektru — še naprej se uporablja zadnja vrednost (vrednosti na poti zajema nikoli ne potečejo).

Če je senzor dodeljen: postane na voljo tip izvoza »Reflectance«, predogledi v realnem času so korekcijirani glede na osvetlitev, predvidljiva samodejna osvetlitev lahko uporabi spektrum, vsak posnetek odbojnosti pa zapiše dejansko uporabljeno vrednost DAQ kot **`.daq` sidecar** zraven slike, tako da se zajem lahko kasneje ponovno obdela.

## Možnosti `array-connect` CLI

| Oznaka | Privzeto | Opis |
| --- | --- | --- |
| `--serials SN1,SN2,…` | samodejno odkrivanje vseh kamer LATTICE (potrebnih je ≥2) | **Prva serijska številka je MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | Sinhronizacijska linija GPIO. |
| `--target-fps F` | auto | Frekvenca sprožanja glavne kamere. |
| `--binning {1,2,4}` | avto | Strojno združevanje slik. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | avto | Strokovno prepisovanje izbire sinhronizacijskega nivoja. |
| `--wire-ceiling-mbps MB_PER_S` | samodejno zaznano | Proračun pasovne širine gostitelja v MB/s — oblika polja **Wire Budget** v CLI. Zmanjšajte ga, če matrika poroča o poškodovanih okvirjih. Shranjeno skupaj s projektom, zato se ob poznejši ponovni povezavi ponovno vzpostavi. |
| `--no-recommend` | izklopljeno | Preskoči korak analize omrežja. |
| `--no-ptp` | izklopljeno | Onemogoči PTP (časovni žigi med kamerami nato niso primerljivi). |

`lattice array-list`, `array-status`, in `array-disconnect` upravljajo trajno sejo. Celoten pregled podukazov, vključno z usklajevanjem (`align-calibrate` / `align-apply`) in omrežnih orodij, je v [CLI Reference § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); ekvivalenti SDK (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) so v [SDK Referenca](../reference/sdk-reference.md). Od Python naprej je proračun za povezave `connect_array(..., wire_ceiling_mbps=120)`, razdelitev med aktivnimi, poškodovanimi in tistimi, ki niso nikoli prispeli, pa je v [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).

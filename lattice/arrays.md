# Mreže z več kamerami

**Mreža**LATTICE je sestavljena iz dveh ali več kamer LATTICE, povezanih v eno sinhronizirano enoto. Ena kamera je**glavna**: ta po skupni sinhronizacijski liniji (privzeto**Line2**) pošlje strojni GPIO sprožilni impulz, tako da vsaka kamera posname isti trenutek. Chloros dodaja časovno sinhronizacijo PTP, predogled v živo (posamezne ploščice po kamerah ali en sam poravnan večpasovni kompozit) ter sinhronizirano zajemanje — vsak zajemni cikel ustvari eno **skupino sličic**, v kateri imajo vse kamere enak časovni žig in ID sličice (na izhodu zajema prikazano kot `fid:N`).

Mono (M3M) kamere ustvarjajo indekse vegetacije s pomočjo matrik – ena kamera prispeva en pas, matrika pa jih poravna v večpasovni niz. Glej [Mono kamere in vegetacijski indeksi](mono-indices.md).

Obstajajo trije enakovredni načini za povezavo niza, vsi pa izvajajo isti potek »smart-prep«:

| Vmesnik | Vstopna točka |
| --- | --- |
| GUI | Zavihek »Kamere« → **Poveži niz** (modri gumb) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (prva serijska številka = glavna) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (prva serijska številka = glavna) |

Smart-prep izvede: preverjanje zmogljivosti omrežja (ICMP DF ping + GVSP preverjanje), izbiro sinhronizacijskega nivoja, projekcijo hitrosti snemanja pri zahtevani velikosti okvirja, odločitev o pakiranem 12-bitnem prenosu (glej [Pakirani 12-bitni prenos](#packed-12-bit-transport)), omogočanje PTP, samodejno izbiranje formata pikslov za vsako kamero posebej, začetno nastavitev samodejne osvetlitve na podlagi shranjenega stanja vsake kamere ter konfiguracijo sprožilca GPIO na Line2.

**Privzeto se uporablja celoten senzor (novost v različici 1.3.0).** Connect ohranja velikost slike, ki ste jo zahtevali — celoten senzor 2048×1536, brez združevanja pik in brez izreza, razen če se ne odločite drugače — ter dopušča, da se hitrost snemanja prilagodi zmogljivostim gostitelja in omrežja. Nikoli samodejno ne združuje pikslov ali izreže slike (v različici 1.2.0 se je združevanje pikslov samodejno sprožilo pri hitrosti pod 1,5 fps). Odločitev o združevanju ali manjšem območju zanimanja (ROI) je vaša: v pogovornem oknu »Povezava« izberite eno od možnosti, v »Predvidenem izidu« pa se prikaže hitrost snemanja, ki jo ta možnost omogoča; `array-connect` izpiše hitrejše možnosti z njihovimi predvidenimi hitrostmi.

{% hint style="info" %}
Kamere morajo biti dosegljive prek povezave, preden bo karkoli od tega delovalo — glejte [Povezovanje kamer](connecting.md) za odkrivanje, naslavljanje in prenos kalibracijskih podatkov ob prvi povezavi. Pri sistemih z več kamerami so nastavitve sprejemnega obroča gostiteljske mrežne kartice enako pomembne kot hitrost povezave; popolna tabela simptomov in rešitev je v [Referenca CLI § Nastavitev in optimizacija gostiteljske mrežne kartice](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Pogovorno okno »Povezava niza«

Zavihek »Kamere« → **Poveži niz**odpre čarovnika v treh korakih:**Izberi → Način prikaza → Nastavitve**.

### Korak 1 — Izberite glavno in podrejene kamere

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered (an FRGB among them). Table showing Camera / Serial / IP / Master radio / Slave checkbox columns in camera order (M3C first, FRGB at the top), with the FRGB pre-selected as master and the green "Sync cable found on N cameras. Any of them can be master; the RGB camera (else the first) is pre-selected." probe banner visible above the table. -->



Pogovorno okno takoj po odprtju pregleda omrežje („Pregledovanje omrežja...“), nato pa preveri sinhronizacijske kable („Preverjanje sinhronizacijskih kablov...“), da ugotovi, katere kamere so priključene na sinhronizacijsko linijo M8. Za vzpostavitev niza potrebujete vsaj **2 kamere**.

Sinhronizacijski kabli M8 se združujejo na enem križišču, zato **lahko katerakoli priključena kamera deluje kot glavna**. Preizkus vnaprej določi vloge — kamera RGB (`-FRGB`) je glavna, če je priključena, sicer pa prva priključena kamera, ostale priključene kamere pa so podrejene — in prikaže enega od treh napisov:

| Napis | Pomen |
| --- | --- |
| »Sinhronizacijski kabel najden na {n} kamerah. Katera koli od njih je lahko glavna; vnaprej je izbrana RGB-kamera (sicer prva).« (zelena) | Sonda je našla kamere na sinhronizacijski liniji; potrditvena polja za glavno in podrejeno kamero sta že izpolnjena. |
| »Glavna kamera ni zaznana – preverite GPIO kabel« (oranžna) | Nobena kamera ni zaznala sprožilnega impulza; preverite sinhronizacijsko ožičenje. Vloge so še vedno vnaprej izbrane (RGB-kamera, sicer pa prva kamera na seznamu, kot glavna) in jih lahko spremenite. |
| »Ni sinhronizacijskega kabla: {serijske številke}« (oranžno) | Naštete kamere nimajo priključenega sinhronizacijskega kabla. |

Tabela kamer ima stolpce **Kamera / Serijska številka / IP / Glavna (radio) / Podrejena (polje za potrditev)**. Vrstice so navedene v**vrstnem redu kamer** — kamere M3C (Bayer) pred kamerami M3M (mono); med kamerami M3C, FRGB, FRGN, FOCN, nato FNGB; kamere M3M pa po valovni dolžini filtra, najprej najnižja (F450 pred F808). Seznam kamer v stranskem meniju in privzeta razporeditev v mreži uporabljata isti vrstni red.

* Izberite natanko **enega glavnega**in**enega ali več podrejenih**. Ponovni klik na gumb trenutnega glavnega kamere ga izbriše. Izbira drugega glavnega kamere je mogoča z enim klikom: novi glavni kamere zapusti podrejene kamere, prejšnji glavni kamere pa postane podrejena (razen če nima sinhronizacijskega kabla).
* Kamere z oznako **»Brez sinhronizacijskega kabla«** ni mogoče izbrati kot podrejeno kamero — podrejena kamera brez sprožilnega ožičenja bi namreč neskončno dolgo čakala na sinhronizacijski vod in oddajala neuporaben signal. To kamero raje priključite kot samostojno kamero.
* Kamere, ki so že priključene kot samostojne, *niso* onemogočene: povezava v niz sprosti samostojno sejo in ponovno odpre kamero znotraj niza.

**Naprej: Način prikaza →**se omogoči, ko sta izbrana glavna kamera in vsaj ena podrejena kamera.**Ponovno skeniranje** ponovno zažene iskanje in preverjanje ožičenja.

{% hint style="warning" %}
**Prekliči** je onemogočen, medtem ko poteka skeniranje ali preverjanje ožičenja – preklic med preverjanjem ožičenja lahko povzroči sesutje kamerSDK

-ja na firmware-u kamere LATTICE. Počakajte, da se vrteči indikator ustavi.
{% endhint %}

### Korak 2 — Način prikaza



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

| Način | Kaj dobite |
| --- | --- |
| **Ločene kamere** | Ena ploščica v živo na kamero, vse se sprožijo skupaj, tako da ostanejo posnetki sinhronizirani. Vsaka kamera ohrani svojo barvo in nastavitve. |
| **Združene kamere** *(privzeto)* | En sam ploščica, ki prikazuje usklajeno večpasovno NDVI/indeksno kompozitno sliko. Kamere si delijo barvo matrike. |

Način prikazovanja spreminja le predstavitev v živo — način zajemanja je v obeh primerih enak.

### Korak 3 — Nastavitve matrike in predvideni izid

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Trigger Rate / Wire Budget controls (1.3.0 has no "Pin resolution" checkbox), right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->



Ob vstopu v to sceno Chloros zaprosi strežnik za **priporočilo**in ga uporabi:**polni senzor – brez združevanja pik (binning), brez izreza ROI – kadarkoli to omogoča kakršna koli hitrost snemanja**, ne glede na hitrost, ki jo zagotavljata gostitelj in povezava. Šele kadar nobena hitrost ne omogoča polnega senzorja, priporočilo pomanjša sliko, pri čemer daje prednost združevanju pik pred izrezovanjem ROI, saj združevanje pik ohranja celotno vidno polje. Od tu naprej izbirate med ločljivostjo in hitrostjo snemanja. Vsaka sprememba, ki jo naredite, ponovno izvede analizo v živo in posodobi desni panel**Predvideni izid**.

Levi stolpec — nastavitve:

| Nadzor | Možnosti | Privzeto | Opombe |
| --- | --- | --- | --- |
| **ROI (vidno polje)** | Celotno (2048×1536) / Polovično (1024×768) / Četrtinsko (512×384) | Celotno | Izrez senzorja: polovični/četrtinski izrez na manjše območje z izvirno gostoto pikslov. |
| **Združevanje pik** | 1× / 2× (vsota 2×2) / 4× (vsota 4×4) | 1× | Strojno združevanje pik: 2×2 = polno FoV pri četrtini stroškov prenosa po vodniku; 4×4 = polno FoV pri 1/16. Skrito, če kamere ne podpirajo združevanja pik. |
| **Slika na vodniku** (odčitavanje) | — | — | Širina × višina po združevanju pik, ki se dejansko pošilja po vodniku, zaokrožena na večkratnike števila 16 (najmanj 64). |
| **Hitrost sprožanja** | 0,5–60 fps, korak 0,1 | prazno = avto | Hitrost sprožanja glavne kamere. Pustite prazno, da jo Chloros izračuna sam. |
| **Zmogljivost omrežja**| 20–2000 MB/s, korak 10 | prazno = samodejno | Koliko lahko gostitelj dejansko sprejme, v MB/s —**edina številka, od katere je odvisna celotna dodelitev polja.** Samodejno zaznana iz omrežnega adapterja. Znižajte jo, če polje poroča o poškodovanih okvirjih: zaznana vrednost precenjuje zmogljivost USB-adapterjev in skupnih stikal. Sprememba te vrednosti ponovno zažene projekcijo v realnem času. Polje se odpre s hitrostjo, s katero je bil priključen zadnji niz (shranjeno skupaj s projektom), tako da se hitrost, ki ste jo enkrat popravili, ohrani. |

Potrditveno polje **Ločljivost pinov** iz različice 1.2.0 je odstranjeno: povezava vedno ohrani velikost okvirja, ki jo izberete, in namesto tega zniža hitrost okvirjev.

Desni stolpec — **Predvideni izid**:

* **Raven sinhronizacije** — »Hkratno zajemanje« (zeleno), »Hkratno zajemanje (FTD-postopno oddajanje)« (zeleno), »Postopno zajemanje (100 ms zamik)« (rumeno) ali »Konfiguracija je prevelika« (rdeča). Chloros izbere stopnjo: sočasno zajemanje, kadar je mogoče deliti povezavo s hitrostjo, ki ji lahko sledijo kamere (hitrost snemanja se zmanjša, ko se delež posamezne kamere zmanjša), in postopno zajemanje le, kadar to ni mogoče — tako se niz poveže v katerem koli omrežju s hitrostjo snemanja, ki jo omrežje omogoča.
* **napoved fps** — hitrost snemanja, pri kateri naj bi niz deloval. Pri sočasnem zajemanju gre za eno število: najprej je določena hitrost, avtomatska osvetlitev pa zapolni časovno obdobje, ki ga ta določa, nato pa poveča ojačanje, tako da jo temna scena ne zniža (glej [Kako avtomatska osvetlitev porablja čas osvetlitve in ojačanje](camera-settings.md)). Pri časovno zamaknjenem zajemanju je prikazano kot razpon („temno → svetlo“), saj vsaka kamera v temni sceni podaljša zamik.
* **NIC linija** — hitrost povezave in trajna prepustnost („NIC {mbps} Mbps · trajna {N} MB/s“).
* **Preverjanje sočasnega oddajanja** — ali lahko sprejemni obroč omrežne kartice gostitelja sprejme en sočasen izbruh podatkov iz vseh kamer („Sočasno oddajanje: X MB · uporaben obroč omrežne kartice: Y MB ✓/✗“).
* **Preverjanje zmogljivosti omrežja** — skupno povpraševanje v stabilnem stanju v primerjavi z zgornjo mejo omrežja, varno pred kolizijami („Zmogljivost omrežja: {povpraševanje} MB/s, ki ga zahteva {n} kamer · zgornja meja {ceiling} MB/s ✓/✗ prekoračena“).
* **»Največje število kamer na tem omrežnem vodniku: {n} — določeno z minimalno pasovno širino na kamero, zato ga združevanje v skupine ne poveča.«** — prikaže se, ko se približujete (ali presegate) zgornjo mejo števila kamer.
* **»Pri teh nastavitvah BO PRIHODILO DO IZGUBLJANJA SLIK.“**— rdeče opozorilo z razlogom strežnika, skupaj s seznamom ovir in modrimi**predlogi za rešitev** („Da bi ta niz ustrezal omrežju“ / „Da bi omogočili sočasno snemanje“).**Uporabi in poveži** je onemogočeno, dokler ni napovedi, njegova oznaka pa vam pove, zakaj je zavrnjeno:

| Oznaka gumba | Pomen | Kaj dejansko pomaga |
| --- | --- | --- |
| »Analizira se...« | Analiza še poteka. | Počakajte. |
| **»Preveč kamer za to omrežje“**| Omrežje ne more prenesti niza niti ene kamere naenkrat (preverjanje agregata ni uspelo): veliko več kamer, kot jih lahko obdela, ali druge kamere, ki po njem prenašajo podatke. | Manj kamer, jumbo okvirji od začetka do konca ali hitrejša mrežna kartica.**Manjši ROI NE bo pomagal** — glej spodaj. |
| **»Zmanjšajte ROI, da omogočite«** | Pri teh nastavitvah bi prihajalo do izgube okvirjev (preverjanje burst/ring ni uspelo). | Zmanjšajte ROI, povečajte binning ali popravite sprejemni ring mrežne kartice. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->



Med povezovanjem se lahko prikaže zeleni **paneel za prenos kalibracije** z napredovalno vrstico za vsako serijsko številko: ko se kamera prvič poveže z napravo, Chloros prek GigE prenese njen tovarniški kalibracijski paket velikosti ~3,8 MB (približno 70 sekund na kamero). Kamere, ki so shranjene v predpomnilniku, tega okna nikoli ne prikažejo; pri več kot šestih kamerah se vrstice pomikajo. Glej [Povezovanje kamer](connecting.md).

### Pakirani 12-bitni prenos

12-bitni formati (`BayerRG12`, `Mono12`) se običajno prenašajo kot 16-bitne besede. **Pakirani**prenos pošilja istih 12 bitov v 1,5 bajta na piksel in jih razpakira na gostitelju, tako da pošiljanje vsakega okvirja traja 25 % manj časa — pri polni zmogljivosti senzorja to pomeni približno tretjino več okvirjev na sekundo.**Shranjeni podatki so identični.** Array Connect to odloči samodejno (`auto`): pakira le, kadar je to koristno in ima gostitelj na voljo dovolj procesorske moči za razpakiranje. V grafičnem vmesniku (GUI) za to ni stikala; v CLI-ju uporabite `array-connect --packed {auto,on,off}` (privzeto `auto`). 8-bitni in 16-bitni formati se nikoli ne pakirajo.

## Pasovna širina: koliko kamer se lahko priključi

Koliko lahko niz prenese, je odvisno od omrežne povezave, ne od Chloros a, zato so podatki za načrtovanje navedeni v priročniku za strojno opremo: **[Načrtovanje pasovne širine niza](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Kaj s temi podatki stori Chloros: pogovorno okno za povezavo izvede preizkus omrežja, napove dosegljivo hitrost snemanja in izbere ustrezno stopnjo. Niz se poveže v katerem koli omrežju: na počasni povezavi se vsaka kamera prilagodi svojemu deležu pasovne širine, hitrost snemanja pa se zmanjša, namesto da bi bil niz zavrnjen ali da bi se njegovi okvirji skrčili. Zavrnjena je le skupina, pri kateri povezava ne more prenesti niti ene kamere naenkrat (**Preveč kamer za to omrežje**), namesto da bi tiho zavračala pakete — glejte zgoraj opisani panel s predvidenimi rezultati.

## Ko se okvirji izgubijo

Kamera lahko manjka v objavljeni skupini iz dveh popolnoma različnih razlogov,
ki zahtevata nasprotna popravka. Chloros jih šteje ločeno, namesto da bi poročal o eni
»nepopolni« številki, ki ne navaja nobenega od razlogov:

| Kaj se je zgodilo | Kaj to pomeni | Kje iskati |
| --- | --- | --- |
| **Poškodovan**— okvir je prispel, a je bil strukturno poškodovan | Izguba paketov GVSP na omrežni poti |**Zmogljivost povezave**, sprejemni obroč mrežne kartice, jumbo okvirji, stikalo |
| **Nikoli ni prispel**— okvir sploh ni prišel | Kamera ni sprožila snemanja ali pa iz nje ni prišlo nič |**Sinhronizacijski kabel M8**, sinhronizacijska linija, ali so vsi člani vklopljeni |

Razdelitev se ponovno ovrednoti vsakih 10 sekund, medtem ko niz prenaša podatke. Nad 5 % se
zabeleži z navedbo obeh številk, vsak poškodovan pomnilnik pa se poroča ob prvem
pojavljanju na posamezni kameri, nato pa se podatki združijo enkrat na minuto, da ostane dolgotrajna seja berljiva.

**Poškodovani okvirji z vrednostjo nič pri »nikoli prispelih« pomenijo, da sta sprožitev in sinhronizacija kabla brezhibni**in da je vsak izgubljen okvir na omrežni poti. Rešitev je zmanjšanje**Wire Budget** in
ponovna povezava.

{% hint style="warning" %}
**Znižanje hitrosti sprožitve ne pomaga pri poškodovanih okvirjih.** Hitrost pošiljanja paketov kamere
se določi enkrat, ob vzpostavitvi povezave. Znižanje hitrosti sprožitve spremeni, kako pogosto se pojavi serija, ne pa, kako hitro se serija sama prenese po kablu. Na izmerjeni napravi s 4 kamerami
5-kratno zmanjšanje hitrosti sprožanja ni spremenilo ničesar, medtem ko je znižanje proračuna za prenos s 240 na
200 MB/s je pri isti napravi zmanjšalo delež poškodovanih okvirjev s 10,4 % na nič.
{% endhint %}

Delujoča mreža se ne more sama na novo načrtovati – prekinite povezavo in se ponovno povežite, da bo izbirnik časa povezave
lahko deloval v skladu z novim proračunom.

### USB-omrežni adapterji so omejeni

USB-Ethernet-adapter oglašuje svojo *Ethernet* hitrost povezave, vendar je tisto, kar lahko dejansko
vzdržuje, omejeno z USB-vodilom in njegovim gonilnikom. USB-ključ 10GbE je bil nekoč pripisan
pretočnosti približno 1000 MB/s — vrednost, ki je nikoli ni nihče izmeril — in sinhronizacija
štirih kamer s to fantomsko rezervo je povzročila poškodbo 6–18 % sličic, medtem ko je niz
še vedno poročal o ustrezni ciljni hitrosti sličic. Adapterji, priključeni prek USB, so zdaj omejeni na
**200 MB/s** v sistemu Windows in na platformi NVIDIA Jetson ter **400 MB/s** v sistemu Linux x86_64. Omejitev je absolutna in ne odstotna, saj je omejitev v
vmesniku: USB-adapter 1 GbE doseže približno 80 MB/s in ga to ne vpliva.

Če je vaš gostitelj dejansko hitrejši od omejitve, povečajte **Wire Budget**, da to odraža.

## Sinhronizacija časa PTP

*Sinhronizacija* okvirjev izhaja iz strojne sprožilne signale; **PTP** (IEEE 1588 PTPv2) zagotavlja primerljive *časovne oznake* na vseh napravah. Privzeto je omogočena ob priključitvi matrike:

* **Zadnji del gostitelja Chloros** deluje kot glavni PTP-vodja. Kamere LATTICE in svetlobni senzorji DAQ-E delujejo kot podrejeni v domeni 0, tako da so časovni žigi slik in spektri DAQ usklajeni z eno uro (~1 ms).
* `--no-ptp` (CLI) ga onemogoči za delo na delovni mizi — časovni žigi med kamerami takrat **niso** primerljivi.
* Preverite stanje sinhronizacije prek CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity, health (OK, or why its packets are not reaching the network)
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Zavihek »Kamere« sam po sebi nima indikatorja PTP; tam so na voljo le za branje podatki o sinhronizaciji za posamezno kamero: **Vloga**(Glavna/Podrejena),**Sinhronizacijska linija** in raven zmogljivosti niza. Stanje PTP za DAQ-E je prikazano v podrobnostih senzorja na zavihku »Svetlobni senzorji«.

## Pogled

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows in camera order (M3C before M3M, monos by wavelength), and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

na niz v realnem času Glavno območje prikaza ponuja dve razporeditvi (preklopite v zgornji vrstici): **pogled v obliki mreže**(vsaka ploščica je celica; preuredite z vlečenjem, ko je ključavnica mreže odklenjena) ter**pogled v obliki seznama**(vrstice v polni širini:**združena**matrica je ena vrstica – njena sestavljena ploščica –, medtem ko**ločena**matrica vsaki kameri dodeli**lastno vrstico**z lastnim živo prikazom in lastno tabelo kalibracijskih ciljev; pod nizi je ena samostojna kamera – aktivna). Drsnik**Povečava posnetka** spreminja velikost ploščic; pri širini celice pod 200 px se prekrivni napisi z imenom/fps samodejno skrijejo. Kamere v nizu se prikažejo v stranskem pasu in v privzeti razporeditvi mreže v vrstnem redu kamer (M3C pred M3M, najprej FRGB, monokromatske kamere po valovni dolžini); odkleneš mrežo in s povlekanjem še vedno prevladaš nad vrstnim redom v mreži.

<!-- SCREENSHOT-NEEDED: list view of a connected SEPARATE array (1.3.0): one full-width row per member camera, each with its live feed and its own Calibration Target table below it. -->



**Ločen način** prikazuje eno ploščico na kamero. Vsaka ploščica prikazuje:

* ime kamere (zgoraj levo),
* **prikaz sličic na sekundo** (zgoraj desno) — to je *dejanska hitrost zajemanja* kamere, ki jo poroča strežnik, ne pa hitrost osveževanja predogleda (predogled v živo je omejen na 30 sličic na sekundo ne glede na hitrost zajemanja),
* statusno piko — zelena (pretakanje) / rumena (nalaganje) / rdeča (napaka),
* **vrtečo ikono zastarelega okvirja**, če v 2 sekundah ni prispel noben nov okvir — to je normalno za približno 5 sekund po vsaki povezavi/prekinitvi, medtem ko backend ponovno uravnava porabo pasovne širine med kamerami.**Kombinirani način**prikazuje eno sestavljeno ploščico: strežnik izvede debayering, prilagodi velikost, poravna, odstrani šum, pretvori v sevanje po pasovih (plus odbojnost DLS, če je vezan svetlobni senzor), izračuna indeksni izraz matrike, uporabi LUT in rezultat prenese kot MJPEG. Dokler se ne prikaže prvi poravnan okvir, ploščica prikazuje svoje stanje: »Priprava matrike…«, »Kalibriranje poravnave…«, »Čakam na prvi okvir…« ali – če se iztečejo poskusi samodejne poravnave ploščice (približno ena minuta) – »Potrebna poravnava« z gumbom**Kalibriraj poravnavo**.

Prikaz fps ploščice matrike prav tako poroča o stanju prenosa:**»Uporabljam nastavitve…«**, medtem ko sprememba nastavitev (format, združevanje, ROI) ponovno zažene kamere,**»Zastalo«**, ko je matrika aktivna, vendar je prenehala pošiljati skupine okvirjev, in**»Ustavljeno“**, ko je prenehal s prenosom – v tem primeru ponovno povežite matriko. Kadar je spektralno ločevanje moralo omejiti pas, prikazan v sestavljeni sliki, na velikem deležu njegovih pikslov (25 % ali več), ploščica prikaže**»Podatki pasu z nizko zanesljivostjo · N %“**: indeksne vrednosti, izračunane na podlagi tega pasu, so sumljive (ponavadi je en pas prejel premalo svetlobe).

Koristna dejstva o kombiniranem načinu:

* Sestavljena slika je poravnana s sličico **glavne**kamere. Usmerjanje AE-ROI in točkovno merjenje osvetlitve na sestavljeni sliki sta natančna za glavno kamero in približna za podrejene kamere; uporabite**Split View** (nastavitve niza → »Pokaži kamere v nizu«) za ploščice, natančne do posameznega pikslja za vsako kamero, brez odpiranja dodatnih povezav s kamerami.
* **Display Layers** (nastavitve matrike; privzeto izklopljeno) vam omogoča izbiro plasti v ospredju in ozadju — katero koli kamero v sistemu ali**Index**. Če je v ospredju izbran**Index**, se v pikslah zunaj LUT Min/Max prikaže plast v ozadju.
* **Ločljivost upodabljanja** (privzeto 720p) nastavi višino živega prenosa *in* velikost izvoza shranjene kompozicije. Slike posameznih kamer se vedno izvozijo v polni ločljivosti.
* Poravnava se izračuna za vsako sejo in se nikoli ne shrani. Chloros **vsako matrico samodejno poravna po povezavi, v obeh načinih prikazovanja**, ko se samodejna osvetlitev članov ustalijo (poskuša do 5 minut). V razdelku**Poravnava**v oknu nastavitev niza so prikazani RMS-ostanki in gumb**Ponovna kalibracija poravnave** — tudi za posamezne nize, ki nimajo sestavljene ploščice.

## Snemanje: spremljanje proti analizi

Površine zajema niza se jasno delijo na **stopnjo spremljanja**(zajemanje tistega, kar vidite) in**stopnjo analize** (zajemanje surovih podatkov, kalibracija kasneje):

| Delovni tok | Stopnja | Kaj shrani | GUI | CLI |
| --- | --- | --- | --- | --- |
| **Zajem**(posnetki) | Analiza | Ena sinhronizirana skupina sličic na prehod; datoteke po kameri na vsaki izbrani ravni izvoza (surove/brez debayeringa/sijaj/odbojnost/predogled/indeks) + `.daq` sidecar |**Gumb **Zajemi vse** + Nastavitve zajema | `lattice array-capture` |
| **Snemanje indeksnega videa** | Spremljanje | Prikazani združen indeksni kompozit v živo — 8-bitni, ločljivost predogleda, vgrajena LUT. Iz CLI-ja brez odprtega grafičnega vmesnika (GUI) backend renderira NDVI (ali `--expr`) matrike, ko je ta poravnana; snemanje, ki ne zapiše nobenega okvirja, se ne uspe | ● Snemanje indeksnega videa (kombinirani nizi) | `lattice array-record` |
| **Surovi niz posnetkov → sestavi video**| Analiza | Surovi okvirji senzorja pri polni hitrosti zajemanja + manifest + `.daq`, nato pa offline rekonstrukcija v kalibrirani video s svetilnostjo / odbojnostjo / indeksom, časovno usklajen z odčitki DAQ | ⦿ Snemanje surovega niza posnetkov →**Sestavi video** | `lattice array-burst` → `lattice array-build-video` |

Pravilo: če bodo piksli služili za *meritve*, uporabite zajem ali serijo posnetkov (za analizo); če pa želite le *pregledati ali prikazati*, kaj je zaznal senzor, posnemite indeksni video (za spremljanje).

### Nastavitve zajema (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->



Zobnik poleg **Capture All** odpre okno z nastavitvami zajema (zahteva odprt projekt — zajeti podatki se shranijo vanj):

* **Način zajema**:**Enkratni**(en prehod) /**Neprekinjen**(zaporedno; omejen s številom zajemov, privzeto 1, ali trajanjem, privzeto 10 s) /**Interval** (časovni preskok: N zajemov vsak X interval za skupno Y, privzeto 1 vsakih 5 s v 1 minuti).
* **Vrste izvoza po kameri**: Raw, Debayered, Radiance, Reflectance, Preview, Index — vse, kar je primerno, je privzeto vklopljeno. Radiance/Reflectance sta skrita za kamere z RGB-filtrom; **Reflectance se prikaže le, če ima kamera svetlobni senzor DAQ** (lastnega ali podedovanega iz niza); Index zahteva konfiguriran indeksni izraz.
* **Poravnano**(na niz, privzeto**vklopljeno**): prilagodi izvoze posameznih datotek profilu poravnave niza, tako da so izvozi registrirani na ravni pikslov. Surovi podatki ostanejo vedno neizkrivljeni, vendar nosijo transformacijo v metapodatkih. Datoteke članov niza se shranijo v izvorni orientaciji vsake kamere (vsaka nastavitev zrcaljenja/vrtenja se zabeleži v datoteki, ne pa uporabi).
* **Najhitrejše zajemanje** (preklop): samo surovi podatki + dodeljena vrednost DAQ + brezplačna sestavljena slika s kombiniranim indeksom, pri čemer se ob zajemanju preskoči kalibracijski izračun za doseganje največje hitrosti — svetilnost/odbojnost/indeks se kasneje ponovno izračunajo iz shranjenega `.daq`.
* Izbire ostanejo shranjene v projektu. Skrite ali zaustavljene kamere se preskočijo.
* **Neprekinjeno** na nizih teče znotraj ozadja Chloros (novost v 1.3.0): ko vsaka kamera na seznamu pripada nizu, ozadje samo izvaja zanko zajemanja, piše eno skupino, medtem ko izbira naslednjo, brez zahtevka za vsak posnetek. Poln pogon ustavi izvajanje („Ustavljeno: pogon poln. Shranjeno N, F ni uspelo“). Glejte [Nastavitve in načini zajemanja](capture.md#capture-modes) za opombo Microsoft Defenderja o visokih hitrostih zajemanja.

Enakovreden CLI (ista končna točka ozadja, ista semantika):

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

TIFF-stiskanje za zajeme je `deflate` (brez izgube, privzeto) ali `none` — popolne tabele zastavic, razpored mape za zajeme in pravila ponovne obdelave so v [Referenca CLI](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Povezovanje svetlobnega senzorja DAQ

Za predoglede z odbojnostjo in osvetlitvijo, popravljene s senzorjem DAQ (priključenim v zavihku **Svetlobni senzorji**), so potrebni podatki o navzdol usmerjeni svetlobi:

* V **vrstici niza**na stranskem pasu se prikaže**gumb »DAQ · vklop/izklop«** — *vklopljen* je, kadar je nastavljen svetlobni senzor na ravni niza **ali** kadar ima katera koli kamera v nizu svojega; v njegovem opisu je natančno navedeno, kateri senzor napaja katero kamero.
* Nastavite za celoten niz v nastavitvah niza → **Senzor okoljske svetlobe**→ spustni meni**Svetlobni senzor**. Izbira ostane shranjena v projektu, prenese se na vsako kamero v nizu, posamezne kamere pa jo lahko še vedno prepisujejo s svojim senzorjem.
* Statusna vrstica pod njo prikazuje trenutno stanje: **Izklopljeno**→ »Čakam na prvi spekter…« →**»Aktivno — osvetlitev vseh kamer v nizu je popravljena«** → ali, če v zadnjih 3 sekundah ni prispel nov spekter, obvestilo o zastarelem spektru — še naprej se uporablja zadnja meritev (meritve na poti zajemanja nikoli ne potečejo).

Če je senzor dodeljen: postane na voljo tip izvoza »Reflectance«, predogledi v realnem času so korekcijo osvetlitve, predvidljiva samodejna osvetlitev lahko uporabi spektrum, vsak zajem odbojnosti pa zapiše dejansko uporabljeno vrednost DAQ kot **datoteko `.daq`** v mapo zajemov `daq/`, tako da se zajem lahko kasneje ponovno obdela. Poleg tega datoteka `.daq.cameras.json` navaja kamere, katerih posnetke ta odčitek opisuje; pri ponovni obdelavi program Chloros uskladi odčitek le s slikami iz teh kamer.

## `array-connect` možnosti CLI



| Oznaka | Privzeto | Opis |
| --- | --- | --- |
| `--serials SN1,SN2,…` | samodejno odkrivanje vseh kamer LATTICE (potrebnih je ≥2) | **Prva serijska številka je MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | Sinhronizacijska linija GPIO. |
| `--target-fps F` | auto | Hitrost sprožanja glavne kamere. |
| `--binning {1,2,4}` | auto | Strojno združevanje pikslov. Nastavitev »auto« pomeni uporabo celotnega senzorja (brez združevanja), kadar koli to omogoča hitrost snemanja; nastavitev »connect never« nikoli ne združuje pikslov, da se ohrani visoka hitrost. |
| `--packed {auto,on,off}` | `auto` | [Pakiran 12-bitni prenos](#packed-12-bit-transport): `auto` pakira le, kadar je to koristno in kadar si gostitelj lahko privošči razpakiranje. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Strokovno prepisovanje izbire sinhronizacijskega nivoja. |
| `--wire-ceiling-mbps MB_PER_S` | samodejno zaznano | Proračun pasovne širine gostitelja v MB/s — oblika polja **Wire Budget** v CLI. Zmanjšajte ga, če niz poroča o poškodovanih okvirjih. Shrani se skupaj s projektom, zato se ob poznejši ponovni povezavi ponovno vzpostavi. |
| `--no-recommend` | izklopljeno | Preskoči korak analize omrežja. |
| `--no-ptp` | izklopljeno | Onemogoči PTP (časovni žigi med kamerami nato niso primerljivi). |

`lattice array-list`, `array-status` in `array-disconnect` upravljajo trajno sejo. Celoten pregled podukazov, vključno z usklajevanjem (`align-calibrate` / `align-apply`) in omrežnimi orodji, je v [Pregledu CLI-ja § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); ustrezni ukazi v SDK-ju (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) so v [Referenca SDK](../reference/sdk-reference.md). V jeziku Python je proračun za prenos `connect_array(..., wire_ceiling_mbps=120)`, razdelitev med poškodovanimi in nikoli prispelimi sporočili v realnem času pa je na [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).

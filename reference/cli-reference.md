# Referenca za Chloros CLI

**Različica:**

1.3.0**Ustvarjeno:**

29. 7. 2026 ob 19:19 ·**Revidirano:**

6. 11. 2026**Ciljna publika:** Optimizirano za uporabo v sistemih za obdelavo velikih jezikovnih modelov (LLM); berljivo za ljudi.**Obseg:** Vsi podukazi programa `chloros-cli`, namenjeni uporabnikom, skupaj z možnostmi in primeri, ki jih je mogoče kopirati in prilepiti.

Ta dokument je popoln priročnik za orodje za ukazno vrstico `chloros-cli`, ki je priloženo programu MAPIR Chloros. Namenoma je izčrpen, da lahko LLM (ali človek) sestavi kateri koli podprt delovni tok iz spodnjih seznamov, ne da bi pregledoval izvorno kodo.

Če potrebujete le najpomembnejše informacije, preskočite na:
- [Petminutni hitri začetek](#five-minute-quickstart)
- [Delovni tok prve povezave kamere LATTICE](#lattice-camera-first-connect-workflow)
- [Delovni tok prve povezave senzorja DAQ](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Načini zajemanja, snemalniki in ponovna obdelava v načinu brez povezave](#capture-modes-recorders--offline-reprocess)

> **Upravljate Chlorosa prek AI pomočnika?** Claude Desktop, Claude Code, Cursor in drugi MCP-odjemalci lahko upravljajo obdelavo, kamere in matrike LATTICE ter svetlobne senzorje DAQ prek `chloros-mcp`, MCP-strežnik, ki se namesti skupaj s Python SDK (in z Linux .deb, v `/usr/bin`): `claude mcp add chloros -- chloros-mcp`. Nastavitve, možnosti in orodja so v referenčnem priročniku SDK pod [MCP-strežnik (AI-pomočniki)](sdk-reference.md#mcp-server-ai-assistants). Pomočnik s lupino lahko tudi neposredno izvede vsak ukaz na tej strani.

> **Spremembe v različici 1.3.0** (kar opazi obstoječi skript ali delovni tok iz različice 1.2.0).
>
> **Spremembe, ki jih bo morda treba upoštevati v skriptu iz različice 1.2.0:**
>
> - **Velja `--preset`.** `lattice capture`, `continuous`, `calibrate`, `multi-capture`, `multi-stream`, `align-calibrate`, `align-apply` in `align-stream` so bili v različici 1.2 odstranjeni.0 in deloval na podlagi lastnih nastavitev kamere. Zdaj nastavi format prednastavitve, osvetlitev, ojačanje in hitrost snemanja: `--preset high_speed` zajema 8-bitno sliko pri 5 ms, 6 dB, 60 sličic na sekundo. Tudi `lattice cam-connect --preset` ohranja format prednastavitve (v različici 1.2.0 je kamera po uporabi prednastavitve nastavila 12-bitno snemanje). Glej [Splošne možnosti](#common-options-shared-by-most-subcommands).
> - **Bayerjev format pikslov na kameri M3M (mono) je njena globina.** Prednastavitve navajajo Bayerjeve formate, ki jih mono senzor ne more zajeti: v različici 1.2.0 je kamera zavrnila ta format in ohranila tistega, ki ga je imela, zato `--preset high_speed` tam ni bil 8-bitni. Zdaj je to mono format iste globine: `high_speed` in `--pixel-format BayerRG8` dajeta `Mono8`, `high_quality`, `triggered` in `BayerRG12` pa dajeta `Mono12`. V mešanem nizu M3C + M3M vsaka kamera prevzame skupni format v svoji družini, pri čemer projekta `network-analysis` / `analyze-array --pixel-format` projicirata vsako kamero na tej globini. Enako velja tudi v nasprotni smeri: M3C, ki zahteva format Mono, prevzame Bayerjev format te globine, zato matrika, katere glavna kamera je M3M, svojih M3C kamer ne odpira več v načinu Mono. Glej [Mono (M3M) enopasovne kamere](#mono-m3m-single-band-cameras).
> - **Izvozi `lattice array-capture` so privzeto poravnani, tako kot izvozi aplikacije.** Vsi člani `debayered`, `display`, `radiance`, `reflectance` in indeksne datoteke so prilagojene okvirju glavne kamere in obrezane na prekrivanje članov (manjše od senzorja za odmike kamer, `XMP-mapir:AlignmentJSON` `"applied": true`), zato so datoteke, zajete z CLI, enake datotekam, ki jih aplikacija Chloros zajame za isto sceno; `raw` ostane v izvirni obliki. Prej je zajem brez `--aligned` shranil vsako datoteko v izvirni velikosti, neizkrivljeno. `--no-aligned` to ohranja. Enaka privzeta nastavitev velja za `--smart`, `--continuous` in `--interval` ter za SDK in `chloros-mcp`. Glej [načine delovanja `array-capture`](#array-capture-modes).
> - **`lattice array-capture --compression` je privzeto nastavljen na `none`.** Datoteke se zapisujejo nestisnjene (približno 6,3 MB na posnetek v polni ločljivosti) za najhitrejšo trajno hitrost; različica 1.2.0 je zapisovala v formatu `deflate` (približno 4,1 MB). Za manjše datoteke uporabite `--compression deflate`. Obe možnosti sta brez izgube.
> - **PNG in JPEG sprejemata le `display` / `preview` in indeksne izdelke.** Zajem `--format png` ali `jpg` z ravnjo, ki je za 8-bitno datoteko prevelika, se zavrne, še preden se kaj zajame (HTTP 400, izhod `1`), enako velja za privzeto `--processing all` z `--format png`/`jpg`. Glej [Ravni izvoza zajetih podatkov](#capture-export-levels-the-all-default).
> - **`lattice dls` potrebuje `--mac`.**Privzetega naslova MAC ni več (bil je naslov ene enote); `daq pool-discover --only ble` poišče naslov senzorja.**`lattice dls --camera SERIAL`** je nov.
> - **Več napak povzroči izhod iz `1`.** `process` se sproži, kadar ni mogoče preveriti izvozov izvajanja ali je izgubljena slika, čeprav so drugi izdelki zapisani; ukaz zajema s katerim koli datotekam, ki jih ni mogel zapisati (ostale datoteke se še vedno shranijo); `lattice continuous` in `multi-stream` ob polnem disku; `array-burst` / `array-record`, ko snemalnik poroča o napaki; `array-build-video`, ko je bil zahtevani izdelek preskočen ali ni bil izdelan v skladu z zahtevo; `project burst` / `stream`, ko zajem delno ne uspe; `project sensor log`, kadar posnetek ne vsebuje nobenega odčitka; `lattice exposure --set`, `gain --set` in `configure --set` z vrednostjo, ki jo kamera zavrne. `lattice network-analysis --json` se zaključi, ko se zaključi povzetek (`1` za `needs_force_slip`). Nasprotno: `daq pool-disconnect` senzorja, ki ni več v sklopu, zaključi `0`. Glej [Kode izhoda](#exit-codes).
> - **Počasen `lattice array-connect` nikoli ne pade, če je niz priključen.** Počaka do 600 s (180 s v različici 1.2.0; `project connect` 90 s), pri čemer sproti izpisuje korak strežnika, povezava, ki do takrat ni vzpostavljena, pa se prekliče s strani strežnika — vsaka kamera se vrne v stanje, v katerem je bila najdena — zato `Error:` (izhod `1`) pomeni, da ni priključen noben niz. V različici 1.2.0 je povezovanje takoj po vklopu kamer lahko trajalo dlje od čakalnega časa in povezalo niz šele po neuspešnem izvajanju ukaza. Ctrl+C zdaj prekine povezovanje. Glej [Trajne povezave](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-connect` ohranja celoten senzor.** Brez `--width` / `--height` / `--binning` je vsaka kamera nastavljena na polni senzor, hitrost snemanja pa pade na raven, ki jo omogočata ta računalnik in omrežje; izhod povezave navaja hitrejše možnosti (združevanje slik ali manjše območje zanimanja) s predvidenimi hitrostmi. Različica 1.2.0 je slike pomanjšala ali združila, da bi ohranila hitrost. Glej [Možnosti `array-connect`](#array-connect-options).
> - **Počasno omrežje ne zavrne več matrike.** Vsaka kamera se prilagodi svojemu deležu pasovne širine, s čimer se zmanjša tudi hitrost snemanja. Kadar kamer ni mogoče tako upočasniti, matrika deluje v načinu zaporednega oddajanja (`slip-emit-and-capture`): vsak okvir prispe, vendar kamere niso izpostavljene hkrati. `--force-tier sim-capture-sim-emit` vztraja pri sočasni izpostavljenosti. Glej [FPS niza in model serijskega zajema](#array-fps--burst-model).
> - **`lattice network-analysis` / `analyze-array` določata, kateri način povezave se uporabi**: brez `--pixel-format`, formate določa »connect« (Mono12 / BayerRG12, prej BayerRG8) in odločitev o [zgoščenem 12-bitnem prenosu](#packed-12-bit-transport---packed). Isti ukaz lahko zato navaja drugačno hitrost snemanja kot v različici 1.2.0.
> - **Relativne poti se nanašajo na mesto, kjer izvedete ukaz.** `-o` in druge poti, ki jih odpre backend (`--input`, `--burst-dir`, `--profile`, `--save`, `--mask`), `output` koraka recepta, ter privzeti `output/` iz `lattice index --live` se razrešijo glede na mapo, iz katere zaženete `chloros-cli`. V različici 1.2.0 jih je backend razrešil v svoji lastni namestitveni mapi, kar je pri nameščeni različici povzročilo napako »Dostop je zavrnjen«. Glej [Konvencije](#conventions).
> - **`lattice index` se v vseh oblikah izvaja v ozadju**, tako kot drugi ukazi `lattice`, zato najprej zaženite ozadje. Offline indeks (`--input`) izračuna backend, ki prebere datoteko TIFF in zapiše rezultate poleg nje ali v `-o`, `--list-presets` in `--list-gradients` pa natisneta tabele strežnika. V sistemu Windows zdaj delujejo: prej je Windows CLI na vsako obliko `lattice index`, vključno z `--live`, odgovoril z `Error: lattice_sdk not available`. V sistemu Linux se je indeks v načinu brez povezave (offline) prej izvajal znotraj CLI brez backenda in brez prijave; zdaj pa potrebuje oboje, tako kot vsak drug ukaz CLI. Glej [Indeks / Matematika vegetacije](#index--vegetation-maths).
> - **Nove datoteke poleg izdelkov.** Vsak produkt sevanja, odbojnosti in indeksa, v katerem je razmešavanje pasov omejilo piksel, dobi masko v mapi `clamp_masks/` poleg mape produkta, v izpisu pa se prikaže vrstica `clamp mask:`. Glejte [Maska omejitve ločevanja](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on).
> - **Usmeritev kamere.** Pri zajemu z eno kamero se datoteki `raw` in `debayered` shranita takšni, kot ju je prebral senzor, v XMP pa se zabeleži obrnitev/rotacija kamere; `process` to uporabi za izdelke, ki jih ustvari. Datoteke članov niza ostanejo v izvirni obliki, razen če prenesete nove `--orient` (`array-capture`, `array-burst`, `project capture` / `burst` / `stream`). Glejte [Usmeritev kamere](#kako-izgleda-mapa-z-posnetki).
> - **Ime ravni izvoza, ki ni eno, se zavrne.** Seznam `lattice capture --processing` in vsi seznami `--levels` vključujejo `raw`, `debayered`, `radiance`, `reflectance` ali `display` (ali psevdonim, kot je `preview`; `--processing` ter `all`). Vsako drugo ime se zavrne, še preden se dotaknete kamere (HTTP 400 `UNKNOWN_LEVEL`, izhod `1`); 1.2.0 je zanj shranil `raw` in se zaključil `0`.
> - **`daq pool-set-cap` potrebuje `--cap-id`.** Brez tega se ukaz zaključi `2`; 1.2.0 je odstranil korekcijo pokrova, s čimer je ostal gol senzor. `--cap-id none` zahteva goli senzor.
> - **`lattice resolution --binning-mode` je odpravljen.** Združevanje je vedno povprečno; zastavica se še vedno sprejme, izpiše opozorilo in ne stori ničesar. Glej [Nastavitev senzorja](#sensor-tuning).
> - **`process --debayer texture-aware` je zavrnjen za zajeme LATTICE.** Za mapo, ki vsebuje posnetke LATTICE, `process` po uvozu zapre `1`, pri čemer navede, koliko jih je, preden sploh začne obdelovati. Posnetki LATTICE vedno dobijo standardno demosaiciranje, zato je različica 1.2.0 to možnost sprejela in jo ignorirala; tudi aplikacija Chloros je za te posnetke ne ponuja. Glej [`chloros-cli process`](#chloros-cli-process).
> - **Kamere se vrnejo v stanje, v kakršnem so bile najdene — razen če je `--keep-settings`.** `lattice array-disconnect`, vsak prehodni ukaz (`multi-capture` brez matrike, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`) ter `lattice continuous` vsako kamero vrnejo v stanje, v kakršnem so jo našli: njena ožičenja sprožilca in sinhronizacijskega vodnika, njen PTP ter vse nastavitve kamere, ki so se med sejo spremenile (osvetlitev in ojačanje, ciljna vrednost AE, omejitve in dušenje, bela ravnovesje, raven črne, format pikslov, združevanje pikslov in ROI, obrati, hitrost snemanja). `--keep-settings` namesto tega shrani vsako kamerov lastni pomnilnik kamere, tako da se ob vklopu zažene z njimi; `--clear-settings` pa ta pomnilnik ponastavi na tovarniške nastavitve. V obeh primerih kamere ostanejo v prostem delovanju. Projekt nastavi svoje kamere iz lastnih shranjenih nastavitev, ko jih priključi. Glej [Pomnilnik kamere](#camera-memory----keep-settings----clear-settings).
> - **Vsak način shrani projekt na enak način.** Vsaka sprememba kamere, matrike ali svetlobnega senzorja, priključenega v okviru projekta – iz aplikacije, CLI, SDK ali strežnika MCP – se shrani v `cameras.json` / `sensors.json`, odpiranje projekta v katerem koli načinu pa nastavi naprave iz njega. Prej je projekt shranjevala le aplikacija, njeno naslednje shranjevanje pa je prepisalo vse ostalo. Glej [En zapis projekta, vsak način](#one-project-record-every-mode).
> - **Datoteke, ustvarjene iz več kamer, imajo v imenu navedene vse kamere.** Večpasovni niz (`align-apply`, `align-stream`, `index --live --save-multiband`), indeks v realnem času, sestavljen iz pasov več kamer, in slika `composite/` iz zajema v obliki matrike vsebujejo serijsko številko, model in kalibracijski paket vsake izvorne kamere v obliki seznamov (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), medtem ko EXIF `SerialNumber` navaja serijske številke, ločene z vejico. Prej niso vsebovale nobene kamere ali pa so vsebovale le serijsko številko prve kamere. Glej [Datoteke, ustvarjene iz več kamer](#files-made-from-several-cameras).
> - **Samo datoteka `raw` vsebuje kalibracijske oznake kamere.** Nabor, združljiv z MicaSense (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration` po posameznih kanalih, sklopi `MicaSenseVignetting*`) ter podatkovni del `CalibrationJSON` opisujejo surove vrednosti senzorja, zato so vključeni le v datoteko zajema `raw`. Vse ostale datoteke – zajemne datoteke `debayered`, `display`, `radiance`, `reflectance` ter indeksne datoteke, okvirji `lattice continuous` in `multi-stream`, in vsak izdelek `process` – vsebuje identifikacijsko številko kamere (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) ter ob zajemu tudi lastni zapis okvirja (`XMP-mapir:FrameJSON`: osvetlitev, ojačanje, temperatura, format pikslov, čas zajema). V različici 1.2.0 so vsebovali tudi niz surovih podatkov, ki ga orodje, ki ga uporablja (Pix4D, Metashape), uporablja za piksle, ki niso surove vrednosti. `process` poišče kalibracijo takšne datoteke po njeni serijski številki in `CalBundleSha256`.
>
> **Druge spremembe:**
>
> - **Samodejna hitrost snemanja matrike se prilagodi tistemu, kar matrika zagotavlja.** Hitrost, ki izgubi več kot 1 % svojih sprožilcev, se zniža – to se presoja na podlagi več daljših opazovanj namesto enega samega –, hitrost, s katere se je znižala, pa se občasno ponovno preizkusi. Hitrost, ki jo nastavite sami, ostane nespremenjena.
> - **Samodejna osvetlitev je stabilnejša.** Ohranja konstantno osvetlitev, se v nekaj sekundah prilagodi spremembam, prezre majhne svetle dele in ponovno vzpostavi delovanje kamere, ki preneha odgovarjati v močnem soncu. Glejte [Kako deluje samodejna osvetlitev](#how-auto-exposure-behaves).
> - **`process` šteje le vaše slike in posnetke.** Ponovno izvajanje na mapi, ki vsebuje `camera_cal/` projekta, teh kalibracijskih datotek ne šteje več („Najdenih 96 slik“ v nizu 90 slik), projekt, ki jih je že vključil, pa jih izloči.
> - **`lattice multi-test` upošteva stanje vklopa kamer.** Za izvedbo vklopi PTP kamer, počaka, da se njihove ure sinhronizirajo in ustalijo pred prvim sprožilcem – vsaka kamera se sinhronizira 120 s, približno 2 minuti od vklopa (`--ptp-settle SECONDS` to nastavi), saj je ura, ki se je pravkar uskladila, približno 2 minuti odstopa za do ~200 us – in vrne PTP vsake kamere v stanje, v kakršnem ga je našel. Na kamero, ki je še vedno usklajena iz prejšnjega testa, se ne čaka ponovno. V preteklosti je PTP ostal izklopljen v omrežju z enim stikalom, kjer ni odgovoril noben DAQ-E, in je prosto tekoči ura vsake kamere prebral kot PTP-uro, ki se ne bi sinhronizirala (`PTP is not locked`, vsak izvedbeni cikel). Glej [Večkamere (prehodne seje)](#multi-camera-transient-sessions).
> - **Zapis `.daq` navaja, ali je bil pokrovček izbran ali domnevan.** DAQ-U, DAQ-M in DAQ-E se dobavljajo brez pokrovčka, program Chloros pa ne more ugotoviti, ali je ta nameščen, zato senzor, priključen brez imenovanega pokrovčka, privzeto dobi oznako `sunshine_cosine`, kar je *predpostavka*. Vsak posnetek Chlorosa (shema 1.30) zdaj poleg `cap_id` shranjuje tudi `cap_id_source`: `operator`, če ste pokrovčku dodelili ime (vključno z `none`), `auto_default`, če je bil privzet. Pri obdelavi zapisa, za katerega je bil pokrovček domnevan, pri čemer je nastavitev DAQ v projektu ostala na »Auto«, izpiše `[daq-dls] WARNING` z imenom datoteke: goli DAQ-U brez korekcije sončnega svetloba kaže približno 30-krat višjo vrednost. **Poimenujte kapico, ki ste jo namestili** ob priključitvi ali nastavite kapico DAQ projekta, in opozorilo bo izginilo.
> - **V sistemu Windows se ne pojavljajo več okna konzole pip.** Na računalniku z nameščenim Pythonom so backend in vsak od njegovih delavnikov za obdelavo slik ob zagonu enkrat izvedli `pip`, vsak pa je odprl okno konzole. Zdaj tega ne počnejo več.
> - **Snemanja `.daq` v realnem času preživijo izpad električne energije.**Snemanje se zapiše z dnevnikom in popolno sinhronizacijo diska ob vsakem potrditvenem koraku, za senzorjem, tako da počasen disk upočasni le datoteko, nikoli pa odčitke. Posnetek, ki se je ustavil sredi potrditve (izpad električne energije, prekinitev delovanja), lahko poleg sebe pusti `<name>.daq-journal`:**kopirajte oba skupaj.**Chloros ob prvem branju datoteke vrne dnevnik nazaj, zato se posnetek odpre z vsem, kar je bilo potrjeno pred ustavitvijo (izgubljenih je približno zadnjih 3 s); v različici 1.2.0 bi izpad električne energije v napačnem trenutku lahko povzročil, da bi bila celotna datoteka neberljiva („slika diska baze podatkov je poškodovana“).
> - **Snemanje na zasedenem disku ne povzroča več zastoja senzorja.** `daq pool-record` odčitke postavi v čakalno vrsto in jih zapiše v lastnem niti; odčitek, ki ga ni bilo mogoče zapisati, se šteje (`daq pool-list` prikazuje `N write error(s)` z zadnjo napako) namesto da bi zamrznil senzor (eno zapisovanje je trajalo 8,4 s ob štirih kamerah, ki so shranjevale datoteke TIFF).
> - **`lattice multi-capture` brez matrike ne izgubi več okvirjev, ki jih je shranil, ko poteče časovni limit sprožilca.** Iztek časa sprožilca kamere (`SC_ERR_TIMEOUT -1011`, opazeno pri štirih kamerah na enem 1 GbE stikalu) zaključi delovanje pri tej skupini sličic: že shranjene skupine so navedene z `Saved:`, nato sledi napaka, izhod `1`. V različici 1.2.0 je bila to napaka HTTP-500 brez seznama, čeprav so bile datoteke na disku.
> - **Kamere s programskim sprožilcem ne izpuščajo več vsakega drugega sprožilca.** Kamera v načinu programskega sprožilca zdaj uvrsti v čakalno vrsto sprožilec, ki prispe med njenim odčitavanjem (ali pa Chloros počaka, da je pripravljen), kot so to že počele kamere s sinhronizacijo prek kabla.
> - **`daq pool-discover` poimenuje DAQ-E-S `DAQ-E-S`** v stolpcu `MODEL` (in `--json` v stolpcu `model`). Za vsak ethernetni senzor je navedel `DAQ-E`; tisti, ki v približno 2 sekundah ne odgovori na preverjanje identitete, je še vedno naveden kot `DAQ-E`.
> - **Izboljšani indeksni produkti LATTICE** (NDVI in drugi indeksi) iz `process`, v vseh načinih obdelave.
> - **Kamere OCN: izboljšani produkti sevanja, odbojnosti in indeksov v sončni svetlobi.** Po ločitvi pasov se uporabi popravek flote; kamere RGN ostajajo nespremenjene, vrednosti OCN pa se razlikujejo od tistih v različici 1.2.0. To velja tudi za posnetke OCN, narejene z zgodnejšo različico, če se obdelujejo na računalniku, ki vsebuje kalibracijo, s katero je bil posnetek narejen (program Chloros ohrani kalibracijo kamere, ko je ta enkrat priključena).
> - **`lattice continuous` zajame surovi posnetek.** Izvedba programa `tiff` je zapisala 3-ravninskih okvirjev, s katerimi `process` ne more začeti. Njegovi okvirji so zdaj datoteka `raw`, ki jo zapiše zajem matrike: ena mozaikna ravnina z lastnimi vrednostmi senzorja, nekomprimirana, s kalibracijskim paketom. Skript, ki iz teh datotek bere tri ravnine, bere le eno.
>
> - **Vsak zajem shrani odčitek DAQ, ki ga je uporabil, tako kot to počne grafični vmesnik.** `lattice capture`, `lattice continuous` (en `.daq` na okvir), `array-capture` in `multi-capture` s priključenim nizom (in SDK) zapišejo `.daq` in njegov `.match.json` vsakič, ko se DAQ veže ali združi v skupino, vključno z zajemom, ki je izključno v surovi obliki; v različici 1.2.0 se zajem v surovi obliki ali z odstranjeno barvno matriko shrani le z `--force-daq`. `--no-daq` to izklopi, `--daq-tolerance-ms MS` nastavi toleranco ujemanja za ta zajem, novi `daq pool-bind` prek CLI poveže kamero ali matriko z DAQ. V `.daq` se zajem, ki ga shrani, v `process` uskladi le s kamerami, ki jih imenuje. Zajem zdaj čaka na naslednje odčitavanje DAQ-ja, da lahko združi odčitke na obeh straneh osvetlitve. Glejte [Odčitek DAQ-ja, ki ga zajem shrani](#the-daq-reading-a-capture-saves) in [Odčitek DAQ-ja daleč od osvetlitve](#a-daq-reading-far-from-the-exposure).
> - **`process --indices NAME`** doda indeks v projekt za pasove vsake kamere, vključen je tudi nov projekt. V različici 1.2.0 je bilo ime, ki ga projekt še ni navajal, z `Unknown index name(s) ignored` prezrto.
> - **`process` ob živem nizu:**ne ponovno zažene več ozadja, ki vsebuje kamere ali DAQ-je, in počaka do 15 s na izvajanje, ki se zaključuje.**Ctrl+C** ustavi izvajanje v ozadju, ne le v CLI-ju. Glejte [Opombe k `process`](#notes).
> - **Projektna mapa, ki je pregloboka za omejitev poti v sistemu Windows (260 znakov)**, se takoj zavrne z najdaljšo mapo, ki ustreza omejitvi, namesto da bi se izvedba sploh ne zapisala. Namestitveni program za Windows zdaj ponuja možnost »Omogoči dolge poti v Windowsu« in izključitev v Microsoft Defenderju za ozadje, obe možnosti sta privzeto izbrani. Glej [Windows (.exe)](#windows-exe).
> - **`lattice array-capture --continuous`** izvaja lastno neprekinjeno zajemanje ozadja: vrstico napredka vsake 2 s, povzetek namesto seznama po posameznih datotekah ter izhod z vrednostjo, ki ni nič, ob kateri koli nezapisani datoteki ali polnem disku. Glejte [načine delovanja `array-capture`](#array-capture-modes).
> - **Prvi `array-capture` po `array-connect`** počaka do 20 s na prvo skupino okvirjev matrike, namesto da bi se pojavila napaka HTTP-503 »Matrika se še ogreva«.
> - **Izpostavljenost matrike v območju ROI:** majhno območje ROI ne omejuje več izpostavljenosti člana matrike na obdobje prostega delovanja senzorja (4,9 ms pri ločljivosti 768 × 304). Glej [Izpostavljenost člana matrike](#exposure-on-an-array-member).
> - **Avtomatska osvetlitev ne znižuje več avtomatske hitrosti snemanja matričnega senzorja.** Matrični senzor, sinhroniziran prek kabla, deluje s hitrostjo, ki jo omogočata omrežje in branje kamer; avtomatska osvetlitev zapolni časovno obdobje, ki ga ta hitrost omogoča, nato pa poveča ojačanje. Ročna osvetlitev, ki traja dlje od tega obdobja (`lattice exposure --set`, `configure --set`), upočasni sprožilec, da se prilagodi tej osvetlitvi, namesto da bi jo zavrnil, hitrost pa se ponovno vzpostavi, ko se element vrne v avtomatski način. Glej [Izpostavitev na elementu matrike](#exposure-on-an-array-member).
> - **Matrike zagotavljajo manj nepopolnih skupin slik** in manj napak 503 pri zajemanju („Ni nove sinhronizirane skupine slik“): sprožilni vhod vsake podrejene kamere je zdaj brez motenj.
> - **Pakirani 12-bitni prenos** (`--packed`, privzeto `auto`) poveča hitrosti snemanja pri nizih s polnim senzorjem za približno tretjino. `array-connect --packet-size` se zdaj uporablja (različica 1.2.0 ga je sprejela, a ignorirala). Glej [Pakirani 12-bitni prenos](#packed-12-bit-transport---packed).
> - **Niz, katerega kamere se po ponovnem zagonu napajanja samodejno obnovijo.** Po ponovnem zagonu napajanja stikala se niz `array-connect` ponovnoponovno odpre kamere, ko se te odzovejo, in ponovno začne pretakanje s ponovno vklopljenim PTP; v različici 1.2.0 se je za vedno ustavila in je bil potreben nov `array-connect`. `array-status` poroča o `ptp_enabled: False` (in `ptp_off:`), medtem ko kamera bere PTP kot izklopljeno, kjer je prej poročal o nastavitvi, s katero je bil niz povezan. Glej [Trajne povezave](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-record` snema, če ni odprt grafični vmesnik.** Ko je združen pogled grafičnega vmesnika zaprt, ozadje samo prikaže pogled: matrikoNDVI ali indeks, ki ga posredujete z novim `--expr`. Snemanje, ki ne zapiše nobenega okvirja, zdaj ne uspe (`no-frames: …`, izhod `1`). Prej se je snemalo le, dokler je bil združen pogled grafičnega vmesnika odprt, snemanje, ki ni zapisovalo ničesar, pa je še vedno zaprlo `0`. Glej [`array-record`](#array-record--combined-index-videogif-monitoring-grade).
> - **Kamera, ki jo Chloros že zadržuje, se ne odpre še enkrat.** Ukaz, ki bi jo sam odprl, namesto tega vrne odgovor HTTP 409 `CAMERA_HELD`, v katerem navede, kaj jo zadržuje, in ukaz, ki jo sprosti. To vključuje tudi kamero, ki jo trenutno uporablja drug ukaz (npr. `multi-test` ali `multi-capture`, ki odpira svoje kamere, ali `lattice capture`): drugi ukaz se zavrne, dokler prvi ne konča. Ponovno odpiranje bi lahko ustavilo tok imetnika ali povzročilo sesutje strežnika. Glej [Trajne povezave](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **Začasni ukazi za več kamer uspešno prikažejo vse kamere ali pa se ne izvedejo.** `lattice multi-capture` brez priključenega niza, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream` in `index --live` zaprejo `1` z navedbo kamere, ki ni posredovala nobenega okvirja. Tokovi preskočijo skupino sličic, v kateri manjka kamera, in se ustavijo po treh zaporednih; `multi-test` natisne vrstico za vsak sprožilec, `(no frame)` pa navede, kje je kamera zamudila; `align-stream` prav tako zapre `1` zaradi napake pri opravilu. Nato vsak izvedbeni cikel vrne vsako kamero v stanje, v katerem jo je našel: njeno ožičenje sprožilca in sinhronizacijske linije, njen PTP (`PtpEnable`) ter vse nastavitve kamere, ki jih je izvedba spremenila – osvetlitev in ojačanje, cilj AE, omejitve in dušenje, belo ravnovesje, raven črne, format pikslov, združevanje in ROI, obrati, hitrost snemanja – razen če `--keep-settings` / `--clear-settings` (glej [Pomnilnik kamere](#camera-memory----keep-settings----clear-settings)). Prej so ti ukazi ohranili vse posnete slike in zaprli `0` (`align-stream` je izpisal napako pri opravilu kot `Note:`), vsako kamero pa so pustili v `TriggerMode` `On`. Prav tako so za vsako kamero nastavili osvetlitev na podlagi prejšnje scene (največ nekaj sekund), pri čemer je povezana matrica vsako kamero zagnala na: 40 multispektralnih, 60 RGB (nastavi ga `--brightness N`) ter jih ohranja med celotnim potekom; enako velja za posamezno kamero `lattice capture`, ki je ni v lasti Chloros. Prej je bil prvi posnetek vsake kamere zajet pri lastni omejitvi osvetlitve kamere, približno 100 ms, ne glede na svetlobo, in je bil pogosto odrezan. Glej [Večkamere (prehodne seje)](#multi-camera-transient-sessions).
> - **Prehodni ukazi za več kamer si delijo omrežje na enak način kot `array-connect` in odprejo le `--serials`.** Vsaka kamera je prilagojena svojemu deležu gostiteljske povezave, tako da štiri kamere delujejo na povezavi s hitrostjo 1 Gbps; prej so štiri kamere skupaj zahtevale 125 MB/s, kar je v celoti izčrpalo 1 GbE, pri čemer je vsak okvir izgubil pakete. Niz, ki ga omrežje ne more prenesti, se zavrne, preden katera koli kamera začne pretakati. `--serials` zdaj izbere kamere, pri čemer prva sproži sprožilec; prej je vsak ukaz odprl vse kamere v omrežju, ne glede na to, kaj je navedel `--serials`. Napaka, ki je izguba paketov GVSP, to zdaj tudi navaja, namesto da bi kazala na sinhronizacijski kabel M8. Glej [Več kamer (prehodne seje)](#multi-camera-transient-sessions).
> - **`lattice array-disconnect` pusti kamere delovati prosto, kot so bile najdene.** `TriggerMode`, `Off` in vse ostalo, kot je bilo ugotovljeno ob povezavi – nastavitve sprožilca in sinhronizacijskega kabla (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`), PTP ter nastavitve kamere, ki jih je spremenila seja – tako en samukaz za kamero, namenjen nekdanjemu članu, pridobi vse posnetke naenkrat, glavna kamera pa ni več nastavljena na `TriggerSource` `Software`. `--keep-settings` shrani nastavitve seje v pomnilnik vsake kamere, `--clear-settings` pa ta pomnilnik ponastavi (glej [Pomnilnik kamere](#camera-memory----keep-settings----clear-settings)). `lattice continuous` prav tako nastavi kamero, ki jo najde v stanju »armed«, na »free-run« za tek (in jo po koncu ponovno vrne v prejšnje stanje), tek brez izhodnih okvirjev pa zaključi `1`.
> - **`lattice configure --get` / `lattice exposure`** izpišejo `note:` zdejansko osvetlitev ali ojačanje kamere, medtem ko deluje avtomatski način. Zavrnjeni `lattice exposure --set` / `configure --set` navaja območje, ki ga kamera sprejema, in ponovno vklopi avtomatski način, namesto da bi izklopil avtomatsko osvetlitev. Glej [Nastavitev senzorja](#sensor-tuning).
> - **Novi ukazi za kamero:**`lattice highlights` (zmanjša svetlost belih površin v prikazu RGB) in `lattice exposure --brightness-auto`.**`lattice multi-detect`** v enem krogu sondiranja poišče žične kamere in kot glavno kamero privzeto nastavi RGB-kamero.
> - **Novi ukazi DAQ:** `daq pool-bind`; `pool-enable-ptp`; ter za lastne nastavitve naprave DAQ-E: `pool-info`, `pool-set-network`, `pool-set-auto-stream`, `pool-reboot` ter `pool-update-firmware`. Glejte [Nastavitve naprave DAQ-E](#daq-e-device-settings).
> - **DAQ-E, ki preneha pošiljati podatke, se ponovno poveže prek backenda.** Ko tok združenega DAQ-E utihne (zaradi ponovnega zagona stikala, ki ga napaja, prekinjena povezava s kamero), ga backend ponovno poveže z njegovimi prejšnjimi nastavitvami in med njegovo odsotnostjo ponavlja poskuse v vse daljših intervalih. Snemanje na njem se nadaljuje v istem `.daq`, pri čemer je v datoteki označena vrzel, `pool-record --stop` pa navaja vsako prekinitev. V različici 1.2.0 je senzor ostal tiho do datotek `pool-disconnect` + `pool-connect`, kar je hkrati pomenilo tudi konec snemanja. Glej [DAQ-E, ki preneha pošiljati podatke](#a-daq-e-that-stops-delivering).
> - **`time-sync status`** poroča, ali paketi PTP grandmastera dosežejo omrežje (`health`), v primeru, da jih ne dosežejo, pa se na stderr izpiše `WARNING:`. Glej [`time-sync`](#chloros-cli-time-sync).
> - **.deb za Linux:** poveča velikost sprejemnih pomnilnikov vtičnic jedra, ki jih potrebujejo tokovi iz kamer (pri privzeti nastavitvi jedra se izgubljajo okvirji in hitrost snemanja), doda strežniško storitev, ki teče pod vašim uporabniškim računom, ter `lattice network` preveri pomnilnike. CLI-je nikoli ne ustavi ozadja, v katerem teče storitev, nadgradnja pa ponovno zažene omogočene storitve v ozadju. Glej [Zagon ozadja kot storitve](#running-the-backend-as-a-service-headless) in [Linux gostitelji — sprejemni pomnilniki vtičnic](#linux-hosts--socket-receive-buffers).
> - **`lattice array-capture` sporoča, kadar ni mogel izvesti poravnave:**vsak razlog, zaradi katerega je bil zajem shranjen neporavnan (ni profila poravnave, elementi z različnimi ločljivostmi, niz, ki ni sinhroniziran s kablom, okvir, na katerega profila ni mogoče preslikati), se izpiše kot vrstica `Alignment:`.**`chloros-cli daq help`** je `daq --help` (program se je zaključil z `2`), v Linuxu pa pomoč navaja ukaze za neposredno upravljanje strojne opreme `daq` , ki jih ta zgradba vključuje.
> - **Jetson:** paket je `chloros_1.3.0_arm64_jp6.deb`, za JetPack 6, njegova obdelava na GPU pa poteka na Orinu z JetPackovim CUDA PyTorchom. Glej [Linux arm64 — Jetson](#linux-arm64--jetson-jetpack-6).

---

## Konvencije

- Vsi ukazi imajo predpono `chloros-cli`. V sistemu Windows je izvedljiva datoteka `chloros-cli.exe`; v sistemu Linux/Jetsonu pa `chloros-cli`.
- Izbirni argumenti so prikazani kot `--flag`. Obvezni pozicijski argumenti so prikazani brez oklepajev.
- Kadar je navedena privzeta vrednost, se ob izpuščanju zastavice uporabi ta vrednost.
- CLI je preprost HTTP-odjemalec prek strežniškega dela Chloros (strežnik na naslovu `127.0.0.1:5000`, ki ga uporablja tudi namizna aplikacija). **Kateri ukazi ga zaženejo:**
  - Osnovni ukazi — `process`, `login`, `logout`, `status`, `export-status`, `time-sync` in `selftest` — uporabljajo že zagnani backend ali ga zagnajo, če še ni zagnan, ter ga po končanem delu ponovno ustavijo.
  - **Ukazi `lattice`, `project` in `daq` ga nikoli ne zaženejo.** Potrebujejo backend, ki že teče: backend aplikacije za namizje v Windowsu, [backend storitev](#running-the-backend-as-a-service-headless) v Linuxu ali tistega, ki ga zaženete sami (glejte [Zaženi backend](#start-the-backend)). Brez njega izpišejo `Error: Chloros backend not reachable at http://127.0.0.1:5000` skupaj z ukazom, ki ga zažene, in se zaključijo `1`.
- `CHLOROS_BACKEND_URL=<url>` usmerja družine ukazov **`lattice`**,**`project`**in**`daq pool-*`** na drug backend (`lattice network-analysis` namesto tega usmerja `--backend-url`). Osnovni ukazi namerno fiksirajo `http://127.0.0.1:<port>` in ga ignorirajo (literal IPv4 prepreči kazen v sistemu Windows v višini približno 2 sekund na zahtevo za `localhost`→`::1`). Glej [Okoljske spremenljivke](#environment-variables).
- **Relativne poti se razrešijo glede na mapo, iz katere zaženete `chloros-cli`.** Backend je proces, ki odpre izhodne in vhodne mape, zato CLI pred pošiljanjem poti to spremeni v absolutno: `-o` / `--output` (vključno s privzeto potjo, `output`), `--input`, `--burst-dir`, `--profile`, `--save` in `--mask`, korak [recepta](#recipe-dsl) `output` (privzeta mapa `output`) ter privzeta mapa `output` v `lattice index --live`. `-o captures/` zato piše v `<current folder>/captures/`, natisnjene poti pa so absolutne. `~` se razširi. Pri oddaljenem backendu (`CHLOROS_BACKEND_URL`, ki imenuje drug računalnik) se poti pošljejo takšne, kot so vnesene, in se razrešijo na tem računalniku.
- Za vse SDK/CLI (enkrat na računalnik zaženite `chloros-cli login`; shranjeno v predpomnilniku v `~/.chloros/`).
- Primeri uporabljajo poti v sistemu Linux; v sistemu Windows zamenjajte `/home/user/...` z `C:/Users/.../...`. Serijska vrata naprave DAQ-U'je v slogu `COM3` v sistemu Windows ter `/dev/ttyUSB0` / `/dev/ttyACM0` v sistemu Linux.
- V sinopsisah so medsebojno izključujoči zastavki zapisani kot `{--a | --b}`, izbirne pa v obliki `[...]`.

---

## Pregled na najvišji ravni

```
chloros-cli [global options] COMMAND [command options]
```

### Globalne možnosti

| Zastavica | Opis |
| --- | --- |
| `--backend-exe PATH` | Prepis samodejno zaznanega izvedljivega datoteka ozadja. |
| `--port N` | HTTP-vrata ozadja (privzeto: `5000`). Uporabljajo ga le osnovni ukazi (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`); `lattice`, `project` in `daq` uporabljajo `CHLOROS_BACKEND_URL` (privzeto `http://127.0.0.1:5000`). |
| `-v, --verbose` | Omogoči podrobni izpis. |
| `--restart` | Prisilni ponovni zagon ozadja (prekine vse tekoče procese `backend_server.py`). Ozadje, ki ga poganja storitev systemd (Linux .deb), se nikoli ne ustavi: ukaz namesto tega izpiše `sudo systemctl restart <unit>` in se zaključi `1`. |
| `--version` | Izpiše različico (`Chloros CLI 1.3.0`). |
| `--help` | Prikaže pomoč najvišje ravni. |

### Kazalo ukazov

| Ukaz | Namen |
| --- | --- |
| [`process`](#chloros-cli-process) | Obdelajte mapo z zajetimi podatki iz programov Survey3 ali »LATTICE« od začetka do konca. |
| [`login`](#chloros-cli-login) | Preverjanje pristnosti tega računalnika z računom Chloros+. |
| [`logout`](#chloros-cli-logout) | Izbriše shranjene poverilnice iz predpomnilnika. |
| [`status`](#chloros-cli-status) | Prikaži trenutno stanje licence / avtentifikacije. |
| [`export-status`](#chloros-cli-export-status) | Prikaz napredka izvoza v realnem času v Thread-4 med izvajanjem `process`. |
| [`language`](#chloros-cli-language) | Nastavi ali prikaži jezik prikaza CLI-ja (podprtih je 38 jezikov). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Privzeta mapa projekta (skupna z GUI). |
| [`update`](#chloros-cli-update) | Preveri in namesti posodobitve Chloros (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Sistemsko diagnosticiranje + hitri testi. |
| [`time-sync`](#chloros-cli-time-sync) | Stanje/nadzor glavnega PTP-strežnika. |
| [`lattice`](#chloros-cli-lattice) | Nadzor in zajem kamere LATTICE (več kot 45 podukazov). |
| [`daq`](#chloros-cli-daq) | Nadzor spektralnih senzorjev DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Odpiranje in upravljanje shranjenega projekta Chloros (kamere + DAQ-ji). |

---

## Namestitev

`chloros-cli` je vključen v namestitveni program Chloros za namizne računalnike na vseh podprtih platformah — ločenega prenosa za CLI ni. Z namestitvijo paketa za platformo se `chloros-cli` doda v vaš `PATH` skupaj z namizno aplikacijo in binarnim programom v ozadju, ki ga ta poganja.

Najnovejši prenosi: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Namestitveni program vsebuje tudi priročne skripte za zagon (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`), ki odprejo takoj uporabno CLI-okolje.

### Windows (.exe)

1. Prenesite namestitveni program za Windows s strani za prenos.
2. Zaženite `Chloros Installer 1.3.0.exe` in sledite navodilom čarovnika. Privzeta namestitvena pot je `C:\Program Files\Chloros\` (CLI se nahaja v `C:\Program Files\Chloros\cli\`, ki ga namestitveni program doda v PATH).
3. Odprite nov terminal (`cmd.exe`, PowerShell ali Windows Terminal), da se zazna posodobljeni `PATH`.

```powershell
chloros-cli --version
```

Namestitveni program samodejno doda `chloros-cli.exe` v vaš sistemski `PATH` in vključi gonilnike za kamere LATTICE. Backend je `C:\Program Files\Chloros\backend\chloros-backend.exe` (mapa `backend` v vaši namestitveni mapi), ki ni v `PATH`.

**Sistemski nastavitve.** Stran *Izberi dodatne naloge* v namestitvenem programu ima pod **Sistemskimi nastavitvami**dve možnosti, ki sta obe**privzeto označeni**:

| Možnost (kot je navedena v namestitvenem programu) | Kaj počne | Izklop tihe namestitve |
| --- | --- | --- |
| **Omogoči dolge poti v sistemu Windows** (omogoča, da so projektne mape globlje od 260 znakov; priporočeno) | Nastavi sistemsko nastavitev `LongPathsEnabled` v sistemu Windows na 1. Windows je privzeto izklopljen, takrat nobena pot ne sme presegati 260 znakov; izdelki Chloros se nahajajo do približno 130 znakov pod projektno mapo, zato se globlja projektna mapa zavrne (glej [Odpravljanje težav](#troubleshooting-pointers)). Velja za programe, ki se zaženejo po namestitvi. Ob odstranitvi ostane vklopljeno, saj se drugi programi morda zanašajo na to nastavitev. | `/MERGETASKS="!longpaths"` |
| **Izključi Chlorosov backend iz skeniranja Microsoft Defenderja**(precej hitrejše zajemanje s kamero; priporočeno) | Dodaja izključitev**procesa** Defenderja za `chloros-backend.exe` v namestitveni mapi, tako da zaščita v realnem času ne skenira vsake datoteke, ki jo backend zapiše, ne glede na to, kam jo zapiše. Noben drug program ni izključen; izključitev se ob odstranitvi programa odstrani. Če Defender ni nameščen ali je blokiran s pravilnikom, ta korak tiho ne uspe in namestitev se nadaljuje. | `/MERGETASKS="!defenderexclusion"` |

**Zakaj je možnost Defender pomembna:** pri visokih hitrostih zajemanja realnočasno skeniranje zajetih datotek porabi približno 30 % hitrosti zajemanja (merjeno pri neprekinjenem zajemanju z 9 kamerami: 77,7 zajemov/s izključenih, 52,7 pregledanih). `lattice array-capture --continuous` prikaže opombo, če zazna, da Defender pri zajemanju uporablja eno ali več jeder procesorja; ponovno zaženite namestitveni program z izbrano možnostjo.

Pri tihem namestitvenem postopku (`/SILENT` ali `/VERYSILENT`) se uporabljata obe možnosti, razen če se izključita, npr. `"Chloros Installer 1.3.0.exe" /VERYSILENT /MERGETASKS="!defenderexclusion"`.

**Požarni zid Windows.** Namestitveni program vedno doda dve pravili za vhodni promet za `chloros-backend.exe` v vsakem omrežnem profilu, program za odstranitev pa jih odstrani: `Chloros Backend (UDP-In)` za tokove kamer, PTP, mDNS in podatkovni tok DAQ-E ter `Chloros Backend (TCP-In)`, omejeno na lokalno podomrežje, za posodobitev firmware-a DAQ-E, pri kateri se senzor ponovno poveže s tem računalnikom, da zbira sliko (glej [Nastavitve naprave DAQ-E](#daq-e-device-settings)). HTTP-API strežnika ostane vezan na `127.0.0.1`.

### Linux amd64 (.deb)

Za delovne postaje z operacijskim sistemom Ubuntu 22.04 LTS ali novejšim oziroma na osnovi Debiana z arhitekturo x86_64.

> **Ubuntu 20.04 ni podprt.** Seznam odvisnosti paketa izhaja iz
> tega, s čim se backend dejansko povezuje, kar vključuje `libc6 (>= 2.35)`;
> focal vključuje glibc 2.31. `apt` zavrne namestitev, namesto da bi dopustil, da ta
> med izvajanjem spodleti. Seznam navaja tudi `libgl1`, ki ga naloži priloženi OpenCV, v minimalni
> strežniški sliki pa ga morda ni; `apt install` ga prinaša.

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
```

Datoteka .deb namesti:
- `chloros-cli` do `/usr/bin/chloros-cli` ter `chloros-backend` (zažene backend) do `/usr/bin/chloros-backend`
- Kompilirani backend in CLI, razpakirana, v `/usr/lib/chloros/backend/` in `/usr/lib/chloros/cli/`
- Gonilniki kamer LATTICE
- Modeli za odstranjevanje šuma, kalibracijski paketi in konfiguracija kanala za posodobitve
- Pythonov SDK za sistem `python3` (`import chloros_sdk`). `apt install` prinaša tudi svoje zahteve, `python3-numpy` in `python3-requests` (priporočeno, ni obvezno: zgornji programi so samostojni; `dpkg -i` jih preskoči, izpis namestitve pa navaja, kaj manjka). V virtualnem okolju: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`
– `/etc/sysctl.d/60-chloros-gige.conf`, ki poveča velikost sprejemnih pomnilnikov jedra za tokove kamere tokov poveča na 32 MB in jih takoj uporabi (že višja vrednost ostane nespremenjena). Odstranitev paketa odstrani datoteko. Glejte [Linux gostitelji — sprejemni pomnilniki vtičnic](#linux-hosts--socket-receive-buffers).
- Dve izbirni storitvi systemd za backend, nobena od njiju ni omogočena (naslednji oddelek)

#### Zagon backenda kot storitve (brez grafičnega vmesnika)

Brez storitve backend (in z njim tudi PTP grandmaster) deluje le, dokler ga nekaj ne zažene: `chloros-backend` v terminalu, skriptu SDK ali osnovnem ukazu `chloros-cli` za čas njegovega delovanja. Za backend, ki je vedno vklopljen na računalniku brez grafičnega vmesnika, omogočite storitev **kot vaš uporabnik**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` zažene backend kot ta uporabnik. Zapisi, ki jih ustvari, pripadajo temu uporabniku, podrobni dnevnik pa se nahaja v njegovem `~/.cache/chloros/logs`, uporablja pa njihov `chloros-cli login` (`~/.chloros`), zato se najprej enkrat prijavite kot ta uporabnik. PTP ne potrebuje pravic root. Kot vsak nameščen backend posluša le na `127.0.0.1:5000`.

`chloros-backend.service` izvaja backend kot **root**. Še vedno je nameščen, da računalnik, na katerem je omogočen, še naprej deluje, vendar vse, kar zapiše backend z root pravicami, pripada rootu, zato uporabnik ne more izbrisati svojih posnetkov. Za preklop na svojega uporabnika:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

Podrobni dnevnik storitve root se nahaja v datoteki `/var/log/chloros`, ki jo lahko berejo člani skupine `adm` (storitev nastavi `CHLOROS_LOG_DIR`; glej [Okoljske spremenljivke](#environment-variables)). Pri namestitvi paketa se izpiše ukaz `enable` z vnesenim imenom vašega uporabnika ter ukaz za preklop, če je storitev root omogočena.

**Omogočite eno backend enoto.** Hkrati deluje le ena backend enota. Druga omogočena enota (`chloros-backend.service` poleg `chloros-backend@<user>.service` ali dve uporabniški enoti `@`) se ne zažene in ostane `failed` s statusom izhoda 98; systemd ga ne zažene ponovno. Njena `systemctl status` navaja razlog: `Another instance is already running. Exiting.`, nato pa tisto, kar zadržuje backend (vrata 5000 ali datoteka zaklepa in uporabnik, ki ji pripada), nato `systemd will not restart <unit> (exit status 98)`. Po ponovnem zagonu lahko deluje katera koli enota. Za preklop onemogočite tisto, ki je ne uporabljate, kot je opisano zgoraj.**CLI**nikoli ne ustavi backenda storitve.**`chloros-cli`**ga uporablja takšnega, kakršen je, tako kot GUI in**SDK**:**`process`** teče na njem in namesto zadnjega projekta naloži svojega. Kjer bi CLI sicer ponovno zagnal ozadje (`--restart` ali `process`, katerega uvoz ni našel ničesar), poimenuje enoto, izpiše `sudo systemctl restart <unit>` in zapre `1`, ker bi backend, ustavljen na ta način, ostal neaktivni. `GET /api/status` poimenuje enoto kot `service` (`null` za backend, na katerem ne teče nobena storitev).**Nadgradnja in odstranitev.** Nadgradnja paketa ustavi vse strežniške storitve, nato pa ponovno zažene vsako, ki je omogočena, najprej enote `@<user>` (tista, ki je bila ročno zagnana, a nikoli omogočena, ostane ustavljena, kot po ponovnem zagonu sistema). Če je omogočenih več storitev, deluje le ena, ostale pa ostanejo v stanju napake, kot je opisano zgoraj; nadgradnja izpiše `Chloros: N backend units are enabled: …` in `Keep one enabled: sudo systemctl disable --now <unit>`. Odstranitev jih ustavi in onemogoči. Če ga izbrišete, se izbriše tudi `/var/log/chloros`.

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Enaka struktura kot pri .deb datoteki za amd64, vključuje sprejemne pomnilnike in [storitve backenda](#running-the-backend-as-a-service-headless), za Jetson Orin / Orin NX / Orin Nano na **JetPack 6** (L4T R36). Njegov PyTorch je CUDA-različica JetPack 6 za Orin, zato obdelava na GPU, vključno z debayerjem, ki upošteva teksture, poteka na GPU-ju Jetsona. Paket sam ne vsebuje CUDA: uporablja knjižnici CUDA in cuDNN, ki ju namesti JetPack 6 (`sudo apt install nvidia-jetpack` na sliki, ki ju nima). Kjer ni na voljo grafičnega procesorja s podporo za CUDA, obdelava poteka na procesorju, obdelava z upoštevanjem tekstur pa uporablja standardni debayer (glej [`process`](#chloros-cli-process)).

### Enkratna avtentifikacija na napravo

Vsaka platforma zahteva enkratno prijavo v Chloros+, preden začnejo delovati klici SDK-ja/CLI-ja:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Pristojnosti so shranjene v predpomnilniku v `~/.chloros/user_session.json`.

### Zagon backenda

Ukazi `lattice`, `project` in `daq` komunicirajo z backendom, ki že teče; nikoli ga ne zaženejo (glej [Konvencije](#konvencije)). Med uporabo teh ukazov poskrbite, da backend ostane zagnan:

| Platforma | Kako |
| --- | --- |
| Windows | Odprite namizno aplikacijo Chloros (njen backend podpira tudi CLI) ali zaženite `"C:\Program Files\Chloros\backend\chloros-backend.exe"` v lastnem terminalu (mapa `backend` v vaši namestitveni mapi) in ga pustite odprtega. |
| Linux/Jetson | Zaženite `chloros-backend` v lastnem terminalu ali pa eno vedno pustite v teku s [storitvijo ozadja](#running-the-backend-as-a-service-headless). |

Hkrati teče le en backend; vsi odjemalci (grafični vmesnik, CLI, skripti SDK) si ga delijo. Tudi osnovni ukazi (`process`, `login`, `status`, …) ga uporabljajo in ga zaženejo, če še noben ne teče.

**Backend deluje neprekinjeno med posameznimi ukazi**, zato naslednji ukaz ne sproži novega zagona. Ustavi se, ko se odjavite (`chloros-cli logout`, razen če je odprta aplikacija Chloros ali poteka izvajanje ali izdelava brez povezave), ko zaženete `chloros-cli backend stop` ali samodejno po 30 minutah, če ga nič ne uporablja: brez ukazov, skriptov ali aplikacij, brez izvajanja ali izdelave brez povezave, brez priključene kamere ali svetlobnega senzorja. `backend stop` zavrne ukaz, medtem ko poteka izvajanje ali izdelava brez povezave, in ga poimenuje; `--force` ga takoj ustavi, pri čemer se to izvajanje ali izdelava izgubi (serija posnetkov se ohrani, `array-build-video` pa jo ponovno sestavi). Nastavite `CHLOROS_BACKEND_IDLE_MIN`, preden se backend zažene, da spremenite 30 minut (`0`: nikoli); [storitev backenda](#running-the-backend-as-a-service-headless) se nikoli ne ustavi sama od sebe. `chloros-cli backend status` prikazuje, ali kateri od njih teče, kaj ga uporablja in ali teče kakšna izgradnja v načinu brez povezave. `process` odpre svoj projekt v tekočem backendu, razen če je aplikacija Chloros odprta s projektom, ki ga odprla: v tem primeru to zavrne, zato morate najprej opraviti postopek iz aplikacije ali zapreti ta projekt. Projekt, ki ga je prejšnji `process` ali skript SDK pustil odprt, se preprosto nadomesti.

### Preverjanje namestitve

```bash
chloros-cli --version           # prints "Chloros CLI 1.3.0"
chloros-cli selftest            # 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

`selftest` se zaključi `0` šele, ko je uspešno opravljenih vseh sedem preverjanj. Zadnje preverjanje zahteva CUDA GPU (NVIDIA GPU, ali Jetson Orin), ter modele za odstranjevanje šuma; zato na računalniku brez njega prikaže `Failed: CUDA + Denoiser` in zaključi delovanje `1`; tam še vedno deluje vse razen debayerja, ki upošteva teksture in je namenjen izključno grafičnim procesorjem.

> **Potrebna je naročnina na Chloros+.**CLI zahteva aktivni načrt Chloros+.**Copper**je osnovna stopnja Chloros+ — vsaka plačljiva stopnja Chloros+ ima dostop do CLI/SDK; le brezplačna stopnja**Iron** ga nima. (Povezava med ID-ji načrtov: `0`=Iron/free, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Nadgradnja na [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> To omejitev uveljavlja strežniški del, ne le CLI: zahteva z oznako SDK/CLI brez plačanega paketa se zavrne s kodom napake `403 PLAN_UPGRADE_REQUIRED`, ne glede na to, ali prihaja iz `chloros-cli`, Python SDK ali ročno napisanega HTTP odjemalca. Klicatelj, ki ni prijavljen, namesto tega prejme napako `401 AUTH_REQUIRED`. Dostop deluje brez povezave v obdobju odloga plačila za načrt (30 dni za mesečni načrt, do izteka veljavnosti za letni načrt) in se ustavi, ko to obdobje poteče; `chloros-cli status` še naprej deluje, da je razlog viden (to je edina pot SDK/CLI, ki je izvzeta iz omejitve po stopnjah — `GET /api/license-status`).

---

## Petminutni hitri začetek

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI GNDVI OSAVI

# 3. The lattice / daq commands need a running backend: on Windows open the
#    Chloros desktop app; on Linux start one (or see "Start the Backend")
chloros-backend &

# 4. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 5. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 6. Read a spectrum from a connected DAQ-U (Windows: --port COM3)
chloros-cli daq pool-connect --port /dev/ttyUSB0
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

`process` (korak 2) zažene backend za lastno izvajanje, če še noben ne teče, in ga ustavi, ko konča, zato ga zaženite za korak 4 naprej. Glejte [Zagon ozadja](#start-the-backend).

---

## `chloros-cli process`

Obdelajte mapo s slikami skozi celoten Chlorosov potek (prepoznavanje cilja → kalibracija → vinjeta → odbojnost → izvoz indeksa).

### Pregled

```
chloros-cli process INPUT [OPTIONS]
```

### Pozicijski argumenti

| Argument | Opis |
| --- | --- |
| `INPUT` | Pot do vhodne mape, ki vsebuje datoteke `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) ali `.dng`. |

### Splošne možnosti

| Oznaka | Privzeto | Opis |
| --- | --- | --- |
| `-o, --output PATH` | nova mapa z imenom `-n` v privzeti projektni mapi | Projektna mapa. Projekt je poimenovan po njej, mapa, v kateri se že nahaja datoteka `project.json`, pa dobi ime `_1` / `_2`, namesto da bi bila prepisana. Privzeta mapa projekta je tista, ki jo je izbral `set-project-folder` (ali grafični vmesnik), sicer pa `Documents/MAPIR/Chloros Projects`. |
| `-n, --project-name NAME` | avtomatsko (časovni žig) | Ime projekta, uporablja se le **brez** `-o` (z `-o` projekt prevzame ime mape). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` uporablja nevronski debayer Chloros+ za posnetke Survey3; počasnejši, a višje kakovosti. Namenjen je izključno za Survey3: Posnetki LATTICE vedno uporabljajo standardno demosaiciranje, zato za mapo, ki vsebuje posnetke LATTICE, `process` po uvozu zavrne `texture-aware` z napako, ki navaja, koliko jih je, in zapre `1`, preden sploh začne obdelovati (aplikacija Chloros tudi za te posnetke ne ponuja funkcije »Texture Aware«). Potreben je grafični procesor CUDA (grafični procesor NVIDIA v sistemih Windows / Linux x64, grafični procesor Orin na Jetsonu): brez njega program uporabi `standard`, backend pa v dnevnik zapiše eno vrstico `[DEBAYER] … needs a CUDA GPU …`, ki pojasnjuje razlog (prikazano z `--verbose`). |
| `--vignette / --no-vignette` | `--vignette` | Popravek vinjetiranja. |
| `--reflectance / --no-reflectance` | `--reflectance` | Kalibracija odbojnosti (uporabi ciljni panel, če je najden, NIST-ovo kalibracijo po serijski številki za LATTICE). Pri multispektralnem LATTICE-u to hkrati deluje kot **produkt** odbojnosti — glej [Stikala za izvoz po posameznih izdelkih](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | izklopljeno | Uporabi PPK GNSS popravke iz sidecar datotek. |
| `--exposure-pin-1 CAMERA_MODEL` | noben | Model kamere, povezan s kontaktom za osvetlitev 1, za PPK časovno sinhronizacijo. |
| `--exposure-pin-2 CAMERA_MODEL` | none | Model kamere, priključen na izhod za osvetlitev 2, za časovno sinhronizacijo PPK. |
| `--recal-interval SECONDS` | `0` | Najkrajši interval ponovne kalibracije v sekundah (*Najkrajši interval ponovne kalibracije* projekta). |
| `--timezone-offset HOURS` | `0` | Časovni zamik svetlobnega senzorja glede na UTC, v urah (projektni *Časovni zamik svetlobnega senzorja*). |
| `--format FORMAT` | `TIFF (16-bit)` | Ena od naslednjih vrednosti: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`. Program Chloros nikoli ne poveča bitne globine slike: pri vseh 8-bitnih slikah se format z večjo globino zapiše kot `TIFF (8-bit)`, kar je v izpisu navedeno enkrat. V projektu, ki združuje 8-bitne in 12/16-bitne posnetke, se vsak posnetek izvozi v svoji globini ali nižji. Sijaj je vedno 32-bitni plavajoči. |
| `--indices NAME [NAME ...]` | noben | Indeksi vegetacije po imenu prednastavitve (`NDVI`, `GNDVI`, `OSAVI`, `SAVI`, `EVI`, `GCI`, …), ne glede na velike in male črke. Vsak indeks se doda v projekt za pasove zadevne kamere, vključen pa je tudi nov projekt. Ime, ki ni prednastavitev ali ki ga pasovi kamere ne morejo ustvariti (`NDRE` potrebuje RE in NIR, česar nobena posamezna kamera ne zagotavlja), se v opozorilu navede in ne dobi indeksne slike. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Prisili začetno točko procesne verige za datoteke LATTICE TIFF (datoteke Survey3.raw to ne vpliva). Prav tako je to izhod v sili, ki omogoča obdelavo zajema **brez surovega podatka** – glej [Kako izgleda mapa zajemov](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | vklopljeno | Izpiše linearni produkt brez bayerske matrike (`Debayered_Images`). Glej [Stikala za izvoz po posameznih izdelkih](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | vklopljeno | Izpiše predogled zaslona (`Preview_Images`): RGB = bela bilanca (DAQ-svetlobni vir, če je na voljo, sicer siva skala) + gama; multispektralno = raztezek v lažnih barvah. |
| `--radiance / --no-radiance` | vklopljeno | Izpisuje se svetilnost v formatu float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referenca za produkt odbojnosti LATTICE: `auto` = ciljna vrednost znotraj okvira, ki je prestala kontrolo kakovosti (QA), je absolutna referenca (ciljna vrednost z oznako QA tekmuje z DAQ glede na izmerjeno oceno), nadomestna vrednost DAQ za navzdolno sevanje (ρ = π·L/E); `target` = strogo (brez zamenjave z DAQ); `daq` = DAQ je avtoritativen. Glej [Stikala za izvoz po posameznih izdelkih](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | noben | Imenik **izmerjenih** skenov ciljne odbojnosti na enoto (`<serial>.csv`); v primeru napake se uporabijo nominalni spektri T3/T4P. |
| `--array-alignment / --no-array-alignment` | vklopljeno | LATTICE-matrike: uporabi poravnavo med moduli, vpisano v XMP datoteki `Chloros:Alignment*` vsakega posnetka, za vsak obdelan izdelek (debayering / predogled / sevanje / odbojnost / indeks). Brez delovanja za slike brez oznak. |
| `--array-alignment-crop / --no-array-alignment-crop` | izrez | Izrezajte poravnane izvoze na skupno prekrivno območje matrike, tako da vsi moduli delijo eno območje zajema; `--no-…` ohrani celotno platno senzorja (črno zapolnjevanje zunaj vira). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Ponovno vzorčenje za popravke poravnave. `nearest` ohranja natančne izvorne DN-je (brez mešanja radiometričnih vrednosti med pikslami). |

### Možnosti zaznavanja ciljev

| Oznaka | Opis |
| --- | --- |
| `--min-target-size PIXELS` | Najmanjša velikost kalibracijskega cilja v piksljih za detektor. |
| `--target-clustering 0-100` | Odstotek združevanja ciljev v skupine (privzeto `60`). |
| `--target / --targets` | Iskanje kalibracijskih ciljev samo v podmapah `target` ali `targets` vhodne datoteke (hitreje). |

### Primeri

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a project folder of your choice (the project is named after it)
chloros-cli process "C:/input" -o "C:/projects/Field_A_2026-05-26"

# Name the project and let it go to the default project folder
chloros-cli process "C:/input" -n "Field_A_2026-05-26"
```

### Stikala za izvoz po posameznih izdelkih (LATTICE multispektralno)

Obdelava LATTICE se razširi na **vse ustrezne izdelke v enem prehodu**. Štiri stikala po tipu — `--debayered`, `--preview`, `--radiance`, `--reflectance` — so**privzeto vklopljena**; uporabite obrazec `--no-<type>`, če želite eno izklopiti. Glavne kamere RGB vedno oddajajo le podatke po odstranjevanju barvnih filtrov + predogled (brez sevanja/odbojnosti po posameznih pasovih), zato sta `--radiance`/`--reflectance` zanje brez učinka. Stikala se za `.raw` v Survey3 (ki sledi standardni poti odbojnosti/cilja) ne upoštevajo. *(Stari `--radiometric-output {reflectance,radiance,sensor-response}` je bil **odstranjen** in nadomeščen s temi stikali; raven `sensor-response` ne obstaja več.)*

| Izdelek | Izhod | Potrebno zajemanje podatkov (DAQ) navzdol? |
| --- | --- | --- |
| `--debayered` | Linearni demosaic (`Debayered_Images`): ista datoteka, kot jo zajem shrani na svoji ravni `debayered` — temni odmik (DSNU) kalibracijskega paketa od surovega podatka, nato pa demosaik OpenCV z upoštevanjem robov; brez LUT za linearnost, izravnave polja ali ločevanja barv. | Ne. |
| `--preview` | Predogled na zaslonu (`Preview_Images`): RGB = WB + gama; multispektralno = raztezek v lažnih barvah. | Ne. |
| `--radiance` | float32 W/m²/sr/nm iz celotne radiometrične verige (`Radiance_Images`). | Št. |
| `--reflectance` | uint16 odbojnost ρ (`32768` = 1,0), Pix4D-ready. | **Da**, razen če ga ne pritrdi cilj znotraj okvirja, ki je uspešno prestal kontrolo kakovosti (glej spodaj). |

`--reflectance-source` izbere referenco odbojnosti:**`auto`**(privzeto) določi cilj v okviru, ki je uspešno prestal kontrolo kakovosti, za**absolutno referenco**— verige empiričnih krivulj, pritrjene na cilj, se navzkrižno primerjajo na izločenih panelih, zmagovalna vrednost pa se uporabi pas za pasom: pas, ki ga DAQ ne more kalibrirati ali pri katerem prilagajanje naklona DAQ ni uspešno, prevzame lastno krivuljo cilja (`ReflectanceElBandMethods` navaja, kateri) — cilj, ki opravi kontrolo kakovosti z oznako (izpuščena plošča, pas v napaki), tekmuje tudi z delitvijo DAQ na podlagi te izmerjene ocene, okvir, proti kateremu izgubi, pa se označi z `ReflectanceChain=daq` — če ni cilja ali če QA ne uspe, se uporabi DAQ-jeva razmejitvena krivulja (ρ = π·L/E);**`target`**je strog (brez DAQ-jevega nadomeščanja);**`daq`**se odloči za obnašanje, pri katerem ima DAQ glavno besedo. Geometrija cilja (ArUco / fiksno območje zanimanja / trak) izhaja iz konfiguracije cilja projekta; `--target-reflectance-dir DIR` hrani**izmerjene** skeniranja (`<serial>.csv`), ki se poiščejo po serijski številki/QR ciljne enote, z nominalnimi spektri T3/T4P kot rezervo.

Pot odbojnosti DAQ samodejno razreši **časovno usklajeno navzdolno sevanje**iz posnetega**`.daq`**(DAQ-U/M/E)**ali iz DAQ-M-ovega lastnega `.csv`**, najdenega skupaj s slikovnim gradivom. Če paket kalibracij za posamezno kamero ali DAQ ni shranjen v lokalnem predpomnilniku, ga proces**samodejno prenese iz AWS** ob prvi uporabi (zahteva enkratno povezavo z internetom; shranjen v predpomnilniku pod `~/.chloros/`).**Čigava je ta vrednost?**Vrednost `.daq`, ki jo shrani zajem, je vrednost kamere(-), ki jo opisuje — imenovana v spremnem datoteki `<file>.daq.cameras.json` poleg nje (`lattice capture`, `multi-capture`, `array-capture` in v grafičnem vmesniku; surovi niz posnetkov v svojem manifestu navaja odčitke vsakega člana) — in `process` ga ujemajo**samo s slikami iz teh kamer**. Samostojna kamera brez lastnih odčitkov zato ne more več uporabljati DAQ-ja druge kamere; ji dodelite DAQ ali posnemite enega. Lastni posnetek DAQ-ja (`daq pool-record` ali posnetek, začet v grafičnem vmesniku) ali kateri koli `.daq` brez tega »sidecarja« (vključno s starejšimi zajemi) še vedno služi vsem kameram.

#### Branje pikslov odbojnosti (Pix4D / Metashape / vaši lastni skripti)

Odbojnost je shranjena kot celo število DN, **DN, ki pomeni ρ = 1,0, pa je odvisen od izvorne kamere**:

| Izvor | ρ = 1,0 je | Kako prepoznati |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (rezerva do ρ 2,0) | V datoteko je vtisnjena oznaka XMP `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (omejeno na ρ 1,0) | Ni oznak XMP `Chloros:*` — prav ta odsotnost *je* znak. |

**Preberite `Chloros:PixelScale` in delite z njim**, namesto da predpostavljate konstanto. Oznaka je definirana v domeni uint16, zato ostane `32768` v vseh izhodnih formatih, ki spreminjajo merilo — `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)` in `TIFF (32-bit, Percent)` so vsi samopisni (najprej normalizirajte shranjeni podatkovni tip nazaj na uint16: ×257 iz 8-bitnega, ×65535 iz float).

> **Datoteke iz starejših različic ne smejo vsebovati merila.** 8-bitna TIFF-datoteka z odbojnostjo se ponovno prilagodi in označi z `Chloros:PixelScale`, tako kot ostale. Različice pred 1.3.0 so ga namesto tega *obrezale* na 0..255, tako da je vsaka vrednost nad ρ≈0,008 postala 255, poleg tega pa so izpustili tako `Chloros:PixelScale` kot tudi niz `MicaSense:RadiometricCalibration`. **Če oznaka v datoteki odbojnosti LATTICE manjka, ne predpostavljajte merila — datoteko ponovno obdelajte**, namesto da delite piksle, ki nikoli niso bili deljivi.

#### Branje indeksnih pikslov (indeks NDVI in drugi vegetacijski indeksi)

**Celoštevilčni indeksni raster je raztezanje kontrasta na posamezni sliki, ne pa zemljevid v območju −1…1.** Indeksni izdelki `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` in `JPG (8-bit)` indeksni izdelki – vključno z 8-bitnim TIFFom, v katerem je zapisana 8-bitna zajemna slika – shranjujejo

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

kjer sta `IndexMin` / `IndexMax` **najnižja in najvišja končna indeksna vrednost tega okvirja**. DN 0 ni −1, polna skala pa ni +1.

Program Chloros označi obe skrajni točki, da ostane preslikava obratna:

| Oznaka | Pomen |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — konvencija spodaj |
| `Chloros:IndexMin` | indeksna vrednost, shranjena kot DN 0 |
| `Chloros:IndexMax` | indeksna vrednost, shranjena kot polna skala |

Preberite jih in obrnite, pri čemer najprej normalizirajte shranjeni DN na 0…1 glede na največjo vrednost shranjenega podatkovnega tipa, tako da en izraz pokriva vsak celoštevilčni format:

```python
import json, subprocess
import numpy as np, tifffile

tags = json.loads(subprocess.run(
    ["exiftool", "-j", "-XMP:all", path],
    capture_output=True, text=True).stdout)[0]

# float(): exiftool hands back a negative real as a quoted string, so
# IndexMin arrives as str on most real NDVI data while IndexMax is a number.
lo, hi = float(tags["IndexMin"]), float(tags["IndexMax"])

raw = tifffile.imread(path)                    # uint16 for TIFF (16-bit)
ndvi = lo + raw.astype(np.float64) / np.iinfo(raw.dtype).max * (hi - lo)
```

Za branje ni potrebno `-config pix4d.config` — exiftool te lastnosti XMP bere samodejno. Konfiguracija je potrebna le za njihovo *pisanje*.

> **`IndexMin == IndexMax`** pomeni, da je vsak piksel tega okvirja imel enako indeksno vrednost. Raster je zapolnjen na sredini skale in ni ničesar, s čimer bi se delilo — vzemi končno točko.

> **Če ni oznak `Chloros:Index*`, to pomeni, da ni navedenega preslikavanja.** Izvoz LUT je obarvan render, katerega piksli so barvni prehodi, vezani na vaše pragovno okno LUT, in okvir brez piksla z določenim indeksom nima obsega, ki bi ga bilo treba navesti. V nobenem primeru ne predpostavljajte preslikave – velja isto pravilo kot pri manjkajočem `PixelScale`.**Ker je raztezek določen za vsako sliko posebej, dva okvirja istega poleta nista primerljiva glede na DN**— vsak ima svoje končne točke, zato isti NDVI v različnih slikah pade na različne vrednosti DN. Za kvantitativno delo v okviru enega poleta (statistike po conah, fiksni pragi, časovne vrste) izvozite**`TIFF (32-bit, Percent)`**: indeksni rasterji so tam zapisani kot float32**same indeksne vrednosti**, brez potrebe po ponovnem merjenju ali oznakah. Kljub imenu formata indeksni produkti niso ×100 – to merjenje velja le za odbojnost s celoštevilskim podatkovnim tipom.

#### Maska omejitve ločevanja (na katerih pikslov je bil pas omejen)

Spektralno ločevanje 3×3 lahko povzroči negativno vrednost pasu – kadar svetloba komaj vsebuje ta pas ali kadar prizor ne sodi v območje, ki ga opisuje matrika ločevanja kamere. To ni dejanska vrednost, zato jo izvoz fiksira na 0 – in v živo prikazani pogled kaže isto 0. Pas se lahko omeji na nekaj pikslov ali na skoraj vse; maska zabeleži oboje. `Chloros:Band{R,G,B}UnmixClampFraction` navaja, na kolikem delu slike se je to zgodilo; maska omejitve pa navaja **katere piksle**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— v mapi `clamp_masks/`**poleg** mape izdelka, nikoli znotraj nje (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`; izvoz iz mape `process` v mape `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`) — za vsak produkt sevanja, odbojnosti in indeksa, v katerem je vsaj en piksel omejen: zajemi in izvozi `process` (ter TIFF-datoteke po posameznih okvirjih `array-build-video --save-tiffs`). Enopasovni `uint8`, na lastni pikselni mreži izdelka (vključno z orientacijo, območjem zanimanja (ROI), združevanjem pikslov in poravnavo matrike). Vsaka maska navaja kamero svojega izdelka (`Make` / `Model` / serijsko številko), čas zajema in oznake objektiva ter navaja pasove, ki jih predstavljajo njeni biti (`XMP-Camera:BandName`, najprej bit 0); oznaki `ImageDescription` in `Software` jo označujeta kot masko za omejevanje, kalibracije pa ne vsebuje.
- **Biti**: bit 0 = pas v R-kanalu izdelka (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Nastavljen bit pomeni, da je bila vrednost piksla v tem pasu omejena na 0 – obravnavajte ga in vsak indeks, zgrajen na njegovi podlagi, kot nizko zanesljivega.
- **`Chloros:UnmixClampMask`** je vtisnjen na vsak izdelek, na katerem je bil izveden postopek ločevanja: pot do maske glede na mapo samega izdelka (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, vedno ločena z `/` – združite jo s mapo slike, da odprete masko) ali `none`, če ni bil omejen noben piksel. Če oznaka ni navedena, to pomeni, da na izdelku ni bil izveden postopek ločevanja barv (kamere RGB ali monokromatske, surove/debayered ravni).
- `lattice capture` / `array-capture` natisnita pot do maske pod shranjeno datoteko (`clamp mask: …`), v seznamu rezultatov zajema pa je navedena kot `clamp_mask` posameznega okvirja.
- `lattice index --live --save-multiband` zapiše eno masko na izvorno kamero za večpasovni niz — glej [`lattice index` — maske za večpasovni niz](#lattice-index--multiband-stack-masks). Video nima maske: `array-build-video` namesto tega poroča o deležih omejitve za vsak posnetek (glej [`array-build-video`](#array-build-video--offline-reprocess-a-saved-burst)).

Ker nobena mapa z izdelkom ne vsebuje maske, na nobeni ravni, se mapa z izdelkom (`radiance/`, `reflectance/`, mapa `*_Images/` izvoza `process`) predati programu Pix4D, Metashape ali drugi fotogrametrični programski opremi takšna, kot je — vključno s podmapami — in sprejema le slike. Posredujte mu mapo z izdelkom, ne pa mape nad njo (ta vsebuje tudi `clamp_masks/` in vse druge vrste izvozov). Chlorosov lastni uvoz preskoči maske. Branje maske: `(mask & 1) != 0` pomeni »pas R-kanala je bil tukaj omejen«, `& 2` G, `& 4` B.

#### EXIF se prenese v izvoz

`process` kopira **GPS-blok in njegov ExifIFD** izvirnega posnetka na vsak izdelek, zato
izvoz vsebuje `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` ter
`CameraSerialNumber` skupaj z georeferenciranjem.

**`FocalLength` ni neobvezen za fotogrametrijo.** Pix4D izračuna razdaljo med točkami na tleh (GSD) iz
goriščne razdalje in nadmorske višine; če oznaka manjka, se program zateče k močno napačnemu merilu. Pri enem
letu nad pomarančnim nasadom s 49 posnetki je manjkajoča oznaka spremenila območje velikosti 411 m × 160 m v rekonstruirano
47,8 km × 13 km – ortofoto z 455 MP, ki je v glavnem vseboval »nodata«, kar se je nato obravnavalo kot problem s sestavljanjem ploščic in
problem z datoteko BigTIFF, preden je kdorkoli preveril GSD. Če se vaš ortofoto prikaže v neverjetni
merilu, najprej za izvoženi izdelek zaženite `exiftool -FocalLength`.

Kopija namerno **ni** `-all:all`: strukturne oznake IFD0'njene strukturne oznake pri kopiranju motijo izhod LATTICE,
`ExifImageWidth` / `ExifImageHeight` pa sta izključena, ker opisujeta
*izvorno* zajemanje – izvoz, katerega velikost je bila kdaj koli spremenjena, bi sicer vseboval dimenzije, ki so v nasprotju z lastno rastrsko sliko. XMP se zapiše neposredno in ne kopira, ker ExifTool
pri kopiranju bloka XMP zavrže oznake XMP iz istega klica (kar bi izpustilo MAPIRove
kalibracijske oznake).

### Kje se izhodni podatki shranijo

Izdelki se shranijo **v projektni mapi, razvrščeni po kameri in nato po datotečnem formatu**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Mapa kamere za LATTICE je `LATT-<sensor>-<lens>-F<filter>` (kar ustreza EXIF-u zajema
`Model`) ter `<model>_<filter>` za Survey3 — dve kameri, ki si delita senzor in filter, a se razlikujeta
po objektivu, imata ločeni drevesni strukturi, saj se razlikujeta v vinjetiranju, vidnem polju in popačenju. Mapa z
map je naslednji: `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` ali `tiff32` za
`TIFF (32-bit, Percent)`.

> **Vsak izvožen izdelek ohrani ime izvorne datoteke.** Izvoz sevanja datoteke
> `capture_…_raw.tif` se še vedno imenuje `capture_…_raw.tif` – le da se nahaja v
> `tiff32/Radiance_Images/`. **Izdelek identificira mapa, ne ime datoteke**, zato iskanje po vzorcu
> za `*radiance*.tif` ne najde ničesar; namesto tega poiščite po imeniku.

### Posnetki svetlobnega senzorja — kalibrirani `.daq` + `.csv`

`process` obdeluje tudi posnetke `.daq` v vaši vhodni mapi, za to pa **ne**
potrebuje nobenih slik za to: DAQ-U / DAQ-M / DAQ-E, ki leti samostojno, predstavlja popoln
posnetek, mapa, ki vsebuje le datoteke `.daq`, pa je veljaven vhod.

DAQ se lahko snema **brez** kalibracije – to je tisto, kar javni
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) snemalnikov
(`record_daq.py`) počnejo privzeto: zapišejo surove vrednosti senzorjev in datoteko označijo, tako da
Chloros pridobi tovarniško kalibracijo tega senzorja **po serijski številki** (najprej iz lokalnega predpomnilnika,
nato iz oblaka MAPIR) in jo uporabi. `process` zapiše rezultat nazaj:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` vsebuje eno vrstico na odčitek: časovni žig UTC, čas integracije, skupno moč,
fotopični/skotopični luks, PPFD (in njegovo razdelitev na modro/zeleno/rdeče), najvišjo valovno dolžino, nato pa
celotno spektrum na lastni valovni mreži senzorja. `.daq` se ponovno uvozi, ne da bi ga bilo treba
ponovno kalibrirati.

`<name>` je ime datoteke posnetka. Dva posnetka v različnih mapah lahko delita
eno – odčitke na posnetek (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`), shranjene v
isti milisekundi z dvema kamerama – in vsak še vedno dobi svoje izdelke: prvi ohrani
`<name>`, naslednji pa doda mape, ki ga ločujejo, **za** imenom
(`<name>__<capture>_daq_calibrated.daq`), tako da datum in čas posnetka še vedno stojita na začetku.

Ob uspešnem izvajanju se poroča o `Light-sensor products written: N (calibrated .daq + .csv)`.
V oklepaju je opisano, kaj je bilo dejansko zapisano, torej se glasi
`(RAW COUNTS — this sensor has no calibration bundle)` za senzor brez svežnja in
`(N calibrated, M raw counts)` za mapo, ki vsebuje oboje. Naslovi samega
backenda, kot sta `[DAQ-EXPORT]` in `[RUN-SUMMARY]`, izhajajo iz istega načina oblikovanja besedila – nobeden od
teh treh ne more označiti surovega izvoza kot kalibriranega.

Zapis DAQ-U / DAQ-M / DAQ-E, katerega kalibracijski paket ni mogoče pridobiti – ker ste
brez povezave ali ker za ta senzor ni shranjene kalibracije – je **preskočen z navedbo razloga** v
vrstici `[DAQ-EXPORT]`, nikoli pa ni shranjen kot »kalibrirana« datoteka z neobdelanimi števili.
Povežite se z internetom in ponovno zaženite. Razlog je tisti, ki ga je bralnik dejansko
ugotovil za to datoteko (neberljiva shema, ni paketa, napaka pri pisanju), povzetek
izvajanja pa navaja **različne** razloge – dvajset preskočenih datotek zaradi enega vzroka se prikaže kot en
vzrok, ne pa kot dvajset ponovitev istega vzroka. CLI na koncu
izvajanja navede vsak preskočen posnetek z razlogom:

```
⚠ Light-sensor recording not exported: <file>.daq: no calibration bundle available for …
```

Mapa z zapisi, iz katere se **ni** izvozil noben, ne uspe: glej
[Izvedba, ki ne zapiše nobene slike, ne uspe](#a-run-that-writes-no-images-fails).

Posnetek, katerega ime se konča z `_calibrated` ali `_raw` (`field_raw.daq`), **ni**: to so imena, ki jih `process` dodeli svojim lastnim izdelkom, in izdelek, ki se prebere nazaj
kot posnetek, bi se štel dvakrat. CLI to tudi navaja in imenuje datoteko —
`Not imported as a light-sensor recording: field_raw.daq … Rename the file to import it.`
— namesto da bi poročal o prazni mapi. Takšen posnetek pred obdelavo preimenujte.

#### Izvoz posnetkov DAQ-A kot surovih števcev

Družina **DAQ-A** je starejša od sistema paketov po serijskih številkah in nima kalibracijskega paketa,
ki bi ga bilo treba pridobiti – namesto tega se kalibrira na terenu z uporabo cilja odbojnosti, zato
ga nikoli ni potrebovala. Zavrnitev teh posnetkov jim je onemogočila, da bi sploh
izvozili svoje podatke, zato se izvažajo pod **drugo ime**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Drugo ime datoteke namesto oznake znotraj datoteke, saj mora oznaka ostati ohranjena
tudi pri pošiljanju po e-pošti kot golo ime. Zgornji del datoteke `.csv` navaja
`raw spectral sensor counts (NOT irradiance)` in opozarja, da so vrednosti primerljive
**znotraj** datoteke – kar je točno tisto, za kar jih uporablja kalibracija na podlagi cilja – in
ne med senzorji. Fotometrični stolpci, odvisni od moči (skupna moč, fotopični in
skotopični luks, PPFD), so zapisani kot **NULL** namesto da bi bili integrirani iz števila impulzov, v povzetku
izvajanja pa je navedeno `RAW COUNTS`, zato »izvoženega« v dnevniku ni mogoče razumeti kot obsevanje.

Starejši zapisi **v1.01 / v1.02** (te jih zapiše DAQ-A-SD) ne vsebujejo epohe za vsak odčitek,
temveč le čas zapisa datoteke. Program za ujemanje slik↔navzdolnega sevanja jih še vedno zavrača – ujemanje
okvirja z časom zapisa bi bilo nevidno napačno –, vendar jih izvoznik prebere,
CSV-datoteka pa izpiše `clock=daq_created_on`, tako da izdelek navaja, na kateri uri teče.

### Opombe

- `process` samodejno zazna, ali je vaša mapa tipa Survey3, »LATTICE« ali mešana.
- Napredek se prenaša prek dogodkov Server-Sent Events; CLI prikazuje napredek v realnem času po posameznih niti (zaznavanje, analiziranje, obdelava, izvoz).
- Pri Linuxu/Jetsonu CLI preveri zamenjavo pomnilnika in lahko izda opozorilo pred obdelavo velikih map. Debayer, ki upošteva teksturo, samodejno uporabi omejitev frekvence GPU na Jetsonih z nizko porabo energije (Nano, Orin Nano).
- Ob uspešnem zaključku izvajanja se prikaže poročilo o številu zapisnih slikovnih izdelkov (`Image products written: N`, v primeru, da je razmešavanje omejilo kateri koli piksel, pa `(plus M clamp masks)`).
- **`process` varno deli zagnani backend.** Že zagnani backend samodejno ponovno zažene le, če je ta backend v mirovanju, ima naložen prejšnji projekt in ne vsebuje kamer, matrik ali DAQ-jev. Backend, ki vsebuje katero koli od navedenih (grafični vmesnik, ki pretaka matriko, skript SDK, snemanje DAQ), se uporablja tak, kot je, in se nikoli ne ponovno zažene, zato lahko `process` teče vzporedno z aktivno matriko. Če backend poroča o tekočem izvajanju, `process` počaka do 15 s, da se ta zaključi (za izbris pravkar zaključenega izvajanja je potrebna približno sekunda), nato pa zavrne z napako `Backend is mid-run (processing). Wait for it to finish, or pass --restart to force a fresh backend (this kills the in-flight run).`. Globalni zastavici `--restart` še vedno prisili zagon novega backenda in zaključi vse seje kamer, nizov ali DAQ na njem. Backend, ki ga izvaja storitev systemd v sistemu Linux, se nikoli ne ponovno zažene: `process` ga uporablja, `--restart` pa namesto tega izpiše `sudo systemctl restart <unit>` (glej [Zagon backenda kot storitve](#running-the-backend-as-a-service-headless)). Prav tako ni lastni backend aplikacije Chloros: ko je aplikacija odprta, ga uporablja `process` in namesto projekta aplikacije naloži svojega (backend, zagnan na njegovem mestu, bi odprto aplikacijo izključil iz njega).
- **Zagon v teku ohrani svoj projekt.** Medtem ko poteka obdelava — ki jo je zagnala aplikacija, `process`, skript SDK ali `chloros-mcp` — backend ne dovoli ustvarjanja, odpiranja, uvoza v projekt, dodajanja ali odstranjevanja datotek iz projekta, premikanja, izbrisati ali spreminjati nastavitev obdelave projekta, pri čemer vrne HTTP-odziv 409 `PROCESSING_IN_PROGRESS` in `A processing run is in progress on project '<name>', so ... now is refused`; `process`, ki to doseže, zapre `1`. Nastavitve prikaza (velikost sličic) se še vedno shranijo. Na enak način se zavrnejo tudi način obdelave in nastavitve ciljne odbojnosti projekta (izvedba ohrani način, v katerem se je začela). Posnetek, shranjen v projekt med izvajanjem (iz aplikacije, CLI, SDK ali `chloros-mcp`), se shrani kot običajno, vendar se pridruži seznamu datotek projekta šele ob koncu izvajanja; tako navaja tudi odgovor (`project_add`), enako velja za snemanje svetlobnega senzorja, ki je bilo ustavljeno v projektu. Prej je lahko drugi odjemalec med izvajanjem preklopil odprt projekt, pri čemer se je tekoče izvajanje izvozilo v drugi projekt.
- **Ctrl+C ustavi izvajanje v ozadju, ne le v CLI-ju.** CLI zaprosi ozadje, naj ustavi obdelavo, izpiše `Waiting for the backend to stop (Ctrl+C again to stop waiting)...` in čaka do 120 s na `Processing stopped.`, da `chloros-cli process … && next` ne bi zagnal `next` na strežniku, ki je še vedno zaseden z izvajanjem. Drugi pritisk na Ctrl+C prekine čakanje. `process` nato zaključi `130`, kot to stori vsak prekinjen ukaz.

#### Izvajanje, ki ne zapiše nobenih slik, se ne uspe

Če ste zahtevali izdelke, izvedba pa ni zapisala **ničesar** — le `project.json` in
`calibration_data.json` — `process` to obravnava kot napako: izpiše
`Processing finished but wrote no image products.` in **se zaključi z vrednostjo, ki ni nič**, tako da ga skript lahko
zazna. Sporočilo navaja ime projektne mape in običajne vzroke:

- vhodna mapa ni bila prepoznana kot zajem (preverite razpored in `--input-level`) ali
- so bili vsi zahtevani izdelki preskočeni, ker niso bili primerni za te kamere (npr. zahteva po
  sijaj/odbojnost iz kamer, ki podpirajo le RGB).

Ponovno zaženite s `--verbose` in v dnevniku ozadja preverite vrstice `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
ki pojasnjujejo preskoke po posameznih kamerah, ki sicer ne pridejo v izhod CLI.

Namerno izvajanje, namenjeno izključno metapodatkom – vsi izdelki so izklopljeni in ni `--indices` – je še vedno
**uspešno**, saj je v tem primeru prazen izhod slike pravilen rezultat.

#### Izvajanje, pri katerem izvozov ni bilo mogoče preveriti, se ne uspe

Po izvedbi `process` preveri oznake izvoženih datotek z ExifToolom. Če se ExifTool ni mogel zagnati,
izpiše `Preverjanje izvoza NEUSPEŠNO: ExifTool se ni mogel zagnati, zato oznake N od M izvoženih datotek
niso bile preverjene (glej vrstice [EXIFTOOL] v dnevniku ozadja)` and `Rezultati so zapisani in ostanejo
na mestu; za preverjanje jih ponovno zaženite.`, does not print `Obdelava zaključena!` ter **se zaključi z vrednostjo, ki ni nič**.
Rezultati se ohranijo.

#### Kateri posnetki so vključeni v izvedbo

Vključeni so posnetki, ki so bili zaključeni pred izvedbo `process`. Posnetki, shranjeni med izvedbo, niso vključeni; `process`
opozori, koliko jih je bilo izpuščenih, ponovna izvedba pa jih vključi.

#### Izvedba, pri kateri je bila izgubljena slika, se ne uspe

Če je slika med izvajanjem spodletela, `process` izpiše `N image(s) failed and were not exported` z
najpogostejšim vzrokom, ne izpiše `Processing complete!` in **zaključi z vrednostjo, ki ni nič**. Ostali izdelki so
zapisani in ostanejo na svojem mestu, v dnevniku strežnika pa so zabeležene vse napake.

Enako velja za **izvedbo, ki uporablja le svetlobni senzor**: mapa z zapisi `.daq` po definiciji ne vsebuje slik za izvoz,
zato se izvedba ocenjuje na podlagi kalibriranih zapisov `.daq` / `.csv`, ki jih je namesto tega zapisal.
Če ni zapisal **ničesar** — so bili vsi posnetki preskočeni — izpiše
`Processing finished but wrote no light-sensor products.`, vsakemu posnetku dodeli ime z
razlogom, zakaj je bil preskočen, in **zaključi z vrednostjo, ki ni nič**.

---

## `chloros-cli login`

Avtentificirajte ta računalnik z oblačnim računom Chloros+. Pristojnosti so varno shranjene v `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Primeri

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (odstranjevanje ali podvajanje delov gesla). Ob napaki 401 CLI samodejno ponovi poskus z `$$`, pri čemer-priloženo, nato pa s polovico gesla brez podvajanja; če je ponovni poskus uspešen, vas prijavi in izpiše pravilno sintakso z enojnimi narekovaji, ki jo morate uporabiti naslednjič.

> **Uporaba brez grafičnega vmesnika/s skripti: odsotnost shranjene seje pomeni poziv za prijavo.** Osnovni ukazi (`process`, `status`, `export-status`, `time-sync`, …) brez shranjene seje zahteva `Email:` / `Password:` na stdin, preden nadaljuje. Če je stdin zaprt (cron, enota systemd), se program takoj ustavi z napako `login cancelled`; če je stdin odprt terminal ali cev, čaka na vnos. Pred načrtovanjem brezzaslonskega dela enkrat na vsakem računalniku zaženite `chloros-cli login EMAIL PASSWORD`.

---

## `chloros-cli logout`

Izbriše shranjeno sejo v predpomnilniku in pri naslednjem klicu prisili uporabnika k novi prijavi.

Odjava med potekom obdelave te ne ustavi: obdelava se zaključi v načinu, v katerem se je začela, naslednja obdelava pa poteka v standardnem načinu. To navaja `logout` (`The processing run in progress finishes in <mode> mode; the next run uses standard`), prav tako pa tudi aplikacija, ko se odjavite tam odjavite.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Prikaže trenutno stopnjo licence (Iron/Copper/Bronze/Silver/Gold), avtentificiranega uporabnika in število vezav na naprave.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Preveri trenutni napredek izvoza Thread-4 v realnem času. Klic je varen **med** izvajanjem `process` iz drugega lupinskega okna.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Nastavi jezik prikaza v CLI-ju (podpira 38 jezikov, vključno s CJK, RTL in indijskimi jeziki). Na starejših konzolah, ki ne morejo prikazati skripta, se brez težav preklopi na angleščino.

```
chloros-cli language [LANG_CODE] [--list]
```

### Primeri

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Ukazi za mapo projekta

Ti ukazi upravljajo privzeto mesto mape projekta (skupno z grafičnim vmesnikom).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

`get-project-folder` izpiše mapo, v katero se shranjujejo novi projekti. Če konfigurirana mapa ne obstaja več (je bila izbrisana, preimenovana ali se nahaja na pogonu, ki ni priključen), CLI, tako kot grafični vmesnik, preklopi na privzeto mapo: privzeta mapa se izpiše na stdout, opozorilo z imenom manjkajoče mape se pošlje na stderr, izhodna koda pa je `0`.

---

## `chloros-cli update`

Samo za Linux/Jetson. Preveri `version_url` iz `/etc/chloros/update.conf` in, če obstaja novejša različica, prenese ustrezno `.deb` in jo takoj namesti z `sudo dpkg -i` (brez poziva za potrditev; `sudo` lahko zahteva vaše geslo). Na napravi Jetson je posodobitev na voljo le za sistem JetPack 6. V sistemu Windows ta ukaz ne izvede ničesar in prikaže sporočilo `up to date`: posodobite z novim namestitvenim programom.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

V sistemu Linux/Jetson CLI ob vsakem zagonu izvede tudi **avtomatsko preverjanje posodobitev** (ne blokira, nikoli ne upočasni ukaza): prebere `/etc/chloros/update.conf`, rezultat shrani v predpomnilnik za 1 uro v `~/.chloros/update_cache.json` in izpiše `Update available: vX.Y.Z / Run: chloros-cli update`, če obstaja novejša različica. V primeru kakršne koli napake in v sistemu Windows se to tiho preskoči; `CHLOROS_DISABLE_UPDATE_CHECK=1` to izklopi. Zaradi 1-urne predpomnilnika se lahko različica, objavljena v tej uri, še vedno prikaže kot posodobljena.

---

## `chloros-cli selftest`

Izvede 7-stopenjski hitri test: različica, razpoložljivost vrat, zagon backenda, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), prisotnost modela za odstranjevanje šuma, pripravljenost CUDA+odstranjevalca šuma. Uporablja že zagnan backend ali ga za test zagnati in ga nato ustavi. Izpiše `N/7 checks passed` in neuspešne preverjanja ter zaključi `0` šele, ko je vseh sedem uspešnih; preverjanje CUDA+odstranjevalca šuma ne uspe na računalniku brez CUDA GPU-ja (glej [Preverjanje namestitve](#verify-the-install)).

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

Stanje in nadzor PTP grandmastera. Gostitelj Chloros izvaja PTP grandmaster; kamere LATTICE in enote DAQ-E delujejo kot podrejeni napravi za zagotavljanje časovnih žigov med napravami (približno 1 ms med napravami). PTP usklajuje **časovne oznake**: kamere v nizu posnamejo skupaj, ker jih sproži strojna sprožilna naprava M8 (manj kot 100 µs med moduli), ne zaradi PTP. `chloros-cli time-sync` brez podukaza zažene `status`.

| Podukaz | Opis |
| --- | --- |
| `status` | Prikaže stanje glavnega strežnika, prioritete BMCA, identiteto ure in ali njegovi paketi dosegajo omrežje: `health` je `OK` (`STOPPED`, če grandmaster ne deluje) ali `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` skupaj z `health_detail`, nato pa se v stderr izpiše vrstica `WARNING:` (stdout ostane v formatu JSON). `sync_sent` / `announce_sent` štejejo le sporočila, ki so bila poslana; `*_send_errors` in `last_send_error` pa tista, ki niso bila poslana. Ko se povezava s kamero prekine in se ponovno vzpostavi, grandmaster samodejno ponovno vzpostavi svoje vtičnice (`rebinds`, `last_rebind`). |
| `peers` | Seznam podrejenih naprav, zaznanih prek Delay_Req (kamere + senzorji DAQ-E). |
| `cameras` | Stanje PTP po posameznih kamerah (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Ponovni zagon procesa glavnega strežnika. |
| `set-priority --priority1 N --priority2 N` | Prepis prioritet BMCA. |

Glavni strežnik pošilja obvestila prek omrežnih kartic kamer (na katerem koli adapterju z naslovom `169.254.x.x`). Če ni nobenega na voljo, se zateče k adapterju privzete poti le, če je ta žični, nikoli pa k Wi-Fi ali virtualnemu adapterju (Tailscale, WSL), saj nobena kamera ali DAQ-E ne stoji za njim. Če ni ničesar za objaviti, deluje brez vtičnic (`health` `NO_SOCKETS`), naslednja kamera ali DAQ pa ga ob povezavi ponovno vzpostavi na omrežnem vmesniku, ki je takrat aktiven; `time-sync restart` to opravi takoj.

Grandmaster teče v lastnem procesu, poleg backenda (`own_process` / `process_pid` v `time-sync status`), z visoko prioriteto v sistemu Windows (`process_priority`), zato zaseden backend (obdelava, izvoz, več ukazov hkrati) ne more zamakniti časovnih oznak; prej je ena taka zamuda lahko hkrati premaknila uro vsake kamere. Ta proces se konča z backendom; če ga ni mogoče zagnati, grandmaster teče znotraj backenda kot prej (`own_process` false, kar je zabeleženo tudi v dnevniku backenda).

### Primeri

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

Upravljanje kamer LATTICE. Vsak podukaz poteka prek backenda Chloros; backend upravlja z naborom kamer, zato naslednji klici prek CLI ponovno uporabijo isti odprti ročaj.

### Pogoste možnosti (skupne večini podukazov)

| Oznaka | Opis |
| --- | --- |
| `-d, --device N` | Indeks kamere (privzeto: 0). |
| `-s, --serial SN` | Posebna serijska številka; prevlada nad `--device`. |
| `--serials SN1,SN2,…` | Serijske številke, ločene z vejico, za delovanje z več kamerami. [Prehodni ukazi za več kamer](#multi-camera-transient-sessions) odprejo le te kamere, pri čemer prva sproži sprožilec. |
| `--all` | Deluje na vsaki odkriti kameri. |
| `--exposure US` | Čas osvetlitve v mikrosekundah. |
| `--gain DB` | Ojačanje v dB. |
| `--pixel-format FMT` | npr. `BayerRG8`, `BayerRG12`; `Mono8`, `Mono12` na M3M. Ime določa globino barvne globine znotraj družine kamere: na M3M ime Bayer določa mono format njegove globine (`BayerRG8` določa `Mono8`), na M3C pa ime Mono določa ime Bayer (`Mono12` določa `BayerRG12`). |
| `--width N` / `--height N` | Dimenzije slike. |
| `--preset {default,high_quality,high_speed,triggered}` | Uporabi prednastavljene nastavitve: format pikslov, osvetlitev, ojačanje in hitrost snemanja (`high_speed`: `BayerRG8`, 5 ms, 6 dB, 60 fps; `high_quality`: `BayerRG12`, 20 ms, 0 dB; `lattice presets` jih navaja). Na M3M je format mono z enako globino: `high_speed` zajame `Mono8`, `high_quality` ter `triggered` `Mono12`. Z njim podan zastavek (`--exposure`, `--gain`, `--pixel-format`) prevlada nad prednastavitvijo, nastavitev, ki ni navedena v prednastavitvi, pa ohrani lastno nastavitev kamere. Vsi prostidelujejo, razen `triggered`, ki kamero aktivira za strojni rob na liniji 2 — če ta linija ni aktivirana, bo kamera čakala v nedogled, namesto da bi posnela. |
| `-o, --output DIR` | Izhodni imenik (privzeto: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Velikost paketa GVSP. `auto` izvaja ICMP+GVSP-preizkuse; `jumbo` = 9000; `standard` = 1500. |

### Potek prvega povezovanja kamere LATTICE

```bash
# 0. A backend must be running: the Chloros desktop app (Windows),
#    `chloros-backend`, or the Linux service (see "Start the Backend")

# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Full sensor (no binning, no ROI); the frame rate falls to
#        what the host and wire deliver, and the faster options are
#        printed
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Referenca podukazov

#### Odkrivanje in informacije

| Podukaz | Namen |
| --- | --- |
| `lattice info` | Seznam povezanih kamer (proizvajalec, model, serijska številka, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analiza gostiteljskega sistema za optimalno konfiguracijo kamere. `--no-discover` preskoči odkrivanje kamer (hitrejše, analiza samo prek omrežne kartice). |
| `lattice network [--fix] [--estimate] [--cameras N] [--pixel-format FMT]` | Preverjanje/popravljanje nastavitev omrežne kartice (v sistemu Linux tudi sprejemni pomnilniki vtičnic jedra — glej [Linux gostitelji](#linux-hosts--socket-receive-buffers)); ocena pasovne širine/FPS. |
| `lattice network-analysis --master SN [--slaves SN1,SN2,…] [--width N] [--height N] [--pixel-format FMT] [--binning {1,2,4}] [--force-tier TIER] [--packed {auto,on,off}] [--backend-url URL] [--json]` | Priporočilo za omrežno zmogljivost backenda s stabilno shemo-zmogljivost + priporočilo za matriko (vrne `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `--width` / `--height` privzeto uporabljata celoten senzor, `2048` × `1536`. Izstopi iz `0` za `ok`, `auto_capped_fps` ter `auto_shrunk`, ter `1` za `needs_force_slip` in `error`, z ali brez `--json`. Backend doseže na `--backend-url` (privzeto `http://127.0.0.1:5000`), ne pa na `CHLOROS_BACKEND_URL`. `auto_capped_fps` ohrani zahtevano ločljivost, vendar omeji ciljno število sličic na sekundo (fps) — preberite `recommended.recommended_target_fps` in ga posredujte kot cilj povezave; obravnavajte ga kot uspeh, ne pa kot napako. `auto_shrunk` se pojavi le, kadar zahtevanega okvirja ni mogoče dostaviti pri nobeni hitrosti snemanja; povezava nikoli sama po sebi ne pomanjša okvirjev. |
| `lattice analyze-array [--models M1,M2,…] [--binning {1,2,4}] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--packed {auto,on,off}] [--json]` | Analiza »kaj če« brez odpiranja kamer (`--width` / `--height` privzeto `2048` × `1536`). Tako ta kot tudi `network-analysis` napovedujeta, kaj bo connect prenašal: brez `--pixel-format`, veljajo formati, ki jih določi »connect« za posamezno kamero (Mono12 / BayerRG12), pri `--packed auto` (privzeto) pa se uporabi odločitev, ki bi jo »connect« sprejel na tem računalniku glede [zgoščenega 12-bitnega prenosa](#packed-12-bit-transport---packed), ki bi jo Connect sprejel na tem računalniku — tako da je navedena vrednost fps hitrost, s katero deluje niz. Oba izpisujeta odločitev o pakiranju. **`--n-active` je skupno število kamer na omrežju, ne le tega niza**— povečajte ga, kadar samostojne kamere prenašajo podatke sočasno ali če je zmogljivost omrežja izračunana na podlagi povpraševanja, ki jih ne upošteva v celoti (privzeto: `len(--models)`). Vedno izpiše skupne vrstice `Wire budget:` (zahtevani MB/s v primerjavi z zgornjo mejo brez kolizij) in `Max cameras:` ter označi `** OVER-SUBSCRIBED**`, kadar niz prekomerno obremenjuje omrežje — glej [Model fps in burst za niz](#array-fps--burst-model). |
| `lattice gpu` | Prikaži stanje GPU. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Preveri ali posodobi firmware kamere. Lokalna izbira `.fwa` je fiksna: datoteka v `firmware/<MODEL_PREFIX>/`, ki ustreza `MIN_FIRMWARE_VERSION` zgradbe, se naloži, če je prisotna (samo najnovejša različica kot rezervna možnost), zato je novejša slika proizvajalca, shranjena na disku, neaktivna, dokler se ta pripet ne spremeni — namerno novejše izdaje pridejo do enot prek podpisanega AWS-manifesta, kar je prednostno, če je novejši. |
| `lattice presets [--apply NAME]` | Prikaži ali uporabi prednastavitve kamere. `--apply` določi, kaj imena prednastavitev (njihov format pikslov, osvetlitev, ojačanje, hitrost snemanja, sprožilec) pomenijo, in izpiše te nastavitve, kot jih kamera bere po: na M3M, mono format globine prednastavitve; hitrost snemanja, s katero deluje, ki je lahko nižja od tiste, ki jo zahteva prednastavitev (`high_speed` zahteva 60 fps; polnoformatna kamera na povezavi 1 Gbps deluje s približno 35). Vse ostale nastavitve, ki jih je zapisal program »open«, se vrnejo v prvotno stanje. `--apply default` ponastavi kamero na tovarniške privzete vrednosti: tovarniške nastavitve s prosto delujočo avtomatsko osvetlitvijo na Line2, omejitev prepustnosti povezave in izklopljeno popravljanje napak, ohranjena kalibrirana bela bilanca kamere RGB (enotna na vseh drugih). V pomnilnik kamere se nič ne shrani, zato se ob naslednjem vklopu zažene iz lastnega pomnilnika, kot prej. Pri zajemanju ali povezovanju `--preset` in `default` ni prednastavljenih vrednosti. |
| `lattice status` | Stanje kamere v živo. |

#### Snemanje

| Podukaz | Namen |
| --- | --- |
| `lattice capture [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--no-daq] [--daq-tolerance-ms MS]` | Posamezni okvir. **Privzeto shrani vse vrste izvoza**(`--processing all`); glej [Ravni izvoza zajema](#capture-export-levels-the-all-default). `--processing` sprejema `all`, `raw`, `debayered`, `radiance`, `reflectance` ali `display` (`preview` je sprejet za `display`); vsako drugo ime se zavrne, preden se dotakne kamere (izhod `1`). `--levels` shrani eksplicitno podmnožico (prepiše `--processing`). Kamera, ki je nChloros, se odpre za zajem, njena osvetlitev pa se najprej nastavi glede na prizor (največ nekaj sekund), tako kot to počnejo [prehodni ukazi](#multi-camera-transient-sessions); ko se zajem konča, ne glede na to, kako se konča, se ta kamera vrne v stanje, v kakršnem je bila najdena (format, osvetlitev, barvni potek, vsaka nastavitev, ki jo je zajem spremenil — seznam pod [Večkamere (prehodne seje)](#multi-camera-transient-sessions)). DAQ, ki bere uporabljeno zajemanje, se shrani kot sidecar `.daq`, vključno z zajemom, ki vsebuje le surove podatke; `--no-daq` / `--daq-tolerance-ms` ga prilagodita ([DAQ, ki bere zajem, shrani](#the-daq-reading-a-capture-saves)). `--force-daq` je še vedno sprejet. `--jpeg-quality` = kakovost JPEG 1–100 (privzeto 95).**Datoteke, shranjene v odprtem projektu, se mu pridružijo.** Tako kot v aplikaciji, zajem, katerega datoteke pristanejo v mapi projekta, odprtega v Chlorosu, jih doda na seznam datotek tega projekta; datoteke, shranjene drugje, ostanejo nedotaknjene. Ukaz izpiše eno vrstico, ki to navaja (`Project: ...`), ali pa, če je obdelava v teku, da so zadržane in se pridružijo, ko se obdelava konča. Enako velja za `multi-capture`, `array-capture` (tudi `--smart`) ter `project capture`. |
| `lattice continuous [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--queue-depth N] [--no-daq] [--daq-tolerance-ms MS]` | Prenos na disk do pritiska na Ctrl+C. Okvirji izvajanja `tiff`so v datoteki `raw` zapisi, ki jih zajame ta kamera: mozaik senzorja, ena ravnina z lastnimi vrednostmi, nekomprimirana, s kalibracijskim paketom (2048 x 1536 12-bitni okvir je 6,0 MiB plus paket, ne glede na prizor), zato `process` začne z njimi. `png` in `jpg` vsebujeta 8 bitov, zato sta ti zapisi slike z odstranjenim mozaikom za ogled, od katerih se `process` ne začne. Pri vezavi na DAQ (`daq pool-bind`) ali združenim, vsak okvir dobi svoj `.daq` in `.match.json` v mapi `daq/` serije, saj `lattice capture` enega shrani; `--no-daq` to izklopi, ob ustavitvi pa se izpiše, koliko jih je. Vsak posnetek se zapiše enkrat, skupaj s polnimi metapodatki; ob pritisku na Ctrl+C se že zajeti okvirji zapišejo, preden se kamera sprosti. Program se samodejno ustavi, ko se pogon zapolni, izpiše `Failed:` / `Disk full:` in zapre program `1`; vsak posnetek, ki ga ni mogel zapisati, prav tako zapre program `1`. Kamera, ki ostane pripravljena za sprožilec strojne opreme (`TriggerMode` `On`) je za ta cikel nastavljena na samostojno delovanje. Ko se cikel konča, se kamera vrne v stanje, v kakršnem je bila najdena — njene nastavitve sprožilca in vse nastavitve, ki jih je cikel spremenil (osvetlitev, ojačanje, format, …) — razen v primeru `--keep-settings`, ki nastavitve snemanja shrani v lastni pomnilnik kamere (prosti tek), ali `--clear-settings`, ki ta pomnilnik ponastavi na tovarniške nastavitve. Zagon, ki ne posname nobenega okvirja, je neuspešen: če 10 sekund ni nobenega okvirja, se samodejno ustavi, natisne `No frames:` z razlogom in se zaključi z `1`. Okvirji, s katerimi zapisovalniki niso uspeli držati koraka, se štejejo za izgubljene, pri zaustavitvi pa se izpiše `Dropped:` z razlogom (hitrost snemanja kamere, hitrost zapisovalnikov in čakalna vrsta); nižja hitrost snemanja ali 8-bitni format pikslov jih ohrani vse. Ukaz se vsake 2 s prijavi pri backendu; če se ustavi, ne da bi prekinil snemanje (`kill -9`, zmanjkalo pomnilnika), ga backend ustavi 30 s po zadnji prijavi in kamero vrne (`stop_reason` `client_gone` ter vrstica v dnevniku strežnika). `multi-stream` in `align-stream` delujeta enako. Zagon, pri katerem kamera potrebuje več časa za odprtje, kot traja čakanje ukaza (60 s; 180 s za `multi-stream` / `align-stream`) — kamera se še vedno zaganjaja — backend prekliče: kamera se vrne v prejšnje stanje in zapre, snemanje pa se ne nadaljuje (504 HTTP `CLIENT_DEADLINE_PASSED`). |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Predogled v živo v formatu MJPEG v brskalniku: vzpostavi povezavo s kamero, izpiše URL tok in ga odpre v vašem brskalniku. `--ae-damping` nastavi dušenje avtomatske osvetlitve (0,4–100). |

##### Živi tokovi (MJPEG; brez RTSP)

Chloros prenaša živo video kot **MJPEG prek HTTP**(`multipart/x-mixed-replace`, meja `frame`) iz strežnika.**RTSP ni podprt.** Odprite URL v brskalniku ali ga preberite s katerim koli MJPEG odjemalcem (`cv2.VideoCapture(url)` iz OpenCV, ffmpeg, VLC). Kamero je treba najprej priključiti v ozadju (`lattice cam-connect`, `lattice array-connect`, `lattice viewer` ali prek grafičnega vmesnika).

| URL (`http://127.0.0.1:5000` + …) | Tok |
| --- | --- |
| `/api/camera/stream/<serial>` | Predogled zaslona, do približno 30 sličic na sekundo. Za element matrike, `?max_h=` (privzeto `720`) in `?jpeg_q=` (privzeto `80`) nastavita višino in kakovost JPEG. To je URL, ki ga odpre `lattice viewer`. |
| `/api/camera/<serial>/stream-annotated` | Enak predogled z vrisanimi prekrivnimi elementi grafičnega vmesnika: `zebra=1`, `crosshair=1`, `grid=3x3` (ali `9x9`), `histogram=1`, `peaking=1`, `spot=1`; brez poizvedbe se uporabijo nastavitve prekrivnih plasti, ki so za to kamero shranjene v projektu. |
| `/api/camera/<serial>/stream-reflectance` | 8-bitna odbojnost na pas: ρ = piksel ÷ del`X-Chloros-Pixel-Scale` (255, razen če je sprožen `refl_max`). `fps` (privzeto `10`), `jpeg_q` (privzeto `95`), `max_h` (privzeto `0` = izvirno), `band=<name>`, `align=1`. JPEG, torej s izgubo kakovosti. Zavrnjeno s HTTP-kodom 409, če kamera nima odbojnosti (RGB-kamera, brez kalibracijskega paketa, brez odčitka DAQ). |

- **Usklajevanje posnetkov med kamerami.**Vsak del, ki ga pošlje**član niza**, v vseh treh tokovih vsebuje glave `X-Chloros-Group-Id` (števec sinhronizirane skupine niza), `X-Chloros-Frame-Id` in `X-Chloros-Timestamp-Ns`. Deli iz različnih kamer z istim `X-Chloros-Group-Id` izhajajo iz istega sprožilca. Samostojna kamera nima skupine in ne pošilja identifikatorja skupine. Brskalniki prezrejo glave posameznih delov; odjemalec, ki sam analizira večdelni tok, jih prebere.
- **Zaustavljeno, dokler je odprt pogled na niz.** `/api/camera/stream/<serial>` člana niza pošilja statični okvir »paused«, dokler je odprt združen živi tok niza (na primer pogled na niz v grafičnem vmesniku), in se nadaljuje, ko se ta zapre.

#### Nastavljanje senzorjev

| Podukaz | Namen |
| --- | --- |
| `lattice configure [{--get N1 N2… \| --set N=V N=V… \| --dump}] [--json]` | Branje/pisanje katerega koli vozlišča GenICam. `--set ExposureTime=…` (`Gain=…`) brez svojih stikal `ExposureAuto` (`GainAuto`), ki najprej izklopijo avtomatski način, kot to stori `lattice exposure --set`; vrednost, ki jo kamera zavrne, natisne `Failed to set <node>: …` na stderr in zaključi `1`. `--get` in privzeti združeni pogled izpišeta `note:` pod `ExposureTime` / `Gain`, medtem ko ta avtomatski način teče — glej [Branje osvetlitve in ojačitve v avtomatskem načinu](#reading-exposure-and-gain-under-auto-mode). |
| `lattice exposure [{--auto \| --auto-once \| --off \| --set US}] [--brightness 0-255] [--brightness-auto] [--damping 0.4-100] [--upper-limit US]` | Izpostavljenost in AE. Izpostavljenost `--set` ali ciljna vrednost `--brightness` se ohrani kot vaša (avtomatski nadzor osvetlitve Chlorosa ga ne spreminja); ojačanje ostane takšno, kot je bilo, zato se v avtomatskem načinu nenehno prilagaja cilju, dokler ne nastavite `lattice gain`. `--auto` vrne nadzor nad osvetlitvijo, `--brightness-auto` pa vrne cilj avtomatskemu cilju Chlorosa. Brez zastavic prebere vozlišča osvetlitve, z istim `note:` kot `configure --get`. Vrednost `--set`, ki jo kamera zavrne, natisne `Failed to set ExposureTime: ExposureTime=<value> refused (<camera error>); the camera accepts <min>..<max> now (ExposureAuto put back to <mode>)` na stderr — v avtomatskem načinu se izklopljeno pisanje ponovno vklopi, namesto da bi ostalo izklopljeno — ne natisne vrstice `Exposure: … us (auto OFF)` in zaključi delovanje `1`. |
| `lattice gain [{--auto \| --off \| --set DB}]` | Ojačanje in samodejno ojačanje. Pri vrednosti `--set` kamera zavrne izpis `Failed to set Gain: …` na stderr, ne izpiše vrstice `Gain: … dB (auto OFF)` in se zaključi `1`. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N]` | Območje zanimanja senzorja (ROI) in združevanje pikslov. Združevanje pikslov je vedno **povprečje** — glej spodaj. |
| `lattice format [--set FMT] [--list]` | Format pikslov. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Strojni/programski sprožilec. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (brez zastavic = enkratno uravnavanje beline) | Operacije uravnavanja beline. Samo kamere RGB/Bayer; na mono M3M se preskoči (no-op). Funkcija »one-shot« ohrani izračunano belo ravnovesje (`BalanceWhiteAuto` in ojačitve); vse druge nastavitve, ki jih odpre, se vrnejo v prvotno stanje, in to v celoti, če funkcija »one-shot« ne more teči. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct 2000-10000] [--get]` | Barvni proces za prikaz RGB. `natural` (privzeto) je poceni obdelava v živo; `enhanced` doda odstranjevanje barvnih robov + živost + lokalni kontrast CLAHE za popoln videz s pariteto osrednjega dela pri približno dvakratnih stroških obdelave na posamezni sliki, kar pomeni nižjo **v živo** hitrost snemanja — shranjene posnetke pa v vsakem primeru vedno obdela v celoti. Samo za kamere RGB/Bayer; preskočeno na mono M3M. |
| `lattice color [--saturation 0-200] [--contrast 0-200] [--reset] [--get]` | Nasičenost/kontrast v živi predogledu (kamere s RGB-filtrom); `100` je nevtralno, `--reset` pa oba nastavi na 100. To ne vpliva na shranjene posnetke. Preskoči se pri mono M3M. |
| `lattice highlights [{--set -100..0 \| --reset \| --get}]` (brez zastavic = branje) | Prikaz **svetlih delov** (kamere z RGB-filtri): zmanjša najsvetlejše, najbeljše piksle — npr. svetla bela površina, ki se prikaže kot ravna bela lisa. `0` = izklopljeno, `-100` = najmočnejše. V živo prikaz in posnetki na ravni zaslona se temu prilagodijo; svetilnost/odbojnost se temu nikoli ne prilagodita, prav tako pa ne more obnoviti podatkov senzorja, ki so bili odrezani (za to zmanjšajte osvetlitev). Preskočeno na mono M3M. |
| `lattice filter [--list]` | Prikaži seznam modelov filtrov, ki jih pozna Chloros (`RGN-IMX265`, `OCN`, `NGB`, …). Filter kamere se prebere iz niza modela in ga ni mogoče nastaviti: `--set NAME` samo poišče model, nato pa izstopi iz `1` z `setting the filter model is not supported`. |
| `lattice power [--sleep]` | Preveri napajalne/termične vozle; vklopi/izklopi mirovanje z nizko porabo energije. `--sleep` pusti kamero v mirovanju (omejitev prepustnosti povezave je vklopljena (`DeviceLinkThroughputLimitMode` `On`) na najnižji vrednosti, varčevalni način pa je vklopljen, če ga kamera ima) vse ostale nastavitve se ob ponovnem zagonu ponastavijo na prvotno stanje; spanje, ki ni uspelo, se ponovno vzpostavi v celoti. Ob prebujanju se povezava ponovno nastavi tako, kot je bila ob spanju (omejitev, njen način in zamik paketov `GevSCPD`, ki ga kamera ohranja na 0, dokler je omejitev vklopljena; zapis se hrani na tem računalniku, tako da ga po ponovnem zagonu strežnika najde tudi prebujenje), ali pa, v primeru kamere, ki je ta računalnik ni spravil v mirovanje, a izgleda, da mirovanje (njena omejitev je nastavljena na najnižjo vrednost), z izklopljeno omejitvijo in vrednostjo `GevSCPD`, ki ostane na 0 pri 80 (tovarniške nastavitve); omejitev, nastavljena nad najnižjo vrednostjo, ostane nespremenjena. Sonda in prebujanje vrneta tisto, kar je zapisal njun ukaz »open«. |

> **Združevanje je vedno povprečno.** `lattice resolution --binning N` nastavi povprečno združevanje in
> ne omogoča preklopa med načini. Binning po načelu »Sum« pomnoži vsako DN z N², kar stane približno dve stopnji
> rezervnega prostora za svetle dele na vsak korak binninga; Chloros ta faktor ponovno odšteje pred
> radiometrično verigo, vendar so piksli, ki jih je »Sum« potisnil v zgornjo mejo senzorja, odrezani in jih ni mogoče
> obnoviti. Povprečni binning ne prinaša ničesar, česar povprečni ne bi – indeks je neodvisen od merila, skalarji
> sevalne moči pa so kalibrirani na povprečnem merilu. Če je kamera že nastavljena na seštevanje (kar je
> storilo drugo orodje ali tovarniška ponastavitev), Chloros to zazna iz odčitka vozlišča in
> to kompenzira; zagnanje `lattice resolution --binning N` jo ponovno nastavi na »Average«. Zastavica 1.2.0
> `--binning-mode` se še vedno sprejema, zato starejši skripti delujejo, vendar se prikaže le opozorilo:
> združevanje pik ostane na »Povprečju«.

> **Sprememba binninga, formata pikslov in sprožilca na kameri ne velja.** Na kameri
> backend ostane odprt in pretaka podatke (`cam-connect`, grafični vmesnik), `lattice resolution --binning`
> ne začne delovati (prikaže, da je združevanje še vedno v veljavi, `Binning: 1x1`),
> `lattice format --set` in `lattice trigger --mode` pa kamera zavrne
> (`SC_ERR_ERROR -1001`). Najprej jih sprostite (`lattice cam-disconnect -s <SN>`) ali jih nastavite
> ob povezavi (`cam-connect --pixel-format …`, `array-connect --binning …`).

##### Odčitavanje osvetlitve in ojačitve v avtomatskem načinu

Medtem ko teče `ExposureAuto` (`GainAuto`) kamere, njen vozlišče `ExposureTime` (`Gain`) ohranja **zadnjo zapisano vrednost**, ne pa trenutnega stanja kamere. Za kamero, ki jo backend drži odprto (`cam-connect`, `array-connect`, grafični vmesnik), `lattice configure --get`, privzeti pogled `lattice configure` in `lattice exposure` (brez zastavic) zato izpišejo vrednost vozlišča, pod njo pa opombo z najnovejšim sličajem kamere:

```

ExposureTime = 2000.448
  note: ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448); the camera's newest frame was at 4900 us.
```

Vrednost okvirja preberite iz opombe; vrednost vozlišča je le zadnji vpis. Prek HTTP-ja isti odgovor `POST /api/camera/<serial>/read-nodes` ohrani `nodes` nespremenjen in doda `frame_values` (`{"ExposureTime": <us>, "Gain": <dB>}`, najnovejšega okvirja) in `notes` (zgornji stavek). Oba sta prisotna le, če avtomatski način ni `Off` in je okvir znan; kamera, odprta samo za branje, nima okvirja in ne dobi nobenega od obeh. `lattice gain` (brez zastavic) natisne samo vozlišče — za opombo uporabite `lattice configure --get Gain`.

##### Osvetlitev člana matrike

Medtem ko matrika prenaša podatke, vsak član deluje z **`AcquisitionFrameRateEnable`**, s hitrostjo snemanja, ki je nekoliko višja od sprožilne hitrosti niza: njegovo obdobje je enako sprožilnemu obdobju minus 15 % (vsaj 3 ms) ali pa največji hitrosti kamere, če je sprožilna hitrost že blizu te vrednosti. Zato `configure --get` na elementu pri tej nastavitvi bere `AcquisitionFrameRateEnable` kot `True` in `AcquisitionFrameRate`; omogočanje se izklopi, ko se matrika ustavi.**Teh dveh vozlišč ne zapisujte na članu matrike** — matrika ju sinhronizira s svojim sprožilcem in ju ponovno zapiše.

Ta nastavitev je tudi zgornja meja osvetlitve člana: približno 85 ms pri sprožilcu 10 fps (perioda 100 ms), pri **katerem koli ROI ali združevanju**. Pred različico 1.3.0 je majhno območje zanimanja (ROI) namesto tega omejilo osvetlitev člana na obdobje prostega delovanja senzorja (4,9 ms pri 768 × 304), ne glede na to, kako počasi se je matrika sprožala.**Ročno nastavljeni `ExposureTime`, ki je daljši od zgornje meje, upočasni sprožilec, da se prilagodi tej vrednosti**, ko se zapiše z `lattice exposure --set` ali `configure --set` (ali prek grafičnega vmesnika). Matrika najprej odpre zgornjo mejo tega člana za to vrednost in pusti, da počasnejša hitrost teče nekaj sprožilnih obdobij, zato lahko zapis traja nekaj sekund dlje; sprožilec nato ne sproži hitreje, kot to dopušča osvetlitev. Pri matrici z avtomatsko hitrostjo se prejšnja hitrost vrne, ko se vsak član, katerega ročna osvetlitev jo je upočasnila, vrne na avtomatsko osvetlitev. Vrednost, ki je kamera sama ne more sprejeti, se še vedno zavrne, pri čemer se v napaki navede obseg, ki ga sprejema, avtomatska osvetlitev člana pa se ponovno vklopi, če je bila izklopljena le za to pisanje. Osvetlitev po receptu `apply` ne upočasni sprožilca: ta ostane na zgornji meji. Chloros nikoli ne skrajša ročne osvetlitve, da bi se prilagodil hitrejšemu sprožilcu; fiksna hitrost (`array-connect --target-fps F`), ki je hitrejša od tiste, ki jo dopušča osvetlitev, temu članu pusti zgornjo mejo, ki je dovolj dolga zanjo, in ta član sprejema sprožilce, ki niso hitrejši od tistih, ki jih dopušča njegova osvetlitev.

#### Kalibracija in senzorji

| Podukaz | Namen |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibracija na podlagi cilja odbojnosti. Za to se preišče do `--attempts` okvirjev (privzeto `30`); nepopoln okvir (izguba paketa GVSP) stane en poskus, ne pa celotnega izvajanja. Izvajanje se prekine pri vrednosti `no target detected in N attempts`, ali `no complete frame in N attempts: <last error>`, če ni prispel noben celoten okvir. |
| `lattice dls {--connect \| --spectrum \| --irradiance} --mac MAC [--filter NAME] [--camera SERIAL] [--json]` | Spektrum in jakosti pasov naprave DAQ-M, prebrani prek Bluetootha (BLE) prek strežnika. `--mac` je lastni BLE MAC senzorja in je potreben pri kateri koli od treh akcij: ni privzete vrednosti, saj ima vsaka enota svojo (`daq pool-discover --only ble` jih navaja); brez njega se ukaz zaključi z `1`. Če ni izbrana nobena akcija, se prikaže pomoč. `--camera SERIAL` integrira prikazane vrednosti pasov prek lastnega izmerjenega spektralnega odziva te kamere namesto krivulje filtra modela — RGB-kamere, če je kalibracijski paket kamere naložen (priključen) ali shranjen v predpomnilniku tega računalnika. Naslov navaja, čigava krivulja je bila uporabljena: `Band magnitudes (camera <SERIAL>):` ali `Band magnitudes (model <filter>):`, če kamera nima nobene (ni RGB-kamera, ali ni naloženega ali shranjenega paketa) in je bila namesto tega uporabljena krivulja flote. Prek HTTP-ja `POST /api/dls/cli-read` sprejme `camera_serial` in vrne `band_magnitudes_source`. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Uporabi popravek vinjetiranja za obstoječe slike, vsaka s svojim ravnim poljem objektiva (iz podatka EXIF Model). `--lens-model` se uporablja za slike, katerih Model ne navaja imena kamere; slika, katere Model je v nasprotju s tem, se preskoči. |

#### Več kamer (začasne seje)

Začasna komanda odpre kamere za eno izvedbo in jih zapre ob njenem koncu: `multi-capture` brez povezave s trajnim nizom, `multi-stream` in `multi-test` tukaj, ter `align-calibrate`, `align-apply`, `align-stream` in `index --live` spodaj. Za vse veljajo naslednja pravila:

- **`--serials SN1,SN2,…` izbere kamere.** Odprejo se le te, prva pa sproži sprožilec (za `align-calibrate`, `--reference`, če sta navedena). Serijska številka, ki ni v omrežju, povzroči neuspeh izvajanja, še preden se odpre katera koli kamera. Brez `--serials` se odprejo vse kamere v omrežju. Kamera, ki jo Chloros že drži (GUI, `cam-connect`, `array-connect`), zavrne izvedbo, ne glede na to, ali jo `--serials` imenuje ali ne: saj si deli omrežje, za katero je izvedba namenjena. (`index --live` brez `--serials` namesto tega odpre kamere, ki so omogočene v njegovem profilu: glej [Indeks / Matematika vegetacije](#index--vegetation-maths).)
- **Kamere si delijo omrežje na enak način, kot si ga deli `array-connect`.** Vsaka kamera je prilagojena svojemu deležu tega, kar prenaša gostiteljska povezava (zamik med paketi, ne pa omejitev prepustnosti kamere, katere spodnjo mejo 31,25 MB/s štiri kamere na 1 GbE že presegajo), tako da štiri kamere delujejo na povezavi s hitrostjo 1 Gbps. Ko kamere ne morejo vse hkrati pošiljati s najpočasnejšim dosegljivim ritmom, se sprožajo ena za drugo, vsaka s povezavo samo zase, ukaz pa izpiše `Serial emit: the wire cannot carry these cameras at once, so each is triggered <ms> ms after the one before; their frames are not exposed together`. Če se v danem trenutku ne prilega niti ena kamera, se izvajanje zavrne, še preden katera koli kamera začne prenašati: `array over-subscribes the wire: <n> cameras demand <x> MB/s but the collision-safe ceiling is <y> MB/s. …`, HTTP 409, izhod `1`.
- **Vsaka kamera mora posredovati podatke, sicer izvedba ni uspešna.** Kamera, ki ne posreduje nobenega okvirja, predstavlja napako: ukaz izpiše `Error: … no frame from <n> of <total> cameras: SN:<serial> (<why>), …`, pri čemer je `<why>` napaka pri pridobivanju podatkov te kamere, nato pa navede vzroke napake in se zaključi z `1`. Okvir, ki je prispel z izgubo paketov GVSP (`IncompleteBufferError: GVSP packet loss …`), pomeni, da omrežje ne prenese skupne pasovne širine kamer (`Every missing frame arrived with GVSP packet loss: …`): preverite hitrost povezave gostitelja in povezave stikala z omrežjem (`lattice network`), uporabite jumbo okvirje, ali odprite manj kamer z ukazom `--serials`. Kamera, ki ni posredovala ničesar, običajno ne prejema sprožilca glavne kamere: preverite njen sinhronizacijski kabel M8, nato pa zaženite ukaz `lattice multi-detect`. Kaj posamezni ukaz stori s skupino okvirjev, v kateri manjka kamera, je navedeno v ustrezni vrstici.
- **Vsaka kamera se vrne v stanje, v kakršnem je bila najdena.** Ko se izvajanje konča, vsaka kamera ponovno dobi tisto, kar je imela ob začetku izvajanja: `TriggerMode`, `TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, ter `LineMode` / `LineSource` na `Line2` in `Line3` (in odstranjevanje motenj na sprožilnem vhodu, ki ga zagotavlja matrika na podrejeni liniji, `LineFilterSelector` / `LineFilterWidth`); njegov `PtpEnable` (in z njim nastavljeni `PtpSlaveOnly` / `PtpDomainNumber`); `ExposureAuto` / `GainAuto` (in `ExposureTime` / `Gain`, kjer je bila vrednost nastavljena ročno); `TargetBrightness`, `ExposureAutoLimitAuto` / `ExposureAutoUpperLimit` / `ExposureAutoLowerLimit`, `GainAutoUpperLimit`, `ExposureAutoDamping`; `BalanceWhiteAuto` (in vsak `BalanceRatio`, kjer je bila nastavitev beline ročna); `BlackLevelAuto` / `BlackLevel`; `PixelFormat`, združevanje slik, `Width` / `Height` / `OffsetX` / `OffsetY`; `GammaEnable` (in `Gamma`, kjer je bila vklopljena gama), `LUTEnable`, `ColorTransformationEnable`; `ReverseX` / `ReverseY`; `UserSetSelector`; velikost paketa (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`); hitrost pošiljanja, `DeviceLinkThroughputLimitMode` (in `DeviceLinkThroughputLimit`, kjer je bil način `On`), zamik medmedpaketna zakasnitev `GevSCPD` in zamik zagona `GevSCFTD`, ki jih vsaka kamera zagotavlja; podatki o delih, ki jih vsak tok vključi (`ChunkModeActive`, vsak `ChunkEnable` pod svojim `ChunkSelector`); `AcquisitionFrameRateEnable` (in `AcquisitionFrameRate`, kjer je bila vklopljena možnost omogočanja — če je izklopljena, je hitrost samo za branje in bere največ, kolikor dopušča ritem, zato to ni nastavitev). Zapisano je le tisto, kar se razlikuje: kamera se je najprej ustavila, hitrost sinhronizacije in hitrost snemanja pred osvetlitvijo, ki zahteva svoje obdobje slike (kamera omeji `ExposureTime` na eno obdobje slike). Pri `--keep-settings` se ožičenje sprožilca in sinhronizacijskega vodnika prekine, kamera deluje samostojno, ostalo pa se shrani v lastni pomnilnik kamere, kot je določeno v nastavitvah delovanja; `--clear-settings` tam nastavi tovarniške nastavitve.
- **Izpostavljenost se nastavi na podlagi prizora pred shranitvijo prvega okvirja.** Serija zajema sinhronizirane skupine slik in jih zavrže, medtem ko na podlagi teh slik nastavi osvetlitev in ojačanje za vsako kamero. Cilj je povprečna svetlost na cilju kamere, pri čemer ne sme biti več kot 1 % slike preosvetljenega; prizor z močnimi svetlimi deli se ustavi pred doseganjem cilja. To običajno traja 2–6 skupin posnetkov in največ 5 s.
  - Vsaka kamera usmerja objektiv na cilj, pri katerem jo povezan niz sproži, in sicer prek svojega filtra: TargetBrightness **40**na multispektralni kameri (RGN, OCN, NGB) in**60** na RGB kameri. To je polovica ciljne vrednosti osvetlitve 80 (tri četrtine pri RGB), zato je manj verjetno, da bo svetla odbojna plošča izrezana. `--brightness N` (0–255) namesto tega nastavi ciljno vrednost za vsako kamero (`multi-test` nima `--brightness`: njegova samodejna osvetlitev je izklopljena). Dnevnik strežnika poimenuje ciljno vrednost vsake kamere: `[transient-ae] AE target per camera (filter default): SN:<serial> <target>, …`.
  - Najprej se določi osvetlitev, do meje, ki jo določa avtomatska osvetlitev kamere. Ojačanje pokriva le tisto, kar ostane.
  - Določeni vrednosti osvetlitve in ojačitve se ohranita skozi celoten cikel, zato je vsak posnetek kamere `multi-stream` posnet z enako osvetlitvijo. Snemanje, ki mora slediti spreminjajoči se svetlobi, sodi v povezano matrico (`array-connect`), katerega avtomatska osvetlitev se nenehno prilagaja.
  - `--exposure` in `--gain` se ohranita točno takšna, kot sta bila določena. Če sta navedena oba, se noben posnetek ne zavrže.
  - Ko se izvajanje konča, se samodejna osvetlitev vsake kamere ponovno vklopi.
  - Dnevnik strežnika poroča o rezultatu: `[array-ae] exposure solved in <s> s (<n> warm-up groups): SN:<serial> <ms> ms <dB> dB mean <m>/<target>, …` ali `exposure NOT solved after …` z navedbo razloga ter `(not settled)` za vsako kamero, pri kateri to ni bilo mogoče. V tem primeru se posnetki še vedno zajamejo.
  - `CHLOROS_TRANSIENT_AE_SETTLE_S` spremeni 5-sekundno omejitev; `0` to izklopi (glej [Okoljske spremenljivke](#environment-variables)).

**Medtem ko Chloros drži katero koli kamero** (grafični vmesnik, `cam-connect` ali povezano matrico), `multi-stream`, `multi-test`, `multi-detect`, `align-apply`, `align-stream` in `index --live` ne delujejo, ker odprejo vsako kamero v omrežju, pri čemer lahko že drugo odprto odprtje zadržane kamere lahko povzroči sesutje strežnika. (`index --live` se zavrne, če je kamera, ki bi jo odprl, zadržana: gre za kamero `--serials` ali kamere iz njegovega profila ter katero koli kamero za profil, ki ne navaja nobene.) Backend odgovori z HTTP-odgovorom 409 `CAMERA_HELD`, pri čemer navede vsako zadržano kamero in ukaze za sprostitev (`chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`), nato pa se ukaz zaključi z `1`. Če je priključen niz, ga `array-capture --continuous` zapiše namesto `multi-stream`, `array-capture` (privzeto poravnano) shrani poravnano skupino namesto `align-apply`; `multi-capture` in `align-calibrate` uporabljata povezano matrico, ki vsebuje njune kamere, namesto da bi jih odpirala.

| Podukaz | Namen |
| --- | --- |
| `lattice multi-info [--serials SN1,SN2,…]` | Prikaže seznam vseh kamer s sinhronizacijskimi vlogami; z `--serials` le tiste, prva pa kot `MASTER`. |
| `lattice multi-capture [--serials SN1,SN2,…] [--format FMT] [--jpeg-quality N] [--processing LEVEL] [--brightness N] [--no-daq] [--daq-tolerance-ms MS]` | En sinhroniziran okvir iz vsake kamere. **Privzeto shrani vse vrste izvoza**, ko je priključen trajni niz, in DAQ-odčitek, ki ga je uporabil ([DAQ-odčitek, ki ga shrani zajem](#the-daq-reading-a-capture-saves)); začasna rešitev brez matrike je**le brez debayeringa** (za ostalo najprej zaženite `array-connect`). `--brightness` velja le za nadomestno rešitev: priključena matrica ohrani svojo avtomatsko osvetlitev. Nadomestna rešitev ne shrani `.daq` (`--no-daq` / `--daq-tolerance-ms` tam nimata učinka), `multi-capture` pa ne upošteva `--force-daq`. V nadomestnem načinu kamera, ki ne prenaša nobenega okvirja, povzroči neuspešno zajemanje (`Error: multi-capture failed: no frame from …`, izhod `1`) in nobena datoteka te skupine okvirjev se ne shrani. |
| `lattice multi-stream [--serials SN1,SN2,…] [--fps F] [--count N] [--format FMT] [--jpeg-quality N] [--brightness N]` | Sinhronizirani okvirji v toku (začasno), `--fps` privzeto `3`, dokler ne pritisnete Ctrl+C ali `--count` skupin. Skupina sličic, za katero kamera ni posredovala ničesar, se ne zapiše, tri zaporedne pa zaključijo tok. Če je bila katera koli skupina preskočena, se izvajanje konča z izpisom `Incomplete: <n> frame group(s) not written, a camera delivered nothing (SN:<serial> x<count>, …)`, skupaj z `Error: the stream stopped: …`, ki navaja ime kamere, ko je ta zaključila tok, in se zaključi z `1`X. Samodejno se ustavi, ko se pogon napolni, izpiše `Failed:` / `Disk full:` in se zaključi z `1`; vsaka datoteka, v katero ni mogla zapisati, prav tako zaključi delovanje `1`. |
| `lattice multi-test [--serials SN1,SN2,…] [--count N] [--ptp-settle SECONDS]` | Test sinhronizacije GPIO (prehoden), `--count` privzeti sprožilci `10`. Izpiše, kako je omrežje razdeljeno (`Wire: each camera paced to <x> MB/s (<n> on a <y> MB/s ceiling, <link> Mbps link)` in vrstica `Serial emit:`, ko se kamere sprožajo ena za drugo, kar se nato odraža v časovnih oznakah), ena vrstica `Sync group: master SN:<serial> -> SN:<serial>, …` na sinhronizacijsko skupino (`(no slaves: not synced)` za kamero brez sinhronizacijskega kabla, ki se sproži samostojno), nato ena vrstica na sprožilec: `[  0] SN:<serial>=<timestamp> \| …`, pri čemer je `SN:<serial>=(no frame)` za kamero, ki za ta sprožilec ni posredovala ničesar. Nato `Offsets from master SN:<serial> (<PTP / device (PTP off)> clock, <n> trigger(s)):` in vrstica `SN:<serial>: mean <±x> us, jitter <y> us` za vsako kamero, vsaka kamera glede na glavno kamero znotraj enega takta: sprožilec, pri katerem žigi mešajo ure naprav kamer in PTP (PTP izklopljen ali še vedno zaklepa), pri čemer jih šteje `Note:`. Vsak izpuščen okvir pomeni neuspeh testa (`Error: multi-test failed: <k> of <n> frame groups incomplete (SN:<serial> missed <k>); …`, izhod `1`); trije nepopolni sprožilci zapored ga predčasno zaključijo. Test se prav tako ne uspe (izhod `1`), če je pri manj kot dveh sprožilcih vsaka kamera na isti uri (`… the clocks cannot be compared`) ali če se ure nikoli ne ujemajo: PTP-ure, ki odstopajo za več kot 1 ms (plus morebitno zamikanje na serijski ravni) od glavne ure (`… PTP is not locked`), ali ure naprav, katerih odmik med sprožilci presega to vrednost. Če PTP glavni vodja ne deluje, ga zažene, vklopi PTP na vsaki kameri (kot to stori `array-connect`, ne glede na to, kako je omrežje sestavljeno) in pred prvim sprožilcem počaka, da se PTP ura vsake kamere uskladi z glavnim vodjo in se ustalijo: `PtpStatus` Slave na enem glavnem uradniku, z odstopanjem največ 100 us od njega in stabilen 3 s, vsaka kamera pa se na ta način sinhronizira brez prekinitve 120 s — približno 2 minuti od vklopa. Ura, ki se je pravkar uskladila, ima odstopanje do ~200 us še približno 2 minuti po tem, ko se prikaže stanje »Slave«, kar pa kamera sama `PtpOffsetFromMaster` ne prikaže; v 120 s se kamere približajo stabilni uri z odstopanjem ~10 us. Kamera je še vedno sinhronizirana od prejšnjega testa, pri katerem je PTP ostal vklopljen, kot je to zaznal ta backend, šteje od tega sinhroniziranega trenutka, zato test, ki sledi takoj za drugim, ne čaka ponovno; kamera, ki jo test vklopi, ali tista, pri kateri je PTP vklopljen, a ta backend še ni zaznal sinhronizacije (vklopljena drugje ali pred ponovnim zagonom backenda), počaka svojih 120 s, medtem ko v nizu obeh kamer, ki sta se pravkar sinhronizirali, še vedno čakata na svoj čas. Kamera, ki zapusti stanje »Slave«, preseže 100 us ali med čakanjem sledi drugemu »grandmasterju« pri 3 odčitkih zapored (1,5 s), začne ponovno šteti; en ali dva takšna odčitka se zabeležita (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`), štetje pa se nadaljuje; ID glavnega vodje, ki ga ni mogoče prebrati, se obravnava kot neznan, ne kot sprememba. Če se štetje vseh kamer hkrati ponovno začne (v roku ene sekunde), gre za premik časa glavnega vodje, ne pa kamer: dnevnik strežnika navaja `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` ga navaja v `grandmaster_steps`, čakanje, ki se nato izteče, pa začne svojo napako z `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Meja za to ni podaljšana. `--ptp-settle SECONDS` nastavi čas umirjanja za izvedbo (0–600; `0` za čakanje brez umirjanja). Čakanje je omejeno na čas stabilizacije plus 90 s (privzeto 210 s, najmanj 90 s), ukaz pa čaka na backend 240 s plus čas stabilizacije (privzeto 360 s). Izpiše `PTP: locked after <s> s (each camera locked <settle> s, already locked: SN:<serial>, …; from the grandmaster: SN:<serial> <±x> us, …)`, v dnevniku strežnika pa se vsake 5 s pojavi vrstica s stanjem in odmikom vsake kamere, kako dolgo je že zaklenjena in koliko časa ji še ostaja: `[array-sync] PTP at <s> s: SN:<serial> Slave <±x> us for <s> s (<s> s left), … -- <s> s left, the wait gives up at <s> s`. Vsaka kamera se vklopi z izklopljenim PTP; od takrat se ure sinhronizirajo v približno 10 s. Nato se vsaka kamera vrne v stanje, v katerem jo je test našel – njen PTP, avtomatska osvetlitev in ojačanje, ki jih test med izvajanjem ohranja izklopljene, ter ostalo (razen če je `--keep-settings`) — torej kamera, pri kateri je PTP ostal vklopljen (`lattice configure -s <serial> --set PtpEnable=True` ali izvedba s `--keep-settings`), ostane sinhronizirana, in ko ta backend ugotovi, da je bila sinhronizirana za čas stabilizacije, kasnejši test nanjo ne čaka. Če se ure med čakanjem ne uskladijo, test ne uspe še pred sprožitvijo (`PTP: NOT locked …`, izhod `1`) in navede, kaj je treba storiti: kamera, ki se nikoli ni zaklenila (`PtpStatus Listening`), pomeni, da grandmaster ne dosega te kamere (`time-sync status`); tista, ki je nenehno izpadala, je poimenovana po tem, kako dolgo je bila zaklenjena (`… had not been locked for the 120 s settle time … (SN:<serial> locked <s> s, <s> s to go)`); ure se še vedno usklajujejo: preverite `time-sync status`, in da se pred testom zaklenejo, vklopite PTP in ga pustite vklopljenega (`lattice configure -s <serial> --set PtpEnable=True` za vsako kamero ali pa test izvedite z `--keep-settings`; test ponovno vzpostavi ugotovljeno stanje, zato ostane vklopljen), nato pa ga izvedite. |
| `lattice multi-detect [--line LINE] [--json]` | Poiščite kamere na sinhronizacijski liniji M8 in privzeto glavno kamero: RGB-kamero (model, ki se konča z `-FRGB`), če je priključena s kablom, sicer pa prvo priključeno kamero po serijski številki. Kabli M8 se združujejo na enem križišču, zato **lahko katerakoli priključena kamera deluje kot glavna** — natisnjena oznaka `MASTER` je privzeta vrednost, ne pa dejansko stanje ožičenja. Vsaka kamera po vrsti poganja linijo, ostale pa se preverjajo glede na njen impulz; kamera, ki je že bila najdena v ožičeni skupini, se ne preverja ponovno, tako da popolnoma ožičena skupina potrebuje en krog. Če ni `--line`, preveri `Line2`, nato pa `Line3`. Natisne `MASTER: SN:…`, po en `SLAVE: SN:…` za vsako drugo ožičeno kamero ter `(probed on <line>)`; `--json` doda celoten rezultat (`master`, `slaves`, `no_cable` — kamere, ki jih impulz ni dosegel — `line` ter `results` za vsako kamero posebej). Ko se postopek konča, vsaka kamera ponovno pridobi nastavitve sprožilca in sinhronizacijskega signala, ki jih je imela prej (iste vozlišča kot zgoraj). |

#### Poravnava

| Podukaz | Namen |
| --- | --- |
| `lattice align-calibrate [--serials SN1,SN2,…] [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N] [--brightness N]` — dodatni gumbi za detektorje/usklajevalnike `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, RANSAC-ovih gumbov `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, kombinacija več okvirjev `[--averaging mean\|median\|inlier_weighted]`, geometrijske omejitve `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, prostorska omejitev `[--roi X0,Y0,X1,Y1] [--mask PATH]` terslave `[--per-cam-override SN:KEY=VALUE]` (ponovljivo) | Izračunaj profil poravnave iz žive slike kamer. Privzete vrednosti: `--method orb`, `--model rigid`, `--frames 3`, `--save alignment_profile.json` (v mapi, iz katere ga zaženete), `--max-features 2000`, `--ratio-threshold 0.75`, `--matcher bf`, `--knn-k 2`, `--ransac-threshold-px 3.0`, `--ransac-iters 2000`, `--ransac-confidence 0.99`, `--averaging mean`. (`project align calibrate` ima svoje privzete nastavitve; glejte [njegovo tabelo](#project-align-calibrate-options).) Privzeta nastavitev za `--prefilter` je `gradient` (zemljevid robov; ustreza GUI/poravnalniku matrik – robovi ostanejo ohranjeni čez spektralne pasove). `--matcher flann` se izplača pri več kot ~5000 značilnostih; `--averaging median` je odporen na eno slabo zajetje, `inlier_weighted` tehtanje po številu ujemanj; `--lock-scale` projicira na najbližjo rotacijo (brez merila), `--lock-axis` eno komponento premika nastavi na nič; `--mask` se uporablja za vsako kamero (za nastavitve po posameznih kamerah uporabite `--per-cam-override`, npr. `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` zavrne shranjevanje kalibracije, če RMS reprojekcije presega prag. `--method manual` črpa svoje točke iz `--points PATH`: JSON-objekt `{"<serial>": [[x, y], ...], ...}`, ki navaja koordinate pikslov istih značilnosti za vsako kamero, vključno z referenčno kamero (vsaj 2 na kamero); vsaka od `--frames` se izračuna iz teh točk. Ko kamere vsebuje povezan niz, kalibracija uporabi žive posnetke tega niza, profil pa postane tudi poravnava niza za njegove posnetke. V nasprotnem primeru odpre [začasni niz](#multi-camera-transient-sessions): kamera, ki ne posreduje nobenega posnetka v nobenem od `--frames`, ne opravi kalibracije (izstop iz `1`, z navedbo imena), profil pa se ne shrani. |
| `lattice align-apply --profile PATH [--serials SN1,SN2,…] [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N] [--brightness N]` | Posname en poravnan večpasovni posnetek. `--bit-depth` privzeto uskladi s kamero; `--no-crop` ohrani celoten okvir (izpolni s črno barvo); `--interpolation` (privzeto `linear`) in `--border-mode`/`--border-value` (privzeto `constant`/0) nadzorujeta deformacijo CPU — pot GPU je v vsakem primeru bilinearna. Odpre [prehodni niz](#multi-camera-transient-sessions): kamera, ki ne prenaša nobenega okvirja, ne uspe pri zajemanju (izstop iz `1`, z navedbo imena), in nič se ne zapiše. |
| `lattice align-stream --profile PATH [--serials SN1,SN2,…] [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N] [--brightness N]` | Tokovno usklajeni večpasovni okvirji (isti gumbi za warp kot pri `align-apply`), `--fps` privzeto `3`. Odpre [začasni niz](#multi-camera-transient-sessions): skupina sličic, za katero kamera ni posredovala ničesar, se ne shrani, trije zaporedni pa zaključijo tok. Če je bila katera koli skupina preskočena, se izvajanje konča z izpisom `Incomplete: <n> frame group(s) not saved, a camera delivered nothing (SN:<serial> x<count>, …)` in `Error: …`, in se zaključi z `1`. Tok, ki se konča zaradi katere koli druge napake, prav tako izpiše `Error: …` in se zaključi z `1`. |
| `lattice align-info --profile PATH [--json]` | Prikaži podrobnosti profila. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Spremeni vrstni red slojev. |

#### Indeks / Matematika vegetacije

```bash
# Offline: compute NDVI from an aligned multi-band TIFF (an RGN camera's bands)
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: open the cameras, capture one aligned set with a profile made by
# `lattice align-calibrate`, and compute the index from it
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --save-multiband -o output/
```

**`--channel` preslika simbole prednastavitve na pasove** (NDVI: `red`, `nir`), po en `--channel SYMBOL=BAND` za vsakega.

- **`--live`: velja le za simbol, na katerega sklop ne more odgovoriti.** Tako kot pri `--input` simbol prednastavitve, ki ga izpustite, prevzame tisti pas iz sklada (pasovi omogočenih kamer profila), katerega ime se z njim začne. To se preveri, preden se odpre katera koli kamera: simbol, na katerega ne odgovarja noben pas ali več pasov, se nato zavrne, pri čemer se jih imenuje in zapre `1`. Prilagojeni `--formula` potrebuje `--channel` za vsak simbol.
- **`--live` izdelki navajajo, komu pripadajo.** `index.tif` in `index_LUT.png` vsebujeta podatke o kameri v sklopu (proizvajalec, model, serijska številka), času zajema (izpostavljenost referenčne kamere, UTC) in objektiv, oba pa imenujeta indeks kot svoj en pas (`XMP-Camera:BandName` `NDVI`; barvni PNG datoteka imenuje indeks, ne pa svojih barvnih kanalov), tako kot to počne izdelek `--input`. Indeks iz pasov več kamer pa namesto tega poimenuje vsakega od njih, njegov `Model` pa je `LATTICE` — glej [Datoteke, narejene iz več kamer](#files-made-from-several-cameras).
- **`--input`: velja le za simbol, na katerega datoteka ne more odgovoriti.** Simbol, ki ga izpustite, prevzame tisti pas datoteke, katerega ime se z njim začne: `red` prevzame `Red_660` ali `Red_660_RGN`, `nir` pa prevzame `NIR_850`. Imena izhajajo iz same datoteke: iz seznama pasov samega sklada ali, v primeru zajema sevanja/odbojnosti Chloros ali izdelka `process`, iz njegovih oznak `Chloros:Band{R,G,B}Name` / `XMP-Camera:BandName`. Zato `lattice index --input <reflectance capture> --preset NDVI` ne potrebuje `--channel`.
- **Dva pasova, ki ustrezata enemu simbolu, se ne ugibata med seboj** (sklop z 9 kamerami ima dva NIR-pasova): poimenujte tistega. Simbol, ki ne ustreza nobenemu pasu, se zavrne z `channel_map missing entries`, izhod `1`.**Imena pasov v sklopu** nosijo filter kamere: pasovi kamere M3C so `<band>_<filter>` — `NIR_850_RGN`, `Green_550_RGN`, `Red_660_RGN` na kameri RGN; `NIR_808_OCN`, `Cyan_490_OCN`, `Orange_615_OCN` na sistemu OCN; `Blue_475_RGB`, `Green_550_RGB`, `Red_625_RGB` na sistemu RGB; `Blue_475_NGB`, `Green_550_NGB`, `NIR_850_NGB` na NGB — in enojni pas kamere M3Mje enojni pas `Mono_<filter>` (`Mono_660`, `Mono_850`). `lattice align-info --profile PATH` navaja pasove profila.

Celoten nabor zastavic: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (ponovljivo), `--capture-level raw|debayered|radiance|reflectance|unknown` (prepisuje raven zajema, zabeleženo v izvorni datoteki TIFF; privzeto: prebrano iz metapodatkov TIFF, v nasprotnem primeru se uporabi vrednost `Chloros:ProcessingLevel` iz datoteke Chloros, tako da zajem odbojnosti poroča o vrednosti `reflectance`), `--output PATH` / `-o` (offline: pot; s končnico slike se datoteke poimenujejo po njenem korenu — `-o ndvi_FRGN.png` zapiše `ndvi_FRGN.tif` in `ndvi_FRGN_LUT.png` — brez končnice pa je to predpona, `-o ndvi` -> `ndvi.tif`, `ndvi_LUT.png`; privzeto poleg vnosa, kot `<input name>_<preset>`; z `--live` in brez `-o`, mapa `output/` pod mapo, iz katere izvajate ukaz), `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Pri datoteki `--live` se uporabljajo tudi drsniki za poravnavo: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`, `--brightness N` (ciljna vrednost avtomatske osvetlitve kamer, glej pravila za prehodne pojave), `--serials SN1,SN2,…`. `--live` odpre [prehodni niz](#multi-camera-transient-sessions): kamera, ki ne posreduje nobenega okvirja, ne uspe v izvedbi (izhod `1`, z navedbo imena), pri čemer se ne zapiše niti sklad niti indeks. Odpre `--serials` (prva sproži sprožilec), sicer pa kamere profila, katerih vnos `layer_order` je omogočen, najprej njena referenca — referenca tudi z onemogočenimi pasovi, saj so ostale poravnane z njo in sproži sprožilec. Kamera, katere pasovi so onemogočeni, se nikoli ne kalibrira, zato RGB-kamera, onemogočena v profilu, ne povzroči več napake pri indeksu odbojnosti (`no-absolute-radiance-scale`). Profil, ki ne imenuje nobene kamere, odpre vse kamere v omrežju, kot prej.

Vsak obrazec teče na strežniku, zato mora ta teči: ga zaženite iz grafičnega vmesnika Chloros ali z ukazom, ki ga CLI izpiše, ko ga ne more doseči. V načinu brez povezave backend prebere `--input` ter sam zapiše TIFF v formatu float in obarvan PNG, zato se relativni `--input` ali `-o` se razreši glede na mapo, iz katere izvajate ukaz (glejte [Konvencije](#conventions)). Oba izdelka v načinu brez povezave vsebujeta podatke o kameri iz vhodnega datoteka, tako kot izvoz indeksa `process`: znamka / model / serijska številka, GPS, oznake objektiva, čas zajema (iz vhodnega datoteka; če v njegovih EXIF-podatkih ni naveden, se upošteva trenutek osvetlitve, ki ga je zabeležil, ali časovni žig v imenu datoteke, v UTC) ter identifikacijsko številko kalibracije MAPIR — nikoli kalibracije raw-DN, ki se ne uporablja za indeks — ter `Chloros:ProcessingLevel=index` in izraz indeksa; plavajoča vrednost TIFF označuje en pas, prednastavitev. `--list-presets` in `--list-gradients` natisneta tabeli prednastavitev in gradientov backenda. Če backend zavrne formulo ali če TIFF-datoteka nima pasu `--channel`, se izhod `1` prikaže z razlogom backenda.

> **Pri simbolih `--channel` se upošteva velika in mala črka.** Stran simbola se mora natančno ujemati z imeni kanalov prednastavitve (prednastavitve uporabljajo male črke, npr. NDVI = `red`,`nir` — preverite `--list-presets`), stran z imenom pasu pa mora ustrezati imenu pasu v poravnanem nizu (ali biti indeks pasu, ki se šteje od 0, v načinu brez povezave). `--channel red=Red_660_RGN --channel nir=NIR_850_RGN` deluje; `--channel RED=660` ne deluje in povzroči napako `channel_map missing entries` .

#### `lattice index` — maske za večpasovne sklade

Pri `--capture-level radiance` ali `reflectance` lahko 3×3 ločevanje vsake kamere omeji pas na 0 (glej [Maska omejitve ločevanja](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on)). Večpasovni niz vsebuje pasove več kamer, vsaka kamera pa omeji svojega, zato `--live --save-multiband` zapiše **eno masko na izvorno kamero** poleg niza:

```
output/
├── index_multiband.tif
└── clamp_masks/
    ├── index_multiband_SN<serial A>-<filter A>_clampmask.tif
    └── index_multiband_SN<serial B>-<filter B>_clampmask.tif
```

- Vsaka maska je enopasovna `uint8` na pikselni mreži **sklopa** (prek istega poravnave, deformacije in izreza), z lastnimi biti te kamere: bit 0 = pas njenega R-kanala, bit 1 = G, bit 2 = B. Pas sklada, vzet iz kanala *c* kamere, se omeji tam, kjer je nastavljen bit *c* te kamere (R = 0, G = 1, B = 2).
- Vgrajena provenienca vsakega pasu imenuje masko te kamere (`unmix_clamp_mask`, glede na mapo sklada, ali `none`) ter bit, ki ga je treba prebrati (`unmix_clamp_mask_bit`).
- Kamera, pri kateri postopek ločevanja ni omejil ničesar, ne dobi datoteke; mono- ali RGB-kamera ne izvede ločevanja in prav tako ne dobi datoteke. CLI izpiše vsako masko pod sklopom (`clamp mask SN…: …`); odgovor jih navaja kot `clamp_masks`.
- Indeksni TIFF/PNG iz istega cikla se izračuna iz istega sklada, na isti mreži, zato ga maske prav tako opisujejo. `--capture-level raw` ne izvaja ločevanja in ne zapiše nobene maske.
- Zbirka in indeks sta neposredno v `--output`, zato se maske nahajajo v `--output`-ovem lastnem `clamp_masks/` — nikoli nad mapo, ki ste jo poimenovali. Ta mapa združuje različne vrste izdelkov (sklad, indeks, obarvano upodabljanje): izberite njene datoteke, ne predajajte celotne mape programski opremi za fotogrametrijo.

#### Trajne povezave (Smart-Prep, potek, enakovreden grafičnemu vmesniku)

Ti ukazi ohranjajo kamere odprte v bazenu v ozadju med posameznimi klici prek CLI.

| Podukaz | Namen |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Dodaj eno kamero v zbirko (ena kamera, brez niza). [Splošne možnosti](#common-options-shared-by-most-subcommands) `--exposure`, `--gain`, `--preset`, `--pixel-format`, `--width` / `--height` in `--packet-size` (`standard` = 1500, `jumbo` = 9000 ali bajti; sondi `auto`) se uporabljajo ob priključitvi (stikala `--exposure` ali `--gain`, ki samodejno izklopita ta način). Vrne se, ko se začne prenos iz kamere (na to počaka približno 15 s, nato pa 10 s na prvi polni zalogovnik) in izpiše `stream: streaming` ali `stream: no_frames -- connected, no frames yet: …`, če prenos ni prinesel ničesar — za kamero, ki uporablja jumbo pakete, s priporočilom, da se ponovno povežete z `--packet-size standard`. Medtem ko se ena kamera povezuje (prva povezava prenese njeno kalibracijo, kar traja približno 70 s), drug ukaz, ki potrebuje kamero, na njo čaka največ 5 s, nato pa se pojavi napaka HTTP-503 `CAMERA_POOL_BUSY`, ki navaja kamero, ki se povezuje; ukaz ponovite čez nekaj trenutkov. `lattice status` in seznam kamer nikoli ne čakata: kamera, ki je v procesu povezovanja, se prikaže kot povezana, skupaj s svojo fazo. |
| `lattice cam-disconnect [--serial SN] [--all] [--keep-settings \| --clear-settings]` | Sprostitev. Kamera se vrne v stanje, v kakršnem je bila ob vzpostavitvi povezave (oživitev sprožilca, PTP, nastavitve); `--keep-settings` namesto tega shrani nastavitve v lastni pomnilnik, `--clear-settings` pa ta pomnilnik ponastavi na tovarniške nastavitve (glej [Pomnilnik kamere](#camera-memory----keep-settings----clear-settings)). Kamera, ki je članica niza, sprosti celoten niz. |
| `lattice cam-list` | Prikaži kamere v bazenu. |
| **`lattice array-connect`**|**Povezava trajnega sinhroniziranega niza (PRIPOROČENA vstopna točka).** Izvede celoten potek pametne priprave v grafičnem vmesniku. Pri 9 kamerah traja približno minuto; takoj po vklopu kamer pa lahko traja tudi več minut. Med čakanjem izpiše vsak korak, ki ga doseže strežnik, kot `    <s> s: <step>` (na primer `    41 s: open SN:214000535 (4 of 9)`), v dnevniku strežnika pa je za vsak korak ena vrstica s časom (`[array-connect <id>] <step>: <s> s …`) in zadnja vrstica z najpočasnejšimi koraki. Čaka do 600 s. Če povezava do takrat ni vzpostavljena, jo strežnik prekine: vsaka kamera, ki jo je odprl, se vrne v stanje, v katerem je bila najdena, in se zapre, ukaz izpiše `Error:` s korakom, do katerega je prišel, in se zaključi z `1`, pri čemer ni povezan noben niz (HTTP 504 `CLIENT_DEADLINE_PASSED`). Ctrl+C prekine povezavo na enak način (`Interrupted: …`); če je bil niz že povezan, to navede vrstica in imenuje ukaz `array-disconnect`, ki ga sprosti. Ukaz, ki je bil prekinjen med čakanjem (`kill -9`), povezava ostane prekinjena, če strežnik v ozadju 30 sekund ni prejel nobenega odgovora. Niz, ki je aktiven, a še nima sinhronizirane skupine okvirjev, se še vedno prikazuje kot povezan, njegov tok pa je označen kot `no_frames`. |
| `lattice array-disconnect [--array-id ID] [--all] [--keep-settings \| --clear-settings]` | Sprostitev niza. Vsaka kamera ostane v prostem delovanju (`TriggerMode` `Off`), pri čemer ostane vse ostalo takšno, kot je bilo ob vzpostavitvi povezave – nastavitve sprožilca in sinhronizacijske linije (glavni`TriggerSource` vrne v prvotno stanje, sinhronizacijsko linijo pa nazaj na vhod), PTP in nastavitve kamere, ki jih je spremenila seja — tako da ukaz za eno kamero na članu deluje takoj. `--keep-settings` shrani nastavitve vsake kamerenastavitve, kot jih je zapustila seja, v lastni pomnilnik kamere; `--clear-settings` ponastavi ta pomnilnik na tovarniške nastavitve. Glej [Pomnilnik kamere](#camera-memory----keep-settings----clear-settings). |
| `lattice array-list` | Seznam povezanih nizov. |
| `lattice array-status [--array-id ID]` | Število sličic na sekundo v živo, PTP, zadnja napaka. `ptp_enabled` je `True` le, če je pri vseh članih PTP vklopljen; `ptp_off:` navaja člana, pri katerem to ne velja. Vrstica `state:` bere `restarting (applying settings)` med ponovnim zagonom nastavitev, `recovering (…)` med ponovnim zagonom člana, ki je izgubil napajanje (glej opombo spodaj), `stalled (…)` medtem ko teče zank za zajem, vendar ne objavlja nobene skupine okvirjev, in `stopped (…)`, ko se zanka za zajem predaja (ponovno povežite niz). |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--orient] [--index\|--no-index] [--no-daq] [--daq-tolerance-ms MS] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | En sinhroniziran zajem iz aktivnega niza — Enkratni / Neprekinjen / Intervalni / Najhitrejši. **Privzeto je `all`**(ena datoteka na ustrezni tip izvoza na kamero),**usklajeno** z zajemom aplikacije (`--no-aligned` za izvorne datoteke). Preskočene kamere (npr. RGB, izključene iz sevanja/odbojnosti) se poročajo z `Skipped: SN:<serial> (<reason>)`; vrednost DAQ, ki je odčitala uporabljen posnetek, se shrani skupaj z njim, vključen je tudi posnetek v surovi obliki, poroča pa se z `DAQ: <path>` ([DAQ, ki odčita posnetek, shrani](#the-daq-reading-a-capture-saves)). Glej [Načini zajema, snemalniki in ponovna obdelava brez povezave](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only] [--expr EXPR]` | Snemanje kombiniranega indeksnega pogleda v realnem času v video/GIF (za namene spremljanja; če ni odprt noben grafični vmesnik, backend upodobi NDVI matrike ali `--expr`). |
| `lattice array-burst [--duration S] [--max-frames N] [--orient] [--build] [--products …]` | Serijsko zajemanje surovih podatkov Bayer z visoko hitrostjo (za analizo; ponovna obdelava v načinu brez povezave). Podobno kot `array-record` in `array-capture --continuous` se pri vsakem poizvedovanju prijavi pri strežniku: ustavi se brez prekinitve (`kill -9`), backend ga ustavi 30 s po zadnjem poročanju (`stop_reason` `client_gone`; manifest serije posnetkov je dokončan in se začne izdelava združenega indeksa, tako kot pri ustavitvi), niz pa ostane povezan. |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Ponovna obdelava shranjenega surovega zaporedja v kalibrirane videoposnetke. |

> **Kamera, ki izgubi napajanje ali povezavo, se sama ponovno priključi.** Ko član preneha odgovarjati (preklopnik ob ponovnem zagonu ponovno zažene vsako kamero, ki jo napaja), niz počaka nanjo, jo ponovno odpre, ko spet odgovori, in ji vrne nastavitve, ki jih je določil `array-connect`: PTP, sprožilec in sinhronizacijsko linijo, hitrost omrežja, format pikslov in velikost okvirja ter samodejno osvetlitev. Nato ponovno prenaša podatke brez novega `array-connect`. Medtem `array-status` prikazuje `state: recovering (...)`, `last_error` pa navaja, na katero kamero čaka; nato izpiše `re-opened:`. Kamera, ki se je ponovno zagnal, se vrne z izklopljenim PTP, zato je, dokler se ne odpre ponovno, `ptp_enabled` enako kot `False`, `ptp_off:` pa jo poimenuje.

> **Zadržana kamera se nikoli ne odpre drugič.** Drugo odprtje ustavi tok zadrževalca in prepiše njegove nastavitve, kar lahko povzroči sesutje strežnika. Torej na kameri, ki jo zadržuje zbirka (GUI ali `cam-connect`), članu niza ali kameri, ki jo zadržuje ozadna naloga (teče `lattice continuous`, `multi-stream` ali `align-stream`, lastna ali drugega odjemalca), `lattice continuous`, `lattice capture`, `cam-connect`, `power --sleep`, `calibrate`, enposkus `white-balance` in `presets --apply` zavrneta z HTTP-kodom 409 `CAMERA_HELD`, pri čemer navajata imetnika in ukaz za sprostitev (`chloros-cli lattice cam-disconnect -s <SN>`, `chloros-cli lattice array-disconnect --array-id <ID>` ali za nalogo njeno ID in kdo jo je začel) ter izhod `1`. Enako velja za kamero, ki jo trenutno uporablja drug ukaz (`multi-test` ali `multi-capture`, ki odpira svoje kamere, še en enkratni ukaz), za vsak ukaz tukaj: napaka pravi, da je treba počakati, da se ta ukaz konča. `lattice power` brez `--sleep` prebere prek lastnega ročaja imetnika, če ta obstaja (sklad, niz, odprta kamera naloge), in je zavrnjen, če kamero uporablja drug ukaz. Vsak odgovor `CAMERA_HELD` vsebuje kodo tako v `code` kot v `error_code`. Za ukaze, ki odprejo vse kamere, glej [Večkamerne (začasne seje)](#multi-camera-transient-sessions).

##### Možnosti `array-connect`

| Oznaka | Privzeto | Opis |
| --- | --- | --- |
| `--serials SN1,SN2,…` | samodejno odkrivanje vseh kamer LATTICE (potrebnih je ≥2) | Prva serijska številka je MASTER. Če se ne navede, se odkrivanje omeji na modele LATTICE (M3C / M3M) in poveže vse. |
| `--line {Line0,Line2,Line3}` | `Line2` | Sinhronizacijska linija GPIO. |
| `--target-fps F` | samodejno | Hitrost sprožanja glavne kamere. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | samodejno | Prepis izbire stopnje. |
| `--wire-ceiling-mbps MB_PER_S` | samodejno zaznano | **Trajna pasovna širina gostitelja v MB/s — številka, od katere je odvisna celotna dodelitev polja.** Zmanjšajte ga, ko niz poroča o okvirjih z napako GVSP: samodejna vrednost izhaja iz oglaševane hitrosti povezave mrežne kartice, ki precenjuje USB-adapterje, ozke PCIe-pasove in zasedene skupne strukture. Shranjeno v bloku zajema niza projekta, zato se ob ponovnem odprtju / ponovni povezavi prek CLI-ja / SDK-ja ponovno vzpostavi. Glej [Stanje polja](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | noben (celoten senzor) | Strojno združevanje (vedno povprečje). |
| `--width N` / `--height N` / `--pixel-format FMT` | celoten senzor; format po kameri | Izrecna velikost okvirja ali format pikslov za vsako kamero, namesto izbire med celotnim senzorjem in formatom po kameri (Mono12 / BayerRG12). |
| `--no-recommend` | izklopljeno | Preskoči korak analize omrežja (vsaka kamera nato ohrani svojo trenutno velikost namesto da bi bila nastavljena na celoten senzor). |
| `--no-ptp` | izklopljeno | Onemogoči PTP (časovni žigi med kamerami nato **niso** primerljivi). |
| `--packed {auto,on,off}` | `auto` | Pretakanje 12-bitnih okvirjev v pakirani obliki in razpakiranje na tem računalniku — več okvirjev na sekundo pri polnem senzorju, enaki podatki. `auto` pakira le, če je to koristno in če ta računalnik to zmorem; `off` nikoli ne pakira. Glej [Pakirani 12-bitni prenos](#packed-12-bit-transport---packed). |
| `--packet-size {auto,jumbo,standard,N}` | `auto` | Velikost paketa GVSP za vsako kamero v nizu. `auto` omogoča vsaki kameri, da preizkuša omrežje; `jumbo` = 9000, `standard` = 1500 ali velikost v bajtih jo omeji. Pred različico 1.3.0 je `array-connect` sprejel ta zastavico in jo ignoriral. |

> **Privzeto polni senzor.** Brez `--width` / `--height` / `--binning` /
> `--pixel-format`, `array-connect` nastavi vsako kamero na polni senzor — brez
> združevanja pik, brez izreza ROI — ter dopušča, da se število sličic na sekundo zniža na raven, ki jo omogočata ta računalnik in
> omrežje, namesto da bi se sličice pomanjšale, da bi se ohranilo število sličic na sekundo. V
> vrstici z analizo omrežja je prikazano predvideno število sličic na sekundo, v naslednji vrstici pa so navedene
> hitrejše možnosti (združevanje pik ali manjši ROI) s predvidenim številom sličic na sekundo; preskočite
> `--binning` ali `--width`/`--height`, da izberete eno od njih. Connect samodejno nikoli ne izvaja združevanja ali izreza
> slik.

##### Pakirani 12-bitni prenos (`--packed`)

Pri polni zmogljivosti senzorja je hitrost snemanja matrike odvisna od tega, kako hitro lahko vsaka kamera pošilja
svoje posnetke. 12-bitni formati (`BayerRG12`, `Mono12`) se običajno prenašajo kot 16-bitne
besede; **zgoščeni** prenos pošlje istih 12 bitov v 1,5 bajta na piksel in
jih razpakira na tem računalniku, zato pošiljanje vsakega okvirja traja 25 % manj časa —
približno **33 % več okvirjev na sekundo pri polni zmogljivosti senzorja** (izmerjeno: 10,7 → 14,0 fps
na nizu 9 kamer). **Shranjeni podatki so identični**: iste vrednosti, iste
16-bitne datoteke, isti metapodatki.

- `auto` (privzeto) pakira le, kadar je to koristno **in** kadar si ta računalnik lahko privošči
  razpakiranje. Ob povezavi backend izmeri lastno hitrost razpakiranja, število procesorskih jeder,
  trenutno obremenitev procesorja in razpoložljivo omrežno pasovno širino, tako da namizni računalnik z rezervnimi jedri stiska
  pri polni zmogljivosti senzorja, medtem ko zaseden ali računalnik z nizko porabo energije prenaša podatke razpakirane. Majhna območja zanimanja (ROI) in združeni posnetki prinesejo
  malo.
- `off` nikoli ne stiska – uporabite ga za onemogočanje stiskanja med diagnostiko ali na
  računalniku, ki mora ves svoj procesor ohraniti za drugo delo.
- `on` vedno pakira 12-bitne formate.

Izhod »connect« prikaže odločitev in razlog (`pakirano 12-bitno: na (avto:
napovedano 11,0 -> 14,4 fps, …)`); `/api/camera/array/status` poroča
`packed_transport` in `packed_decision`. Če želite nastaviti privzeto vrednost za celoten računalnik
namesto da vsakič posredujete zastavico, nastavite `CHLOROS_ARRAY_PACKED12=off` (ali
`on`) v okolju backenda — izrecno nastavljen `--packed on|off` ima še vedno prednost.
8-bitni in 16-bitni formati se nikoli ne pakirajo.

### En projektni zapis, vsi načini

Projektna mapa vsebuje nastavitve za vsako napravo: `cameras.json` – za vsako kamero (`cameras[].settings`, za posamezno kamero in za vsakega člana niza) ter nastavitve za celoten niz (`arrays[].capture`: format, ROI, združevanje slik, hitrost sprožanja, proračun za povezave) – ter `sensors.json`, za vsak svetlobni senzor (čas integracije, povprečje okvirja, omejitev). Aplikacija, CLI, SDK in strežnik MCP vsi berejo in zapisujejo iste datoteke na enak način:

- **Naprava, povezana v okviru projekta, se shrani v njem in vanj beleži vsako spremembo.** »V okviru projekta« pomeni, da je projekt odprt v ozadju (v aplikaciji) ali – iz SDK-ja – prek `ChlorosProject`, ki imenuje svojo mapo. Kamera, matrika ali svetlobni senzor, ki ga projekt še ne vsebuje, se doda v njegove datoteke, ko se poveže (`lattice cam-connect`, `array-connect`, `daq pool-connect`, aplikacija, SDK, `chloros-mcp`), skupaj z njenimi nastavitvami. Sprememba iz katerega koli načina (`lattice exposure`, `gain`, `configure --set`, `resolution`, nastavitve matrikein hitrost sprožanja, pokrov svetlobnega senzorja, upravljalniki aplikacije) se trenutek kasneje shrani v datoteke tega projekta. Nastavljena osvetlitev, ojačitev ali ciljna vrednost avtomatske ekspozicije (AE) na kameri se shrani kot ohranjena, tako da se ob ponovnem odprtju ponovno ohrani.
- **Ob odprtju projekta se naprave nastavijo v skladu z njim**, v vsakem načinu: kamera, matrika ali svetlobni senzor, ki je priključen brez lastnih nastavitev, prevzame shranjene nastavitve projekta (`lattice cam-connect` / `array-connect` ob odprtem projektu, `chloros-cli project connect`, `open_project().connect_all()`, ponovno odprtje aplikacije). Niz, priključen pod novo identifikacijsko številko, posodobi niz, ki ga je projekt shranil z istimi kamerami; niz z drugimi kamerami se doda poleg njega.
- **Lastno shranjevanje aplikacije nikoli ne prepiše trenutnih nastavitev naprave** s svojo starejšo kopijo teh nastavitev in ohrani shranjene prekrivke za posamezne člane, ki jih je shranil klicatelj prek CLI-ja ali SDK-ja.
- **Pikselni format, ki ga nastavite, je tisti, ki ga projekt ohrani.** Format, nastavljen v nastavitvah niza, prek `--pixel-format` ali `connect_array(pixel_format=)`, vključno z 8-bitnim, se shrani in ponovno poveže natančno. Če ni nič nastavljeno, se vsaka kamera poveže s svojo privzeto 12-bitno nastavitvijo (`BayerRG12`, `Mono12` na monokromatskem senzorju). Ime formata označuje njegovo bitno globino; družina se nanaša na senzor, zato barvna kamera nikoli ni nastavljena na mono format, niti mono kamera na Bayerjev format, kar je navedeno v vrstici `[PixelFormat <serial>] ... written`, ko je bil format dodeljen. Izvozi 8-bitni posnetek se izvozi v 8-bitni obliki, vključno z odbojnostjo: ponovno skaliran in označen z `PixelScale`.
- Naprava, priključena brez projekta, ne snema ničesar.
- **Snemanje sledi projektu, ustvarjenemu po njem; odprtje ga pusti v svojem projektu.**Ko se ustvari projekt – SDK-jev `create_project()`, `chloros-mcp`-ov `create_project` ali aplikacijski**Nov projekt**— ali podvojen (možnost**Podvoji**v aplikaciji, ki odpre kopijo), medtem ko snemanje še vedno zapisuje v projekt, ki je bil odprt prej, se snemanje nadaljuje, zapisano pa se shrani v novem projektu, na istem mestu znotraj njega (spodaj). Odpiranje obstoječega projekta — `chloros-cli process`, `project devices --reconnect`, `ChlorosProject.process()`, funkcija**Odpri projekt** v aplikaciji — pusti snemanje še vedno v teku v projektu, v katerega se snema: tam se konča, njegove datoteke pa se pridružijo seznamu datotek tega projekta (aplikacija ustavi vsako snemanje, preden odpre projekt).
  - `array-record` in `array-burst`: video ali mapa z vsemi posameznimi sličicami se tja prenese v celoti, ko se snemanje ustavi (njegov `--duration`, Ctrl+C). `Burst dir`, ki ga izpiše, je nov, `--build` in `array-build-video` pa delujeta na podlagi njega. Če pogon zavrne premik, snemanje ostane tam, kjer je bilo zapisano;dnevnik navaja razlog, prav tako pa tudi HTTP-odgovor ob ustavitvi (`relocation_error`). Surovi okvirji ustavljenega zaporedja in `.daq` se dodajo na seznam datotek projekta, v katerega se uvrstijo; video (`array-record`, iz serije posnetkov sestavljen video) pa se ne doda na seznam datotek projekta.
  - `array-capture --continuous`: vsak posnetek po spremembi se shrani v nov projekt; tisti, ki so bili shranjeni prej, ostanejo tam, kjer so.
  - `daq pool-record` in `project sensor log` (ter snemanje z lastnim svetlobnim senzorjem aplikacije): datoteka `.daq` se po koncu snemanja premakne v mapo novega projektamapo `light_sensor`, ko se snemanje ustavi, in se pridruži seznamu datotek tega projekta.

  Snemanje v mapo zunaj odprtega projekta (na primer `-o` nekje drugje) ostane tam, kjer je, `lattice continuous` in `multi-stream` pa ohranita mapo, ki jima je bila dodeljena.
- **V aplikaciji se posnetki po ustvarjanju novega projekta shranijo v ta projekt.** Ustvarjanje ali podvajanje projekta ohrani vsak tekoči posnetek in snemanje, vsak posnetek, narejen po tem (Posnemi vse, Začni snemati), pa se shrani v nov projekt in se doda na njegov seznam datotek. Odpiranje obstoječega projekta v aplikaciji najprej ustavi vse te procese.
- **Projekt se med snemanjem ne premesti.**Premikanje odprtega projekta v drug delovni imenik (v aplikaciji**Nastavitve → Delovni imenik**; CLI-ukaz ne premesti projekta, to pa stori HTTP-odjemalec, ki pošilja na `/api/move-project`) se zavrne s HTTP 409 `PROJECT_RECORDING`, medtem ko v njo pišejo: snemanje matrike ali serija posnetkov, neprekinjeno zajemanje matrike, opravlja naloga `lattice continuous` / `multi-stream` ali snemanje s svetlobnim senzorjem vanj zapisuje: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.` Aplikacija zavrne na enak način, preden zaprosi za mapo, medtem ko teče njena lastna funkcija »Capture All«, »Start Capturing« ali snemanje s svetlobnim senzorjem.

To je ločeno od tega, kakšne so nastavitve kamer ob prekinitvi povezave (glej [Pomnilnik kamere](#camera-memory----keep-settings----clear-settings)): projekt sam nastavi svoje kamere ob zagonu.

### Pomnilnik kamere — `--keep-settings` / `--clear-settings`

Ko je niz povezan, kamere prevzamejo nastavitve niza, ki jih lahko nato spremenite. Ko program Chloros sprosti kamere — `array-disconnect`, konec prehodnega ukaza (`multi-capture` brez niza, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`), zaustavitev `lattice continuous`, konec `lattice capture` kamere, ki je ni v lasti — vsako vrne v stanje, v kakršnem je bila pred povezavo: njena ožičenja sprožilca in sinhronizacijskega vodnika, nastavitve PTP in kamere (celoten seznam je na voljo v [Večkamerni sistem (začasne seje)](#multi-camera-transient-sessions)).

Za uporabo kamer zunaj projekta dva zastavka na katerem koli od teh ukazov zapišeta v **lastni pomnilnik** kamere — tisto, s čimer se kamera vklopi v kateri koli programski opremi:

- **`--keep-settings`** shrani nastavitve vsake kamere v njen pomnilnik tako, kot so bile ob koncu seje (UserSet2, ki je postal zagonski nabor). Kamere še vedno delujejo samostojno (`TriggerMode` `Off`), ožičenje sinhronizacijskega vodila pa ostane takšno, kot je bilo najdeno. Kamera brez UserSet2 shrani v UserSet1; ta računalnik si to zapomni, zato se Chlorosov niz samodejno popravi — kar ponastavi UserSet1, ki se je oddaljil od Chlorosove izhodiščne vrednosti — ga pusti pri miru.
- **`--clear-settings`** ponastavi pomnilnik vsake kamere na tovarniške nastavitve: Chlorosovo izhodišče, tisto, ki ga matrika samodejno popravi — tovarniške privzete nastavitve s `TriggerMode`, `Off`, `TriggerSource` in `Line2`, avtomatska osvetlitev, ohranjena kalibrirana bela bilanca RGB-kamere — v UserSet1, ki je postal zagonski nabor.
- Oboje ni mogoče kombinirati. Vsaka kamera dobi vrstico: `SN:…: settings saved to the camera's memory (UserSet2); it powers up with them.` ali `SN:…: camera memory reset to the factory settings.`; odgovor `camera_memory` je enak za vsako kamero.

Aplikacija teh zastavic ne ponuja.

**Vsak izhod vrne kamere v prvotno stanje.**Enako vrnjenje v prvotno stanje se izvede na vsaki izhodni poti, ne le pri zgornjih ukazih: `cam-disconnect` (in kamera**X**v aplikaciji); aplikacijski**X** na nizu ali v vrstici katerega koli člana niza (niz ne more delovati brez člana, zato se sprosti celoten niz); zapiranje aplikacije (backendu da do 20 s časa, da vrne vsako kamero, niz in tekoči `lattice continuous` nazaj, preden ga ustavi; ukaz, ki takrat še vedno teče — prehodni ukaz, ki odpira svoje kamere ali čaka na njihove PTP-ure, enkratni `lattice capture`, `power`, `white-balance` ali `presets --apply` – se ustavi na naslednji varni točki, najprej ponovno vzpostavi kamere, kar traja do 8 s, in odgovori z HTTP-odgovorom 503 `SHUTTING_DOWN`; povezava v obliki niza, ki še vedno teče – `lattice array-connect`, povezava aplikacije – se ustavi pred naslednjo kamero in ponovno zapre kamere, ki jih je odprla, za največ 10 s, in odgovori z HTTP 503 `SHUTTING_DOWN`; tista, ki prispe med zaustavitvijo, pa ničesar ne odpre: HTTP 503 `SHUTTING_DOWN`); in ukaz CLI, ki je prekinjen ali katerega terminal se zapre (SIGTERM / SIGHUP ustavita njegovo delo — in vrne kamere v prvotno stanje — preden se zapre `143` / `129`, podobno kot Ctrl+C). `lattice continuous`, `multi-stream` ali `align-stream`, ki je bil takoj ukinjen (SIGKILL) in ne more ničesar ustaviti, ima svojo nalogo ustavljeno, kamere pa jih backend vrne v prvotno stanje 30 s po zadnjem poročanju. Backend, ki se sesuje ali je prisilno ustavljen, ne more ničesar ponovno vzpostaviti; ponovni zagon vrne vsako kamero v njen pomnilnik, naslednja povezava s Chlorosom pa ponovno poveže kamero, ki jo najde v stanju pripravljenosti.**Projekt sam nastavi svoje kamere.** Ko se niz poveže z odprtim projektom, vsak član prejme nastavitve, ki jih je projekt shranil zanj – ekspozicijo, ojačanje ali ciljno vrednost AE, ki je zanj shranjena (shranjena kot vaša, zato jo avtomatska ekspozicija pusti pri miru) ter način uravnoteženja beline (korekcija Chloros; lastno uravnoteženje beline kamere ostane izklopljeno) – ne glede na to, kaj je shranjeno v pomnilniku kamere.

### Smart-AE / Smart-Capture

Nizi LATTICE v ozadju izvajajo neprekinjeno avtomatsko osvetlitev (AE), takoj ko se povežejo, vendar pa traja nekaj trenutkov, da se novo usmerjena scena stabilizira. `array-capture --smart` je **pripravljena rešitev**: počaka, da se avtomatska osvetlitev ustalijo na vseh kamerah v nizu, nato pa sproži zajem. Uporabite ga, ko med sejo spremenite prizor.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Privzeto je politika stabilizacije konzervativna: časovni limit 5 s, okno stabilnosti 1,5 s, toleranca razpona osvetlitve ±5 %. Če potrebujete drugačno delovanje avtomatizacije, jo prilagodite prek SDK-ja (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`).

#### Kako deluje avtomatska osvetlitev

- **Najprej osvetlitev, na koncu ojačanje.** Ko je ojačanje nastavljeno na »Auto«, ostane na 0 dB, dokler osvetlitev ne doseže najdaljše dovoljene vrednosti.
- **Pri ročni osvetlitvi ostane ojačanje v načinu »Auto«**, dokler ne spremenite ojačanja ali ga ne nastavite na ročni način. Ko se vrnete v način »Auto«, je omejitev ojačanja enaka prejšnji.
- **Majhni svetli deli ne potemnijo prizora.** Odsev, ki zavzema do 1 % kadra, se ignorira, v panelih znanega kalibracijskega cilja pa do 0,5 % posameznega panela.
- **Zgornja meja osvetlitve DAQ velja le, dokler je kalibracijski cilj v vidnem polju.** Preprečuje preklipanje belega panela; če panel ni viden, kamera izpostavi sliko za prizor.
- **Pri konstantni svetlobi ostanejo vrednosti nespremenjene, spremenjena svetloba pa se ustalijo v nekaj sekundah.** Pri konstantni svetlobi ostanejo osvetlitev in njena meja nespremenjeni, po večji spremembi svetlobe pa se slika ustalijo v nekaj sekundah. Izrez ne omejuje več osvetlitve za vedno, sprememba območja zanimanja (ROI) pa ponovno najde ustrezno osvetlitev.
- **Ciljna svetlost ostane v dosegu.** Ne narašča, dokler je kamera na svoji meji osvetlitve in se njen dobiček ne more povečati, zato svetloba, ki se vrne, ne povzroči preosvetlitve slike.
- **Kamera, katere avtomatska osvetlitev ne odziva več, ostane nastavljena na raven, ki ustreza trenutni svetlobi.** V močnem soncu bi kamera lahko padla na najkrajšo osvetlitev in tam ostala s temnim kadrom, vse njene nastavitve pa so normalne. Program Chloros spremlja sliko, ohranja kamero na ravni, ki jo je vzdrževala za to svetlobo, in jo sprosti, ko se svetloba spremeni. Temni ali zakriti svetlobni senzor tega ne moti.

### Ravni zajema in izvoza (privzeta nastavitev `all`)

Od te izdaje, `lattice capture`, `lattice multi-capture` in `lattice array-capture` **privzeto na `--processing all`** — ena shranjena datoteka na vrsto izvoza, ki velja za vsako kamero, kar ustreza delovanju funkcije »Capture All« v grafičnem vmesniku. Ravni so:

| Raven | Izhod | Velja za |
| --- | --- | --- |
| `raw` | Enokanalni Bayer (črno-bele kamere: en sam pas) neposredno iz senzorja. | Vse kamere. |
| `debayered` | 3-kanalna barva, shranjena v vrstnem redu kanalov **RGB**(datoteka TIFF s fotometrično interpretacijo RGB; datoteke PNG/JPEG se prav tako odprejo kot RGB): temni odmik (DSNU) iz kalibracijskega paketa, odvzet iz surovega podatka, nato pa demosaikiranje z upoštevanjem robov iz OpenCV— brez LUT za linearnost, izravnave polja ali razmešanja (črno-bele kamere: 1-kanalna siva lestvica). Okvirji brez bayerske matrike v pomnilniku, ki jih vrne Python SDK (`grab()`, `frame_stream()`), so v zaporedju**BGR**, kot ga določa OpenCV; samo shranjena datoteka je v formatu RGB. | Vse kamere. |
| `radiance` | float32 W/m²/sr/nm prek celotne radiometrične verige. | Samo multispektralne kamere (M3C/M3M) — **preskočeno za kamere s RGB-filtrom**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), pripravljeno za Pix4D. | Samo multispektralno in **samo, če je vezan DAQ + je kamera kalibrirana**; sicer se preskoči. |
| `display` | Celotna veriga predogleda v grafičnem vmesniku (CCM + WB + gama v skladu s profilom kamere). Shranjeno v mapi `preview/`. `lattice capture` za to sprejema tudi ime `preview`; `array-capture` / `multi-capture` sprejemata le `display`. | Vse kamere. |

Če navedete eno samo stopnjo, se shrani le ta (`--processing debayered`). Ko zahtevate `all`, se stopnje, ki ne veljajo za dano kamero, preskočijo (in se o tem poroča), ne pa da se pojavi napaka — nepovezana ali nekalibrirana kamera še vedno prejme `raw` / `debayered` / `preview`.

#### DAQ, ki bere zajem, shrani

Vsakič, ko je DAQ povezan s kamero ali nizom kamer ali združen v skupino, se odčitek navzdolnega toka, uporabljen pri zajemu, zapiše v datoteko **`.daq`** v podmapo `daq/` mape »captures«, skupaj z vrednostjo `.match.json` (kako daleč je bila vrednost od osvetlitve) in `.cameras.json` (kateremu pripada vrednost), ter prikazana v vrstici `DAQ:`. To velja za vsako raven, vključno z neobdelanimi ali debayered posnetki, tako da je mogoče vsak posnetek kasneje ponovno obdelati v radianco, odbojnost ali indeks. Gre za isto datoteko z istim imenom in spremljajočimi datotekami, ne glede na to, ali je zajem izviral iz grafičnega vmesnika (GUI), ukazne vrstice (CLI) ali razvojnega kompleta (SDK). Če ni nobene meritve DAQ, ni datoteke `.daq`, datoteka `multi-capture` brez povezanega niza (začasna rešitev) ne shrani ničesar.

| Oznaka | Učinek |
| --- | --- |
| `--no-daq` | Ne shranjuj odčitka pri zajemu, katerega ravni ga ne uporabljajo (`raw`, `debayered`). Raven `radiance`, `reflectance` ali `display` jo še vedno zapiše, ker ponovni uvoz potrebuje odčitek, s katerim je bil izdelek narejen. |
| `--daq-tolerance-ms MS` | Koliko se lahko odčitek razlikuje od izpostavljenosti in še vedno velja za v toleranci (privzeto 500). Spremeni se le ocena (za `.match.json`, `in_tolerance` in `tolerance_ms` ter za odbojnost `DlsInTolerance`), nikoli pa se ne spremeni uporabljena vrednost. Glej [Meritev DAQ, ki je daleč od osvetlitve](#a-daq-reading-far-from-the-exposure). |
| `--force-daq` | Starejši način zahtevanja vrednosti `.daq` pri zajemu, ki vsebuje le surove podatke. Zdaj je to privzeta nastavitev, vrednosti `lattice capture` in `array-capture` še vedno sprejemata ta zastavico (`multi-capture` pa ne). `--no-daq` ima prednost pred njima. |

Kateri DAQ: tisti, ki je vezan na kamero ali niz (izbira svetlobnega senzorja v grafičnem vmesniku, shranjena vezava projekta ali `chloros-cli daq pool-bind`), sicer pa najnovejši, ki ga ima backend. Serija surovih podatkov (`array-burst`) ohrani svoj `.daq` za vsako posamezno odčitavanje v mapi serije, kot prej.

**Datoteki PNG in JPEG imata 8 bitov na kanal, zato sprejemata le `display` / `preview` in indeksne produkte.** `--format png` ali `jpg` z ravnjo, ki je ne morejo shraniti, se zavrnejo še pred zajemom, pri čemer se navede raven in TIFF kot format, ki jo shranjuje:

- `radiance` (float32) in `reflectance` (uint16, `32768` = 1,0) — vedno, zato je zavrnjen tudi `--processing all` (privzeto) z `--format png`/`jpg`: uporabite `--format tiff` ali `--levels display`.
- `raw` in `debayered` iz kamere, ki prenaša več kot 8 bitov (12-bitni okvirji M3C / Mono12 M3M) — CLI ne more prepoznati bitne globine kamere, zato jih strežnik zavrne (HTTP 400). Posameznislikovni `lattice capture --processing debayered` zajame 8-bitni okvir za formate PNG in JPEG ter ni zavrnjen, prav tako pa ni zavrnjen `multi-capture` brez trajnega niza (8-bitni brez debayeringa).

Vsaka pot zajemanja odgovarja na enak način (`error_code` `FORMAT_CANNOT_HOLD_LEVEL`): `lattice capture`, `multi-capture`, `array-capture`, `project capture` / `burst` ter HTTP-poti za njimi.

### Kako izgleda mapa z zajetimi posnetki

Vsaka vrsta izvoza se shrani v **lastno podmapo**pod `-o`, tako da se pri večstopenjskem zajemanju vrste nikoli ne mešajo. Na skici je prikazan**privzeti zajem `all` ene kamere s programom `lattice capture`** — več ravni na kamero, zato ima vsaka datoteka končnico `_<level>`:

```
output/
├── raw/           capture_<ts>_SN<serial>-<filter>_raw.tiff
├── debayered/     capture_<ts>_SN<serial>-<filter>_debayered.tiff
├── radiance/      capture_<ts>_SN<serial>-<filter>_radiance.tiff
├── reflectance/   capture_<ts>_SN<serial>-<filter>_reflectance.tiff
├── preview/       capture_<ts>_SN<serial>-<filter>_display.tiff
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_radiance_clampmask.tif, reflectance/…, index/… — the unmix clamp
│                  masks, one subfolder per product folder, only when a band clamped
└── daq/           daq_data_<sensor id>_<ts>.daq: the downwelling reading matched to the capture
                   (+ .daq.cameras.json: whose)
```

Kako se imena datotek razlikujejo:

- **Polje kamere — serijska številka, nato filter:** vsaka zajeta datoteka nosi ime kamere, ki jo je posnela, v obliki `SN<serial>-<filter>`, kjer je `<filter>` del modela kamere, ki označuje filter: `SN261902142-F450` za `M3M-L87-F450`, `SN213800234-FRGN` za model `M3C-L87-FRGN`, `SN214701292-FRGB` za RGB-kamero. Filter se pridruži serijski številki v obliki `-`, ne pa z `_`, zato ime vsebuje enaka polja, ločena z `_`, kot prej (predpona, datum, čas, milisekunde, kamera, raven). Kamera, katere modela ni mogoče prebrati (izbrisan DeviceUserID), se imenuje samo `SN<serial>`. Posnetki iz prejšnjih izdaj nimajo filtra v imenu; Chloros bere oba.
- **Prefiks:** `lattice capture` poimenuje datoteke `capture_…`; `array-capture`, `multi-capture` s povezano serijo ter `project capture` / `burst` / `stream`, ki so del niza, pa jih poimenuje `sync_<ts>_SN<serial>-<filter>…`.
- **Pripona ravni se uporablja le pri več ravneh.**Kamera, ki v enem zajemu shrani**več**ravni (`all` ali `--levels a,b`), doda `_<level>`. Kamera, ki shrani**eno** raven, ohrani preprosto ime brez končnice – `--processing reflectance` zapiše `reflectance/sync_<ts>_SN<serial>-<filter>.tiff` –, tako da skripti, napisani za posnetke z eno ravnjo, še naprej delujejo. Raven je še vedno določena s pomočjo mape.
- **Maske omejevanja sledijo imenu izdelka:** `<product name>_clampmask.tif`. Posnetek odbojnosti z eno stopnjo zapiše `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`; posnetek `all` zapiše `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`.
- **Pripona** sledi `--format`: privzeta vrednost `tiff` da `.tiff`, `--format tif` pa da `.tif` (maske omejitve so vedno `.tif`).
- **Ena izjema:** `lattice capture --processing debayered` (ali `--levels debayered`) vzame hitro enojno- in zapiše `capture_<ts>_SN<serial>-<filter>.tiff` neposredno v `-o` — brez podmap.
- **Tokovi in serije** prenašajo isto polje: `lattice continuous` zapiše `frame_<seq>_SN<serial>-<filter>.tiff`, `lattice multi-stream`, `stream_<seq>_SN<serial>-<filter>_<frame id>.tiff` in surovo serijo `<burst>_f<frame>_SN<serial>-<filter>_raw.tiff`.

`<ts>` je časovni žig zajema, `<serial>` pa serijska številka kamere, tako da si ena sinhronizirana skupina deli
časovni žig med kamerami. **Upoštevajte eno nesimetričnost:** raven `display` je shranjena v mapi
z imenom `preview/`, medtem ko večravenske datoteke ohranjajo `_display` v imenu — mapa in končnica se razlikujeta
samo za to raven. Neznane ravni se shranijo v mapo z lastnim imenom, če pa podmapo
ni mogoče ustvariti, se datoteka zapiše v korensko mapo izhodnih datotek, namesto da bi se izgubila.

**Ponovna obdelava mape posnetkov:**usmerite `chloros-cli process` na**koren mapo »captures«**
(`output/`). `process` običajno uvozi le mapo, ki jo imenujete, vendar če ta mapa ne vsebuje
slik, pa vsebuje podmape, se samodejno spusti v globino — tako se podmape na vseh ravneh in
podmapo `daq/` zajamejo v enem zamahu. Vsaka raven zajema se uvozi kot ena sama slika,
ostale ravni pa so na voljo kot načini, namesto da bi bila vsaka raven uvožena kot posamezna slika.

Deluje tudi neposredno poimenovanje **podmape ravni** (npr. `output/raw/`). Pri tem ostane `daq/`
za sabo, zato kopirajte ali usmerite odčitek DAQ-ja ob strani, koizpeljujete radiometrični
produkt iz `raw/` – sicer se časovni žig ne more ujemati z ničemer.

**Obdelava se vedno začne z `raw`.** Znotraj vsakega zajema je surovi okvir vir za obdelavo;
`debayered`, `radiance`, `reflectance` in `preview` so na voljo kot načini prikaza, vendar se nikoli ne vračajo
nazaj skozi obdelovalno verigo. Ponovna obdelava izpeljanega izdelka bi ponovno uporabila vinjeto, CCM in
matematične izračune sevanja, ki so že vgrajeni v njegove piksle, zato Chloros to zavrne, namesto da bi
izvedel dvojno obdelavo. Dve posledici, ki jih je vredno poznati:

- Renderji `index/` in `composite/` **nikoli** obdelani. So izhodni podatki, ne zajeti posnetki —
  render LUT-a NDVI nima smiselne interpretacije sevanja.
- Mapa zajetih posnetkov, izvožena **brez** `raw` (npr. `array-capture --processing reflectance`), nima
  veljavnega vira v delovnem toku. Ti posnetki se uvozijo in prikažejo normalno, vendar jih `process` preskoči
  in to tudi sporoči:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Če resnično potrebujete, da se izpelje izpeljani izdelek – sejo vozlišča, zajeto z
  vklopljenim `demosaic` ali iz stare mape –, `--input-level {raw,debayered,processed}` prisili vstopno
  točko in prekliče preskok. Ta zastavica je namerno vgrajena rešitev; `auto` (privzeto)
  nikoli ne obdela zajema, ki nima surovih podatkov.

**Oznake kalibracije: samo na `raw`.** Datoteka `raw` zajema vsebuje kalibracijski paket kamere: koristni tovor `CalibrationJSON` in oznake, združljive z MicaSense (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), ki opisujejo surove vrednosti senzorja. Vse ostale datoteke na tej ravni (`debayered`, `display`, `radiance`, `reflectance`, render `index/`) ter vsak okvir `lattice continuous` / `multi-stream` okvir vsebuje identifikacijo kamere (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) ter lastni zapis okvirja, `XMP-mapir:FrameJSON` (osvetlitev, ojačanje, temperatura, format pikslov, čas zajema, DAQ ujemanje), saj ima vsak izdelek `process` svojo identifikacijo. `process` poišče kalibracijo takšne datoteke po njeni serijski številki, `CalBundleSha256` pa v kalibracijskem predpomnilniku Chlorosa, pri čemer paket prenese le enkrat, če ga ga računalnik nima.

#### Datoteke, ustvarjene iz več kamer

Datoteka, sestavljena iz pasov več kamer – večpasovni niz (`align-apply`, `align-stream`, `index --live --save-multiband`), indeks v realnem času, sestavljen iz več kamer (`index.tif`, `index_LUT.png`), slika `composite/`, posneta z matriko – navaja vsako kamero, ki je bila vključena vanjo:

- **Seznami po posameznih kamerah, v vrstnem redu pasov** (kompozit: ozadje, nato prednji plan; indeksna plast, kamere, iz katerih izvirajo njeni pasovi): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (kalibracijski paket, skozi katerega so šli pasovi vsake kamere, sicer — niz `align-apply` / `align-stream`, katerega pasovi niso bili kalibrirani — paket kamere, ki ga je Chloros shranil v predpomnilnik, tisti, po katerem so poimenovane njene datoteke posnetkov), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — vzporedni seznami, pri čemer *n*-ti element vsakega opisuje isto kamero; `-`, kjer kamera nima vrednosti (ni kalibracijskega paketa, neznan objektiv). Vsaka kamera je **ena** postavka: če ima njen paket več vrednosti za določeno polje — na primer certifikati za halogenske žarnice v paketu — so te povezane z `;` (`F-1479-1;F-1480-1;F-1481-1`), nikoli z vejico, tako da seznam prikazuje eno postavko na kamero, ne glede na to, kako je natisnjen.
- **EXIF `SerialNumber`** vsebuje iste serijske številke, povezane z vejico (`214000535,214000551`).
- Ne vsebuje **nobene** identifikacije posamezne kamere (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): ti imenujejo eno kamero in njen paket, `process` pa bi na podlagi njih določil kalibracijo datoteke. Indeks, izračunan iz pasov več kamer (`index.tif`, `index_LUT.png`) vsebuje EXIF-podatke `Model` `LATTICE` (sistem), ne pa model `LATT-…` ene same kamere.

Datoteka pasov **ene** kamere — datoteka TIFF (`--per-band`), posamezni `align-apply --per-camera` okvir, niz ali indeks v realnem času ene same kamere – vsebuje identifikacijske oznake te kamere, tako kot vsaka druga datoteka posnetka. Zbirka datotek `align-apply`, datoteke po posameznih pasovih in po posameznih kamerah ter vsaka zbirka datotek `align-stream` in njene datoteke po posameznih pasovih vsebujejo čas zajema svoje skupine (osvetlitev referenčne kamere, UTC), in `--per-camera`-okvir enokanalne kamere je ena ravnina, pri `--bit-depth` pa je podoben sklopu.**Usmeritev kamere (preobrnitev / vrtenje).** Nastavitev usmeritve kamere (v grafičnem vmesniku Mirror H /
Mirror V / rotacija) v grafičnem vmesniku določa, kaj se pri zajemu shrani:

- **`raw` in `debayered` se shranita točno tako, kot ju je senzor prebral**, ne glede na nastavitev.
  Bayerjev mozaik, ki je obrnjen ali preobrnjen, bi predstavljal drugačen vzorec barvnih filtrov, kalibracijske
  karte (dark offset, flat field) pa so na lastni mreži senzorja.
- **`preview`/`display`, `radiance`, `reflectance` in render `index/` so usmerjeni** — zavrti
  enkrat, po vseh kalibracijskih izračunih, tako da se odprejo tako, kot je bila kamera usmerjena. Njihove
  datoteke `clamp_masks/…_clampmask.tif` so na isti mreži.
- **Vsaka datoteka posnetka beleži nastavitev** v XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, v smeri urinega kazalca, po
  preobratih) ter `ChlorosOrientationApplied` — `false` na surovih in debayered (izvirnih pikslov),
  `true` na usmerjenih izdelkih. `ChlorosProcessing` navaja operacije, uporabljene na
  pikslah te datoteke (`none`, če so izvirni). To vključuje vsak okvir, ki ga prenaša `lattice continuous` (
  lastna nastavitev kamere, neuporabljena), in vsak okvir `lattice multi-stream` (identiteta: njegovi
  članski okvirji so izvirni, kot pri zajemu matrike). `lattice align-stream` / `align-apply`
  skladi zabeležijo nastavitve glavne kamere, ki niso bile uporabljene: njihovi pasovi so deformirani v okvir glavne kamere,
  vendar niso zavrti.
- **`process` uporabi zabeleženo orientacijo**, ko obdeluje surove podatke: najprej temni odmik, izravnavo
  polja, demosaik in ločevanje v koordinatah senzorja, nato izkrivljenje poravnave matrike,
  usmeritev — tako da so izdelki predogleda, sevanja, odbojnosti in indeksa popolnoma enaki
  kot pri zajemu v živo, in zabeleži `ChlorosOrientationApplied=true`. Nekalibrirani nadomestni
  (`Sensor_Response_Images`, `Vignette_Corrected_Images`) so usmerjeni na enak način, po
  izravnave polja. Produkt brez Bayesove matrike ostane v izvirni obliki.
- **Chlorosov pregledovalnik slik prikazuje surovo ali de-bayersko datoteko, obrnjeno glede na njen zapis** (samo za prikaz —
  datoteka se ne spremeni), tako da izgleda kot njeni produkti; miniatura, pogled v polni velikosti,
  odčitek kurzorja in peskovnik Index/LUT uporabljajo isto rotacijo. Datoteka brez zapisa in
  produkt (že usmerjen) se prikažeta tako, kot sta shranjena. Izvoz v peskovnik zabeleži orientacijo, ki jo
  nosijo njegovi piksli (`ChlorosOrientationApplied=true`; identifikator za sloj brez zapisa).
- **Posnetki brez zapisa se obdelujejo kot izvirni** (nespremenjeno delovanje). Pred različico 1.3.0 je
  posnetek z eno kamero, pri katerem je bila nastavljena obrnitev ali rotacija, sam obrnil svoje surove in debayirane datoteke;
  program Chloros ne more razlikovati takšne že-obrnjene surove datoteke razlikovati od izvirne in ne ugiba, zato jo
  obdeluje tako, kot je shranjena. Kalibracijske karte so na lastni mreži senzorja, zato za takšen
  posnetek uporabite tistega, posnetega z različico 1.3.0, kadar je kalibracija pomembna.
- **Matrike privzeto ne obračajo nobene posamezne datoteke** (poravnava in XMP datoteka `Alignment*` sta v
  lastnih, neobračenih okvirjih kamer) ter zabeležijo identiteto. `array-capture --orient` obrne
  izdelke vsakega člana po poravnalni deformaciji: glede na orientacijo **glavnega**, ko so bili
  deformirani v njegov okvir (tako da člani ostanejo usklajeni), sicer pa glede na orientacijo lastne kamere. Vsaka
  datoteka beleži orientacijo okvirja, v katerem so njeni lastni piksli – surova datoteka, orientacija lastne kamere –
  in surova datoteka, ki vsebuje transformacijo poravnave, prav tako beleži `ChlorosAlignedFlipHorizontal` /
  `ChlorosAlignedFlipVertical` / `ChlorosAlignedRotation`: glavnega posnetka, za izdelek, ki je bil izkrivljen
  v okvir glavne kamere. `process` vsak izdelek zavrti za vrednost, ki pripada njegovim pikslom:
  usklajeno, ko izkrivlja surovo datoteko (`--array-alignment`, privzeto — izdelki v živo usklajenega
  posnetka), lastni iz kamere, ko tega ne stori (`--no-array-alignment` – iz
  neusklajenega zajema v živo). Enaka možnost obstaja za `array-burst`, `project capture` / `burst` /
  `stream`, receptih in SDK.

### Preskočeni posnetki v nizih z mešanimi filtri

Ko v enem nizu združite RGB- in multispektralne kamere, `array-capture --processing radiance` (ali `reflectance`) shrani multispektralne posnetke in **preskoči** RGB-kamere — sevanje na posamezni Bayerjevi točki namreč nima pomena za širokopasovni senzor. CLI izpiše vsako shranjeno datoteko (z njeno stopnjo izvoza), vsak zapisani `.daq` in vsak preskok izrecno, zato število datotek ni presenetljivo:

```
  Saved: …/output/reflectance/sync_…_SN213800234-FRGN.tiff [reflectance] (SN:213800234, fid:1)
  Saved: …/output/reflectance/sync_…_SN214000533-FOCN.tiff [reflectance] (SN:214000533, fid:1)
  Saved: …/output/reflectance/sync_…_SN214701288-FNGB.tiff [reflectance] (SN:214701288, fid:1)
  DAQ:   …/output/daq/daq_data_daq-e-54b5e0_….daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Oznake razlogov za preskok sledijo vzorcu `<level>-not-applicable-to-rgb-cam`. Kamera, katere kalibracijski paket ni naložen, preskoči `radiance` / `reflectance` z `<level>-needs-calibration (connect the camera so its cal pack loads)`, brez uporabne odbojnosti, izmerjene z DAQ, pa preskoči z `reflectance-skipped-no-dls`. Odbojnost se lahko preskoči tudi z `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, in z `dls-uncalibrated-band-<nm>`, kadar pas večinoma leži zunaj radiometrično kalibriranega območja svetlobnega senzorja DAQ (~374–974 nm) — med dostavljenimi artikli le F988, katerega podprta pot je delovni tok z odbojnostnim panelom.

Uporabite `--processing debayered` (ali `display`), da vključite vse kamere ne glede na vrsto filtra, ali privzeto `all`, da v enem posnetku pridobite vse ustrezne ravni za vsako kamero.

### Odčitek DAQ, ki je daleč od osvetlitve

Vsaka vrednost `.daq`, ki jo zajem shrani, je odčitek DAQ, ki ustreza tej osvetlitvi, vrednost `.match.json` poleg nje pa beleži, kako daleč sta bili ti dve vrednosti narazen (`gap_ms`) ter ali je ta razlika znotraj tolerance (`in_tolerance`, 500 ms, razen če je v nastavitvah zajema določeno drugače `--daq-tolerance-ms`). Vrednost zunaj tega območja se še vedno uporabi, tako da se noben posnetek ne izgubi, in je označena: datoteka odbojnosti v formatu XMP vsebuje `DlsInTolerance=False` in `DlsMatchGapS`. `lattice capture`, `multi-capture` in `array-capture` jo prav tako natisnejo pod zadevno `.daq`, pri čemer se pojavi napaka toka DAQ'to pojasnjujemo:

**Kako se odčitek uskladi z osvetlitvijo.** Zajem čaka na prvi odčitek DAQ-ja po osvetlitvi, da lahko združi odčitke na obeh straneh tega odčitka (`method` `interpolated`). DAQ-E pošilja odčitek vsake 0,4 s na dnevni svetlobi in vsake 1–2 s v zaprtih prostorih, zato se čakanje prišteje k ukazu za približno eno in pol takšnih intervalov, ne pa k osvetlitvi. Na DAQ, ki je prenehal pošiljati, se ne čaka. Odčitek DAQ-E je povprečje svetlobe v intervalu od prejšnje vrednosti, zato je umeščena na sredino tega intervala: `gap_ms` se meri od osvetlitve do te sredine, `bracket_stamps_ns` pa podaja časovne oznake samih vrednosti. `array-capture --continuous` čaka, ne da bi upočasnil zajem: njegove surove datoteke se zapišejo takoj, njihove `.daq` pa takoj, ko prispe odčitek. Pri `--interval` vsak prehod čaka kot posamezen zajem.

```
  DAQ:   …/output/daq/daq_data_daq-e-54b774_….daq
         DAQ reading 114.9 s from the exposure (tolerance 500 ms): reflectance uses it, labelled DlsInTolerance=False
         DAQ stream fault: no frame accepted from daq-e-54b774.local in 25 s, after 1932 good frame(s). …
```

Če iz te odčitke ni bila zapisana nobena datoteka odbojnosti, se prva vrstica namesto tega konča z `saved anyway; its .match.json says in_tolerance: false`.

Zajem se še vedno konča z `0`: vse datoteke so bile zapisane in navajajo, kaj vsebujejo. DAQ, katerega tok se je ustavil, še naprej pošilja svojo zadnjo odčitano vrednost, zato jo preverite z `chloros-cli daq pool-list`, preden se zanesete na odbojnost tega zajema. Backend ponovno poveže združen DAQ-E, ki je prenehal pošiljati podatke (glej [DAQ-E, ki preneha pošiljati podatke](#a-daq-e-that-stops-delivering)). Odčitek zunaj tolerance natisne le svojo vrstico `DAQ:`, razen če gre za ohranjeni odčitek (spodaj).

**Odčitek, ohranjen po izpadu DAQ-a.** Kadar noben DAQ ne more zagotoviti odčitka za določeno osvetlitev (vezani DAQ ni na voljo ali noben združen DAQ ga nima, npr. po `daq pool-disconnect`), zajem ponovno uporabi zadnjo vrednost, ki jo je imel Chloros, in njegova vrednost `.match.json` opisuje to vrednost glede na to izpostavljenost: `method` `retained`, `gap_ms` iz te osvetlitve do vzorcev, iz katerih je bila izmerjena vrednost, `in_tolerance` iz te vrzeli ter `retained_from_epoch_ns`, osvetlitev, s katero je bila vrednost najprej usklajena. Posnetek, narejen dolgo po tem, ko je DAQ prenehal delovati, zato natisne zgornje vrstice, pod njimi še eno, ki se natisne, čeprav je odčitek še vedno znotraj tolerance:

```
         This is the DAQ's last reading: the DAQ is no longer reporting, so each capture reuses it until a new one arrives
```

Njegova vrednost `.daq` še vedno navaja DAQ, ki je izmeril vrednost, skupaj s kalibracijo in pokrovčkom, pod katerima je bila vrednost izmerjena, tako da ponovni uvoz ve, kateri DAQ, kalibracija in pokrovček so jo ustvarili.

### Datoteke, ki se ne morejo zapisati

Kamera, katere datoteke ni mogoče zapisati – zaradi polnega diska ali napake pri dovoljenjih –, ne prepreči shranjevanja datotek drugih kamer. Vsak ukaz zajema (`lattice capture`, `multi-capture`, `array-capture`; `project capture`, `burst`, `stream`; ter `capture`, `burst` in `stream` iz recepta) izpiše vsako datoteko, ki se ni uspela shraniti, v stderr, nato **se zaključi `1`**, čeprav so bile ostale datoteke shranjene:

```
  Failed: SN:214701292 [reflectance] OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

Vključene so vse datoteke, ki jih zajem dolguje, vključno z enosliko brez debayeringa `lattice capture --levels debayered` vključena. Branje `.daq`, katerega datoteka s podatki o izvoru `.match.json` ni mogla biti zapisana, je navedeno kot `[match]` (sam `.daq` je bil shranjen).

Na polnem disku se ponavljajoči posnetki (`array-capture --continuous` / `--interval`, `project burst` / `stream`, recept `burst` / `stream`) **se ustavi pri tem prehodu**, namesto da bi za preostanek izvajanja na isti pogon ničesar ne zapisoval. Vsaka druga napaka se poroča ob vsakem prehodu, ko se pojavi, izvajanje pa se nadaljuje do konca, nato pa se zaključi z `1`. `array-capture --continuous` (neprekinjeno zajemanje backenda) ne navaja nobenih datotek, vključno z neuspešnimi: te se upoštevajo v vrsticah napredka in povzetka (`N failed`, `N file(s) failed to write.`) ter zaključi delovanje z `1` — glej [Neprekinjeni način](#continuous-mode---continuous).

Snemalniki — `array-burst` in `array-record` — poročajo o svojih napakah. `array-burst` se samodejno ustavi, ko se pogon napolni ali tik pred tem. Oba nadaljujeta delovanje kljub morebitnim drugim neuspešnim zapisom. Ko kateri koli od njiju poroča o napaki, se ta izpiše za števci, ukaz **zapre `1`**:

```
  Recorder error: disk-full: OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

`Disk low:` namesto `Disk full:` pomeni, da se je zaporedje ustavilo, preden se je napolnil disk. Okvirji, posneti pred ustavitvijo, se ohranijo. `array-burst --build` preskoči izdelavo, ko zmanjka prostora, saj bi se videoposnetki shranili na isti disk. Najprej sprostite prostor, nato pa zaženite `array-build-video --burst-dir …`. Po kateri koli drugi napaki se izdelava še vedno izvaja.

Tudi `array-record`, ki ni zapisal nobenega okvirja, pomeni neuspešen posnetek (`Recorder error: no-frames: the combined view published no frame to record …`, izhod `1`) — glej [`array-record`](#array-record--combined-index-videogif-monitoring-grade).

---

## Načini zajemanja, snemalniki in ponovna obdelava brez povezave

Vsi delujejo na **trajnem nizu** (najprej zaženite `array-connect`). Odražajo ploščo za zajemanje v grafičnem vmesniku.

### Načini delovanja `array-capture`

`array-capture` je en sam ukaz s štirimi načini zaklopa ter nizom stikal za izvoz:

| Način | Oznaka | Delovanje |
| --- | --- | --- |
| **Enkratno** *(privzeto)* | (ni) | Ena sinhronizirana skupina zajemov, nato izhod. |
| **Neprekinjeno** | `--continuous` | Zaporedni zajemi, dokler `Ctrl+C`, `--count N` ne zajamejo `--duration S` ali polni disk. Izvaja lastno neprekinjeno zajemanje strežnika (način »Neprekinjeno« v grafičnem vmesniku) — glej [Način »Neprekinjeno«](#continuous-mode---continuous) spodaj. |
| **Interval** | `--interval S` | En prehod vsakih `S` sekund (merjeno od začetka vsakega prehoda), enake meje. |
| **Najhitrejši** | `--fastest` | Samo surovi podatki-samo + dodeljena vrednost DAQ + sestavljeni indeks; preskoči izračune svetilnosti/odbojnosti/prikaza, da se slika hitro prikaže. Vključuje `--processing raw --force-daq`. Shranjene `.daq` kasneje ponovno obdelajte v kalibrirane izdelke. |

Preklopniki za izvoz (kombinirajte s katerim koli načinom; vsi uporabljajo isti GUI/končni točki SDK):

| Oznaka | Učinek |
| --- | --- |
| `--array-id ID` | Matrika, iz katere se zajema (izpustite, če je priključena le ena). |
| `-o, --output DIR` | Izhodna mapa (privzeto `output`, v mapi, iz katere izvajate ukaz). |
| `--format {tiff,tif,png,jpg,jpeg}` | Oblika datoteke (privzeto `tiff`). PNG / JPEG sprejemata le `display` in indeksno upodabljanje; glej [Ravni izvoza zajema](#capture-export-levels-the-all-default). |
| `--processing LEVEL` | Enotna raven izvoza (`raw`, `debayered`, `radiance`, `reflectance`, `display`) ali `all` (privzeto). |
| `--levels L1,L2,…` | Izrecna podmnožica tipov izvoza (npr. `raw,radiance,reflectance`); **prevladuje nad `--processing`**. |
| `--aligned` / `--no-aligned` | **Privzeto: poravnano, tako kot zajem istega niza v aplikaciji Chloros**(njena nastavitev zajema**Poravnano** za vsak niz, vklopljena, razen če jo izklopite). Izvoz v ne-surovi obliki za vsakega člana (`debayered`, `display`, `radiance`, `reflectance`, render indeksa) je prilagojen v okvir glavne kamere in obrezan na skupno prekrivanje članov, tako da so datoteke pikselno usklajene in imajo enako velikost, obrez in oznake `Alignment*` kot v aplikaciji – manjše od senzorja (ali območja zanimanja – ROI) za vrednost odmikov članov; v okviru območja zanimanja (ROI) se transformacija ponovno osnuje na okno. `XMP-mapir:AlignmentJSON` navaja `"applied": true` in podaja `crop_roi`. Surova datoteka ostane nepreoblikovana v izvorni velikosti, vendar nosi transformacijo v metapodatkih (`"applied": false`). `--no-aligned` shrani vsako datoteko v izvorni velikosti, nepreoblikovano. `--aligned` je privzeta nastavitev. Ne glede na to, ali je popačenje uporabljeno ali ne, vsaka datoteka v nizu zabeleži transformacijo `Alignment*` svoje kamere, ko ima niz profil poravnave — razen kamere, ki se s profilom ne poravna (izpuščena ali prisiljena v identiteto zaradi neuspešnega reševanja), in referenčne kamere, kadar ni poravnana nobena druga kamera: te ne vsebujejo oznak `Alignment*`. Če niz nima profila, se posnetek shrani neporavnan, CLI pa izpiše razlog v vrstici `Alignment:`, tako kot v vseh drugih primerih, ko popravka ni mogoče uporabiti (člani z različnimi ločljivostmi, niz, ki ni sinhroniziran prek kabla, okvir, na katerega profila ni mogoče preslikati). Samodejno poravnavanje niza ustvari profil, ko se osvetlitev ustalijo; `align-calibrate` pa ga ustvari na zahtevo. |
| `--orient` | Obrnite **produkte**vsakega člana — `display`, `radiance`, `reflectance`, render `index/` in `composite/` — glede na orientacijo kamere po poravnalnem izkrivljanju: produkt, izkrivljen v okvir glavnega (privzeto) glede na**orientacijo izvirnika**, tako da ostanejo elementi usklajeni; neusklajen izdelek (`--no-aligned` ali posnetek, na katerega ni bilo mogoče uporabiti popravka) pa glede na orientacijo lastne kamere.**Privzeto: izklopljeno — vsaka datoteka elementa ostane v izvirni obliki**, kot je to vedno pri posnetkih v obliki niza. `raw` in `debayered` ostajata v izvirnem okviru v obeh primerih. Vsaka datoteka beleži orientacijo, ki jo nosijo njeni izdelki (glej [Orientacija kamere](#kako-izgleda-mapa-z-posnetki)); datoteka XMP `Alignment*` ostane transformacija izvirnega okvirja, uporabljena pred usmeritvijo, tako da `process` (deformacija, nato orientacija) reproducira izvirni izdelek. |
| `--index` / `--no-index` | Shrani / preskoči prekrivni sloj indeksa vegetacije za posamezno kamero, če je ta konfiguriran. Privzeto: ga prikaži. |
| `--no-daq` | Preskoči stranski datotek `.daq`, če nobena izbrana raven ne uporablja DAQ (zajem samo surovih podatkov). Privzeto se odčitek shrani, tako da se posnetke lahko naknadno ponovno obdelajo v odbojnost/indeks. `--force-daq` je še vedno sprejet (privzeto). |
| `--daq-tolerance-ms MS` | Toleranca ujemanja DAQ za te zajeme (privzeto 500 ms). |
| `--smart` | Pred sprožitvijo počakajte, da se AE ustalijo na vseh kamerah (glejte [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | Stiskanje pikslov v formatu TIFF. `none` (privzeto od različice 1.3.0; različica 1.2.0 je zapisala `deflate`) = nestisnjeni — piksli senzorja so natančno takšni, kot so bili prebrani, ~6,3 MB na posnetek v polni ločljivosti in najhitrejša trajna hitrost; `deflate` = brezizgubna stiskanja zlib L1 + vodoravni prediktor, ~4,1 MB na posnetek, počasnejša. Obe sta brezizgubni in se ob uvozu berejo enako. |

> **TIFF z enim zapisom + model s trajno hitrostjo.**Vsaka datoteka se zapiše**enkrat**: piksli, paket XMP, IFD0 Make/Model/CameraSerialNumber ter oznake EXIF `ExposureTime` / `SerialNumber` v enem prehodu, pri čemer se kasneje nič ne prepisuje. `--continuous`X zajema zaporedne sinhronizirane skupine – vsak zajem prevzame skupino po prejšnji, vključno s tisto, ki je bila objavljena, medtem ko se je ta zajem še shranjeval –, zato deluje s hitrostjo matrike. DEFLATE zadržuje GIL v Pythonu, zato se stisnjena zapisovanja**ne**izvajajo vzporedno v niti za zapisovanje po posameznih kamerah; za neprekinjeno zajemanje v polni ločljivosti s hitrostjo senzorja so potrebna nekomprimirana zapisovanja (privzeto)**in** disk razreda NVMe (~600 MB/s trajno za niz 9 kamer pri ~10 fps). Isti gumb je na voljo kot `compression` na `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (aligned is the default; drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --no-index -o out/

# Each member at its native size, unwarped
chloros-cli lattice array-capture --processing reflectance --no-aligned -o native/

# Continuous raw capture for 60 s, as fast as the array publishes
chloros-cli lattice array-capture --continuous --duration 60 --fastest -o flightline/
```

#### Neprestani način (`--continuous`)

`array-capture --continuous` (brez `--interval` ali `--smart`) predaja izvajanje **lastnemu neprekinjenemu zajemanju v ozadju** — seji, v kateri teče neprekinjeni način v grafičnem vmesniku. Backend sam shrani vsako sinhronizirano skupino, pri čemer potekata dva zajema hkrati in med skupinami ni zahtevkov iz CLI, tako da uspeva slediti hitremu nizu, kjer zahtevek na zajem zaostaja (izmerjeno 73–77 zajemov/s na nizu 9 kamer pri ROI 512 pikslov). Vsako zgoraj navedeno preklapljanje izvoza velja za vsak posnetek.

- **Meje:** `--count N` se ustavi po N posnetkih (en posnetek = ena sinhronizirana skupina: datoteke vseh članov); `--duration S` po S sekundah. Brez nobene od teh nastavitev teče do `Ctrl+C`, polnega diska, napake ali izklopa niza. `Ctrl+C` ustavi sejo strežnika, nato pa izpiše povzetek.
- **Napredek** vsake 2 s, pri čemer se dodata `, N failed` in `, N groups skipped`, če sta vrednosti različni od nič (preskočena skupina je tista, ki jo je niz objavil, a ni bila zajeta):

  ```
     12.0s  880 captures (73.3/s), 7920 files
  ```

- **Ni seznama po posameznih datotekah.** Izvajanje se konča z eno vrstico povzetka — zajemi, hitrost, datoteke in izhodna mapa — ki ji sledita `N file(s) failed to write.` / `N group(s) skipped.`, če vrednost ni nič:

  ```
  Done — 880 captures (73.3/s), 7920 files in /home/user/flightline.
  ```

- **Ustavi se** ob polnem disku z `Continuous capture stopped: the drive is full.` ter ob napaki ali izpadu matrice z `Continuous capture stopped: <reason>.` (oba na stderr).
- **Izhodni status:** `0`, če so bili posnetki shranjeni in so bile zapisane vse datoteke, vključno z `Ctrl+C`; `1`, če ni bilo mogoče zapisati katere koli datoteke, pri polnem disku, napaki, izklopu niza ali če ni bilo nobenega zajema.
- **Microsoft Defender:** kadar Defender porabi eno jedro procesorja ali več za skeniranje datotek, ki jih zapiše zajem, se med izvajanjem enkrat izpiše na stderr: `Note: Microsoft Defender is using N.N CPU cores scanning the capture files, CPU the capture needs. Re-run the Chloros installer with "Exclude the Chloros backend from Microsoft Defender scanning" checked.` Glejte [Sistemskih nastavitvah](#windows-exe).
- Hkrati je mogoče izvajati le en neprekinjen zajem na matrico; drugi se zavrne z napako `a continuous capture is already running for this array`. Prek HTTP-ja je seja `POST /api/camera/array/continuous/start` (telo `/api/camera/array/capture` skupaj z `max_captures` / `duration_s`), `GET /api/camera/array/continuous/status?array_id=ID` ter `POST /api/camera/array/continuous/stop`.

**Interval**(`--interval S`, z ali brez `--continuous`) in**`--continuous --smart`** ohranijo zanko na zahtevo: ena zahteva za zajem na prehod — v vsakem načrtovanem trenutku za Interval, po stabilizaciji AE za `--smart` — s seznami datotek vsakega prehoda, kot v načinu Single, vrstico `Done — N capture pass(es).` na koncu in `--count N`, ki šteje prehode.

### `array-record` — video/GIF s kombiniranim indeksom (za namene spremljanja)

Zapiše **pogled v živo s kombiniranim indeksom** matrike v datoteko `.avi` (in po želji v datoteko `.gif`). Ko je v grafičnem vmesniku odprt kombinirani pogled, zapiše ta pogled tak, kot je prikazan. Če ni odprt noben pogled, backend sam prikaže pogled: indeks, ki ga posredujete z `--expr`, ali privzeto NDVI matrike, ko je matrika poravnana. Privzeti NDVI uporablja rdeči in NIR pas, ki sta najbližja 660 nm in 850 nm, tako kot v živo prikaz. Pogled GUI, odprt med snemanjem, se snema, dokler ostane odprt. Vsake 2 s preveri napredek in se ustavi ob `--duration`, `Ctrl+C` ali ko se snemalnik samodejno zaključi.

Snemanje, ki ne zapiše nobenega okvirja, se ne uspe: snemalnik poroča o napaki `no-frames: …` (matrika se morda še vedno poravnava ali stabilizira svojo osvetlitev) in ukaz **se zaključi `1`**.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF (no GUI needed)
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# Record a chosen index instead of the default NDVI
chloros-cli lattice array-record --duration 30 \
  --expr "(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)" -o monitoring/
```

| Oznaka | Privzeto | Opis |
| --- | --- | --- |
| `--array-id ID` | samo matrika | Ciljna matrika (izpustite, če je priključena le ena). |
| `-o, --output DIR` | `output` | Izhodni imenik. Relativna pot se nahaja v mapi, iz katere izvajate ukaz. |
| `--fps F` | `10` | Hitrost snemanja. |
| `--duration S` | do Ctrl+C | Samodejni zaustavitev po `S` sekundah. |
| `--gif` | izklopljeno | Zapiši tudi animirani GIF. |
| `--gif-only` | izklopljeno | Shrani samo GIF (brez `.avi`). |
| `--expr EXPR` | NDVI matrike | Indeksni izraz za zapis, kadar ni odprt noben GUI-pogled, v imenih pasov matrike (npr. `'(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)'`; glej [imena pasov](#index--vegetation-maths)). Kratko ime barve (`NIR`, `Red`) deluje le, če ga ima en pas matrike. Odprt pogled GUI se namesto tega zapiše tak, kot je prikazan. |

### `array-burst` — surovi Bayerjev niz s hitrostjo snemanja več slik na sekundo (za analizo)

Neposredno bere sinhronizirani skupinski pomnilnik zanke zajemanja — **ni potrebna kalibracijska veriga, ni potrebno orodje exiftool, ni potreben pogled v živo** — zato deluje s polno hitrostjo zajemanja kamere. Zapiše surove okvirje + manifest za vsak okvir + en `.daq` za vsako ločeno branje DLS pod `<output>/bursts/<base>/`. Ponovno obdelajte v načinu brez povezave (naslednji ukaz) ali prenesite `--build`, da se to opravi takoj ob ustavitvi.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Oznaka | Privzeto | Opis |
| --- | --- | --- |
| `--array-id ID` | samo niz | Ciljni niz. |
| `-o, --output DIR` | `output` | Izhodni imenik (izhodni podatki se shranijo v `<DIR>/bursts/<base>/`). |
| `--duration S` | do Ctrl+C | Samodejni zaustavitev po `S` sekundah. |
| `--max-frames N` | neomejeno | Samodejniustavitev po `N` surovih okvirjev. |
| `--orient` | izklopljeno | Izdelki, ki jih zgradba ustvari iz te serije, se obrnejo glede na orientacijo kamere – izdelki posamezne kamere glede na lastno kamero, združen indeks pa glede na glavno kamero — tako kot to počne `array-capture --orient`. Surovi okvirji ostanejo v izvirni obliki; `manifest.json` serije posnetkov zabeleži usmeritev in nastavitev vsakega posameznega posnetka. |
| `--build` | izklopljeno | Po zaustavitvi takoj ponovno obdelajte serijo posnetkov (enako kot `array-build-video`). |
| `--products …` | `combined:index` | Pri `--build`: kateri videoposnetek(-i) naj se ustvari(-jo) (glej spodaj). |
| `--fps F` | `10` | Pri `--build`: število sličic na sekundo (fps) izhodnega videa. |
| `--save-tiffs` | izklopljeno | Z `--build`: shrani tudi kalibrirane TIFF-datoteke za vsak posamezni okvir (vsaka s svojo [masko omejitve razmešavanja](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on), kadar je piksel omejen; rezultat gradnje jih navaja kot `clamp_masks`). |
| `--gif` | izklopljeno | Z `--build`: zapiše tudi animirane GIF-datoteke. |

### `array-build-video` — offline ponovna obdelava shranjene serije

Vsak surovi okvir časovno uskladi z najbližjo shranjeno vrednostjo `.daq` in ga preusmeri skozi **isto verigo sevanja / odbojnosti / indeksa kot uvozni proces**, pri čemer ustvari enega ali več videoposnetkov.

`--products` je seznam elementov `kind:level`, ločenih z vejicami, kjer velja: `kind` ∈ `per_cam` | `combined` ter `level` ∈ `radiance` | `reflectance` | `index`. Če je naveden samo `level` (brez `kind:`), je privzeta vrednost `per_cam`. Privzeta vrednost je `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Oznaka | Privzeto | Opis |
| --- | --- | --- |
| `--burst-dir DIR` | (obvezno) | Pot do mape za serijske posnetke (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | Seznam `kind:level`, kot zgoraj. |
| `--fps F` | `10` | Število sličic na sekundo (fps) izhodnega videa. |
| `--save-tiffs` | izklopljeno | Poleg videoposnetkov shranite tudi kalibrirane TIFF-datoteke za vsako sličico posebej. |
| `--gif` | izklopljeno | Shrani tudi animirane GIF-datoteke. |

**Videoposnetek ne vsebuje maske omejitve** (en tok slik, 8-bitni). Namesto tega rezultat gradnje vsebuje `clamp_fractions`: za vsak video izdelek, delež pik vsakega posameznega okvirja, ki ga je razmešavanje omejilo v pasu, ki ga vsebuje njegov R/G/B kanal — `Chloros:Band{R,G,B}UnmixClampFraction`, s katerim bi bil označen produkt sevanja/odbojnosti tega okvirja (za združen indeks vsake posamezne kamere). Izpis izpisa najslabši okvir za vsak izdelek v vrstici `Clamp:`. `--save-tiffs` doda maske na piksel za datoteke TIFF, v `clamp_masks/<level>/` poleg mape `<level>/` za datoteke TIFF.**Če se gradnja prekine, ne ostane noben poškodovan video.** Vsak video se zapiše kot `<name>.partial.avi` in dobi končno ime šele, ko je dokončan, zato starejši video z istim imenom ostane, dokler njegov nadomestek ni celoten. Izvedite gradnjo še enkrat, da nadomestite `.partial.avi`, ki je ostal za sabo.

Izvedba izpiše vsak posnetek, ki ga je zapisala, nato pa še vsak izdelek `Skipped:` (zahtevana kamera/raven – ali kombinirani indeks –, ki ni ustvarila posnetka, skupaj z razlogom – npr. ni kalibracijskega paketa, ni DLS-odčitka za odbojnost ali ni para rdečega/NIR-pasov za kombinirani indeks) ter vse `Note:`. RGB-kamera, za katero je zahtevano sevanje ali odbojnost, namesto tega dobi svoj prikazni video, z uravnoteženo belino, če je na voljo kalibracijski paket kamere; kamera, ki je prikazala sliko brez tega, je navedena v `Note:`. Funkcija **vrne vrednost, ki ni nič**, če je bil zahtevani izdelek preskočen ali ni bil ustvarjen v skladu z zahtevo (konfigurirani izraz indeksa se ne izračuna v načinu brez povezave); videoposnetki, ki so bili ustvarjeni, se še vedno zapišejo in navedejo.

> **Izberite pravi snemalnik.** `array-record` je *primeren za spremljanje* — snema renderirano kompozitno sliko v živo: združen pogled grafičnega vmesnika ali, če grafični vmesnik ni odprt, lastni pogled NDVI (ali `--expr`). `array-burst` → `array-build-video` je *za analizo* — shranjuje surove podatke senzorja s polno hitrostjo in naknadno rekonstruira kalibrirane videoposnetke sevanja/odbojnosti/indeksa, pri čemer ni potreben pogled v živo.

### Mono (M3M) enopasovne kamere

Serija **M3M**je mono različica Bayerjeve serije**M3C**: en ozkopasovni interferenčni filter na kamero (`M3M-<lens>-F<wavelength>`, npr. `M3M-L87-F685`), zato senzor zagotavlja**en sam pas v sivih odtenkih** brez Bayerjevega mozaika. Ni potrebe po demosaiciranju, ni medkanalnega prekrivanja, ki bi ga bilo treba ločiti, in ni treba nastavljati beline – celoten barvni proces za prikaz v RGB preprosto ne velja.

Kaj to pomeni v CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**zaznavajo črno-belo kamero in**preskočijo z enovrstičnim sporočilom**, namesto da bi pošiljali nesmiselne nastavitve. V isti seji še vedno delujejo normalno z RGB/Bayer M3C kamero.
- **Bayerjev format pikslov pomeni njegovo globino.** M3M ponuja le mono formate, zato Bayerjev format iz `--pixel-format` ali `--preset` nastavi mono format iste globine: `BayerRG8` in `--preset high_speed` dajejo `Mono8`; `BayerRG12`, `high_quality` in `triggered` dajejo `Mono12`; zgoščeno ime ostane zgoščeno (`BayerRG12p` daje `Mono12p`). Datoteke in ime `lattice format` označujejo format Mono. Niz, ki združuje kamere M3C in M3M, uporablja en format na na enak način: Bayer na vsaki M3C, njena globina Mono na vsaki M3M. Ime je njena bitna globina: enako velja tudi v obratni smeri, tako da M3C nikoli ni v formatu Mono (`--pixel-format Mono12` je `BayerRG12` na M3C, ne glede na to, katera kamera je glavna v nizu), `lattice format --set` pa se preslika na enak način in natisne format, ki je bil nastavljen.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** še vedno delujejo — sevanje in odbojnost sta *na pas* radiometrični karti in sta za en pas popolnoma dobro definirani. Mono okvirji vsebujejo **identitetno** matriko odziva senzorja (brez 3×3 razmešavanja), zato ravnina ostane v kalibracijskih izračunih nespremenjena.
- **Ena sama monokromatska kamera ne more izračunati indeksa vegetacije.**NDVI, NDRE in podobni indeksi potrebujejo vsaj dva pasova (npr. Red + NIR). Da bi iz monokromatske opreme pridobili indeks, usmerite**več** M3M kamer na različne valovne dolžine, jih poravnajte v en večpasovni niz in izračunajte indeks *iz tega*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Mono_660 --channel nir=Mono_850 \
  --save-multiband -o output/
```

`index --live` samodejno odpre kamere (kot [začasna seja](#multi-camera-transient-sessions)), zato najprej ne priključujte ničesar: `--serials`, sicer bodo kamere, omogočene v profilu (glej [Indeks / Matematika vegetacije](#index--vegetation-maths)). Medtem ko Chloros drži katero koli kamero (GUI, `cam-connect`, `array-connect`) se zavrne z HTTP-odzivom 409 `CAMERA_HELD` in se zapre `1`; sprostite jih z `chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`.

`--channel` simboli morajo **natančno** ustrezati imenom kanalov v prednastavitvi (upošteva se velike in male črke; NDVI so zapisani z malimi črkami `red`, `nir` — glej `--list-presets`), imena pasov pa morajo označevati pasove v poravnanem nizu (v načinu brez povezave so sprejemljivi tudi indeksi pasov, ki se začnejo z 0, npr. `--channel red=0 --channel nir=1`).

Razlikovalni znak v celotnem nizu je token `M3M` v nizu modela (nikoli se ne pojavi v nizu `M3C`), ki se v grafičnem vmesniku (GUI) in SDK-ju prikaže kot `is_mono`.

---

## Nastavitev in optimizacija omrežne kartice gostitelja (nizi LATTICE)

Kamere LATTICE prenašajo GVSP prek ethernetnega adapterja gostitelja, zato sta pri nizih z več kamerami **gonilnik**in**velikost sprejemnega obroča** prav tako pomembna kot hitrost povezave. Napačne nastavitve se kažejo kot `FRAMES WILL DROP` / `Reduce ROI to enable` v oknu »Nastavitve niza« (ter v `lattice network-analysis` / SDK-ju `analyze_array_network()`), tudi če so same kamere v dobrem stanju.

### USB 10GbE adapterji — Realtek RTL8157 („Realtek USB 10GbE Family Controller“)

Spodnje nastavitve so lastnosti gonilnika za Windows. V sistemu Linux je tudi za USB-adapter določena omejitev (400 MB/s na x86_64, 200 MB/s na NVIDIA Jetson) (glej [Model fps in burst za matrike](#array-fps--burst-model)), pri čemer so za vsak adapter pomembni sprejemni pomnilniki vtičnic jedra — glej [Gostitelji Linux](#linux-hosts--socket-receive-buffers).

| Element | Zahtevana vrednost | Zakaj je pomembna |
| --- | --- | --- |
| **Različica gonilnika**|**≥ v10.67 (januar 2026)**, INF `rtump64x64sta.inf` | Stari gonilnik iz leta**2016**(v10.65, `rtump64x64.inf`) nepravilno obravnava izklop napajanja in napake s**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**ob izklopu/ponovnem zagonu/prehodu v stanje mirovanja. Prehod se zatakne (časovna omejitev ~5 min), uporabnik prisilno izklopi napravo, ponavljajoče se nepravilno izklapljanje pa**poškoduje repozitorij WMI**(PowerShell/orodja začnejo odpovedovati z napako `Invalid class`) in**zablokira USB-sklad** ob naslednjem zagonu (mrežna kartica se ne vklopi; USB-pogoni prenehajo s prepoznavanjem). Posodobite s spletne strani realtek.com (ali pri proizvajalcu ključa), preden se zanesete na čiste ponovne zagone. |
| **Prejemni pomnilniki**— ključna beseda `ReceiveBufferLen` |**256**(največja vrednost gonilnika) | Obroč za sprejem (RX) mrežne kartice. Privzeta vrednost gonilnika**32**pušča na voljo le ~0,26 MB uporabnega obroča — kar je premalo za serijo posnetkov z več kamerami — zato nadzorna plošča poroča o napaki `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` in blokira povezave. Pri vrednosti**256**je obroč velik (**~13,5 MB, izmerjeno na laboratorijskem gostitelju 10GbE**), kar daje sprejemnemu cevovodu resnično rezervo za serije posnetkov GVSP z več kamerami. (Ali se določena konfiguracija dejansko *poveže*, se odloči na podlagi dveh preverjanj — **preverjanja prepustnosti z upoštevanjem izpraznitve**preverjanje sprejemljivosti in preverjanje**agregatne prekomerne zasedenosti** — ne gre za surovo primerjavo med serijo in obročem; glej [Model fps in serijskih prenosov v polju](#array-fps--burst-model).) |
| **Prejemni URB-ji**— ključna beseda `PendingReceives` |**64** (maks.) | Bloki zahtevkov USB v prenosu; povečajte skupaj s sprejemnimi pomnilniki za absorpcijo izbruhov. |
| **Jumbo Frame** — ključna beseda `*JumboPacket` | **9014** | Potreben za GVSP-pakete velikosti 9000 bajtov (6× manj paketov/okvirja kot pri 1500). |

> ⚠️ **Posodobitev gonilnika mrežne kartice te napredne lastnosti ponastavi na privzete vrednosti.**Po posodobitvi ali zamenjavi gonilnika kartice**ponovno uporabite** `ReceiveBufferLen=256` in `PendingReceives=64`, sicer se bo plošča ponovno blokirala, čeprav se »v strojni opremi ni nič spremenilo«. To je glavni vzrok, da se sistem, ki je prej deloval, nenadoma ne želi več povezati.

Uporabite v **privilegiranem** PowerShellu (vstavite ime svojega adapterja, npr. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` zajema USB 10GbE adapterje.** Zdaj zazna tip adapterja in nastavi pravilno ključno besedo za sprejemni obroč: `*ReceiveBuffers`→2048 za PCIe mrežne kartice (Intel I219, itd.) ali `ReceiveBufferLen`→256 + `PendingReceives`→64 za Realtek **USB** 10GbE krmilnik (ki ne razkriva `*ReceiveBuffers`). Ciljne vrednosti so omejene na največjo vrednost, ki jo poroča posamezni gonilnik (`NumericParameterMaxValue`), zato se nikoli ne zapiše vrednost zunaj območja. Zagonite ga iz terminala z **povečanimi pravicami**; kot pri vseh nastavitvah, ki temeljijo na registru, sprememba začne veljati šele po ponovnem zagonu adapterja ali ponovnem zagonu računalnika. Zgornji ročni ukazi `Set-NetAdapterAdvancedProperty` ostajajo dobra alternativa — veljajo takoj (ponovna vezava adapterja) brez ponovnega zagona.

### Linux gostitelji — sprejemni pomnilniki vtičnic

V sistemu Linux, vključno z NVIDIA Jetson, so sprejemni pomnilniki vtičnic jedra enako pomembni kot sprejemni obroč mrežne kartice. Pri privzetih nastavitvah jedra (`net.core.rmem_max` / `net.core.rmem_default` = 212.992 bajtov) serija GVSP-paketov iz kamere prepolni tok vtičnice, še preden jih Chloros prebere. Paketi se v jedru izgubijo, medtem ko mrežna kartica sama ne poroča o izgubi, zaradi česar niz kamer izgubi skupine okvirjev in zmanjša svojo hitrost snemanja. Pri merjenju na nizu s 4 kamerami, priključenim prek USB 10 GbE adapterja, je povečanje obeh vrednosti na 32 MB po vzpostavitvi povezave povečalo hitrost niza s 4,96 na 7,52 sličic na sekundo, neprekinjeno zajemanje surovih podatkov pa s 4,7 na 7,6 zajemov na sekundo, pri čemer v jedru ni bilo izgubljenih paketov.

- **Datoteka .deb jih nastavi.** Namestitev paketa zapiše vrednosti `/etc/sysctl.d/60-chloros-gige.conf` (obe na `33554432`) in jih takoj uporabi. Vrednost, ki je že nastavljena na višjo vrednost, ostane nespremenjena.
- **`lattice network` jih preverja.** Če je katera koli od vrednosti nižja, se v rubriki *Ugotovljene težave* prikaže `socket receive buffers too small for GigE Vision (…)`, v dnevniku strežnika pa se ob prvem priklopu kamere ali niza prikaže enako opozorilo.
- **Če jih želite ročno povečati** (do naslednjega ponovnega zagona; datoteka paketa jih uporabi ob vsakem zagonu):

  ```bash
  sudo sysctl -w net.core.rmem_max=33554432 net.core.rmem_default=33554432
  ```

  `lattice network --fix` jih prav tako poveča in ohrani, vendar le, če backend teče kot root. Backend, ki teče kot vaš uporabnik (tisti, ki se zažene z `chloros-backend` ali [`chloros-backend@<user>`](#running-the-backend-as-a-service-headless)) ne more spreminjati nastavitev jedra in namesto tega izpiše vrstico `sudo sysctl`.

**Avtomatska optimizacija adapterjev zadeva le priključene adapterje.** Ob prvem priključku kamere backend, ki teče kot root, optimizira žične adapterje, katerih obroč za sprejem je majhen. Zdaj prilagaja le adapterje s povezavo, zato odklopljeno vgrajeno vrata ohranijo svojo MTU, sprejemnih pomnilnikov pa ne spreminja. `lattice network` ne navaja več MTU odklopljenega adapterjakot težavo. `lattice network`, `--fix`, `lattice probe` in nastavitev adapterjev upoštevajo le žične ethernetne vtičnice, ne pa CAN-vmesnika Jetsona (`can0`) ali povezav v načinu USB-naprave (`usb0`, `usb1`), ki jih ponuja gostiteljski računalnik; USB-Ethernetni adapter za kamere se še vedno upošteva.

### Osnove omrežja (vse povezave LATTICE)

- **Naslavljanje:** lokalno za povezavo `169.254.0.0/16` (GigE Vision LLA). Gostiteljski računalnik uporablja statični naslov `169.254.x.x/16`; kamere in DAQ-E si sami dodelijo naslove v istem obsegu. DHCP/prehod ni potreben.
- **Velikost paketa:**prednostno jumbo (9000), vendar naj jo samodejno preizkušanje poišče – prek GVSP-preizkusa prezre omejitev ICMP kamere na 1500 bajtov, tako da se ustali na jumbo, kjerkoli jo kabel dejansko prenese.**Vsaka kamera se izmeri enkrat na sejo strežnika in ponovno, ko se spremeni njena povezava (drugi adapter, druga hitrost povezave ali prekinjena povezava, kot se zgodi pri zamenjavi stikala); pri ponovni povezavi se uporabi že pridobljen rezultat. Po spremembi za stikalom, ki je ostalo vklopljeno, ponovno zaženite strežnik, da se izmeri ponovno.**Prva povezava lahko pri tem preverjanju občasno traja nekaj minut, kar je v dnevniku tudi navedeno. Uporabite fiksiranje s kodo `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` le, če veste več kot sonda, in raje uporabite fiksiranje po posameznih ukazih kot trajno: fiksiranje preskoči sondo, tako da, če pot dejansko ne more prenesti 9000,**vsak** zajem poteče v časovno omejitev z `SC_ERR_TIMEOUT -1011` (glej [Okoljske spremenljivke](#environment-variables)).
- **Obseg RX-obroča se prilagaja vrednosti `ReceiveBufferLen`:**pri privzeti vrednosti `32` je uporaben obroč ~0,26 MB (premajhen za kateri koli večkamerni izbruh); pri največji vrednosti `256` je velik (~13,5 MB, izmerjeno na laboratorijskem gostitelju 10GbE), kar zagotavlja resnično rezervo. Ali se konfiguracija poveže, se nato odloči na podlagi preverjanja sprejemljivosti, ki upošteva izčrpavanje zmogljivosti,**in** preverjanja agregatne prekomerne zasedenosti spodaj — ne na podlagi surovega primerjanja serije s obročem.

### Model fps in serijskega snemanja za Array

Kako brati ploščo »Array Settings« (in `lattice analyze-array` / `analyze_array_network` v SDK):

- **Burst se sešteva po posamezni kameri v dejanskem formatu pikslov vsake kamere.**Mono**M3M**kamere prenašajo**Mono12 (2 B/px)**ali, z `--pixel-format`, mono format svoje globine (`BayerRG8` je `Mono8`, 1 B/px);**M3C**kamere z Bayerjevim filtrom prenašajo 8- ali 12-bitne podatke (firmware M3C tiho oddaja BayerRG12, tudi če je zahtevan BayerRG8). Torej je slika v polni ločljivosti s 4 kamerami**~12,6 MB, če so vse 8-bitne, vendar ~25 MB s tremi 12-bitnimi mono kamerami**. Projekcija določi format vsake kamere na podlagi njenega modela (predpomnilnik identitet), tako da serija posnetkov ustreza dejanskemu prenosu po kablu – ne gre za enotno predpostavko BayerRG8.
- **USB-ethernetni adapter je omejen na 200 MB/s ne glede na navedbe na tipski tablici.** Tabela učinkovitosti, ki pretvori hitrost povezave v trajno vrednost, izhaja iz standarda PCIe; USB-mrežna kartica oglašuje svojo *Ethernet* hitrost povezave, vendar je omejena z USB-vodilom in svojim gonilnikom. USB-ključ 10GbE je dosegal ~1063 MB/s »trajno« – vrednost, ki ni bila nikoli preverjena –, pri čemer je posledično napačno uravnavanje poškodovalo 6–18 % okvirjev, medtem ko je še vedno poročalo o ustrezni ciljni hitrosti sličic na sekundo (fps). Mrežne kartice, priključene prek USB-ja, so zdaj omejene na absolutno vrednost: **200 MB/s** v sistemu Windows in na NVIDIA Jetson, **400 MB/s** v sistemu Linux x86_64 (v sistemu Linux je za adapter na vratih USB 2.0 predvidena hitrost **30 MB/s**, v navodilih pa je navedeno, da ga je treba priključiti na vrata USB 3) (omejitev je v USB-vodilu in gonilniku gostitelja, zato se ne prilagaja navedeni zmogljivosti; USB 1 GbE-adapter doseže ~80 MB/s in to omejitev ne vpliva nanj). V sistemu Linux se adapter prepozna prek sysfs (z nadomestno oznako `ethtool -i` za `bus-info`). Vsaka vrednost je izmerjena: v sistemu Windows je isti niz štirih kamer poškodoval 6–10 % slik nad 200 MB/s; v sistemu Linux x86_64 je isti adapter deloval brez napak pri vseh mejnih vrednostih do 480 MB/s (dosegli smo 273 MB/s, kar je omejitev samih kamer). Na NVIDIA Jetson je isti adapter deloval brez napak le pri 200 MB/s, nad to hitrostjo pa je občasno izgubljal okvirje. `wire_ceiling_source` v zapisu zmogljivosti to navaja z besedami, `nic_is_usb` pa to označi. V obeh primerih lahko to prepisate z `--wire-ceiling-mbps`.
- **Dostop upošteva porabo, ne pa razmerja med celotnim izbruhom in obročem.** Hkratni burst mora ustrezati le *prehodnemu zaostanku* = `max(0, Σ per-cam arrival − host drain) × emit_window`, ne pa celotnemu burstu. V strukturi s hitrim gostiteljem in počasnimi kamerami (**PCIe**10G gostitelj + 4× 1 GbE kamere: prihod ≈ 320 MB/s, odvod ≈ 1063 MB/s) gostitelj prazni hitreje, kot se kamere polnijo, zaostanek ≈ 0, zato**dovoljuje**oddajanje v polni ločljivosti, čeprav 25 MB izbruh presega 13,5 MB obroč. Če iste štiri kamere priključimo na**USB** 10GbE adapter v sistemu Windows, je izhodna hitrost 200 MB/s, ne 1063 — prihodni tok jo prehiti, izguba pa se kaže kot poškodovani okvirji in ne kot nižja hitrost okvirjev. Na gostitelju z 1 GbE spodnja meja DLThr kamer 31,25 MB/s povzroči, da prihod prehiti odvod → pravilno **blokira** (za *to* vrsto blokade zmanjšajte ROI ali uporabite združevanje ≥ 2). Dostop je eden od **dveh** povezovalnih vrat – drugo je spodaj navedeni pregled agregatne prekomerne naročnine.
- **Predvideni fps je cikel najpočasnejše kamere.**Pri sočasnem oddajanju vsaka kamera pošlje svoj okvir istočasno, zato niz deluje v ciklu, ki je daljši od časa branja senzorja najpočasnejšega člana in časa oddajanja (bajtov okvira ÷ njegova hitrost oddajanja:**dostopna povezava** kamere, 1 GbE ≈ 80 MB/s, ali njen delež največje pasovne širine gostitelja, če je ta manjša). Vsaka osvetlitev se prekriva z odčitavanjem prejšnjega okvirja, zato osvetlitev ne vstopi v cikel: avtomatska osvetlitev zapolni časovno obdobje, ki ga določa hitrost, nato pa poveča ojačanje, pri čemer temna scena ne znižuje več hitrosti (v različici 1.2.0 je to še veljalo). Le ročna osvetlitev, daljša od tega časovnega obdobja, upočasni sprožilec (glej [Osvetlitev na članu niza](#exposure-on-an-array-member)). Dodajanje kamer ne upočasni matrike, dokler se njihovi deleži žice ne zmanjšajo. Pri serijskem oddajanju (`slip-emit-and-capture`) se namesto tega seštevajo časi oddajanja kamer, ekspozicija vsakega člana pa se upošteva pri časovnem zamiku. Merjeno pri polnem senzorju, 12-bitno, na nizu 9 kamer z gostiteljem PCIe 10 GbE: 10,7 fps ali 14,0 fps s [zgoščenim prenosom](#packed-12-bit-transport---packed).
- **Po neprekinjenem zajemanju projekcija navaja, kakšno hitrost je ohranila.** Zajemanje, ki traja 30 s ali več pri avtomatski hitrosti, zabeleži hitrost, na kateri se je ustalilo pri tej velikosti niza na tem računalniku; kratki poskusi krmilnika hitrosti z višjo hitrostjo se izpustijo. Od takrat naprej vrstica `array-connect` in plošča »Nastavitve matrike« navajata to hitrost, kadar je nižja od hitrosti modela, skupaj z razlogom za omejitev: `clock-bound` (matrika je delovala s hitrostjo, na kateri se je ustalila, zanka zajemanja je čakala na sprožilni tak) `loop-bound` (zanka zajemanja ni imela več časa), `capture-bound` (zajemanje je zaostajalo in preskočilo skupine) ali `measured` (ni navedeno, kateri). Podatek modela je v `wire_fps_max`. Niz še vedno dosega hitrost, ki jo omogoča omrežje.
- **Niz deluje v vsakem omrežju – odločilna je hitrost snemanja.**Vsaka kamera dobi enak delež**zgornje meje povezave, varne pred kolizijami**(`sustained × sim_emit_factor`), usklajene z GevSCPD, in vse še vedno posnamejo ob istem sprožilcu. Na počasnem omrežju je delež majhen in s tem pada tudi hitrost snemanja: 8 kamer s polno ločljivostjo in 12-bitno globino barv na 1 GbE brez jumbo okvirjev dobi vsaka 52 / 8 = 6,5 MB/s, kar je približno**1 fps**(približno 1,3 sličice na sekundo v paketu), medtem ko jih 10G gostitelj poganja s hitrostjo 11–14. Če kamer ni mogoče usklajevati tako počasi – to preverite z branjem obsega GevSCPD vsake kamere –, se niz preklopi na**serijsko**oddajanje (`slip-emit-and-capture`): kamere se po vrsti izmenjujejo na povezavi s polno hitrostjo povezave, sprožene ena za drugo, tako da vsak okvir še vedno prispe, vendar elementi**niso izpostavljeni hkrati**. `analyze-array` in izhodni podatki ukaza connect poimenujejo ta način; nastavite `--force-tier sim-capture-sim-emit` na »refuse«, da zavrnete preklop na rezervni način. Velikost okvirja ne vpliva na to delitev, zato**združevanje slik in manjše območje zanimanja (ROI) povečata hitrost snemanja, vendar ne spremenita tega, ali se niz prilega**. Le vodnik, ki ne more prenesti niza niti serijsko, je**preobremenjen**— npr. veliko več kamer, kot jih lahko en vodnik obvladuje, ali pa po njem pretaka veliko drugih kamer — in tam je simptom izguba paketov GVSP, ne nižja hitrost: `analyze-array` nastavi vrednosti dosegljivih sličic na sekundo na nič in izpiše `** OVER-SUBSCRIBED**`, povezava se vzpostavi z glasnim opozorilom, HTTP-povezava pa z `pin_resolution: true` (strogi način) **zavrne**. Rešitve so manj kamer, uporaba jumbo okvirjev od začetka do konca ali hitrejša mrežna kartica. `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` zmanjša zavrnitev na glasno opozorilo za testiranje – glej [Okoljske spremenljivke](#environment-variables).

### Stanje polja – kateri podsistem izgublja okvirje

`GET /api/camera/array/<array_id>/capability` povezanega polja prenaša blok v realnem času
`health`, ki se ponovno ocenjuje v tekočem **10-sekundnem** oknu. Izgubo okvirjev razčleni
na dva vzroka, ki zahtevata nasprotna popravila, namesto da bi poročal o eni »nepopolni«
stopnji, ki ne navaja nobenega od njiju:

| Polje | Kaj pomeni | Kateri podsistem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (po serijski številki) | Okvir **je prispel, a je bil strukturno poškodovan**— izguba paketov GVSP. |**Omrežje**: zmogljivost povezave, hitrost prenosa, obroč sprejemnika mrežne kartice, MTU |
| `never_arrived_rate_pct` (po serijski številki) | Okvir **sploh ni prišel**— kamera ni sprožila ali pa iz nje ni prišlo nič. |**Sprožilec / sinhronizacija**: kabel M8, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Najslabša stopnja kamere za vsakega. | — |
| `per_cam_rate_pct` | Skupna stopnja nepopolnosti na kamero (oba vzroka skupaj). | — |
| `stable_for_seconds` | Kako dolgo je vsaka kamera ostala pod 0,01 %. | — |

Nad 5 % backend zabeleži vrstico `[array-health <id>] WARN`, v kateri navede razlog za prekoračitev — ob
prvem prekoračitvi, ob spremembi stopnje resnosti, enkrat na minuto, dokler stanje traja, in enkrat, ko
se stanje popravi. Pokvarjena polovica zapiše `[gvsp-corrupt <SN>]` v dnevniško datoteko strežnika ob
prvem zadetku na kamero in vzrok, nato pa vsakih 60 s povzetek; namesto tega se v dnevniškem oknu aplikacije prikaže vrstica
namesto tega vrstico `[array-health <id>]`. Vsaka ocena se še vedno zapiše v dnevniško datoteko strežnika;
števci se povečajo ob vsakem pomnilniku, ne glede na to, kaj se izpiše.

Isti zapis poroča o številki, od katere je odvisna celotna dodelitev:

| Polje | Kaj pomeni |
| --- | --- |
| `wire_ceiling_mbps` | Trenutni trajni proračun gostitelja za prenos podatkov, MB/s. |
| `wire_ceiling_source` | Izvor te številke v besedah — npr. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` (na Linux u x86_64, `USB-capped 400 MB/s (…; sysfs: bus=usb (usb2/2-1/2-1:1.0))`) ali `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, če ga je določil `--wire-ceiling-mbps` (ali polje **Wire Budget** v grafičnem vmesniku). |
| `nic_is_usb` | `true` za USB-ethernetni adapter — glej zgornjo omejitev za USB. |

**Razlaga:** vrednost `gvsp_corrupt_rate_pct`, ki ni enaka nič, pri vrednosti `never_arrived_rate_pct`, ki je enaka 0
pomeni, da sta sprožanje in sinhronizacija kabla brezhibna, 100 % izgube pa je na omrežni
poti — znižajte vrednost `--wire-ceiling-mbps` in se ponovno povežite. Nasprotni vzorec kaže na
sinhronizacijski kabel ali sprožilno linijo.

> **`--target-fps` ni ključni dejavnik za poškodovane okvirje.** Hitrost GevSCPD se zapiše
> enkrat ob povezavi, zato znižanje hitrosti sprožilca spremeni delovni cikel, ne pa
> hitrosti sočasnega oddajanja. Izmerjeno 5-kratno zmanjšanje zahtev ni prineslo izboljšanja;
> znižanje zgornje meje povezave s 240 na 200 MB/s je pri isti napravi zmanjšalo delež
> poškodovanih okvirjev s 10,4 % na 0,00 %.

> **Delujoči niz se ne more samodejno ponovno načrtovati na firmwareu LATTICE (M3C / M3M).** Sam
> tega ne more popraviti; prekinite povezavo in jo ponovno vzpostavite, da bo izbirnik časa povezave lahko
> ponovno načrtoval z novo zgornjo mejo.

### Simptom → rešitev

| Simptom (Nastavitve polja / povezava / `analyze_array_network`) | Vzrok | Rešitev |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` se ponastavi na 32 (običajno po posodobitvi gonilnika) | Nastavite `ReceiveBufferLen`→256, `PendingReceives`→64; ponovno odprite nadzorno ploščo (ponovno zaženite backend, če je v predpomnilniku shranil staro velikost obroča) |
| Zastoj pri ponovnem zagonu/izklopu; kasneje napake WMI `Invalid class`, mrežna kartica se ne vklopi, manjkajo USB-ključi | Stari gonilnik Realtek USB 10GbE iz leta 2016 → BSOD `0x9F` → prisilno izklapljanje | Posodobite gonilnik adapterja na različico ≥ v10.67 (2026), nato ponovno uporabite zgornje nastavitve sprejemnega obroča |
| Array se poveže pri polni zmogljivosti senzorja, vendar z nizko hitrostjo slik | Način »Auto« ohranja polno zmogljivost senzorja in dopušča, da hitrost pade na raven, ki jo omogoča povezava | Izberite eno od hitrejših možnosti s seznama izhodov povezave (`--binning 2` / `4` ali manjši `--width`/`--height`), omogočite jumbo okvirje ali nadgradite povezavo |
| Array poroča o ustrezni ciljni hitrosti sličic na sekundo (fps), vendar prenaša le delček te vrednosti; `health.gvsp_corrupt_rate_pct` ni enak nič, `never_arrived_rate_pct` 0 | Izračunani proračun pasovne širine gostitelja je višji od dejanskega (tipično pri USB-Ethernetnem adapterju, ozkem PCIe-pasu ali skupni strukturi) | Ponovno se povežite z nižjo vrednostjo `--wire-ceiling-mbps` in ponovno preverite blok delovanja. **Ni** `--target-fps` — hitrost GevSCPD je ob vzpostavitvi povezave fiksna |
| Linux: niz izgubi skupine okvirjev in zmanjša hitrost prenosa okvirjev, medtem ko mrežna kartica ne poroča o izgubah; `lattice network` navaja `socket receive buffers too small` | Buferji za sprejem v jedru so nastavljeni na privzeto vrednost, zato se paketi kamere izgubijo v jedru | Povečajte `net.core.rmem_max` / `rmem_default` na 32 MB — glej [Linux gostitelje](#linux-hosts--socket-receive-buffers). Datoteka .deb to opravi ob namestitvi |
| Kamere manjkajo v objavljenih skupinah; vrednost `health.never_arrived_rate_pct` ni enaka nič, `gvsp_corrupt_rate_pct` 0 | Sprožilec / sinhronizacijska pot — kamere se ne sprožajo, ne gre za omrežno težavo | Preverite sinhronizacijski kabel M8 in `--line`; preverite, ali so vsi člani vklopljeni (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` presežena v `analyze-array` ali zavrnjena povezava v strogem načinu (`pin_resolution`; `array over-subscribes the wire`) | Kabel ne more prenesti podatkovnega niza pri nobeni hitrosti, ki jo kamere dosežejo, niti hkrati niti zaporedno — veliko več kamer, kot jih kabel lahko podpira, ali druge kamere, ki po njem prenašajo podatke | Manj kamer, jumbo okvirji od začetka do konca ali hitrejša mrežna kartica. **ROI/binning NE bosta pomagala** (delitev je neodvisna od velikosti okvirja). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` prevlada na testni napravi (sprejme izgubo paketov) |
| Niz se poveže v `slip-emit-and-capture` (serijsko) v počasnem omrežju | Kamer ni mogoče sinhronizirati dovolj počasi, da bi si hkrati delile povezavo, zato delujejo izmenično; okvirji niso izpostavljeni v istem trenutku | Jumbo okvirji ali hitrejša mrežna kartica ponovno omogočijo sočasno zajemanje; `--force-tier sim-capture-sim-emit` vztraja pri tem (in namesto tega izgublja pakete) |

---

## `chloros-cli daq`

Ukazi za spektralni senzor. Ukazi `pool-*` so »tanki« HTTP-odjemalci, ki senzor upravljajo prek trajnega sklopa backenda, zato potrebujejo delujoč backend (glej [Zaženi backend](#start-the-backend)). Backend upravlja s prenosom, zato skripti CLI in SDK delijo en aktivni ročaj, namesto da bi se prepirali za serijsko vrata. `pool-discover` je edini način za ugotavljanje BLE MAC-a DAQ-M.

> **`chloros-cli daq --help`** (ali `-h`, ali `daq help`) prikaže seznam podukazov `pool-*` (v sistemu Linux skupaj z ukazi za neposredno upravljanje strojne opreme, ki jih vključuje zgradba), `chloros-cli daq <subcommand> --help` (ali `daq help <subcommand>`) pa prikazuje možnosti za vsakega od njih. Te zajemajo vzpostavljanje povezave, pretok, snemanje kalibriranih datotek `.daq`, zamenjavo profilov kapacitete ter povezavo DAQ-ja s kamero, za DAQ-E pa tudi omrežne nastavitve, PTP, ponovni zagon in posodobitve strojne programske opreme. DAQ je mogoče upravljati tudi iz Python a s pomočjo `chloros_sdk.connect_daq_sensor()`, ki uporablja isto združeno pot.
>
> **Uporabljajte ukaze `pool-*` na vseh platformah.** Delujejo enako v sistemih Windows, Linux in Jetson. V sistemu Windows sta sta `daq discover` in `daq list` sinonima za `pool-discover`, vsak drug ukaz `daq`, ki ni `pool-*` zaključi `1` z `DAQ direct-hardware commands require the full daq package`. V sistemih Linux in Jetson so ukazi `daq discover`, `daq list` in drugi ukazi, ki niso `pool-*` ali `daq`, neposredni-strojne ukaze: senzor odprejo v lastnem procesu CLI, zunaj sklopa backenda (torej ne na senzorju, ki ga zadržuje backend, GUI ali skript), in tam sprejemajo ukaze `daq discover`, `--only` ter `--timeout`, ne pa `--json`. Ta referenca dokumentira le ukaze `pool-*`.

### Potek prvega povezovanja senzorja DAQ

```bash
# 1. Scan every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U serial port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq pool-discover
chloros-cli daq pool-discover --only ble           # BLE only
chloros-cli daq pool-discover --json               # machine-readable

# 2. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect                                   # smart-detect
chloros-cli daq pool-connect --port /dev/ttyUSB0               # DAQ-U on a serial port (Windows: COM3)
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 3. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 4. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

#    Bind it to a camera, so that camera's captures use and save its readings
chloros-cli daq pool-bind --camera 213800234 CB-7C-A8-2E-5F

# 5. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 6. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Referenca za `pool-*`

| Podukaz | Namen |
| --- | --- |
| `daq pool-connect` (smart-detect) | Odpre senzor v bazenu strežnika. |
| `daq pool-connect --port PORT` (`-p`) | DAQ-U na določenem serijskem vmesniku. |
| `daq pool-connect --ble` | DAQ-M prek BLE, samodejno iskanje MAC-naslova. |
| `daq pool-connect --mac MAC` (`-m`) | DAQ-M prek BLE na znanem naslovu MAC (vključuje `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E prek omrežja Ethernet z znanim imenom gostitelja ali IP-naslovom (vključuje `--eth`). |
| `daq pool-connect --eth` | DAQ-E prek omrežja Ethernet, samodejno odkritje gostitelja (mDNS + ARP kot rezervna možnost; deluje tudi s praznim predpomnilnikom ARP v sistemih Windows in Linux). |
| `daq pool-connect --integration-time MS (-t) --frame-avg N (-f) --no-ae` | Nastavitev integracijskega okna / stanja AE. |
| `daq pool-connect --no-stream` | Poveži se, vendar še ne začni s prenosom (nadaljuj z `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Profil korekcije zgornje meje. Privzeta nastavitev na strežniku je `sunshine_cosine`; `none` je goli senzor. [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor) ga ignorira (fiksni difuzor). **Senzorji se dobavljajo brez pokrova, zato poimenujte pokrov, ki ste ga namestili:** privzeta vrednost se zabeleži kot predpostavljena (`cap_id_source=auto_default`), obdelava pa o tem opozori, saj goli DAQ-U pri korekciji sončnega sevanja odčitava približno 30-krat višje vrednosti. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Preglejte vsak transport za senzorje, s katerimi bi se lahko povezali, ne da bi se dejansko povezali. **Tako poiščete BLE MAC naslov DAQ-M.** `--timeout` velja za posamezni transport (privzeto 5 s, 1–20). Senzorji, ki so že odprti v zbirki, niso navedeni – priključen DAQ-M preneha oddajati oglase –, zato za te uporabite `pool-list`. `usb` prikaže le serijske naprave USB: lastna serijska vrata računalnika (`COM1`, `/dev/ttyS*` v Linux u) ne morejo biti senzorji in so izpuščena. |
| `daq pool-list` | Prikaži vse senzorje v bazenu strežnika. |
| `daq pool-bind (--camera SERIAL \| --array ARRAY_ID) {SENSOR \| --clear}` | Poveži kamero ali niz senzorjev z DAQ-om, tako kot to stori izbira svetlobnega senzorja v grafičnem vmesniku: njene zajeme uporabljajo, usklajujejo in shranjujejo odčitke tega DAQ-ja. `SENSOR` je identifikator senzorja iz `pool-list` ali naslov, na katerem je DAQ dosegljiv. `--clear` (brez `SENSOR`) prekine povezavo, zajemanje pa se vrne na najnovejši DAQ. Povezava traja do ponovnega zagona strežnika ali do prekinitve povezave s to kamero, nizom ali DAQ-jem: prekinitev povezave (tukaj iz SDK-ja ali v aplikaciji) prekine vse povezave s strojno opremo, ki jo prekine, ne glede na to, kdo jih je vzpostavil, vključno s člani niza, ki so povezani z nizom. Kamera mora biti priključena v odprtem projektu ali pa mora biti že prej odprta na tem računalniku, niz pa mora biti priključen ali v odprtem projektu; vsaka druga serijska številka ali identifikator se zavrne po imenu in nič ni vezano. |
| `daq pool-disconnect {--sensor-id ID \| --all}` | Sprostitev — ne glede na to, kdo je odprl senzor, tako da se sprosti tudi senzor, ki ga je v bazenu pustil skript, ki se je sesul (vsak snemalni proces na njem se najprej zaključi). Senzor, ki ni v bazenu, nima ničesar za sprostiti: izpiše `ID: already released (not in the pool)` in se zaključi z `0`; sprostitev, ki jo je backend zavrnil ali ni potrdil, se zaključi z `1`. SDK: `DAQSensorSession.force_disconnect()`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Zadnjih N spektralnih okvirjev (privzeto 1). |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Nadaljuj / zaustavi pretakanje. |
| `daq pool-record --sensor-id ID [--duration SEC] [-o DIR] [--device-name NAME] [--confirm-after S] [--heartbeat S]` / `--stop` | Začni / ustavi snemanje v formatu .daq. `-o` je privzeto nastavljen na `~/Documents/DAQ Live View` na računalniku strežnika (relativna pot je v mapi, iz katere izvajate ukaz); `--duration` (`-d`, privzeto `0` = do `--stop`). `--confirm-after S` opozori, če S sekund po začetku ni prispela nobena vrednost (privzeto 20; 0 to izklopi); z `--duration` `--heartbeat S` vsakih S sekund izpiše število odčitkov in opozori, ko se to število ne povečuje več (privzeto 30; 0 to izklopi). Snemanje na DAQ-E, ki utihne, se nadaljuje v isti datoteki, ko se backend ponovno poveže, `--stop` pa prikaže vrzeli ([DAQ-E, ki preneha pošiljati podatke](#a-daq-e-that-stops-delivering)). Ctrl+C predčasno prekine snemanje `--duration` in o tem poroča; brez `--duration` se snemanje po Ctrl+C (izhod iz `130`) nadaljuje do `--stop`. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Zamenjajte profil korekcije kapacitivnosti med izvajanjem. Potreben je `--cap-id`; `--cap-id none` odstrani korekcijo kapacitivnosti (goli senzor). DAQ-E-S ohrani `as_recorded` ne glede na nastavljeno vrednost. |
| `daq pool-enable-ptp --sensor-id ID` | Ponovno vklopite PTP za DAQ-E, pri katerem je PTP izklopljen. Nastavitev se shrani na enoti, tako da naprava, ki je ostala izklopljena, ostane izklopljena – njeni časovni žigi niso sinhronizirani s PTP – dokler se ta ukaz ne izvede. `pool-list` označi takšno enoto (`!! PTP is switched OFF`) in zanjo izpiše ta ukaz. Chloros nikoli samodejno ne vklopi enote nazaj, saj je lahko izklop nameren. |
| `daq pool-info --sensor-id ID [--json]` | Lastne nastavitve naprave DAQ-E: njena vgrajena programska oprema (in ali ta izdaja Chloros a vsebuje novejšo), njen naslov in kako ga je pridobila (DHCP / link-local / statična), shranjena omrežna nastavitev, dokler jo senzor ne zažene (tista, shranjena z vključenim `--no-reboot`), tok ob vklopu in PTP. Pri senzorju, katerega tok ne dosega tega računalnika, prebere senzor ob novi povezavi in to sporoči. |
| `daq pool-set-network --sensor-id ID {--auto \| --static IP [--mask M] [--gateway G]} [--no-reboot] [--force] [--timeout S]` | Nastavite DAQ-E na DHCP (`--auto`) ali statični naslov (`--mask` privzeto `255.255.255.0`). Senzor se ponovno zažene, da nastavitve uporabi, in se ponovno poveže z nastavitvami, ki jih je imel, pri čemer na to čaka do `--timeout` sekund (privzeto 45); `--no-reboot` jo le shrani. Glej [Nastavitve naprave DAQ-E](#daq-e-device-settings). |
| `daq pool-set-auto-stream --sensor-id ID {--on \| --off}` | Ali DAQ-E ob vklopu začne s prenosom podatkov, če ni priključena nobena programska oprema (shranjeno na senzorju). Program Chloros v obeh primerih samodejno začne s prenosom podatkov. |
| `daq pool-reboot --sensor-id ID [--timeout S]` | Ponovni zagon naprave DAQ-E in ponovna povezava z njenimi prejšnjimi nastavitvami (počaka do `--timeout`, privzeto 45 s). |
| `daq pool-update-firmware --sensor-id ID [--timeout S]` | Naložite firmware naprave DAQ-E, vključen v to izdajo Chloros, na senzor, na katerem teče starejša različica, nato ga ponovno povežite (traja približno minuto; ne izklapljajte ga; `--timeout` privzeto 45 s, da se ponovno vklopi). Senzor, ki je že posodobljen, se pusti pri miru. |

### DAQ-E, ki preneha pošiljati podatke

DAQ-E v skupini, ki 20 sekund ne pošilja spektra, se ponovno poveže prek strežnika. To se doseže z izklopom in ponovnim vklopom stikala, ki napaja senzor: senzor se s tem ponovno zažene in ne pošilja podatkov, dokler ne prejme ukaza (razen če je nastavljen `pool-set-auto-stream --on`).

- **Najprej preveri, ali gre za isti senzor.** Na naslovu, ki ga je imel senzor, brez kakršnih koli sprememb poizveduje po imenu senzorja, MAC-naslovu in serijski številki. Poizveduje tudi pod imenom senzorja `daq-e-<id>.local`, vendar zaupa temu odgovoru le, če prihaja iz senzorjevega lastnega omrežnega adapterja (MAC). Noben drug odgovor ne povzroči ponovne povezave ali zapisa podatkov.
- **Ponovno vzpostavi povezavo z nastavitvami seje:** čas integracije, povprečenje okvirjev, samodejna osvetlitev in profil omejitve ostanejo takšni, kot so bili, ne pa privzete nastavitve senzorja ob vklopu. Senzor, ki je imel vklopljen PTP in se je vrnil z izklopljenim PTP, se ponovno vklopi. Nato ponovno zažene tok podatkov in računalniški sprejemnik zanj.
- **Ponovno poskuša v vse daljših intervalih:** takoj, nato po 5, 10, 20 in 40 sekundah, nato pa vsakih 60 sekund, dokler senzor ni prisoten. Pri vsakem poskusu senzor le vpraša, kdo je. Senzor, ki odgovori, a po treh poskusih ponovne povezave še vedno ne pošilja ničesar, se pusti pri miru in se o njem poroča (`auto-reconnect GAVE UP`). `pool-stream --start` poskusi znova.
- **`pool-list` prikazuje trenutno stanje.** Medtem ko senzor molči, vrstica `STREAM FAULT` poroča o poteku ponovnega povezovanja. Vrstica se izbriše z prvim spektrom, senzor pa nato prikaže `reconnected automatically N time(s)`.
- **Snemanje se nadaljuje v istem `.daq`.** Datoteka označi vsako vrzel z dvema vrsticama: `event_type` 5 pri zadnjem spektru pred vrzeljo in 6 pri prvem po njej, vsaka pa vsebuje le časovni žig. `pool-record --duration` izpiše `!! STREAM LOST …`, medtem ko senzor ni prisoten, in `stream resumed after …`, ko se vrne. `pool-record --stop` navaja vsako vrzel in opozori, tako kot pri pri prazni datoteki (zapis, ki ne vsebuje nobenega odčitka, se zaključi z `1`).
- **Zapis ohrani eno kalibracijo.** Če se senzor med potekom zapisa vrne z drugačno kalibracijo, preostanek zapisa ohrani tisto, s katero se je začel, in opozorilo ob ustavitvi to navaja. `pool-disconnect` + `pool-connect` prevzame novo.
- **Vaši ukazi imajo prednost.** Senzor, ki ga ustavite z `pool-disconnect` ali zaustavite z `pool-stream --stop`, se nikoli ne poveže ponovno, tudi če je ponovna povezava že v teku.
- **`pool-info` na nemem senzorju** odčita senzor ob novi povezavi in doda `!! stream NOT reaching this computer`. Senzor, ki ne odgovarja, se takoj prijavi. `pool-list` ne prikazuje več stanja PTP za senzor, ki je prenehal odgovarjati.
- DAQ-U in DAQ-M nista zajeta. DAQ-M se samodejno ponovno poveže prek Bluetootha, DAQ-U, ki je odklopljen iz USB-ja, pa ponovno potrebuje `pool-connect`.

### DAQ-E-S — fiksni difuzor in senzor naklona

**DAQ-E-S** je DAQ-E z dvema razlikama, ki sta pomembni za programsko opremo. Vse ostalo — odkrivanje, povezovanje, PTP, snemanje, spodnje nastavitve naprave — deluje popolnoma enako kot pri DAQ-E.**Njegov difuzor je fiksno vgrajen, zato ni potrebna nobena korekcija pokrovčka.** Difuzor tipa »Sunshine« je vgrajen in je bil na napravi že ob merjenju tovarniške kalibracije, tako da ta kalibracija že prikazuje dejansko navzdolno sevanje. Program Chloros nastavi njegov pokrovček na `as_recorded` (brez profila), ne glede na zahtevo: omejitev, poimenovana po njem (`sunshine_cosine`, `fov_*`, `none`), se prezre s sporočilom, njegovi zapisi pa shranjujejo `cap_id_source=model`, zato se opozorilo o domnevni omejitvi zanj nikoli ne sproži. Omejitve na njegove podatke ne uporabljajte niti ročno: `sunshine_cosine` na vrhu bi pokazal približno 11-krat višjo vrednost.

**Opremljen je s 3-osnim pospeškomerom.** Kosinusni odziv razpršilnika sončne svetlobe velja le, ko je v vodoravnem položaju, zato nagnjen senzor poroča o napačni obsevanosti – napaka se prenese tudi na odbojnost – brez drugih simptomov. Vsaka odčitana vrednost DAQ-E-S zato vsebuje podatke o svojem položaju:

| Polje | Pomen |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Koti v stopinjah. `None`, kadar programska oprema označi kot za nedoločen (prost pad, mrtva os). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Pospešek v mili-g. Vrednost `mag_mg`, ki se močno razlikuje od ~1000, pomeni, da se je enota premikala in da koti za ta vzorec niso pomembni. |
| `cal_applied` | Nastavljeno: osi in koti so v sklopu difuzorja (uporabljena je kalibracija namestitve enote). Izbrisano: surovi sklop plošče, ki lahko prikazuje do ~2,3° nagiba, ki ni dejanski. |
| `sample_age_ms` | Starost vzorca pospeškometra v trenutku pošiljanja spektra. |
| `temp_raw` | Nekalibrirani **trend** temperature čipa — samo razlike, nikoli odčitek termometra. |

`daq pool-latest --json` jih vrne pod ključem `imu` vsakega okvirja (`null` na DAQ-U, DAQ-M ali navadnem DAQ-E). Podatek `.daq`, posnet z DAQ-E-S, shranjuje iste vrednosti v stolpcih `imu_*` (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) poleg vsakega spektra, tako da bralec lahko razlikuje med oblakom in prevrnjenim stativom. Pri vseh drugih modelih so ti stolpci prazni.

Podatki o položaju se poročajo ločeno: `daq pool-list` izpiše `!! ATTITUDE STREAM FAULT (spectra unaffected)`, medtem ko je naprava izklopljena, ter navede, iz koliko napak se je kasneje obnovila. Napaka na tem mestu nikoli ne povzroči vrzeli v spektralnem zapisu. `daq pool-discover` navaja enoto kot `DAQ-E-S` (kot `DAQ-E`, če ne odgovori na preverjanje identitete v približno 2 sekundah).

### Nastavitve naprave DAQ-E

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330

# A fixed address on the network this computer is on (192.168.1.x)
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1

# Back to the factory setting: DHCP, falling back to a 169.254.x.x
# link-local address when no DHCP server answers
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto

chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

- **`--static` brez `--gateway` ne shrani nobenega prehoda** (vsi prej nastavljeni prehodi se izbrišejo), kar je primerno za omrežje kamer brez usmerjevalnika.
- **Statični naslov mora biti v omrežju tega računalnika.** Chloros zavrne tistega, čigar podomrežje ne vključuje naslova tega računalnika v omrežju senzorja: po ponovnem zagonu senzorja ni mogoče doseči, da bi nastavitev spremenili nazaj. `--force` uporabite le, če bo temu računalniku dodeljen nov naslov ali bo preusmerjen v novo podomrežje.
- **Ponovni zagon je zavrnjen, medtem ko senzor snema.** Najprej ustavite snemanje (`pool-record --stop`). Snemanje, ki se je začelo tik pred ponovnim zagonom, se s tem zaključi, `pool-set-network`, `pool-reboot` in `pool-update-firmware` pa se končajo z vrstico `Warning:`, ki navaja ime datoteke snemanja (tudi če se senzor nato ne vrne).
- **Po ponovnem zagonu Chloros ponovno poveže le sam senzor.** Čaka na mestu, kjer je bil senzor – in po `--auto` tudi pod njegovim imenom `daq-e-<id>.local` – na odgovor z imenom senzorja, ime gostitelja in model, prek omrežnega adapterja (MAC), ki ga je imel, ter ga ponovno poveže na naslov, ki je odgovoril. Odgovor pod tem imenom se sprejme le z navedenim MAC-naslovom, zato se senzor, ki ga je DHCP premaknil na nov naslov, ne poveže ponovno iz računalnika, ki dostopa do senzorja prek usmerjevalnika, ali iz WSL2 v njegovem privzetem omrežju NAT: ukaz to sporoči in izpiše ukaz `pool-connect --eth-host <its address> …`, ki ga ponovno vzpostavi z nastavitvami osvetlitve in omejitve, ki jih je imel.
- **`pool-update-firmware` zavrne senzor, katerega različice vgrajene programske opreme ne more prebrati.** Senzor, ki po bliskavici potrebuje več časa za vrnitev, kot je čas čakanja, ni okvarjen: takoj ko odgovori, se ponovno poveže z vnesenimi nastavitvami nove programske opreme, ukaz pa opozori, da je bil počasen.
- **Posodobitev programske opreme zahteva vhodno TCP-povezavo.** Senzor se ponovno poveže s tem računalnikom, da pridobi sliko. Program Chloros prisluhne na vmesniku, ki usmerja promet do senzorja (na vseh vmesnikih, le kadar ne more ugotoviti, kateri je pravi), in obravnava le naslov senzorja samega; vsaka druga povezava se zapre, ne da bi bila obdelana. V sistemu Windows jo pravilo požarnega zidu `Chloros Backend (TCP-In)` v namestitvenem programu dovoli (glej [Windows (.exe)](#windows-exe)); požarni zid, ki jo blokira, povzroči napako pri posodobitvi z napako `… device never connected back on TCP <port> …`.
- **V okolju WSL2 uporabite zrcaljeno omrežje** (`networkingMode=mirrored` v `%UserProfile%\.wslconfig`). V privzetem načinu NAT je naslov tega računalnika v omrežju senzorja naslov samega WSL-ja, zato se vsak statični naslov zavrne (razen `--force`), neposredni prenos ne doseže WSL-ja, posodobitev strojne programske opreme pa se ne more zaključiti, ker se senzor ponovno poveže z računalnikom, da prejme sliko.
- **ID senzorja in ime gostitelja se ne spremenita.** Ne glede na naslov je DAQ-E še vedno `daq-e-<id>` in `daq-e-<id>.local`, zato ga `--eth-host daq-e-<id>.local` še naprej najde.
- **GUI ima enake nastavitve**: zavihek »Light Sensors«, nastavitve senzorja (zobnik), »Network«. GUI ohranja lastno povezavo s senzorjem, zato sprememba senzorja, s katerim ima GUI odprto povezavo, iz GUI: ponovni zagon prek CLI-ja prekine sejo grafičnega vmesnika s senzorjem.
- Ustrezni SDK-ji so `DAQSensorSession.device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` in `update_firmware()`.

---

## `chloros-cli project`

Odprite, povežite se z in upravljajte shranjen projekt Chloros (mapa z datotekami `cameras.json` + `sensors.json` + `project.json`). Vse poteka prek strežniškega dela, zato grafični vmesnik (GUI) in vmesnik ukazne vrstice (CLI) prikazujeta enako stanje strojne opreme.

### Referenca podukazov

| Podukaz | Namen |
| --- | --- |
| `project open PATH [--relink FOLDER]` | Izpiše seznam naprav projekta (kamere, matrice, senzorji), prebere podatke iz njegovih datotek in preveri izvorne datoteke (glej [Premaknjen projekt](#a-moved-project---relink)). Projekt se v ozadju ne odpre: projekt, ki je tam odprt, ter njegove naprave in posnetki ostanejo takšni, kot so. Izstopi iz `1`, če ni mogoče najti nobene izvorne datoteke projekta. |
| `project devices PATH [--reconnect]` | Prikaže seznam ali ponovno izvede odkrivanje. |
| `project connect PATH [--cameras-only] [--sensors-only] [--packed {auto,on,off}]` | Poveže vsako shranjeno kamero / niz / senzor. Vsak niz se poveže na enak način kot `lattice array-connect`: do 600 s, njegovi koraki se izpišejo v stderr (`[array <id>]    <s> s: <step>`; stdout je poročilo v formatu JSON), niz, ki v tem času ni povezan, pa se razveljavi — kamere se vrnejo v stanje, v katerem so bile najdene — in se poroča pod `errors`, nikoli pa ne ostanejo povezane. `--packed` nastavi pakirani 12-bitni prenos za vsak niz (privzeto `auto`; projekt ne shranjuje nastavitev za posamezne nize; glej [Pakirani 12-bitni prenos](#packed-12-bit-transport---packed)). Vsak svetlobni senzor se odpre v DAQ-skladišču backenda na shranjenem naslovu s shranjenim časom integracije, povprečevanjem okvirjev, samodejno osvetlitvijo in omejitvijo: senzor, za katerega ni izbrana omejitev, dobi `sunshine_cosine`, tisti, ki je shranjen brez omejitve (`none`), ostane brez omejitve, DAQ-E-S pa nikoli ne uporablja omejitve. DAQ-panel v grafičnem vmesniku ni del sklopa: senzor najprej zaprite tam. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Enkratni zajem iz imenovane kamere ali matrice (`--prefix` privzeto, `capture`). `--orient` (nizi): niz shranjuje surove podatke, ki ostanejo v izvirni obliki — to pomeni, da je treba izdelke, ki jih `process` ustvari iz njih, usmeriti, kot je `array-capture --orient`. Privzeto: izvirna oblika. `project burst` / `stream` ter `capture` / `burst` / `stream` (`orient: true`) imajo enako možnost. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Serija N posnetkov iz imenovane kamere ali matrice (`-n/--count` privzeto 5; `-i/--interval` sekunde med posnetki, privzeto 0). Serije posnetkov iz niza odstranijo podvojene ponavljajoče se sinhronizirane skupine (nadzor zastarelosti), tako da niz z delnim ciklom ne more vrniti N kopij enega okvirja; izpiše rezultate po posameznih ponovitvah. Če zajem med potekom ne uspe, se serija ustavi in program `1` se zaključi s sporočilom, koliko posnetkov je bilo shranjenih pred tem. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--orient]` | Zajemanje na disk v zanki: `--fps F` zajame drugo (privzeto 3), en okvir na zajem iz kamere, eno sinhronizirano skupino okvirjev iz niza (z istim nadzornikom zastarelosti kot `project burst`), dokler `-n N` ne zajame (privzeto `0` = dokler ne pritisnete Ctrl+C, kar izpiše, koliko jih je bilo shranjenih, in zaključi `0`, ali `1`, če ni bilo shranjenih ali je prišlo do napake pri datoteki; če pritisnete Ctrl+C, preden `-n N` zajame, se `130` zapre). Zajem, ki se med potekom prekine, ustavi tok in zaključi `1`, pri čemer navede, koliko sličic je bilo shranjenih pred tem. `--poll-interval` je zastarel in se ignorira: še vedno se sprejema, zato starejši skripti še vedno delujejo, s pripombo na stderr. |
| `project sensor read PATH NAME [--json]` | Najnovejši spektralni okvir. |
| `project sensor log PATH NAME [--seconds SEC] [-o DIR] [--device-name NAME]` | Snemanje datoteke .daq za `--seconds` (privzeto 10). Če posnetek ne vsebuje nobenih odčitkov, se to sporoči na stderr in program se zaključi `1`. |
| `project run PATH RECIPE.yaml` | Izvedba recepta za zajem v formatu YAML/JSON. `--dry-run` preveri veljavnost brez izvedbe. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Izračunaj poravnavo za niz — glej [tabelo zastavic spodaj](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Izpiše trenutni profil poravnave. |
| `project align clear PATH NAME` | Izbriše profil iz predpomnilnika. |
| `project align tweak PATH NAME --serial SN [--dx N] [--dy N] [--rotation-deg N] [--scale N]` | Premakne transformacijo enega od podrejenih procesov (privzeto `0`, `0`, `0`, `1.0`: brez spremembe). |
| `project align export PATH NAME --to FILE` | Shrani profil v JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Naloži shranjeni profil. |

#### `project align calibrate` Možnosti

| Oznaka | Privzeto | Opis |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Metoda poravnave. **Te zapise se razlikujejo od `lattice align-calibrate`**, ki sprejema kratke oblike `orb` / `akaze` / `phase`; ti dve ukazi nista zamenljivi pri tem zastavku. |
| `--model {translation, rigid, affine, homography}` | `affine` | Preoblikuj model, da se prilega. |
| `--frames N` | `1` | Sinhronizirane posnetke okvirjev na povprečje. |
| `--reference SN` | glavne | Serijska številka referenčne kamere; vsi ostali člani so deformirani nanjo. |
| `--max-features N` | `5000` | Omejitev števila značilnosti ORB. |
| `--ratio-threshold F` | `0.75` | Test razmerja Lowe'test razmerja. |
| `--ransac-threshold-px F` | `3.0` | Prag za notranje točke v RANSAC-u. |
| `--min-matches N` | `15` | **Kakovostni prag** — zavrni rešitev, če je število ujemanj z notranjimi točkami manjše od te vrednosti. |
| `--max-reproj-err-px F` | `4.0` | **Kakovostni prag** — zavrni rešitev, če presega to RMS napako reprojekcije. |
| `--checkerboard RxC` | — | Geometrija plošče za `--method checkerboard`, npr. `9x6`. |
| `--name PROFILE` | prazno | Ime profila, vključeno v shranjeni JSON. **Ne ime niza** — to je pozicijski `NAME`. |

Ti dve merili kakovosti sta razlog, da lahko kalibracija uspešno reši problem, a kljub temu
zavrne shranjevanje: profil, ki ne izpolnjuje enega od njiju, bi tiho napačno registriral vsak
kasnejši posnetek, zato se zavrne, namesto da bi se shranil.

### Premaknjen projekt (`--relink`)

Projekt `project.json` poimenuje vsako izvorno datoteko – vsako sliko in posnetek svetlobnega senzorja
– po njeni polni poti. Če premaknete mapo projekta ali celotno mapo projektov
na drug pogon, te poti še vedno kažejo na staro mesto.

Odpiranje projekta samodejno popravi, kar lahko: datoteka, ki je bila v projektni
mapi, se poišče v mapi, v kateri se projekt nahaja zdaj, vsaka pot, ki se tam najde,
pa se preusmeri in shrani (`project.json` se najprej varnostno kopira kot
`project.json.before-relink-<date>-<time>`). To se zgodi ob vsakem odpiranju – aplikacije,
`chloros-cli project open`, `chloros-cli process` v obstoječi projekt, v SDK-je
in v `chloros-mcp`-je. Pot se vedno preusmeri le na datoteko, ki je tam.

Datotek, ki so bile zunaj projektne mape – seansa prek Hub-a, uvoz z »Dodaj mapo« –
na ta način ni mogoče najti. Navedite ime mape, v kateri se zdaj nahajajo:

```bash
chloros-cli project open "N:/Chloros Projects/Field_A" --relink "E:/backup/Hub Projects"
```

`--relink` upošteva mapo, v katero so bile datoteke prenesene, ali katero koli mapo nad njo: mapa vsake manjkajoče
datoteke se išče pod njo po končnih imenih map
(`Hub Projects/<session>/raw/<serial>`, nato `<session>/raw/<serial>`, ...), datoteke v vsaki
datoteke v mapi se vzamejo iz tistega mesta, kjer se jih najde največ. `project open` izpiše,
kaj je bilo ponovno usmerjeno, koliko izvornih datotek je bilo najdenih in v katerih mapah so bile
ostale. Nekaj jih še vedno manjka: opozorilo, izhod `0` – projekt se odpre z ostankom. Ni
najdenih: napaka pri poimenovanju teh map in `--relink`, izhod `1`; tudi backend
noče odpreti takšnega projekta za `process` ali `project devices --reconnect` (HTTP 409 `PROJECT_SOURCES_MISSING`).

### Primeri

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Its hub captures moved to another drive: point it at them
chloros-cli project open "/home/user/Chloros Projects/Field_A" --relink "/media/backup/Hub Projects"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### Jezik DSL za recepte

`project run RECIPE.yaml` sprejema datoteko v formatu YAML ali JSON, ki opisuje zaporedje dejanj:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        auto_exposure: "Off"
        gain_db: 6.0
        auto_gain: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Podprta dejanja: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. `apply` prevzame ključe `cameras.json`, `settings` — `exposure_us`, `auto_exposure`, `gain_db`, `auto_gain`, `target_brightness`, … (starejša imena `exposure_time`, `exposure_auto`, `gain`, `gain_auto` se sprejemajo kot sinonimi). Na ta način nastavljena osvetlitev, ojačanje ali ciljna vrednost se ohranita kot vaši: avtomatsko krmiljenje osvetlitve v programu Chloros (vključno z osvetlitvijo, izmerjeno z DAQ) ju pusti pri miru. Ročna osvetlitev pusti ojačanje takšno, kot je bilo: v avtomatskem načinu kamera še naprej prilagaja ojačanje (v celotnem obsegu) v smeri ciljne svetlosti, dokler ne nastavite `gain_db` ali `auto_gain: "Off"`. Kasnejši ukaz `auto_exposure: "Continuous"` vrne osvetlitev v avtomatski način, ukaz `target_brightness_auto: true` pa zadržano ciljno vrednost vrne v avtomatsko nastavitev ciljne vrednosti programa Chloros. Akcija `burst` sprejme `name` (obvezno), `count` (privzeto 5), `interval` (sekunde, privzeto 0), `output`, `format`, in `settings` (enaka oblika nastavitev na kamero kot `apply`); zaporedni posnetki v nizu uporabljajo isti nadzorni mehanizem za sveže sinhronizirano skupino kot `project burst`.

A `capture`, `burst` ali `stream` se privzeto nastavi na `output`. Relativna pot `output` — vključena kot privzeta — se razreši glede na mapo, iz katere zaženete `chloros-cli project run`, ne glede na mapo datoteke recepta ali mapo projekta (v različici 1.2.0 se je nahajala v namestitveni mapi backenda).

Zaženite ga:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

**Vsako izvajanje se najprej preveri glede na projekt.**Vsaka akcija mora imeti kot ključ natanko enega od zgornjih glagolov — `- capture: {name: FrontLeft}`, ne pa `- action: capture` ali `- type: capture` — pri `name` ima projekt `cameras.json` / `sensors.json` kamero ali matriko za `apply` / `capture` / `burst` / `stream` ter svetlobni senzor za `sensor`; `stream` potrebuje `count` > 0, `wait` pa trajanje. Recept z kakršno koli takšno napako**ne** izvede nobenega ukrepa (`[recipe] error: …`, `[recipe] not run: …`, izhod `1`), niti ob prisotnosti `stop_on_error: false`. Ključi, ki jih korak ne uporablja, so `[recipe] warning:`, prav tako pa tudi vsak niz ali svetlobni senzor, ki ga recept upravlja, a trenutno ni priključen. Recept nikoli ne poveže naprav: najprej zaženite `chloros-cli project connect PATH`. `--dry-run` natisne recept, nato iste napake in opozorila, `[recipe] valid: …` / `NOT valid: …`, ne izvede ničesar in se zaključi z `1`, če recept vsebuje napako.

---

## Okoljske spremenljivke

| Spremenljivka | Učinek |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Prepisuje URL strežnika (privzeto `http://127.0.0.1:5000`) — **upošteva se le pri družinah ukazov `lattice`, `project` in `daq pool-*`** (`lattice network-analysis` namesto tega uporablja `--backend-url`). Osnovni ukazi (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) nastavijo `http://127.0.0.1:<port>` in prezrejo to spremenljivko (literal IPv4 zaobide kazen sistema Windows, ki znaša približno 2 sekundi na zahtevo za `localhost`→`::1`), zato vedno ciljajo na lokalni računalnik. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | Nastavi se v okolju strežnika. `1` zmanjša zavrnitev zaradi prekomernega števila prijav — povezava s poljem `pin_resolution` ali [začasni ukaz za več kamer](#multi-camera-transient-sessions) (`error_code` `ARRAY_OVERSUBSCRIBED`) — na glasno opozorilo, pri čemer se dopušča izguba paketov GVSP. Samo za testiranje — glej [Model fps in burst za matriko](#array-fps--burst-model). |
| `CHLOROS_ARRAY_PACKED12` | Nastavi se v okolju backenda: `on` ali `off` nastavi privzeto vrednost [zgoščenega 12-bitnega prenosa](#packed-12-bit-transport---packed) za vsak niz (privzeto `auto`). Izrecna nastavitev `--packed` ima še vedno prednost. |
| `CHLOROS_DISABLE_UPDATE_CHECK` | `1` (ali `true` / `yes`) ob zagonu izklopi [samodejno preverjanje posodobitev](#chloros-cli-update) v Linuxovem CLI-ju. |
| `CHLOROS_CLI_MODE` | Nastavi ga sam CLI; naroča backendu, naj omogoči vzporedno obdelavo. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` preskoči GVSP-preizkus za nadomestno rešitev (samo rezultati ICMP). **To izklopi jumbo, ne le utiša dnevnik** — kamera odgovarja na DF-pingove le do 1500 na vsaki poti, zato je ta preizkus edina stvar, ki lahko zazna jumbo. Prihrani ~1 s na kamero na povezavo; stane ~1,45× zmogljivost povezave, če bi omrežje *lahko* prenašalo jumbo pakete. SDK opozori, ko to nastavite. |
| `CHLOROS_GVSP_SESSION_REUSE` | Nastavite v okolju strežnika: `0` meri velikost paketa ob vsaki povezavi namesto enkrat na kamero na sejo strežnika. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Fiksirajte velikost paketa GVSP na N bajtov; popolnoma preskoči preizkušanje. Prednost dajte nastavitvi na ukaz (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) pred trajno nastavitvijo: fiksirana velikost se ne prilagaja več omrežju pred njo, fiksiranje vrednosti 9000 na poti, ki ne podpira jumbo paketov, pa povzroči, da **vsak** zajem pri `SC_ERR_TIMEOUT -1011` preseže časovno omejitev. |
| `CHLOROS_TRANSIENT_AE_SETTLE_S` | Nastavi se v okolju strežnika: največje število sekund, ki jih traja [začasna večkamerna ukazna sekvenca](#multi-camera-transient-sessions) ali `lattice capture` kamere, ki ničesar ne zadržuje, porabi za nastavitev osvetlitve vsake kamere iz prizora, preden shrani posnetek (privzeto `5`). `0` to izklopi: avtomatska osvetlitev vsake kamere se nato izvaja od prvega okvirja, začenši z njeno mejo osvetlitve. |
| `CHLOROS_LOG_DIR` | Nastavi se v okolju strežnika: mapa, v katero se shranjuje podrobni dnevnik. Privzeto `%LOCALAPPDATA%\Chloros\logs` v sistemu Windows, `~/.cache/chloros/logs` v sistemu Linux (`$XDG_CACHE_HOME/chloros/logs`, če je to nastavljeno). Koreninska storitev paketa .deb nastavi `/var/log/chloros`, ki je berljiva za skupino `adm` — glej [Zagon backenda kot storitve](#running-the-backend-as-a-service-headless). |
| `TMPDIR` (Linux) | Mesto za začasne datoteke. CLI samodejno uporabi `/mnt/ssd/tmp`, če je prisoten. |

---

## Izhodne kode

| Koda | Pomen |
| --- | --- |
| `0` | Uspeh. |
| `1` | Splošna napaka (večina napak podukazov). Ukaz za zajem se zaključi tudi s kodo `1`, če se katera koli datoteka, ki naj bi se zapisala, ni uspela zapisati, čeprav so se ostale shranile, ter s kodo `array-burst` / `array-record` zaključi delovanje `1`, ko snemalnik poroča o napaki — glej [Datoteke, ki jih ni mogoče zapisati](#files-that-fail-to-write). `process` se prav tako zaključi `1`, če izvozov iz izvedbe ni bilo mogoče preveriti ali če je izgubil sliko, čeprav so bili drugi izdelki zapisani. |
| `2` | Napaka v argumentu (neznan ali manjkajoč zastavica; prav tako `time-sync set-priority` brez obeh prednostnih stopenj). |
| `130` | Prekinjeno s Ctrl+C: kateri koli ukaz, ki ga prekine — `lattice`, `project`, `daq` in `process` (`process` po tem, ko je backend ustavil izvajanje, glej [Opombe k `process`](#notes)). Ukaz, ki se običajno ustavi s tipko Ctrl+C — `lattice array-capture --continuous` / `--interval`, `lattice viewer`, `lattice continuous`, `multi-stream`, `align-stream`, `array-record`, `array-burst` in `project stream` z `-n 0` — se namesto tega zaključi glede na to, kaj je naredil: `0`, ko je bilo vse zapisano, `1`, ko je kaj spodletelo. |
| `143` / `129` | Prekinjeno s signalom SIGTERM / SIGHUP (128 + signal), potem ko je potekalo čiščenje ukaza: tekoča naloga se je ustavila in kamere so bile vrnjene v prvotno stanje, obdelava v ozadju se je ustavila. Tudi `lattice continuous` ali drug posnetek, ki se običajno ustavi s tipkama Ctrl+C, zapre `143`, ne pa `0`. |

`lattice network-analysis` zaključi `1` za odločitve `needs_force_slip` in `error`, `selftest` pa zaključi `1`, razen če je vseh sedem preverjanj uspešnih.

---

## Nasveti za odpravljanje težav

- **»Potrebna je prijava«** → Na tem računalniku enkrat zaženite `chloros-cli login EMAIL PASSWORD`.
- **»Mapa projekta je za ta računalnik pregloboka«** → Na tem računalniku so dolge poti v sistemu Windows onemogočene (privzeta nastavitev sistema Windows), zato nobena pot ne sme presegati 260 znakov, izdelki pa se nahajajo do približno 130 znakov pod mapo projekta. Nič ni bilo obdelano. Sporočilo navaja najdaljšo mapo projekta, ki ustreza: uporabite krajšo (`-o` ali `chloros-cli set-project-folder`) ali vklopite dolge poti v Windowsu in ponovno zaženite Chloros. Namestitveni program za Windows to ponuja kot možnost »Omogoči dolge poti v Windows«, ki je privzeto izbrana; sicer uporabite skupinsko politiko: *Konfiguracija računalnika > Upravne predloge > Sistem > Datotečni sistem > Omogoči dolge poti Win32*.
- **»backend unreachable«** → Zaženite namizno aplikacijo Chloros (Windows) ali sami zaženite backend: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` v sistemu Windows (mapa `backend` v vaši namestitveni mapi; ni v mapi `PATH`), `chloros-backend` v sistemu Linux. Ali pa preverite `CHLOROS_BACKEND_URL`, če gre za oddaljeni dostop.
- **Ukazi `lattice` se ne izvedejo z napako »LATTICE camera drivers not found«** → Backend ni mogel naložiti izvedbenega okolja za kamere LATTICE, s katerim je bil nameščen. Vsak ukaz `lattice` se izvaja v ozadju; sam CLI ne vsebuje gonilnikov za kamere. Ponovno namestite Chloros: namestitveni program za Windows in datoteka .deb namestita izvedbeno okolje poleg ozadja.
- **V oknu »Array connect« / »Array Settings« se prikaže sporočilo »FRAMES WILL DROP« ali »Reduce ROI to enable«** → Obroč za sprejem na omrežni kartici gostitelja je premajhen (običajno se po posodobitvi gonilnika omrežne kartice ponastavi na 32). Glejte [Nastavitev in optimizacija omrežne kartice gostitelja](#host-nic-setup--tuning-lattice-arrays) — nastavite `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Računalnik se ob ponovnem zagonu/izklopu zatakne, nato pa WMI `Invalid class` / omrežna kartica se ne vklopi / manjkajo USB-pogoni** → Zastarel gonilnik USB 10GbE adapterja povzroča napako `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Posodobite gonilnik adapterja — glejte [Nastavitev in optimizacija omrežne kartice gostitelja](#host-nic-setup--tuning-lattice-arrays).
- **Sistem Linux: zajeti podatki pripadajo uporabniku root, zato jih ne morete izbrisati, dnevnik backenda pa ni v vašem `~/.cache`** → Backend teče kot root prek `chloros-backend.service`. Preklopite na `chloros-backend@<user>.service`, kjer se izvaja kot vi — glejte [Zagon backenda kot storitve](#running-the-backend-as-a-service-headless). Do takrat se dnevnik storitve z uporabniškim računom root nahaja v `/var/log/chloros` (berljiv za skupino `adm`).
- **Linux: enota backenda je `failed` z napako »Druga instanca že teče« (izhodni status 98)** → Drugi backend že zaseda vrata: hkrati teče le ena instanca. Če sta omogočeni dve enoti backenda, ohranite eno: `sudo systemctl disable --now <unit>` za tisto, ki je ne uporabljate — glejte [Zagon backenda kot storitve](#running-the-backend-as-a-service-headless). Če je omogočena le ta enota, jo zaseda backend, ki je bil zagnan zunaj systemd (z `chloros-backend`, `chloros-cli` ali skriptom SDK): ustavite ta backend, nato pa `sudo systemctl start <unit>`.
- **Opozorilo o zamenjavi pomnilnika na Jetsonu** → Dodajte zamenjavo pomnilnika, podprto s datoteko; CLI izpiše natančne ukaze `fallocate` / `swapon`.
- **»backend ni dosegljiv« iz ukaza `lattice`, `project` ali `daq`** → noben backend ne teče: ti ukazi ga nikoli ne zaženejo. Zaženite ukaz `chloros-cli backend status` in si oglejte [Zagon backenda](#start-the-backend).
- **»`daq net ...` ni na voljo v Chlorosu«** (Linux in Jetson) → Nekatere lastne nastavitve naprave DAQ-E so rezervirane za storitev MAPIR. Mrežne nastavitve, ponovni zagon in posodobitve firmware-a so `pool-set-network`, `pool-reboot` in `pool-update-firmware`; za ostalo se obrnite na podporo MAPIR.
- **»Neposredni strojninski ukazi za DAQ zahtevajo celoten paket daq«** (Windows) → Uporabite `pool-*`, ki isti senzor upravlja prek backenda, ali `chloros_sdk.connect_daq_sensor()` iz Pythona.

---

## Glejte tudi

- [Referenca Python SDK](sdk-reference.md) — programski ekvivalent vsakega ukaza CLI.
- [Senzorji svetlobe DAQ](https://mapir.gitbook.io/chloros/daq-light-sensors) v priročniku Chloros — senzorji, njihove omejitve in kalibrirano območje, snemanje, omrežje DAQ-E.
- [Kamere LATTICE](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) v priročniku za Chloros — priključevanje, nastavitve kamere, načini zajemanja, matrike.
- Spletna dokumentacija: `https://mapir.gitbook.io/chloros/cli`

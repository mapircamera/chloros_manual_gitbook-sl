# Referenca za Chloros Python SDK

**Različica:**

1.3.0**Ustvarjeno:**

29. 7. 2026 ob 19:19 ·**Posodobljeno:**

6. 11. 2026**Paket:** `chloros-sdk` (PyPI)**Ciljna publika:** Optimizirano za uporabo v velikih jezikovnih modelih (LLM); berljivo za ljudi.**Obseg:** Vse javne razrede, funkcije in pomožne funkcije, ki jih ponuja `import chloros_sdk`, s primeri, ki jih je mogoče kopirati in prilepiti ter ki zajemajo obdelavo slik, nadzor ene kamere, sinhronizirane nize, DAQ-senzorje in avtomatizacijo projektov, ter strežnik `chloros-mcp` za AI-pomočnike.

Če vas zanimajo le najpomembnejše informacije, preskočite na:
- [Namestitev in hiter začetek](#namestitev)
- [Smart-Connect za matrike LATTICE](#smart-connect-za-kamere-lattice)
- [Sejne nastavitve senzorjev DAQ](#sejne-nastavitve-senzorjev-daq)
- [Pripomočki za kamere](#camera-utilities)
- [Prehodni nizi, profili poravnave in indeksi](#transient-arrays-alignment-profiles--indices)
- [Projektna mapa in recepti](#project-folder--recipes)
- [Avtomatizacija projektov](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Živi okvirji v vašem lastnem delovnem toku](#live-frames-in-your-own-pipeline)
- [MCP-strežnik (AI-pomočniki)](#mcp-server-ai-assistants)

> **Spremembe v različici 1.3.0** (kar zazna obstoječi skript za različico 1.2.0).
>
> **Spremembe, ki jih bo morda treba obravnavati v skriptu za različico 1.2.0:**
>
> - **Izdaja, ki jo backend ne potrdi, zdaj sproži napako.** `CameraSession.disconnect()`, `ArraySession.disconnect()` / `force_disconnect()` na nizu, ki ga je odprla seja, `DAQSensorSession.disconnect()` / `force_disconnect()` na senzorju, ki ga je odprla, ter `ChlorosProject.disconnect_all()` sprožijo napako `ChlorosConnectError`, kadar backend zavrne sprostitev ali je ne potrdi pravočasno, saj je naprava morda še vedno zasedena. V različici 1.2.0 so se ti tiho vrnili. Izhod bloka `with` še vedno nikoli ne sproži izjeme.
> - **Izposojena naprava se nikoli ne razveljavi.** `with attach_array(...)` in eksplicitni `disconnect()` na njej pustita niz v delovanju (v različici 1.2.0 je zapustitev bloka povzročila razveljavitev, tudi če je bil v lasti grafičnega vmesnika); pokličite `force_disconnect()`, če ga želite zaključiti. Podobno tudi `connect_daq_sensor()`, ki je zaznal, da je senzor že v zbirki (vključno z inteligentnim zaznavanjem z zastavico nič), ga pusti odprtega na `disconnect()` in ob zapustitvi svojega bloka `with`.
> - **PNG in JPEG sprejemata le `display` in indeksne izdelke.** `capture()` z `format="png"` / `"jpg"` (`ext=".png"` / `".jpg"`) ter na ravni `radiance` ali `reflectance`, ali `processing="all"`, pred vsako zahtevo sproži `ValueError`; 12-bitni `raw` / `debayered` je zavrnjen s strani backenda. Glej [PNG / JPEG](#png--jpeg-take-the-display-and-index-products-only).
> - **`connect_daq_sensor(sensor_id=...)` odpre ta senzor ali sproži** `ChlorosConnectError`, ki navaja, kaj je našel; nikoli ne preklopi na drug senzor. Glej [Pinned Transport](#pinned-transport).
> - **`frame_stream()` sproži izjemo namesto da utihne:** `FrameFetchError` ob zavrnitvi v naslednjem taktu ponovi (odčitek odbojnosti brez vključenega DAQ), po 60 sekundah brez okvirja in po 60 sekundah brez novega sprožilca. Glej [En sprožilec na odčitek](#one-trigger-per-read--group_id).
> - **Časovna omejitev odčitka ali prekinjena povezava sproži `ChlorosConnectError`** pri vsakem klicu smart-connect in `analyze_array_network()`, kjer je različica 1.2.0 sprožila osnovno časovno omejitev ali napako povezave.
> - **`connect_array()` po privzetku čaka 600 s (`timeout`, 180 s v različici 1.2.0), povezava, ki jo opusti, pa se razveljavi.** Klic pošlje svojo napako `timeout` v ozadje, ki vsako kamero vrne v stanje, v katerem je bila najdena, in ne vzpostavi nobene povezave, če se povezava ne vzpostavi pravočasno; klic sproži napako `ChlorosConnectError` (HTTP 504 `CLIENT_DEADLINE_PASSED`). V različici 1.2.0 se je povezava takoj po vklopu kamer lahko zavlekla dlje od trajanja klica in povezala niz, potem ko je ta že sprožila napako. `connect_all()` na enak način čaka na vsak shranjen niz (120 s v različici 1.2.0). Glej [`connect_array()` podpis](#connect_array-signature).
> - **`connect_array()` ohranja celoten senzor.** Brez `width` / `height` / `binning` je vsaka kamera nastavljena na polni senzor, hitrost snemanja pa pade na raven, ki jo zagotavljata gostitelj in omrežje; v različici 1.2.0 je okvirje skrčil ali združil, da bi ohranil hitrost. Glej [Sinhronizirani niz](#synchronized-array--arraysession-smart-prep).
> - **Bayerjev format pikslov na kameri M3M (mono) je njena globina.** Kamera M3M ponuja le mono formate, prednastavitve pa so poimenovane po Bayerjevem formatu: v različici 1.2.0 je kamera zavrnila ta format in ohranila svojega, medtem ko je `settings.pixel_format` poimenoval Bayerjevega. `LatticeCamera(preset=...)`, `apply_preset()`, `set_format()`, `CameraSettings(pixel_format=...)`, `connect_camera(preset=...)` ter nizskupni format zdaj določajo format Mono iste globine na M3M: `"BayerRG8"` in `high_speed` dajeta `Mono8`; `"BayerRG12"`, `high_quality` in `triggered` dajejo `Mono12`; `cam.settings.pixel_format` ga poimenuje. Enako velja tudi v nasprotni smeri: M3C se nikoli ne shrani v formatu Mono, zato `"Mono12"` – iz katerega koli od teh, `connect_array(pixel_format=...)`, `set_settings(pixel_format=...)` ali shranjenega formata niza – na M3C ustvari `BayerRG12`, ne glede na to, katera kamera je glavna v nizu. `CameraSettings`, ki ga posredujete, ohrani svoje ime, zato isti objekt še vedno zahteva od vsake kamere to globino v svoji družini. `connect_camera(preset=...)` ohrani format prednastavitve (v različici 1.2.0 se kamera po uporabi prednastavitve nastavi na 12-bitno delovanje), kamera, odprta na podlagi prednastavitve, pa ne zapisuje več tistega, kar je zajela, v `PRESETS`. Vsak drug format, ki ga kamera zavrne (ki ga ni na njenem seznamu `PixelFormat`, kot na primer `RGB8` na M3M ali `BayerRG16` na firmwareju brez njega), se tudi ne poimenuje kot samostojen: po povezavi s `LatticeCamera` povezave, `apply_settings()` ali `set_format()`, `cam.settings.pixel_format` in `packed_transport` navajajo format, v katerem kamera prenaša podatke, ena natisnjena vrstica pa navaja format, ki ga je zavrnila.
> - **`analyze_array_network()` prikazuje, kateri priključek se uporablja:** `pixel_format=None` (prej `"BayerRG8"`) pomeni formate, ki jih določa priključek (Mono12 / BayerRG12), `packed="auto"` pa odločitev o pakiranem prenosu, zato se lahko število sličic na sekundo, ki ga navaja, razlikuje od 1.2.0'. `pixel_format`, ki ga posredujete, predstavlja M3M na tej globini. Glejte [Pomočnik za analizo omrežja](#network-analysis-helper).
> - **`CameraSession.capture()` vrne `CaptureResult`** (še vedno seznam slovarjev okvirjev) z `.skipped`, `.failed`, `.daq_files` in `.daq_matches`. Rezultat `ArraySession.capture()` vsebuje tudi `.alignment` (ali je bila uporabljena poravnava in zakaj ne) ter `.oriented`.
> - **Zajemi v obliki niza so privzeto poravnani, tako kot so poravnani tudi v aplikaciji.** `ArraySession.capture()`, `continuous()`, `capture_repeated()` ter `ArrayHandle.capture()` / `capture_smart()` poravnajo `debayered`, `display`, `radiance`, `reflectance` ter indeksne datoteke v okvir glavne kamere in jih obrežejo na prekrivanje članov (manjše od senzorja za odmike kamer, `XMP-mapir:AlignmentJSON` `"applied": true`), tako da so enake datoteke kot posnetek iste scene v aplikaciji Chloros; `raw` ostane v izvirni velikosti. Prej je zajem brez `aligned=True` vsako datoteko shranil v izvirni velikosti, brez popravka izkrivljenosti. `aligned=False` to ohranja. `chloros-mcp``capture_array` in `start_capture_run` uporabljata isto privzeto nastavitev. Glej [`ArraySession` metode](#arraysession-methods).
> - **`connect_all()` navaja backend, ki zavrača SDK.** Delujoči backend, ki je odjavljen ali ima naročniški paket nižji od Copper, se poroča pod `errors` z lastnim razlogom (HTTP 401 / 403), ne pa kot »ni zagnanega backenda Chloros«, vsak klic `connect_all()` pa ponovno preveri backend, zato klic po prijavi deluje na istem projektnem objektu.
> - **Relativni `output_dir` deluje v povezavi z lokalnim backendom.** Relativne poti se pred pošiljanjem pretvorijo v absolutne glede na delovno mapo vašega skripta. V različici 1.2.0 jih je backend razrešil v svoji lastni namestitveni mapi, kar je povzročilo napako »Dostop je zavrnjen«.
> - **`numpy` je zdaj obvezna zahteva** SDK-ja (klici live-frame vrnejo numpy matrike).
> - **Ime na ravni izvoza, ki ni eno, se zavrne.** `processing=` in `levels=` sprejemata `raw`, `debayered`, `radiance`, `reflectance` ali `display` (ali psevdonim, kot je `preview`; `processing=` in tudi `"all"`). Vsako drugo ime se zavrne, še preden se dotaknete kamere (HTTP 400 `UNKNOWN_LEVEL`: `capture()` sproži `ChlorosConnectError`, `grab()` / `frame_stream()` `FrameFetchError`); različica 1.2.0 je za to shranila ali vrnila `raw`.
> - **Funkcija »Texture Aware« je zavrnjena za zajeme LATTICE.** V projektu, ki vsebuje zajeme LATTICE, `configure(debayer="Texture Aware (Slow, Highest Quality)")` sproži `ChlorosConfigurationError`, enako pa velja za `process()` po `configure()`, ki je za to pred uvozom. Vrednost »Texture Aware«, ki jo je projekt shranil prej, je nastavljena na »Standard«, s `UserWarning`. Posnetki LATTICE vedno dobijo standardno demosaiciranje, zato je različica 1.2.0 nastavitev sprejela in jo ignorirala; tudi aplikacija Chloros je za te posnetke ne ponuja. Glej [Podprte vrednosti](#supported-values).
> - **Kamere se vrnejo v stanje, v kakršnem so bile najdene — razen če je `keep_settings=True`.** `ArraySession.disconnect()` / `force_disconnect()`, konec vsake prehodne funkcije (`capture_cameras_transient()`, `start_multi_camera_stream()`, `test_array_sync()`, `calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index()`) ter zaustavitev `start_camera_continuous()` vsako kamero vrnejo v stanje, v kakršnem so jo našli: njeno ožičenje sprožilca in sinhronizacijskega vodnika, njen PTP ter vse nastavitve kamere, ki so bile spremenjene med sejo (osvetlitev in ojačanje, cilj avtomatske ekspozicije, omejitve in dušenje, belo ravnovesje, raven črne, format pikslov, združevanje pikslov in območje zanimanja, obrati, hitrost snemanja). `keep_settings=True` namesto tega shrani nastavitve vsake kamere v njen lasten pomnilnik, tako da se ob vklopu zažene z njimi; `clear_settings=True` pa ta pomnilnik ponastavi na tovarniške nastavitve. V obeh primerih kamere delujejo samostojno. Projekt nastavi kamere iz lastnih shranjenih nastavitev, ko se z njimi poveže. Glej [Pomnilnik kamere](#camera-memory--keep_settings--clear_settings).
> - **Vsak način shranjuje projekt na enak način.** Vsaka sprememba kamere, niza ali svetlobnega senzorja, priključenega v okviru projekta — iz SDK-ja, aplikacije, CLI-ja ali strežnika MCP — se shrani v `cameras.json` / `sensors.json`, odpiranje projekta v katerem koli načinu pa nastavi naprave iz njega; `ChlorosProject` poimenuje svojo mapo v ozadju. Prej je projekt shranjevala le aplikacija. Glej [En zapis projekta, vsak način](#one-project-record-every-mode).
> - **Datoteke, ustvarjene iz več kamer, navajajo vsako posamezno kamero.** Večpasovni niz, indeks v živo, sestavljen iz pasov več kamer, in slika zajema v obliki matrike `composite/` vsebujejo serijsko številko, model in kalibracijski paket vsake izvorne kamere v obliki seznamov (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), medtem ko EXIF `SerialNumber` navaja serijske številke, ločene z vejico. Prej niso vsebovale podatkov o kameri ali pa so vsebovale le serijsko številko prve kamere. Glej [Datoteke, ustvarjene z več kamerami](#files-made-from-several-cameras).
> - **Samo datoteka `raw` vsebuje kalibracijske oznake kamere.** Komplet, združljiv z MicaSense (`BlackLevel`, `RadiometricCalibration`,kanal `Band{R,G,B}RadiometricCalibration`, kompleti `MicaSenseVignetting*`) ter podatkovni del `CalibrationJSON` opisujejo surove vrednosti senzorja, zato so vključeni le v datoteko zajema `raw`. Vse druge datoteke — zajemne datoteke`debayered`, `display`, `radiance`, `reflectance` ter indeksne datoteke, okvirji neprekinjenega snemanja in večkamernihter vsak izdelek `process()` — vsebuje identifikacijsko številko kamere (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) ter pri zajemu tudi lastne podatke posameznega okvirja (`XMP-mapir:FrameJSON`: osvetlitev, ojačanje, temperatura, format pikslov, čas zajema). V različici 1.2.0 so vsebovali tudi niz surovih podatkov, ki ga je orodje za obdelavo (Pix4D, Metashape) uporabilo za piksle, ki niso surove vrednosti. Program za obdelavo najde kalibracijo takšne datoteke na podlagi serijske številke in `CalBundleSha256`.
>
> **Druge spremembe:**
>
> - **Samodejna hitrost snemanja niza se prilagodi temu, kar niz zagotavlja.** Hitrost, ki izgubi več kot 1 % svojih sprožilcev, se zniža – to se presoja na podlagi več daljših opazovanj namesto enega samega –, hitrost, s katere se je znižala, pa se občasno ponovno preizkusi. Hitrost, ki jo nastavite sami, ostane nespremenjena.
> - **Samodejna osvetlitev je stabilnejša.** Ohranja konstantno osvetlitev, se v nekaj sekundah prilagodi spremembam, prezre majhne svetle dele in ponovno vzpostavi delovanje kamere, ki v močnem soncu preneha odgovarjati. Glejte [Kako deluje samodejna osvetlitev](#how-auto-exposure-behaves).
> - **`import_images(recursive=True)` šteje le vaše slike in posnetke.** V mapi, ki vsebuje `camera_cal/` projekta, se te kalibracijske datoteke ne štejejo več kot slike in posnetki, projekt, ki jih je že vključil, pa jih odstrani.
> - **Velja `configure(radiometric_output=...)`.** Zdaj nastavi izdelke projekta `Export radiance` / `Export reflectance`; prej so imele prednost lastne nastavitve izdelkov vsakega projekta, tako da so `radiance`, `sensor-response` in `all` vsi izvozili sevanje in odbojnost.
> - **`test_array_sync()` se zažene iz stanja vklopljenih kamer.** Vklopi PTP kamer za izvedbo, počaka, da se njihove ure sinhronizirajo in ustalijo pred prvim sprožilcem (`ptp_lock`) — vsaka kamera se sinhronizira 120 s, približno 2 minuti od vklopa (`ptp_settle_s=` ga nastavi), saj je ura, ki se je pravkar uskladila, približno 2 minuti odstopa za do ~200 us — in vrne PTP vsake kamere v stanje, v kakršnem ga je našel. Na kamero, ki je še vedno sinhronizirana iz prejšnjega testa, se ne čaka ponovno. Prej je PTP ostal izklopljen v omrežju z enim stikalom, kjer ni odgovoril noben DAQ-E, in je prosto tekoči časovnik vsake kamere prebral kot PTP-časovnik, ki se ne bi sinhroniziral (»PTP ni sinhroniziran«, pri vsakem izvajanju).
> - **Zapis `.daq` navaja, ali je bil pokrovček izbran ali domnevno nameščen.** DAQ-U, DAQ-M in DAQ-E se dobavljajo brez pokrovčka, *Chloros* pa ne more ugotoviti, ali je ta nameščen, zato senzor, priključen brez imenovanega pokrovčka, privzeto dobi ime `sunshine_cosine`, kar je *predpostavka*. Vsak zapis *Chloros* (shema 1.30) zdaj poleg `cap_id` shranjuje tudi `cap_id_source`: `operator`, če ste pokrovčku dodelili ime (vključno z `none`), `auto_default`, če je bil privzet. Obdelava zapisa, pri katerem je bil kondenzator predpostavljen, pri čemer je nastavitev DAQ kondenzatorja v projektu ostala na »Auto«, izpiše `[daq-dls] WARNING` z imenom datoteke: goli DAQ-U brez korekcije sončnega svetloba kaže približno 30-krat višjo vrednost. **Poimenujte kondenzator, ki ste ga namestili** ob priključitvi (`connect_daq_sensor(cap_id=...)` ali `DAQSensorSession.set_cap()`, `chloros-cli daq pool-connect --cap-id` / `daq pool-set-cap` ali iz spustnega menija za kondenzatorje v grafičnem vmesniku), ali pa nastavite kondenzator DAQ za projekt, in opozorilo bo izginilo.
> - **V sistemu Windows se ne odpirajo več okna konzole pip.** Na računalniku z nameščenim Pythonom so backend in vsak od njegovih obdelovalcev slik ob zagonu enkrat zaželi `pip`, pri čemer je vsak odprl okno konzole. To se zdaj ne dogaja več.
> - **Snemanja `.daq` v realnem času preživijo izpad električne energije.**Snemanje se zapiše z dnevnikom in popolno sinhronizacijo diska ob vsakem potrditvenem koraku, za senzorjem, zato počasen disk upočasni le datoteko, nikoli pa odčitkov. Posnetek, ki se je ustavil sredi potrditve (izpad električne energije, prekinitev delovanja), lahko poleg sebe pusti `<name>.daq-journal`:**kopirajte oba skupaj.**Chloros ob prvem branju datoteke povrne dnevnik v prejšnje stanje, zato se posnetek odpre z vsem, kar je bilo potrjeno pred ustavitvijo (izgubljenih je približno zadnjih 3 s); v različici 1.2.0 bi izpad električne energije v napačnem trenutku lahko povzročil, da bi bila celotna datoteka neberljiva („slika diska baze podatkov je poškodovana“).
> - **`DAQSensorSession.record_start()` na zasedenem disku ne povzroča več zastoja senzorja**: odčitki se postavijo v čakalno vrsto in zapišejo v lastnem niti, tisti, ki ga ni bilo mogoče zapisati, pa se upošteva v `status()`-ovem `record_errors` (skupaj z `record_last_error`).
> - **Neposredni `LatticeCamera`:**okvir brez nastavitve osvetlitve/ojačitve zdaj poroča o**dejanski** osvetlitvi in ojačitvi kamere (rezervna rešitev je brala vrednost iz predpomnilnika, ki bi lahko ostala nespremenjena skozi celoten cikel avtomatske osvetlitve), `apply_settings(trigger_source='LineN')` / `set_trigger_source('LineN')` pa uporabljata isti 5 µs odglitš na vhodu, kot ga dobijo kabelski podrejeni senzorji Chloros (nefiltrirano, podrejeni senzor je posnel okvirje brez poslanega sprožilca).
> - **Kamere s programskim sprožilcem ne izpuščajo več vsakega drugega sprožilca.** Kamera v načinu programskega sprožilca zdaj postavi v čakalno vrsto sprožilec, ki prispe med njenim odčitavanjem (ali pa Chloros počaka, da je pripravljena), tako kot so to že počele kamere, sinhronizirane prek kabla.
> - **`discover_daq_sensors()` poimenuje DAQ-E-S `DAQ-E-S`.** Njegovo ime `model` je bilo `DAQ-E` za vsak ethernetni senzor; tisti, ki v približno 2 sekundah ne odgovori na preverjanje identitete, je še vedno naveden kot `DAQ-E`.
> - **Novo: `chloros-mcp`, strežnik MCP za AI-pomočnike.** Claude Desktop, Claude Code, Cursor in drugi MCP-odjemalci lahko obdelujejo slike (ustvarjanje projekta, uvoz posnetkov, izbira nastavitev, obdelava, poročanje o izvoženih podatkih) ter upravljajo kamere LATTICE, matrike in svetlobne senzorje DAQ: povezovanje, predogled, nastavitev osvetlitve in barv, zajem, snemanje. Njegova orodja pokrivajo funkcionalnost tega SDK-ja. Namesti se skupaj z SDK-jem, datoteka .deb za Linux pa ga namesti v `/usr/bin`. Glej [MCP strežnik (AI pomočniki)](#mcp-server-ai-assistants).
> - **Novo: pripomočki za kamere**, ukazi `chloros-cli lattice`, ki jih SDK pokliče: barvni profil, nasičenost / kontrast, svetli deli, bela (enkratno, avtomatsko, fiksna razmerja), prednastavitve, modeli filtrov, stanje in posodobitev firmware-a, napajanje (preverjanje, spanje, prebujanje), kalibracija cilja, omrežni adapterji (preverjanje, ocena, popravilo), popravljanje vinjetiranja na datotekah, odčitavanje DLS in preverjanje gostitelja. Glej [Pripomočki za kamero](#camera-utilities).
> - **Novo: preostali del CLI kot klici SDK**, tako da CLI, SDK in `chloros-mcp` ponujajo enake funkcije (CLI zaradi varnosti ohranja le prijavo, konfiguracijo DAQ-E in neposredne DAQ-strežnike): kamere, ki se odprejo z enim ukazom (`start_camera_continuous()`, `start_multi_camera_stream()`, `capture_cameras_transient()`, `test_array_sync()`, `detect_array_master()`, `analyze_array_models()`), datoteke profilov poravnave (`calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `alignment_profile_info()`, `reorder_alignment_profile()`), indeksi (`index_catalog()`, `compute_index()`, `capture_live_index()`), glavni PTP (`time_sync_status()` in podobni), projektna mapa (`project_folder()`, `set_project_folder()`, `reset_project_folder()`) ter recepti (`run_project_recipe()`, lastni izvajalnik CLI). Za svetlobne senzorje: `connect_daq_sensor(cap_id=...)`, `DAQSensorSession.set_cap()`, `daq_bindings()` / `set_daq_bindings()`, `daq_band_presets()` in `light_sensor_reflectance()` (odbojnost in indeksi okolja ter predmeta za dva senzorja); za kamero: `CameraSession.status()` in `capture_repeated()`; za matriko: `ArraySession.capability()` in `set_member_settings()` (osvetlitev, ojačitev člana matrike, ...). Glejte [Prehodne matrike, profili poravnave in indeksi](#transient-arrays-alignment-profiles--indices), [Projektna mapa in recepti](#project-folder--recipes) ter [Sejne datoteke senzorjev DAQ](#daq-sensor-sessions).
> - **Izboljšani indeksni produkti LATTICE** (NDVI in drugi indeksi) iz `process()`, v vseh načinih obdelave.
> - **OCN-kamere LATTICE: izboljšani izdelki sevanja, odbojnosti in indeksov v sončni svetlobi.** Popravek flote se uporabi po ločevanju pasov – v realnem času, v zajemih in v `process()`; RGN-kamere ostajajo nespremenjene. Glejte [Radiometrični izhod](#radiometric-output-lattice-multispectral-pipeline).
> - **Vsak posnetek shrani vrednost DAQ, ki jo je uporabil, tako kot to počne grafični vmesnik.** `CameraSession.capture()` in `ArraySession.capture()` / `continuous()` zapišejo `.daq` in njegov `.match.json`, kadarkoli je DAQ vezan ali združen v skupino, vključno z zajemom, ki vsebuje le surove podatke; v različici 1.2.0 je bil surovi ali debayerični zajem shranjen le z `force_daq=True`. `daq=False` to izklopi, `daq_tolerance_ms=` pa nastavi toleranco ujemanja; oba rezultata navajata, kaj je bilo shranjeno v `.daq_files` in `.daq_matches`. Zajem zdaj čaka na naslednje odčitavanje DAQ-ja, da lahko združi odčitke na obeh straneh osvetlitve. Glej [Odčitek DAQ-ja, ki ga zajem shrani](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
> - **`ArraySession` vsebuje posnetke v realnem času in neprekinjeno zajemanje na strani strežnika.** `grab()` in `frame_stream()` vrneta numpy okvirje iz enega sprožilca (enako branje kot pri projektnem API-ju `ArrayHandle.grab()` / `frame_stream()`), `continuous()` pa vsako sinhronizirano skupino shrani v ozadju in vrne [`ContinuousHandle`](#continuoushandle). Glejte [Živi okvirji v vašem lastnem potoku](#live-frames-in-your-own-pipeline).
> - **Niz `grab()` je pritrjen na en sprožilec** (`ArrayHandle.grab()` in novi `ArraySession.grab()`). V različici 1.2.0 je 12 od 20 štirihkamere `ArrayHandle.grab()` zajemov je zajemalo dva sprožilca. Vsak okvir zdaj vsebuje `group_id`, niz `frame_stream()` pa vsakemu sprožilcu pripada enkrat. Glej [En sprožilec na branje](#one-trigger-per-read--group_id).
> - **Okvirji po pasovih in poravnani okvirji:** `grab()` / `frame_stream()` prevzemata `band=`, `align=`, `bit_depth=` in `refl_max=` na ročajih kamere in matrike. Glej [Izvoz enega pasu](#exporting-a-single-band--band).
> - **Nove metode in argumenti `ArraySession`:** `set_target_fps()`, `force_disconnect()`, `capture(after_seq=..., orient=...)` (z `CaptureResult.group_seq` in `.failed`), `continuous()`, `connect_array(packed=...)`; `capture_repeated()` veriga zaporedne skupine in se ustavi ob polnem pogonu.
> - **`ArraySession.record()` beleži, ko ni odprt grafični vmesnik.** Ko je združen pogled grafičnega vmesnika zaprt, backend sam prikaže pogled:NDVI ali indeks, ki ga posredujete kot `expr=`. `RecorderHandle.stop()` sproži `ChlorosConnectError` za snemanje, ki ni uspelo, pri katerem ni bil zapisan noben okvir, in ohrani rezultat zaustavitve v `.result` (če pustite blok `with`, se izjemo ne sproži: preverite `rec.result`). Glejte [`RecorderHandle`](#recorderhandle).
> - **Nove metode `DAQSensorSession`:** `enable_ptp()`, `force_disconnect()` ter za lastne nastavitve naprave DAQ-E `device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` in `update_firmware()`. `record_stop()` vrne `warning` in `gaps`. Ponovni zagon, ki je zaprl snemanje, ga poimenuje v svojem odgovoru ali, če ponovni zagon ni uspel, v `ChlorosConnectError` (`recording_closed`; glej [Nastavitve naprave DAQ-E](#daq-e-device-settings)). Glejte [Metode `DAQSensorSession`](#daqsensorsession-methods).
> - **Združeni DAQ-E, ki preneha pošiljati podatke, se ponovno poveže prek strežnika** z nastavitvami, ki jih je imel, in snemanje na njem se nadaljuje v istem `.daq`, pri čemer je vrzel označena v datoteki; `record_stop()` vrne vrzeli kot `gaps`. V različici 1.2.0 je senzor ostal tiho, dokler ni bil odklopljen in ponovno priključen, kar je prekinilo snemanje. Glejte referenco CLI [DAQ-E, ki preneha pošiljati podatke](cli-reference.md#a-daq-e-that-stops-delivering).
> - **`open_project()` deluje na podlagi nameščenega SDK-ja.** Wheel (namestitveni programin PyPI) zdaj vsebuje potrebne nalagalnike projektov; prej je `open_project()` sprožil `ImportError`, ki je poimenoval `project_cameras` / `project_sensors` v katerem koli SDK-ju, nameščenem prek pip. `connect_all()` prek delujočega backenda (svetlobni senzorji prek njegovega DAQ-skladišča), shranjena naprava, ki je ne more poiskati, pa je navedena pod `errors` z navedbo razloga, namesto da bi bila brez pojasnila preskočena. Shranjen niz, ki je že aktiven v ozadju (v grafičnem vmesniku, ali v `chloros-cli project connect`) je priključen in ostane v teku, medtem ko je bilo prej to prikazano kot napaka (`Camera <sn> is already in array <id>`). Glej [`ChlorosProject` metode](#chlorosproject-methods).
> - **`ChlorosLocal()`, `process_folder()` in `process_lattice_capture()` poiščejo Chloros v sistemu Windows.** Iščeta, kam namestitveni program shrani backend (privzeto `<install folder>\backend\`, `C:\Program Files\Chloros\backend\`, v mapi po izbiri iz lastnega zapisa namestitvenega programa) ter se povežejo z backendom, ki že teče, ne da bi ga sploh iskali. V različici 1.2.0 so na nameščenem računalniku prikazali sporočilo »Chlorosov backend ni bil najden«, razen če ste posredovali `backend_exe=`.
> - **O backendu, ki se ustavi ob zagonu, se takoj poroča.** Ko `ChlorosLocal()` ali klic »smart-connect« zažene backend, ki se ustavi, preden odgovori, se v nekaj sekundah sproži napaka `ChlorosBackendError` iz `ChlorosLocal` in `ChlorosConnectError` iz klicov »smart--connect, skupaj z izhodno kodo, ukazom, ki ga je izvedel, in zadnjimi vrsticami, ki jih je backend izpisal. Različica 1.2.0 je počakala 60-sekundni časovni limit za zagon in ni navedla razloga. Glej [Samodejni zagon backenda](#backend-auto-start).
> - **Backend med skripti še naprej deluje.** Ko se blok `with ChlorosLocal(...)` zaključi, vrnitvijo `process_folder()` / `process_lattice_capture()` in izhodom `chloros-mcp` ne ustavi več backenda, ki ga je zagnal SDK; naslednji skript ga ponovno uporabi. Ustavijo se pri `logout()` (razen če je odprta aplikacija Chloros ali poteka izvajanje), pri `shutdown_backend()` ali samodejno po 30 minutah, če ga nič ne uporablja (`CHLOROS_BACKEND_IDLE_MIN` v okolju backenda to spremeni; `0`: nikoli).
> - **Backend, ki ga je zagnal SDK, je pozvan k zaustavitvi, ne pa ukinjen.** `shutdown_backend()` omogoči backendu, da sprosti svoje kamere in svetlobne senzorje ter zaključi morebitno snemanje; zaključi se le, če še vedno teče 10 sekund kasneje. Različica 1.2.0 ga je takoj zaključila (v sistemu Windows z nasilnim ukinitvijo), kar je lahko povzročilo, da svetlobni senzor DAQ, povezan s tem backendom, ni več odgovarjal. Backend, ki ga je SDK zaznal kot tekočega, se še vedno pusti pri miru.
> - **V sistemu Linux samodejni zagon nikoli ne zažene `backend_server.py` iz trenutne mape.** Če ni nameščenega backenda, ga je skript, zagnan v mapi, ki ga je vsebovala, zagnal; zdaj se zagnane le nameščene backende (ali, v primeru `ChlorosLocal`, tistega, ki ga imenuje `backend_exe=`).
> - **`configure(indices=...)`**doda vsako prednastavitev v projekt za pasove svoje kamere, vključi nov projekt in opozori (`UserWarning`) na ime, ki ga ne more dodati.**`import_images()`** poroča o posnetkih `.daq` kot `scan_count` in ne opozarja več na mapo za zajem LATTICE.
> - **`read_image_audit_tags()` deluje po namestitvi s pip.** Sam bere metapodatke slike; v različici 1.2.0 je vrnil `{}` brez notranjih datotek namestitve za namizje v poti Python.
> - **`capture()` takoj po `connect_array()`** počaka do 20 sekund na prvo skupino okvirjev niza, namesto da bi se pojavila napaka HTTP-503 »Niz se še ogreva«.
> - **`status()` poroča o zastoju.** Novo polje `stalled` je True, medtem ko teče zanka zajemanja, vendar ne objavi ničesar, `actual_fps` pa je nato 0. V različici 1.2.0 je `actual_fps` ohranilo svojo zadnjo vrednost med zastoja ohranil svojo zadnjo vrednost, zato je zastal niz prikazoval `actual_fps > 0`, medtem ko so zajemi spodleteli z napako 503 »Ni nove sinhronizirane skupine okvirjev«.
> - **Manj nepopolnih sklopov okvirjev in napak zajema 503:** sprožilni vhod vsake podrejene kamere je zdaj brez motenj.
> - **Pakirani 12-bitni prenos** (`connect_array(packed="auto")`) poveča hitrost snemanja celotnega niza senzorjev za približno tretjino. Glej [Pakirani 12-bitni prenos](#packed-12-bit-transport-packed).
> - **Izpostavljenost matrike v območju ROI:** majhno območje ROI ne omejuje več izpostavljenosti člana na obdobje prostega delovanja senzorja (4,9 ms pri 768 × 304). Glejte [Izpostavljenost na članih matrike](#exposure-on-array-members).
> - **Avtomatska osvetlitev ne znižuje več avtomatske hitrosti snemanja matričnega niza.** Matrika, sinhronizirana prek kabla, deluje s hitrostjo, ki jo omogočata omrežje in branje kamer; samodejna osvetlitev zapolni obdobje, ki ga ta hitrost omogoča, nato pa poveča ojačanje. Ročna osvetlitev, ki traja dlje od tega obdobja, upočasni sprožilec, da se prilagodi, namesto da bi bila zavrnjena. Glej [Osvetlitev na elementih matrike](#exposure-on-array-members).
> - **`read_nodes()`**doda `frame_values` in `notes` med delovanjem avtomatske osvetlitve (glej [`read_nodes()` med avtomatsko osvetlitvijo](#read_nodes-under-auto-exposure--frame_values-and-notes)). Ko kamera zavrne osvetlitev ali ojačanje,**`set_settings()`** navede območje, ki ga sprejme, in ponovno vklopi avtomatski način, ki ga je za to vrednost izklopil; `set_settings(target_brightness_auto=True)` je nov.
> - **`orient=`** na `ArraySession.capture()` / `capture_fastest()` / `continuous()` / `burst()` in `ArrayHandle.capture()` / `capture_smart()` obrača izdelke glede na usmeritev kamere. Glej [`ArraySession` metode](#arraysession-methods).
> - **`DLS()` potrebuje lastni `mac_address` senzorja.** Privzetega naslova MAC ni več (bil je naslov ene enote); `DLS()` brez njega sproži `DLSError`. Glejte [velikosti pasov DAQ-M](#daq-m-band-magnitudes--dlsband_magnitudes).
> - **Namestitveni program za Windows** ponuja možnosti »Omogoči dolge poti v Windowsu« in »Izključi backend Chloros iz skeniranja Microsoft Defender«, ki sta privzeto označeni. Glej [Korak 1](#step-1--install-the-chloros-platform-package).
> - **Linuxov .deb** namesti Pythonov SDK za sistem `python3` (prej je na disk namestil le wheel), poveča velikost sprejemnih pomnilnikov vtičnic jedra, ki jih potrebujejo tokovi kamer, ter doda storitev backenda, ki teče pod vašim uporabniškim računom. Paket za Jetson je `chloros_1.3.0_arm64_jp6.deb`, za JetPack 6, vključno z JetPackovim CUDA PyTorchom. Glejte [Korak 1](#step-1--install-the-chloros-platform-package) in [Korak 2](#linux-deb).

---

## Arhitektura v 60 sekundah

SDK je tanek Python-sloj nad Chloros-backendom (isti Flask-strežnik, ki ga uporabljata namizni GUI in CLI). Za avtomatizacijo uvozite `chloros_sdk` in pokličete metode visoke ravni; v ozadju vsak klic postane HTTP-zahteva do lokalnega backenda na vratih 5000 — `http://127.0.0.1:5000/api/...` (namerno ne `localhost`, ki se v sistemu Windows najprej razreši v `::1` in stane približno 2 s na zahtevo pri backendu, ki podpira le IPv4). Backend upravlja s sklopom strojne opreme — kamere, DAQ-senzorji, profili poravnave, okvirni pomnilniki — tako da lahko skripti SDK delujejo vzporedno z grafičnim uporabniškim vmesnikom, ne da bi se borili za serijske vmesnike ali pasovno širino omrežne kartice.

Na voljo so tri vmesniške ploskve, ki jih boste uporabljali:

1. **`ChlorosLocal` + proste funkcije** (`process_folder`, `process_lattice_capture`) – Potek obdelave slik. Z enim samim klicem v Pythonu izvedite kalibracijo, odstranjevanje Bayerjevega filtra in izvoz indeksov za celotno mapo.
2. **Ročaji Smart-connect** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Odprite trajno sejo v ozadju za delujočo strojno opremo. Enak potek »pametne priprave« kot v grafičnem vmesniku: preverjanje omrežja, samodejna izbira ravni, PTP, nastavitev AE, konfiguracija sprožilca GPIO.
3. **`ChlorosProject` / `open_project`** — Naloži shranjen projekt (mapa z datotekami `cameras.json` + `sensors.json` + `project.json`), hkrati poveže njegove kamere, matrike in svetlobne senzorje ter sproži zajeme z imenovanimi ročaji.

Površini 1 in 2 **samodejno zaženeta lokalni backend**, če ta še ne posluša (ista priložena izvedljiva datoteka, ki jo ustvarita GUI in CLI) — tako da skript deluje iz novega lupinskega okna, ne da bi morali najprej zagnati backend: `ChlorosLocal` (in `process_folder()` / `process_lattice_capture()`), `connect_camera()`, `connect_array()`, `connect_daq_sensor()`, `discover_lattice_cameras()` in `discover_daq_sensors()`. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` in `analyze_array_network()` se nikoli ne zaženejo: potrebujejo delujoč backend. Da se izključite, prenesite `auto_start_backend=False` (npr. ko kažejo na oddaljeni backend, ki se nikoli ne zažene). Glej [Samodejni zagon backenda](#backend-auto-start). Surface 3 se obnaša drugače: `open_project()` ne sprejema parametra `auto_start_backend`, `connect_all()` pa nikoli ne zažene ozadja — enkrat preveri `http://127.0.0.1:5000` in, če se nič ne odzove, preklopi na neposredno (brez backenda) `lattice_sdk` upravljanje naprave. Za to je potrebno izvajalsko okolje Arena SDK in OpenCV (`cv2`), ki ga je mogoče uvoziti v Python, kar pa kolo SDK ne namesti; brez njih se vsaka shranjena kamera in matrika prijavi kot napaka, skupaj z razlogom. Svetlobni senzorji projekta nimajo poti brez backenda: če backend ne teče, se vsak shranjen senzor prijavi kot napaka, skupaj z razlogom (glej [`SensorHandle`](#sensorhandle)). Le `proj.process()` in `stream(..., overlays=True)` leno ustvarjata `ChlorosLocal()` (ki se samodejno zažene).

Vsi trije so avt-odvisni od avtentifikacije: enkrat zaženite `chloros-cli login` na računalniku ali se prijavite prek namiznega grafičnega vmesnika. Kaj sproži klic brez veljavne seje, je odvisno od vmesnika (glej [Izjeme](#exceptions)): `ChlorosAuthenticationError` iz `ChlorosLocal` (in `process_folder()` / `process_lattice_capture()`), `ChlorosConnectError` iz pametnih-connect in njihovih sej, `status: "error"` z `error_code` iz `analyze_array_network()` ter vnos pod `errors` v poročilu `connect_all()`.

Zahteve:
- Python 3.7+ (kot navaja paket; razvit/preizkušen na 3.10)
- Lokalno nameščen Chloros Desktop (binarna datoteka za ozadje je vključena v namestitveni program)
- Aktivna prijava v Chloros+ na **kateri koli plačljivi stopnji Chloros+ (Copper ali višji)**; brezplačna stopnja**Iron**nima dostopa do SDK/CLI. To se uveljavlja**na strani strežnika**strani**: vsaka zahteva z oznako SDK/CLI mora vsebovati tako aktivno sejo kot plačani paket. Paket, nižji od Copper, prejme napako `403` skupaj z napako `error_code: PLAN_UPGRADE_REQUIRED` (napaka `ChlorosLicenseError` iz `ChlorosLocal`, `ChlorosConnectError` iz klicev smart-connect). Odjavljeni klicatelj namesto tega prejme `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError` iz `ChlorosLocal`, `ChlorosConnectError` iz klicev prek Smart Connect) — ti dve sta različni, ker ponovno izvajanje `chloros-cli login` odpravi napako `401`, ne more pa odpraviti napake `403`, za katero je potrebna nadgradnja načrta.
- Uporaba brez internetne povezave je podprta v okviru odloga plačila paketa: stopnja se prebere iz predpomnilnika za preverjanje strežnika (5 min) ali iz predpomnilnika podpisane, na napravo vezane licence (30 dni za mesečne pakete, do izteka naročnine za letne). Ko se to obdobje odloga izteče, se paket spremeni v brezplačnega, dostop do SDK in CLI pa se prekine, dokler računalnik ne vzpostavi povezave s strežnikom. Napaka `chloros-cli status` (`GET /api/license-status`) ostane dostopna v brezplačnem nivoju, zato je razlog viden.
- Windows 10/11 64-bit, **Ubuntu 22.04 LTS ali novejši**ali Jetson (JetPack 6). Ubuntu 20.04**ni** podprt: odvisnosti `.deb` izhajajo iz tega, s čimer se povezuje backend, vključno z `libc6 (>= 2.35)`, Focal pa vključuje glibc 2.31.

---

## Namestitev

Pythonov SDK je tanek Pythonov sloj nad backendom Chloros. Če backenda ne poganjate na drugem računalniku (glejte [Način oddaljenega backenda](#remote-backend-mode-pip-only-host-via-tunnel)), potrebujete **lokalno nameščen namizni paket Chloros** (namestitveni program za Windows ali Linux `.deb`), ki vključuje senzorje svetlobe DAQ. Paket vsebuje binarno datoteko backenda, izvedbeno okolje Arena SDK za kamere LATTICE ter kalibracijske pakete.

Najnovejše datoteke za prenos: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Korak 1 — Namestite paket platforme Chloros

#### Windows (.exe)

1. Prenesite `Chloros Installer 1.3.0.exe` s strani za prenos.
2. Zaženite namestitveni program in sledite navodilom čarovnika. Privzeta namestitvena pot je `C:\Program Files\Chloros\`.
3. Vsaj enkrat zaženite Chloros in se prijavite s svojim računom Chloros+.

Namestitveni program omogoča, da backend (`chloros-backend.exe`, nič drugega) skozi požarni zid Windowsa v vsakem omrežnem profilu: vhodni UDP za tokove kamer, PTP, mDNS in tok DAQ-E ter vhodni TCP samo iz lokalnega podomrežja za povezavo, ki jo DAQ-E vzpostavi nazaj z računalnikom med posodobitvijo strojne programske opreme ([`update_firmware()`](#daq-e-device-settings)). Odstranjevalnik obeh pravil odstrani obe pravili. HTTP-API strežnika ostane v vsakem primeru na naslovu `127.0.0.1`.

Skupina **Sistemski nastavitve** v čarovniku ima dve možnosti, ki sta privzeto obe izbrani:

| Možnost | Kaj počne | Izključitev tihe namestitve |
| --- | --- | --- |
| **Omogoči dolge poti v sistemu Windows** | Omogoča, da so projektne mape globlje od omejitve 260 znakov za poti v sistemu Windows. Če so dolge poti izklopljene (privzeta nastavitev sistema Windows), se projektna mapa, ki je pregloboka za svoje izdelke, zavrne, še preden se karkoli obdela — glej [Odpravljanje težav](#troubleshooting-pointers). | `/MERGETASKS="!longpaths"` |
| **Izključi backend Chlorosa iz skeniranja Microsoft Defenderja** | Prepreči, da bi Defender v realnemskeniranje vsake datoteke, ki jo zapiše zajem kamere. Pri visokih hitrostih zajema skeniranje zavzame približno 30 % hitrosti: merjeno pri ROI 512 na nizu 9 kamer je neprekinjen zajem surovih podatkov potekal s hitrostjo 52,7 zajemov/s s skeniranjem in 77,7 zajemov/s z izključenim backendom. | `/MERGETASKS="!defenderexclusion"` |

Neprekinjeno zajemanje poroča o obremenitvi programa Defender kot `defender_cores` (glej [`ContinuousHandle`](#continuoushandle)).

#### Linux amd64 (.deb)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Namestite z `apt install ./…`, ne z `dpkg -i`: apt namesti tudi zahteve Python SDK, ki jih paket priporoča, ne pa zahteva (glej [Korak 2](#linux-deb)).

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Za Jetson Orin / Orin NX / Orin Nano na JetPack 6 (L4T R36). Njegov PyTorch je CUDA-različica za JetPack 6, zato obdelava na grafičnem procesorju poteka na grafičnem procesorju Orina; uporablja knjižnici CUDA in cuDNN, ki ju namesti JetPack 6 (`sudo apt install nvidia-jetpack` na sliki, ki ju nima).

#### Linux: sprejemni pomnilniki in strežniška storitev

Oba .deb datoteka povečata sprejemne pomnilnike vtičnic jedra, ki jih potrebujejo tokovi kamer LATTICE, na 32 MB (`/etc/sysctl.d/60-chloros-gige.conf`). Pri privzeti nastavitvi jedra so paketi kamere v jedru zavrnjeni, zaradi česar niz izgubi posnetke in hitrost snemanja; `chloros-cli lattice network` poroča o stanju zalog, ko so te nizke. Glejte referenco CLI [Linux gostitelji — zaloge za sprejem v vtičnicah](cli-reference.md#linux-hosts--socket-receive-buffers).

Na računalniku brez grafičnega vmesnika zaženite backend kot storitev **pod svojim uporabniškim imenom**, tako da skript najde tistega, ki že teče (`connect_all()` nikoli ne zažene backenda):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
```

Backend nato teče kot vi: zajeti podatki, ki jih zapiše, so vaši, uporablja pa vaš `chloros-cli login`. Rootov `chloros-backend.service` je še vedno nameščen za računalnike, ki so ga omogočili, vendar vsaka datoteka, ki jo zapiše rootov backend, pripada rootu. Glejte referenco za CLI, [Zagon backenda kot storitve](cli-reference.md#running-the-backend-as-a-service-headless).

Backend, ki ga izvaja storitev, je skupen in ga odjemalec nikoli ne ustavi: skripti SDK in `chloros-cli` ga uporabljajo takšnega, kot ga najdejo (`ChlorosLocal.shutdown_backend()` ustavi le backend, ki ga je sam zagnal `ChlorosLocal`), `GET /api/status` pa enoto poimenuje kot `service`. Ponovno ga zaženite z `sudo systemctl restart chloros-backend@$USER.service`. Nadgradnja paketa ustavi storitve backenda med namestitvijo, zato tekoči skript izgubi povezavo s kamerami in senzorji, nato pa ponovno zažene vse omogočene; odstranitev paketa jih ustavi in onemogoči tiste, ki so bile omogočene. Hkrati teče le en backend, zato omogočite le eno enoto: druga se ne zažene in ostane v stanju `failed` (izhodni status 98, ni se ponovno zagnal), razlog pa je naveden v `systemctl status`. Če želite preklopiti med enotami, onemogočite tisto, ki je ne uporabljate.

### Korak 2 — Namestitev Pythonovega SDK-ja

**Namestitveni program Chloros vsebuje ustrezen SDK-kolesec.** Vsak namestitveni program za Windows in .deb datoteka za Linux na disk namesti datoteko `chloros_sdk-X.Y.Z-py3-none-any.whl`, ki se natančno ujema z različico grafičnega vmesnika (GUI), ukazne vrstice (CLI) in ozadja (backend). Ni vam treba spremljati PyPI, da bi ostali sinhronizirani.

#### Windows

Namestitveni program samodejno zažene `pip install` s priloženim paketom »wheel« ob uporabi vašega sistemskega Pythona (prednostno se uporabi zaganjalnik `py.exe`, v nasprotnem primeru pa `python -m pip`). Ni potrebno nobeno ukrepanje — `import chloros_sdk` deluje v vašem Pythonovem okolju po uspešni namestitvi. Če na računalniku ni Pythona, namestitveni program tiho preskoči ta korak, GUI in CLI pa še naprej delujeta.

#### Linux (.deb)

Oba .deb paketa (x86_64 in Jetson) namestita SDK za sistem `python3`, zato `import chloros_sdk` deluje po namestitvi, brez pip in brez omrežja. Paketi (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` in nalagalniki projektov) se namestijo v `/usr/lib/python3/dist-packages`, ukaz SDK-ja `chloros-mcp` ([MCP strežnik](#mcp-server-ai-assistants)) pa v `/usr/bin`, vsi pa so v lasti paketa: nadgradnja jih nadomesti, odstranitev paketa pa jih odstrani.

Lastne zahteve SDK-ja, `python3-numpy` in `python3-requests`, so **priporočene, niso pa obvezne** (grafični vmesnik, CLI in backend so samostojni): `sudo apt install ./chloros_<version>_amd64.deb` (`…_arm64_jp6.deb` na Jetsonu) jih namesti, `dpkg -i` pa ne. Izpis namestitve prikazuje različico SDK, ki jo uvozi `python3`, morebitne manjkajoče zahteve, ter o `chloros_sdk` drugje v poti `python3`, ki prekriva kopijo paketa (na primer tisto, ki je bila prej nameščena z `pip install --user`).

V virtualnem okolju (virtualenv), ki ne vidi sistemskih paketov, vanj namestite priloženo kolesce:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Pri namestitvah Jetsona v izoliranem omrežju (air-gapped) poteka vse popolnoma brez povezave – SDK in kolesce sta že na disku.

#### Javni PyPI

Za gostitelje, ki uporabljajo izključno pip (ni nameščen namizni paket Chloros; SDK takrat poganja backend na drugem računalniku, glej [Način oddaljenega backenda](#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

PyPI vsebuje najnovejšo stabilno različico. Poskrbite, da je SDKja v isti različici kot backend Chloros, ki ga poganja (glej [Različice in združljivost](#versioning--compatibility)). Pomožni program DAQ-M `DLS` potrebuje tudi `bleak`: `pip install "chloros-sdk[camera]"` (ali `pip install bleak`).

#### Preverjanje

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Potrebna je naročnina na Chloros+.** Vsi klici SDK zahtevajo aktivno prijavo v Chloros+. Na vsakem računalniku enkrat zaženite `chloros-cli login user@example.com 'YourPassword'`; prijavni podatki se shranijo v `~/.chloros/`.

### Ali potrebujem namizni paket?

Samo paket pip za večino delovnih tokov **ni** dovolj. Tukaj je navedeno, kaj potrebuje vsaka površina SDK:

| Površina SDK | Potrebuje namizni paket? | Zakaj |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Da**(za zagon)**/ Ne** (če je strežnik že dosegljiv) | Uporabi že zagnan strežnik (strežnik namizne aplikacije ali drug zagnan skript); sicer samodejno zaženi nameščenega: `/usr/lib/chloros/chloros-backend` (Linux) ali `<install folder>\backend\chloros-backend.exe` (za Windows; privzeto `C:\Program Files\Chloros\backend\`, namestitvena mapa po meri se prebere iz zapisa namestitvenega programa). Posredujte `backend_exe=`, da ga poimenujete sami. |
| `chloros-mcp` ([MCP strežnik](#mcp-server-ai-assistants)) | **Da**(za zagon)**/ Ne** (zadnji del je že dosegljiv) | Zgrajen na podlagi `ChlorosLocal`, torej enako kot v vrstici zgoraj. |
| `read_image_audit_tags` | **Ne** | Prebere metapodatke obdelane slike (samo standardna knjižnica Python). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Da**(lokalno)**/ Ne**(oddaljeno) | Čisti HTTP-odjemalci prek ozadja. Lokalno ozadje → potreben namizni paket. Oddaljeno ozadje → `backend_url=`**prek tunela** (glej Način oddaljenega ozadja — priložena ozadja se vežejo le na zanko). |
| `ChlorosProject` / `open_project` | **Da** | Poganja shranjene projekte prek ozadja. |
| Neposredni razredi LATTICE (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Da** | Potrebujete izvedbeno okolje Arena SDK, ki je vključeno v namizni paket, ter OpenCV (`cv2`) v vašem Pythonu, ki ga niti namizni paket niti SDK ne namestita (`pip install opencv-python`). `CAMERA_AVAILABLE` je sicer ob uvozu enak `False`. `DLS` prav tako potrebuje `bleak`. |

### Način oddaljenega strežnika (gostitelj samo za pip, prek tunela)

> **Namestjeni strežnik ni dosegljiv prek lokalnega omrežja.** Posluša
> samo na loopbacku (`127.0.0.1` / `::1`), zato `backend_url="http://<lan-ip>:5000"`
> **ne more doseči nameščenega Chlorosa**. Če želite upravljati backend na drugem
> računalniku, sami preusmerite njegovo vrata za zanko in usmerite SDK na
> tunel:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"     # the tunnel endpoint
serials = ["213800234", "214000533"]   # master first

# auto_start_backend=False: the tunnel URL looks local, so without it a tunnel
# that is down would start this machine's own backend, if one is installed.
arr = chloros_sdk.connect_array(serials, backend_url=BACKEND,
                                auto_start_backend=False)
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                     backend_url=BACKEND,
                                     auto_start_backend=False)

# The backend opens every path on ITS machine: pass absolute paths valid there.
# (A relative path would be made absolute against this machine's working folder.)
arr.capture("/home/chloros/captures/run1", processing="raw")
```

Gostitelji brez zaslona / CI / robotika lahko en računalnik z namestitvijo polnega namizja ohranijo kot »Chloros strežnik«, drugje pa uporabljajo `pip install chloros-sdk` — vendar je prenos med njimi zgoraj navedeni tunel, ki ga uredi uporabnik, in ne neposredna LAN URL-naslov.

> **`ChlorosLocal` na gostitelju, ki podpira le pip.** `ChlorosLocal(backend_url=BACKEND)` se poveže z backendom, do katerega lahko dostopa brez namestitve namizja: lokalno binarno datoteko backenda poišče le, kadar ga mora ZAŽENITI (če se nihče ne odzove in se pojavi `auto_start_backend=True`). Do različice 1.2.0 je najprej iskal binarno datoteko in na gostitelju, ki podpira le pip, sprožil napako `ChlorosBackendError` („Chlorosov backend ni bil najden…“), čeprav je bil backend dosegljiv. Mape, ki jih posredujete (`process_folder`, `import_images`), se na tem računalniku preverijo in jih prebere backend, zato morajo pri oddaljenem backendu imeti enako ime na obeh straneh, na primer na skupnem disku.

### Delovni tok, namenjen izključno DAQ (gostitelj, namenjen izključno za pip)

`pip install chloros-sdk` sam ne more upravljati senzorja DAQ: senzor DAQ se vedno upravlja prek backenda Chlorosa s programom `connect_daq_sensor()`:

- **Senzor na tem računalniku:** tukaj namestite namizni paket Chloros. `connect_daq_sensor()` zažene svoj backend, če ta še ne teče.
- **Senzor na drugem računalniku:** ta računalnik potrebuje namizni paket; do njegovega backenda dostopajte prek tunela v [načinu oddaljenega backenda](#remote-backend-mode-pip-only-host-via-tunnel). Backend odpre senzor na svojem računalniku, tako da je DAQ-U priključen na ta računalnik, DAQ-M pa je v dosegu Bluetootha tega računalnika, DAQ-E pa je v omrežju, do katerega ima dostop.

```python
import chloros_sdk

# http://127.0.0.1:5000 is the local desktop install's backend, or the tunnel.
with chloros_sdk.connect_daq_sensor() as daq:   # smart-detect USB / BLE / ETH
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

V obeh primerih backend uveljavlja načrt »Chloros+«: prijavite se na računalniku, na katerem teče backend (`chloros-cli login` ali namizna aplikacija), s katerim koli plačljivim paketom (Copper ali višjim). Izpisani backend in paket, nižji od Copper, oba sprožita `ChlorosConnectError` z navedbo razloga backenda (`AUTH_REQUIRED` / `PLAN_UPGRADE_REQUIRED`).

---

## Hitri začetek

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Indeks API-jev najvišje ravni

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by CameraSession.capture / ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job
chloros_sdk.ContinuousHandle                      # handle for an array continuous() capture
chloros_sdk.GrabResult                            # dict returned by grab() / yielded by frame_stream()
chloros_sdk.FrameFetchError                       # raised by grab()/frame_stream() on a refusal

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()
chloros_sdk.daq_bindings() / set_daq_bindings(cameras=, arrays=)   # which DAQ feeds which camera
chloros_sdk.daq_band_presets(filter_code=None)    # band sets for Ambient + Object reflectance
chloros_sdk.light_sensor_reflectance(ambient_id, object_id, ...)   # two DAQs -> reflectance + indices
chloros_sdk.time_sync_status() / time_sync_peers() / time_sync_cameras()   # PTP grandmaster
chloros_sdk.restart_time_sync() / set_time_sync_priority(priority1=, priority2=)

# === Camera utilities (the CLI's lattice commands; by serial) ===
chloros_sdk.camera_color_profile(serial) / set_camera_color_profile(serial, ...)
chloros_sdk.camera_color_adjust(serial) / set_camera_color_adjust(serial, ...)
chloros_sdk.camera_highlights(serial) / set_camera_highlights(serial, value)
chloros_sdk.camera_white_balance_once(serial) / set_camera_white_balance(serial, ...)
chloros_sdk.camera_presets() / apply_camera_preset(serial, preset)
chloros_sdk.camera_filter_models() / camera_filter_model_info(name)
chloros_sdk.camera_firmware_status() / update_camera_firmware(serial)
chloros_sdk.camera_power_probe(serial) / sleep_camera(serial) / wake_camera(serial)
chloros_sdk.calibrate_camera_with_target(serial, ...)
chloros_sdk.network_adapters(...) / fix_network_settings()
chloros_sdk.apply_vignette_correction(input_dir, output_dir, ...)
chloros_sdk.read_dls(mac, ...) / probe_system(...) / system_info()

# === Transient cameras, alignment profiles, indices (cameras opened for one command) ===
chloros_sdk.start_camera_continuous(serial, ..., daq=None, daq_tolerance_ms=None)  # → CliJob; a DAQ bound to the camera records one .daq per frame (daq=False: none)
chloros_sdk.start_multi_camera_stream(serials, ...)   # → CliJob
chloros_sdk.capture_cameras_transient(serials, ...)
chloros_sdk.test_array_sync(serials, count, ptp_settle_s=None) / detect_array_master(line) / analyze_array_models(models)
chloros_sdk.calibrate_alignment_profile(serials, save_path, ...)
chloros_sdk.capture_aligned_multiband(profile_path, ...) / start_aligned_stream(profile_path, ...)  # → dict / CliJob
chloros_sdk.alignment_profile_info(path) / reorder_alignment_profile(path, ...)
chloros_sdk.index_catalog() / compute_index(input_path, ...) / capture_live_index(profile_path, ...)
chloros_sdk.CliJob                                # stats() / stop() / wait()

# === Project lifecycle ===
chloros_sdk.open_project(path, relink=None)       # → ChlorosProject
chloros_sdk.project_folder() / set_project_folder(path) / reset_project_folder()
chloros_sdk.run_project_recipe(recipe, project_path)  # `chloros-cli project run`
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics               # DLS: DAQ-M over Bluetooth; needs bleak

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosProjectFilesMissingError      # a ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True when chloros_sdk.project imported; open_project()
                                 # still raises ImportError without the project loaders
```

---

## Obdelava slik — `ChlorosLocal`

Glavni razred poteka obdelave. Ob ustvarjanju zažene ozadje (če se nobeno ne odziva in `auto_start_backend=True`), ustvari/konfigurira projekte, spremlja napredek in vrne povzetke po zaključku izvajanja.

### Konstruktor

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Metode

| Metoda | Opis |
| --- | --- |
| `create_project(project_name, camera=None)` | Ustvari nov projekt (po želji s predlogo kamere, kot je `"Survey3N_RGN"`). |
| `open_project(project_path, relink=None)` | Odpre shranjen projekt (njegovo mapo ali ime v mapi projektov) in ga določi kot odprt projekt. Poti prestavljenega projekta se preusmerijo na mapo, v kateri se projekt zdaj nahaja; `relink` poimenuje mapo, v kateri so zdaj ostale manjkajoče datoteke. Vrne odgovor ozadja z `source_files`; sproži `ChlorosProjectFilesMissingError`, če jih ni mogoče najti. Glejte **Premaknjen projekt** spodaj. |
| `import_images(folder_path, recursive=False)` | Uvoz slike v formatih RAW/TIF/JPG/DNG **in `.daq` posnetke svetlobnega senzorja**. Vrne `count` (slike) in `scan_count` (zapisi). Opozori le, če mapa ne vsebuje niti enega niti drugega. |
| `export_light_sensor(daq=True, csv=True)` | Zapiše kalibrirane `.daq` + `.csv` za vsak posnetek svetlobnega senzorja v projektu, v `<project>/Light Sensor/`. Glej [Posnetki svetlobnega senzorja](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Nastavi gumbe za obdelavo. `indices` so imena prednastavitev (`NDVI`, `GNDVI`, `OSAVI`, …), ki so dodana v projekt za pasove posamezne kamere; ime, ki ni prednastavitev ali ki ga pasovi ne morejo tvoriti (`NDRE` potrebuje RE in NIR, česar nobena kamera sama ne zagotavlja), dobi ime `UserWarning` in nima indeksne slike. `daq_log_path` poimenuje posnetek svetlobnega senzorja (datoteko `.daq` ali `.csv` ali mapo z njimi), s katero se primerja vsak posnetek, tako v LATTICE kot v Survey3, namesto posnetkov, uvoženih skupaj s slikami; če pot manjka ali ne vsebuje nobenega posnetka, se uporabijo uvoženi posnetki. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Zaženi potek obdelave. Vrne `{"status": "complete", "async": False}`, ter ključ `summary`, če ga zagotovi backend — glej [Povzetek in namigi po izvedbi](#post-run-summary--hints) — `export_check_failure`, če izvozov izvajanja ni bilo mogoče preveriti, in `image_failure`, če je izgubljena slika. `wait=False` vrne `{"status": "started", "async": True}` naenkrat. |
| `get_config()` / `get_status()` / `status()` | Preverite stanje ozadja. |
| `logout()` | Odjava: izbriše shranjene poverilnice iz predpomnilnika in sprosti režo za napravo tega računalnika (v rezultatu se prikaže `device_deregistration`). Tekoči obdelovalni cikel se ne ustavi: zaključi se v načinu, v katerem se je začel (`run_in_progress: true` in `run_mode` v rezultatu), naslednji cikel pa uporabi standardno obdelavo. |
| `shutdown_backend()` | Ustavi backend, če ga je ta instanca zagnala; backend, ki ga najde v teku, se pusti pri miru. Če ostane blok `with` ali če se instanca odstrani s pomočjo zbiralnika smeti, se backend ne ustavi: backend še naprej deluje za naslednji skript in se sam ustavi po 30 minutah, če ga nič ne uporablja. Backend je pozvan, naj se zaustavi, zato najprej sprosti svoje kamere in svetlobne senzorje ter konča morebitno snemanje; ukonči se le, če 10 sekund kasneje še vedno deluje. |
| `discover_cameras()` | Odkrij kamere LATTICE **prek backenda te instance** (`/api/camera/discover`). Vrne seznam slovarjev (`serial`, `model`, `ip`, …) — v isti obliki, kot jo vidita GUI in CLI. Seznam je prazen, če ni najdenih kamer ali če je backend nedosegljiv. |
| `camera_capture(output_dir, format="tiff", **settings)` | Posname en sam okvir**prek backenda**(samodejni zagon prek tega ročaja), tako da dobi enako pripravo kot GUI/CLI (privzeto 12-bitno, ponovna uporaba sklada, vgrajeni kalibracijski metapodatki). Cilj določite z `serial=` ali `device_index=`; posredujte `exposure`/`gain`/`pixel_format`/`preset` kot `**settings`. Vrne slovar metapodatkov stare različice (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Pridobi predogledne okvirje, sestavljene s prekrivanjem, iz združene kamere — tanek MJPEG-odjemalec prek poti `/api/camera/<serial>/stream-annotated` v ozadju (zebra / mreža / križec / histogram / peaking / točka, narisana na strežniški strani). `decode=True` generira BGR-nize in potrebuje OpenCV (`cv2`), ki ga SDK ne namesti; `False` generira surove JPEG-bajte. Dostopen je tudi na ravni projekta kot `ChlorosProject.stream(overlays=True)`. |

**Tekoči izvedbeni proces ohrani svoj projekt.** Medtem ko poteka obdelava — tega skripta, aplikacije, `chloros-cli process` ali `chloros-mcp` — backend ne dovoli ustvarjanja, odpiranja, uvoza v projekt ali spreminjanja nastavitev obdelave projekta (HTTP 409 `PROCESSING_IN_PROGRESS`): `create_project()` in `import_images()` sprožita `ChlorosProcessingError`, `configure()` pa sproži `ChlorosConfigurationError`, `process()` z načinom, ki se razlikuje od trenutno aktivnega, pa sproži `ChlorosProcessingError` (izvajanje ohrani način, v katerem se je začelo), vsak s sporočilom backenda. Posnetek, shranjen v projekt, se nato pridruži projektu, ko se izvajanje konča (tako navaja odgovor `project_add`). Počakajte, da se izvajanje konča (`get_status()`), nato nadaljujte.**Premaknjen projekt.** `project.json` projekta poimenuje vsako izvorno datoteko (vsako sliko in posnetek svetlobnega senzorja) s polno potjo, zato jih po premestitvi projektne mape – ali celotne mape »projekti« na drug pogon – še vedno poimenuje po stari lokaciji. Vsaka odprta datoteka sama popravi, kar lahko: datoteka, ki je bila v mapi projekta, se poišče v mapi, v kateri se projekt nahaja zdaj, vsaka najdena pot pa se preusmeri in shrani (`project.json` se najprej varnostno kopira kot `project.json.before-relink-<date>-<time>`). Pot se vedno preusmeri le na datoteko, ki se tam dejansko nahaja. Datoteke, ki so bile zunaj projektne mape – seansa v središču, uvoz z »Dodaj mapo« – se najdejo le v mapi `relink=<the folder they are in now>` (ali kateri koli mapi nad njo). `open_project()` vrne `source_files`: `total`, `found`, `missing`, `missing_roots` (mape, v katerih so bile manjkajoče datoteke), `missing_outside_project` in `relink` (kaj je bilo preusmerjeno, od kod in kam). Projekt, v katerem manjkajo nekatere datoteke, se odpre z ostalimi; tisti, v katerem ni bilo najdenih nobenih datotek, se ne odpre, `open_project()` pa sproži napako `ChlorosProjectFilesMissingError` (`.missing_roots`, `.audit`; HTTP 409 `PROJECT_SOURCES_MISSING`). `project open --relink` in `chloros-mcp` v CLI`open_project` (`relink_folder`) delujejo enako.

Uporabite kot upravitelja konteksta za zagotovljeno čiščenje:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Zapisi svetlobnega senzorja — kalibrirani `.daq` + `.csv`

DAQ-U / DAQ-M / DAQ-E je mogoče snemati **brez** njegovega kalibracijskega paketa. To je
tisto, kar javni [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) počnejo privzeto: zapisujejo surove vrednosti senzorjev in označijo
datoteko, tako da Chloros najprej pridobi tovarniško kalibracijo senzorja **po serijski številki** — najprej iz lokalnega predpomnilnika, nato pa iz oblaka MAPIR — in jo uporabi ob uvozu.

Chloros rezultat zapiše nazaj kot dva izdelka na posnetek, pod
`<project>/Light Sensor/`:

| Izdelek | Kaj je |
| --- | --- |
| `<name>_calibrated.daq` | Arhiv, ki ga je mogoče ponovno obdelati – ista shema kot pri posnetku v realnem času, zdaj pa navaja paket, ki ga je ustvaril. Ponovni uvoz ga **ne** kalibrira še enkrat. |
| `<name>_calibrated.csv` | Spektralna obsevanost v W/m²/nm na lastni valovni mreži senzorja, ena vrstica na odčitek, skupaj s fotometričnimi stolpci (skupna moč, fotopični/skotopični luks, PPFD in njegova razdelitev na modro/zeleno/rdeče, vrhunska valovna dolžina). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Samo senzorji brez paketa (DAQ-A).** Surovi spektralni odčitki senzorja — *ne* sevanje. Glej spodaj. |

`<name>` je ime datoteke posnetka. Dva posnetka v različnih mapah si lahko delita
eno — odčitke na posnetek (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`), shranjene v
isti milisekundi z dvema kamerama — in vsaka še vedno dobi svoje izdelke: prva ohrani
`<name>`, naslednji pa doda mape, ki ga ločujejo, **za** imenom
(`<name>__<capture>_daq_calibrated.daq`), tako da datum in čas posnetka še vedno stojita na prvem mestu.

`process()` izvede ta izvoz kot eno od svojih faz. Za to **ni** potrebna slika:
samostojno letenje svetlobnega senzorja je prvovrsten delovni tok, takšen projekt pa po svoji zasnovi nima
nobene slike.

**Zapisi DAQ-A se izvozijo kot surove številčne vrednosti.** Družina DAQ-A je starejša od sistema
paketov po serijskih številkah in nima paketa, ki bi ga bilo treba pridobiti – namesto tega se kalibrira na terenu glede na
reflektančni cilj, zato ga nikoli ni potrebovala. Ti zapisi se izvozijo
pod korenom `_raw` namesto `_calibrated`: gre za drugačno ime datoteke namesto zastavice
znotraj datoteke, saj mora oznaka ostati ohranjena tudi pri pošiljanju po e-pošti kot golo ime. V
Zgornji del datoteke `.csv` navaja `raw spectral sensor counts (NOT irradiance)` in opozarja, da so
vrednosti primerljive **znotraj** datoteke – točno to, za kar se uporablja kalibracija na podlagi cilja
za to — in ne med senzorji. Fotometrični stolpci, odvisni od moči (skupna moč,
fotopični/skotopični luks, PPFD), se vrnejo kot **NULL** namesto da bi bili integrirani iz števila impulzov.

DAQ-U / DAQ-M / DAQ-E, katerega sveženj preprosto ni bilo mogoče pridobiti, se še vedno **preskoči**,
ne zapiše v surovi obliki: v tem primeru sveženj obstaja in »ponovna povezava in ponovna obdelava« je resničen nasvet.

Stari posnetki **v1.01 / v1.02** (te piše DAQ-A-SD) ne vsebujejo epohe za vsak odčitek,
temveč le čas pisanja datoteke. Program za ujemanje slike↔navzdolnega toka jih še vedno zavrača – ujemanje
posameznega okvirja z časom zapisa bi bilo nevidno napačno — vendar jih izvoznik prebere,
datoteka CSV pa natisne `clock=daq_created_on`, tako da izdelek navaja, na kateri uri deluje.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Zapis, katerega kalibracijskega paketa ni mogoče pridobiti (offline ali senzor brez
kalibracije v datoteki), se poroča pod `skipped` **z navedbo razloga**. Nikoli se ne
zapiše kot »kalibrirana« datoteka z neobdelanimi števili – povežite se z internetom in
ponovno zaženite postopek, da se izvoz zaključi.

### Povratni klici o napredku

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Povzetek po zaključku in nasveti

Ob zaključku `process()` pridobi `GET /api/processing-summary` in priloži vsebino kot `result["summary"]`. Pridobivanje poteka po načelu »najboljšega možnega prizadevanja« in nikoli ne blokira uspešnega vrnitve — če povzetek ni na voljo, se `process()` vrne na osnovno obliko `{"status": "complete", "async": False}`. Vsak vnos v `summary["hints"]` — celotni stavki s predlaganimi popravki, npr. zakaj je izvedba dala ničelni izhod — se prav tako ponovno izpiše kot Pythonov `UserWarning`, tako da so izvedbe z ničelnim izhodom samodiagnostične, tudi če slovarja nikoli ne pregledate:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` je strojno berljiva polovica:

| Ključ | Kaj šteje |
| --- | --- |
| `models` | Skupine kamer v izvedbi. |
| `images_in_groups` | Izvorne slike v teh skupinah. |
| `targets_found` | Zaznane tarče odbojnosti. |
| `images_calibrated` | Slike, za katere je izvedba ustvarila produkt odbojnosti. |
| `images_failed` | Slike, pri katerih je prišlo do napake in niso bile izvožene. |
| `exported_files` | **Datoteke slikovnih produktov , ki jih je ustvaril tek**: vsak produkt (debayering, predogled, sevanje, odbojnost, indeks, upodabljanje ciljev) v vseh formatih. |
| `reflectance_files` | Med njimi so tudi produkti odbojnosti (`Reflectance_Calibrated_Images`), ne glede na to, ali so bili kalibrirani s ciljem ali z obsevanostjo svetlobnega senzorja (DAQ). Izvedba brez cilja, ki je kljub temu zapisala odbojnost, to navede v namigu; le izvedba, ki ni zapisala ničesar, dobi obvestilo, da manjkajo izhodi odbojnosti. |
| `clamp_masks_written` | [Maske omejitve razmešavanja](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on), zapisane poleg njih — so spremljevalci, ne pa izdelki, zato niso v `exported_files` (CLI jih šteje ločeno na enak način). |
| `captures_left_out` | Surovi okvirji v mapi `captures` projekta, ki jih ta izvedba ni zajela. Zagon ob zagonu čaka na to, kar je bilo zajeto pred njim, zato je ta vrednost 0, razen če je bil med zagonom shranjen posnetek ali če se je čas čakanja iztekel. `summary["captures_left_out_note"]` navaja, kaj storiti z njimi. |
| `daq_recordings_exported` / `daq_recordings_skipped` | Zapisi svetlobnih senzorjev, namerno šteti ločeno — izhajajo iz druge faze in obstajajo tudi za izvajanja brez kakršnih koli slik, zato bi njihovo vključevanje povzročilo, da bi se izvajanje, namenjeno izključno zajemanju podatkov (DAQ), kazalo, kot da je izvozilo slike. |

Poleg njih: `summary["output_dirs"]` (vsak imenik, v katerega se je zapisovalo),
`summary["light_sensor_export"]`, `summary["stopped"]` (velja, če je uporabnik prekinil
izvedbo, tako da se delni izračuni ne berejo kot zaključen izvedbeni cikel z nezadostno proizvodnjo) ter
`summary["groups"]` (razčlenitev po skupinah).

`exported_files` se v cevovodu zabeleži **med zapisovanjem**, ne pa naknadno s
skeniranjem slikovnih objektov projekta. Strategiji vzporednega izvajanja in GPU ustvarjata svoje lastne slikovne
objekte (v delavniških podprocesih za poti GPU), zato je staro skeniranje za vsak tak izvajanje poročalo o
`0 file(s) written` za vsak tak izvedbeni cikel in nato izdal namig o ničelnih izvozih – pri izvedbenih ciklih,
kjer je vse delovalo. Če pišete skripte na podlagi te številke, zdaj uspešen vzporedni izvedbeni cikel
poroča o številu, ki ni nič.

Preskoki svetlobnega senzorja poročajo o razlogu, ki ga je bralnik dejansko ugotovil za vsako datoteko –
neberljiva shema, manjkajoči sveženj, napaka pri pisanju – **brez podvajanja**, tako da se dvajset datotek,
preskočenih zaradi enega vzroka, obravnava kot en vzrok in ne kot dvajset ponovitev istega vzroka.

> **`process()` se ne sproži, če izvedba ne ustvari nobenih slik.** To je eno od dveh mest, kjer se SDK in
> CLI namerno razlikujeta: `chloros-cli process` obravnava »zahtevani so bili izdelki, vendar ni bil
> zapisan noben« kot napako in se zaključi z vrednostjo, ki ni nič, medtem ko se SDK normalno vrne in poroča o
> stanje prek `summary` / namigov. Če naj se vaš potek ob praznem izvajanju ustavi, to preverite
> sami – preglejte `summary` (ali preštejte datoteke v projektni mapi), namesto da se zanašate na
> odsotnost izjeme. Običajni vzroki so vhodna mapa, ki ni bila prepoznana kot
> zajem, in izdelki, ki so bili preskočeni, ker niso primerni za prisotne kamere (npr. sevanje iz kamer, ki snemajo le v RGB
> formatu).
>
> `process()` se ne sproži niti takrat, ko izvozov iz izvajanja ni bilo mogoče preveriti: izda opozorilo in nastavi
> `result["export_check_failure"]`. Produkti so zapisani. Če želite proces ustaviti ob tem, preverite ta ključ. Enako velja za izvedbo, ki
> je izgubila sliko: izda opozorilo in nastavi `result["image_failure"]`, pri čemer `chloros-cli process`
> zaključi z vrednostjo, ki ni nič.

### Funkcije za lažjo uporabo

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "GNDVI", "OSAVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# process_folder() / process_lattice_capture() also take input_level=,
# array_alignment=, array_alignment_crop=, array_alignment_interpolation=,
# progress_callback=, poll_interval=, timeout=, processing_stuck_timeout=,
# backend_exe= and api_url= (NOT backend_url=). indices defaults to ["NDVI"].

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

Vsak teče znotraj svojega `with ChlorosLocal(...)`; backend, ki ga je moral zagnati, še naprej teče, ko se vrne, za naslednji klic, in se samodejno ustavi po 30 minutah, če ga nihče ne uporablja. Nobena od njiju ne sprejema `auto_start_backend`, `radiometric_output` ali `daq_log_path`; za te uporabite `ChlorosLocal` in `configure()`. `process_lattice_capture()` ne sprejema `camera`, `debayer` ali `ppk`: preskoči zaznavanje plošče in uporabi standardni debayer.

### Podprte vrednosti

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"TIFF (8-bit)"            # all-8-bit images are written as this; never raised
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer for Survey3 imagery, Chloros+ only.
                                         # Survey3 only: on a project holding LATTICE captures
                                         # (which always use the standard demosaic) configure()
                                         # or process() raises ChlorosConfigurationError; a
                                         # Texture Aware value the project saved earlier is set
                                         # to Standard with a UserWarning, as the app does. Needs a
                                         # CUDA GPU the backend's PyTorch can use (an NVIDIA
                                         # GPU, or a Jetson Orin's). Without one, process()
                                         # uses the standard debayer, and the backend prints
                                         # one "[DEBAYER] ... needs a CUDA GPU" line saying why.
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as Chloros's saved debayered (3-channel) product
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Radiometrični izhod (multispektralni procesni tok LATTICE)

Raven izvoza LATTICE multispektralnega (M3C/M3M) procesa `process` — `reflectance` (privzeto), `radiance`, `sensor-response` ali `all` (vsak ustrezen način za vsako sliko) — določa **»Izvoz sevanja«**in**»Izvoz odbojnosti«**: `radiance` izvozi samo sevanje, `reflectance` samo odbojnost, `sensor-response` pa nobenega (samo izdelki brez bayerjevega filtra in predogledni izdelki), `all` pa oba. Produktov brez Bayerjevega filtra in predoglednih produktov na tej ravni ni mogoče spreminjati. `configure()` ima za to namensko ključno besedo:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Napredna rešitev – pisanje produktov prek `custom_settings` – še vedno deluje, vendar ne pozabite, da nadomesti celoten blok nastavitev (glejte opozorilo spodaj). Vnesite ključe `"Export <product>"`: lastne nastavitve projekta `"Export radiance"` / `"Export reflectance"` prevladajo nad starejšim ključem `"Radiometric output"`, zato ta ključ sam po sebi ničesar ne spremeni.

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Export radiance": True, "Export reflectance": False},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (privzeto) deli sevanje kamere z **časovno usklajenim navzdolnim tokom DAQ**, ki se samodejno izračuna iz posnetega `.daq` (DAQ-U/M/E)**ali izvirnega `.csv`**sistema DAQ-M, najdenega skupaj s slikovnim gradivom; vsak paket kalibracij za posamezno kamero ali DAQ, ki lokalno manjka, se ob prvi uporabi**samodejno prenese iz AWS**. CLI to prikazuje v skladu spreklopnikov tipa izdelka na `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.**Čigava je ta vrednost?**Vrednost, ki jo shrani zajem `.daq`, je vrednost kamere(-), ki jo opisuje — imenovane v spremnem datoteki `<file>.daq.cameras.json` poleg nje (posnetki posameznih, sinhroniziranih in nizov; surovi zaporedni posnetki imenujejo odčitke vsakega člana v svojem manifestu) — in se ujemajo**samo s slikami iz teh kamer**. Samostojna kamera brez lastnega odčitka zato ne more več uporabljati DAQ-ja druge kamere; ji dodelite DAQ ali posnemite enega. Lastni posnetek DAQ-ja (`chloros-cli daq pool-record` ali posnetek, začet v grafičnem vmesniku) ali kateri koli `.daq` brez tega stranskega datoteka (vključno s starejšimi zajemi) še vedno služi vsem kameram.**Kamere OCN na soncu.** Vsak kalibracijski paket OCN je sestavljen v ozkopasovni svetlobi, z eno LED-diodo na pas, medtem ko sončna svetloba meša pasove. Chloros 1.3.0 po ločitvi pasov uporabi hitro korekcijo na podlagi sončne svetlobe, kar izboljša OCN-ove produkte sevanja, odbojnosti in indeksa v sončni svetlobi. Deluje povsod, kjer poteka ločevanje pasov — `process()`, posnetki, `grab()` / `frame_stream()` in v živo — za pakete OCN, ki to podpirajo; kamere RGN ostajajo nespremenjene. OCN-slike, zajete z zgodnejšo različico, to prav tako dobijo, če se obdelujejo na računalniku, ki hrani kalibracijo, s katero so bile posnete (Chloros ohrani kalibracijo kamere, ko je ta enkrat priključena); v nasprotnem primeru se obdelujejo kot prej. Vrednosti sevanja, odbojnosti in indeksa OCN se zato razlikujejo od tistih v različici 1.2.0.

> `custom_settings` **nadomesti** celoten blok izračunanih nastavitev (po zasnovi preskoči druge ključne besede in preverjanje veljavnosti `configure()`). Ko ga uporabljate, vključite vse ključne besede `Project Settings`, ki so vam pomembne, kot v zgornjem primeru.

#### Branje indeksnih pikslov (indeks NDVI in drugi vegetacijski indeksi)

**Celoštevilčni indeksni raster predstavlja raztezanje kontrasta za posamezno sliko, ne pa zemljevida v območju od −1 do 1.** Indeksni izdelki `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` in `JPG (8-bit)` indeksni izdelki — vključno z 8-bitnim TIFFom, v katerem je zapisana 8-bitna zajemna slika — shranjujejo

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

kjer sta `IndexMin` / `IndexMax` **najnižja in najvišja končna indeksna vrednost tega okvirja**. DN 0 ni −1, polna skala pa ni +1.

Program Chloros označi obe skrajni točki, da ostane preslikava obratna:

| Oznaka | Pomen |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — konvencija spodaj |
| `Chloros:IndexMin` | indeksna vrednost, shranjena kot DN 0 |
| `Chloros:IndexMax` | indeksna vrednost, shranjena kot polna lestvica |

Preberite jih in obrnite, pri čemer najprej normalizirajte shranjeni DN na 0…1 glede na največjo vrednost shranjenega podatkovnega tipa, tako da en izraz pokriva vse celoštevilčne oblike:

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

Za branje ni potreben `-config pix4d.config` — exiftool te lastnosti XMP bere samodejno. Konfiguracija je potrebna le za njihovo *pisanje*.

> **`IndexMin == IndexMax`** pomeni, da je vsak piksel tega okvirja imel enako indeksno vrednost. Raster je zapolnjen na srednji ravni in ni ničesar, s čimer bi se delilo — upoštevajte končno točko.

> **Če ni oznak `Chloros:Index*`, to pomeni, da ni navedenega preslikavanja.** Izvoz LUT je obarvan render, katerega piksli so barvni prehodi, vezani na vaše pragovno okno LUT, in okvir brez piksla z določenim indeksom nima obsega, ki bi ga bilo mogoče navesti. V nobenem od primerov ne predpostavljajte preslikave — velja isto pravilo kot pri manjkajočem `PixelScale`.**Ker je raztezek določen za vsako sliko posebej, dva okvirja istega poleta nista primerljiva glede na DN**— vsak ima svoje končne točke, zato isti NDVI v različnih slikah pade na različne vrednosti DN. Za kvantitativno delo v okviru enega poleta (statistike po conah, fiksni pragi, časovne vrste) izvozite**`TIFF (32-bit, Percent)`**: indeksni rasterji so tam zapisani kot float32**same indeksne vrednosti**, brez potrebe po ponovnem merjenju ali oznakah. Kljub imenu formata indeksni produkti niso ×100 – to merjenje velja le za odbojnost s celoštevilskim podatkovnim tipom.

#### Maska omejitve ločevanja – na katerih pikslovih je bil pas omejen

Spektralno ločevanje 3×3 lahko povzroči negativno vrednost pasu – kadar svetloba komaj vsebuje ta pas ali kadar prizor ne sodi v območje, ki ga opisuje matrika ločevanja kamere. To ni dejanska vrednost, zato jo izvoz fiksira na 0 — in v živo prikazani pogled kaže isto vrednost 0. Pas se lahko omeji na nekaj pikslov ali na skoraj vse; maska zabeleži oboje. `Chloros:Band{R,G,B}UnmixClampFraction` navaja, na kolikem delu okvirja se je to zgodilo; maska omejitve pa navaja **katere piksle**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— v mapi `clamp_masks/`**poleg** mape izdelka, nikoli znotraj nje (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`) — za vsak produkt sevanja, odbojnosti in indeksa, v katerem je bil omejen vsaj en piksel. Enopasovni `uint8`, na lastni pikselni mreži izdelka (vključno z usmeritvijo, ROI, združevanje in popravki poravnave matrike vključeni). Vsaka maska navaja kamero svojega izdelka (`Make` / `Model` / serijska številka), čas zajema in oznake objektiva ter navaja pasove, ki jih predstavljajo njeni biti (`XMP-Camera:BandName`, najprej bit 0); oznaki `ImageDescription` in `Software` jo označujeta kot masko z omejitvijo, kalibracije pa ne vsebuje.
- **Biti**: bit 0 = pas v R-kanalu izdelka (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Nastavljen bit pomeni, da je bila vrednost piksla v tem pasu omejena na 0 — obravnavajte ga in vsak indeks, zgrajen na njegovi podlagi, kot nizko zanesljivega.
- **`Chloros:UnmixClampMask`** je vtisnjen na vsak izdelek, na katerem je bil izveden postopek ločevanja: pot do maske glede na mapo samega izdelka (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, vedno ločena z `/`- ločena s presledkom — združite jo s mapo slike, da odprete masko), ali `none`, če ni bil omejen noben piksel. Če oznaka ni navedena, to pomeni, da na izdelku postopek ločevanja ni bil nikoli izveden (kamere RGB ali mono, surove/debayered ravni).
- Rezultati zajema to vsebujejo kot **`clamp_mask`** okvirja (pot do maske ali `None`, če ni bilo omejeno nič) — v rezultatih `CameraSession.capture()` so to okvirji `ArraySession.capture()`slikah `CaptureResult` in odzivih zajema prek HTTP. Offline zgradba navaja svoje maske kot `clamp_masks`.
- Večpasovni sklad živega indeksa (`/api/camera/cli-array-index` z `save_multiband`) dobi eno masko na izvorno kamero, v lastni mapi izhodnih datotek `clamp_masks/` — glej referenco CLI'`lattice index`. Video nima maske: rezultat offline izgradnje namesto tega poroča o deležu omejitve za vsak posamezni okvir.

Ker nobena mapa izdelka na nobeni ravni ne vsebuje maske, je mogoče mapo izdelka v celoti – vključno s podmapami – predati programski opremi za fotogrametrijo, ki sprejema le slike. Predajte ji mapo izdelka, ne pa mape nad njo. Chlorosov lastni uvoz preskoči maske.

```python
import os, numpy as np, tifffile

mask = tifffile.imread(
    "clamp_masks/radiance/capture_<ts>_SN<serial>-<filter>_radiance_clampmask.tif")
# or, from the image: os.path.join(os.path.dirname(image_path), tag), with
# tag = the image's Chloros:UnmixClampMask
r_clamped = (mask & 1) != 0      # the band Chloros:BandRName names
g_clamped = (mask & 2) != 0
b_clamped = (mask & 4) != 0
```

---

## Smart-Connect za kamere LATTICE

Trajne seje v ozadju za delovanje v realnem času. Uporabljajo se iste končne točke kot v grafičnem vmesniku, zato je delovanje enako v SDK-ju, CLI-ju in grafičnem vmesniku.

### Ena kamera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

Tukaj nastavljena osvetlitev, ojačanje in cilj se ohranijo kot vaši: Chlorosov avtomatski nadzor osvetlitve (vključno z osvetlitvijo, izmerjeno z DAQ) jih pusti pri miru. `cam.set_settings(exposure_auto="Continuous")` (ali `gain_auto=`) vrne osvetlitev (ojačanje), `cam.set_settings(target_brightness_auto=True)` pa vrne cilj avtomatskemu cilju programa Chloros.

#### `connect_camera()` Podpis

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

- **Vrne se, ko se tok začne.** Backend čaka, omejeno (približno 15 s, da se tok začne, nato 10 s za prvi popoln buffer – ne za prvi predogledni okvir, ki sledi približno 1,5 s kasneje), tako da se `status()` pojavi takoj po tem, ko je navedeno `streaming`. `connect_stream` seje je blok odgovora `stream`: `state` `streaming`, `no_frames` (tok se je začel, a ni posredoval ničesar — to navaja `note`, za kamero z jumbo paketi pa se je treba ponovno povezati s paketno velikostjo 1500 bajtov), `starting` ali `start_failed`, pri čemer je `waited_s`. `None` za kamero, ki je bila že povezana. Predogled (`get_camera_preview`, `/api/camera/jpeg/<serial>`), zahtevan takoj po čakanju, omejen (4 s), na prvi predogledni okvir kamere namesto odgovora »Še ni na voljo nobenega okvirja«; kamera, katere tok še ni posredoval ničesar (`no_frames` ali pripravljena za sprožilec strojne opreme), še vedno odgovori takoj.
- **Velikost paketa lahko določite sami.** `settings={"gev_packet_size": 1500, "gev_packet_size_auto": False}` odpre kamero s paketi velikosti 1500 bajtov, brez preizkušanja (`chloros-cli lattice cam-connect --packet-size standard` pošlje enako).
- **V sklopu kamer je hkrati mogoča le ena povezava**: odprtje, preverjanje velikosti paketa, prvi prenos kalibracijskih podatkov (približno 70 s za kamero, ki je na tem računalniku nova), začetek prenosa. Zahteva, ki medtem potrebuje kamero (`read_nodes`, `set_settings`, zajem, druga povezava) nanjo čaka največ 5 s, nato pa se pojavi napaka HTTP 503 `CAMERA_POOL_BUSY`, ki navaja kamero, ki se povezuje (`ChlorosConnectError`); vprašaj poskusite ponovno čez nekaj trenutkov. Seznami nikoli ne čakajo: `list_cameras()` (`GET /api/camera/pool`), `list_arrays()` in `CameraSession.status()` odgovorijo takoj. Kamera, ki je v procesu povezovanja, je navedena pod odgovorom skupine `connecting` (`serial`, `phase` — `opening the camera`, `rebalancing the pool bandwidth`, `loading its calibration`, `starting its stream`, `waiting for its stream` — ter `elapsed_s`), pri čemer `status()` navaja `connecting: True` z njegovim `phase`; ostale kamere se berejo v realnem času, če je sklop prost, sicer se odgovarja na podlagi njihovega zadnjega odčitka (`live: False`).

#### Metode `CameraSession`

| Metoda | Opis |
| --- | --- |
| `status(timeout=15.0)` | Trenutno stanje kamere, kot ga odčita `chloros-cli lattice status`: `streaming`, `stream_state` (`starting`, `no_frames`, `streaming`, `start_failed` ali `stopped`), `device` (model, serijska številka, firmware, temperatura, `is_mono`) ter njegove trenutne nastavitve. |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Prebere vozlišča GenICam; vrne `{nodes, errors, enums, device, was_in_pool, in_array_id}`, poleg tega pa še `frame_values` in `notes` med izvajanjem samodejne osvetlitve (glej spodaj). |
| `set_settings(**kwargs)` | Zapiše vozlišča po prijaznem imenu (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `target_brightness_auto`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). `exposure_time` (`gain`) je ročna osvetlitev (ojačanje): če je zapisana brez `exposure_auto` (`gain_auto`), je treba najprej izklopiti avtomatski način, da vrednost ne bo zavrnjena zaradi delujočega avtomatskega načina. Rezultat `applied` vsebuje vrednost, ki jo je zajela kamera; zapis, ki ga kamera zavrne, je v `errors` in se ne ohrani. Napaka navaja območje, ki ga kamera trenutno sprejema (največja vrednost `ExposureTime` sledi hitrosti snemanja kamere), avtomatski način, ki je bil izklopljen le za zavrnjeno vrednost, pa se ponovno vklopi; `restored` ga navaja (npr. `{"ExposureAuto": "Continuous"}`). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, daq=None, daq_tolerance_ms=None, settings=None, timeout=None)` | Posname **en sam** sliko. Vrne `CaptureResult`: seznam slovarjev metapodatkov slik (po enega na shranjeno raven), z `.skipped`, `.failed`, `.daq_files` in `.daq_matches`. (Snemanje v seriji/več posnetkov je bilo odstranjeno — če potrebujete serijo, pokličite `capture()` v zanki.) |
| `capture_repeated(output_dir="output", *, count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Zaporedno (`interval_s=0`) ali intervalno zajemanje s to kamero, za zajeme `count` in/ali `duration_s` (vsaj enega, da se zaključi), tako kot to počne `ArraySession.capture_repeated()` za niz. Vrne seznam `CaptureResult`-ov; ustavi se po zajemu, ki poroča o polnem disku. `on_capture(i, result)` se izvede po vsakem; druge ključne besede gredo v `capture()`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Sprostitev iz skladišča, kamera se vrne v stanje, v kakršnem je bila ob povezavi (oživitev sprožilca, PTP, nastavitve); `keep_settings=True` namesto tega shrani nastavitve v lastni pomnilnik, `clear_settings=True` pa ta pomnilnik ponastavi (glej [Pomnilnik kamere](#camera-memory--keep_settings--clear_settings)); vrne `camera_memory`, ko je zahtevano katero koli od obeh. Kamera, ki je članica niza, sprosti celoten niz. Ni učinka, če smo se priključili na že odprto sejo. Vrne se, ko backend potrdi sprostitev ali ko ni ničesar za sprostiti (kamera je že odstranjena ali ni nobenega delujočega backenda); sproščanje, pri katerem je potekel časovni limit ali ga je backend zavrnil, sproži napako `ChlorosConnectError`, saj je kamera morda še vedno zasedena in bi pri naslednji povezavi prišlo do kolizije z njo. Izhod iz bloka `with` nikoli ne sproži napake. |

Atributi: `serial`, `device` (zapis o odkritju kamere) in `backend_url`.

`capture()` nadzor izvoza (isti model kot niz + grafični vmesnik):

- `processing` / `levels` — `processing="all"` shrani vse ustrezne vrste izvoza; `levels=["raw","radiance"]` shrani le tiste (prepisuje `processing`). Oba izpustite za privzeto nastavitev v ozadju.
- `daq` — odčitek DAQ-ja, ki je bil uporabljen za zajem, se privzeto shrani kot stranski datotek `.daq`, vključno z zajemom v surovi obliki, tako da se okvir lahko kasneje ponovno obdela v odbojnost/indeks. `daq=False` ga preskoči, če ga nobena raven ne uporablja. `force_daq=True` je starejša pisava privzete vrednosti. Brez DAQ-ja, ni `.daq`.
- `daq_tolerance_ms` — koliko se lahko odčitek oddalji od osvetlitve in še vedno ostane v toleranci (privzeto 500). Spremeni se le ocena.

#### `read_nodes()` pri avtomatski osvetlitvi — `frame_values` in `notes`

Medtem ko teče `ExposureAuto` (`GainAuto`), vozlišče `ExposureTime` (`Gain`) odčita **zadnjo zapisano vrednost**, ne pa trenutnega stanja kamere. `nodes` še vedno ohranja to vrednost vozlišča. Ko zahtevate `ExposureTime` ali `Gain` na priključeni kameri, katere avtomatski način ni `Off`, odgovor vsebuje:

- `frame_values` — vrednost, ki jo je prenašal najnovejši okvir kamere, pod imenom vozlišča (`ExposureTime` v µs, `Gain` v dB);
- `notes` — en stavek na tak vozlišče, ki navaja obe vrednosti.

```python
r = cam.read_nodes(["ExposureTime", "Gain"])
print(r["nodes"]["ExposureTime"])                  # last value written, e.g. 2000.448
print(r.get("frame_values", {}).get("ExposureTime"))  # what the newest frame used, e.g. 19600.0
for name, note in r.get("notes", {}).items():
    print(note)
# ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448);
# the camera's newest frame was at 19600 us.
```

Oba ključa sta odsotna, kadar je avtomatski način `Off` (vozlišče je takrat dejanska vrednost kameredejanska vrednost kamere) in za kamero, ki ni povezana z backendom. Pri nizu `ArraySession.status()["member_ae"]` hkrati poda vrednosti okvirja za vsakega člana.

#### Zgornja meja osvetlitve, merjena z DAQ (HTTP)

Če je priključen svetlobni senzor DAQ, Chloros omeji osvetlitev × ojačitev kamere na podlagi izmerjene dnevne svetlobe, tako da se bela referenčna plošča ne preosvetli. Zgornja meja velja le, dokler je kalibracijski cilj v vidnem polju te kamere (viden v njenem kadru v zadnjih 10 sekundah); če cilj ni v vidnem polju, namesto tega deluje običajna avtomatska osvetlitev kamere z lastno zaščito pred preosvetljenimi deli, tako da preostali del prizora ni temnejši, kot je potrebno. SDK-ov ovitek ni na voljo; nastavite ali spremenite to vrednost za vsako kamero posebej s kodi `GET` / `POST /api/camera/<serial>/daq-exposure-cap` (vsebina zahtevka POST s kodo `{"enabled": false}` to funkcijo za zadevno kamero izklopi). Odgovor vsebuje `enabled`, `cap_us` (zgornja meja v realnem času, `None`, ko je funkcija neaktivna, vključno s primerom, ko ni v vidnem polju nobenega kalibracijskega cilja), `k_exp_g` (konstanta, s katero se izračuna zgornja meja) ter:

| Polje | Pomen |
| --- | --- |
| `k_exp_g_source` | Katera konstanta se uporablja: `camera` — lastna konstanta te kamere, izračunana iz laboratorijske točke v njenem kalibracijskem paketu (kamere RGB); `fleet` — konstanta flote, ki se uporablja, kadar paket nima laboratorijske točke; `override` — prepis polja, nastavljen na strežniku. |
| `k_exp_g_fleet` | Konstanta flote, za primerjavo. |

### Sinhronizirani niz — `ArraySession` (Smart-Prep)

`connect_array` je **priporočena izhodiščna točka** za konfiguracije z več kamerami. V ozadju izvaja celoten potek Smart-Prep prek grafičnega uporabniškega vmesnika:

1. **Analiza omrežja**(`/api/camera/array/recommend`) — nastavi**celoten senzor** (brez združevanja pikslov, brez izreza ROI) in omogoči, da se hitrost snemanja prilagodi zmogljivostim gostitelja in omrežja, pri čemer se upoštevajo predvideni formati pikslov in način pakiranja, ki ga bo uporabljal Connect. Slike se nikoli ne pomanjšajo, da bi ohranili visoko hitrost: nastavite manjši `width` / `height` ali `binning=2` / `4`, da zamenjate ločljivost za hitrost snemanja (končna točka`alternatives` navaja te možnosti s predvidenimi fps).
2. **Samodejna izbira stopnje** — `sim-capture-sim-emit` (vsaka kamera se osvetli ob istem sprožilcu, oddaja je usklajena za souporabo vodnika, hitrost snemanja pa pada, ko se souporaba zmanjša) v katerem koli omrežju, v katerem je mogoče kamere uskladiti za souporabo; `slip-emit-and-capture` (kamere se na povezavi izmenjujejo, ne posnamejo pa hkrati) le, kadar to ni mogoče.
3. **PTP omogočen**privzeto — časovni žigi med kamerami se uskladijo z eno skupno uro z natančnostjo**~1 ms**, [PTP grandmaster](#ptp-grandmaster--time_sync_status) v ozadju. Sočasna osvetlitev izhaja iz strojne sprožilne funkcije M8 (**< 100 µs** med moduli), ne iz PTP: PTP usklajuje *časovne oznake*, ne pa osvetlitev.
4. **Samodejna izbira formata pikslov za vsako kamero** — črno-belo → `Mono12`, barvno → `BayerRG12`.
5. **Zagon avtomatske osvetlitve (AE)** — posname trenutno stanje avtomatske osvetlitve vsake kamere, da povezava med delovanjem ne ponastavi osvetlitve.
6. **Konfiguracija sprožilca GPIO** — `connect_array` aktivira vsako kamero (`TriggerMode=On`, `TriggerSource=Line2`), tako da impulz glavne kamereimpulz prek kabla M8 poganja podrejene kamere. To je korak, ki velja le za niz: posamezna kamera, odprta s `LatticeCamera`, namesto tega deluje v prostem načinu. Ob sprostitvi niza (`disconnect()` / `force_disconnect()`, `chloros-cli lattice array-disconnect`) ponastavi vse kamere v samostojno delovanje (`TriggerMode=Off`), pri čemer ostane vse ostalo takšno, kot ga je zaznal program Connect: nastavitve sprožilca in sinhronizacijskega signala (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`; sinhronizacijski vod glavne kamere nazaj na vhod), PTP ter nastavitve kamere, ki so se med sejo spremenile – razen če `keep_settings=True` / `clear_settings=True` (glej [Pomnilnik kamere](#camera-memory--keep_settings--clear_settings)).

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

Atributi: `array_id`, `master_serial`, `slave_serials`, `ptp_enabled`, `sync_mode`, `packed_transport` in `packed_reason` (zadnji štirje so `None` na ročaju iz `attach_array`), `connect_stream` in `backend_url`. `connect_stream` je blok `stream` odgovora na povezavo, oblika posamezne povezave plus `members` (`{serial: {state, why?}}`): `streaming`, ko prispe prva sinhronizirana skupina niza (zadnji del sistema na to čaka, omejeno na približno 10 s), ali `no_frames`, pri čemer je zadnja neuspešna pridobitev vsakega člana njegov `why`, za jumbo pakete pa rezervno vrednost 1500 bajtov v `note`. `None` na ročaju `attach_array`.

#### Podpis `connect_array()`

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # pin the master trigger rate (auto if None); see set_target_fps
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    packed="auto",                        # packed 12-bit transport: "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=600.0,                        # s; also the backend's deadline (see below)
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`timeout` je čas, kolikor dolgo klic čaka na povezavo, in se pošlje v
zadnji del sistema kot rok. Če povezava do takrat ni vzpostavljena, se prekliče: vsaka kamera, ki jo je
odprl, se vrne v stanje, v katerem je bila najdena, in se zapre, noben niz ni povezan, klic pa
povzroči napako `ChlorosConnectError` (HTTP 504 `CLIENT_DEADLINE_PASSED`, pri čemer je naveden
korak, do katerega je povezava dosegla). Zato klic, ki je sprožil napako, nikoli ne pusti niza
za seboj. Niz 9 kamer se poveže v približno eni minuti; takoj po
vklopu kamer lahko to traja nekaj minut. Niz, ki je aktiven, a še nima
sinhronizirane skupine okvirjev, se vrne z napako `connect_stream["state"]`
`no_frames`. Dnevnik strežnika vsebuje eno vrstico s časom za vsako stopnjo povezovanja
(`[array-connect <id>] <step>: <s> s …`) in zadnjo vrstico, ki navaja najpočasnejšo.

Vrednosti `force_tier`:
- `"sim-capture-sim-emit"` — resnično sočasno (vse kamere se sprožijo na istem robu takta).
- `"sim-capture-ftd-stagger"` — prilagodljivo časovno zamikanje (kamere oddajajo v rahlo zamaknjenih časih, tako da se paketi na vodniku zaporedno prenašajo).
- `"slip-emit-and-capture"` — zaporedno zajemanje po posamezni kameri (brez časovne sinhronizacije). Izbere se samodejno le, kadar kamer ni mogoče dovolj upočasniti, da bi si omrežni vod hkrati delile.

`wire_ceiling_mbps` prevlada nad **trajnim proračunom gostitelja za omrežni vod** v MB/s — to je edina
številka, od katere je odvisna celotna dodelitev matrike. Pustite nastavitev `None`, da se uporabi samodejno zaznana
vrednost. Znižajte jo, kadar niz poroča o okvirjih, poškodovanih zaradi GVSP: avtomatska vrednost izhaja
iz oglaševane hitrosti povezave mrežne kartice, ki precenjuje USB-adapterje, ozke PCIe-pasove in
zasedene skupne strukture – ta precenitev se kaže kot poškodovani okvirji in ne kot
vidno počasna povezava. Vrednost se shrani v bloku zajemanja niza projekta, zato jo
ponovno odprtje ali kasnejša nastavitev `connect_array` obnovi kot katero koli drugo nastavitev niza.
Glej [Stanje niza](#array-health--which-subsystem-is-losing-frames).

#### Pakirani 12-bitni prenos (`packed=`)

Pri polni zmogljivosti senzorja je hitrost posnetkov matrike določena s tem, kako hitro lahko vsaka kamera pošilja
svoje posnetke. 12-bitni formati (`BayerRG12`, `Mono12`) se običajno prenašajo kot 16-bitne
besede; stisnjeni prenos pošlje istih 12 bitov v 1,5 bajta na piksel in
jih razpakira na gostitelju, tako da pošiljanje vsakega okvirja traja 25 % manj časa — približno
**33 % več posnetkov na sekundo pri polni zmogljivosti senzorja** (izmerjeno: 10,7 → 14,0 fps na
nizu 9 kamer). **Shranjeni podatki so identični**: iste vrednosti, iste 16-bitne
datoteke, isti metapodatki.

- `packed="auto"` (privzeto) pakira le, kadar je to koristno **in** si gostitelj lahko
  privošči razpakiranje: ob povezavi backend izmeri lastno hitrost razpakiranja,
  število procesorjev, trenutno obremenitev in omrežni proračun. Namizni računalnik z dovolj prostimi jedri
  komprimira pri polni zmogljivosti senzorja; zaseden ali nizkoenergetski gostitelj (enoploščni računalniki,
  vgrajeni krmilniki) prenaša podatke v razpakirani obliki. Majhna območja zanimanja (ROI) in združeni okvirji prinašajo le majhno korist.
- `packed="off"` (ali `False`) nikoli ne stiska — da se to izključi med diagnostiko,
  ali na gostitelju, ki mora svoj procesor ohraniti za drugo delo.
- `packed="on"` (ali `True`) vedno pakira 12-bitne formate.
- Vsaka druga vrednost sproži `ValueError`. 8-bitni in 16-bitni formati se nikoli
  ne pakirajo.

`analyze_array_network(..., packed="auto")` deluje enako,
tako da je njegova hitrost v fps tista, s katero bo deloval niz (odgovor vsebuje
`packed_transport` in `packed_reason`); njegova privzeta vrednost `pixel_format` je tista, ki jo
uporablja povezava (Mono12 / BayerRG12), namesto 8-bitne.

Odločitev in njen razlog sta v seji:

```python
arr = chloros_sdk.connect_array(serials)          # packed="auto"
print(arr.packed_transport, arr.packed_reason)
# True  auto: projected 11.0 -> 14.4 fps, unpack 0.16 of 1.50 budgeted cores

arr = chloros_sdk.connect_array(serials, packed="off")
```

`ChlorosProject.connect_all(packed=...)` uporablja isto prepisovanje za vsak
niz v projektu (privzeto `None`, kar pomeni `"auto"`: projekt ne shranjuje
nastavitev pakiranja za vsak niz posebej).
Za privzeto nastavitev na ravni računalnika nastavite `CHLOROS_ARRAY_PACKED12=off` (ali `on`) v
okolju backenda; izrecna nastavitev `packed="on"` / `"off"` pa ima še vedno prednost.
`/api/camera/array/status` poroča o `packed_transport` in `packed_decision`.

#### Pričakovane hitrosti snemanja

Izmerjene hitrosti snemanja matrike (sinhronizirane s strojno opremo, 12-bitne; polni senzor = 2048 × 1536). Uporabite jih za preverjanje pravilnosti `status()["actual_fps"]`; `analyze_array_network()` napoveduje hitrost za vaš gostiteljski računalnik in omrežje.

| Nastavitev | Hitrost snemanja v nizu |
| --- | --- |
| Celoten senzor, gostiteljska mrežna kartica PCIe 10 GbE, pakirani prenos (`packed="auto"` na namiznem računalniku) | ~14 sličic na sekundo |
| Celoten senzor, gostiteljska mrežna kartica PCIe 10 GbE, nekomprimirano (`packed="off"`) | ~10,5 sličic na sekundo |
| Celoten senzor, 8 kamer na USB-adapterju 10 GbE | ~3,8 sličic na sekundo |
| Celoten senzor, 1 GbE gostiteljska povezava | 1,2–1,7 sličic na sekundo |
| 2 × 2 združevanje slik (`binning=2`), 10 GbE gostiteljska mrežna kartica | ~30–33 sličic na sekundo |

- Pri povezavi PCIe 10 GbE so pri polnem senzorju omejitev **kamere**, ne omrežje: 4 in 8 kamer delujejo z enako hitrostjo.
- **`target_fps` je hitrost sprožitve, niz pa nikoli ne deluje hitreje.** `target_fps=1` poganja niz s hitrostjo 1 fps; `None` (privzeta nastavitev) ga poganja z najvišjo hitrostjo, ki jo omogočata gostitelj in omrežje. Vrednost, ki presega to hitrost, se zniža na njo (glej [`set_target_fps`](#interval-capture-for-drone-surveys)).
- **`link_speed_mbps` iz `4294` v `status()` ni hitrost povezave.** To pomeni, da Windows ni mogel prebrati hitrosti adapterja (gonilnik je poročal o največji 32-bitni vrednosti), kar je značilno za USB-Ethernet-adapterje. Uporabna hitrost USB-adapterja v sistemu Windows je ne glede na vse omejena na 200 MB/s (glej [Array Health](#array-health--which-subsystem-is-losing-frames)); za nastavitev glejte poglavje »USB 10GbE adapterji« v referenčnem priročniku za CLI pod [Nastavitev in optimizacija gostiteljske mrežne kartice](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Omejitev za USB velja tudi v sistemu Linux: 400 MB/s na x86_64, 200 MB/s na NVIDIA Jetson.** Adapter se prepozna prek sysfs (v primeru napake se uporabi `ethtool -i`), zato se za USB 5 GbE ali 10 GbE adapter upošteva omejitev namesto njegove nazivne hitrosti 500 / ~1060 MB/s. Vrednost za x86_64 je bila izmerjena v sistemu Linux: isti adapter, ki v sistemu Windows poškoduje okvirje pri hitrosti nad 200 MB/s, je brez težav obdeloval niz štirih kamer pri vseh omejitvah do 480 MB/s. Na NVIDIA Jetson je isti adapter deloval brez napak le pri 200 MB/s, nad to vrednostjo pa je občasno izgubljal okvirje. Če vaš gostitelj sprejme več okvirjev brez `gvsp_corrupt`, ga povečajte z `wire_ceiling_mbps`. V sistemu Linux je za adapter na vratih USB 2.0 predvidena hitrost 30 MB/s; priključite ga na vrata USB 3.

#### Prekomerna dodelitev (spodnja meja na kamero)

Mreža deluje v katerem koli omrežju; odločilna je hitrost snemanja. Sim-emit pacing vsaki kameri dodeli enak delež proračuna za varno povezavo brez kolizij, vse pa še vedno posnamejo ob istem sprožilcu, zato je na počasni povezavi delež — in hitrost snemanja — majhen: 8 polnihres 12-bitnih kamer na 1 GbE brez jumbo okvirjev dobi vsaka 6,5 MB/s, približno **1 fps**(približno 1,3 fps ob pakiranju). Delež je omejen na najnižjo hitrost, na katero se kamere lahko sinhronizirajo (`per_cam_floor_bps`: ob povezavi se prebere obseg GevSCPD vsake kamere, nikoli pa ne pod 1 MB/s). Če omrežje zahteva nižjo hitrost, se povezava preklopi na**serijsko**pošiljanje (`slip-emit-and-capture`): kamere se izmenjujejo pri polni hitrosti povezave, tako da vsak okvir prispe, vendar člani**niso izpostavljeni hkrati**— to navajajo stopnja odziva in blokatorji (`SERIAL: ...`) to potrjujejo; posredujte `force_tier="sim-capture-sim-emit"`, da zavrnete preklop. Velikost okvirja ne vpliva na delitev, zato**združevanje in ROI povečata hitrost okvirjev, vendar ne spremenita tega, ali se niz prilega**. Le povezava, ki ne more prenesti matrike pri nobeni hitrosti, niti sočasno niti izmenično,**preobremeni** — način okvare je izguba paketov GVSP, ne nižja hitrost posnetkov. Rešitve: manj kamer, jumbo okvirji od začetka do konca ali hitrejša mrežna kartica.

- Odgovori `analyze_array_network()` in `/api/camera/array/connect` prenašajo `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` in `per_cam_floor_bps`. Ko je `oversubscribed` resničen, projekcija **polja fps nastavi na nič** (`achievable_fps_max` / `fps_bright` / `fps_dark`), namesto da bi poročala o zavajajoči hitrosti, ki je sicer počasna, a deluje.
- `POST /api/camera/array/connect` sprejema parameter telesa `pin_resolution` (**samo HTTP — ni ključni argument SDK**; `connect_array` ga ne razkriva). Gre za strog način: preobremenjena povezava z nastavljenim `pin_resolution` je**odločno zavrnjena** z napako, ki navaja vse možne rešitve. Brez tega se povezava nadaljuje z opozorilom, da bo niz izgubil pakete. V obeh primerih povezava ohrani zahtevano velikost okvirja — nikoli sama ne združuje ali obrezuje.
- Samo za testiranje: nastavite `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` v okolju backenda, da zmanjšate zavrnitev na glasno opozorilo – povezava se vzpostavi kljub temu in sprejmete izgubo paketov.

#### Stanje niza – kateri podsistem izgublja okvirje

`ArraySession.capability()` (`GET /api/camera/array/<array_id>/capability`) prenaša blok v realnem času `health` na
povezanem nizu, ki se ponovno ocenjuje v tekočem **10-sekundnem** oknu. Izgubo okvirjev razčleni
na dva vzroka, ki zahtevata nasprotna popravila, namesto ene stopnje »nepopolnosti«, ki
ne navaja nobenega od njiju:

| Polje | Kaj pomeni | Kateri podsistem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (po serijski številki) | Okvir je **prispel, a je bil strukturno poškodovan**— izguba paketov GVSP. |**Omrežje**: zmogljivost povezave, hitrost prenosa, obroč sprejemnika mrežne kartice, MTU |
| `never_arrived_rate_pct` (po serijski številki) | Okvir **sploh ni prišel**— kamera se ni sprožila ali pa iz nje ni prišlo nič. |**Sprožilec / sinhronizacija**: kabel M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Najslabša hitrost kamere za vsako. | — |
| `per_cam_rate_pct` | Kombinirana stopnja nepopolnosti na kamero (oba vzroka skupaj). | — |
| `stable_for_seconds` | Kako dolgo je vsaka kamera ostala pod 0,01 %. | — |

Poleg `health` isti zapis navaja tudi številko, na kateri visi celotna dodelitev:

| Polje | Kaj pomeni |
| --- | --- |
| `wire_ceiling_mbps` | Veljavni trajni proračun za pasovno širino gostitelja, MB/s. |
| `wire_ceiling_source` | Izvor te številke, v besedah — npr. `USB-capped 200 MB/s (was theoretical 1062; …)` ali `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, ko ga je nastavil `wire_ceiling_mbps=`. |
| `nic_is_usb` | `true` za USB-ethernetni adapter. |

```python
import chloros_sdk

serials = ["213800234", "214000533"]           # master first
arr = chloros_sdk.attach_array(serials)
h = arr.capability()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    # force_disconnect(), not disconnect(): this handle came from attach_array,
    # and disconnect() never tears down an array the session only attached to.
    # It returns only once the backend confirms the release; a teardown that
    # timed out or was refused raises ChlorosConnectError instead, so the
    # reconnect below never races a half-released array.
    arr.force_disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Razlaga:** vrednost `gvsp_corrupt_rate_pct`, ki ni enaka nič, pri vrednosti `never_arrived_rate_pct`, ki je enaka 0, pomeni, da
sta sprožitev in sinhronizacija kabla brezhibni, 100 % izgube pa je na omrežni poti — znižajte
`wire_ceiling_mbps` in ponovno vzpostavite povezavo. Nasprotni vzorec kaže na sinhronizacijski kabel ali
sprožilno linijo.

> **`target_fps` ni ključni dejavnik za poškodovane okvirje.** Hitrost GevSCPD se zapiše enkrat ob
> vzpostavitvi povezave, zato znižanje hitrosti sprožilca spremeni delovni cikel, ne pa
> hitrosti sočasnega oddajanja. Izmerjeno 5-kratno zmanjšanje zahtev ni prineslo izboljšanja, medtem ko
> znižanje zgornje meje kabla s 240 na 200 MB/s pa je pri isti napravi zmanjšalo delež poškodovanih okvirjev s 10,4 % na
> 0,00 %.

> **Delujoči niz se ne more samodejno ponovno načrtovati na firmwareu LATTICE (M3C / M3M).** Tega ne more
> sam popraviti; odklopite in ponovno priključite, da bo izbirnik časa povezave ponovno načrtoval glede na
> novo zgornjo mejo.

**USB-ethernetni adapter je s sondo omejen na 200 MB/s**, ne glede na njegovo
oznake: tabela učinkovitosti, ki pretvori hitrost povezave v trajno vrednost, izhaja iz
PCIe, USB-mrežna kartica pa oglašuje svojo hitrost ethernetne povezave, pri čemer je omejena z
USB-vodilom in njegovim gonilnikom. Omejitev je absolutna, ne relativna — USB 1 GbE adapter
dosega ~80 MB/s in ni prizadet. Velja za Windows in Linux (vključno z Jetsonom);
glejte [Pričakovane hitrosti okvirjev](#expected-frame-rates) za podatke o tem, kje je bila izmerjena.

#### Metode `ArraySession`

| Metoda | Opis |
| --- | --- |
| `status(timeout=10.0)` | Trenutno stanje delujočega niza: `target_fps` / `target_fps_auto` (hitrost sprožilca in ali jo upravlja krmilnik hitrosti), `target_fps_clamped_reason` (zakaj je fiksirana hitrost nižja, sicer `None`), `actual_fps` (izmerjeno; 0, ko se zanka zajemanja vda ali zastane), `recommended_fps`, `streaming` (False, ko se zanka zajemanja vda — preverite to, ne le `last_error`; ostane »True« tudi po ponovnem zagonu nastavitev, kar označi `restarting`), `stalled` (»True«, dokler zanka zajemanja teče, vendar ni objavila nobene skupine okvirjev v približno desetih obdobjih okvirjev, 2–30 s; `last_error` to sporoči šele po 60 s), `last_frame_age_s`, `last_error`, `serials`, `ptp_enabled` (`True` le, če je bil niz povezan s PTP **in** je vsak član nazadnje prebral `PtpEnable` v stanju »on«; to ni isto kot »zaklenjeno«, kar meri `ptp_epoch`), `ptp_configured` (kar je zahteval povezovalnik), `ptp_off_members` / `ptp_unread_members` (člani, ki so nazadnje prebrali `PtpEnable` izklopljeno / niso odgovorili), `member_recovery` (`None` ali član, ki je izgubil napajanje ali povezavo: `state`, `waiting`, čeprav ne odgovarja, `reopening`, nato `recovered`, skupaj z `members` in `why` — niz ga ponovno odpre, ko spet odgovori, in mu vrne njegov PTP časa povezave, sprožilec in ožičenje sinhronizacijskega vodila, hitrost in nastavitve kamere, nato pa ponovno prenaša brez novih `connect_array`) ter `ptp_epoch` (izmerjeno), `sync_mode` / `sync_tier`, `packed_transport` / `packed_decision`, `per_cam_emit_mbps`, `link_speed_mbps` (`4294` pomeni, da Windows ni mogel prebrati hitrosti adapterja — glej [Pričakovane hitrosti slik](#expected-frame-rates)), `trigger_pipeline` / `pipeline_tune`, `radiometric_chain` (ali je veriga sevanja po pasovih uporabna in kateri član jo je prekinil) ter `band_clamp` — medtem ko v živo prikazani pogled upodablja matriko: delež osvetljenih pikslov, ki jih je razmešavanje omejilo na 0, po posameznem elementu (`members`: `{serial: [band0, band1, band2]}` v vrstnem redu pasov R, G, B kamere) ter po posameznem pasu, ki ga prikazuje sestavljeni posnetek (`composite`: `{band name: share}`), pri čemer `warn` predstavlja delež, ki ga grafični vmesnik označi kot »podatki pasov z nizko zanesljivostjo« (merjeno približno enkrat na sekundo; `None`, kadar ne teče živi prikaz), ter `member_ae` — za vsakega člana, osvetlitev (`exposure_us`) in ojačanje (`gain_db`) v zadnjem objavljenem okvirju, ki ju je prinesel sam okvir (`read_nodes` ali `ExposureTime` / `Gain` vrne zadnjo vrednost, zapisano med delovanjem avtomatske osvetlitve, ne pa tisto, kar trenutno počne). |
| `set_member_settings(serial, **kwargs)` | Zapiše nastavitve ene kamere iz niza, kot jih `CameraSession.set_settings()` prevzame (osvetlitev, ojačanje, ciljna svetlost, ...); član ostane v nizu. Izvede `ChlorosConnectError` za serijsko številko, ki ni član. Glej [Osvetlitev na članih niza](#exposure-on-array-members). |
| `capability(timeout=10.0)` | Na čem se je dogovoril smart-prep (stopnja zajema, velikost okvirja, `pixel_format`), blok v živo `health` in proračun gostitelja za prenos podatkov. Glejte [Stanje matrice](#array-health--which-subsystem-is-losing-frames). |
| `set_target_fps(fps, timeout=10.0)` | Spremenite hitrost sprožanja med delovanjem matrice, brez ponovnega povezovanja. Številka jo fiksira (krmilnik hitrosti jo pusti pri miru); `None` ga vrne v avtomatski način. Vrne `{requested, target_fps, target_fps_auto, clamped}` — `clamped` je True, ko je bila frekvenca znižana na vrednost, ki jo lahko preneseta gostitelj in omrežje. Glej [Intervalno zajemanje za raziskave z droni](#interval-capture-for-drone-surveys). |
| `grab(processing="debayered", with_metadata=False, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, bit_depth=None, band=None, align=False, refl_max=None)` | En posnetek iz vsake kamere, **vsi iz enega sprožilca**, kot numpy-matrike (datoteke se ne shranjujejo). Vrne `GrabResult` (ali `dict`): `{serial: array}` ali `{serial: (array, metadata)}` z `with_metadata=True`. `.group_id` je sprožilec, ki ga posnetki delijo; `.errors` vsaki izpuščeni kameri dodeli razlog (kamera, ki pri tem sprožilcu ne more biti obdelana, je izpuščena in se nikoli ne dopolni iz druge). `processing` je `raw` / `debayered` / `radiance` / `reflectance` / `display`. Enako kot pri projektnem API-ju [`ArrayHandle.grab()`](#one-trigger-per-read--group_id) — ena skupna implementacija — zato vsaka ključna beseda tam (`band`, `align`, `bit_depth`, `refl_max`) deluje tudi tukaj. Potreben je `numpy`. |
| `frame_stream(*, processing="debayered", fps=5.0, duration=None, count=None, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, with_metadata=False, bit_depth=None, band=None, align=False, refl_max=None)` | Iterator, ki na vsak tick vrne en rezultat `grab()` (samo ključni argumenti), s hitrostjo `fps`, omejen na `duration` (sekund) in/ali `count` (`None` = neomejeno). **Vsak vrnjeni niz je nov sprožilec:** tick, katerega najnovejši sprožilec je množica, ki jo je že vrnil, počaka na naslednji sprožilec, namesto da bi ga ponovil, zato tok teče s hitrostjo, ki je manjša od vrednosti `fps` ali lastne hitrosti osveževanja niza. Sproži `FrameFetchError`, namesto da bi utihnil: takoj, ko vsaka kamera iz kakršnega koli razloga zavrne ponovitev naslednjega takta, po 60 sekundah takta brez okvirja in po 60 sekundah brez novega sprožilca („brez novega sprožilca 60 sekund“). Enaka implementacija kot `ArrayHandle.frame_stream()`. Glej [Žive slike v vašem lastnem potoku](#live-frames-in-your-own-pipeline). |
| `continuous(output_dir="output", *, format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, max_captures=None, duration_s=None, daq=None, daq_tolerance_ms=None, timeout=30.0)` | Zaženi lastno neprekinjeno zajemanje backenda (način »Continuous« v grafičnem vmesniku) → [`ContinuousHandle`](#continuoushandle). Shrani **vsako** sinhronizirano skupino, ki jo objavi niz, tako kot bi jo shranil `capture()` (enaki nadzorni mehanizmi za izvoz), dokler se ne ustavi ali dokler `max_captures` / `duration_s` (`None` = dokler se ne ustavi). Izvrši `ChlorosConnectError`, če za to matrico že teče. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, smart=False, after_seq=None, daq=None, daq_tolerance_ms=None, timeout=300.0)` | Ena sinhronizirana skupina zajemov. Vrne `CaptureResult` (seznam slovarjev okvirjev + `.skipped` + `.failed` + `.group_seq` + `.daq_files` + `.daq_matches` + `.alignment` + `.oriented`). Predhodni rezultat`group_seq` kot `after_seq`, da dobite naslednjo sinhronizirano skupino po njej, namesto da čakate na novo. Nadzor izvoza spodaj. |
| `capture(..., smart=True)` | **Pametno zajemanje** — počaka, da se AE ustalijo na vseh kamerah, nato sproži zajem. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, orient=None, timeout=120.0)` | Najhitrejše zajemanje: samo surovi podatki + dodeljena vrednost DAQ (+ prosti kombinirani indeks). Odraža gumb »Najhitrejše zajemanje« v grafičnem vmesniku. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Enkratno / neprekinjeno / intervalno zajemanje v enem omejenem ciklu. Vrne `list[CaptureResult]`.**Zahteva `count` in/ali `duration_s`**, da se zaključi (SDK nima Ctrl+C). Prav tako se predčasno zaključi po prehodu, ko prehod poroča o polnem pogonu (vnos `.failed` z `disk_full`); ta prehod je zadnji rezultat. |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, expr=None, timeout=30.0)` | Začni snemati pogled v živo s kombiniranim indeksom v video/GIF → `RecorderHandle`. Video je v formatu Motion-JPEG `.avi` (ffmpeg ni potreben). Če ni odprt noben grafični vmesnik, ga renderira backend: `expr` (indeksni izraz v imenih pasov matrike, npr. `"(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)"`) ali privzeto NDVI matrike. En sestavljeni snemalnik na matriko. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, orient=None, timeout=30.0)` | Začni serijsko snemanje surovih Bayerjevih podatkov z visoko hitrostjo snemanja → `RecorderHandle`. Ponovno obdelaj offline z `build_video()`. `orient=True`: izdelki, ki jih ustvari `build_video()`, so obrnjeni glede na orientacijo kamere (izdelki posamezne kamere glede na njeno lastno orientacijo, združen indeks pa glede na orientacijo glavne kamere); surovi posnetki ostanejo v izvirni orientaciji, manifest serije pa beleži orientacijo vsakega posameznega posnetka. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Offline-ponovna obdelava shranjene surove serije posnetkov v kalibrirane videoposnetke. Blokira delovanje, dokler ni končano (`wait=True`), in vrne rezultat izdelave: `{success, outputs, errors, skipped, substitutions, notes, degraded, combined}` — glej [rezultat izdelave](#the-build_video-result) spodaj. |
| `build_video_status(job_id, timeout=15.0)` | Preveri stanje offline gradbene naloge: `{running, result, error, burst_dir}`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Sprosti celoten niz — **če ga je ta seja odprla** (`connect_array`): takrat je `force_disconnect()`, in sproži `ChlorosConnectError`, če sprostitev ni potrjena (izhod iz bloka `with` nikoli ne sproži napake). Pri ročaju iz `attach_array` pusti niz v delovanju in v stderr izpiše eno vrstico, ki to navaja: niz lahko pripada grafičnemu vmesniku ali drugemu skriptu. Vsaka kamera ostane v prostem delovanju in sicer takšna, kot jo je povezava našla; `keep_settings=True` shrani njene nastavitve v lastni pomnilnik kamere, `clear_settings=True` pa ta pomnilnik ponastavi na tovarniške nastavitve (glej [Pomnilnik kamere](#camera-memory--keep_settings--clear_settings)). |
| `force_disconnect(timeout=60.0, keep_settings=False, clear_settings=False)` | Sprosti celoten niz ne glede na to, kdo ga je odprl. Uporabite ga na priključenem ročaju, če želite zaključiti niz (npr. da ga ponovno povežete z drugačnimi nastavitvami). Vrne se, ko backend potrdi sprostitev ali ko ni ničesar za sprostiti (niz je že izginil ali backend ne teče); razgradnja, pri kateri je potekel časovni limit ali jo je backend zavrnil, sproži `ChlorosConnectError`. Ne glede na zastavico vrne, kaj je dobil pomnilnik vsake kamere (`camera_memory`), sicer pa `None`. |

`capture()` nadzor izvoza (ista končna točka, kot jo uporabljata GUI in CLI):

- `processing` / `levels` — `processing="all"` (ali `levels=["raw","radiance",…]`) shrani vsako ustrezno vrsto izvoza za vsako kamero; ena sama vrednost `processing` shrani le to raven.
- `aligned` — **privzeto vklopljeno (`None`), saj je zajem istega niza v aplikaciji Chloros**(njena nastavitev zajema**Poravnano** za posamezni niz). Izvoz v ne-surovi obliki vsakega člana se prilagodi v okvir glavne kamere z [profilom poravnave](#array-alignment) in obrezan na skupno prekrivanje elementov, tako da so datoteke pikselno poravnane ter imajo enako velikost, izrez in oznake `Alignment*`, kot jih določa aplikacija (znotraj ROI se transformacija ponovno prilagodi oknu); `XMP-mapir:AlignmentJSON` navaja `"applied": true` s svojim `crop_roi`. Surova datoteka ostane neizkrivljena v izvorni velikosti, vendar nosi transformacijo v metapodatkih. `aligned=False` shrani vsako datoteko v izvorni velikosti, neizkrivljeno; `aligned=True` je privzeta nastavitev. `continuous()`, `capture(smart=True)`, `capture_repeated()` ter `ArrayHandle.capture()` / `capture_smart()` na ravni projekta uporabljajo isto privzeto nastavitev. Ne glede na to, ali je popačenje uporabljeno ali ne, vsaka datoteka člana zabeleži transformacijo `Alignment*` svoje kamere, ko ima niz profil poravnave — razen kamere, ki se s profilom ne poravna (izpuščena ali prisiljena v identiteto zaradi neuspešnega reševanja) ter referenčna kamera, kadar ni poravnana nobena druga kamera: te ne vsebujejo oznak `Alignment*`. Če niz nima profila, se posnetek shrani neporavnan, kar je razvidno iz rezultata: `result.alignment` je `{"requested": True, "applied": False, "warnings": [...]}`, eno opozorilo za vsak razlog, zaradi katerega popravka ni bilo mogoče uporabiti (ni profila, člani z različnimi ločljivostmi, niz, ki nisinhroniziran, okvir, na katerega profila ni mogoče preslikati). `applied` je `True` le, če je vsak izvoz, ki je zahteval popravljanje, to tudi dobil. Samodejno poravnavanje matrike ustvari profil, ko se osvetlitev ustalijo; `ArrayHandle.calibrate_alignment()` / `align-calibrate` v CLI ga ustvarita na zahtevo. `result.oriented` navaja, ali je `orient=True` obrnil izdelke.
- `render_index=False` — preskoči prekrivni sloj indeksa vegetacije za vsako kamero; privzeto se prikaže tam, kjer je konfiguriran.
- `daq` / `daq_tolerance_ms` — odčitek DAQ se privzeto shrani kot stranski datotek `.daq`, tudi če ga nobena izbrana raven ne potrebuje; `daq=False` ga v tem primeru preskoči, `daq_tolerance_ms` pa nastavi toleranco ujemanja (privzeto 500). `force_daq=True` je starejša pisava privzete vrednosti. Glej [Meritev DAQ, ki jo shrani zajem](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
- `orient=True` — vklopi **produkte**vsakega člana (`display`, `radiance`, `reflectance`, indeks po kameri in sestavljeni) z orientacijo kamere po poravnavi: z orientacijo**glavne kamere**, ko so bili prilagojeni v njen okvir (privzeta nastavitev `aligned` — člani ostanejo usklajeni), sicer pa z orientacijo lastne kamere.**Privzeto: vsaka datoteka člana v izvirnem formatu**, kot je vedno pri zajemih v obliki matrike; `raw` in `debayered` ostajata v izvirnem formatu v vsakem primeru. Vsaka datoteka beleži orientacijo okvirja, v katerem so njeni piksli (`XMP-Camera:ChlorosRotation` / `ChlorosFlip*` / `ChlorosOrientationApplied` — surova datoteka, izvirni okvir lastne kamere), datoteka XMP `Alignment*` pa ohranja transformacijo izvirnega okvira, ki je bila uporabljena pred usmeritvijo; surova datoteka, ki vsebuje to transformacijo, beleži tudi `ChlorosAligned*` (orientacijo referenčnega okvirja), tj. orientacijo izdelka, prilagojenega v referenčni okvir. Obdelava vsak izdelek zavrti za vrednost, ki pripada njegovim pikslom: za vrednost poravnave, če prilagaja surovo datoteko (vklopljena poravnava matrike), ali za vrednost kamere, če ne. `capture_repeated()` in `capture_fastest()` ga prepustita; tudi `ArrayHandle.capture()` / `capture_smart()` na ravni projekta ga prevzameta.

**Stiskanje TIFF (gumb HTTP):** `ArraySession.capture()` ne pošilja ključa `compression`, zato velja privzeta nastavitev strežnika — `POST /api/camera/array/capture` prebere parameter telesa `compression`, `"none"` pa privzeto: neskomprimirano, piksli senzorja točno takšni, kot so bili prebrani (~6,3 MB na posnetek v polni ločljivosti). `"deflate"` (brezizgubni zlib L1 + horizontalni prediktor, ~4,1 MB/okvir) piše manjše datoteke počasneje — obe sta brezizgubni in se ob uvozu berejo enako. SDK za to ne ponuja nobenega ključnega argumenta; izhod v sili je `chloros-cli lattice array-capture --compression deflate` ali surovi HTTP. DEFLATE zadržuje Pythonov GIL, zato se stisnjeno zapisovanje ne izvaja vzporedno v niti za zapisovanje posameznih kamer — za neprekinjeno zajemanje v polni ločljivosti s hitrostjo senzorja je potrebna nestisnjena privzeta nastavitev. Podrobnosti: [Referenca CLI-ja → zajem v obliki niza](cli-reference.md).**Zaporedne skupine (protokol HTTP):** končna točka vrne `group_seq`; zanka, ki jo pri naslednji zahtevi pošlje nazaj kot `after_seq`, pridobi naslednjo sinhronizirano skupino — vključno s tisto, ki je bila objavljena med shranjevanjem prejšnjega zajema — namesto da bi čakala na skupino, objavljeno po zahtevku, `skipped_groups` pa poroča o vseh, ki jih je zanka spregledala. `capture_repeated`, `--continuous` v CLI in neprekinjeni način v GUI to počnejo; intervali ostanejo posodobljeni ob vsakem načrtovanem trenutku.**Prepisovanje izvoza po posameznih članih (samo HTTP):**ista končna točka sprejema tudi `exclude_serials` (seznam – izključitev članov iz shranjenega niza; niz se še vedno sproži kot ena sinhronizirana skupina, izključeni člani pa se vrnejo v `excluded`), `serial_levels` (prepisovanja na ravni posamezne kamere `{serial: [level tokens]}`) ter `serial_index` (prepisovanja prekrivanja indeksov na ravni posamezne kamere `{serial: bool}`). To so parametri telesa, ki ustrezajo grafičnemu vmesniku (GUI), in**še niso ključni argumenti SDK**; člani, ki niso prisotni v zemljevidih, se vrnejo na vrednosti za celoten niz `levels` / `render_index`.

##### Pregled preskočenih kamer — `CaptureResult.skipped`

`ArraySession.capture()` vrne `CaptureResult`, ki je podrazred `list`: ga prehodite, indeksirajte, `len()` — vsak obstoječi vzorec še naprej deluje. Nova koda lahko pregleda atribut `.skipped`, da ugotovi, katere kamere so bile izključene in zakaj. Najpogostejši primer so RGB-kamere v matrici z mešanimi filtri, ko zahtevate `processing="radiance"` ali `"reflectance"` — saj je sevanje na posamezni Bayerjevi celici za širokopasovni senzor brez pomena, zato backend te kamere preskoči, namesto da bi ustvarjal nesmiselne podatke.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Oznake razlogov sledijo vzorcu `<level>-not-applicable-to-rgb-cam` (en vnos na preskočeno raven, vsak z oznako `level`). Kamera, katere kalibracijski paket ni naložen, preskoči `radiance` / `reflectance` z `<level>-needs-calibration (connect the camera so its cal pack loads)`, če pa ni uporabne vrednosti odbojnosti iz DAQ, preskoči z `reflectance-skipped-no-dls`. Drugi preskoki, povezani z odbojnostjo, so `reflectance-skipped-no-fresh-dls` (ni na voljo nove vrednosti navzdolnega sevanja), `reflectance-skipped-bound-daq-unavailable (…)` (do vezanega DAQ-ja ni bilo mogoče vzpostaviti povezave) ter `dls-uncalibrated-band-<nm>` — pas leži večinoma zunaj radiometrično kalibriranega območja svetlobnega senzorja DAQ (~374–974 nm), zato je absolutna delitev na podlagi odbojnosti, ki temelji na DAQ-ju, zavrnjena, okvir pa se glasno vrne na odziv senzorja. Med serijskimi SKU-ji to sproži le F988; podprta pot te kamere je delovni tok z odbojnostnim panelom.

##### Datoteke, ki jih ni bilo mogoče zapisati — `CaptureResult.failed`

Zajem, ki je lahko zapisal le nekatere od svojih datotek, še vedno vrne: seznam vsebuje vse datoteke, ki **so bile** zapisane, `.failed` pa navaja preostale, en slovar na datoteko z `serial` (`None` za združeno kompozitno sliko in sidecar `.daq` matrike), `level` (`raw`, `radiance`, …, `index`, `composite`, `daq`, ali `match` — izvor `.daq`-a in `.match.json`-a, ko je bil `.daq` zapisan, njegov stranski datotek pa ne) ter `error`. Na polnem disku vnos vsebuje tudi `disk_full: True`; vsak poznejši poskus zajema na ta disk bi na enak način spodletel, zato se `capture_repeated()` tam ustavi. Za delni zajem se ne sproži nobena izjema, datoteka v `.failed` pa ni ostala na disku napol zapisana. `.failed` je prazen pri čistem zajemu.

```python
result = arr.capture("output/", processing="all")
for f in result.failed:
    print(f"SN:{f['serial']} {f['level']} not written: {f['error']}")
if any(f.get("disk_full") for f in result.failed):
    raise SystemExit("drive full -- free space and capture again")
```

##### DAQ, ki bere zajem, shrani — `.daq_files` in `.daq_matches`

Vsakič, ko je DAQ vezan na kamero ali niz ali združen v skupino, vsak zajem shrani odčitek navzdolnega toka, ki ga je uporabil: datoteko `.daq` v podmapo `daq/` mape z zajetimi podatki, skupaj z datotekama `.match.json` in `.cameras.json` (katerega odčitek je). To velja tudi za surove ali debayirane zajeme, datoteka pa je enaka ne glede na to, ali je bil zajem opravljen prek grafičnega vmesnika (GUI), ukazne vrstice (CLI) ali razvojnega kompleta (SDK). `daq=False` to preskoči za zajem, katerega ravni ne uporabljajo DAQ-ja (raven `radiance`, `reflectance` ali `display` jo še vedno zapiše); `daq_tolerance_ms` premakne mejo tolerance za to posnetka in nikoli za uporabljeno odčitano vrednost. Kateri DAQ: tisti, vezan na kamero ali matriko (izbira svetlobnega senzorja v grafičnem vmesniku, vezava projekta ali `chloros-cli daq pool-bind`), sicer pa najnovejši, ki ga ima backend. CLI navaja ukaze `--no-daq` in `--daq-tolerance-ms` ([CLI Reference](cli-reference.md#the-daq-reading-a-capture-saves)).

```python
r = cam.capture("out", levels=["raw"])      # a CameraSession; ArraySession results match
for m in r.daq_matches:                      # one per .daq in r.daq_files
    print(m["daq_file"], m["gap_ms"], m["in_tolerance"])
```

Zajem shrani odčitek svetlobnega senzorja, ki ga je uporabil, kot `.daq`, z `<file>.daq.match.json` poleg njega, ki navaja, kako je bil odčitek povezan z osvetlitvijo: `method`, `gap_ms` (od osvetlitve do najbližje vrednosti DAQ) in `in_tolerance` (v roku 500 ms ali `daq_tolerance_ms` zajema). XMP datoteke odbojnosti vsebuje enako oceno kot `Chloros:DlsMatchMethod`, `Chloros:DlsMatchGapS` in `Chloros:DlsInTolerance`.

Zajem čaka na prvo odčitano vrednost DAQ-ja po osvetlitvi, da lahko združi odčitke na obeh straneh te vrednosti (`method` `interpolated`). DAQ-E pošilja odčitek vsake 0,4 s na dnevni svetlobi in vsake 1–2 s v zaprtih prostorih, zato se `capture()` vrne šele približno en in pol takšnega intervala kasneje; sama osvetlitev ni zamaknjena. Na DAQ, ki je prenehal pošiljati podatke, se ne čaka. Odčitek DAQ-E je povprečje svetlobe v intervalu od prejšnjega odčitka, zato se umesti na sredino tega intervala: `gap_ms` se izmeri glede na to sredino, `.match.json`'`bracket_stamps_ns` pa podaja časovne oznake samih odčitkov. `ArraySession.continuous()` čaka, ne da bi upočasnil zajem. `capture_repeated()`-jevi povratnine čakajo, zato ohranijo svojo hitrost in se uskladijo z najbližjo vrednostjo pred osvetlitvijo.

Vrednost nikoli ne poteče, zato se zaradi zamude ne izgubi noben okvir – namesto tega je označena:

- **Tok DAQ-ja se je ustavil** — uporabi se njegova najnovejša vrednost, `gap_ms` pa navaja, kako stara je (`in_tolerance: false` več kot 500 ms).
- **Ni DAQ-ja za odčitavanje**— prekinjena povezava (`disconnect()`, `chloros-cli daq pool-disconnect`) ali pa ni mogoče vzpostaviti povezave z vezanim DAQ-jem — ponovno se uporabi zadnja vrednost kamere in se pripiše**tej** ekspozicijo: `method`, `retained`, `gap_ms` iz te ekspozicije, `in_tolerance` se ponovno izračuna, `retained_from_epoch_ns` pa navaja osvetlitev, s katero je bil prvotno usklajen. Kamera, ki ni nikoli imela odčitka, še vedno nima odbojnosti, rezultat pa navaja razlog (`reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)` ali zavrnitev zajema).

Datoteke, ki jih zajem shrani v mapo projekta, odprtega v programu Chloros, se dodajo na seznam datotek tega projekta, tako kot to stori zajem v aplikaciji (datoteke, shranjene drugje, ostanejo nedotaknjene): `CameraSession.capture()`, `ArraySession.capture()` (tudi `smart=True`) in `capture_cameras_transient()`. Rezultat je naveden v datoteki `.project_add` (`capture_cameras_transient()`: `project_add` v slovarju) -- `held`, `files` in `message` v ozadju; `held` je `True`, medtem ko poteka obdelava, datoteke pa se združijo, ko se ta konča. `None`, če v projekt ni bila dodana nobena datoteka.

`CameraSession.capture()` in `ArraySession.capture()` ju vrneta kot rezultat `.daq_files` in `.daq_matches`, kot odgovori zajetih HTTP-odzivov (`/api/camera/<serial>/cli-capture`, `/api/camera/capture`, `/api/camera/capture-all-synced`, `/api/camera/array/capture`) vsebujejo `daq_files` in `daq_matches`, po enega na `.daq`: `daq_file`, `cameras`, `method`, `gap_ms`, `in_tolerance`, `tolerance_ms` in `stream_fault` — lastna napaka toka DAQ-ja, ki se preveri le v primeru odčitka zunaj dovoljenega odstopanja. Preden se zanesete na odbojnost iz zajema, preverite `in_tolerance` v njegovem `.match.json` ali XMP, ali pa ravni DAQ-ja [`status()`](#daqsensorsession-methods).

Ravni `processing`:

| Raven | Izhod |
| --- | --- |
| `"raw"` | Enokanalni Bayer (monokromatske kamere: en pas) neposredno iz senzorja. |
| `"debayered"` *(privzeta nastavitev SDK)* | 3-kanalna barva: temni odmik (DSNU) iz kalibracijskega paketa, odvzet iz surovih podatkov, nato pa demosaikiranje z upoštevanjem robov po OpenCV — brez LUT za linearnost, izravnave polja ali ločevanja (črno-bele kamere: 1-kanalna siva lestvica). Shranjena datoteka TIFF je v vrstnem redu kanalov **RGB**(fotometrični RGB, zato vsak pregledovalnik slik prikaže prave barve); nizi v pomnilniku, ki jih vrneta `grab()` / `frame_stream()`, so v vrstnem redu**BGR** — za primerjavo s shranjeno datoteko obrnite kanale (`frame[..., ::-1]`). `chloros-cli process` zapiše enak izdelek kot `Debayered_Images`. |
| `"radiance"` | float32 W/m²/sr/nm prek celotne radiometrične verige. Samo za multispektralne kamere — RGB-kamere se preskočijo. |
| `"reflectance"` | uint16 0..32768 (pripravljeno za Pix4D); za absolutno referenco je potrebno povezovanje z živim DAQ. Samo za multispektralne posnetke. |
| `"display"` | Celotna veriga, ki ustreza predogledu v grafičnem vmesniku (CCM + WB + gama v skladu s profilom kamere). |
| `"all"` | **Ena datoteka na ustrezno raven** za vsako kamero (v skladu s privzeto nastavitvijo GUI »Capture All« / CLI). Vrnjeni `CaptureResult` nato vsebuje en slovar okvirja na `(cam, level)`, z ravnjo v vsakem slovarju; neuporabne ravni pa se pojavijo v `.skipped`. DAQ, ki bere uporabljen posnetek, se shrani kot stranski datotek `.daq`. |

> **Opomba — privzeta vrednost se razlikuje od tiste v CLI.** `ArraySession.capture()` je privzeto nastavljen na `processing="debayered"`; ukaz `chloros-cli lattice array-capture` je privzeto nastavljen na `processing="all"`. Izrecno prenesite `processing="all"` iz SDK, da posnemate večstopenjsko shranjevanje v CLI/GUI.

##### Imena datotek zajetih nizov

Vsaka raven se shrani v svojo podmapo v mapi `output_dir` (`raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` za `display`, `index/`, `composite/`). Vsaka datoteka ima ime kamere v obliki `SN<serial>-<filter>`: serijska številka, nato pa del modela kamere, ki označuje filter — `SN214000533-FOCN` za `M3C-L87-FOCN`, `SN261902142-F450` za model `M3M-L87-F450` — združeno z `-`, tako da ima ime enaka polja, ločena z `_`, kot prej. Kamera, katere modela ni mogoče prebrati, je samo `SN<serial>`, posnetki iz prejšnjih izdaj pa v imenu nimajo filtra. Ime datoteke je odvisno od tega, koliko ravni je član zahteval:

| Zahtevana raven | Ime datoteke | Primer (`reflectance`, privzeto `format="tiff"`) |
| --- | --- | --- |
| **Ena raven** (`processing="reflectance"` ali eno-elementni `levels`) | `sync_<timestamp>_SN<serial>-<filter>.<format>` — brez končnice ravni, kot v prejšnjih izdajah, zato obstoječi skripti še naprej delujejo | `reflectance/sync_<ts>_SN214000533-FOCN.tiff` |
| **Več ravni** (`processing="all"` ali `levels` z več kot enim elementom) | `sync_<timestamp>_SN<serial>-<filter>_<level>.<format>` | `reflectance/sync_<ts>_SN214000533-FOCN_reflectance.tiff` |

[maska omejitve](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) sledi imenu izdelka, `clamp_masks/<product folder>/<product name>_clampmask.tif`: zajem odbojnosti na eniravni zajema odbojnosti zapiše `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`, medtem ko `processing="all"` zapiše `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`. Pravilo za končnico se določi za vsak element posebej glede na ravni, ki so bile zahtevane, zato preskočena raven (npr. odbojnost na RGB-kameri) tega ne spremeni. Preberite poti iz vrnjenih okvirjev, namesto da jih sestavljate. Posnetek z eno kamero (`CameraSession.capture()`) vedno doda raven: `capture_<ts>_SN<serial>-<filter>_<level>.tiff`.

##### PNG / JPEG: upoštevata le prikazne in indeksne izdelke

PNG ali JPEG vsebuje 8 bitov na kanal. Zahteva za format z ravnjo, ki je ne more podpreti, se zavrne še pred zajemom, s sporočilom, ki navaja raven in TIFF kot format, ki jo podpira:

- **`radiance`**(float32) in**`reflectance`**(uint16, `32768` = 1,0) — ter s tem tudi `processing="all"`, ki ju vključuje. SDK to preveri na argumentih: `capture()` sproži**`ValueError`** brez zahtevka (`CameraSession`, `ArraySession`, `CameraHandle`, `ArrayHandle`, `capture_smart`).
- **`raw` / `debayered` iz kamere, ki prenaša več kot 8 bitov** (12-bitni M3C, Mono12 M3M) — strežnik jih zavrne, saj pozna bitno globino vsake kamere: HTTP-odziv 400 z `error_code` `FORMAT_CANNOT_HOLD_LEVEL`, sprožen kot `ChlorosConnectError` (`CameraSession` / `ArraySession`) ali `RuntimeError` (`CameraHandle` / `ArrayHandle`). Enostopenjski zajem `debayered` ene kamere zajame 8-bitni okvir za formate PNG in JPEG ter ni zavrnjen.

Radiometrične podatke shranite kot `format="tiff"` (`ext=".tiff"`); za `display` in indeksirane upodobitve uporabite formate PNG/JPEG.

#### Kako deluje samodejna osvetlitev

- **Najprej osvetlitev, na koncu ojačanje.** Ko je ojačanje nastavljeno na »Auto«, ostane na 0 dB, dokler osvetlitev ne doseže najdaljše dovoljene vrednosti.
- **Pri ročni osvetlitvi ostane ojačanje v načinu »Auto«**, dokler ne spremenite ojačanja ali ga ne nastavite na ročni način. Ko se vrnete v način »Auto«, je omejitev ojačanja enaka prejšnji.
- **Majhne svetle točke ne potemnijo prizora.** Odsev, ki zavzema do 1 % okvirja, se ignorira, v panelih znanega kalibracijskega cilja pa do 0,5 % panela.
- **Zgornja meja osvetlitve DAQ velja le, dokler je kalibracijski cilj v vidnem polju.** To preprečuje preosvetlitev belega panela; če panel ni viden, kamera izpostavi sceno.
- **Pri konstantni svetlobi ostanejo vrednosti nespremenjene, spremenjena svetloba pa se ustalijo v nekaj sekundah.** Pri konstantni svetlobi ostanejo osvetlitev in njena meja nespremenjeni, po večji spremembi svetlobe pa se slika ustalijo v nekaj sekundah. Preklip ne omeji več osvetlitve za vedno, sprememba območja zanimanja (ROI) pa ponovno najde ustrezno osvetlitev.
- **Ciljna svetlost ostane v dosegu.** Ne narašča, medtem ko je kamera na svoji meji osvetlitve in se njen ojačevalni faktor ne more povečati, zato svetloba, ki se vrne, ne povzroči preosvetlitve slike.
- **Kamera, katere avtomatska osvetlitev preneha odgovarjati, se ohrani na ravni, ki ustreza trenutni svetlobi.** V močnem soncu bi kamera lahko padla na najkrajšo osvetlitev in tam ostala s temnim kadrom, pri čemer bi bile vse njene nastavitve v normalnem območju. Chloros spremlja kadr, ohranja kamero na ravni, ki jo je vzdrževala za to svetlobo, in jo sprosti, ko se svetloba spremeni. Temni ali zakriti svetlobni senzor tega ne moti.

#### Osvetlitev na članih matrike

Kamera LATTICE omeji `ExposureTime` na eno obdobje svoje hitrosti snemanja. Medtem ko matrika prenaša podatke, Chloros izvaja vsak element z vklopljenim `AcquisitionFrameRateEnable`, s hitrostjo, ki je nekoliko višja od sprožilne hitrosti, tako da zgornja meja osvetlitve člana leži nekoliko pod sprožilnim obdobjem, **v katerem koli ROI**:

- Obdobje slike člana je obdobje sprožitve z odbitkom 15 % tega obdobja (najmanj 3 ms, največ polovica obdobja), njegova zgornja meja osvetlitve pa je to obdobje slike z odbitkom približno 0,1 ms. Primer: sprožilec s 10 fps (100 ms) daje hitrost člana približno 11,8 fps in zgornjo mejo izpostavljenosti približno 85 ms.
- **Ročna nastavitev `ExposureTime`, ki je daljša od te zgornje meje, upočasni sprožilec, da se prilagodi tej vrednosti.** Chloros najprej odpre zgornjo mejo tega člana za to vrednost in pusti, da počasnejša frekvenca teče nekaj sprožilnih obdobij (pisanje lahko traja nekaj sekund dlje), nato jo zapiše, sprožilec pa se sproži največ tako hitro, kot to dopušča čas osvetlitve. Pri nizu z avtomatsko hitrostjo se prejšnja hitrost vrne, ko se vsak član, katerega ročna osvetlitev je upočasnila delovanje, ponovno preklopi na avtomatsko osvetlitev. Vrednost, ki je kamera sama ne more sprejeti, je še vedno zavrnjena, pri čemer je obseg, ki ga zdaj sprejema, naveden v napaki, avtomatska osvetlitev člana pa se ponovno vklopi, če je bila izklopljena le za to zapisovanje. Pred različico 1.3.0 je majhno območje zanimanja (ROI) omejilo vsakega člana na lastno prosto obdobje delovanja senzorja, ne glede na hitrost sprožilca: 4,9 ms pri ločljivosti 768 × 304, zato ni bilo mogoče posvetliti nobenega člana.
- Chloros nikoli ne skrajša ročne osvetlitve, da bi se prilagodil hitrejšemu sprožanju. Avtomatska frekvenca ostane v mejah, ki jih dopuščajo ročne osvetlitve; fiksirana frekvenca, ki je hitrejša od tiste, ki jo dopušča osvetlitev, temu elementu pusti dovolj dolg časovni limit, in sprejema sprožitve, ki niso hitrejše od tistih, ki jih dopušča njegova osvetlitev. Znižajte osvetlitev ali fiksirano frekvenco, kadar mora vsak element obdelati vsako sprožitev.
- **Povratno branje:** branje vozlišč člana prikaže `AcquisitionFrameRateEnable` True in `AcquisitionFrameRate` s hitrostjo člana; omogočanje se izklopi, ko se niz ustavi.
- **Niz je lastnik teh dveh vozlišč. Ne zapisujte `AcquisitionFrameRateEnable` ali `AcquisitionFrameRate` na član niza** — namesto tega nastavite hitrost nizahitrost z `target_fps` / `set_target_fps()`.

Izpostavljenost člana zapišite s `arr.set_member_settings(serial, exposure_time=20000)` (ključne besede so `CameraSession.set_settings()`; `chloros-cli lattice exposure --serial` deluje enako). Član ostane v nizu. Ne odpirajte člana niza s `connect_camera()`: connect zavrne kamero, ki je že v nizu, `CameraSession` pa ob prekinitvi povezave kamero odstrani iz niza. `ArraySession.status()["member_ae"]` poroča o izpostavljenosti vsakega članaizpostavljenost in ojačanje iz najnovejšega okvirja.

### En projektni zapis, vsak način

Projektna mapa vsebuje nastavitve vsake naprave: `cameras.json` — za vsako kamero (`cameras[].settings`, za posamezno kamero in za vsakega člana niza) ter nastavitve za celoten niz (`arrays[].capture`: format, ROI, združevanje slik, hitrost sprožanja, proračun za povezave) — ter `sensors.json`, nastavitve za vsak svetlobni senzor (čas integracije, povprečje okvirja, omejitev). Aplikacija, CLI, SDK in strežnik MCP vsi berejo in zapisujejo iste datoteke na enak način:

- **Naprava, priključena v okviru projekta, se shrani v ta projekt in vanj beleži vsako spremembo.** »V okviru projekta« pomeni, da je projekt odprt v ozadju (aplikacije) ali — prek `ChlorosProject` (`open_project()`), ki v ozadju določi ime lastne mape (`project_path`). Kamera, matrika ali svetlobni senzor, ki ga projekt še ne vsebuje, se ob povezavi doda v njegove datoteke (`connect_camera()`, `connect_array()`, povezava svetlobnega senzorja, CLI, aplikacija, `chloros-mcp`), skupaj z nastavitvami. Sprememba iz katerega koli načina (`set_settings()`, `ArraySession.set_member_settings()`, `set_target_fps()`, nastavitve matrike, `set_cap()`, CLI, upravljalnih elementov v aplikaciji) se trenutek kasneje shrani v datoteke tega projekta. Nastavljena osvetlitev, ojačanje ali ciljna vrednost AE na kameri se shrani kot zadržana, tako da se ob ponovnem odprtju ponovno ohrani.
- **Ob odprtju projekta se naprave nastavijo v skladu z njim**, v vsakem načinu: kamera, matrika ali svetlobni senzor, ki je priključen brez lastnih nastavitev, prevzameshranjene nastavitve projekta (`lattice cam-connect` / `array-connect` ob odprtem projektu, `chloros-cli project connect`, `open_project().connect_all()`, ponovno odprtje aplikacije). Niz, priključen pod novo identifikacijsko številko, posodobi niz, ki ga je projekt shranil z istimi kamerami; niz z drugimi kamerami se doda poleg njega.
- **Lastno shranjevanje aplikacije nikoli ne prepiše trenutnih nastavitev naprave** s svojo starejšo kopijo teh nastavitev in ohrani shranjene prekrivne nastavitve za posamezne člane, ki jih je shranil klicatelj prek CLI-ja ali SDK-ja.
- **Pikselni format, ki ga nastavite, je tisti, ki ga projekt ohrani.** Format, nastavljen v nastavitvah niza, prek `--pixel-format` ali `connect_array(pixel_format=)`, vključno z 8-bitnim, se shrani in ponovno poveže natančno. Če ni nič nastavljeno, se vsaka kamera poveže s svojo privzeto 12-bitno nastavitvijo (`BayerRG12`, `Mono12` na monokromatskem senzorju). Ime formata pomeni njegovo bitno globino; družina pa se nanaša na senzor, zato barvna kamera nikoli ni nastavljena na mono format, niti mono kamera na Bayerjev format, vrstica `[PixelFormat <serial>] ... written` pa to navaja, ko je bil format dodeljen. Izvozi 8-bitnega zajema so 8-bitni, vključno z odbojnostjo: ponovno skalirani in označeni z `PixelScale`.
- Naprava, priključena brez projekta, ne beleži ničesar.
- **Posnetek sledi projektu, ki je bil ustvarjen po njem; odprt projekt ga pusti pri svojem projektu.**Ko se ustvari projekt — `create_project()`, `chloros-mcp`-ov `create_project` ali aplikacijska funkcija**Nov projekt**— ali podvojen (funkcija**Podvoji**v aplikaciji, ki odpre kopijo), medtem ko posnetek še vedno zapisuje v projekt, ki je bil odprt prej, se posnetek nadaljuje, kar pa zapiše, se shrani v novem projektu, na istem mestu znotraj njega (spodaj). Odpiranje obstoječega projekta — `ChlorosProject.process()` (kar odpre projekt v ozadju), `chloros-cli process`, možnost**Odpri projekt** v aplikaciji — pusti snemanje še vedno v teku v projektu, v katerega se snema: tam se konča, njegove datoteke pa se pridružijo seznamu datotek tega projekta (aplikacija ustavi vsako snemanje, preden odpre projekt).
  - `ArraySession.record()` in `ArraySession.burst()`: video ali mapa z zaporednimi posnetki, ki vsebuje vsak posamezni okvir, se tja prenese v celoti, ko se snemanje ustavi. Rezultat ustavitve poroča o novi poti (`out_dir`, `video_path`), iz katere se nato sestavi zaporedje posnetkov. Če pogon zavrne premik, ostane posnetek tam, kamor je bil zapisan, kar je navedeno v rezultatu (`relocation_error`). Surovi posnetki ustavljenega zaporedja in `.daq` se dodajo na seznam datotek projekta, v katerega se uvrstijo; video se ne doda na seznam datotek projekta.
  - `ArraySession.continuous()`: vsak posnetek po spremembi se shrani v nov projekt; tisti, ki so bili shranjeni prej, ostanejo tam, kjer so.
  - `DAQSensorSession.record_start()` (in lastni posnetek svetlobnega senzorja aplikacije): `.daq` se po koncu snemanja premakne v mapo `light_sensor` novega projekta; `record_stop()` vrne novo pot, datoteka pa se doda na seznam datotek tega projekta.

  Posnetek v mapi zunaj odprtega projekta ostane tam, kjer je, `start_camera_continuous()` in večtokovne naloge pa ohranijo mapo, ki jim je bila dodeljena.
- **V aplikaciji se posnetki, narejeni po ustvarjanju novega projekta, shranijo v ta projekt.** Ustvarjanje ali podvajanje projekta ohrani vsa tekoča zajemanja in snemanja, vsak posnetek, narejen po tem (Zajemi vse, Začni zajemati), pa se shrani v nov projekt in se doda na njegov seznam datotek. Odpiranje obstoječega projekta v aplikaciji najprej ustavi vse te postopke.
- **Projekt se med snemanjem ne premakne.**Premikanje odprtega projekta v drug delovni imenik (v aplikaciji**Nastavitve → Delovni imenik**; noben klic SDK ne premakne projekta, HTTP-odjemalec, ki pošilja podatke na `/api/move-project`, pa to stori) se zavrne s statusom HTTP 409 `PROJECT_RECORDING`, medtem ko vanj pišejo: snemanje matrike ali serijsko snemanje, neprekinjeno zajemanje matrike, neprekinjeno delo kamere / večtokovno delo ali snemanje svetlobnega senzorja: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.`

To je ločeno od tega, kakšne so nastavitve kamer ob prekinitvi povezave (glej [Pomnilnik kamere](#camera-memory--keep_settings--clear_settings)): projekt sam nastavi svoje kamere ob odprtju.

### Pomnilnik kamere — `keep_settings` / `clear_settings`

Ko je niz priključen, kamere prevzamejo nastavitve niza, ki jih lahko nato spremenite. Ko program Chloros sprosti kamere — `ArraySession.disconnect()` / `force_disconnect()`, ob koncu prehodne funkcije, ustavi `start_camera_continuous()` — vsako kamero vrne v stanje, v kakršnem je bila pred priključitvijo: ožičenje sprožilca in sinhronizacijskega vodnika, nastavitve PTP in kamere — `ExposureAuto` / `GainAuto` (in ročno `ExposureTime` / `Gain`), `TargetBrightness` ter omejitve in dušenje AE, belo ravnovesje, raven črne, `PixelFormat`, združevanje slik in območje zanimanja (ROI), barvni tok (`GammaEnable` / `Gamma`, `LUTEnable`, `ColorTransformationEnable`), `ReverseX` / `ReverseY`, `UserSetSelector`, PTP-ji `PtpSlaveOnly` / `PtpDomainNumber`, velikost paketa (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`), hitrost (`DeviceLinkThroughputLimitMode` / `DeviceLinkThroughputLimit`, `GevSCPD`, `GevSCFTD`), podatki v delih (`ChunkModeActive`, vsak `ChunkEnable` pod svojim `ChunkSelector`), odstranjevanje motenj na vhodu sprožilca, pri čemer niz zagotavlja sinhronizacijsko linijo podrejenega (`LineFilterSelector` / `LineFilterWidth`) ter hitrost zajemanja okvirjev (`AcquisitionFrameRate` le, če je bila omogočitev vklopljena: če je izklopljena, se hitrost bere kot največja, ki jo dopušča ritem, ne pa kot nastavitev). Zapisano je le tisto, kar se razlikuje: kdaj se je kamera najprej ustavila, ritem in hitrost zajemanja pred osvetlitvijo, ki potrebuje njuno obdobje slike.

Za uporabo kamer zunaj projekta dva argumenta zapišeta v **lastni pomnilnik** kamere — tisto, s čimer se kamera vklopi, v kateri koli programski opremi (na `disconnect()` / `force_disconnect()`, vsaka prehodna funkcija, `start_camera_continuous()` in `ChlorosProject.disconnect_all()`; MCP: isti argumenti kot pri `disconnect_device` in prehodnih orodjih; CLI: `--keep-settings` / `--clear-settings`):

- **`keep_settings=True`** shrani nastavitve vsake kamere takšne, kot so bile ob koncu seje v njenem pomnilniku (UserSet2, ki je postal zagonski nabor). Kamere še vedno delujejo samostojno (`TriggerMode` `Off`), ožičenje sinhronizacijskega vodila pa ostane takšno, kot je bilo najdeno. Kamera brez UserSet2 shrani nastavitve v UserSet1; ta računalnik si to zapomni, zato jo Chlorosova samopopravljalna mreža — ki ponastavi UserSet1, ki se je oddaljil od Chlorosove izhodiščne vrednosti — pusti pri miru.
- **`clear_settings=True`** ponastavi pomnilnik vsake kamere na tovarniške nastavitve: Chlorosovo izhodišče, tisto, ki ga vgrajuje samopopravljanje matrike — tovarniške privzete nastavitve s `TriggerMode`, `Off`, `TriggerSource`, `Line2`, samodejno osvetljevanje ter kalibrirano belo ravnovesje RGB-kamere ohranjena — v UserSet1, ki je postal zagonski nabor.
- Ne oboje (`ValueError`; strežnik odgovori s 400). Odgovor vsebuje `camera_memory`, za vsako kamero pa `{"saved_to": "UserSet2" | "UserSet1" | None, "errors": [...]}` ali `{"cleared": bool, "errors": [...]}` — `force_disconnect()` ga vrne; za nalogo je priložen `stop()`.

Aplikacija teh ne ponuja.

**Vsak izhod kamere vrne na prvotno mesto.**Vrnitev v prvotno stanje poteka na vsaki izhodni poti: `CameraSession.disconnect()` (kamera**X**aplikacije),**X** v nizu ali kateri koli element niza (celoten niz se sprosti), zapre aplikacijo (backend ima na voljo do 20 s, da vrne vse kamere, nize in tekoče neprekinjeno snemanje; prehodna funkcija ali enkratni klic, ki še vedno teče, se ustavi na naslednji varni točki, najprej vrne kamere, v največ 8 sekundah, ter odgovori z HTTP 503 `SHUTTING_DOWN`; še vedno tekoča `connect_array()` se ustavi pred naslednjo kamero in vrne kamere, ki jih je odprla, v največ 10 sekundah ter sproži `ChlorosConnectError` (HTTP 503 `SHUTTING_DOWN`); tista pa, ki prispe med zaustavitvijo, ne odpre ničesar: HTTP 503 `SHUTTING_DOWN`), ter ukaz `chloros-cli`, ki je bil prekinjen ali katerega terminal se zapre (njegova naloga se ustavi in kamere se najprej vrnejo v prvotno stanje). Backend, ki se sesuje ali je prisilno prekinjen, ne more ničesar vrniti v prvotno stanje; ponovni zagon vrne vsako kamero v njen pomnilnik.**Projekt sam nastavi svoje kamere.** Ko se niz poveže prek projekta – odprtega projekta aplikacije ali `ChlorosProject`, ki se sam poimenuje v ozadju (`project_path`) – vsak član dobi nastavitve, ki jih je projekt shranil zanj: ekspozicijo, ojačanje ali ciljno vrednost AE, ki se zanj ohranja (ohranja se kot vaša, zato avtomatska ekspozicija ne posega vanjo) ter način uravnoteženja beline (korekcija Chloros; lastna belina kamere ostane izklopljena), ne glede na to, kaj je shranjeno v pomnilniku kamere.

### Načini zajemanja in snemalniki

Površina matrike odraža ploščo zajemanja v grafičnem vmesniku: načini posameznega / neprekinjenega / intervalnega / najhitrejšega zaklopa, poleg tega pa še dva snemalnika (kompozitni video v živo in serija surovih posnetkov → ponovna obdelava brez povezave).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Continuous — every synced group for 60 s, saved by the backend itself
    with arr.continuous("run1/", processing="raw", duration_s=60) as run:
        run.wait()
    print(run.result["captures"], run.result["captures_per_s"],
          run.result["stop_reason"])

    # Combined-index video/GIF recorder: the GUI's combined view, or with no
    # GUI open the array's NDVI (pass expr="..." for another index)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    if rec.result and rec.result.get("error"):   # leaving the block does not raise
        print("recording failed:", rec.result["error"])
    else:
        print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**je zanka neprekinjenega/intervalnega snemanja v SDK-ju. Ker ni `Ctrl+C`, s katerim bi jo lahko prekinili iz skripta,**morate** posredovati `count` in/ali `duration_s` (zank se ustavi, ko je dosežen kateri koli od njiju). `interval_s` se meri od začetka vsakega prehoda (v skladu z grafičnim vmesnikom). Neprekinjeni prehodi (`interval_s=0`) povezujejo `group_seq` → `after_seq`, zato so zaporedne sinhronizirane skupine v hitrosti niza. Preostali kwargi se neposredno prenesejo v `capture()`.
- **`continuous`**je lastno neprekinjeno zajemanje backenda — način »Continuous« v grafičnem vmesniku.**Uporabite ga namesto `capture_repeated()`, če želite, da se vsak sprožilec shrani na disk z visoko hitrostjo posnetkov:** `capture_repeated()` iz vašega skripta pošlje eno HTTP-zahtevo na skupino, medtem ko `continuous()` ohranja dva zajema v teku znotraj backenda brez povratnega prenosa med skupinami. Merjeno pri ROI 512 na nizu 9 kamer: 73–77 zajemov/s, v primerjavi z nizom, ki objavlja približno 78. `capture_repeated()` ohranite za intervalni zajem (`interval_s`) in za prehode, ki jih pregledujete enega po enega. V sistemu Windows izključite backend iz programa Microsoft Defender, da dosežete visoke hitrosti (glejte [Korak 1](#step-1--install-the-chloros-platform-package)).
- **`record`** je *za namene spremljanja*: shranjuje renderirano kombinirano indeksno kompozicijo v realnem času. Ko je odprt združen pogled v grafičnem vmesniku, ta posnetek zajame prikazani pogled. Če ni odprt noben pogled, backend sam renderira pogled: `expr`, če ga posredujete, sicer pa NDVI matrike (rdeči in NIR pasovi, najbližji 660 nm in 850 nm, kot jih zajame pogled v živo), ko je matrika poravnana. Pasove v `expr` poimenujte tako, kot jih poimenuje matrika: `<band>_<filter>` za kamero M3C (`NIR_850_RGN`, `Red_660_RGN`, `Orange_615_OCN`), `Mono_<filter>` za kamero M3M (`Mono_850`); kratko ime barve (`NIR`, `Red`) deluje le, če ga ima en pas matrike. Pogled GUI, odprt med snemanjem, se snema, dokler ostane odprt. Snemanje, ki ne zapiše nobenega okvirja, se ne uspe z napakami `error`, `"no-frames: …"` (matrika morda še vedno usklajuje ali stabilizira svojo osvetlitev), in se sproži napaka `stop()` (glej [`RecorderHandle`](#recorderhandle)). En sestavljeni snemalnik na matriko (sproži napako, če eden že teče).
- **`burst` → `build_video`** je *za analizo*: `burst` zapiše surove okvirje + manifest za vsak okvir + en `.daq` za vsak posamezen DLS-odčitek pod `<output>/bursts/<base>/` pri polni hitrosti zajemne zanke (brez verige, brez exiftoola, brez živega predogleda). `build_video` časovno uskladi vsak posnetek z najbližjim `.daq` in ponovno izvede verigo svetilnosti/odbojnosti/indeksa. `products` je seznam `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (privzeto: združen indeks). `burst().stop()` prav tako samodejno sproži izgradnjo združenegaindeksa, ki se v končnem rezultatu vrne kot `build_job`.

##### Rezultat `build_video`

| Ključ | Pomen |
| --- | --- |
| `success` | `True` le, če je bil vsak zahtevani izdelek zgrajen v skladu z zahtevo: vsaj en izhod, nič v `skipped`, in ni v `degraded`. |
| `outputs` | Poti do posnetih videoposnetkov (in GIF-ov). |
| `errors` | Težave, v obliki besedila. |
| `skipped` | En `{"serial", "level", "reason"}` na vsak zahtevan izdelek, za katerega **ni** bil ustvarjen videoposnetek — zaradi (ni kalibracijskega paketa za to kamero, ni DLS-odčitka za odbojnost, napaka v verigi obdelave) ali kombiniranega indeksa, ki vsebuje `"kind": "combined"` in `serial` `None` (med kamerami ni para pasov rdeče/NIR, ni skupine s sevanjem za oba pasova). Vsak vnos ustvari `success` `False`; ostali videoposnetki se še vedno zapišejo in navedejo. |
| `substitutions` | Kamere so bile upodobljene na drugačni ravni, kot je bila zahtevana, in razlog za to — RGB-kamera, za katero je bil zahtevan sevalni/odbojni koeficient, dobi svoj prikazni posnetek, z uravnoteženo belino, če je na voljo kalibracijski paket kamere. To ni napaka. |
| `notes` | Informativno besedilo, ne napake — npr. kamera brez indeksnega izraza za posamezno kamero nima indeksnega videoposnetka ali pa prikazni videoposnetek RGB-kamere nima uravnotežene beline (ni kalibracijskega paketa ali pa ta ne vsebuje tovarniške uravnotežene beline), pri čemer je navedena kamera. To ni napaka. |
| `degraded` | `True`, kadar je bil izdelek zapisan, vendar ni tisti, ki je bil zahtevan: kombinirani indeks v načinu brez povezave je vgrajena normalizirana razlika med rdečo in NIR, zato se konfigurirani indeksni izraz ne izračuna. Povzroči `success` `False`. |
| `combined` | Povzetek kombiniranega indeksa: uporabljeni pasovi, obseg LUT, zgrajeni okvirji. |
| `clamp_masks` | Poti [mask za omejevanje mešanja](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) za vsakposameznega okvirja (`save_tiffs=True`), v `clamp_masks/<level>/` poleg mape `<level>/` s TIFF-datotekami, po ena na TIFF-datoteko, v kateri je bil piksel omejen. Video ne vsebuje nobene. |
| `clamp_fractions` | Kaj video navaja namesto maske: en `{"kind", "serial", "level", "frames"}` na video izdelek (`kind` `"combined"` za kamere, vključene v združen indeks), `frames` seznam `[frame, R, G, B]` — delež pikslov v tem okviru, ki jih je razmešavanje omejilo v pasu, ki ga zaseda njegov kanal R / G / B (`Chloros:Band{R,G,B}UnmixClampFraction`). |

#### `RecorderHandle`

Vrnejo ga `ArraySession.record()` in `ArraySession.burst()`. Uporabite ga kot upravitelja konteksta za samodejno ustavitev ob izhodu iz obsega ali ga upravljajte ročno.

| Član | Opis |
| --- | --- |
| `job_id` | ID naloge v ozadju (niz). |
| `kind` | `"composite"` (iz `record`) ali `"raw"` (iz `burst`). |
| `start_stats` | Slovar, ki ga vrne klic `start`. |
| `result` | `None` med izvajanjem; končni slovar rezultatov po zaustavitvi. |
| `stats(timeout=10.0)` | Statistični podatki o tekočem opravilu v realnem času (zapisani okvirji, doseženi fps, pretekel). |
| `stop(timeout=60.0)` | Zaustavitev snemalnika; vrne in shrani končni rezultat (`.result` ostane `None`, če je zahteva za zaustavitev sama po sebi spodletela). Za snemanje `record()`, ki je spodletelo — tisto, ki ni zapisalo nobenega okvirja (`error` `"no-frames: …"`) — shrani rezultat v predpomnilnik na `.result`, nato pa sproži `ChlorosConnectError`. Idempotentno (drugi klic vrne rezultat iz predpomnilnika, brez sprožitve). Če pustite blok `with`, se snemalnik ustavi brez sprožitve napake, zato po njem preverite `rec.result.get("error")`. |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

#### `ContinuousHandle`

Vrne ga `ArraySession.continuous()`. Uporabite ga kot upravitelja konteksta — zapustitev bloka `with` ustavi zajem, če ta še vedno teče — ali ga upravljajte s `status()` / `wait()` / `stop()`. Hkrati teče le eno neprekinjeno zajemanje na niz.

| Član | Opis |
| --- | --- |
| `start_status` | Slovar stanja, ki ga vrne klic za zagon. |
| `result` | `None` med izvajanjem; končni slovar stanja po koncu zajema (nastavi ga `stop()`, ali s klicem `status()` / `wait()`, ki je zaznal konec). |
| `status(timeout=10.0)` | Slovar stanja v realnem času (polja spodaj). |
| `wait(timeout=None, poll_s=0.5)` | Blokiraj, dokler se zajem ne konča samodejno — dosežen `max_captures` ali `duration_s`, izpraznjen niz, poln disk, napaka — in vrni končno stanje. Z `timeout` vrni najnovejši status po toliko sekundah, tudi če še vedno teče: preveri `running`. |
| `stop(timeout=30.0)` | Ustavi zajem in vrne njegovo končno stanje. Idempotentno (drugi klic vrne rezultat iz predpomnilnika). |

Polja stanja:

| Polje | Pomen |
| --- | --- |
| `running` | True, dokler zajem poteka. |
| `captures` | Sinhronizirane skupine so shranjene. |
| `files` | Datoteke so zapisane. |
| `failed_files` | Datoteke, ki jih ni bilo mogoče zapisati (kot v [`CaptureResult.failed`](#datoteke-ki-jih-ni-bilo-mogoče-zapisati--zajem-ni-uspel)). |
| `failed_captures` | Poskusi zajemanja, ki so v celoti spodleteli, vključno s tistimi, pri katerih je bilo zajemanje ponovljeno (HTTP-napaka 503, ker niz ni pravočasno prejel nove skupine). |
| `skipped_groups` | Skupine, ki jih je matrika objavila, a niso bile shranjene (zajemanje je zaostalo). |
| `captures_per_s` | Povprečna stopnja shranjevanja od začetka. |
| `elapsed_s` / `started_at` | Število sekund delovanja do sedaj; čas začetka (sekunde Unixove dobe). |
| `last_group_seq` | Najnovejša sinhronizirana skupina, ki je bila shranjena. |
| `last_error` | Zadnja napaka ali `None`. |
| `stop_reason` | Ko se je končalo: `count` (doseženo `max_captures`), `duration` (doseženo `duration_s`), `stopped` (poklicano `stop()`), `array_gone` (niz je bil prekinjen), `error` (20 zaporednih napak ali zahteva, ki jo je backend zavrnil), `disk_full` (datoteka, za katero na disku ni bilo prostora). `None` med delovanjem. |
| `max_captures` / `duration_s` | Meje, s katerimi se je zagnal (`0` = nobene). |
| `defender_cores` | Jedra procesorja, ki jih je Microsoft Defender uporabil v zadnjih 10 sekundah; `None`, dokler ni izmerjeno nobeno, in vedno `None` na gostitelju brez Defenderja. |
| `defender_warned` | Resnično, ko zajem zabeleži, da ga Defender upočasnjuje (na 1 jedru ali več). Izključite backend iz skeniranja Defenderja — glejte [Korak 1](#step-1--install-the-chloros-platform-package). |

```python
run = arr.continuous("captures/run1", processing="raw", max_captures=5000)
try:
    while True:
        st = run.wait(timeout=5)              # returns every 5 s while running
        print(f"{st['captures']} captures, {st['captures_per_s']:.1f}/s, "
              f"{st['skipped_groups']} skipped")
        if not st["running"]:
            break
finally:
    final = run.stop()                        # no-op once it has ended
if final["failed_files"] or final["stop_reason"] in ("disk_full", "error"):
    raise SystemExit(f"continuous capture: {final['stop_reason']}: "
                     f"{final['last_error']}")
```

### Živi okvirji v vašem lastnem potoku

Če želite sinhronizirane okvirje posredovati v svoj lasten kod – strojno učenje, prilagojen pregledovalnik, lasten snemalnik –, uporabite `ArraySession.frame_stream()`. Vsak tick ustvari `GrabResult`: `{serial: numpy array}` za vsako kamero, **vse iz enega sprožilca**, z identifikatorjem tega sprožilca v `.group_id`. Gre za enako implementacijo kot pri projektnem API-ju `ArrayHandle.frame_stream()`, zato vse pod [En sprožilec na branje](#one-trigger-per-read--group_id), [`band`](#exporting-a-single-band--band), [`align`](#usklajevanje-tokov--usklajevanje) in [`bit_depth`](#bitna_globina--izmenjava-natančnosti-za-pasovno-širino) velja za oba.

```python
import chloros_sdk
from chloros_sdk import FrameFetchError

serials = ["213800234", "214000533", "214701288", "214701292"]   # master first

with chloros_sdk.connect_array(serials) as arr:
    try:
        for frames in arr.frame_stream(processing="raw", fps=10, duration=300,
                                       with_metadata=True):
            if not frames:              # no camera answered this tick
                continue
            if frames.errors:           # cameras missing from this trigger
                print("left out:", {s: str(e) for s, e in frames.errors.items()})
            for serial, (img, hdr) in frames.items():
                # img: numpy array (raw: one Bayer or mono plane)
                # hdr: group_id, frame_id, timestamp_ns, exposure_us, gain_db, ...
                my_pipeline(frames.group_id, serial, img, hdr["timestamp_ns"])
    except FrameFetchError as exc:      # the stream cannot continue
        print("stream ended:", exc.status, exc)
```

- **Vsak izdan niz predstavlja nov sprožilec.** `frames.group_id` se v toku nikoli ne ponovi; ko zahtevate več `fps`, kot jih niz obdeluje, tok počaka na naslednji sprožilec, zato teče s hitrostjo, ki je nižja od obeh.
- **Manjkajoče kamere so poimenovane, ne pa nadomeščene.** Kamera, ki ob tem sprožilcu ni mogla biti obdelana, ni vključena v slovar in je navedena v `frames.errors`.
- **Konča se glasno.** `FrameFetchError` se ob zavrnitvi ponovi pri naslednjem taktu, po 60 sekundah brez okvirja ali po 60 sekundah brez novega sprožilca.**Kaj ponuja.**`frame_stream()` prenaša cele numpy-okvirje prek lokalne HTTP-povezave in doseže največjo hitrost blizu**170 MB/s**. To zadostuje za obdelavo surovih okvirjev v realnem času ali za obdelavo združenih ali ROI-nizov, vendar ne za vsak sprožilec pri polni ločljivosti — da bi ohranili**vsak** sprožilec, shranite na disk z [`continuous()`](#continuoushandle). Izmerjeno na nizu štirih kamer (FOCN, FRGN, FRGB, F450) s stisnjenim prenosom; vsak tick je bil popoln niz enega sprožilca:

| | Celoten senzor (niz 14,2–14,4 fps) | združevanje 2 × 2 (niz ~33 fps) |
| --- | --- | --- |
| `frame_stream(processing="raw")` | 6,7 niza/s | 22,2 niza/s |
| `frame_stream(processing="debayered")` | 2,3 niza/s | 9,3 niza/s |
| `frame_stream(processing="display")` | 0,8 niza/s | 3,0 niza/s |
| MJPEG na kamero, 4 tokovi pri ločljivosti 720p | 13,5 sličic na sekundo za vsakega; vsaka sličica se ujema s sprožilcem | ~25 sličic na sekundo za vsakega; ~45 % sprožilcev prisotnih na vseh 4 |

Uporabite `processing="raw"` za prepustnost in demosaik na vaši strani; raven `debayered` prav tako odstrani temni odmik kalibracijskega paketa, kar demosaic `raw` ne stori.

#### MJPEG-tokovi na kamero (brez RTSP)

**RTSP ni podprt.** Vmesnik za pretakanje je MJPEG prek HTTP-a (`multipart/x-mixed-replace`, en JPEG na del, vsak del z `Content-Length`), en URL na kamero:

| URL | Kaj ponuja |
| --- | --- |
| `GET /api/camera/stream/<serial>` | 8-bitni predogled, do ~30 sličic na sekundo. Za član matrice `max_h` (privzeto 720, 64–2160) in `jpeg_q` (privzeto 80) določata velikost in kakovost. |
| `GET /api/camera/<serial>/stream-annotated` | Enak predogled z narišenimi grafičnimi prekrivki (zebra, mreža, križec, histogram, peaking, točkovni merilnik). `ChlorosLocal.camera_stream()` ga prebere za vas. |
| `GET /api/camera/<serial>/stream-reflectance` | 8-bitna odbojnost na pas — glej [Tok odbojnosti v realnem času](#live-reflectance-stream-mjpeg). |

- **Zagonski glavi.** Za element matrike vsak del vseh treh tokov vsebuje `X-Chloros-Group-Id`, `X-Chloros-Frame-Id` in `X-Chloros-Timestamp-Ns`. Deli iz različnih kamer z istim `X-Chloros-Group-Id` predstavljajo isti sprožilec: zberite dele in jih združite na podlagi tega glava, da ponovno sestavite niz. Samostojna kamera nima skupine: njeni predogledni deli ne vsebujejo nobenega od teh glav (njeni deli za odbojnost nosijo prazno oznako `X-Chloros-Group-Id`).
- **Tok predogleda izpušča okvirje pod obremenitvijo.** Prikaže najnovejši okvir in preskoči vse tiste, ki jih ni uspel poslati pravočasno, zato pri visokih hitrostih matrike ne doseže vsak sprožilec tok vsake kamere (glej zgornjo tabelo). Za vsak sprožilec uporabite `frame_stream()` ali `continuous()`.
- **`/api/camera/stream/<serial>` se zaustavi, medtem ko je odprt združen živi tok niza** (`/api/camera/array/stream/<array_id>` — odpre ga pogled na niz v grafičnem vmesniku). Tok vsakega člana nato pošlje en nadomestni okvir »Paused (array stream active)« brez glav `X-Chloros-*`, in ničesar več, dokler se združen tok ne zapre; nato se samodejno nadaljuje. `stream-annotated` in `stream-reflectance` se ne zaustavita. Zaprite pogled na niz v grafičnem vmesniku, ko skript bere predogledne tokove posameznih kamer.
- **Za številke, ne za slike, uporabite `frame_stream()`.**JPEG je format z izgubo in 8-bitni; `frame_stream()` vrne iste izdelke brez kodeka v poti.

### Intervalno zajemanje za raziskave z droni

Raziskovalni polet potrebuje eno sinhronizirano skupino slik na vsakih nekaj metrov poti, ne pa vsakega posnetka, ki ga lahko ustvari matrika. To nadzorujeta dve nastavitvi:**`target_fps`**določa, kako pogosto kamere zajamejo posnetek,**`capture_repeated(interval_s=…)`** pa določa, kako pogosto se posnetek shrani.**Kamere med shranjevanji še naprej snemajo.** Pri vsakem intervalu se shrani najnovejša sinhronizirana skupina; posnetki vmes se ne zapišejo. Samodejna osvetlitev se prilagaja pri vsakem posnetku, zato je osvetlitev ob shranjevanju že ustaljena. Ni načina, v katerem bi se kamere med posnetki ustavile in se za vsakega posebej ponovno aktivirale: vsak posnetek bi namreč moral čakati, da se začne prenos in da se osvetlitev ustalijo.**Fiksirajte hitrost snemanja glede na potrebe intervala.**Če pustite nastavitev na avtomatsko, sistem deluje z najvišjo hitrostjo, ki jo omogočata gostitelj in omrežje. Z znižanjem hitrosti zmanjšate segrevanje kamer, obremenitev omrežja in porabo procesorske moči gostitelja, ne da bi spremenili vsebino shranjenih posnetkov. Izberite hitrost, ki omogoča**2–4 posnetka na interval**:

- Vsako shranjevanje počaka na naslednji posnetek po tem, ko je zahtevano, zato se shranjena skupina konča do enega obdobja posnetka po načrtovanem trenutku. Višja frekvenca ohranja ta razmik majhen.
- Samodejna osvetlitev se prilagaja enkrat na posnetek, zato zelo nizka frekvenca upočasni, kako hitro osvetlitev sledi mimo letajočemu oblaku.

**Interval izračunajte na podlagi načrta leta:**

```
spacing    = along-track footprint × (1 − forward overlap)
interval_s = spacing / ground speed
```

Na primer, 60-metrski obseg snemanja pri 80-odstotnem prekrivanju v smeri naprej pomeni razmik 12 metrov; pri hitrosti 10 m/s je to `interval_s=1.2`, fiksirana vrednost `target_fps=3` pa pomeni med 3 in 4 sličicami na interval. Snemanje poteka na podlagi časa: SDK-ju ni načina, ki bi se sprožil na podlagi razdalje ali GPS-a, zato na merilnih linijah vzdržujte konstantno hitrost glede na tla.

**Surove podatke shranite na letalu in jih kalibrirajte kasneje.** `interval_s` se meri od začetka vsakega shranjevanja, zato shranjevanje, ki traja dlje od intervala, povzroči, da se naslednje začne z zamudo in da se razmik spremeni. Shranjevanje surovih podatkov skupaj z odčitkom svetlobnega senzorja (`processing="raw"`; odčitek se shrani privzeto) zagotavlja, da je vsako shranjevanje kratko in da je vsak posnetek kasneje pretvorljiv v odbojnost.

```python
import chloros_sdk

INTERVAL_S = 1.2      # from the flight plan
FLIGHT_S = 20 * 60    # bounds the loop; it also stops at count= if given

with chloros_sdk.connect_array(serials, target_fps=3.0) as arr:
    groups = arr.capture_repeated(
        "survey/", interval_s=INTERVAL_S, duration_s=FLIGHT_S,
        processing="raw", render_index=False,
        on_capture=lambda i, r: print(f"group {i}: {len(r)} frames"))
```

- **`connect_array(target_fps=…)`**fiksira hitrost od začetka;**`arr.set_target_fps(fps)`**jo spremeni med letom brez ponovne povezave,**`arr.set_target_fps(None)`** jo vrne na avtomatsko nastavitev. Hitrost, ki presega zmogljivosti gostitelja in omrežja, se zniža na to zgornjo mejo — preverite `clamped` v rezultatu — najnižja hitrost pa je 0,1 fps.
- **Ne uporabljajte `burst()` za redčenje okvirjev.** Zapisuje vsak okvir, ki ga ustvari niz; sledi fiksirani vrednosti `target_fps`, zato je primerno orodje le, če želite vsak okvir.
- **`target_fps` ne popravi poškodovanih okvirjev.** Znižanje te vrednosti zmanjša število proizvedenih okvirjev, ne pa hitrosti pošiljanja posameznih okvirjev; za okvirje, poškodovane v GVSP, namesto tega znižajte vrednost `wire_ceiling_mbps` (glejte [Stanje niza](#array-health--which-subsystem-is-losing-frames)).

```python
r = arr.set_target_fps(5.0)
if r["clamped"]:
    print(f"asked for {r['requested']} fps, running at {r['target_fps']}")
```

### Priključitev na že povezano matrico — `attach_array`

Če je matrica že vzpostavljena (ga je odprl grafični vmesnik ali ga je prejšnja seja SDK poklicala `connect_array`), uporabite `attach_array`, da pridobite ročaj do njega, namesto da se ponovno povežete. `connect_array` v takšni situaciji vedno vrne napako `Camera <sn> is already in array <id>`, saj pošiljanje zahtevka POST za `/array/connect` za člana v bazenu ni idempotentno; `attach_array` prebere `/api/camera/array/list` in ga primerja bodisi po array_id bodisi po serijskih številkah.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Vzorec: Skripti SDK, ki delujejo skupaj z namiznim grafičnim vmesnikom, naj najprej poskusijo z `attach_array` in se zatečejo k `connect_array`, če v bazenu še ni nobenega niza.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Priključen niz je izposojen, ni v lasti.** Ročaj iz `attach_array` nikoli ne razgradi niza: niti izhod iz bloka `with` niti eksplicitni `disconnect()` ga ne sprosti (eksplicitni klic natisne eno vrstico v stderr, ki pravi, da je niz ostal v teku). Enako pravilo velja za `CameraSession` / `DAQSensorSession`. Zato je souporaba z grafičnim vmesnikom varna z ali brez `with`:
>
> ```python
> with chloros_sdk.attach_array(serials) as arr:
>     arr.capture("output/", processing="reflectance")
> # … the array stays up for the GUI
> ```
>
> Ko ga resnično želite zaključiti – na primer, da se ponovno povežete z drugim `wire_ceiling_mbps` – pokličite `arr.force_disconnect()`. Ponovna povezava po navadnem `disconnect()` na priključenem ročaju se ne uspe, saj se prikaže sporočilo `Camera <sn> is already in array <id>`, ker ni bilo nič sproščeno.

### Pomočnik za analizo omrežja

Koristno pred odprtjem niza – predvideva, ali bodo vaše predlagane nastavitve ustrezale:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    # width / height are sensor pixels BEFORE binning (what connect_array
    # takes with binning=); out_width / out_height is what is streamed.
    print(f"Connect with width={r['width']} height={r['height']} "
          f"binning={r['binning']} ({r['out_width']}x{r['out_height']} streamed)")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

```python
analyze_array_network(
    master_serial,
    slave_serials=None,
    width=2048, height=1536,              # sensor pixels before binning
    pixel_format=None,                    # None = what connect applies (Mono12 / BayerRG12)
    binning=1,                            # 1 | 2 | 4
    force_tier=None,
    packed="auto",                        # "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",
    timeout=30.0,
) -> dict
```

Nikoli ne zažene strežnika: tisti, ki ne odgovori, sproži napako `ChlorosConnectError`, medtem ko se zavrnjena zahteva vrne kot rezultat. Nepravilna napaka `packed` sproži napako `ValueError`.

`status` je eden izmed naslednjih: `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (v nasprotnem primeru `error` — vključno z zavrnjeno zahtevo: niste prijavljeni, `error_code: AUTH_REQUIRED`, ali načrtom pod spodnjo mejo SDK, `PLAN_UPGRADE_REQUIRED`). `auto_capped_fps` pomeni, da zahtevana ločljivost ustreza RX-obroču le pri omejeni sprožitveni frekvenci — ohranite ločljivost in preidite s `target_fps=result["recommended"]["recommended_target_fps"]` na `connect_array` (glejte [Primer 6](#6-capability-probe-before-connecting-a-4-cam-array)). `auto_shrunk` se prikaže le, kadar zahtevanega okvirja ni mogoče dostaviti pri nobeni hitrosti okvirjev; povezava sama po sebi nikoli ne skrči okvirjev, zato `recommended` uporabite sami.

**Kako brati projekcijo** (isti model kot v oknu »Nastavitve niza« v grafičnem vmesniku):

- **Burst (`frame_bytes_total`) se sešteva po posameznih kamerah v dejanskem formatu pikslov vsake kamere.**Mono**M3M**kamere prenašajo Mono12 (2 B/px) ali, z uporabo `pixel_format`, mono format svoje globine (`"BayerRG8"` je `Mono8`, 1 B/px), torej brez tega je polno-resolucijski okvir s 4 kamerami**~25 MB** s tremi mono kamerami, ne pa ~12,6 MB, kot predvideva predpostavka o 8-bitni globini. Backend določi format vsake kamere na podlagi njenega modela.
- **Admittance (`burst_fits_nic_ring`) upošteva izpraznjevanje**, ne pa na razliko med celotnim izbruhom in obročem: sim-emit se prilagodi, ko gostitelj prazni sprejemni obroč hitreje, kot ga kamere polnijo. 10G gostitelj + 1 GbE kamere**dopuščajo** polno ločljivost tudi takrat, ko izbruh presega obroč; 1 GbE gostitelj pa blokira (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` je konzervativna zgornja meja za serijsko pridobivanje** — `max(readout+emit, N×emit)` z oddajo na kamero, omejeno na 1 GbE povezavo kamere, neodvisno od osvetlitve. Npr. ~2,8 sličic na sekundo za niz 4 kamer s polno ločljivostjo in 12-bitno globino barv (ustreza izmerjenim ~2,7–3,0 v izvedbenem okolju). Celoten model: [Referenca CLI→ Model fps in serijskega zajema za niz kamer](cli-reference.md#array-fps--burst-model).
- **Po neprekinjenem zajemanju projekcija navaja, kakšno hitrost je ohranila.** Zajemanje, ki traja 30 s ali več pri avtomatski hitrosti, zabeleži hitrost, na kateri se je ustalilo pri tej velikosti matrike na tem računalniku; kratki poskusi krmilnika hitrosti z višjo hitrostjo se ne upoštevajo. Od takrat naprej v vrstici `array-connect`', analiza `connect_array()` in okno »Nastavitve matrike« navajata to hitrost, kadar je nižja od tiste v modelu, skupaj z razlogom za omejitev v `cycle_limiter`: `"clock"` (niz je deloval s hitrostjo, na kateri se je ustalil, zanka zajemanja pa je čakala na svoj sprožilni tak), `"loop"` (zanka zajemanja ni imela več časa), `"capture"` (zajemanje je zaostajalo in preskočilo skupine) ali `"measured"` (ni navedeno, kateri primer). Podatek modela je v `wire_fps_max`. Niz še vedno naraste do vrednosti, ki jo prenese vodilo.
- **Prekomerna obremenitev (`oversubscribed: true`) pomeni, da vodilo ne more prenesti niza pri nobeni hitrosti, ki jo dosežejo kamere** — polja fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) kažejo vrednost 0, binning pa tega ne more popraviti (zmanjša število bajtov na sliko, ne pa števila bajtov na sekundo pri določenem ritmu). Sam po sebi počasen kabel tega ne povzroči: namesto tega se zmanjša hitrost slikanja, če pa kamer ni mogoče nastaviti na dovolj počasen ritem, se stopnja spremeni v `slip-emit-and-capture` (serijska). Rešitve so manj kamer, jumbo okvirji ali hitrejša mrežna kartica; `max_cams_collision_safe` poroča o zgornji meji za vašo povezavo in kamere. Odgovor vsebuje tudi `aggregate_demand_bps`, `collision_safe_ceiling_bps` in `per_cam_floor_bps`. Glejte [Prekomerna naročnina](#over-subscription-the-per-cam-floor).

### Odkrivanje in seznam

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

Vsaka sprejema le ključne besede `backend_url=` in `timeout=30.0` ter vrne seznam slovarjev. `discover_lattice_cameras()` (`serial`, `ip`, `mac`, `model`, `lattice_model`, `vendor`, `connected`, `array_id`, …) sprejme tudi `auto_start_backend=True`; klici `list_*` nikoli ne zaženejo ozadja.

---

## Smart-AE / Smart-Capture

Matrice LATTICE izvajajo neprekinjeno avtomatsko ostrenje (AE) v ozadju takoj, ko so priključene, vendar pa novo usmerjena scena potrebuje nekaj časa, da se stabilizira. **Smart-capture** je priročna rešitev: preverja osvetlitev vsake kamere, počaka, da se niz stabilizira v celotnem oknu, nato pa sproži zajem. Deluje enako kot v grafičnem vmesniku: gumb »smart« za zajem v namizni aplikaciji pokliče isto končno točko ozadja.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Pri upravljanju prek `ChlorosProject` (naslednji odstavek) imate na voljo več nastavitev:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Pravilo pametne avtomatske ekspozicije (smart-AE) je privzeto konzervativno. Za zahtevno radiometrično delo zategnite `exposure_tolerance_pct`; za hitro spreminjajoče se prizore, kjer želite le »dovolj natančno«, ga sprostite.

---

## Seje senzorjev DAQ

Trajni sklop strežnikov za spektralne senzorje (DAQ-U prek USB, DAQ-M prek BLE, DAQ-E prek etherneta). Odraža površino kamere: pametno zaznavanje, ponovna uporaba sklopa, idempotentno priključevanje.

### Pametno zaznavanje (brez konfiguracije)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Prednost: Ethernet → BLE → USB. Posredujte kateri koli eksplicitni namig, da določite transportni protokol.

### Določen transportni protokol

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# A specific sensor by id (as list_daq_sensors() shows it; ':' or '-' alike)
daq = chloros_sdk.connect_daq_sensor(sensor_id="CB-7C-A7-5A-73")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

`sensor_id=` imenuje en senzor in odpre se izključno ta senzor. Primerja se z že odprtimi senzorji, nato se od DAQ-E, ki ga omrežni skener zazna le po IP-naslovu, zahteva (samo za branje) njegovo ime, MAC-naslov in serijsko številko, tako da ga najde tudi kateri koli od njih), BLE MAC naslova naprave DAQ-M ter identifikatorja, ki ga naprava DAQ-U ali DAQ-M sporoči ob odprtju (vsaka druga naprava, ki se odpre za preverjanje, se ponovno zapre). Če se ne najde, `connect_daq_sensor` sproži `ChlorosConnectError` z navedbo najdenih podatkov; skupaj z naslovom (`port=` / `mac=` / `eth_host=`), ki vsebuje drug senzor, se prav tako sproži. Nikoli ne preklopi na kateri koli senzor, ki je viden — za to je namenjen klic z zastavico nič.

### `connect_daq_sensor()` Podpis

```python
connect_daq_sensor(
    *,                                    # every argument is keyword-only
    sensor_id=None,                       # one named sensor (see above)
    transport=None,                       # "usb" | "ble" | "eth"; None = smart-detect
    port=None,                            # DAQ-U serial port ("COM3", "/dev/ttyUSB0")
    mac=None,                             # DAQ-M BLE MAC (implies transport="ble")
    eth_host=None,                        # DAQ-E hostname or IP (implies transport="eth")
    integration_time=None,                # ms; None = the sensor's own
    frame_avg=None,
    enable_ae=None,
    cap_id=None,                          # cap correction: "sunshine_cosine", "fov_30", ..., "none" (bare)
    start_streaming=True,
    backend_url="http://127.0.0.1:5000",
    timeout=60.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> DAQSensorSession
```

Senzor, ki je že v naboru, se vrne tak, kot je (izposojen): `integration_time`, `frame_avg` in `enable_ae` se nanj ne uporabljajo. Najprej ga odklopite, da ga odprete s svojimi nastavitvami. Izjema je `cap_id`: na izposojenem senzorju se nastavi z `set_cap()`. Če se to izpusti, velja privzeta vrednost strežnika (glejte opombo pod tabelo metod).

### Metode `DAQSensorSession`

| Metoda | Opis |
| --- | --- |
| `status(timeout=10.0)` | Povzetek vnosa v zbirko (stanje pretakanja/snemanja, valovno območje, kalibracijski sha, čas integracije, frame_avg, stanje AE). Za DAQ-E, katerega tok je utihnil: `reconnect` (če je avtomatska ponovna povezava backenda še vedno aktivna: `state`, `attempts`, `next_attempt_in_s`, `last_error`; `None` med prenosom), `stream_lost_s`, `reconnects` in `last_reconnect` ter `record_gaps` za snemanje, ki je v teku; `rx_stream_fault` označuje stanje ponovne povezave, dokler ne prispe prvi spektralni okvir. |
| `latest(*, n=1, timeout=10.0)` | Vrne do N najnovejših spektralnih okvirjev (vsak `ts_ns`, `spectrum`, `is_saturated`, `integration_time`, `x`, `y`, `z`). |
| `stream_start()` / `stream_stop()` | Nadaljuj / zaustavi pretakanje (ročaj ostane odprt). |
| `record_start(*, output_dir=None, device_name=None, timeout=15.0)` | Začni snemanje datoteke .daq. Vrne pot do datoteke. Zavrne se za DAQ-U/M brez kalibracijskega paketa AWS (DAQ-E je izvzet). |
| `record_stop()` | Ustavi snemanje. Vrne `{path, rows, warning, gaps}`; `warning` je `None` za nepoškodovano snemanje, v nasprotnem primeru pa diagnozo backenda (na primer prazna datoteka ali vrzel v toku), ki se prav tako izpiše na stderr. `gaps` navaja vsak odsek, v katerem tok ni prinesel ničesar: `start_utc`, `end_utc` (`None`: še vedno izgubljen ob ustavitvi), `duration_s`, `reconnected`, `marked_in_file`. Pri neprekinjenem snemanju je ta stolpec prazen. |
| `set_cap(cap_id, timeout=15.0)` | Nastavi profil korekcije zgornje meje, ki se uporablja za odčitke od naslednjega naprej: `"sunshine_cosine"`, `"fov_15"` ... `"fov_90"` ali `"none"` (ali `None`) za goli senzor. Vrne povzetek senzorjapovzetek senzorja, ki ga `cap_id` ponovno prebere. DAQ-E-S ohranja vrednost `as_recorded` ne glede na zahtevo (vgrajen difuzor). Poveča vrednost `ChlorosConnectError` za profil, ki se ne naloži. CLI: `chloros-cli daq pool-set-cap`. |
| `enable_ptp(timeout=15.0)` | Samo DAQ-E: ponovno vklopi PTP za enoto, pri kateri je PTP izklopljen (`status()["rx_ptp_enabled"]` je `False`; nastavitev je shranjena na enoti, zato ostane izklopljen, dokler se ponovno ne vklopi). Vrne posodobljeno stanje. Sproži `ChlorosConnectError` za DAQ-U / DAQ-M ali kadar naprava ne potrdi. Program Chloros naprave nikoli samodejno ponovno ne vklopi, saj je lahko izklop nameren. CLI: `chloros-cli daq pool-enable-ptp`. |
| `device_info(timeout=15.0)` | Samo za DAQ-E: lastne nastavitve naprave. `firmware` (`running`, `bundled`, `update_available`), `ip` in `ip_mode` (`dhcp` / `linklocal` / `static`: kar trenutno teče), `network` (shranjena nastavitev: `mode`, ter `ip` / `mask` / `gateway` v mirovanju), `network_change_pending` (velja, dokler naprava ne zažene shranjene nastavitve, tiste, ki vključuje `set_network(..., reboot=False)`), `auto_stream`, `ptp`. `firmware["update_available"]` je `None`, korazličice ni bilo mogoče prebrati. Medtem ko tok naprave ne dosega tega računalnika, se ta prebere ob novi povezavi in slovar doda `stream_silent` (`True`), `stream_fault` in `reconnect`; enota, ki ne odgovarja, sproži napako. CLI: `pool-info`. |
| `set_network(mode="auto", *, ip=None, mask=None, gateway=None, reboot=True, force=False, comeback_timeout=45.0)` | DAQ-E samo: DHCP (`"auto"`) ali statični naslov. Napravo ponovno zažene, da se nastavitve uporabijo, in jo ponovno poveže; ta seja še naprej deluje, `address` pa sledi napravi. `reboot=False` jo samo shrani. Glejte [Nastavitve naprave DAQ-E](#daq-e-device-settings). CLI: `pool-set-network`. |
| `set_auto_stream(enabled, timeout=15.0)` | Samo DAQ-E: prenos ob vklopu brez povezane programske opreme (shranjeno na napravi). Vrne potrjeno nastavitev. CLI: `pool-set-auto-stream`. |
| `reboot(*, comeback_timeout=45.0)` | Samo za DAQ-E: ponovni zagon naprave in ponovna povezava z nastavitvami, ki jih je imela. CLI: `pool-reboot`. |
| `update_firmware(*, comeback_timeout=45.0)` | Samo za DAQ-E: posodobite vgrajeno programsko opremo, vključeno v to izdajo Chloros, na napravo, ki deluje s starejšo različico, nato jo ponovno priključite (traja približno minuto; naprave ne izklapljajte). Vrne `updated`, `fw_before`, `fw_after`, `bundled` in `warning`, če se je naprava počasi vrnila ali se je vrnila z drugo različico; `updated` je `False` za napravo, ki že ima najnovejšo različico. Sproži `ChlorosConnectError` za enoto, katere različice ni mogoče prebrati. CLI: `pool-update-firmware`. |
| `disconnect()` | Sprostitev iz skladišča. Brez-op za ročaje, ki so pritrjeni, a niso v lasti. Na senzorju, ki ga je odprla ta seja, je to `force_disconnect()`, in sproži `ChlorosConnectError`, če sprostitev ni potrjena (izhod iz bloka `with` nikoli ne sproži izjeme). |
| `force_disconnect(timeout=30.0)` | Sprostitev senzorja ne glede na to, kdo ga je odprl — na primer senzor, ki ga je v bazenu pustil sesut skript in do katerega nov skript dostopa le kot pritrjen ročaj, katerega `disconnect()` ne stori ničesar. Zaključek senzorja drugega odjemalca zaključi vsak snemanje, ki ga ta izvaja. Vrne se, ko backend potrdi sprostitev ali ko ni ničesar za sprostiti (senzor ni v bazenu ali ne teče noben backend); sprostitev, pri kateri je potekel časovni limit ali jo je backend zavrnil, sproži `ChlorosConnectError`. CLI: `chloros-cli daq pool-disconnect --sensor-id ID`. |

Atributi: `sensor_id`, `model`, `transport`, `address` (sledi DAQ-E, ki ga je `set_network()` ponovno naslovil), `info` in `backend_url`.

> **Označite pokrovček.** DAQ-U, DAQ-M in DAQ-E se dobavljajo brez kondenzatorja, Chloros pa ne more ugotoviti, ali je vgrajen: senzor, priključen brez `cap_id`, dobi vrednost `sunshine_cosine`, kar naj bi po njegovih zapisih pomenilo, da je bil vgrajen (`cap_id_source=auto_default`). Nezaščiten DAQ-U pod tem popravkom kaže približno 30-krat višje vrednosti, zato za nezaščiten senzor uporabite `cap_id="none"` (ali `set_cap("none")`), profil pokrovčka, ki ste ga namestili, pa drugače.

### DAQ-E-S — fiksni difuzor in senzor naklona

**DAQ-E-S** je DAQ-E z dvema razlikama, ki sta pomembni za programsko opremo. Vse ostalo — odkrivanje, povezovanje, PTP, snemanje, spodnje nastavitve naprave — deluje popolnoma enako kot pri DAQ-E.**Njegov difuzor je fiksno vgrajen, zato ni potrebna nobena korekcija pokrovčka.** Difuzor tipa »Sunshine« je vgrajen in je bil na napravi že ob merjenju tovarniške kalibracije, zato ta kalibracija že prikazuje dejansko navzdolno sevanje. Chloros nastavi svoj pokrov na `as_recorded` (brez profila), ne glede na to, kaj se zahteva: pokrov, poimenovan po njem (`sunshine_cosine`, `fov_*`, `none`) se prezre s sporočilom, njegovi zapisi pa shranjujejo `cap_id_source=model`, zato se opozorilo o domnevni omejitvi zanj nikoli ne sproži. Tudi ročno ne nastavljajte omejitve za njegove podatke: `sunshine_cosine` na vrhu bi pokazal približno 11-krat višjo vrednost.

**Opremljen je s 3-osnim merilnikom pospeška.** Kosinusni odziv razpršilnika sončne svetlobe velja le, ko je v vodoravnem položaju, zato nagnjen senzor poroča o napačni obsevanosti – napaka se prenese tudi na odbojnost – brez drugih simptomov. Vsaka odčitana vrednost DAQ-E-S zato vsebuje podatke o svojem položaju:

| Polje | Pomen |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Koti v stopinjah. `None`, kadar vgrajena programska oprema označi kot za nedoločen (prost pad, neaktivna os). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Pospešek v miligih. `mag_mg`, ki se močno razlikuje od ~1000, pomeni, da se je naprava premikala in da koti za ta vzorec niso pomembni. |
| `cal_applied` | Nastavljeno: osi in koti so v sklopu difuzorja (uporabljena je kalibracija namestitve enote). Izbrisano: surovi sklop plošče, ki lahko prikazuje do ~2,3° nagiba, ki ni dejanski. |
| `sample_age_ms` | Starost vzorca merilnika pospeška v trenutku pošiljanja spektra. |
| `temp_raw` | **Trend** nekalibrirane temperature čipa — samo razlike, nikoli odčitek termometra. |

`DAQSensorSession.latest()` jih vrne pod ključem `imu` vsakega okvirja (`None` na DAQ-U, DAQ-M ali navadnem DAQ-E). `.daq`, posnet z napravo DAQ-E-S, shranjuje iste vrednosti v stolpcih `imu_*` (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) poleg vsakega spektra, tako da bralec lahko razloči oblak od prevrnjenega stativa. Pri vseh drugih modelih so ti stolpci prazni.

Poročilo o položaju vsebuje podatke v svojih poljih v `status()` (`rx_imu_stream_fault`, `rx_imu_stream_faults`, `rx_last_imu_stream_fault`), ločeno od spektralnih, tako da izpad merilnika pospeška nikoli ni razumljen kot vrzel v zapisu. `discover_daq_sensors()` in `model` seje navajata `DAQ-E-S`.

### Nastavitve naprave DAQ-E

```python
with chloros_sdk.connect_daq_sensor(eth_host="daq-e-def330.local") as daq:
    info = daq.device_info()
    print(info["ip"], info["ip_mode"], info["firmware"])

    # A fixed address on the network this computer is on
    daq.set_network("static", ip="192.168.1.50", gateway="192.168.1.1")
    print(daq.address)              # 192.168.1.50: the session follows it

    if info["firmware"]["update_available"]:
        daq.update_firmware()
```

- **V omrežju tega računalnika mora biti statični naslov.** `set_network` sproži `ChlorosConnectError` za tiste, katerih podomrežje ne vključuje naslova tega računalnikanaslova tega računalnika v omrežju senzorja: po ponovnem zagonu senzorja ga ni mogoče doseči, da bi nastavitev spremenili nazaj. `force=True` uporabite le, če bo temu računalniku dodeljen nov naslov ali bo preusmerjen v novo podomrežje.
- **`set_network`, `reboot` in `update_firmware` ponovno zaženejo senzor** ter sprožijo `ChlorosConnectError` med snemanjem, in če se ne vrne v roku `comeback_timeout` sekund. ID senzorja in ime gostitelja `daq-e-<id>.local` se nikoli ne spremenita.
- **Snemanje, ki se je začelo tik pred ponovnim zagonom, se s tem zaključi in poimenuje.** Ko se senzor vrne, vrnjeni slovar vsebuje `recording_closed` (pot do datoteke) in vrednost `warning`, ki to navaja. Če se senzor ne vrne ali če prenos firmware-a ni uspešen, to namesto tega navaja vrednost `ChlorosConnectError`: njegovo sporočilo se konča z opozorilom, pri čemer sta nastavljeni vrednosti `e.recording_closed` / `e.warning` (oba `None`, če ni bil zaprt noben posnetek). Ko se senzor vrne, ponovno zaženite snemanje.
- **Ponovno se poveže le sam senzor.** Po ponovnem zagonu mora naprava odgovoriti, kje je bila -- in po `set_network("auto")` tudi pod svojim imenom `.local` -- s svojim imenom, imenom gostitelja in modelom, iz omrežnega adapterja (MAC), ki ga je imela; `address` postane IP-naslov, ki je odgovoril. Odgovor pod imenom velja le za tisti MAC-naslov, torej s računalnika, ki doseže senzor prek usmerjevalnika, ali iz privzetega omrežja NAT v WSL2, senzor, ki ga je DHCP premaknil, ni ponovno povezan (`ChlorosConnectError`; ponovno ga povežite na njegov nov naslov).
- **Za posodobitev vgrajene programske opreme se mora senzor ponovno povezati.** `update_firmware()` povabi enoto, ta pa odpre TCP-povezavo nazaj do tega računalnika za prenos slike, in sicer na vratih, ki jih izbere operacijski sistem. Backend prisluhne na adapterju, ki doseže enoto (na vseh adapterjih, le kadar ne more ugotoviti, kateri je pravi), in streže le tej enoti: vsaka druga povezava je zavrnjena (`[DAQ-FW] … refused a firmware connection from <ip>` v izhodu strežnika), medtem ko ta še čaka. Požarni zid, ki blokira povezavo, povzroči neuspeh posodobitve z napako `device never connected back on TCP <port>`. Windowsov namestitveni program doda pravilo zanj (glej [Korak 1](#windows-exe)); na drugem požarnem zidu dovoli vhodni TCP promet do strežnika iz podomrežja senzorja.
- **V okolju WSL2 uporabite zrcaljeno omrežje** (`networkingMode=mirrored` v `.wslconfig`): v privzetem načinu NAT se zavrne vsak statični naslov (lastni naslov WSL ni v omrežju senzorja), tok ne doseže WSL in posodobitev firmware-a se ne more zaključiti.
- Grafični vmesnik ohranja lastno povezavo s senzorjem: ponovni zagon iz skripta prekine sejo grafičnega vmesnika s senzorjem. Grafični vmesnik ima enake nastavitve (zavihek Svetlobni senzorji, nastavitve senzorja, Omrežje).

### Odkrivanje — iskanje naslova za povezavo

`discover_daq_sensors()` pregleduje USB / BLE / ETH za senzorje, ki jih *bi lahko* odprli. To je DAQ-ov ekvivalent `discover_lattice_cameras()` in edini način za pridobitev **BLE MAC-naslova DAQ-M** — DAQ-E ima ime gostitelja, DAQ-U pa COM-vrata, MAC-naslov pa ni natisnjen na napravi niti ga operacijski sistem ne prikazuje.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               DAQ-U  {'vid': '0403', 'pid': '6001', 'manufacturer': 'FTDI'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Polje | Opis |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM-vrata / BLE MAC / ime gostitelja — prenesite v `connect_daq_sensor` kot `port=` / `mac=` / `eth_host=`. |
| `display` | Berljiva oznaka. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` \| `DAQ-E-S`, ali `None`. Ethernet senzor je `DAQ-E-S` ali `DAQ-E`, saj se sam prijavi, ko je to zahtevano, in je med skeniranjem samo za branje (`DAQ-E`, če ne odgovori pravočasno). Serijska vrata USB se berejo kot `DAQ-U`, če je njihov mostični čip eden od tistih, ki jih uporablja DAQ-U (FTDI, CH340, CP210x, Prolific) — namig iz opisa vrat, ne gre za sondo — in `None` za katero koli drugo USB-napravo, ki je še vedno na seznamu, tako da jo lahko posredujete kot `port=`. Serijska vrata brez identifikacijske številke proizvajalca USB in brez takšnega mostičnega čipa (UART na matični plošči, kot je `/dev/ttyS*` v Linuxu, serijska povezava Bluetooth, Intel AMT) ne morejo biti DAQ-U in niso navedena na seznamu. |
| `extra` | Podrobnosti po posameznih prenosih (oglaševano ime BLE; USB `vid`, `pid`, `manufacturer`, `serial_number`; DAQ-E ip/fw/…). Prazne vrednosti so izpuščene. |

| Parameter | Privzeto | Opis |
| --- | --- | --- |
| `transports` | vse tri | Zaporedje (ali niz v formatu CSV), ki omejuje skeniranje. Vredno je to podati, če veste, kaj želite — BLE je počasnejši del. |
| `scan_timeout` | 5 | Okno skeniranja na transport v sekundah; backend omeji vrednost na 1–20. |
| `timeout` | 60,0 | Zgornja meja HTTP za celoten klic (kot drugje v SDK). |
| `auto_start_backend` | `True` | Zažene lokalni backend, če še noben ne teče. Nikoli se ne zažene za oddaljeni `backend_url`. |

> **Senzorji, ki so že odprti v bazenu, se ne prikažejo.** Povezana BLE-periferija preneha z oglaševanjem, odprtega COM-vrata pa ni mogoče preveriti, zato seznam odkritih naprav prikazuje tisto, kar je *na voljo za povezavo*. Takoj po priključitvi naprave je pričakovati prazen rezultat — za naprave, ki jih že imate, uporabite `list_daq_sensors()`. Prevozi, katerih skeniranje se ne more izvesti (ni nameščenih programov bleak / zeroconf), se preskočijo, namesto da bi sprožili napako, zato računalnik brez Bluetootha še vedno prejme odgovore za USB in ETH.

### Seznam

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Kateri DAQ napaja katero kamero — `daq_bindings()`

Odbojnost zajema in `.daq`, ki ga shrani, uporabljata DAQ, vezan na to kamero ali niz, sicer pa najnovejši DAQ, ki ga ima backend. Izbira svetlobnega senzorja v grafičnem vmesniku in `chloros-cli daq pool-bind` to nastavita; prav tako pa:

```python
chloros_sdk.set_daq_bindings(cameras={"214701281": "COM7"},          # a sensor id or an address
                             arrays={"array-1779862544497": "daq-e-def330"})
chloros_sdk.set_daq_bindings(cameras={"214701281": None})            # None clears it
for serial, b in chloros_sdk.daq_bindings()["cameras"].items():
    print(serial, b["binding"], b["source"], b["resolved"]["in_pool"])
```

`daq_bindings()` vrne `{"cameras": {serial: b}, "arrays": {array_id: b}}`: vsak `b` vsebuje `binding` (nastavljena vrednost), `source` (`project`, iz `sensors.json` projekta ali `runtime`), `known` / `known_as` (ali ime kamere ali matrike poimenuje eno, ki jo Chloros pozna, in kako: `connected`, `project` — `cameras.json` odprtega projekta — ali, za kamero, `seen`: že prej odprt na tem računalniku; `known: False` za ključ, ki ne poimenuje ničesar) ter `resolved` (senzor v skupini, na katerega se zdaj nanaša: `in_pool`, `sensor_id`, `address`, `model`, `streaming`). Vezi v času izvajanja prevladajo nad tistimi v projektni datotekiza zadevno sejo; projektna datoteka se ne spremeni. Odklop kamere, niza ali DAQ-ja prekine vsako vezavo te strojne opreme, ne glede na to, kdo jo je nastavil (člani niza se vežejo z nizom).

`set_daq_bindings()` veže le kamero, ki je priključena, v odprtem projektu ali je bila že prej odprta na tem računalniku, ter niz, ki je priključen ali v odprtem projektu (ali ključ, ki je že vezan). Vsak drug ključ sproži `ChlorosConnectError` z imenom (`400`, `code`, `UNKNOWN_BINDING_TARGET`, s ključi `unknown`), pri čemer se v tem klicu **nič** ne veže. `None` izbriše kateri koli ključ.

### Odbojnost okolja + objekta — `light_sensor_reflectance()`

Dva združena DAQ-a, eden usmerjen navzgor proti nebu (okolje) in drugi navzdol proti cilju (objekt), podata odbojnost cilja in indekse vegetacije na podlagi svojih najnovejših odčitkov: par »Ambient + Object« v grafičnem vmesniku ter `chloros-cli daq reflectance`, ne da bi bilo treba senzorje odpreti še enkrat. Oba morata biti priključena in morata prenašati podatke.

```python
r = chloros_sdk.light_sensor_reflectance("daq-e-def330", "daq-e-54b774")
print(r["reflectance_type"])                       # "Apparent Reflectance" when both are calibrated
for name, band in r["bands"].items():
    print(name, band["rho"] if band["valid"] else band["reason"])
for idx in r["indices"]:
    print(idx["name"], idx["value"])
```

```python
light_sensor_reflectance(
    ambient_sensor_id, object_sensor_id, *,
    bands=None,          # [{"name", "center_nm", "fwhm_nm", "role"}]; role blue / green / red / nir
    preset=None,         # a preset id from daq_band_presets(); default: the suggested one, else "GENERIC"
    expressions=None,    # custom indices: [{"name": "MyNDVI", "expr": "(NIR_850 - Red_660) / (NIR_850 + Red_660)"}]
    builtins=True,       # include the built-in indices
    backend_url="http://127.0.0.1:5000", timeout=30.0,
) -> dict
```

- **Pasovi.** Odbojnost vsakega pasu je pasovnaintegrirano sevanje objekta nad sevanjem okolja. `daq_band_presets(filter_code=None)` navaja nize pasov: enega na filter LATTICE M3C (`RGN`, `OCN`, `NGB`, lastni pasovni model kamere, zato sta DAQ NDVI in NDVI kamere enaka veličina) ter `GENERIC` širokopasovni. Njegov `suggested` je prednastavljen in ustreza `filter_code`, sicer pa filtru prve priključene kamere (`suggested_from` navaja, kateri).
- **Vrne** `bands` (po pasu `rho`, `valid`, `reason`, `e_ambient`, `e_object`), `indices` (vgrajenivgrajeni), `custom` (vsak z `value` ali `valid: False` in `error`), `invalid_bands`, `reflectance_type`, `preset` ter vrednosti obeh odčitkov `timestamps` (`ts_ns`). Pas, ki ga senzor okolja ni mogel izmeriti (zunaj svojega območja, na ravni šuma), je neveljaven, z navedbo razloga, in vsak indeks, ki ga potrebuje, se izpusti, nikoli pa se ne izračuna glede na ničlo.
- **Navidezni ali relativni.** Indeksi, odvisni od merilne lestvice (SAVI, OSAVI, EVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI), potrebujejo absolutno razmerje, zato se izračunajo le, če sta oba senzorja kalibrirana glede na moč (vsak ima `calibration_bundle_sha`): `"Apparent Reflectance"`. V nasprotnem primeru je rezultat `"Relative Reflectance"`, ki vsebuje le indekse, neodvisne od merila.
- **Dve različni valovni mreži** se primerjata glede na njuno prekrivanje: točke senzorja okolja znotraj obsega objekta, spekter objekta pa se na njih interpolira.
- **Dva senzorja, nikoli le eden.** Poimenovanje enega senzorja kot obeh (ista identifikacijska številka, v kateri koli pisavi, ki jo sprejme zbirka) povzroči napako `ChlorosConnectError`: odčitek senzorja nad samim seboj je 1,0 na vsakem pasu, kar ne meri ničesar. Tudi backend to zavrne (`POST /api/daq/indices` odgovori na `400`, `code` na `SAME_SENSOR`, medtem ko sta `ambient_sensor_id` in `object_sensor_id`, ki ju pošlje SDK, en združen senzor, npr. njegova ID in njegovo vrata).

### Skupna raba z CLI-jem in drugimi skripti

Skupino senzorjev DAQ v ozadju si delijo vsi odjemalci CLI in SDK na računalniku. Če je senzor že v skupini — odprt s `chloros-cli daq pool-connect` ali z drugim skriptom — klic `connect_daq_sensor(port="COM3")` iz Pythonja vrne izposojeni ročaj: njegov `disconnect()` ne izvede nobene operacije, zato vaš skript ob izhodu iz okna ne odvzame senzorja odjemalcu, ki ga je odprl. Enako velja za `sensor_id=` (vneseno z `:` ali `-`, obe možnosti ustrezata), za DAQ-E, imenovanega po njegovem IP-naslovu, če ga je odprlo ime gostitelja (ali obratno), ter za pametno zaznavanje z zastavico nič, ko backend odgovori s senzorjem, ki ga že ima — blok `with` vedno sprosti le senzor, ki ga je sam odprl.

GUI Chloros **ni** del tega sklopa: njegov DAQ-panel vzpostavlja lastno povezavo s senzorjem, zato `connect_daq_sensor` nikoli ne izposodi senzorja, ki ga ima odprt GUI. Ne uporabljajte istega senzorja hkrati iz grafičnega vmesnika in iz skripta. DAQ-U ali DAQ-M, ki ga drži grafični vmesnik, se običajno ne uspe odpreti, vendar DAQ-E sprejme drugo sejo: povezava skripta spremeni nastavitve zajemanja enote v grafičnem vmesniku, njegov `disconnect()` pa ustavi tok enote, kar prekine prikaz v živo v grafičnem vmesniku in kakršno koli snemanje, ki ga ta izvaja. Najprej zaprite senzor v grafičnem vmesniku.

### PTP Grandmaster — `time_sync_status()`

V ozadju teče PTP grandmaster, iz katerega kamere LATTICE in enote DAQ-E črpajo svoje časovne oznake. Zažene se na zahtevo: `connect_camera()`, `connect_array()`, `connect_daq_sensor()` in `connect_all()` ga zaženejo, če še ne teče — vključno s povezavo, ki nato ne uspe, na primer ko `connect_all()` poskuša vzpostaviti povezavo z enoto DAQ-E, ki je ni — enako pa storijo tudi `test_array_sync()` in vsak klic, ki odpre kamere za en ukaz (`capture_cameras_transient()`, `start_multi_camera_stream()`, klici za poravnavo, `capture_live_index()`).

- **Kje se to sporoči.** Na žičnih omrežnih adapterjih, ki so povezani s kamerami (vključno z lokalnim naslovom povezave `169.254.x.x`), sicer pa na adapterju privzete poti, če je ta žična. Nikoli prek Wi-Fi-ja ali virtualnega adapterja (Tailscale, WSL): za njim ni nobene kamere ali DAQ-E. Če tak adapter ni aktiven, se oglašuje na ničemer (`health` `NO_SOCKETS`) in se veže na adapter, ki ga vzpostavi naslednja povezava kamere ali DAQ-ja.
- **Po prekinitvi povezave.** Ko adapter kamere izgubi povezavo in jo ponovno vzpostavi (izvlečen kabel, zamenjan stikalo, ponovno prepoznan USB-adapter), grandmaster samodejno ponovno vzpostavi svoje vtičnice brez ponovnega zagona, in ponovno preveri vsak priključen DAQ-E ter ponovno aktivira tistega, ki je ostal v mirovanju. Ponovno vzpostavi povezave tudi takrat, ko vse kamere in DAQ-E, ki jih je oskrboval, 15 sekund ne oddajajo podatkov.
- **Ali je povezava prekinjena?** Preberite `time_sync_status()` (`GET /api/time-sync/status`). `state` navaja le, da teče glavni niz; `health` pa navaja, ali njegovi paketi odhajajo: `OK` (napaka ni zaznana), `STOPPED`, `NO_SOCKETS`, `SENDS_FAILING`, `DEGRADED` (nekateri adapterji ne delujejo) ali `PEERS_SILENT` (kamere in DAQ-E, ki jih je oskrboval, so prenehale odgovarjati), s stavkom v `health_detail`. `announce_sent` / `sync_sent` / `delay_resps_sent` štejejo le sporočila, ki so bila poslana; `announce_send_errors`, `sync_send_errors`, `follow_up_send_errors`, `delay_resp_send_errors`, `last_send_error`, `failing_interfaces` in `last_delay_req_age_s` opisujeta tista, ki niso bila poslana, `rebinds` / `last_rebind` pa ponovna pošiljanja. `sync_sends_slow` šteje sinhronizacije, katerih pošiljanje je trajalo več kot 200 us, `sync_send_max_us` pa je najdaljša; čas izvora vsakega Follow_Up šteje največ 200 us njegovega pošiljanja. Grandmaster teče v lastnem procesu (`own_process`, `process_pid`), z visoko prioriteto v sistemu Windows (`process_priority`), tako da zaseden backend ne more zamujati z žigosanjem časov in hkratnim premikanjem ure vsake kamere; če tega procesa ni mogoče zagnati, teče znotraj backenda kot prej (`own_process` false).

```python
st = chloros_sdk.time_sync_status()
if st.get("health") not in (None, "OK", "STOPPED"):
    print("PTP:", st["health"], "-", st.get("health_detail"))
    chloros_sdk.restart_time_sync()
```

| Funkcija | Kaj počne |
| --- | --- |
| `time_sync_status()` | Stanje glavnega strežnika, njegovo delovanje, identifikacija ure, prioritete in števci sporočil (zgoraj). `chloros-cli time-sync status`. |
| `time_sync_peers()` | PTP-podrejeni strežniki, ki jih zazna (kamere, DAQ-E): `peers`, vsak `port_identity`, `last_seen_ip`, `last_delay_req_age_ms`, `delay_req_count`. `time-sync peers`. |
| `time_sync_cameras()` | Vsaka povezana matrična kamera: `PtpStatus`, `PtpOffsetFromMaster` in `PtpMeanPathDelay` (kamera, katere vgrajena programska oprema nima PTP-vozlišč, ima le serijsko številko). `time-sync cameras`. |
| `restart_time_sync()` | Zaustavi in ponovno zaženi glavni strežnik; po tem se ponovno preveri PTP vsake združene kamere DAQ-E (`daq_ptp_rechecked`). Izda `ChlorosConnectError`, ko se se ne vrne. `time-sync restart`. |
| `set_time_sync_priority(priority1=None, priority2=None)` | Njegove PTP-prioritetne vrednosti (0–255, nižja vrednost zmaga v izvolitvi najboljšega glavnega strežnika) za omrežje z drugim PTP-glavnim strežnikom. Zahteva, da ta teče. `time-sync set-priority`. |

Vsak od njih vzame `backend_url=` in `timeout=` ter zažene lokalni backend, ki še ne teče.

---

## Pripomočki za kamere

Kaj `chloros-cli lattice` počne poleg povezovanja in zajemanja, kot preproste funkcije v `chloros_sdk` (modul `chloros_sdk.camera_utils`). Vsak od njiju pokliče pot do ozadja, ki jo uporablja ukaz CLI, v skladu s pravili CLI, tako da se obe obnašata enako. Sprejemata **serijsko** številko kamere, ne pa seje, in iste ključne besede `backend_url=` / `timeout=` kot klici smart-connect; če backend ne teče, se zažene, tako kot to stori `connect_camera()`. V primeru zavrnitve se sproži `ChlorosConnectError` z razlogom backenda.

```python
import chloros_sdk as c

c.set_camera_color_profile("214701281", profile="natural")
c.set_camera_highlights("214701281", -40)
print(c.camera_firmware_status()["cameras"])
st = c.network_adapters(estimate=True, cameras=4, pixel_format="BayerRG12")
```

| Funkcija | Kaj počne |
| --- | --- |
| `camera_color_profile(serial)` / `set_camera_color_profile(serial, profile=, custom_cct_k=)` | Barvni profil zaslona: `raw`, `linear`, `natural`, `enhanced` ali `custom_temp` ter njegova barvna temperatura (omejena na 2000–10000 K; odčitana, če je profil `custom_temp`). |
| `camera_color_adjust(serial)` / `set_camera_color_adjust(serial, saturation=, contrast=, reset=False)` | Nasičenost in kontrast zaslona, vsak od 0 do 200 % (100 = nespremenjeno); `reset=True` oba vrne na 100. |
| `camera_highlights(serial)` / `set_camera_highlights(serial, value)` | Zmanjša svetlost najsvetlejših, najbeljših pikslov zaslona: -100..0 (okroženo proti ničli, nato omejeno; 0 = izklopljeno). |
| `camera_white_balance_once(serial, settle_s=None)` | Enkratna nastavitev beline: kamera kratko prenaša sliko, se uskladi in vrne `red_ratio` / `blue_ratio` (`ratios_read` False, če jih ne more sporočiti). Ohrani izračunano belo ravnovesje; vse ostale nastavitve, ki jih je zapisal v odprto datoteko, se vrnejo takšne, kot so bile najdene. |
| `set_camera_white_balance(serial, mode=None, red=None, blue=None)` | `mode="auto"` (neprekinjeno) ali `"off"`; ali fiksna razmerja beline `red` / `blue`, ki najprej izklopijo avtomatsko nastavitev. Razmerje, ki ga kamera zavrne, je poimenovano pod `errors`. |
| `camera_presets()` / `apply_camera_preset(serial, preset)` | Prednastavitve kamere in uporaba ene od njih: kamera ohrani nastavitve, ki jih imenujejo prednastavitve (odgovor je `settings`, saj jih kamera prebere naknadno, npr. hitrost snemanja, s katero deluje; `None` za polje, ki ga prednastavitev ne imenuje), vse druge nastavitve, ki jih je zapisal odprti program, pa se vrnejo v prvotno stanje. `"default"` ponastavi kamero na tovarniške privzete vrednosti (tovarniške nastavitve s prosto delujočo avtomatsko osvetlitvijo, omejitev prepustnosti povezave in izklopljeno popravljanje napak, ohranjena kalibrirana bela bilanca RGB-kamere), ki niso shranjene v njenem pomnilniku: odgovor vsebuje `factory_defaults: True` in vsa polja iz `settings`. V `camera_presets()` je `default` enak `{factory_defaults: True, description}`. |
| `camera_filter_models()` / `camera_filter_model_info(name)` | Modeli filtrov in podrobnosti enega modela (pasovi, število pogojev spektralnega odziva). Model filtra kamere se prebere iz njenega modelnega niza; ne nastavi se ročno. |
| `camera_firmware_status(serial=None)` | Firmware vsake kamerevgrajena programska oprema: `current_version`, `available_version`, `update_offered`, `needs_minimum_update` in `minimum_firmware`. |
| `update_camera_firmware(serial, force=False)` | Naložite firmware te različice Chloros na eno kamero: ponastavitev na tovarniške nastavitve, naložitev, ponovni zagon, ponovno zapisane osnovne nastavitve Chloros; kalibracija kamere ostane ohranjena. Traja nekaj minut; kamera mora ostati pod napetostjo. `force=True` ponovno namesti firmware na trenutno kamero. |
| `camera_power_probe(serial)` | Napajanje kamere, toplotni vozli in njena temperatura. |
| `sleep_camera(serial)` / `wake_camera(serial)` | Način mirovanja kamere z nizko porabo energije (ostane v mirovanju, dokler ni prebudena: omejitev prepustnosti povezave je vklopljena (`DeviceLinkThroughputLimitMode` `On`) na najnižji vrednosti, vklopljen je način varčevanja z energijo, če ga kamera ima), s temperaturo pred in po; prebujenje vrne povezavo v stanje, v kakršnem je bila ob prehodu v mirovanje (omejitev, njen način in zakasnitev paketov `GevSCPD`, ki jo kamera ohranja na 0, dokler je omejitev vklopljena; zapis se hrani na tem računalniku, tako da ga najde tudi prebujenje iz kasnejšega strežnika), ali pa za kamero, ki je ta računalnik ni spravil v stanje mirovanja, a izgleda, da miruje (njena omejitev je vklopljena na najnižji vrednosti), pri čemer se omejitev izklopi, vrednost `GevSCPD` pa ostane na 0 pri 80 (tovarniške nastavitve); omejitev, nastavljena nad najnižjo vrednostjo, ostane nespremenjena. Ohrani se le stanje napajanja; vse druge nastavitve, ki jih je odprto zapisalo, se vrnejo v prvotno stanje. |
| `calibrate_camera_with_target(serial, filter=None, attempts=30, settings=None)` | Zajemi okvirje, dokler ni v vidnem polju kalibracijski cilj MAPIR, in prilagodi koeficiente po kanalih iz njegovih plošč: `target_id`, `panel_dns`, `coefficients`. |
| `network_adapters(estimate=False, cameras=1, pixel_format="BayerRG8")` | Ethernetni adapterji tega računalnika za GigE Vision (MTU, hitrost povezave, jumbo okvirji) in vsi `issues`; `estimate=True` dodaja hitrost snemanja in pasovno širino, ki bi ju imele kamere `cameras`. |
| `fix_network_settings()` | Uporabi nastavitve GigE Vision adapterja za adapter kamere tega računalnikakamere. To spremeni nastavitve adapterja in deluje le, če se program Chloros izvaja kot skrbnik; v nasprotnem primeru se prikaže dnevnik z ugotovljenimi podatki. |
| `apply_vignette_correction(input_dir, output_dir, lens_model=None)` | Popravite vinjetiranje slik v mapi, pri čemer vsaka slika uporabi lastno karto ravnega polja objektiva: `processed`, `lens_models` (objektiv, za katerega je bila posamezna datoteka popravljena), `skipped` (z razlogi). Brez `lens_model` se objektiv za vsako datoteko določi na podlagi podatka »Model« v EXIF-u; datoteka `lens_model` velja za datoteke, katerih model ne navaja imena kamere, datoteka, katere model je v nasprotju s tem, pa se preskoči. Samo datoteke. |
| `read_dls(mac, filter=None, camera_serial=None)` | En odčitek iz DLS (Bluetooth senzorja navzdol usmerjene svetlobe): spekter, obsevanost po posameznih pasovih in vrednosti pasov skozi lastni odziv kamere, kadar je podan `camera_serial`. `discover_daq_sensors(transports=["ble"])` poišče MAC. |
| `probe_system(pixel_format="BayerRG12", discover=True)` / `system_info()` | Strojna oprema gostitelja za kamere in priporočena konfiguracija (`report_text`) ter procesor (CPU), grafična kartica (GPU) in izvajalno okolje PyTorch sistema. |

- **Nekatere kamere potrebujejo izključno zase.** `camera_white_balance_once()`, `apply_camera_preset()`, `sleep_camera()` / `wake_camera()`, `calibrate_camera_with_target()` in `update_camera_firmware()` odprejo kamero. Medtem ko jo zadržuje Chloros (povezana v aplikaciji, z `connect_camera()` kot članica niza ali trenutno v uporabi zaradi drugega ukaza, kot je na primer test niza ali zajem, ki odpira kamere), strežnik zavrne z HTTP-odzivom 409 `CAMERA_HELD` (tako v `code` kot v `error_code`), v kateri je navedeno, kako ga sprostiti. Nastavitve barv, osvetljenih delov, `camera_power_probe()` in odčitki delujejo na priključeni kameri.
- **Barva, osvetljeni deli in bela ravnovesje so namenjeni RGB-kameram.** Na enopasovni monokromatski (M3M) kameri nastavitve ničesar ne spremenijo in vrnejo `{"skipped": True, "reason": ...}`, tako kot to stori CLI. To so nastavitve prikaza: pogled v živo in posnetki na ravni zaslona, nikoli pa ne svetilnosti ali odbojnosti.

---

## Prehodni nizi, profili poravnave in indeksi

Kaj počne `chloros-cli lattice` brez trajne povezave, kot funkcije v `chloros_sdk` (modul `chloros_sdk.array_utils`): `continuous`, `multi-capture`, `multi-stream`, `multi-test`, `multi-detect`, `analyze-array`, `align-calibrate`, `align-apply`, `align-stream`, `align-info`, `align-reorder` in `index`. Vsak od njih pošlje zahtevo, ki jo pošlje ukaz CLI, s privzetimi nastavitvami CLI.

```python
import chloros_sdk as c

prof = c.calibrate_alignment_profile(["214000535", "214000551", "214000556"],
                                     "D:/rig/alignment_profile.json", method="orb", model="rigid")
print({sn: t["reprojection_error"] for sn, t in prof["transforms"].items()})

c.capture_aligned_multiband("D:/rig/alignment_profile.json", "D:/rig/out")
idx = c.capture_live_index("D:/rig/alignment_profile.json", preset="NDVI",
                           output_dir="D:/rig/ndvi", capture_level="reflectance")

with c.start_multi_camera_stream(["214000535", "214000551"], "D:/rig/stream", fps=5, count=50) as job:
    print(job.wait()["final_stats"])          # waits for the 50 groups, then closes the cameras
```

- **Same odprejo kamere.** Vsak ukaz odpre svoje kamere, opravi svoje delo in jih zapre, zato potrebuje kamere, ki jih Chloros ne zadržuje (priključene v aplikaciji, z `connect_camera()` ali kot član niza); strežnik zavrne z HTTP-odzivom 409 `CAMERA_HELD` in navede, kako kamero sprostiti. Začasno odpiranje traja nekaj časa (približno 40 s za nekaj kamer). Za ponavljajoče se zajeme je je `connect_array()` enkratna uporaba najhitrejša pot in omogoča shranjevanje v vseh izvoznih formatih, medtem ko začasno zajemanje shrani sliko brez debayeringa. `calibrate_alignment_profile()` je izjema: na povezanem nizu teh kamer rešuje iz lastnih okvirjev niza (`used_live_array`).
- **`serials`** so kamere, ki jih je treba odpreti, prva pa sproži sprožilec; `None` odpre vse kamere v omrežju.
- **`settings`** je slovar prepisov nastavitev kamer, polja, ki jih določajo zastavice v CLI: `exposure_time` (µs), `gain` (dB), `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `frame_rate`, `gev_packet_size` in `preset` (prednastavitev kamere po imenu). Določena osvetlitev ali ojačanje izklopi avtomatsko krmiljenje, tako kot to storijo zastavice.
- **`gpu`**: `True` prisili uporabo GPU za warp, `False` pa CPU (`--gpu` / `--no-gpu`); `None` prepusti izbiro backendu.
- **`keep_settings`**/**`clear_settings`** (vsaka funkcija, ki odpira kamere, privzeto `False`): na koncu se vsaka kamera vrne v stanje, v kakršnem je bila najdena; `keep_settings=True` namesto tega shrani nastavitve izvajanja v lastni pomnilnik kamere (prosto delovanje), `clear_settings=True` pa ta pomnilnik ponastavi na tovarniške nastavitve — glej [Pomnilnik kamere](#camera-memory--keep_settings--clear_settings). Odgovor (za nalogo, `stop()`) vsebuje `camera_memory`.
- **Poti so poti backenda**: relativne poti se pretvorijo v absolutne glede na vaš delovni imenik, če je backend na tem računalniku. Funkcije za datoteko profila poravnave (`alignment_profile_info()`, `reorder_alignment_profile()`) berejo datoteko na tem računalniku, tako kot to počne CLI.

| Funkcija | Kaj počne |
| --- | --- |
| `start_camera_continuous(serial, output_dir="output", *, format="tiff", jpeg_quality=95, queue_depth=64, settings=None)` | Neprekinjeno snema eno kamero, dokler se ne ustavi. Vrne `CliJob`. Sliki, posnete med izvajanjem `tiff`, so v datoteki `raw`, ki jo zapisuje zajemnik za matriko te kamere (ena mozaikna ravnina zlastnih vrednosti, nekomprimirane); `png` in `jpg` vsebujeta 8 bitov, zato so ti zapisi okvirji z demosaicingom za ogled. Veženi ali združeni DAQ zapiše eno datoteko `.daq` na okvir vmapo `daq/` (`daq_files` v njenih statistikah). Vsak posnetek se zapiše enkrat s polnimi metapodatki (model, čas zajema, objektiv, pasovi, kalibracija na surovem posnetku); `stop()` se vrne, ko je zapisan vsak že zajeti posnetek (do 25 s), šele takrat se kamera zapre. Kamera, ki ostane pripravljena za sprožilec strojne opreme (`TriggerMode` `On`) je za ta cikel nastavljena na prosti tek (odgovor ob zagonu `trigger` navaja, kaj je bilo ugotovljeno). Ob zaustavitvi se kamera vrne v stanje, v kakršnem je bila najdena — z nastavitvami sprožilca in vsemi nastavitvami, ki jih je spremenil tek — razen če je uporabljen `keep_settings=True`, ki shrani nastavitve teka v lastni pomnilnik kamere (prosti tek), ali `clear_settings=True`, ki ta pomnilnik ponastavi. Zagon, ki ne posname nobenega okvirja, se samodejno zaključi po 10 sekundah z `stop_reason` `no_frames`, razlog pa je naveden v `last_error`, njegova funkcija `stop()` pa ta razlog vrne kot `error`. Sliki, ki so bile izpuščene, ker zapisovalniki niso uspeli slediti, se štejejo, pri čemer `drop_reason` navaja razlog. Za kamero, ki jo drži Chloros, uporabite `CameraSession.capture_repeated()`. `lattice continuous`. |
| `start_multi_camera_stream(serials=None, output_dir="output", *, fps=3.0, count=None, format="tiff", jpeg_quality=95, settings=None)` | Sinhronizirane skupine okvirjev v toku na `fps`, za skupine `count` ali dokler se ne ustavi. Vrne `CliJob` (njegov `start_reply` ima `n_cams` in povezavo `pacing`); njene končne statistike štejejo zajete, zapisane in izpuščene skupine, datoteke, ki jih zapisovalnik ni mogel zapisati, ter skupine, za katere kamera ni dostavila ničesar. `lattice multi-stream`. |
| `capture_cameras_transient(serials=None, output_dir="output", *, format="tiff", jpeg_quality=95, settings=None)` | En sinhroniziran zajem, ena slika brez bayera na kamero: `results` (`filepath`, `serial`, `frame_id`) ter `pacing`. Prikaže se opozorilo o manjkajoči kameri, pri čemer je najprej navedeno, koliko datotek je bilo shranjenih. `lattice multi-capture` brez priključenega niza. |
| `test_array_sync(serials=None, count=10, ptp_settle_s=None, timeout=None)` | `count` sinhronizirani posnetki z fiksno osvetlitvijo: `sync_groups`, `master`, ena vrstica `group_timestamps` na sprožilec (kamera, ki je en posnetek izpustila, v svoji vrstici manjka) in `sync`: odmik vsake druge kamere od glavne kamere, sprožilec za sprožilcem (`offsets_us`: na kamero `mean_us`, `min_us`, `max_us`, `jitter_us`, `groups`), na ENEM taktu (`clock_domain`: `ptp` ali `device` — PTP izklopljen, vsaka kamera ima lastno uro delovanja, zato je njen odmik določen ob zagonu, sinhronizacijo pa meri le nihanje). Sprožilec, katerega časovni žigi mešajo uro naprave in PTP (PTP izklopljen ali še vedno v sinhronizaciji), se izpusti in upošteva (`groups_mixed_domain`), prav tako kot tisti, pri katerem je PTP ura še vedno več kot `agree_within_us` oddaljena od glavne (`groups_clocks_disagree`); to potrjujeta `notes`. `clock_offset_mean_us` / `clock_offset_jitter_us` sta največja povprečja in nihanja na kamero. `success` je lažna vrednost, skupaj z `error`, kadar je bil sprožilec zamujen, kadar imata manj kot dva sprožilca vse kamere na enem taktu ali kadar se takti nikoli ne ujemajo (`sync.problems`); vrstice se vrnejo v vsakem primeru. Zažene PTP grandmaster, če ta ne teče (`time_sync` navaja `grandmaster_running` in `grandmaster_started_by_this_test`), vklopi PTP na vsaki kameri (kot to stori `connect_array()`) in pred prvim sprožilcem počaka, da se PTP-ura vsake kamere sinhronizira z glavnim časomerilcem in se ustalijo: `PtpStatus` Slave na enem glavnem sinhronizatorju, z odstopanjem največ 100 us od njega in stabilen 3 s, pri čemer je vsaka kamera sinhronizirana na ta način, brez prekinitve, 120 s — približno 2 minuti od vklopa. Ura, ki se je pravkar uskladila, ima odstopanje do ~200 us še približno 2 minuti po tem, ko se prikaže stanje »Slave«, kar pa kamere lastni `PtpOffsetFromMaster` ne prikaže; v 120 s se kamere približajo stabilnemu taktu z odstopanjem ~10 us. Kamera, ki je še vedno sinhronizirana od prejšnjega testa, pri katerem je PTP ostal vklopljen, kot je to zaznal ta backend, šteje od te sinhronizacije, tako da test, ki sledi takoj za drugim, ne čaka ponovno; kamera, ki jo test vklopi, ali tista, pri kateri je PTP vklopljen, a ta backend še ni zaznal sinhronizacije (vklopljena drugje ali pred ponovnim zagonom backenda), počaka svojih 120 s, medtem ko v nizu obeh kamer tisti, ki so se pravkar sinhronizirale, še vedno čakajo na svoj čas. Kamera, ki zapusti stanje »Slave«, preseže 100 us ali sledi drugemu »grandmasterju« pri 3 odčitkih zapored (1,5 s) med čakanjem, začne odštevanje znova; ena ali dve takšni odčitki se zabeležita (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`), odštevanje pa se nadaljuje, ID glavnega vodje, ki ga ni mogoče prebrati, pa se obravnava kot neznan, ne kot sprememba. Če se štetje vseh kamer ponovno začne hkrati (v roku ene sekunde), se premika čas glavnega vodje, ne pa čas kamer: dnevnik strežnikadnevnik prikazuje `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` ga navaja v `grandmaster_steps`, čakanje, ki se nato izteče, pa začne svojo napako z `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Meja se zanj ne podaljša. `ptp_settle_s` (0–600; `0` brez čakanja na stabilizacijo) nastavi čas stabilizacije za izvedbo; druga vrednost se zavrne (HTTP 400). Čakanje je omejeno na čas poravnave plus 90 s (privzeto 210 s, najmanj 90 s), klic pa čaka na backend `timeout` s: privzeto 240 plus čas poravnave (360 s), tako kot to počne `chloros-mcp` v `test_array_sync`. `ptp_lock` ima `converged`, `waited_s`, `timeout_s`, `settle_s`, `locked_s` (kako dolgo je vsaka kamera že zaklenjena in stabilna), `found_settled` (kamere, ki so bile ob odkritju že zaklenjene za določen čas) ter, za vsako kamero posebej, `status`, `offsets_ns`, `slave_s` (kako dolgo je že zaklenjena; None: ni zaklenjena) ter `remaining_s` (None, če se PTP nobene kamere ne vklopi). Dnevnik strežnika vsebuje vrstico vsakih 5 s s stanjem vsake kamere, kako dolgo je že zaklenjena in koliko časa ji še ostane. Vsaka kamera se vklopi z izklopljenim PTP; od takrat se ure sinhronizirajo v približno 10 sekundah. Nato se vsaka kamera vrne v stanje, v katerem jo je test našel – njen PTP, avtomatska osvetlitev in ojačanje, ki jih test med izvajanjem ohranja izklopljene (`ExposureAuto` / `GainAuto` ter ročni `ExposureTime` / `Gain`), ostale pa, razen če je `keep_settings=True` – torej kamera, pri kateri ostane PTP vklopljen (`PtpEnable=True` ali izvedba s `keep_settings=True`) ostane zaklenjena, in ko ta backend ugotovi, da je zaklenjena za čas stabilizacije, kasnejši test nanjo ne čaka. Če se ure ne stabilizirajo, je `success` lažna pred katerim koli sprožilcem, `error` pa navaja, kaj storiti: kamera, ki se nikoli ni zaklenila (`not_slave`), pomeni, da grandmaster ne doseže te kamere (`time_sync_status()`); tista, ki je nenehno izpadala, je poimenovana po tem, kako dolgo je bila zaklenjena (`remaining_s`); ure se še vedno usklajujejo (`far`): preverite `time_sync_status()`, da pa bodo zaklenjene pred testom, vklopite PTP in ga pustite vklopljenega (`PtpEnable=True` na vsaki kameri ali `keep_settings=True`; test ponovno nastavi ugotovljene vrednosti, zato ostane vklopljen), nato pa ga zaženite. `lattice multi-test`. |
| `detect_array_master(line=None)` | Kamere na sinhronizacijski liniji in privzeti glavni strežnik: `master_serial`, `slave_serials`, ki jih je preizkusil `line`. `lattice multi-detect`. |
| `analyze_array_models(camera_models=None, *, width=2048, height=1536, n_active=None, force_tier=None, binning=1, packed="auto", pixel_format=None)` | Kakšen nabor teh modelov kamer (ime izdelkov, `"LATT-M3C-L87-FOCN"`) bi se pojavila v omrežju tega računalnika: `tier`, `achievable_fps_max`, omejitev števila naprav (`max_cams_collision_safe`, `oversubscribed`), `cam_caps` in `fix_suggestions`. Privzeto so to modeli odkritih kamer; ni odprta nobena kamera. Za kamere, ki jih imate, z nastavitvami, ki bi jih uporabila povezava, glejte [`analyze_array_network()`](#network-analysis-helper). `lattice analyze-array`. |
| `calibrate_alignment_profile(serials=None, save_path="alignment_profile.json", *, frames=3, method="orb", model="rigid", reference_serial=None, checkerboard=None, control_points=None, preview=False, vignette=False, roi=None, mask_path=None, rms_threshold_px=None, per_cam_overrides=None, settings=None, ...)` | Izračunajte profil poravnave iz posnetkov v realnem času in ga shranite. `method`: `orb`, `akaze`, `phase`, `checkerboard` (z `checkerboard="9x6"`, notranji koti) ali `manual` (z `control_points={serial: [[x, y], ...]}`, tudi referenčne kamere). `model`: `translation`, `rigid`, `affine` ali `homography`. Natančnejše možnosti so zastavice v CLI: `max_features`, `ratio_threshold`, `matcher` (`bf` / `flann`), `knn_k`, `prefilter` (`none` / `gradient` / `clahe` / `blur` / `hist_match`), `ransac_threshold_px`, `ransac_max_iters`, `ransac_confidence`, `averaging`, `lock_rotation`, `lock_scale`, `lock_axis`; `per_cam_overrides` je `{serial: {option: value}}`. `rms_threshold_px` ne shrani slabše rešitve. Vrne `save_path`, `output_width` / `output_height`, `crop_roi`, `reference_serial`, `transforms` (za vsako kamero `reprojection_error`, `num_matches`) ter `preview_path`. `lattice align-calibrate`. |
| `capture_aligned_multiband(profile_path, output_dir="output", *, serials=None, format="tiff", bit_depth=None, gpu=None, no_crop=False, bands=None, order=None, vignette=False, per_band=False, per_camera=False, interpolation="linear", border_mode="constant", border_value=0, settings=None)` | Ena poravnana večpasovna slika skozi profil: `files` (večpasovni TIFF s svojim `bands` in velikostjo, plus ena na pas z `per_band` in ena na kamero z `per_camera`) ter `vignette_corrected`. `bands` izbere, `order` pa razvrsti pasove po imenu (seznam ali niz z vejicami). `lattice align-apply`. |
| `start_aligned_stream(profile_path, output_dir="output", *, serials=None, fps=3.0, count=None, ...)` | Poravnava večpasovnih slik v toku (iste možnosti poravnave kot pri `capture_aligned_multiband()`), za `count` ali dokler se ne ustavi. Vrne `CliJob`; njegove končne statistike so `frames_written` in morebitni `frame_groups_incomplete`. `lattice align-stream`. |
| `alignment_profile_info(path)` | Datoteka profila, povzeta kot `align-info`, jo natisne: `name`, `created`, `method`, `reference_serial`, `sensor_resolution`, `output_width` / `output_height`, `crop_roi`, `camera_serials`, `transforms` (na kamero `model`, `reprojection_error`, `num_matches`, `reference`), `camera_models`, `layer_order` (za vsako kamero `band_names`, `wavelengths_nm`, `enabled`); `profile` je celotna datoteka. |
| `reorder_alignment_profile(path, order=None, enable=None, disable=None)` | Spremeni datoteko profila na mestu: `order` so imena pasov (vsak premakne sloj svoje kamere navzgor, v tem vrstnem redu), `enable` / `disable` so serijske številke. Vrne novo `layer_order`. `lattice align-reorder`. |
| `index_catalog()` | Prednastavitve indeksa (`presets`: vsaka `name`, `channels`, `formula`) ter barvne `gradients`. `lattice index --list-presets` / `--list-gradients`. |
| `compute_index(input_path, *, preset=None, formula=None, channel_map=None, output=None, capture_level="unknown", output_format="all", colorize=False, gradient="RdYlGn", vmin=None, vmax=None, percentile=None, bg_mode="clip")` | Indeks za poravnano večpasovno datoteko TIFF: `preset` (`NDVI`, `NDRE`, ...) ali `formula` nad simboli, ki jih `channel_map` preslika na pasove (`{"NIR": "NIR_850", "Red": 0}`, po imenu ali indeksu). Prednastavljensimbol `channel_map` izpusti en pas datoteke, katerega ime se začne z njim (`red` -> `Red_660`, `nir` → `NIR_850_RGN`), poimenovan po sami datoteki (njenem seznamu pasov ali `Chloros:Band{R,G,B}Name` zajema/proizvoda Chloros), zato NDVI na posnetku odbojnosti Chloros ne potrebuje `channel_map`; dva pasova, ki ustrezata enemu simbolu, se zavrneta, ne pa ugibata. `capture_level` navaja, kaj pasovi vsebujejo: `reflectance` ali `radiance` vrednosti naredita kvantitativne; nekalibrirani niz (`align-apply`) daje kvalitativne vrednosti; `"unknown"` to prebere iz datoteke, vključno z datoteko Chloros, `Chloros:ProcessingLevel`. Vrne `stats` (obliko, razpon 1–99 percentila, razpon LUT, uporabljeni `capture_level`) ter `files`, zapisan ob vhodu (`<input stem>_<preset>.tif`, `..._LUT.png`) ali na `output`, pot: če imajo datoteke slikovno končnico, so poimenovane po osnovi (`output="D:/out/ndvi_FRGN.png"` zapiše `ndvi_FRGN.tif` in `ndvi_FRGN_LUT.png`), brez nje pa gre za predpono (`D:/out/ndvi` → `ndvi.tif`, `ndvi_LUT.png`); preberite poti iz `files`. Vsebujejo podatke o kameri iz vhodnega zapisa (proizvajalec / model / serijska številka, GPS, objektiv, čas zajema v UTC, identifikacijsko številko kalibracije, nikoli pa kalibracijo raw-DN), `Chloros:ProcessingLevel=index` ter indeks kot svoj edini pas (`XMP-Camera:BandName`, TIFF in LUT PNG). Samo datoteke; brez kamere. `lattice index --input`. |
| `capture_live_index(profile_path, *, preset=None, formula=None, channel_map=None, serials=None, output_dir="output", output_prefix="index", capture_level="reflectance", save_multiband=False, ...)` | Posnetek, kalibracija (`reflectance` ali `radiance`; `raw` to preskoči), poravnava in izračuna indeks v enem koraku; `save_multiband=True` ohrani kalibrirani niz, ki ga `compute_index()` lahko kasneje prevzame. Vrne `stats` (skupaj z `serials`, katerega pasove je uporabil) in `files`. Odpre `serials` (prva sproži sprožilec), sicer pa kamere profila, katerih vnos `layer_order` je omogočen, najprej referenčno (odpre se tudi, če so njeni pasovi onemogočeni); kamera, katere pasovi so onemogočeni, se nikoli ne kalibrira. `channel_map` je potreben le za simbol, na katerega sklad ne more odgovoriti, kot v `compute_index()`: simbol prednastavitve vzame eno omogočeno pasovno območje, katerega ime se začne z njim, kar se preveri, preden se odpre katera koli kamera (če se ne odzove nobena ali se odzove več, se zavrne, pri čemer se jih imenuje); vsak simbol potrebuje prilagojeni `formula`. `index.tif` in LUT vsebujeta kamero sklada, čas zajema (osvetlitev referenčne kamere, UTC) in objektiv, oba pa imenujeta indeks kot svoj edini pas (`XMP-Camera:BandName`, npr. `NDVI`: barvna datoteka PNG navaja indeks, ne pa svojih barvnih kanalov). `lattice index --live`. |

### `CliJob`

Ročaj, ki ga vrnejo trije tokovi. Naloga teče v ozadju, kjer ima kamere v lasti, dokler ni ustavljena.

| Član | Opis |
| --- | --- |
| `stats(timeout=10.0)` | `running`, `state`, `stats` (shranjeni / zapisani, izpuščeni, `fps`, `elapsed`, napake pri pisanju, `stop_reason`; pri neprekinjeni nalogi tudi `frames_grabbed` in `frames_queued` – vsak zajeti okvir je zapisan, v čakalni vrsti, izpuščen ali ni uspel – ter `drop_reason`, `writers` in `write_ms`, povprečni čas, ki je bil potreben za zapis enega okvirja) ter `output_dir`. `state` je `running`, `ended` (se je sam ustavil in še ni bil ustavljen), `stopping` (ustavitev zapisuje svojo čakalno vrsto: statistike v realnem času) ter `stopped_by`) ali `finished` (njene končne statistike, ustavitev je `error` in `stopped_by`). Zapis ustavljenega opravila se hrani 10 minut (največ zadnjih 32), za vsakega odjemalca: drug skript ali `chloros-mcp` prebere isto stanje in končne statistike. Opravilo `chloros-cli` se je začelo (`lattice continuous`, `multi-stream`, `align-stream`) zadnji del sistema ustavi, ko se CLI30 sekund ne javi — prekinjeno, računalnik v stanju mirovanja — z `stop_reason`, `client_gone` in `stopped_by` v zadnjem delu sistema; branje njegovih statistik od tu ga ne ohranja aktivnega. Naloga, ki se zažene iz SDK, teče, dokler ni ustavljena. Vsak zagon pošlje svoj `timeout` kot rok backenda (`client_deadline_s`): kamera, ki se odpre po tem (tista, ki se še zaganjata), se vrne in zapre, naloga pa se ne zažene, klic pa sproži `ChlorosConnectError` (`CLIENT_DEADLINE_PASSED`). Kamera, ki jo zadržuje naloga, se zavrne vsaki drugi odprti (`connect_camera()`, drugi zagon, prehodno zajemanje) z napako HTTP 409 `CAMERA_HELD`, ki navaja nalogo in tistega, ki jo je začel. |
| `stop(timeout=30.0)` | Ustavi jo in zapri njene kamere: `final_stats`, `output_dir` in `error`, če zaustavitev ni potekala brez težav. Drugi klic vrne enak odgovor, enako pa velja tudi za zaustavitev s strani drugega odjemalca: tista, ki je poslana, medtem ko prva še vedno zapisuje svojo čakalno vrsto, na njo počaka (do 25 s; po tem času sproži izjemo, končne statistike pa sledijo v `stats()`). |
| `wait(timeout=None, poll_s=0.5)` | Počakajte, da se samodejno konča (dosegel je `count`, poln pogon, napaka), nato pa ga `stop()` za zbiranje končnih številk. `timeout` sproži `TimeoutError`, medtem ko ta še vedno teče; naloga brez `count` se konča šele, ko jo ustavite. |
| `job_id`, `kind`, `output_dir`, `start_reply`, `stopped` | Njegova identifikacijska številka in vrsta (`continuous`, `multi-stream`, `align-stream`), kam piše, odgovor začetne poti in ali je `stop()` že bil izveden. |

Uporabite ga kot upravitelja konteksta, da se ustavi ob izhodu iz obsega.

---

## Projektna mapa in recepti

**Projektna mapa** je mapa, v kateri se ustvarjajo novi projekti, ki jo skupaj uporabljajo aplikacija Chloros, CLI in SDK (modul `chloros_sdk.system_utils`; `chloros-cli project-folder`):

```python
print(chloros_sdk.project_folder())                # the configured folder, else the default
chloros_sdk.set_project_folder("D:/Flights")       # created if missing
chloros_sdk.reset_project_folder()                 # -> {"path": <default>, "was_set": True}
```

Če konfigurirana mapa ne obstaja več, se uporabi privzeta mapa (`Documents/MAPIR/Chloros Projects`), tako kot pri aplikaciji. Relativna pot, navedena v `set_project_folder()`, se vzame iz vašega delovnega imenika, če je strežnik na tem računalniku.

**Recept** je seznam akcij v formatu YAML/JSON, ki ga `chloros-cli project run PATH RECIPE` izvede na shranjenem projektu. `run_project_recipe()` ga izvede z CLI-jevimlastnim izvajalnikom (nameščenim z SDK), zato se en recept obnaša enako ne glede na to, ali gre za:

```python
r = chloros_sdk.run_project_recipe("D:/campaign/morning.yaml", "D:/Projects/Field_A")
print(r["ok"], r["actions"])
print(r["log"])                                    # what the CLI would have printed

chloros_sdk.run_project_recipe({"actions": [{"apply": {"name": "FrontLeft", "settings": {"exposure_us": 5000}}},
                                            {"wait": "2s"},
                                            {"capture": {"name": "FrontLeft", "output": "pose_a/", "format": "tiff"}}]},
                               "D:/Projects/Field_A")
```

```python
run_project_recipe(recipe, project_path, *, dry_run=False, echo=False,
                   backend_url="http://127.0.0.1:5000") -> dict
```

- **`recipe`** je datoteka `.yaml` / `.yml` / `.json` ali sam recept: slovar z `actions` (`apply`, `capture`, `burst`, `stream`, `sensor`, `wait`) ter po želji `overrides` in `stop_on_error` (privzeto true: izvajanje se ustavi ob prvem neuspešnem dejanju). Dejanja so opisana v referenčnem priročniku za CLI [Recipe DSL](cli-reference.md#recipe-dsl); relativna pot `output` se razreši glede na delovno mapo vašega skripta.
- **Vrne** `ok` (vse akcije so uspele), `actions` (koliko), `log` (izhod izvajalnika; `echo=True` ga med izvajanjem tudi izpiše) ter `validation`. Če datoteka z receptom manjka ali se ne more razčleniti, se sproži `ChlorosConnectError`.
- **Preverjeno pred zagonom.**`validation` je `{"ok", "errors", "warnings", "steps"}`: vsaka akcija mora imeti kot ključ natanko en znan glagol (`{"capture": {"name": ...}}`, ne `{"action": "capture"}` ali `{"type": "capture"}`), ki poimenuje kamero, niz ali senzor, ki ga ima projekt, in je prave vrste. Recept z napako**ne** izvede nobenega dejanja (`ok` false, napake v `log`), ne glede na to, kaj navaja `stop_on_error`. Neuporabljeni ključi, in vsak niz ali svetlobni senzor, ki ga recept upravlja, a trenutno ni priključen, so `warnings`. Recept ne priključuje naprav: naprave priključite v projektu (`ChlorosProject.connect_all()`, `chloros-cli project connect`, ali orodje MCP `connect_project_devices`). `dry_run=True` ne izvaja ničesar in vrne `{"recipe": ..., "validation": ...}`.
- V procesu se hkrati izvaja le en recept. Recept v formatu YAML potrebuje PyYAML.

---

## Avtomatizacija projekta — `ChlorosProject`

Shranjeni projekt Chloros je mapa, ki vsebuje `cameras.json` + `sensors.json` + `project.json`. `open_project(path, relink=None)` naloži manifest (`relink`: mapa, v kateri so zdaj manjkajoče izvorne datoteke premaknjenega projekta, ki se uporablja, ko ga backend naloži — `process()` — glej **Premaknjen projekt** pod `ChlorosLocal`), `connect_all` pa vključi vse shranjene kamere, matrike in svetlobne senzorje z njihovimi shranjenimi nastavitvami — v enakem stanju strojne opreme, kot bi ga ustvaril grafični vmesnik. Svetlobni senzorji se povezujejo prek [skladišče senzorjev DAQ](#daq-sensor-sessions), zato potrebujejo delujoč backend (glej [`SensorHandle`](#sensorhandle)).

### Minimalni primer

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Light sensors are addressable by name OR slot_id
reading = proj.sensors["Sky"].read()   # (spectrum, is_saturated, ...) or None

# One frame from every camera and array, one spectrum from every sensor
proj.capture_all("./out")

proj.disconnect_all()
```

Ali kot upravitelj konteksta:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### Metode `ChlorosProject`

| Metoda | Opis |
| --- | --- |
| `connect_all(*, cameras=True, arrays=True, sensors=True, verbose=False, align=None, packed=None)` | Odkrije in poveže vse shranjene kamere, matrike in svetlobne senzorje. Vrne poročilo o povezavah po razredih (`{"cameras": {...}, "arrays": {...}, "sensors": {...}}`, vsaka z `connected`, `missing` in `errors`; `arrays` vključuje tudi `alignment`). Če strežnik prisluhne na `127.0.0.1:5000`, se shranjene kamere in matrike poiščejo in povežejo prek njega. Če ga ni, ta proces sam poišče in jih upravlja prek `lattice_sdk`, ki potrebuje izvedbeno okolje Arena SDK in OpenCV (`cv2`) – kolo SDK ne namesti nobenega od njiju; če tega ne more storiti, so vse shranjene kamere in matrike navedene pod `errors` z navedbo razloga, `missing` pa ostane prazen. Backend, ki teče, a zavrača SDK (odjavljen: HTTP 401; načrt pod Copper: 403), ni »brez backenda«: vsaka shranjena kamera, matrika in svetlobni senzor so navedeni pod `errors` z razlogom backenda, v tem procesu pa ni nič odprto. Vsak klic ponovno preveri strežnik, zato se `connect_all()` po prijavi ali po zagonu programa Chloros poveže z istim projektnim objektom. Shranjena matrica, ki je že vzpostavljena v backendu z istimi kamerami (iz grafičnega vmesnika ali iz `chloros-cli project connect`), se priključi takšna, kot je, brez ponovne uporabe shranjenih nastavitev, `disconnect_all()` pa ga pusti delovati. Senzorji svetlobe vedno potekajo prek ozadja (glej [`SensorHandle`](#sensorhandle)). Nikoli ne ustvari ozadja. |
| `disconnect_all()` | Razgradi vse. Najprej se sprosti vsaka naprava; nato, če backend ni potrdil sprostitve kamere, niza ali svetlobnega senzorja (časovna omejitev razgradnje je potekla ali je bila zavrnjena), vsaka od njih dobi ime `ChlorosConnectError`. Naprava, ki je že zaprta, ali če ne teče noben backend, je tiho. Matrika ali svetlobni senzor, ki ga je imel odprt drug odjemalec, ostane v teku. Izhod projekta `with` se nikoli ne sproži. |
| `capture_all(output_dir=".")` | En okvir iz vsake povezane kamere + matrike, in en spekter (`SensorHandle.read()`) iz vsakega povezanega svetlobnega senzorja, zajeti po vrsti (kamere, nato matrice, nato senzorji), ne v istem trenutku. |
| `stream(camera, overlays=False, fps=10.0)` | Generator, ki ustvarja BGR-okvirje `numpy` iz imenovane kamere (ali matrike). `overlays=False` bere okvirje z ročajem`grab()` na ravni `debayered`: prek backenda, če je projekt povezan prek njega, sicer pa prek neposredne zanke zajemanja `lattice_sdk` (nizi ustvarjajo slovarje `{serial: frame}`). `overlays=True` usmerja prek `ChlorosLocal.camera_stream()` → MJPEG-vir backenda `/api/camera/<serial>/stream-annotated`, pri čemer se shranjeni blok kamere `ui.overlay` posreduje kot parametri poizvedbe. Zahteva način backenda, OpenCV (`cv2`) in **samostojno kamero**: kamera v neposrednem načinu sproži `RuntimeError` (backend ne more prevzeti kamere, ki pripada temu procesu), niz pa sproži `NotImplementedError` (prekriva kompozit po kameri — pretaka element po imenu). Enkratni ekvivalent: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Izvedi poravnavo za vsak trenutno priključen niz. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Izvedi kalibracijski/indeksni proces na slikah projekta (zavije `ChlorosLocal.process`; ti štirje so **edini** sprejeti ključni argumenti — `indices=` itd. povzroči izjemo `TypeError`; nastavi indekse prek `ChlorosLocal.configure()`). Lenično ustvari `ChlorosLocal()`, ki samodejno zažene backend. |

Atributi:
- `proj.cameras` — `Dict[str, CameraHandle]`, indeksiran po imenu IN serijski številki.
- `proj.arrays` — `Dict[str, ArrayHandle]`, indeksiran po imenu IN array_id.
- `proj.sensors` — `Dict[str, SensorHandle]`, indeksiran po imenu IN slot_id.
- `proj.config` — `project.json["config"]` slovar.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Ravni obdelave.** `capture()`, `grab()` in `frame_stream()` vsi uporabljajo isti `processing`, veriga pa je kumulativna — vsaka stopnja izvaja vse, kar je nad njo:

| Stopnja | Izhod | Opombe |
| --- | --- | --- |
| `raw` | 1-kanalni Bayer, senzorski izvorni | Brez demosaika. Prekrivni sloji na tej stopnji niso na voljo. |
| `debayered` | 3-kanalni barvni (**privzeto** za `CameraHandle` in vsak `grab()` / `frame_stream()`; `ArrayHandle.capture()`, `capture_smart()` in `stream()` so privzeto nastavljeni na `raw`): BGR v pomnilniku (`grab()` / `frame_stream()`), shranjeno kot RGB s pomočjo `capture()` | V načinu backend, kot zajem: temni odmik (DSNU) kalibracijskega paketa od surovega podatka, nato pa OpenCV-jevo demosaiciranje z upoštevanjem robov — brez LUT-a za linearnost, izravnave polja ali ločevanja barv. Edina raven, ki deluje brez načina backend, kjer gre za navadno bilinearno demosaiciranje knjižnice kamere (brez temnega odmika). Shranjeni TIFF je fotometrični RGB, zato obrnite kanale okvirja v pomnilniku (`frame[..., ::-1]`), da ga primerjate s datoteko. |
| `radiance` | float32, W/m²/sr/nm, **ena ravnina na pas** | Celotna radiometrična veriga: demosaik + 3×3 ločevanje (multispektralno) + DSNU + flat-field + NIST-lestvica, pri čemer sta osvetlitev × ojačanje izločena, tako da so vrednosti absolutne. |
| `reflectance` | uint16, 32768 = 1,0 (ali uint8, 255 = 1,0 — glej `bit_depth`), **ena ravnina na pas** | Sijaj, deljen z navzdolnim sevanjem (ρ = π·L/E). Potrebna je vrednost DLS/DAQ — glej opombo spodaj. |
| `display` | 8-bitni, podoben sRGB | upodabljanje, enakovredno GUI: CCM + bela bilanca + gama prek aktivnega barvnega profila kamere. |

V neposrednem (brez backenda) načinu je na voljo le `debayered`: `grab()` na drugi ravni povzroči, da
`NotImplementedError`, `capture()` prezre `processing`, `frame_stream()` pa sproži
`RuntimeError` na vsaki ravni. `reflectance` potrebuje uporabno vrednost navzdolnega sevanja — končna točka okvirja samodejno potegne
združeni DAQ v režo DLS kamere, vendar brez vezanega DAQ-ja veriga zavrne
izhod odbojnosti in pošteno označi znižanje v vrnjenih metapodatkih, namesto da bi tiho
vrnila slabši izdelek.

> **Lestvica DN odbojnosti – ne jo trdno kodirajte.** Reflektanca LATTICE uporablja `32768` = ρ 1,0 in označi
> XMP `Chloros:PixelScale=32768`; reflektanca Survey3 uporablja `65535` = ρ 1,0 ter ne vsebuje
> oznake `Chloros:*`. Preberite oznako in delite z njo. Opredeljena je v domeni uint16, zato ostane
> `32768` za vsak format, ki spreminja merilo (16-bitni TIFF, 8-bitni TIFF, 8-bitni PNG/JPG, 32-bitni odstotek)
> — najprej normalizirajte shranjeni podatkovni tip nazaj na uint16 (×257 iz 8-bitnega, ×65535 iz float). Edina
> izjema je datoteka iz različice pred 1.3.0: 8-bitni TIFF je bil takrat *odrezan*, ne pa prilagojen, zato
> ga ne opisuje nobena lestvica in ne vsebuje niza `PixelScale` ali MicaSense. Manjkajočo oznako v
> datoteki odbojnosti LATTICE obravnavajte kot »ni veljavne lestvice«, ne kot privzeto vrednost.

**Oznake kalibracije: samo na `raw`.** Datoteka `raw` posnetka vsebuje kalibracijski paket kamere: koristni tovor `CalibrationJSON` in oznake, združljive z MicaSense (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), ki opisujejo surove vrednosti senzorja. Vsaka datoteka drugih ravni (`debayered`, `display`, `radiance`, `reflectance`, render `index/`) ter vsak posnetek pri neprekinjenem snemanju / večkamernih tokov vsebuje identifikacijsko številko kamere (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) ter lastne podatke o posnetku, `XMP-mapir:FrameJSON` (osvetlitev, ojačanje, temperatura, format pikslov, čas zajema, ujemanje DAQ), saj ima vsak izdelek `process()` svojo identifikacijo. Obdelava poišče kalibracijo takšne datoteke po njeni serijski številki in `CalBundleSha256` v kalibracijskem predpomnilniku programa Chloros ter paket prenese enkrat, če ga ta računalnik nima.

#### Datoteke, ustvarjene iz več kamer

Datoteka, ustvarjena iz pasov več kamer – večpasovni niz (`capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index(save_multiband=True)`, `Registration.save_multiband()`), indeks v realnem času, sestavljen iz več kamer (`index.tif`, `index_LUT.png`), slika `composite/` iz zajema v matrici — navaja vsako kamero, ki je bila vključena vanjo:

- **Na-seznam po kamerah, po pasovnem vrstnem redu** (kompozit: najprej ozadje, nato prednji plan; indeksni sloj, kamere, iz katerih izvirajo njeni pasovi): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (kalibracijski paket, skozi katerega so šli pasovi vsake kamere, sicer pa — niz `capture_aligned_multiband()` / `start_aligned_stream()`, katerega pasovi ne gredo skozi nobenega — paket kamere, ki ga je Chloros shranil v predpomnilnik, tisti, po katerem so poimenovane njene datoteke posnetkov), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — vzporedni seznami, pri čemer *n*-ti element vsakega opisuje isto kamero; `-`, kjer kamera nima vrednosti (brez kalibracijskega paketa, neznan objektiv). Vsaka kamera je **en** element: če ima njen paket več vrednosti za določeno polje — certifikati za halogenske žarnice v paketu — so združeni z `;` (`F-1479-1;F-1480-1;F-1481-1`), nikoli z vejico, zato se seznam bere kot ena postavka na kamero, ne glede na to, kako je natisnjen.
- **EXIF `SerialNumber`** vsebuje iste serijske številke, povezane z vejico (`214000535,214000551`).
- **Ne** vsebuje identifikacijskih podatkov za posamezno kamero (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): ti podatki poimenujejo eno kamero in njen paket, obdelava pa bi na podlagi njih določila kalibracijo datoteke. Indeks, izračunan iz pasov več kamer (`index.tif`, `index_LUT.png`) vsebuje EXIF `Model` `LATTICE` (oprema), ne pa model `LATT-…` ene same kamere.

Datoteka pasov **ene** kamere – TIFF datoteka za vsak pas, `capture_aligned_multiband(per_camera=True)` okvir, niz ali indeks v realnem času ene kamere – vsebuje identifikacijske oznake te kamere, tako kot vsaka druga datoteka posnetka. Niz `capture_aligned_multiband()`, datoteke po posameznih pasovih in po posameznih kamerah ter vsak niz `start_aligned_stream()` in njegove datoteke po posameznih pasovih vsebujejo čas zajema svoje skupine (osvetlitev referenčne kamere, UTC), okvir `per_camera` enokanalne kamere pa je enoslojen, pri `bit_depth` podobno kot niz.**Usmerjenost kamere (obrnitev / rotacija).** Okvirji iz `grab()` / `frame_stream()` so vedno v
izvirni usmeritvi kamere na vseh ravneh. Shranjeni posnetki sledijo nastavitvi usmeritve kamere
(v grafičnem vmesniku Mirror H / Mirror V / rotacija):

- **Datoteke `raw` in `debayered` so točno tisto, kar je senzor odčital**, ne glede na nastavitev – Bayerjev
  mozaik, ki je zavrt ali obrnjen, bi predstavljal drugačen vzorec barvnih filtrov, kalibracijske karte pa so
  na lastni mreži senzorja.
- **Datoteke `display`, `radiance`, `reflectance` in indeksne datoteke so usmerjene** – enkrat zavrtene po vseh
  kalibracijskih izračunov. Njihove maske omejevanja so na isti mreži.
- **Vsaka datoteka posnetka beleži nastavitev** v XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, v smeri urinega kazalca, po
  preobratih) ter `ChlorosOrientationApplied` — `false` na surovih in debayered datotekah, `true` na orientiranih
  izdelkih. `ChlorosProcessing` navaja operacije, uporabljene na pikslah te datoteke (`none`, če je
  v izvirni obliki). To vključuje vsak posnetek neprekinjenega snemanja (lastna nastavitev kamere,
  neuporabljena) in večkamernega toka (identiteta: njegovi posnetki so izvirni, kot pri
  zajemu matrike).
- **Obdelava projekta uporabi zabeleženo usmeritev** za izdelke, ki jih ustvari iz surovega
  (najprej kalibracija v koordinatah senzorja, nato izravnava matrike, nazadnje orientacija),
  tako da se ujemajo s produkti zajema v živo — vključno z nekalibriranimi produkti z odzivom senzorja / popravljenimi glede na vinjeto; njegov produkt brez bayerjevega filtra ostane v izvirni obliki.
- **Chlorosov pregledovalnik slik prikazuje surovo ali debayirano datoteko, ki je bila obrnjena v skladu z njenim zapisom** – le za prikaz,
  datoteka se ne spremeni; datoteka brez zapisa se prikaže takšna, kot je shranjena.
- **Posnetki brez zapisa se obdelujejo kot izvirni.** Pred različico 1.3.0 je zajem z eno kamero
  sam obrnil svoje surove in debayirane datoteke; program Chloros ne more razlikovati takšne surove datoteke od izvirne in
  ne ugiba.
- **Zajemi v nizu ohranijo vsako posamezno datoteko privzeto v izvirni obliki** in zabeležijo identiteto;
  `ArraySession.capture(orient=True)` pretvori njihove izdelke (glej nadzorne elemente za zajem niza).

#### Izhod po pasovih in mešani nizi

Na ravneh `radiance` in `reflectance` kamera vrne **eno ravnino na pas te
kamere** — tri za M3C (njene tri nepomešane pasove, vrstni red kanalov BGR), eno za M3M (njegov
en sam ozkopasovni filter). Replicirane ravnine mono kamere so zložene, zato **mešani
niz M3C/M3M daje različne oblike nizov za vsako serijsko številko**. Preberite ključ `bands` iz metapodatkov okvirja, da ugotovite, kaj je posamezna ravnina; tega ne sklepajte iz števila kanalov:

```python
for serial, (arr, hdr) in arr_handle.grab(processing="reflectance",
                                          with_metadata=True).items():
    print(serial, arr.shape, hdr["bands"])   # ('…', (1536, 2048, 3), ['nir','green','red'])
                                             # ('…', (1536, 2048, 1), ['F685'])
    rho = arr.astype("float32") / hdr["pixel_scale"]
```

Pred uporabo vrednosti vedno preverite `hdr["processing_level"] == "reflectance"`. Ponižanje
(brez odčitka DAQ, brez kalibracijskega paketa, neznana razporeditev pasov) se poroča v `processing_level` in
`radiometric_refused` namesto povišanja – niz, ki ga dobite nazaj, je še vedno veljaven okvir, le
da ni tisti, ki ste ga zahtevali. Shranjen posnetek, ki je bil razvrščen nižje, nosi v svojem XMP-ju isto kodo
kot `Chloros:RadiometricRefused` (npr. `no-band-layout`, `no-dls-snapshot`).

#### En sprožilec na odčitek — `group_id`

**Člani niza se prenašajo iz sinhronizirane skupine okvirjev niza, ne iz lastnega
zabojnika posnetkov vsake kamere.** Kamere strojne sinhroniziranega niza se izpostavljajo skupaj, vendar N branj na kamero
se ne *vzorčijo* skupaj — branje najnovejšega okvirja v zabojniku vsake kamere presega meje sprožilca
meje, en sam izpuščen okvir pa desinhronizira niz, ne da bi to kdor koli opazil.

Pridobivanje iz skupine je nujno, vendar samo po sebi ni zadostno: vsaka zahteva še vedno
posname *najnovejšo* skupino v svojem trenutku, medtem ko lahko zanka zajemanja vmes objavi nove slike.
Pri merjenju na delujočem nizu s 4 kamerami je **12 od 20 vzporednih branj segalo čez dve skupini** — vedno
natančno sosednji. Zato je branje **pritrjeno**:

1. Vsak okvir nosi `group_id`, monotonični števec objav sistema.
2. `grab()` prebere glavno kamero (ali, če glavna kamera ni na voljo, naslednjo kamero, ki je),
   nato pa preostale kamere pritrdi na identifikator skupine, na kateri se je ustavil (`?group_id=N` na HTTP
   končni točki).
3. Backend hrani nedavne skupine **vsaj pol sekunde**, zato se brat, ki ga je že
   prehitela kasnejša objava, še vedno obdeluje kot sprožilec — dovolj dolgo, da se lahko izvede
   izvoz odbojnosti, tudi pri polni senzorski matrici z 9 kamerami. Pripetost, ki je strežnik ne
   hrani več, se zavrne s kodama `409` in `group_aged_out: true`, namesto da bi se tiho odgovorilo
   z drugačnim sprožilcem – `grab()` ponovno prebere niz.

Ponovno izmerjeno s fiksiranjem: **20 od 20 odčitkov je vrnilo en sprožilec.** `grab()` to opravi za vas;
`group_id=` potrebujete le, če neposredno upravljate s končno točko HTTP.

- Kamera, ki je ni mogoče obdelati v skupini s fiksiranimi kamerami — ker v njej manjka ali zavrača zahtevo
  (`band`, ki ga nima, ali raven, ki je ne more ustvariti) — je **izpuščena, namesto da bi bila nadomeščena**
  iz drugega sprožilca, druge kamere pa so še vedno vrnjene.
- Slovar, ki ga vrne `grab()`, je `GrabResult` (navadni podrazred `dict`;
  `from chloros_sdk import GrabResult`): `.errors` vsako
  izpuščeno serijsko številko preslika na izjemo, ki jo je sprožilo branje, `.group_id` pa je skupna skupina. Prazna
  `.errors` pomeni, da so vse kamere sestavljale to skupino.
- Samostojna kamera nima skupine, poroča `group_id: None` in **zavrne** `group_id=`
  pin, namesto da ga ignorira.

```python
frames = arr_handle.grab(processing="reflectance")
for serial, err in frames.errors.items():
    print(f"SN:{serial} omitted: {err}")       # e.g. status=400 ... band 'F685' not in [...]

frames = arr_handle.grab(processing="reflectance", with_metadata=True)
ids = {hdr["group_id"] for _a, hdr in frames.values()}
assert len(ids) == 1, f"straddled triggers: {ids}"     # belt and braces
```

Isti števec se objavi v **vsakem MJPEG-toku posamezne kamere** člana matrike — v
predogledu (`/api/camera/stream/<serial>`), v predogledu z opombami (`/stream-annotated`) in v
tok odbojnosti (`/stream-reflectance`) – kot `X-Chloros-Group-Id` na vsakem delu, poleg
`X-Chloros-Frame-Id` in `X-Chloros-Timestamp-Ns`. Tokovi so neprekinjeni in ne
točkovni, zato se usklajujejo z zbiranjem delov in združevanjem na podlagi tega glava – ni
potrebno nastavljati nobenega pina. Samostojna kamera nima skupine. Glej
[MJPEG-tokovi po posameznih kamerah](#per-camera-mjpeg-streams-no-rtsp).

**`frame_stream()` nikoli ne utihne.** Znak, ki ne uspe zaradi razloga, ki se lahko samodejno odpravi,
se preskoči – tok kamere zanj ne prinese ničesar, tok niza pa prinese kamere, ki so
odgovorile (morda nobene). Ti razlogi so: še ni okvirja; nit zajemanja kamere je zaustavljena za
sekundo ali dve (`404 not streaming`, na kar backend odgovori, kadar se priključi
ali odključi druga kamera ali pa sprememba nastavitev ponovno uravnoteži sklop); ponovni zagon strežnika ali `5xx`; poravnava
se še vedno stabilizira (`409 … retry shortly`); in zavrnitev odbojnosti, ker DAQ še nima odčitka
(`no-dls-snapshot`, zastarel ali prazen odčitek). Zavrnitev v naslednjem taktu bi se le ponovila —
neznani `band`, `bit_depth=8` pri sevanju, kalibracijski paket brez lestvice sevanja, zavrnitev odbojnosti
z **popolno odsotnostjo DAQ** (noben ni vezan na kamero ali njen niz, noben ni povezan z
backendom: `dls_source` zavrnitve je `"none"`), kateri koli drug `4xx` —
povzroči `chloros_sdk.FrameFetchError` (`RuntimeError`, ki prenaša `.status` in `.body`) na
enkrat. Enako velja za **60 sekund** preskočenih takov brez okvirja vmes, ne glede na `fps` — dovolj
dolgo, da se izčaka poravnava in ponovni zagon strežnika, dovolj kratko, da kamera, ki je
resnično izpadla, prekine prenos. Ta napaka ob prekinitvi je prav tako `FrameFetchError`, tako za kamero kot za
niz, pri čemer je izjema zadnjega branja `.errors` (in, za kamero, kot `__cause__`). Za
niz »zavrnjeno« pomeni, da je vsaka kamera zavrnila; izjema `.errors` vsebuje
razlog za vsako kamero.

**Niz `frame_stream()` vsak sprožilec izvede enkrat** (tako `ArrayHandle` kot `ArraySession`).
Tik, katerega najnovejši niz je sprožilec, ki ga je že vrnil, počaka na naslednji sprožilec, namesto da bi
ponovno vrnil isti niz, zato tok teče s hitrostjo, ki je manjša od `fps` ali lastne hitrosti niza, zaporedni sklopi pa si nikoli ne delijo `group_id`. Niz, ki preneha objavljati, sproži
`FrameFetchError` („60 sekund brez novega sprožilca“) namesto da bi ponovil svoj zadnji niz. Brez tega bi
tok, ki je zahteval več, kot je niz objavil – ali se je začel takoj po povezavi –, razdelil
ponovitve: v enem izmerjenem izvajanju je 20 sklopov vsebovalo 11 različnih identifikatorjev skupin.

```python
from chloros_sdk import FrameFetchError

try:
    for arr in cam.frame_stream(processing="radiance", bit_depth=8):
        my_analysis(arr)
except FrameFetchError as exc:
    print(exc.status, exc.body)        # the HTTP status and the backend's reason
```

#### Izvoz enega samega pasu — `band`

`grab()` / `frame_stream()` uporabita `band="<name>"` za izvoz le enega pasu kamere kot
enokanalnega niza `(H, W, 1)` — enake oblike, kot jo proizvaja monokamera, zato se izhod enega pasu
obravnava enako, ne glede na to, ali izhaja iz M3M ali iz razdelitve M3C. Rdeči pas FRGN:

```python
red = cam.grab(processing="reflectance", band="red")        # (1536, 2048, 1)
```

Sprejemljive zapise so polno ime razporeda (`Red_660`, `NIR_850`, `Green_550`) ali njegova kratka
glava (`red`, `nir`, `green`), pri čemer se ne upošteva velika in mala črka. Neznano ime se zavrne z navedbo razpoložljivih
imen.

> **Izbira poteka po imenu, ne po številki kanala – namerno.** »Kanal 1« je `Green_550` v
> vrstnem redu kanalov BGR matrike, vendar *rdeč* v vrstnem redu črk v kodi filtra(`RGN`), in ti dve
> se ne ujemata. Izbor napačnega prinese popolnoma verjeten okvir napačnega pasu, ki ga
> nič v nadaljevanju ne more zaznati. Zato ni številčnega indeksa.

#### Usklajevanje tokov — `align`

`align=True` prilagodi sliko v skupni referenčni okvir svojega niza. Vsaka kamera se prilagodi
prek **iste** ločljivosti profila in izrezanega območja zanimanja (ROI), tako da so tokovi posameznih kamer, odprti neodvisno,**medsebojno usklajeni** — ni jih treba zahtevati skupaj:

```python
frames = arr_handle.grab(processing="reflectance", align=True)   # {serial: array}
# or one MJPEG stream per camera, each co-registered on its own:
#   GET /api/camera/<serial>/stream-reflectance?align=1
```

Zahteve in delovanje:

- Kamera mora biti v **strojno sinhronizirani** matrici (sinhronizacijski kabel M8), ki ima profil poravnave
  — so-registracija predpostavlja, da so člani izpostavljeni sočasno, česar matrica s prostim tekom ali
  niz v prostem teku ne zagotavlja. V nasprotnem primeru bo zahteva zavrnjena z napako `409`.
- **Zavrnjeno, nikoli tiho neusklajeno.** Pot zajemanja lahko shrani neusklajene posnetke z opozorilom,
  ker datoteka ohrani oznake izvora in jo je mogoče kasneje ponovno registrirati; v živo prenašani tok nima
  takšne druge priložnosti, zato je manjkajoči profil napaka in ne nadomestna rešitev.
- **Zahteva za poravnavo jo sproži.** Zahteva za `align` na nizu brez profila sproži
  kalibracijo in vrne `409` s tem, na kar čaka (npr. *»čakam, da se osvetlitev
  ustalijo na 214000551 (1/3)«*) — poskusite znova in uspe, običajno v nekaj desetih sekundah.
  To je pomembno pri brezzaslonskem delovanju: samodejno poravnavanje ob vzpostavitvi povezave se po 30 sekundah prekine, vsak poskus
  v tem časovnem oknu se preskoči, medtem ko se pametna avtomatska osvetlitev (smart-AE) še vedno stabilizira, zato je niz, katerega osvetlitev
  se počasi stabilizira, prej ostal brez profila za celotno sejo in brez možnosti, da bi ga
  pridobili brez odpiranja združenega pogleda na namizju. `alignment_status` poroča tudi o
  `ae_align_holdouts`, pri čemer navaja kamere, ki se še stabilizirajo, in njihov napredek.
- Vsakemu toku ene matrike dodelite **isti `max_h`**, sicer se bodo njihovi izrezi znašli na različnih pikselnih
  mrežah.
- Poravnava se uporabi *pred* kodiranjem bitne globine, zato deformacija interpolira vrednosti s polno natančnostjo.

#### `bit_depth` — zamenjava natančnosti za pasovno širino

`grab()` in `frame_stream()` sprejemata `bit_depth=16` (privzeto) ali `bit_depth=8`:

| `bit_depth` | Kodiranje odbojnosti | Stroški |
| --- | --- | --- |
| `16` | uint16, 32768 = ρ 1,0, rezerva do ρ 2,0 | Polna natančnost. |
| `8` | uint8, ki zajema `0..refl_max` (privzeto ρ 1,0) | Polovica bajtov; ~0,004 kvantizacija ρ pri privzeti zgornji meji. Vrednosti nad zgornjo mejo se odrežejo — glej spodaj. |

##### 8-bitna zgornja meja — `refl_max`

`refl_max` določa vrednost ρ, ki jo zajema 8-bitno območje, tako da je `pixel_scale = 255 / refl_max` in ρ še vedno
`pixel / pixel_scale`. `grab()` in `frame_stream()` ga obravnavata kot ključno besedo tako na kameri kot
nizov (`?refl_max=` na HTTP-končnem točki). Povečajte ga, da pridobite rezervo za odbojne
svetlobne poudarke, v sorazmerno grobih korakih:

```python
frame = cam.grab(processing="reflectance", bit_depth=8, refl_max=1.5)   # 0..1.5 in 0..255
```

| `refl_max` | `pixel_scale` | korak |
| --- | --- | --- |
| `1.0` (privzeto) | 255 | 0,0039 ρ |
| `1.5` | 170 | 0,0059 ρ |
| `2.0` | 127,5 | 0,0078 ρ |

> **Samodejna osvetlitev ne more popraviti 8-bitnega izreza, poskus z njo pa podatke še poslabša.**
> ρ = π·L/E, pri čemer sta osvetlitev in ojačitev izločeni, zato je ρ lastnost prizora, ne pa
> nastavitve kamere — merjeno na aktivni matrici je zmanjšanje osvetlitve za polovico povzročilo premik srednje vrednosti ρ za 1,3 %. Povečanje
> osvetlitve, da bi se »prilagodila« razponu, namesto tega povzroči zasičenje pikslov v **senzorju**, kar
> podcenjuje L in povzroča **nizko** vrednost ρ brez prikazane napake (izmerjeno: maks. ρ 1,203 → 0,985 pri
> 2× osvetlitvi, povprečje −23 %). Izberite razpon z `refl_max`; preprečevanje
> zasičenosti senzorja je že naloga avtomatske ekspozicije (AE), ki ščiti *veljavnost* vrednosti ρ in ne njenega razpona.

**Preberite `reflectance_clipped_fraction`, ne oznako.** `reflectance_clipped_at_1_0` se sproži že pri
enem samem pikselu — v resnični sceni je poročal o vrednosti `True`, medtem ko je bil delež `0.000001`, en piksel
na milijon. Delež je tisti, ki vam pove, ali je treba zvišati zgornjo mejo.

**`bit_depth=8` se nanaša izključno na odbojnost.** Zahteva zanj za `radiance` se zavrne, namesto da bi se
izpolnila: odbojnost ima fiksno območje 0..1, ki ostane nespremenjeno pri 8-bitnem kodiranju, medtem ko sevanje nima
fiksnega območja, zato je za prilagoditev v 8 bitov potrebno raztezanje po percentilih za vsak posamezni okvir, pri čemer se preslikava
spreminja z vsakim okvirjem — gre za sliko, ne za meritev.

#### Tok odbojnosti v realnem času (MJPEG)

`GET /api/camera/<serial>/stream-reflectance` prenaša 8-bitno odbojnost po pasovih kot
`multipart/x-mixed-replace` — M3C kot 3-kanalni JPEG, M3M kot siva lestvica. Parametri poizvedbe:
`fps` (privzeto 10), `jpeg_q` (privzeto 95), `max_h` (0 = izvirno), `band` (en poimenovan pas, kot
zgoraj), `align` (`1` za so-registracijo — ista pravila kot za `align` v SDK-ju), `refl_max` (zgornja meja 8-bitnega
kodiranja, kot zgoraj). Vsak del nosi
`X-Chloros-Group-Id`, `X-Chloros-Frame-Id`, `X-Chloros-Timestamp-Ns`, `X-Chloros-Bands`,
`X-Chloros-Pixel-Scale`, `X-Chloros-Aligned` in `X-Chloros-Reflectance-Clipped`.
**Deli iz različnih kamer, ki nosijo isti `X-Chloros-Group-Id`, predstavljajo isti sprožilec** —
tako se ponovno sestavijo več tokov na kamero znotraj ene matrike, kar je zagotovljeno
na strani strežnika, saj se elementi matrike prenašajo iz sinhronizirane skupine okvirjev (glej
[En sprožilec na branje](#one-trigger-per-read--group_id)). Vrne `409`, če odbojnost ni
na voljo ob vzpostavitvi povezave, in prekine tok, če postane nedostopna med delovanjem, zato zastoj
toka pomeni »odbojnost ni več na voljo« in ne »še vedno v redu, le statično«.

> **Format JPEG je izgubni — ti piksli so meritev.** Napaka pri stiskanju se neposredno odraža v
> vrednostih odbojnosti in je najhujša na robovih, kjer so meje materialov. Kakovost
> je iz tega razloga privzeto nastavljena na 95. Če potrebujete neokrnjene vrednosti, uporabite
> `grab(processing="reflectance")` / `frame_stream(...)`, ki vrneta isti izdelek brez
> kodeka v poti (in `bit_depth=8`, če je omejitev strošek prenosa, ne pa kodek).

> **Podatki EXIF se prenesejo v izvoz.** `process()` kopira blok GPS izvirnega posnetka
> **in njegov ExifIFD** na vsak izdelek, zato izvozi vsebujejo `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` in `CameraSerialNumber` ter
> georeferenciranje. `FocalLength` je vrednost, na podlagi katere Pix4D izračuna razdaljo med vzorci na tleh – brez nje
> se rekonstrukcija vrne na povsem napačno merilo (v enem izmerjenem primeru se je lokacija velikosti 411 m
> spremenila v lokacijo velikosti 47,8 km). Kopija namerno ni `-all:all`: strukturne oznake IFD0 prekinjajo
> izhod LATTICE, `ExifImageWidth`/`Height` pa sta izključena, ker opisujeta izvorno
> zajemno sliko in ne izvožene rastrske slike.

Podoznake faze zajema (veljajo za radiometrične ravni — `radiance`, `reflectance`, `display`):

| Zastavica | Privzeto | Pomen |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + flat-field + 3x3 unmix + radiometrična lestvica NIST. |
| `apply_white_balance` | `True` | WB LUT. Upošteva DLS, če je DAQ povezan s kamero. |
| `apply_index` | `False` | Ocena indeksa vegetacije. |
| `index_expression` | `None` | Prepis formule. Če je polje-prazno → samodejno omogoči indeks. |
| `annotated` | `False` | Prekrivanje okraskov grafičnega vmesnika (zebra/mreža/vrhovi). Ni na voljo za `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Vrnjena vrednost je `chloros_sdk.CapturePaths`, tip `CapturePathMap`, ne `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` je `Dict[str, Union[str, List[str]]]`: enonivojski
> `processing` vsaki serijski številki dodeli eno pot, medtem ko `processing="all"` ji dodeli
> **urejen seznam** vseh izdelkov, shranjenih za to kamero. `ArrayHandle.capture()` ne sprejme
> ne sprejme `levels=` in prezre ključne argumente, ki jih ne pozna (`levels`, `daq`,
> `render_index`, `force_daq`), zato posredujte le tiste, ki so prikazani tukaj (`aligned=False`
> shranjuje izvorne datoteke, tako kot `ArraySession.capture()`); za te nadzorne elemente uporabite
> `ArraySession.capture()`. Kombinirani kompozit v realnem času, če se je predvajal, prispe pod dodatnim
> ključ `"combined"` in ne pod serijsko številko. Koda, ki predpostavlja `str`, se pokvari v
> obliki seznama, ne da bi kateri koli preverjevalec tipov ugovarjal — opomba je navajala `Dict[str, str]`
> še nekaj časa po izdaji oblike seznama, zato ta alias obstaja. Normalizirajte
> , če želite ravno obliko:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

> **Kar ni bilo mogoče zapisati: `.failed`.** Zemljevid, ki ga vrne `capture()`, je
> `chloros_sdk.CapturePaths` — navaden podrazred `dict`, zato vse zgoraj navedeno deluje
> nespremenjeno — ki vsebuje tudi `.failed`: datoteke, ki jih backend ni mogel zapisati,
> enake vnose kot [`CaptureResult.failed`](#datoteke-ki-jih-ni-bilo-mogoče-zapisati--captureresultfailed)
> (`serial`, `level`, `error` in `disk_full: True` na polnem disku). Delni
> zajem še vedno vrne vse poti, ki so bile zapisane; na polnem disku je zemljevid
> prazen, `.failed` pa pojasni, zakaj. `.skipped` navaja preskoke po posameznih kamerah,
> `.group_seq` pa sinhronizirano skupino, `.alignment` in `.oriented` pa sta
> podatki strežnika, kot je prikazano v `CaptureResult`. `capture_smart()` jih vrne kot
> `result["frames"].failed` in `result["failed"]`.
>
> ```python
> paths = arr.capture("./out", processing="all")
> if any(f.get("disk_full") for f in paths.failed):
>     raise SystemExit("drive full -- free space and capture again")
> ```

### Poravnava nizov

`ArrayHandle` razkrije celotno površino poravnave. Profili so privzeto veljavni le za tekoče seje – za trajno shranjevanje izrecno pokličite `export_alignment()`.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Manual control points: each camera's pixel coordinates of the same
# features, the reference camera's included (>= 2 per camera). Every one
# of num_frames is solved from these points.
arr.calibrate_alignment(method="manual", model="rigid", control_points={
    "214701173": [[412, 300], [1630, 310], [1020, 1180]],   # reference (master)
    "214701292": [[418, 297], [1637, 306], [1027, 1177]],
})

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Poravnava ob vzpostavitvi povezave

`connect_all(align=...)` lahko ob vzpostavitvi povezave samodejno poravna vsak niz:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Če ni drugače določeno, se uporabi `project.json["config"]["auto_align_on_connect"]`.

### `SensorHandle`

Shranjeni svetlobni senzor (DAQ-U, DAQ-M ali DAQ-E), ki ga `connect_all()` odpre v [skladu senzorjev DAQ](#daq-sensor-sessions) — istem skladu, ki ga uporabljata `connect_daq_sensor()` in `chloros-cli daq pool-connect`.

```python
reading = proj.sensors["Sky"].read()
if reading is not None:
    spectrum, is_saturated, integration_time, x, y, z = reading
```

- `read()` čaka na **naslednji** spekter senzorja — tisti, ki je bil že shranjen v pomnilniku ob klicu, se ne šteje — in vrne `(spectrum, is_saturated, integration_time, x, y, z)` ali `None`, če v 5 sekundah ne prispe noben. Zaustavljen tok se nadaljuje. `spectrum` je izražen v W/m²/nm, če je senzor kalibriran.
- `.sensor` je [`DAQSensorSession`](#daq-sensor-sessions), za vse ostalo: `latest(n=...)`, `status()`, `record_start()` / `record_stop()`.
- `.slot_id`, `.model`, `.serial`, `.name` in `.settings` so shranjeni vnosi iz `sensors.json`.

**Kaj obnovi `connect_all`.** Vsak senzor se odpre na svojem shranjenem naslovu — svojem COM-vmesniku, naslovu Bluetooth ali gostitelju DAQ-E — in, če je projekt zabeležil identifikator senzorja, mora biti to prav ta senzor: druga enota na tem naslovu se zavrne, saj ni povezana. Uporabijo se njegov shranjeni integracijski čas, povprečenje okvirjev in samodejna osvetlitev ter njegova korekcija pokrivnosti:

| Shranjena omejitev | Povezan z |
| --- | --- |
| Profil omejitve (npr. `fov_30`) | Ta profil. |
| Ni izbran | `sunshine_cosine`, privzeta povezava, zabeležena kot predpostavljena (`cap_id_source=auto_default`), zato obdelava izda opozorilo. Senzorji se dobavljajo brez pokrova: izberite pokrov, ki je nameščen. |
| `none` — izbran kot **Nič (senzor brez pokrova)** v grafičnem vmesniku | Brez popravka za pokrov. |

DAQ-E-S ne upošteva profila pokrovčka, ne glede na to, kaj je bilo shranjeno: njegov difuzor je del njegove kalibracije (glej [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor)). Projekt, shranjen s prejšnjo različico programa Chloros, je kot edino možnost zabeležil gol senzor, zato se ponovno poveže z `sunshine_cosine`; v grafičnem vmesniku ponovno izberite **Nič (gol senzor)** in shranite projekt.

Senzor, ki ga ima drug odjemalec že odprt v bazenu (CLI, drug skript), se priključi tak, kot je, brez ponovne uporabe nastavitev, `disconnect_all()` pa ga pusti delovati; senzor, ki ga je odprl `connect_all()`, se sprosti. Tako kot pri `connect_daq_sensor()`, grafični vmesnik Chloros **ni** del sklopa: zaprite senzor v grafičnem vmesniku, preden projekt povežete iz skripta.**Zahteve in poročilo.** Senzorji svetlobe potrebujejo delujoč Chlorosov backend, prijavljen na plačljivi stopnji Chloros+ (Copper ali višji), `connect_all()` pa ga nikoli ne zažene. Če noben ne deluje, so vsi shranjeni senzorji navedeni pod `report["sensors"]["errors"]`, kjer je to tudi navedeno. Tudi senzor, ki ga backend ne more odpreti, je tam naveden z razlogom backenda; `report["sensors"]["missing"]` ostane prazen za svetlobne senzorje.

---

## Neposredna strojna oprema (brez backenda)

Če želite, da kamere LATTICE (CI, roboti brez zaslona, vgrajene) sploh ne bi bile odvisne od backenda, neposredno uvozite `lattice_sdk` — ta se ponovno izvozi prek `chloros_sdk`. Opozorilo glede `CAMERA_AVAILABLE`: `lattice_sdk` je v paketu SDK, vendar potrebuje Arena SDK runtime in OpenCV (`cv2`), ki pa ju paket ne namesti. Senzorji svetlobe potekajo prek backenda — [`connect_daq_sensor()`](#daq-sensor-sessions) ali `proj.sensors` projekta — z eno izjemo, pomočnikom za velikost pasov DAQ-M [`DLS`](#daq-m-band-magnitudes--dlsband_magnitudes), ki prav tako potrebuje `bleak`.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Prednastavitve in sprožilec

Tri od štirih prednastavitev delujejo v načinu **free-run**: kamera neprekinjeno osvetljuje,
`capture()` pa vrne naslednji okvir. `triggered` je izjema — ta nastavi
kamero na čakanje na strojni rob na liniji 2, zato ne zajame ničesar, dokler ta ne prispe.

| Prednastavitev | Sprožilec | Uporabite, ko |
| --- | --- | --- |
| `default` | prosto delovanje | splošna raba |
| `high_speed` | prosto delovanje | 8-bitno, omejitev 60 fps, kratka osvetlitev |
| `high_quality` | prosto delovanje | 12-bitno, brez omejitve fps — običajna izbira za fotografije |
| `triggered` | **pripravljeno, vodilo 2** | kamera je priključena na sinhronizacijski kabel M8 in jo sproži nekaj drugega |

Če izberete `triggered` (ali sami nastavite `trigger_mode="On"`), pri čemer nič
ne aktivira linije 2, bo vsak `capture()` dosegel časovno omejitev — kar je pravilno, saj ste
kamero prosili, naj počaka. SDK to pojasni, ko se to zgodi; glejte
[SC_ERR_TIMEOUT med zajemanjem](#direct-hardware-backend-free).

> **Opomba — sporočila »GVSP probe« / `SC_ERR_TIMEOUT -1011` ob vzpostavitvi povezave niso napake.**
> Ob vzpostavitvi povezave SDK poskuša dogovoriti**jumbo okvirje** (9000-bajtni GVSP-paketi) za večjo prepustnost. Na neposredni povezavi med omrežnimi karticami (npr. lokalni naslov `169.254.x.x`) omrežje običajno ne more prenašati jumbo okvirjev, zato ta preizkus poteče in v dnevnik zabeleži vrstice, kot so:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> To je **predvideni nadomestni postopek**: SDK samodejno preklopi na standardne pakete velikosti 1500 bajtov, kamera pa se še naprej normalno povezuje (vrstice `[chunk-enable …]`, ki sledijo, so del običajnega postopka povezovanja). Snemanje še vedno deluje.
>
> To preizkušanje lahko preskočite, vendar **ne gre le za utišanje dnevnika – izklopi jumbo okvirje.** Kamera odgovarja na pingove »Don't-Fragment« le do 1500 bajtov, ne glede na to, kako dobro je vaše omrežje, zato sam ping test nikoli ne more odkriti jumbo okvirjev; to je edina preizkusna funkcija, ki to zmore. Če jo onemogočite, bo kamera za vedno uporabljala standardne 1500-bajtne pakete, v katerem koli omrežju:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> To se splača le v omrežju, za katero *veste*, da ne podpira jumbo okvirjev, kjer to prihrani približno sekundo časa povezovanja na kamero. Ker gre za resnično kompromisno rešitev in ne le za kozmetično spremembo, vas SDK zdaj na to opozori, ko jo uporabljate:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Pustite nastavitev takšno, kot je, razen če imate za to razlog.**Če ostane omogočeno, se vsaka kamera izmeri enkrat na sejo strežnika in ponovno, ko se spremeni njena povezava (drugi adapter, druga hitrost povezave ali prekinjena povezava, kot se zgodi pri zamenjavi stikala); ponovne povezave ponovno uporabijo ta rezultat.**Po spremembi za stikalom, ki je ostalo vklopljeno, ponovno zaženite strežnik, da se izmeri ponovno.** Prva povezava lahko pri tem preverjanju včasih traja nekaj minut, kar je v dnevniku tudi navedeno.
>
> Če *želite* prepustnost jumbo paketov, omogočite jumbo od konca do konca (MTU mrežne kartice 9000 + stikalo, ki jih prepušča) ali jo fiksirajte z `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`, če veste, da povezava to podpira — čeprav je bolje uporabiti `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` za posamezne ukaze kot pa trajno nastavitev, saj fiksirana velikost preskoči preizkus in preneha s prilagajanjem omrežju pred njo. **Vsaka** naprava na poti mora prepuščati jumbo pakete — vključno z vsakim PoE razdelilnikom ali injektorjem, kar je običajen razlog, da sicer jumbo-združljiva konfiguracija ne more prenašati teh paketov.

> **`SC_ERR_TIMEOUT -1011` med `capture()` / `grab*()` je drugačen problem — ta je prava napaka.**
> Zgornja opomba se nanaša le na `-1011`, ki ga je zabeležila**sonda za čas povezave**. Enaka napaka, ki se pojavi pri**zajemanju**, pomeni, da je kamera uspešno vzpostavila povezavo, vendar ne pošilja nobenih slik:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Razpoznavni znak je kamera, katere *nadzorni* kanal deluje pravilno — odkrivanje deluje, nastavitve in zapisi `[chunk-enable …]` so uspešni — medtem ko *vsak* okvir preseže časovno omejitev.
>
> **Običajni vzrok je, da je kamera nastavljena na sprožilec strojne opreme.** Pri napakah `trigger_mode="On"` in `trigger_source="Line2"` kamera ne pošilja ničesar, dokler na sinhronizacijskem kablu M8 ne pride do električnega prehoda. Če nimate kabla, ki bi vodil to linijo, vsak posnetek čaka v nedogled. Kamera ni pokvarjena in omrežje deluje pravilno — deluje točno tako, kot ji je bilo naročeno.
>
> `CameraSettings()` ter `default` / `high_speed` / `high_quality` nastavijo prosti tek, zajem, ki se izteče med aktiviranjem, pa se sam pojasni, namesto da bi izpisal le `-1011`. `PRESETS["triggered"]` aktivira Line2, kot je predvideno.
>
> Če želite katero koli kamero prisiliti v prosti tek:
>
> ```python
> import copy
> settings = copy.deepcopy(PRESETS["high_quality"])   # PRESETS entries are shared
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Če se s kodo `trigger_mode="Off"` še vedno pojavi časovna omejitev, kamera dejansko ne pošilja podatkov – pošljite nam dnevnik in kodo `ip link show`.

#### Barvni profili (predogled v živo v RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` izbere barvni profil zaslona za **predogled v živo** na RGB kamerah (multispektralne kamere to nastavitev prezrejo):

| Profil | Pomen |
| --- | --- |
| `raw` | Popolnoma preskoči radiometrično verigo. |
| `linear` | DSNU + flat + WB, brez CCM, brez gama. |
| `natural` | Linearni + izmerjeni CCM + gama sRGB, samo s preprostim zaključkom (izglajevanje barvne globine + desaturacija svetlih delov) — realistična privzeta nastavitev. |
| `enhanced` | `natural` plus popolna obdelava »hub-parity« (odstranjevanje barvnih robov, živahnost, lokalni kontrast CLAHE). Bogatejši videz ob približno **dvojnih stroških obdelave na posamezni okvir**, kar pomeni nižjo hitrost snemanja v živo. |
| `custom_temp` | `natural`, vendar je WB fiksiran na `custom_cct_k` Kelvin (DLS se ne upošteva; omejeno na 2000–10000 K na strani strežnika). |

Profil je **samo za predogled v živo**, ki vpliva na hitrost in videz: shranjeni posnetki vedno dobijo polno bogato obdelavo ne glede na izbrani profil, zato izbira `natural` za pridobitev časa na posameznem okvirju ne zniža kakovosti tistega, kar se shrani na disk. Neznani profil sproži `ValueError`; ko je backend chloros dosegljiv, se sprememba posreduje tudi nanj, tako da jo naslednji predogledni okvir upošteva (uporabniki neposrednega SDK-ja brez backenda še vedno dobijo spremembo nastavitev).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Poudarki (zaslon RGB) — `set_highlights`

`LatticeCamera.set_highlights(value)` izloči najsvetlejše, najbeljše piksle **zaslona** RGB-kamerah — popravek za svetlo belo površino, ki se prikaže kot ravna bela lisa. `value` je `-100`..`0` (omejeno); `0` je izklopljen, prikaz pa je takrat točno takšen, kot bi bil brez te nastavitve. Močnejše nastavitve znižajo belino zaslona (na približno 60 % pri `-100`) in razširijo tonsko krivuljo, tako da površina, ki jo je samodejna svetlost predogleda potisnila v čisto belo, ponovno pridobi svoje podrobnosti.

Gre za nastavitev zaslona, vendar za razliko od nasičenosti in kontrasta (ki spreminjata le predogled v živo) spreminja tudi tisto, kar shranite: predogled v živo in posnetki na **ravni zaslona** se temu prilagodijo, zato so tudi ti datoteki temnejši (vsebujejo `Chloros:DisplayHighlights`); izvozi na podlagi sevanja in odbojnosti se nikoli ne spremenijo, prav tako pa ta nastavitev ne more obnoviti podatkov senzorja, ki so bili odrezani – za to zmanjšajte osvetlitev. Multispektralne in monokromatske kamere to nastavitev ignorirajo. Vrednost se shrani v `settings.highlights` in se pošlje prek POST-a na dosegljiv chloros backend, prav tako kot `set_color_profile`; ustreznik v CLI-ju je `lattice highlights --set N`.

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_highlights(-60)                      # 0 = off, -100 = strongest
```

#### Mono (M3M) kamere in `Calibration`

Mono **M3M** kamera (`M3M-<lens>-F<wavelength>`) je enopasovna: ena ravnina sivo-tonov, brez Bayerjevega mozaika, brez 3×3 spektralne matrike prekrivanja. `Calibration` jo prepozna in izpostavi zastavico `is_mono`. Odbojnost se še vedno uporablja kot radiometrični zemljevid po posameznih pasovih (razmešavanje je identitetna matrika), vendar večpasovna matematika na eni sami kameri prinaša smiselne rezultate namesto nesmiselnih:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Za izračun indeksa vegetacije z monokromatsko opremo združite več kamer M3M z različnimi valovnimi dolžinami v poravnani večpasovni niz (glejte [Poravnava niza](#array-alignment)) in izračunajte indeks za celoten niz namesto za eno kamero.

#### Magnitudi pasov DAQ-M — `DLS.band_magnitudes`

`DLS(mac_address, filter_model="RGN-IMX265")` se prek Bluetootha poveže z DAQ-M z lastnim BLE MAC-om. To je edini razred svetlobnih senzorjev, ki komunicira neposredno s senzorjem in ne prek strežniškega dela, zato potrebuje tisto, kar potrebujejo razredi za neposredno komunikacijo s kamero (`CAMERA_AVAILABLE`: izvedbeno okolje Arena SDK in OpenCV), ter `bleak` za Bluetooth (`pip install "chloros-sdk[camera]"` ali `pip install bleak`; brez njega `DLS` sproži `DLSError`). Če DAQ-M, povezan drugje (v bazenu backenda, v projektu, v GUI) preneha oddajati oglase, zato ga tam najprej sprostite. **`mac_address` je obvezen**: privzeta vrednost ne obstaja, saj ima vsaka enota svojo, `DLS()` pa brez te vrednosti sproži `DLSError`, preden se poskusi vzpostaviti kakršno koli povezavo. Poiščite MAC z `discover_daq_sensors(transports=["ble"])` (glej [Odkrivanje](#discovery--finding-an-address-to-connect-with)) ali `chloros-cli daq pool-discover --only ble`.

`DLS.band_magnitudes(spectrum=None, *, spectral_response=None)` vključi spektrum DAQ-M (zadnjo vrednost, ko je `spectrum` enako `None`; `DLSError`, če ga še ni) prek odzivov treh kanalov kamere in vrne `{"red": …, "green": …, "nir": …}` — ključi poimenujejo kanale v tem vrstnem redu; pri krivuljah RGB-kamere je tretji (`nir`) njen modri kanal.

- Brez `spectral_response` uporabi krivuljo flote filtrskega modela (`filter_model`, s katero je bil izdelan `DLS`), ki je bila izmerjena na eni enoti.
- `spectral_response` namesto tega uporablja **lastne** izmerjene krivulje kamere: slovar `{"wavelengths": [...], "R": [...], "G": [...], "B": [...]}`, kot je shranjen v kalibracijskem paketu te kamere (`color_correction.spectral_response_smoothed` — RGB-kamere; multispektralni paketi takšne krivulje ne vsebujejo). Če slovarja ne more uporabiti, se vrne na krivuljo flote.

```python
from chloros_sdk import DLS

# camera_curves: {"wavelengths": [...], "R": [...], "G": [...], "B": [...]}
with DLS(mac_address="AA:BB:CC:DD:EE:FF") as dls:
    reading = dls.acquire()
    fleet = dls.band_magnitudes(reading)                         # fleet curve
    own = dls.band_magnitudes(reading,
                              spectral_response=camera_curves)   # the camera's own
```

CLI-ova funkcija `chloros-cli lattice dls --camera SERIAL` stori enako prek ozadja, poišče krivuljo kamere v njenem naloženem ali shranjenem v predpomnilniku kalibracijskem paketu ter navede, čigavo krivuljo je uporabila. Prek HTTP-ja je to `POST /api/dls/cli-read` z `mac` (obvezno: brez tega je odgovor HTTP-napaka 400, `code` `MAC_REQUIRED`) ter `camera_serial` v telesu sporočila; odgovor `band_magnitudes_source` je `camera <serial>`, če je bila uporabljena lastna krivulja te kamere, ali `model <filter>` za krivuljo flote (kamera brez krivulje: ni RGB ali ni naložen paket ali shranjen v predpomnilniku).

---

## Izjeme

Uporabite osnovni razred za obravnavo »vsega, kar je šlo narobe v Chloros«:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` in `ChlorosConfigurationError` sta atributa `chloros_sdk` (in `chloros_sdk.exceptions`), vendar ne v `__all__`, zato jih `from chloros_sdk import *` ne vključi: uvozite jih po imenu. Družina `ChlorosError` izhaja iz `ChlorosLocal` in funkcij, zgrajenih na njem (`process_folder()`, `process_lattice_capture()`); površina »smart-connect« izhaja iz `ChlorosConnectError`.

Hierarhija:

```

ChlorosError
├── ChlorosBackendError           (backend binary not found / failed to start)
├── ChlorosConnectionError        (HTTP transport failure; no backend running
│                                  with auto_start_backend=False)
├── ChlorosLicenseError           (403 PLAN_UPGRADE_REQUIRED: plan below Copper)
├── ChlorosAuthenticationError    (401 AUTH_REQUIRED: not logged in)
├── ChlorosConfigurationError     (bad configure() inputs)
└── ChlorosProcessingError        (pipeline failed; create/import refused)
    └── ChlorosProjectFilesMissingError  (open_project(): none of the
                                  project's source files found, even after
                                  the relink; .missing_roots, .audit)

ChlorosConnectError                (the smart-connect surface: connect_*, attach_array,
                                    list_*, discover_*, every CameraSession /
                                    ArraySession / DAQSensorSession method,
                                    RecorderHandle, ContinuousHandle,
                                    disconnect_all(), and analyze_array_network()
                                    when no backend answers; 401 and 403 included.
                                    Derives from plain Exception, NOT from
                                    ChlorosError, so `except ChlorosError` will
                                    not catch it. .warning: a warning the
                                    backend sent beside its error (also at the
                                    end of the message); .recording_closed: the
                                    recording a failed DAQ-E restart closed;
                                    both None otherwise)

FrameFetchError (RuntimeError)     (grab() / frame_stream() refusals; .status,
                                    .body, .errors)
ValueError                         (capture() arguments: PNG/JPEG with a level they
                                    cannot hold; a bad packed=)
FileNotFoundError / ImportError    (open_project(): no such folder / no project
                                    loaders)
RuntimeError                       (CameraHandle / ArrayHandle: a backend refusal)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Primeri od začetka do konca

### 1. Obdelava mape s prilagojenim kazalnikom napredka

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Matrika LATTICE v realnem času → Odbojnost + referenca DAQ

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Projektno usmerjena kampanja zajemanja

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Tok sličic iz več kamer → NumPy-cevovod

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Skript za zajem brez grafičnega vmesnika neposredno na strojni opremi (brez backenda)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Preverjanje zmogljivosti pred priključitvijo niza 4 kamer

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["width"], height=r["height"],     # before binning
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Ekvivalent recepta za zajem (čisti Python)

`run_project_recipe()` izvede recept za CLI tak, kot je (glej [Projektna mapa in recepti](#project-folder--recipes)). Če ga namesto tega napišemo v Pythonu, so isti koraki:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply: connect_all() applied each camera's saved settings. CameraHandle
    # has no settings method; a CameraSession on the same pooled camera does
    # (it is borrowed, so it never releases the camera)
    chloros_sdk.connect_camera(cam.serial).set_settings(
        exposure_time=5000, gain=6.0)

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## MCP-strežnik (AI-pomočniki)

`chloros-mcp` omogoča AI pomočniku, ki uporablja [Model Context Protocol](https://modelcontextprotocol.io) (MCP), kot so Claude Desktop, Claude Code ali Cursor, da upravlja s sistemom Chloros: obdeluje slike (ustvari projekt, uvozi mapo posnetkov, izbira nastavitev, obdelava, poročanje o izvoženih podatkih) ter upravljanje kamer LATTICE, matrik in svetlobnih senzorjev DAQ (povezovanje, predogled, nastavitev osvetlitve in barve, zajem, snemanje, posodobitev programske opreme). Njegova orodja pokrivajo funkcionalnosti tega SDK-ja. Vi vprašate v preprostem jeziku, pomočnik pa pokliče orodja strežnika:

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

The assistant calls:
  create_project          {"project_name": "Flight_2026-10-17"}
  import_images           {"folder_path": "D:\\Flights\\2026-10-17"}
  configure_processing    {"indices": ["NDVI"]}
  start_processing        {}
  get_processing_status   {}      (every 30-60 s until its state is "finished")

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.

The assistant calls:
  list_devices            {}
  get_camera_preview      {"array_id": "array-1779862544497"}
  start_capture_run       {"array_id": "array-1779862544497", "mode": "interval",
                           "interval_s": 10, "duration_s": 300,
                           "processing": "reflectance"}
  get_job_status          {"job_id": "..."}   (until its state is "finished")
```

Strežnik je zgrajen na tem SDK-ju: orodja za obdelavo pokličejo [`ChlorosLocal`](#image-processing--chloroslocal), strojna oprema pa [kliče smart-connect](#smart-connect-for-lattice-cameras), [DAQ-seje](#daq-sensor-sessions), `ChlorosProject` in [pripomočke za kamere](#camera-utilities), ki vsi komunicirajo z istim backendom, ki ga uporablja aplikacija Chloros. Njihove zahteve vsebujejo `X-Chloros-Client: sdk`, zato strežniški del izvede preverjanje prijave in naročnine iz SDK-ja, pri čemer veljajo tudi pravila SDK-ja (na primer, funkcija »Texture Aware« je za posnetke LATTICE zavrnjena).

### Namestitev

Strežnik je vključen v paket `chloros-sdk`; ničesar drugega ni treba namestiti. SDK namesti ukaz `chloros-mcp`, ki je enak ukazu `python -m chloros_sdk.mcp_server`:

```bash
pip install chloros-sdk        # not needed where the Chloros installer set up the SDK (Step 2)
chloros-mcp --help
```

- **Windows:** namestitveni program Chloros prek pip namesti SDK v vaš Python, kar `chloros-mcp.exe` shrani v mapo `Scripts` tega Pythona.
- **Linux (.deb):** paket namesti SDK za sistemski `python3` in umesti ukaz `chloros-mcp` v `/usr/bin`, tako da je na voljo v vsakem uporabniškem `PATH`. V virtualnem okolju virtualenv `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` doda `chloros-mcp` v `bin` tega virtualnega okolja.

Strežnik uporablja le standardno knjižnico poleg SDK-ja (Python 3.7+). MCP-odjemalec ga zažene in komunicira z njim prek stdin/stdout; sami ga ne zaženete, razen z `--help`.

### Nastavitev odjemalca

**Claude Desktop:**dodajte strežnik v `claude_desktop_config.json` (v sistemu Windows v `%APPDATA%\Claude\claude_desktop_config.json`; odprete ga prek**Nastavitve > Razvijalec > Uredi konfiguracijo**), nato ponovno zaženite Claude Desktop:

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

**Claude Code:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Kazalec:** isti vnos `mcpServers` kot v programu Claude Desktop, v `~/.cursor/mcp.json` (za vsak projekt; `%USERPROFILE%\.cursor\mcp.json` v sistemu Windows) ali `.cursor/mcp.json` v enem projektu.

Možnosti se vpišejo v `args` ali za ukazom za Claude Code (`claude mcp add chloros -- chloros-mcp --no-auto-start`). Če želite namesto `chloros-mcp` na `PATH` uporabiti določen Pythonov SDK, zaženite modul s tem Pythonom (njegova polna pot kot `command`, če ni prvi `python3` na `PATH`):

```json
{
  "mcpServers": {
    "chloros": {"command": "python3", "args": ["-m", "chloros_sdk.mcp_server"]}
  }
}
```

Odjemalec zažene strežnik brez vašega lupinskega programa, zato če poroča, da se strežnik ni uspel zagnati, `chloros-mcp` verjetno ni na `PATH`, ki ga vidi odjemalec. Namesto tega navedite polno pot (`where chloros-mcp` v sistemu Windows, `which chloros-mcp` v sistemu Linux, kjer je .deb datoteka `/usr/bin/chloros-mcp`) ali pa modul zaženite s Pythonom, ki vsebuje SDK.

### Možnosti

| Možnost | Privzeto | Kaj počne |
| --- | --- | --- |
| `--backend-url URL` | `http://127.0.0.1:5000` | Backend Chloros, ki ga je treba uporabiti. |
| `--no-auto-start` | izklopljeno | Nikoli ne zaženi backenda: uporabi že tistega, ki teče, na primer aplikacijo Chloros, storitev backenda za Linux ali tunel do mape na drugem računalniku (glej [Način oddaljenega backenda](#remote-backend-mode-pip-only-host-via-tunnel); poti do map morajo v tem primeru voditi do iste mape na obeh računalnikih). |
| `--backend-exe PATH` | kjer ga namesti Chloros | Backend, ki se zažene, če noben ne teče, za Chloros, nameščen na mestu, kjer SDK-ja ne išče. |
| `--log-level LEVEL` | `WARNING` | `DEBUG`, `INFO`, `WARNING` ali `ERROR`. Dnevniki se shranjujejo v stderr, kar večina odjemalcev prikaže v svojem dnevniku MCP; stdout prenaša le protokolna sporočila. |

### Orodja

Argumenti vsakega orodja so enaki argumentom klica SDK, ki ga to orodje ovija; odjemalec vsakega navaja s celotno shemo. Poti so absolutne.

**Obdelava**

| Orodje | Klic SDK | Kaj počne |
| --- | --- | --- |
| `create_project` | `ChlorosLocal.create_project` | Ustvari projekt in ga določi kot odprt projekt, na katerem delujejo orodja za obdelavo. Z navedbo samega imena se mapa ustvari v privzeti projektni mapi; z absolutno potjo se ustvari v tej mapi. |
| `open_project` | `ChlorosLocal.open_project` | Odpre shranjen projekt in ga določi kot odprt projekt. Poti premaknjenega projekta se preusmerijo na mapo, v kateri se projekt zdaj nahaja; `relink_folder` (absolutno) imenuje mapo, v kateri so zdaj ostale manjkajoče datoteke. Vrne `source_files` in `image_count`. Ni bilo najdeno: napaka, ki navaja, kje so bile, in poimenuje `relink_folder`; nič ni odprto. |
| `import_images` | `import_images` | Uvozi pare datotek Survey3 `.RAW`+`.JPG` ter »LATTICE« `.tif` zajeme in DAQ `.daq` posnetke; vrne `count` (slike) in `scan_count` (posnetke). |
| `configure_processing` | `configure` | Spremeni posredovane nastavitve in nobene druge (vrednosti: [Podprte vrednosti](#supported-values)); `custom_settings` pošlje celotno konfiguracijo, kot je podana. |
| `get_project_settings` | `get_config` | Prebere shranjene nastavitve obdelave, izvoza in indeksiranja odprtega projekta. |
| `start_processing` | `process(wait=False)` | Začne obdelavo odprtega projekta in se takoj vrne. |
| `get_processing_status` | `get_status` | Poroča o `state` (`idle`, `starting`, `processing` ali `finished`) ter napredek; ko je končano, se zapišejo `summary`, `export_check_failure`, če izvozov ni bilo mogoče preveriti, in `image_failure`, če so bile slike izgubljene: ostali izdelki se zapišejo; izvedbo se poroča kot neuspešno. |
| `process_folder` | `process_folder` / `process_lattice_capture` | Ustvari projekt za mapo, ga uvozi, konfigurira in zažene izvedbo v enem klicu (`lattice: true` za zajeme LATTICE). Medtem ko izvajanje poteka, se prekine, preden se karkoli ustvari, in tudi backend takrat zavrne spremembo projekta iz katerega koli orodja. |
| `export_light_sensor` | `export_light_sensor` | Zapiše kalibrirane datoteke `.daq` in `.csv` za posnetke svetlobnega senzorja projekta brez obdelave slik, pri čemer pridobi tovarniško kalibracijo iz MAPIR Cloud, če ta ni shranjena v predpomnilniku (glej [Zapisi svetlobnega senzorja](#light-sensor-recordings--calibrated-daq--csv)). |
| `read_image_audit_tags` | `read_image_audit_tags` | Prebere metapodatke revizije Chlorosa za obdelano sliko (vir, kalibracija, uporabljene stopnje). |

**Kamere in matrike**

| Orodje | Klic SDK | Kaj počne |
| --- | --- | --- |
| `list_devices` | `list_cameras`, `list_arrays`, `list_daq_sensors` | Kaj je trenutno povezano v katerem koli odjemalcu, vsak z `connected_here`; poleg tega tudi naloge, ki jih strežnik trenutno izvaja. |
| `discover_devices` | `discover_lattice_cameras`, `discover_daq_sensors` | Kamere in svetlobni senzorji, ki bi jih bilo mogoče priključiti. |
| `get_device_status` | `CameraSession.status`, `ArraySession.status` / `capability`, `DAQSensorSession.status` | Trenutno stanje ene kamere, niza kamer (z njihovimi zmogljivostmi in stanjem) ali svetlobnega senzorja. |
| `analyze_array_network` | `analyze_array_network` | Kaj lahko ta gostitelj in omrežje podpirata za predlagano matrico. |
| `connect_camera` / `connect_array` | `connect_camera` / `connect_array` | Povezovanje kamere ali sinhroniziranega niza (najprej glavna kamera). |
| `connect_project_devices` | `open_project(...).connect_all` | Priključite vsako napravo, ki jo shranjeni projekt pomni, skupaj z njenimi nastavitvami. |
| `disconnect_device` | `disconnect` / `force_disconnect` | Odklopite kamero, niz ali svetlobni senzor (glejte »Obnašanje«). `keep_settings: true` na nizu shrani nastavitve vsake kamere v njen lasten pomnilnik, `clear_settings: true` pa ta pomnilnik ponastavi (njegov rezultat nosi `camera_memory`). |
| `disconnect_project_devices`, `align_project_arrays`, `capture_project_devices` | `disconnect_all`, `align_arrays`, `capture_all` | Naprave projekta, skupaj. |
| `read_camera_settings` / `set_camera_settings` | `CameraSession.read_nodes` / `set_settings` | Branje ali pisanje nastavitev kamere po prijaznem imenu ali imenu vozlišča GenICimena vozlišča. |
| `set_array_frame_rate` | `ArraySession.set_target_fps` | Določitev hitrosti niza ali vrnitev na samodejno hitrost. |
| `capture_camera` / `capture_array` | `CameraSession.capture` / `ArraySession.capture` | En posnetek (`fastest` in `wait_for_exposure` sta `capture_fastest` in zajem z določeno osvetlitvijo). Če ni mape `output_dir`, se shranijo v mapo odprtega projektamapo `captures`, datoteke pa se dodajo projektu (zadržane do konca obdelave); to potrjuje odgovor `project_add`. |
| `start_capture_run` | `ArraySession.continuous`, `capture_repeated` | Neprekinjeno ali intervalno zajemanje kot naloga. |
| `start_array_recording` | `ArraySession.record` / `burst` | Video s kombiniranim indeksom ali surova serija posnetkov kot naloga. |
| `build_burst_video` | `ArraySession.build_video` | Ponovna obdelava serije posnetkov v kalibrirani video, kot naloga. |
| `get_job_status` / `stop_job` | | Sledenje ali zaustavitev naloge. |
| `get_camera_preview` | | Predogled v živo vsake kamere, vrnjen kot slika. |
| `grab_frames` | `ArraySession.grab`, `CameraHandle.grab` | En okvir (ali en sinhroniziran niz) na ravni obdelave, brez shranjevanja: statistika po ravninah in datoteka PNG za vsako. |
| `calibrate_array_alignment`, `get_array_alignment`, `clear_array_alignment`, `nudge_array_alignment`, `export_array_alignment`, `import_array_alignment` | `ArrayHandle.calibrate_alignment`, `alignment_status`, `clear_alignment`, `tweak_alignment`, `export_alignment`, `import_alignment` | Poravnava niza. |

**Svetlobni senzorji**

| Orodje | Klic SDK | Kaj počne |
| --- | --- | --- |
| `connect_light_sensor` | `connect_daq_sensor` | Priključite DAQ-U, DAQ-M ali DAQ-E (`cap_id` nastavi popravek omejitve). |
| `get_light_sensor_reading` | `DAQSensorSession.latest` | Najnovejše odčitke: časovni žig, nasičenost, spektralni razpon in vrh ter spektralno povprečje v valovnih dolžinskih razponih (`full_spectrum` za vsako točko). |
| `set_light_sensor_streaming` | `stream_start` / `stream_stop` | Zaženi ali ustavi tok podatkov. |
| `start_light_sensor_recording` / `stop_light_sensor_recording` | `record_start` / `record_stop` | Snemanje datoteke `.daq` kot naloge; `duration_s` ga samodejno ustavi. |
| `set_light_sensor_cap` | `DAQSensorSession.set_cap` | Popravek pokrova (`none` za goli senzor). |
| `get_light_sensor_bindings` / `set_light_sensor_bindings` | `daq_bindings` / `set_daq_bindings` | Kateri senzor prenaša odbojnost posamezne kamere in niza. |
| `get_light_sensor_band_presets` / `get_light_sensor_reflectance` | `daq_band_presets` / `light_sensor_reflectance` | Nabor pasov ter odbojnost in indeksi okolja in objekta za dva senzorja s pretočnim prenosom. |
| `get_light_sensor_device_info`, `set_light_sensor_network`, `set_light_sensor_auto_stream`, `enable_light_sensor_ptp`, `reboot_light_sensor`, `update_light_sensor_firmware` | `device_info`, `set_network`, `set_auto_stream`, `enable_ptp`, `reboot`, `update_firmware` | Lastne nastavitve naprave DAQ-E, shranjene v napravi. |

**Pripomočki za kamero** ([Pripomočki za kamero](#camera-utilities))

| Orodje | Klic SDK | Kaj počne |
| --- | --- | --- |
| `get_camera_color_settings` / `set_camera_color_settings` | `camera_color_profile`, `camera_color_adjust`, `camera_highlights` in njihovi nastavljivci | Barva zaslona RGB-kamere: profil in barvna temperatura, nasičenost / kontrast, svetli deli. |
| `run_camera_white_balance` / `set_camera_white_balance` | `camera_white_balance_once` / `set_camera_white_balance` | Enkratno uravnoteženje beline ali avtomatsko / izklopljeno / fiksna razmerja. |
| `list_camera_presets` / `apply_camera_preset` | `camera_presets` / `apply_camera_preset` | Prednastavitve kamere. |
| `list_filter_models` | `camera_filter_models`, `camera_filter_model_info` | Modeli filtrov ali po imenu. |
| `get_camera_firmware_status` / `update_camera_firmware` | `camera_firmware_status` / `update_camera_firmware` | Različice vgrajene programske opreme in bliskavica (privzeto delo). |
| `get_camera_power_info` / `set_camera_power` | `camera_power_probe`, `sleep_camera`, `wake_camera` | Stanje napajanja in temperature; mirovanje ali aktivno stanje. |
| `calibrate_camera_with_target` | `calibrate_camera_with_target` | Koeficienti iz kalibracijskega cilja v pogledu. |
| `get_network_adapters` / `fix_network_settings` | `network_adapters` / `fix_network_settings` | Mrežni vmesniki kamere, ocena in popravek (samo za skrbnika). |
| `apply_vignette_correction` | `apply_vignette_correction` | Popravite vinjeto v mapi s slikami. |
| `read_dls` | `read_dls` | En branje DLS. |
| `probe_system` / `get_system_info` | `probe_system` / `system_info` | Strojna oprema in okolje izvajanja gostitelja. |

**Prehodne kamere, profili poravnave in indeksi** ([Prehodni nizi, profili poravnave in indeksi](#transient-arrays-alignment-profiles--indices)). Vsako orodje tukaj, ki odpira kamere, uporablja `keep_settings` / `clear_settings` (privzeto false): glej [Pomnilnik kamere](#camera-memory--keep_settings--clear_settings).

| Orodje | Klic SDK | Kaj počne |
| --- | --- | --- |
| `start_camera_continuous` / `start_multi_camera_stream` | `start_camera_continuous` / `start_multi_camera_stream` | Snemanje ene kamere ali sinhronizirano predvajanje več kamer kot naloga (`duration_s` ga samodejno ustavi). |
| `capture_cameras_transient` | `capture_cameras_transient` | En sinhroniziran zajem brez priključitve niza. |
| `test_array_sync` / `detect_array_master` | `test_array_sync` / `detect_array_master` | Preverjanje sinhronizacije sprožilca in katere kamere so na sinhronizacijski liniji. |
| `analyze_array_models` | `analyze_array_models` | Kaj bi niz teh modelov dosegel v tem omrežju. |
| `calibrate_alignment_profile` | `calibrate_alignment_profile` | Rešite in shranite datoteko profila poravnave (`options` vsebuje podrobnejše možnosti reševalnika). |
| `capture_aligned_multiband` / `start_aligned_stream` | `capture_aligned_multiband` / `start_aligned_stream` | Poravnana večpasovna slika ali niz takih slik kot naloga. |
| `read_alignment_profile` / `reorder_alignment_profile` | `alignment_profile_info` / `reorder_alignment_profile` | Vsebina datoteke profila; vrstni red pasov in omogočene kamere. |
| `list_index_presets`, `compute_index`, `capture_live_index` | `index_catalog`, `compute_index`, `capture_live_index` | Prednastavitve indeksa in prehodi; indeks večpasovne datoteke TIFF; zajem v indeks v enem koraku. |

**Sinhronizacija časa, projektna mapa in recepti** ([PTP Grandmaster](#ptp-grandmaster--time_sync_status), [Projektna mapa in recepti](#project-folder--recipes))

| Orodje | Klic SDK | Kaj počne |
| --- | --- | --- |
| `get_time_sync_status` | `time_sync_status`, `time_sync_peers`, `time_sync_cameras` | Stanje PTP-grandmastera, njegovih vrstnikov in stanje PTP vsake kamere v nizu. |
| `restart_time_sync` / `set_time_sync_priority` | `restart_time_sync` / `set_time_sync_priority` | Ponovni zagon; nastavitev prioritet. |
| `get_project_folder` / `set_project_folder` / `reset_project_folder` | `project_folder` / `set_project_folder` / `reset_project_folder` | Mapa, v kateri se ustvarjajo novi projekti. |
| `run_project_recipe` | `run_project_recipe` | Izvedite recept shranjenega projekta kot nalogo z dnevnikom. Najprej se opravi preverjanje: recept z napako se zavrne in se ne izvede nič; o nepovezanih matrikah / senzorjih se izda opozorilo (ne poveže jih: najprej se izvede `connect_project_devices`). `dry_run` vrne rezultat preverjanja in ne izvede nič. |

Ni orodja za odpiranje shranjenega projekta za obdelavo: vsaka naloga obdelave se začne z `create_project` (ali `process_folder`).

### Zahteve

- **Chloros mora biti nameščen na istem računalniku.** Obdelavo opravlja Chlorosov backend. Strežnik lahko namesto tega uporabi backend, do katerega dostopa prek tunela (`--backend-url` z `--no-auto-start`, glej [Možnosti](#options)).
- **Prijavljen na tem računalniku**, v aplikaciji Chloros ali prek `chloros-cli login`. Strežnik se ne more prijaviti namesto vas: orodje, ki se zažene, ko niste prijavljeni, odgovori »Niste prijavljeni v Chloros« z razlogom backenda, o čemer vas obvesti pomočnik.
- **Katera koli plačljiva stopnja Chloros+ (Copper ali višja).** Strežnik ima dostop prek SDK, brezplačna stopnja Iron pa ga nima: orodje, ki se pokliče na brezplačni stopnji, odgovori »Ni na voljo v tem načrtu Chloros« z razlogom iz ozadja.
- Python 3.7 ali novejši z SDK ter MCP-odjemalec, ki zažene lokalne (stdio) strežnike.

### Delovanje

- **Backend.** Zagon strežnika ali prikaz seznama njegovih orodij ne sproži ničesar; prvi klic orodja vzpostavi povezavo. Če teče aplikacija Chloros (ali storitev backenda za Linux), strežnik uporabi njen backend in ga pusti teči. Če noben backend ne teče, strežnik zažene nameščenega, brez okna. Ko se strežnik zapre (zaprete odjemalca ali se strežnik ponovno zažene), ustavi zajemne naloge in snemanja svetlobnih senzorjev, ki jih je začel, ter sprosti naprave, s katerimi je bil povezan; backend še naprej teče, tako da se nedokončan proces obdelave nadaljuje do konca, sam pa se ustavi po 30 minutah, če ga nihče ne uporablja. Pri `--no-auto-start` strežnik nikoli ne zažene ozadja: orodje, ki ga pokličete, ko nobeno ne teče, vas prosi, da odprete aplikacijo Chloros, naslednji klic po tem pa vzpostavi povezavo.
- **En odprt projekt.** Backend ima odprt en projekt, ki ga deli z aplikacijo Chloros, ko je ta odprta, zato v aplikaciji ne delajte na drugem projektu, medtem ko deluje pomočnik.
- **Spremenijo se le nastavitve, ki so bile posredovane.** `configure_processing` pusti vsako nastavitev, ki ni bila posredovana, na njeni shranjeni vrednosti (SDK-jev `configure()` ponastavi debayer, korekcijo vinjete, kalibracijo odbojnosti, PPK in format izvoza na privzete vrednosti, če niso posredovane; strežnik jih dopolni iz projekta). Če ne more prebrati shranjenih nastavitev projekta, ne spremeni ničesar in to tudi sporoči, razen če je navedenih vseh pet. `indices` nadomesti indeksni seznam projekta. Rezultat navaja, kaj je bilo poslano kot `settings_sent`, pod `warnings` pa navaja vse indekse, ki jih pasovi kamere ne morejo ustvariti.
- **Obdelava poteka po načelu »začni in nato preverjaj«.** `start_processing` se vrne takoj, ko se obdelava začne, kar nato traja od nekaj minut do nekaj ur, pomočnik pa mu sledi z `get_processing_status`. `state` je `idle` (ni obdelave), `starting` (zadnji del sistema še ni poročal o izvedbi, do 20 s), `processing` ali `finished`, skupaj s [povzetek](#post-run-summary--hints): kaj je izvozil, ter namigi, če je izvozil manj, kot je bilo pričakovano (brez slik, ni bil najden kalibracijski cilj). Rezultati se shranijo v projektno mapo, tako kot v aplikaciji. `start_processing` se zavrne, medtem ko je izvajanje v teku, backend, ki preneha odgovarjati, pa se poroča kot napaka, ne kot zaključeno izvajanje.
- **Kaj vidi pomočnik.** Opozorila SDK-ja se prikažejo v rezultatih orodja pod `warnings`, napake (odjava, načrt, napačna pot) pa kot besedilo napake, ki ga lahko prebere in posreduje vam. Seznam, ki vsebuje več kot 20 elementov, se skrajša na prvih 20, celotna dolžina pa se prikaže kot `<key>_total`. Poti morajo biti absolutne.
- **Naprave so skupne.** Kamere, matrice in svetlobni senzorji se nahajajo v ozadju, zato lahko pomočnik upravlja z eno od naprav, povezanih z aplikacijo Chloros, spremembe pa se prikažejo v aplikaciji. `disconnect_device` prikaže, s čim je povezan strežnik; napravo, s katero je povezan drug odjemalec (aplikacija, `chloros-cli`, skript, drug `chloros-mcp`) se sprosti le z `force: true`, ki je `force_disconnect` iz SDK-ja, pri čemer se v primeru zavrnitve navedejo imena tistih, ki so jo povezali, če to ve backend; naprava, ki je v backendu sploh ni, se poroča kot nepovezana (`released: false`, `connected: false`, ni ničesar za sproščanje). Snemanje ali tok, ki ga izvaja backend, vodi svoj pogled `get_job_status` s `counts` — `frames_written`, `frames_dropped`, `frames_failed`, `drop_reason` (skupine sličic za tok z več kamerami) — ter enovrstični `summary` („zapisanih 220 okvirjev, 55 izpuščenih, 0 neuspešnih: …“), medtem ko teče, in po koncu njegove končne številke; seznam tega pri drugem odjemalcu (`other_clients_jobs`) vsebuje enako (`list_cameras()`, `list_arrays()` in `list_daq_sensors()` vsebujejo za vsako napravo`connected_by`: `client`, `label`, `name`, `at`). Vsaka naprava `chloros-mcp` se v ozadju imenuje kot `chloros-mcp (pid N)` (`chloros_sdk.connect.CLIENT_LABEL`, poslan kot `X-Chloros-Client-Label`; kateri koli SDK-skript lahko to nastavi). Za ustavitev snemanja svetlobnega senzorja drugega odjemalca je prav tako potreben `force`.
- **Dolgotrajno delo je naloga.** `start_capture_run`, `start_array_recording`, `build_burst_video`, `start_light_sensor_recording`, `start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`, `run_project_recipe`, in ob vsakem zahtevanem zajemanju, reševanju ali indeksiranju za `background` takoj vrne `job_id`; sledi mu `get_job_status`, `stop_job` pa ga zaključi . Izvajanje se nadaljuje, dokler ga ne ustavite, razen če mu dodelite `max_captures`, `duration_s` (ali, za serijo, `max_frames`); `min_free_gb` se ne zažene, če je na disku premalo prostega prostora. Omejitev se ne uporablja, razen če jo nastavite. Snemanje ali tok, ki ga izvaja backend (`start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`) ohranja identifikator naloge strežnika, zato ga drug `chloros-mcp` — tisti, ki se je zagnal kasneje — najde po tem identifikatorju: `get_job_status` ga sledi (in navaja takšne naloge, kot so `other_clients_jobs`, od `GET /api/camera/cli-jobs`, vsaka z `started_by`), `stop_job` pa ga ustavi šele z `force: true`. Vsaka druga naloga (zajem, snemanje v matrici ali serijski zajem, zajem v ozadju) teče v `chloros-mcp`, ki ga je zagnal, in le ta strežnik to ve: tako pravi drug strežnik, ki ga je zaprosil. Ustavljena naloga bere `stopping`, medtem ko zapisuje svojo čakalno vrsto, nato pa `finished` s končnimi številkami, 10 minut in iz katerega koli `chloros-mcp`: tisti, ki je proces začel, na enak način zazna prisilno zaustavitev drugega strežnika. `get_job_status` nikoli ne čaka na zaustavitev: med zaustavitvijo (svoje lastne `stop_job`, drugega odjemalcaali zaustavitev, ki sprosti kamere naloge, ki se je sama zaključila) takoj odgovori z `stopping` in trenutnimi števili, `stop_job` naloge, ki se že ustavlja, pa to tudi sporoči. Naloga, ki se je zaključila, v `stop_reason`: svoj razlog (`no_frames`, `disk_full`, `member_missing`, ali za neprekinjeno izvajanje niza `count`, `duration`, `array_gone`), `limit reached` pa le, če njegove vrednosti kažejo na `count` (ali `max_frames` za serijo, `duration_s` za posnetek) dosežen, `error`, `stopped`), `stopped_by`, `finished`, `captures`, `output_dir`, `get_camera_preview`, `grab_frames`, `grab_frames`, XPROT`duration_s`) doseženo, `error`, `stopped` (z `stopped_by`), če ga je ustavil odjemalec, sicer `finished`.
- **Kam gredo posnetki.** V mapo `captures` odprtega projekta, kamor jih aplikacija shrani, razen če je navedeno `output_dir`.
- **Slike.** `get_camera_preview` in `grab_frames` vrneta slike, ki jih lahko pomočnik pregleda, poleg njihovega JSON-a. Slike `grab_frames` so za ogled kontrastno raztegnjene; njihove statistike so dejanske vrednosti.
- **Ko se strežnik zapre**, se ustavijo naloge, ki jih je zagnal, in zapre se snemanje svetlobnega senzorja, ki ga je zagnal. Če je strežnik zagnal backend, pred ustavitvijo backenda sprosti tudi naprave, s katerimi je bil povezan; v backendu aplikacije ostanejo povezane.
- **Protokol.** Samo stdio (brez končne točke HTTP), samo orodja (brez virov ali pozivov), rezultati v obliki besedila in slik, različice protokola MCP od 2024-11-05 do 2025-11-25.**Kaj ni vključeno.** Prijava ali odjava (račun je vaš; strežnik se ne more prijaviti) ter zaustavitev backenda, ki ga strežnik ni zagnal. Za vse ostalo, kar ta SDK počne v zvezi z obdelavo in strojno opremo, obstaja orodje. (Nastavitev DAQ-E – njegovo ime, avtentifikacijski žeton, geslo OTA, zahteva po podpisu, ponastavitev na tovarniške nastavitve – ter neposredni DAQ-strežniki so izključno v pristojnosti `chloros-cli`, tudi v SDK-ju: napačno ime ali žeton blokira vsako namestitev Chlorosa v napravi.)

---

## Samodejni zagon ozadja

Vstopne točke za pametno povezovanje — `connect_camera`, `connect_array`, `connect_daq_sensor`, `discover_lattice_cameras` in `discover_daq_sensors` – so enostavni HTTP-odjemalci, ki predpostavljajo, da backend prisluhne na naslovu `127.0.0.1:5000` (privzeti URL vmesnika Smart-Connect). Če je grafični vmesnik (GUI) ali vmesnik CLI že zagnan, ta strežnik že deluje. Pri zagonu iz samega skripta ga morda ni — zato te funkcije **samodejno zaženejo priloženo binarno datoteko strežnika** (brez okna, na enak način kot to stori `ChlorosLocal`) pred prvim klicem, nato pa počakajo do 60 s, da se ta zažene. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` in `analyze_array_network()` ga nikoli ne zaženejo.

Pravila:

- **Ustvarja se izključno lokalni URL.** `backend_url`, ki kaže na `localhost` / `127.0.0.1` / `[::1]` je primeren; vsak drug gostitelj se obravnava kot računalnik nekoga drugega in se nikoli ne sproži.
- **Če se backend zaključi, preden odgovori, se čakanje konča.** Klic se takoj prekine z izhodno kodo, ukazom, ki ga je izvedel, in zadnjimi vrsticami, ki jih je izpisal backend, namesto da bi počakal 60 s. Izhodna koda 98 je izjema: drugi backend že zaseda vrata ali datoteko zaklepa (aplikacije Chloros, storitve backenda ali drugega skripta, ki se še zagonja), zato klic še naprej čaka, da ta odgovori. `ChlorosLocal` deluje enako.
- **Backend ostane zagnan za ponovno uporabo** (enako kot v CLI-ju) — ob zaključku skripta ne pride do implicitnega zaustavitve. Ponovno zagnanje skripta ponovno uporabi aktivni backend.
- **To možnost lahko izključite z `auto_start_backend=False`** pri katerem koli od teh klicov (npr. če ste usmerili na oddaljeni backend ali če sami upravljate življenjski cikel backenda).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Če priložene izvedljive datoteke ni mogoče najti ali zagnati, naslednji HTTP-klic sproži izvedljivo, **na platformo prilagojeno** napako `ChlorosConnectError` namesto navadne napake »zavrnjena povezava« — v sistemu Windows vas usmeri k namizni aplikaciji ali ukazu `chloros-cli`; v sistemu Linuxu (brez grafičnega vmesnika) vas usmeri k ukazu `chloros-cli` ali `.deb`.

---

## Okolje in glave

SDK označi vsak HTTP-klic v ozadju z `X-Chloros-Client: sdk`. Ozadje uporablja pravila licenciranja SDK/CLI (potrebna sta prijava **in** plačljiv načrt Chloros+), namesto poti brezplačnega nivoja v grafičnem vmesniku. To se nastavi samodejno ob uvozu – ničesar vam ni treba storiti.

Zgornji del se doda vsakemu klicu `requests` / `urllib`, katerega URL vsebuje `localhost` ali `127.0.0.1`, na katerem koli vratu – vključno z vašimi lastnimi lokalnimi storitvami — in nadomesti vrednost, ki ste jo nastavili sami. Klici na druge gostitelje ostanejo nespremenjeni.

Prepisujte URL zadnjega dela s posredovanjem `backend_url=` (`ChlorosLocal` sprejema `backend_url=` ali `api_url=`; `process_folder()` in `process_lattice_capture()` sprejemata le `api_url=`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Nameščen backend posluša le na zanki, zato ga `backend_url` v lokalnem omrežju ne more doseči; glej [Način oddaljenega backenda](#remote-backend-mode-pip-only-host-via-tunnel) za vzorec tunela.)

---

## Različice in združljivost

- Različica SDK-ja je označena kot `chloros_sdk.__version__`.
- SDK vezuje delovanje na različico priloženega backenda. Kombiniranje starejšega SDK-ja z novejšim backendom običajno deluje (naprej združljive končne točke), vendar lahko kombiniranje novejšega SDK-ja s starejšim backendom povzroči napake `404` na novih končnih točkah — posodobite namizno aplikacijo, da bo ustrezala.
- Vmesnik Smart Connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) in končna točka za analizo omrežja vračata stabilne sheme JSON; nova polja so dodatna.

---

## Nasveti za odpravljanje težav

- **`ChlorosAuthenticationError: Login required`** (iz `ChlorosLocal`; klici Smart-Connect sprožijo napako `ChlorosConnectError: Login required. …` zaradi istega vzroka) → Na tem računalniku enkrat zaženite `chloros-cli login EMAIL PASSWORD` ali se prijavite prek namizne aplikacije Chloros.
- **`ChlorosProcessingError: Failed to start processing: … The project folder is too deep for this computer …`** (ali `Failed to create project: …` iz `create_project()`) → Na tem računalniku so dolge poti v sistemu Windows onemogočene (privzeta nastavitev sistema Windows), zato nobena pot ne sme presegati 260 znakov, izdelki pa se nahajajo do približno 130 znakov pod projektno mapo. Nič ni bilo obdelano. Sporočilo navaja najdaljšo projektno mapo, ki ustreza: uporabite krajšo ali vklopite dolge poti v Windowsu in ponovno zaženite Chloros. Windowsov namestitveni program to ponuja kot »Omogoči dolge poti v Windowsu«, kar je privzeto označeno; sicer uporabite skupinsko politiko: *Konfiguracija računalnika > Upravne predloge > Sistem > Datotečni sistem > Omogoči dolge poti Win32*.
- **`ChlorosConnectError: No Chloros backend is running …`** → Funkcija »smart-connect« samodejno zažene lokalni backend, zato se to pojavi le, če priloženega izvedljivega datoteka ni mogoče najti/zažeti (npr. gostitelj, ki deluje izključno prek pipbrez namiznega paketa). Sporočilo je odvisno od platforme: v sistemu Windows odprite namizno aplikacijo ali zaženite kateri koli ukaz `chloros-cli`; v sistemu Linux zaženite ukaz `chloros-cli` (grafični vmesnik ne obstaja) ali namestite `.deb`. Za oddaljeni backend prenesite `backend_url=` (in `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** ob uvozu → `lattice_sdk` ni uspelo naložiti: izvedbenega okolja Arena SDK ali OpenCV (`cv2`), ni mogoče uvoziti v tem Pythonu. Površina, ki jo poganja backend (`connect_*`, projicira prek backenda, `ChlorosLocal`), še vedno deluje.
- **Array se poveže pri polnem senzorju, vendar z nizko hitrostjo snemanja**→ Tako je zasnovano: avtomatski način ohranja polni senzor in dovoli, da hitrost pade na tisto, kar prenese povezava. Uporabite `binning=2` / `4` ali manjše `width` / `height` za več sličic na sekundo, ali pa omogočite jumbo sličice / nadgradite povezavo. Združevanje slik (binning) in ROI**ne** pokrivata skupne prekomerne zasedenosti (`oversubscribed: true`, polja fps 0): preveč kamer za povezavo ni mogoče odpraviti z združevanjem/ROI — zmanjšajte število kamer, omogočite jumbo okvirje ali preklopite na hitrejšo mrežno kartico (glejte [Prekomerna naročnina](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` poroča, da je sprejemni obroč mrežne kartice zelo majhen (~0,26 MB) / povezovalna vrata z napako »FRAMES WILL DROP«** → Sprejemni obroč gostiteljske mrežne kartice je nastavljen na privzeto vrednost (pogosto se po posodobitvi gonilnika mrežne kartice ponastavi na 32). Na Realtek USB 10GbE adapterju nastavite `ReceiveBufferLen=256` in `PendingReceives=64` (z višjimi vrednostmi), nato pa ponovno zaženite backend, da ponovno prebere obroč. Celoten postopek: [Referenca CLI-ja → Nastavitev in optimizacija gostiteljske mrežne kartice](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Gostitelj se ob ponovnem zagonu/izklopu zatakne, kasneje se pojavijo napake WMI `Invalid class` / mrežna kartica se ne vklopi** → Zastarel gonilnik USB 10GbE povzroča napako `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Posodobite gonilnik adapterja na najnovejšo različico (≥ 2026) in ponovno uporabite nastavitve sprejemnega obroča. Glejte [Referenca CLI-ja → Nastavitev in optimizacija omrežne kartice gostitelja](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Zajemanje niza ni uspelo zaradi napake HTTP503 »Ni nove sinhronizirane skupine okvirjev«** → Niz ni objavil nobene nove skupine okvirjev v času čakanja. Preverite `arr.status()`: `stalled: True` (ali `streaming: False`) z `actual_fps`; vrednost 0 pomeni, da je zajemni krog prenehal objavljati — prekinite povezavo z nizom in se ponovno povežite. Če je `stalled` False in je `actual_fps` večji od 0, poskusite znova. Takoj po vrednosti `connect_array()` zajem že čaka do 20 s na prvo skupino.
- **`status()` prikazuje vrednost `link_speed_mbps: 4294`** → Sistem Windows ni mogel prebrati hitrosti adapterja; to je značilno za USB-Ethernet-adapterje in ni napaka. Glej [Pričakovane hitrosti okvirjev](#expected-frame-rates).
- **`update_firmware()` se ne uspe z `device never connected back on TCP <port>`** → Požarni zid je blokiral povezavo naprave DAQ-E nazaj s tem računalnikom. Namestitveni program za Windows doda pravilo (glejte [Nastavitve naprave DAQ-E](#daq-e-device-settings)); pri drugih požarnih zidih omogočite vhodni TCP promet v strežnik iz podomrežja senzorja.
- **Odbojnost zavrnjena** → Za odbojnost v absolutni lestvici mora biti aktivni DAQ vezan na kamero (ali niz kamer). Vežite ga prek grafičnega vmesnika ali uporabite `processing="radiance"` (W/m²/sr/nm), ki ne zahteva seznanjenega senzorja.
- **Zajem s programom `smart=True` traja dlje, kot je pričakovano** → Konvergenca AE je odvisna od dinamike prizora. Pri programu `ArrayHandle.capture_smart()` razširite program `exposure_tolerance_pct`, skrajšajte `stability_window_s` ali zmanjšajte `settle_timeout_s` za hitrejši, manj stabilen sprožilec.

---

## Glej tudi

- [Referenca CLI](cli-reference.md) — vsak podukaz CLI ustreza klicu SDK.
- [Senzorji svetlobe DAQ](https://mapir.gitbook.io/chloros/daq-light-sensors) v priročniku za Chloros — senzorji, njihove omejitve in kalibrirano območje, snemanje, omrežje DAQ-E.
- [Kamere LATTICE](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) v priročniku za Chloros — priključevanje, nastavitve kamere, načini zajemanja, matrike.
- Spletna dokumentacija: `https://mapir.gitbook.io/chloros/api-python-sdk`

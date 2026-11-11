# Snemanje in format .daq

Datoteka `.daq` je format snemanja svetlobnega senzorja sistemChloros

: **SQLite baza podatkov**, ki vsebuje kalibrirane spektralne posnetke iz enega senzorja DAQ. Če med snemanjem posnamete eno takšno datoteko, lahko proces obdelave odbojnosti kasneje vsako sliko deli z navzdolnim sevanjem, izmerjenim v tistem točno trenutku.

## Kaj vsebuje datoteka .daq

| Lastnost | Vrednost |
| --- | --- |
| Kontejner | Baza podatkov SQLite, ena datoteka na senzor na snemanje |
| Ime datoteke | Vključuje **ID senzorja**in**časovni žig**, npr. `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spektrum na posnetek | 135 točk, 340–1010 nm v korakih po 5 nm, plus tristimulna koordinata CIE XYZ |
| Enote | Kalibrirana spektralna sevanja, **W/m²/nm** (uporabljen tovarniški kalibracijski paket + profil pokrova) |
| Vtisnjeni metapodatki | ID senzorja (ključ za pridobitev tovarniške kalibracije te enote) in veljavni profil pokrova — glej [Profili pokrovov in kalibrirano območje](caps-and-range.md) |

Format je enak za DAQ-U, DAQ-M, DAQ-E in DAQ-E-S, zato pri nadaljnji obdelavi ni pomembno, kateri prenosni sistem je posnetek zabeležil. Posnetek DAQ-E-S poleg vsakega spektra shranjuje tudi naklon, kotaljenje in nagib enote.

**Zapisi ostanejo ohranjeni tudi ob izpadu napajanja (1.3.0).**Zapis v realnem času se zapiše z dnevnikom in popolno sinhronizacijo diska ob vsakem potrditvenem koraku, zato izpad napajanja ali prekinjen proces povzroči izgubo le zadnjih 3 sekund, ne pa celotne datoteke. Posnetek, ki se je ustavil sredi zapisovanja, lahko poleg datoteke `.daq` pusti tudi datoteko `<name>.daq-journal`:**obe datoteki shranite in kopirajte skupaj.** Program Chloros

ob prvem branju datoteke vrne dnevnik nazaj. Zaseden disk upočasni zapis datoteke, nikoli pa ne odčitkov senzorja.

**DAQ-E, ki preneha pošiljati podatke, se ponovno poveže (1.3.0).** Ko tok podatkov DAQ-E utihne (zaradi ponovnega zagona stikala ali prekinjene povezave), ga strežnik ponovno poveže z istimi nastavitvami, snemanje pa se nadaljuje v isti datoteki `.daq`, pri čemer je v datoteki označena vrzel.

Za kalibrirano snemanje je potreben tovarniški kalibracijski paket senzorja. Za naprave DAQ-U in DAQ-M strežnik pridobi paket iz MAPIR

jevega oblaka na podlagi ID-ja senzorja (če to ni mogoče, se snemanje zavrne); naprave DAQ-E so izvzete, saj imajo kalibracijo shranjeno na sami napravi.

## Snemanje prek grafičnega vmesnika

Snemanje v grafičnem uporabniškem vmesniku zahteva **odprt projekt** (v nasprotnem primeru so gumbi za snemanje onemogočeni):

* **Snemi vse / Ustavi vse** — na vrhu stranskega pasu Svetlobni senzorji; hkrati začne ali ustavi snemanje `.daq` na vseh priključenih senzorjih.
* **Snemanje / Ustavi snemanje** — za vsak senzor posebej, v modalnem oknu z nastavitvami v obliki zobnika. Med snemanjem se v vrsticah z informacijami v realnem času senzorja prikaže rdeča oznaka »REC«.

Datoteke se shranijo v `<project>/light_sensor/`, ko pa se snemanje ustavi — bodisi z gumbom »Ustavi«, »Ustavi vse« ali z odklopom senzorja za snemanje — se končana datoteka `.daq` **samodejno doda k odprtemu projektu**. Prikaže se na seznamu datotek projekta brez ročnega dodajanja, že pripravljen za obdelavo odbojnosti.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->



## Snemanje prek CLI



CLI

snema prek sklopa senzorjev strežnika (strežnik mora delovati – ti ukazi so preprosti HTTP

-odjemalci):

```bash
# Connect the sensor into the backend pool
chloros-cli daq pool-connect --eth-host daq-e-def330.local

# Record for 150 seconds, with a human-friendly device label
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
    -o ./out --device-name "rooftop-A"

# Or run open-ended and stop explicitly
chloros-cli daq pool-record --sensor-id daq-e-def330            # --duration defaults to 0 = run until --stop
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

Pridobi vrednost `--sensor-id` iz `chloros-cli daq pool-list`. Dve privzeti vrednosti, ki jih je vredno poznati:

| Možnost | Privzeto |
| --- | --- |
| `--duration` | `0` — snemaj do `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` v datotečnem sistemu **zadnjega dela**, ne v datotečnem sistemu CLI

|

Razlikovanje med izhodnimi mapami je pomembno, kadar CLI

cilja na backend na drugem računalniku: datoteka se shrani tam, kjer teče backend. Če je backend na istem računalniku, se relativna pot `-o` (kot na primer `./out` zgoraj) razreši glede na mapo, v kateri ste izvedli ukaz.

## Snemanje iz Python

a `DAQSensorSession` (ki ga vrne `chloros_sdk.connect_daq_sensor()`) razkrije isto združeno snemanje: `record_start(output_dir=None, device_name=None)` vrne pot do datoteke, `record_stop()` pa vrne `{path, rows}`. Za celoten API

za seje glejte [Referenco SDK

](../reference/sdk-reference.md). Razredi SDK

za neposredno upravljanje strojne opreme (samo za namestitve na namiznih računalnikih) privzeto zapisujejo posnetke v `~/Documents/DAQ/`; za izdane različice je podprta zgornja združena pot.

## Uporaba datoteke .daq med obdelavo

Za izračun odbojnosti iz posnetkov program Chloros

potrebuje sevanje navzdol, ki ustreza vsaki ekspoziciji:

* **Datoteko `.daq` shranite skupaj s posnetki.**Ob obdelavi procesna veriga samodejno določi**navzdolno sevanje, usklajeno s časovnim žigom**, iz posnetega datoteka `.daq` (katerikoli model DAQ) — ali iz datoteka `.csv` v formatu DAQ-M — ki se nahaja skupaj s slikami. Posnetki iz grafičnega vmesnika to zahtevo izpolnjujejo samodejno, saj se dodajo v projekt takoj, ko se ustavijo.
* **Kalibracija se pridobi na zahtevo.** Če paket tovarniške kalibracije za posamezno kamero ali DAQ še ni shranjen v lokalnem predpomnilniku, ga program Chloros

ob prvi uporabi samodejno prenese iz MAPIR

ovega oblaka (potreben je enkraten dostop do interneta; shranjeno v datoteki `~/.chloros/`).
* **Posnetki v realnem času ustvarijo lastno spremno datoteko.** Za vsak posnetek odbojnosti, zajet v živo, se dejansko uporabljena vrednost DAQ shrani kot datoteka `.daq` v mapi `daq/` posnetkov, tako da je mogoče posnetek kasneje ponovno obdelati brez izvirnega posnetka.
* **Posnetki iz snemalnikov `chloros_scripts` (1.3.0).**Datoteka `.daq`, ki jo zapišejo javni snemalniki [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts), je navedena kot model DAQ, ki jo je ustvaril, njeno omejitev pa je mogoče nastaviti v**Nastavitve projekta → Svetlobni senzor DAQ**, tako kot pri vseh drugih posnetkih. Ti snemalniki zapisujejo surove vrednosti senzorja; Chloros

pridobi tovarniško kalibracijo senzorja prek serijskega vmesnika in jo uporabi.
* **Čigava je ta vrednost (1.3.0).**Poleg vsakega zajema `.daq` je v `.daq.cameras.json` navedena kamera (kamere), katerih posnetke ta vrednost opisuje, obdelava pa jo uskladi**izključno s slikami iz teh kamer**. Samostojna kamera brez lastnega svetlobnega senzorja zato ne uporablja več odčitka druge kamere – ji dodelite senzor ali posnemite lastnega. Posnetek, narejen z DAQ-om samim (GUI Record, `daq pool-record`), in kateri koli `.daq` brez te datoteke (vključno s starejšimi zajemi), še vedno služi vsem kameram.

## Ponovno pridobivanje obsevanja

Obdelava projekta izvozi tudi vse posnetke svetlobnih senzorjev, ki jih vsebuje, v mapo
mapo `Light Sensor/` poleg slikovnih produktov. Za to **ni** potrebna slika:
samostojni posnetek svetlobnega senzorja je popoln posnetek, mapa, ki vsebuje le datoteke `.daq`,
pa je veljaven vhod. Projekt, ki vsebuje le zapise svetlobnih senzorjev, jih prikaže na seznamu in
obdeluje tako v aplikaciji kot v CLI

, po zaključku obdelave pa ostanejo na seznamu. Poročilo o obdelavi
navaja, koliko izdelkov svetlobnih senzorjev je bilo shranjenih.

| Izdelek | Kaj je |
| --- | --- |
| `<name>_calibrated.daq` | Arhiv, ki ga je mogoče ponovno obdelati, v isti shemi kot posnetek v realnem času, ki zdaj navaja kalibracijski paket, s katerim je bil ustvarjen. Ponovni uvoz ga **ne** kalibrira še enkrat. |
| `<name>_calibrated.csv` | Spektralna obsevanost v W/m²/nm na senzorjevi lastni valovni mreži, ena vrstica na odčitek, skupaj s fotometričnimi stolpci: skupna moč, fotopični in skotopični luks, PPFD z razdelitvijo na modro/zeleno/rdeče ter vrhunska valovna dolžina. |

DAQ-U ali DAQ-M, katerega kalibracijskega paketa ni mogoče pridobiti – niste povezani z internetom ali
ta senzor nima shranjene kalibracije –, se **preskoči z navedbo razloga** in se nikoli ne zapiše
kot »kalibrirana« datoteka, ki vsebuje surove številske vrednosti. Povežite se z internetom in ponovno zaženite postopek. DAQ-E
ima lastno kalibracijo, zato jo potrebuje le, kadar naprava ni povezana in
ni nič shranjeno v lokalnem predpomnilniku.

### DAQ-A: surove vrednosti in zakaj je to pravi odgovor

**DAQ-A** je starejši od sistema kalibracijskih paketov po serijskih številkah in nima paketa, ki bi ga bilo mogoče
pridobiti. To ni spregled: DAQ-A se kalibrira na terenu z
reflektančnim ciljem, kalibracija na podlagi cilja pa potrebuje le *relativni*
odziv senzorja — kar je točno to, kar so njegove surove vrednosti. Chloros

se danes kalibrira z njimi.

Zato se zapis DAQ-A izvozi, vendar pod drugim imenom:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, ne pa `_calibrated` – gre za drugačno ime datoteke in ne za oznako znotraj datoteke,
saj mora ta informacija ostati ohranjena tudi, če se datoteka pošilja po e-pošti kot golo ime. Zgornji del datoteke `.csv`
navaja `raw spectral sensor counts (NOT irradiance)` in opozarja, da so vrednosti
primerljive **znotraj** datoteke, ne pa med senzorji. Stolpci, ki imajo pomen
le za dejansko sevanje – skupna moč, luks, PPFD – ostanejo prazni in se ne
izračunajo iz števila impulzov.

Starejši zapisi DAQ-A-SD (shema v1.01 / v1.02) beležijo le čas zapisa datoteke, ne pa
časovnega žiga za vsako odčitano vrednost. Chloros

ne bo primerjal slik s temi podatki – povezovanje posameznega okvirja s
časom zapisa bi bilo napačno, čeprav na prvi pogled ne bi bilo videti tako –, vendar jih izvoz pravilno prebere,
CSV

-datoteka pa navaja, na kateri uri se nahaja.

Za celoten pregled delovanja odbojnosti – en senzor s kamero in dva senzorja za okolico/objekt – glej [Delovni tokovi odbojnosti](reflectance.md).

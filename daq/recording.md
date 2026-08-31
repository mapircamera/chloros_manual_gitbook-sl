# Snemanje in format .daq

Datoteka `.daq` je format snemanja svetlobnega senzorja Chloros: **SQLite baza podatkov** kalibriranih spektralnih okvirjev iz enega senzorja DAQ. Če med snemanjem posnamete eno takšno datoteko, lahko sistem za obdelavo odbojnosti kasneje vsako sliko deli z navzdolnim sevanjem, izmerjenim v tistem točno trenutku.

## Kaj vsebuje datoteka .daq

| Lastnost | Vrednost |
| --- | --- |
| Kontejner | Baza podatkov SQLite, ena datoteka na senzor na snemanje |
| Ime datoteke | Vključuje **ID senzorja**in**časovni žig**, npr. `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spektrum na posnetek | 135 točk, 340–1010 nm v korakih po 5 nm, plus tristimulacijska koordinata CIE XYZ |
| Enote | Kalibrirana spektralna sevanja, **W/m²/nm** (uporabljen tovarniški kalibracijski paket + profil pokrova) |
| Vtisnjeni metapodatki | ID senzorja (ključ za pridobitev tovarniške kalibracije te enote) in veljavni profil pokrova — glej [Profili pokrovov in kalibrirano območje](caps-and-range.md) |

Oblika je enaka za DAQ-U, DAQ-M in DAQ-E, zato pri nadaljnji obdelavi ni pomembno, kateri prenosni sistem je podatke zabeležil.

Za kalibrirano snemanje je potreben tovarniški kalibracijski paket senzorja. Pri modelih DAQ-U in DAQ-M strežnik pridobi paket iz oblaka MAPIR na podlagi ID senzorja (če to ni mogoče, se snemanje zavrne); enote DAQ-E so izvzete, saj imajo kalibracijo shranjeno na sami napravi.

## Snemanje prek grafičnega vmesnika

Za snemanje v grafičnem uporabniškem vmesniku je potreben **odprt projekt** (v nasprotnem primeru so gumbi za snemanje onemogočeni):

* **Snemi vse / Ustavi vse** — na vrhu stranskega menija »Svetlobni senzorji«; hkrati zažene ali ustavi snemanje `.daq` na vseh priključenih senzorjih.
* **Snemanje / Ustavi snemanje** — za vsak senzor posebej, v oknu nastavitev z ikono zobnika. Med snemanjem se v vrsticah z informacijami v realnem času senzorja prikaže rdeča oznaka »REC«.

Datoteke se shranijo v `<project>/light_sensor/`, ko pa se snemanje ustavi — bodisi z gumbom »Ustavi«, »Ustavi vse« ali z odklopom senzorja za snemanje — se končani `.daq` **samodejno doda k odprtemu projektu**. Pojavi se na seznamu datotek projekta brez ročnega dodajanja, že pripravljen za obdelavo odbojnosti.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->

## Snemanje iz CLI

CLI snema prek sklopa senzorjev strežnika (strežnik mora delovati – ti ukazi so lahki HTTP odjemalci):

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
| `--output` / `-o` | `~/Documents/DAQ Live View/` v datotečnem sistemu **zadnjega dela**, ne v datotečnem sistemu CLI |

Razlika v izhodni mapi je pomembna, kadar CLI cilja na backend na drugem računalniku: datoteka se shrani tam, kjer teče backend.

## Snemanje iz Python

`DAQSensorSession` (ki ga vrne `chloros_sdk.connect_daq_sensor()`) razkrije isto združeno snemanje: `record_start(output_dir=None, device_name=None)` vrne pot do datoteke, `record_stop()` pa vrne `{path, rows}`. Za celotno sejo API glej [Referenco za SDK](../reference/sdk-reference.md). Razredi SDK za neposredno delovanje s strojno opremo (samo za namestitve na namiznih računalnikih) privzeto zapisujejo posnetke v `~/Documents/DAQ/`; za izdane različice je podprta zgornja združena pot.

## Uporaba datoteke .daq med obdelavo

Za izračun odbojnosti iz posnetkov program Chloros potrebuje sevanje navzdol, ki ustreza vsaki osvetlitvi:

* **Datoteko `.daq` shranite skupaj s posnetki.**Ob obdelavi procesna veriga samodejno določi**navzdolno sevanje, usklajeno z časovnim žigom**, iz posnetega `.daq` (kateri koli model DAQ) — ali iz datoteke `.csv` v formatu DAQ-M — ki se nahaja skupaj s slikami. Posnetki iz grafičnega vmesnika to zahtevo izpolnjujejo samodejno, saj se dodajo v projekt takoj, ko se ustavijo.
* **Kalibracija se prenese na zahtevo.** Če paket tovarniške kalibracije za posamezno kamero ali DAQ še ni shranjen v lokalnem predpomnilniku, ga Chloros ob prvi uporabi samodejno prenese iz oblaka MAPIR (enkrat je potreben internet; shranjeno v predpomnilniku pod `~/.chloros/`).
* **Žive zajeme shranijo svoj stranski datotek.** Za vsak okvir odbojnosti, zajet v živo, se dejansko uporabljena vrednost DAQ shrani kot stranska datoteka `.daq` poleg slikovnega materiala, tako da se zajem lahko kasneje ponovno obdela brez izvirnega posnetka.

## Ponovno pridobivanje sevanja

Obdelava projekta izvozi tudi vse posnetke svetlobnih senzorjev, ki jih vsebuje, v
mapo `Light Sensor/` poleg slikovnih izdelkov. Za to **ni** potrebna slika:
svetlobni senzor, ki leti samostojno, predstavlja celoten posnetek, mapa, ki vsebuje le datoteke `.daq`,
pa je veljaven vhodni podatek. Izvedba poroča, koliko izdelkov svetlobnega senzorja je zapisala.

| Izdelek | Kaj je |
| --- | --- |
| `<name>_calibrated.daq` | Arhiv, ki ga je mogoče ponovno obdelati, v isti shemi kot posnetek v realnem času, ki zdaj navaja kalibracijski paket, s katerim je bil ustvarjen. Ponovni uvoz ga **ne** kalibrira še enkrat. |
| `<name>_calibrated.csv` | Spektralna obsevanost v W/m²/nm na senzorjevi lastni valovni mreži, ena vrstica na odčitek, skupaj s fotometričnimi stolpci: skupna moč, fotopični in skotopični luks, PPFD z razdelitvijo na modro/zeleno/rdeče ter vrhunska valovna dolžina. |

DAQ-U ali DAQ-M, katerega kalibracijskega paketa ni mogoče pridobiti – ste brez internetne povezave ali
ta senzor nima shranjene kalibracije –, se **preskoči z navedbo razloga** in se nikoli ne zapiše
kot »kalibrirana« datoteka, ki vsebuje surove števne vrednosti. Povežite se z internetom in ponovno zaženite postopek. DAQ-E
ima lastno kalibracijo, zato jo potrebuje le, kadar naprava ni povezana in
ni nič shranjeno v lokalnem predpomnilniku.

### DAQ-A: surove vrednosti in zakaj je to pravi odgovor

**DAQ-A** je starejši od sistema kalibracijskih paketov po serijskih številkah in nima paketa, ki bi ga bilo mogoče
pridobiti. To ni spregled: naprava DAQ-A se kalibrira na terenu z
reflektančnim ciljem, kalibracija na podlagi cilja pa potrebuje le *relativni*
odziv senzorja — kar je točno to, kar so njegove surove vrednosti. Chloros se danes kalibrira z njimi.

Zato se zapis DAQ-A izvozi, vendar pod drugim imenom:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, ne pa `_calibrated` – gre za drugačno ime datoteke in ne za oznako znotraj datoteke,
saj mora ta označba ostati ohranjena tudi, če se datoteka pošilja po e-pošti kot golo ime. Zgornji del datoteke `.csv`
navaja `raw spectral sensor counts (NOT irradiance)` in opozarja, da so vrednosti
primerljive **znotraj** datoteke, ne pa med senzorji. Stolpci, ki imajo pomen
le za dejansko sevanje – skupna moč, luks, PPFD – ostanejo prazni, namesto da bi se
izračunali iz števila impulzov.

Starejši zapisi DAQ-A-SD (shema v1.01 / v1.02) beležijo le čas zapisa datoteke, ne pa
časovnega žiga za vsako odčitano vrednost. Chloros ne bo uskladil slik s temi podatki – povezovanje posameznega okvirja s
časom zapisa bi bilo napačno, čeprav na prvi pogled ne bi bilo videti tako –, vendar jih izvoz pravilno prebere,
CSV pa navaja, na kateri uri temelji.

Za celotno zgodbo o reflektanci – en senzor s kamero in dva senzorja za okolico/objekt – glej [Delovni postopki za reflektanco](reflectance.md).

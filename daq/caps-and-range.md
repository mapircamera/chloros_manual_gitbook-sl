# Profili pokrovčkov in kalibrirano območje

> Sami pokrovi – kateri pokrov je priložen kateremu senzorju, kako se pritrdijo in kakšno je njihovo optično obnašanje – so dokumentirani v **[uporabniškem priročniku za DAQ](https://mapir.gitbook.io/daq)**. Ta stran obravnava *deklariranje* nameščenega pokrova v sistemu Chloros
, kar omogoča pravilno korekcijo.

Tovarniška radiometrična kalibracija vsakega svetlobnega senzorja DAQ opisuje *goli* senzor. Fizični pokrovček, nameščen nad difuzorjem, spreminja svetlobo, ki jo senzor zajema, zato Chloros
na kalibracijski paket uporabi **profil korekcije pokrovčka**, izmerjen v tovarni. Določitev pravilnega pokrova je del pridobivanja kalibriranih podatkov – ta stran obravnava, kateri pokrovi so na voljo za posamezne modele, kako jih določiti in kakšen je dejanski kalibrirani spektralni razpon senzorja.

## Razpoložljivost pokrovčkov po modelih

| Profil pokrovčka (`cap_id`) | Fizični pokrovček | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Pokrovček za korekcijo sončnega kosinusa (**privzeto na vsakem modelu**) | Da | Da | Da |
| `fov_15` / `fov_45` / `fov_90` | Koni za omejevanje vidnega polja (15° / 45° / 90°) | Da | — | Da |
| `fov_30` / `fov_60` | Koni za omejevanje vidnega polja (30° / 60°) | Da | — | — |
| `none` | Brez nameščenega pokrova | — | — | Da |

Opombe glede posameznih modelov:

* **DAQ-M ima en profil pokrova: `sunshine_cosine`.** »Bare-plus-Sunshine-cap« je njegova opredelitev izdelka, zato goli DAQ-M ne potrebuje geometrijskega profila.
* **Goli DAQ-U je resnično gol** — sploh ne potrebuje geometrijskega profila, zato zanj ne obstaja profil `none`.
* **`none` na DAQ-E NI brez učinka.** Vdolbeni, s steklom pokrit difuzor DAQ-E ima svojo lastno geometrijsko korekcijo, zato je »brez pokrova« pri tem modelu sam po sebi izmerjeni profil.
* **Goli DAQ-E ne more meriti neposrednega sonca pri nobeni višini** — pokrovček Sunshine je konfiguracija za uporabo na terenu. Ne načrtujte dela na prostem z golim DAQ-E.

V nastavitvah za posamezne senzorje v grafičnem uporabniškem vmesniku (ikonka zobnika v zavihku Svetlobni senzorji) spustni meni **Pokrovček** ponuja tudi možnost »Nič (gol senzor)« na modelih DAQ-U in DAQ-M — na teh dveh modelih »gol« preprosto pomeni, da se korekcija pokrovčka ne uporablja, kot je navedeno v zgornjih opombah. To možnost izberite le, če je pokrovček fizično odstranjen.

## Navajanje pokrovčka — in zakaj je to pomembno

**Navedeni `cap_id` mora ustrezati pokrovčku, ki je fizično nameščen na senzorju.** Niti senzor niti programska oprema ne moreta zaznati nameščenega pokrova. Deklaracija vpliva na dve stvari:

1. **Korekcijo v realnem času**, ki se uporablja za vsak spekter.
2. **Oznako pokrova, vpisano v vsak zapis `.daq`**, na katero se zanaša nadaljnja obdelava odbojnosti.

Pokrovček Sunshine **po zasnovi**zmanjša svetlobo za približno**12-krat**, zato snemanje z napačno deklariranim pokrovčkom povzroči napačno merjenje spektra za približno ta faktor. Spremembe pokrovčka takoj deklarirajte.

### Nastavitev pokrovčka

GUI: Zavihek Svetlobni senzorji → ikona zobnika v vrstici senzorja → spustni meni **Pokrovček**. Privzeta vrednost za vsak model je `sunshine_cosine`, kar je privzeta nastavitev (modeli DAQ-U, DAQ-M in DAQ-E se dobavljajo brez pokrovčka, pri snemanju pa se domneva, da je bil pokrovček uporabljen, dokler ga ne izberete), izbira pa ostane veljavna za celoten projekt.



CLI<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->
(backend mora teči):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

CLI
sintaktično sprejema celoten seznam `cap_id` (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); vsak profil se ob povezavi preveri glede na model senzorja, zato nedosegljiva identifikacijska številka pokrova (na primer identifikacijska številka, ki velja samo za E, na DAQ-U) povzroči jasno napako namesto napačne korekcije. Privzeta vrednost strežnika, če ni posredovano nič, je `sunshine_cosine`.

Python
SDK
: ob vzpostavitvi povezave poimenujte kapico z `connect_daq_sensor(cap_id="sunshine_cosine")` ali jo med izvajanjem zamenjajte z `DAQSensorSession.set_cap("fov_45")`; glejte [Referenco SDK
](../reference/sdk-reference.md#daq-sensor-sessions).

Napredno: profili so vgrajeni v namestitvi Chloros
pod `daq/cap_profiles/<u|m|e>/<cap_id>.json` in jih lahko posamezni uporabnik prepiše pod `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Ne glede na omejitve senzorji, ki še nikoli niso bili ponovno kalibrirani, samodejno prejmejo majhno izboljšavo temnega odmika, izpeljano iz flote – brez posredovanja uporabnika.

## DAQ-E-S: fiksni difuzor in nagib

**DAQ-E-S** je DAQ-E z dvema razlikama, ki sta pomembni zChloros
a. Vse ostalo (odkrivanje, povezovanje, PTP, snemanje, nastavitve naprave) deluje točno tako kot pri DAQ-E.

* **Njegov difuzor je fiksni, zato ne potrebuje popravka omejitve.** Difuzor je bil nameščen na enoti že ob merjenju tovarniške kalibracije, zato ta kalibracija že prikazuje dejansko navzdolno sevanje. Chloros
mu ne uporablja nobenega profila omejitve, ne glede na zahtevo: omejitev, poimenovana po njem, se prezre s sporočilom, njegovi zapisi kažejo, da je omejitev izhajala iz modela, opozorilo o predpostavljeni omejitvi pa se zanj nikoli ne sproži. V nastavitvah projekta → DAQ svetlobni senzor v njegovi vrstici ni prikazan noben omejevalnik, prav tako pa nanj ne vpliva prekrivanje omejevalnika na ravni projekta. Na njegove podatke ne uporabljajte omejevalnika ročno: popravek sončnega sevanja bi v tem primeru pokazal približno 11-krat višjo vrednost.
* **Opremljen je s 3-osnim pospeškomerom.** Kosinusni odziv difuzorja velja le, ko je senzor v vodoravnem položaju, zato nagnjeni senzor poroča o napačni obsevanosti brez drugih simptomov. Vsaka odčitana vrednost DAQ-E-S zato vsebuje podatke o nagibu, vrtenju in naklonu, `.daq`, posnet iz enega senzorja, pa jih shrani ob vsakem spektru (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, …), tako da lahko razlikujete med oblakom in poškodovanim nosilcem.

Za vsako polje lege in način poročanja o napaki v toku podatkov o legi glejte [DAQ-E-S v referenčnem priročniku za CLI
](../reference/cli-reference.md#daq-e-s--fixed-diffuser-and-tilt-sensor).

## Delovanje sončnega pokrova (konfiguracija za zunanjo uporabo)

Številke, na katerih lahko gradite postopke:

| Lastnost | Vrednost |
| --- | --- |
| Vidno polje | 180° polkrožno |
| Napaka kosinusnega odziva | ≤ ±4 % do 60° vpadnega kota; ≤ ±4,5 % do 70° |
| Meja za nizko sonce | Ne priporoča se pod ~15° višine sonca |
| Slabljenje | ~12× (po zasnovi) |
| Ponovljivost ponovne namestitve pokrova | ≈ 1,5 % |
| Količinska sončna obsevanost | Povprečje **≥ 15 s** odčitkov (značilnost merilnega instrumenta, ne gre za napako) |

Za katero koli vrednost količinske sončne obsevanosti – vključno z referenčnimi vrednostmi odbojnosti – uporabite povprečje odčitkov iz najmanj 15 sekund namesto posameznega posnetka.

## Kalibrirano spektralno območje

| Lastnost | Vrednost |
| --- | --- |
| Spektralno vzorčenje | 340–1010 nm v korakih po 5 nm (135 točk) |
| Radiometrično kalibrirano območje | **~374–974 nm** (določeno v programski opremi) |

Senzor poroča o celotni mreži 340–1010 nm, vendar radiometrična ojačitev, sledljiva po standardih NIST, zajema območje ~374–974 nm. Chloros
**zavrača delitev po absolutni odbojnosti** za kateri koli pas kamere, katerega spektralna teža v tem obsegu ne dosega niti polovice, in poroča o razlogu za preskok `dls-uncalibrated-band-<nm>`, namesto da bi ustvaril nekalibriran izdelek. Med serijskimi kamerami le filter F988 ne sodi v ta razpon; namesto tega uporablja delovni tok z odbojnostno ploščo – glej [Delovni tokovi odbojnosti](reflectance.md).

Za modele senzorjev, transportne poti in identifikacijske številke senzorjev glejte [Pregled DAQ](README.md). Za informacije o tem, kako se med obdelavo porablja omejitev števila posnetkov, glejte [Snemanje in format .daq](recording.md).

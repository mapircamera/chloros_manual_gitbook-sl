# Profili pokrovčkov in kalibrirano območje

> Sami pokrovčki – kateri pokrovček je priložen kateremu senzorju, kako se pritrdijo in kakšno je njihovo optično obnašanje – so opisani v **[uporabniškem priročniku za DAQ](https://mapir.gitbook.io/daq)**. Ta stran obravnava *deklaracijo* nameščenega pokrova v Chloros, kar omogoča pravilno korekcijo.

Tovarniška radiometrična kalibracija vsakega svetlobnega senzorja DAQ opisuje *gol* senzor. Fizični pokrovček, nameščen nad difuzorjem, spreminja svetlobo, ki jo senzor zajema, zato Chloros na kalibracijski paket uporabi tovarniško izmerjen **profil korekcije pokrovčka**. Določitev pravega pokrovčka je del pridobivanja kalibriranih podatkov – ta stran obravnava, kateri pokrovčki obstajajo za posamezne modele, kako jih določiti in kakšen je dejanski kalibrirani spektralni razpon senzorja.

## Razpoložljivost pokrovčkov po modelih

| Profil pokrovčka (`cap_id`) | Fizični pokrovček | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Pokrovček za korekcijo sončnega kosinusa (**privzeto na vsakem modelu**) | Da | Da | Da |
| `fov_15` / `fov_45` / `fov_90` | Koni za omejevanje vidnega polja (15° / 45° / 90°) | Da | — | Da |
| `fov_30` / `fov_60` | Koni za omejevanje vidnega polja (30° / 60°) | Da | — | — |
| `none` | Brez nameščenega pokrova | — | — | Da |

Opombe, specifične za posamezne modele:

* **DAQ-M ima en profil pokrova: `sunshine_cosine`.** »Bare-plus-Sunshine-cap« je njegova opredelitev izdelka, zato osnovni DAQ-M ne potrebuje geometrijskega profila.
* **Osnovni DAQ-U je resnično »bare«** — sploh ne potrebuje geometrijskega profila, zato zanj ne obstaja profil `none`.
* **`none` na DAQ-E NI neaktivna nastavitev.** Vdolbeni, s steklom pokriti difuzor DAQ-E ima lastno dejansko geometrijsko korekcijo, zato je »brez pokrova« pri tem modelu sam po sebi izmerjen profil.
* **Goli DAQ-E ne more meriti neposrednega sonca pri nobeni višini** — pokrovček Sunshine je konfiguracija za uporabo na terenu. Ne načrtujte dela na prostem z golim DAQ-E.

V nastavitvah po posameznih senzorjih v grafičnem uporabniškem vmesniku (ikonka zobnika v zavihku Svetlobni senzorji) spustni meni **Pokrovček** na modelih DAQ-U in DAQ-M ponuja tudi možnost »Nič (goli senzor)« — na teh dveh modelih »gol« preprosto pomeni, da se korekcija pokrovčka ne uporablja, kot je navedeno v zgornjih opombah. To možnost izberite le, če je pokrovček fizično odstranjen.

## Navedba pokrovčka — in zakaj je to pomembno

**Navedena oznaka `cap_id` mora ustrezati pokrovčku, ki je fizično nameščen na senzorju.** Niti senzor niti programska oprema ne moreta zaznati nameščenega pokrova. Deklaracija vpliva na dve stvari:

1. **Korekcijo v realnem času**, ki se uporablja za vsak spekter.
2. **Oznako pokrova, vpisano v vsak zapis `.daq`**, na katero se zanaša nadaljnja obdelava odbojnosti.

Pokrovček Sunshine **po zasnovi**zmanjša svetlobo za približno**12-krat**, zato snemanje z napačno deklariranim pokrovčkom povzroči napačno skaliranje spektrov za približno ta faktor. Spremembe pokrovčka takoj deklarirajte.

### Nastavitev pokrovčka

GUI: Zavihek Svetlobni senzorji → ikona zobnika v vrstici senzorjev → spustni meni **Pokrovček**. Privzeta vrednost za vsak model je `sunshine_cosine` (vsi senzorji DAQ se dobavljajo z vgrajenim kosinusnim korektorjem), izbira pa ostane shranjena v projektu.

<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->

CLI (backend mora biti v teku):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

CLI sintaktično sprejema celoten seznam `cap_id` (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); vsak profil se ob povezavi preveri glede na model senzorja, zato nedosegljiva identifikacijska številka senzorja (na primer identifikacijska številka, ki velja le za E-senzorje na DAQ-U) povzroči jasno napako namesto napačne korekcije. Privzeta vrednost backenda, če ni posredovano nič, je `sunshine_cosine`.

Python SDK opomba: `cap_id` **ni** gumb SDK — `connect_daq_sensor()` / `DAQSensorSession` ne razkrivata parametra pokrovčka. Izberite omejitev prek zgornjih ukazov CLI ali spustnega menija v grafičnem vmesniku; glejte [SDK Referenca](../reference/sdk-reference.md).

Napredno: profili so vgrajeni v namestitvi Chloros v `daq/cap_profiles/<u|m|e>/<cap_id>.json` in jih lahko posamezni uporabnik prepiše v `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Neodvisno od omejitev senzorji, ki še niso bili ponovno kalibrirani, samodejno prejmejo majhno izboljšavo temnega odmika, izpeljano iz podatkov flote – brez posredovanja uporabnika.

## Delovanje omejitve sončnega sevanja (konfiguracija za zunanjo uporabo)

Številke, na katerih lahko gradite postopke:

| Lastnost | Vrednost |
| --- | --- |
| Vidno polje | 180° polkrožno |
| Napaka kosinusnega odziva | ≤ ±4 % do kota padanja 60°; ≤ ±4,5 % do kota padanja 70° |
| Meja za nizko sonce | Ne priporoča se pod ~15° višine sonca |
| Slabljenje | ~12× (po zasnovi) |
| Ponovljivost ponovnega namestitve pokrivala | ≈ 1,5 % |
| Količinska obsevanost | Povprečje **≥ 15 s** odčitkov (značilnost merilnega instrumenta, ne gre za napako) |

Za katero koli vrednost količinske obsevanosti — vključno z referenčnimi vrednostmi odbojnosti — uporabite povprečje odčitkov iz najmanj 15 sekund namesto posameznega posnetka.

## Kalibrirano spektralno območje

| Lastnost | Vrednost |
| --- | --- |
| Spektralno vzorčenje | 340–1010 nm v korakih po 5 nm (135 točk) |
| Radiometrično kalibrirano območje | **~374–974 nm** (določeno v programski opremi) |

Senzor poroča o celotni mreži 340–1010 nm, vendar se radiometrični dobiček, sledljiv po standardih NIST, razteza od ~374 do 974 nm. Chloros **zavrne delitev po absolutni odbojnosti** za kateri koli pas kamere, pri katerem je manj kot polovica spektralne teže znotraj tega obsega, in poroča o razlogu za preskok `dls-uncalibrated-band-<nm>`, namesto da bi ustvaril nekalibriran izdelek. Med trenutno dostopnimi kamerami le filter F988 ne sodi v to območje; namesto tega uporablja delovni tok z odbojnostno ploščo – glej [Delovni tokovi odbojnosti](reflectance.md).

Za modele senzorjev, transportne poti in ID-je senzorjev glejte [Pregled DAQ](README.md). Za informacije o tem, kako se med obdelavo porablja omejitev števila posnetkov, glejte [Snemanje in format .daq](recording.md).

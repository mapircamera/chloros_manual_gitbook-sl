# Hitri začetek z CLI-jem (pool-*)

Priloženi `chloros-cli` upravlja senzorje DAQ prek družine ukazov **`daq pool-*`** — preprostih HTTP-odjemalcev, ki senzorje upravljajo prek trajnega sklopa senzorjev v ozadju Chloros. Backend upravlja s prenosom, zato si grafični vmesnik (GUI), CLI in skripti SDK delijo eno aktivno povezavo, namesto da bi se borili za vrata. Vse, kar stranka potrebuje, je dostopno prek `pool-*`: povezovanje, pretakanje, snemanje kalibriranih datotek `.daq` in zamenjava profilov kapic.

`pool-*` je podprta DAQ-površina v izdanih različicah. `chloros-cli daq --help` prikaže seznam podukazov `pool-*`, `daq discover` / `daq list` izvedejo isto skeniranje kot `pool-discover`. Kadar podukaz DAQ za neposredno upravljanje strojne opreme ni na voljo, se program zaključi z izrecno napako, ki navaja manjkajoči paket in vas usmeri nazaj na `pool-*` — nič ne odpove tiho. (Tudi `pip install chloros-sdk` ne zagotavlja ukazov za neposredno komunikacijo s strojno opremo.)

***

## Predpogoji

* **Zadnji del sistema Chloros mora teči** — ukazi `pool-*` so HTTP-odjemalci, ne pa strojne gonilnike. V sistemu Windows zaženite namizno aplikacijo Chloros (ta zažene zadnji del sistema). Na sistemu Linux/Jetson brez grafičnega vmesnika omogočite storitev kot svoj uporabnik: `sudo systemctl enable --now chloros-backend@$USER.service` (glejte [Namestitev v sistemu Linux](../linux/linux-installation.md#always-on-ptp-for-headless-hosts)).
* **Prijava v Chloros+ (plačljiva stopnja)**: najprej zaženite `chloros-cli login`. Preverjanje velja na strani strežnika — brez prijave ukazi ne uspejo z `401 AUTH_REQUIRED`; v brezplačni (Iron) stopnji ne uspejo z `403 PLAN_UPGRADE_REQUIRED`.
* Ukazi so privzeto namenjeni `http://127.0.0.1:5000`; družina `daq pool-*` upošteva okoljsko spremenljivko `CHLOROS_BACKEND_URL`, če vaš backend teče drugje.

***

## Petminutna seja

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)
#    (a DAQ-M's BLE MAC is not printed anywhere: find it with `chloros-cli daq pool-discover --only ble`)

# 2. List the pool — this shows the sensor_id used by every command below
chloros-cli daq pool-list

# 3. Read the most recent calibrated spectrum frame (add --json for scripting)
chloros-cli daq pool-latest --sensor-id daq-e-def330 --json

# 4. Record a calibrated .daq file for 60 seconds
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 60 \
  --device-name "field-A"

# 5. Release the sensor when done
chloros-cli daq pool-disconnect --sensor-id daq-e-def330
```

***

## `pool-connect` — odpri senzor v bazenu

| Različica | Pomen |
| --- | --- |
| `daq pool-connect` | Pametno zaznavanje: poišči kateri koli DAQ na tem računalniku. |
| `daq pool-connect --port PORT` | DAQ-U na določenem serijskem vmesniku (npr. `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | DAQ-M prek BLE, samodejno skeniranje MAC-naslova. |
| `daq pool-connect --mac MAC` | DAQ-M na znanem BLE MAC-u (kar pomeni `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E z znanim imenom gostitelja ali IP-naslovom — **zanesljiva pot**. |
| `daq pool-connect --eth` | DAQ-E s samodejnim odkrivanjem (mDNS, z nadomestno rešitvijo ARP). Glejte opozorilo spodaj. |

Zastavice za nastavitev, vse so neobvezne:

| Zastavica | Pomen |
| --- | --- |
| `--integration-time MS` / `-t MS` | Ročni integracijski čas v milisekundah. |
| `--frame-avg N` / `-f N` | Število okvirjev, iz katerih se izračuna povprečje za posamezni spekter. |
| `--no-ae` | Onemogoči samodejno izpostavitev (AE je privzeto vklopljena). |
| `--no-stream` | Poveži se brez zagona prenosa (kasneje nadaljuj s `pool-stream --start`). |
| `--cap-id CAP` | Profil popravka zgornje meje; privzeta nastavitev v ozadju je `sunshine_cosine`. Glej [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

Če povežete DAQ-E, ki je že v skupini pod drugo od svojih naslovov — pod imenom gostitelja `daq-e-<id>.local` ali pod svojim IP-naslovom —, se vrne obstoječi vnos v skupini, namesto da bi se enota odprla še enkrat.

{% hint style="warning" %}
**Opozorilo glede samodejnega odkrivanja `--eth`.** Na gostitelju z več omrežnimi vmesniki (več kot en aktivni omrežni vmesnik) se lahko *prvi* `pool-connect --eth` po zagonu izkaže za praznega, čeprav je senzor v dobrem stanju – iskanje lahko spregleda vmesnik senzorja, dokler je ARP-predpomnilnik prazen. Če `--eth` ne najde ničesar, poskusite znova ali pa iskanje v celoti preskočite z `--eth-host <ip-or-hostname>`, kar je zanesljiva pot na računalnikih z več omrežnimi vmesniki. Ime gostitelja DAQ-E je `daq-e-<id>.local` (npr. `daq-e-def330.local`); deluje tudi njegov navaden IP-naslov.
{% endhint %}

## `pool-discover` — skeniranje brez povezovanja

```bash
chloros-cli daq pool-discover                  # USB, BLE and Ethernet
chloros-cli daq pool-discover --only ble       # BLE only: this is how you find a DAQ-M's MAC
chloros-cli daq pool-discover --json           # machine-readable
```

Išče senzorje, s katerimi se lahko povežete, in prikaže naslov za povezavo, ne da bi se dejansko povezali. `--only` sprejme z vejico ločen podnabor `usb,ble,eth`; `--timeout SEC` nastavi čas iskanja na posamezni transport (privzeto 5 s, omejeno na 1–20 s). Senzorji, ki so že odprti v sklopu, niso navedeni (priključen DAQ-M preneha oddajati podatke) — za te uporabite `pool-list`. V dobavljeni različici `chloros-cli daq discover` in `daq list` izvajata isto skeniranje.

## `pool-list` — prikaz povezanih naprav

Prikaže vse senzorje v bazenu na strežniški strani, vključno z `sensor_id`, ki ga potrebujejo vsi drugi ukazi:

| Model | Oblika `sensor_id` | Primer |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-oktetni z vezajem | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

DAQ-E, pri katerem je bil PTP izklopljen (nastavitev je shranjena na enoti), je v `pool-list` označen z `!! PTP is switched OFF on this unit (saved in its flash): its timestamps are not PTP-synced.` in ukazom, ki ga ponovno vklopi — glej [`pool-enable-ptp`](#pool-enable-ptp-switch-a-daq-es-ptp-back-on).

## `pool-latest` — branje spektralnih okvirjev

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Vrne najnovejši okvir ali najnovejše okvire `--recent N`; `--json` izpiše strojno berljiv izhod za skriptiranje. Okvirji predstavljajo radiometrično kalibrirano spektralno sevanje (W/m²/nm) na 135-točkovni mreži v območju 340–1010 nm, pri čemer je že upoštevan profil pokrova senzorja. Za kvantitativne vrednosti sevanja izračunajte povprečje vsaj 15 sekund posnetkov – to je značilnost instrumenta, ne pa napaka.

## `pool-stream` — zaustavitev ali nadaljevanje prenosa

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` — snemanje datoteke `.daq`

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| Oznaka | Privzeto | Pomen |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | Trajanje snemanja v sekundah; `0` pomeni, da se izvaja, dokler ne izdate ukaza `--stop`. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Izhodni imenik. Pri strežniku na tem računalniku se relativna pot razreši glede na mapo, v kateri ste izvedli ukaz; pri oddaljenem strežniku se pot razreši **na računalniku, na katerem teče strežnik**. |
| `--device-name NAME` | — | Oznaka, shranjena skupaj s posnetkom. |
| `--stop` | — | Prekini tekoči posnetek. |
| `--confirm-after SEC` | `20` | Po zagonu počakajte toliko časa, da se zapiše prva vrstica, in glasno opozorite (`!! WARNING: no rows written after …s. This recording is producing an EMPTY file.`), če ta ne prispe. `0` onemogoči preverjanje. |
| `--heartbeat SEC` | `30` | Z `--duration` izpiši število vrstic vsakih SEC sekund in opozori, če se število ne povečuje več. `0` to onemogoči. |

{% hint style="info" %}
Snemanje poteka v ozadju, zato se datoteka `.daq` shrani v datotečnem sistemu **strežnika v ozadju** — privzeto v `~/Documents/DAQ Live View/` tam, ne nujno tam, kjer ste zagnali CLI. Imena datotek vključujejo ID senzorja in časovni žig.
{% endhint %}

## `pool-set-cap` — določite nameščeno pokrovčko

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

ID pokrovčka izbere tovarniško izmerjeni korekcijski profil, ki se uporabi za vsak spekter, in **mora ustrezati pokrovčku, ki je fizično nameščen na senzorju** — niti senzor niti programska oprema ne moreta sama zaznati pokrovčka, izbira pa je vtisnjena v vsako datoteko `.daq`. Privzeta vrednost povsod je `sunshine_cosine` (vsak DAQ se dobavlja z nameščenim pokrovčkom Sunshine za kosinusno korekcijo, ki ima po zasnovi ~12-kratno slabljenje — nedeklarirana zamenjava pokrovčka povzroči napačno korekcijo spektrov približno za ta faktor).

| `--cap-id` | Na voljo za |
| --- | --- |
| `sunshine_cosine` (privzeto) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | samo DAQ-U |
| `none` | samo DAQ-E — glej opombo |

ID pokrovčka, ki ne sodi v nabor senzorja, se ob povezovanju zavrne z jasno napako. `none` (DAQ-E) pomeni, da je pokrovček fizično odstranjen — še vedno se uporablja tovarniški geometrijski profil za vdolbeni stekleni difuzor DAQ-E, zato ne gre za neaktivno stanje, goli DAQ-E pa je konfiguracija za laboratorijsko delo, ne pa podprta konfiguracija za uporabo v terenu. (Goli DAQ-U je resnično gol in sploh ne potrebuje korekcijskega profila; DAQ-M se uporablja s pokrovčkom Sunshine.)

## `pool-disconnect` — sprostitev senzorjev

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

`pool-disconnect` sprosti senzor ne glede na to, kdo ga je odprl, zato sprosti tudi senzor, ki ga je v bazenu pustil sesut skript (najprej se zaključi snemanje na njem). Senzor, ki ni v bazenu, nima ničesar za sprostitev: ukaz izpiše `<sensor-id>: already released (not in the pool)` in se zaključi z `0`. Sprostitev, ki jo je backend zavrnil ali ni potrdil, se zaključi z `1`.

## `pool-enable-ptp` — ponovno vklopite PTP na DAQ-E

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

DAQ-E shranjuje nastavitev vklopa/izklopa PTP v lastnem pomnilniku, zato naprava, katere PTP je bil izklopljen, ostane izklopljena — in njeni časovni žigi niso sinhronizirani s PTP — dokler se ne vklopi nazaj. `pool-list` takšno enoto označi in zanjo izpiše ta ukaz. Program Chloros enote nikoli samodejno ponovno ne vklopi, saj je lahko izklop nameren. Novost v različici 1.3.0.

## Nastavitve naprave DAQ-E

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto
chloros-cli daq pool-set-auto-stream --sensor-id daq-e-def330 --on
chloros-cli daq pool-reboot --sensor-id daq-e-def330
chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

`pool-info` prikaže firmware senzorja (in ali ta različica Chlorosa vsebuje novejšo), njegov naslov in kako ga je pridobil, shranjene omrežne nastavitve, če se razlikujejo od trenutno veljavnih, pretakanje ob vklopu in PTP. `pool-set-network` ponovno zažene senzor, da se sprememba uporabi, in ga ponovno poveže z nastavitvami, ki jih je imel; `--no-reboot` jih le shrani. Statični naslov mora biti v omrežju tega računalnika, sicer ga Chloros zavrne; `--force` to prekliče, vendar le za računalnik, ki bo preusmerjen na ta naslov ali usmerjen vanj. Ponovni zagon ni dovoljen, medtem ko senzor beleži. `pool-update-firmware` je v grafičnem vmesniku **Update to \<version\>** za ukazno vrstico ter način za posodobitev DAQ-E v sistemu Linux, ki nima grafičnega vmesnika. Novost v različici 1.3.0.

***

## Pregled ukazov

| Ukaz | Namen |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Odpre senzor v bazenu strežnika. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Iskanje senzorjev brez povezave (najde BLE MAC naslov naprave DAQ-M). |
| `daq pool-list` | Prikaže vse senzorje v skupini z njihovim `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Zadnjih N kalibriranih spektralnih okvirjev. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Nadaljuj / zaustavi pretakanje. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--confirm-after SEC] [--heartbeat SEC] [--stop]` | Začni / ustavi snemanje `.daq` (na strani strežnika). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Zamenjava profila korekcije omejitve med izvajanjem. |
| `daq pool-disconnect --sensor-id ID [--all]` | Sprostitev enega senzorja ali vseh senzorjev. |
| `daq pool-enable-ptp --sensor-id ID` | Ponovno vklopi PTP za DAQ-E, pri katerem je PTP izklopljen. |
| `daq pool-info --sensor-id ID [--json]` | Firmware, naslov, shranjene omrežne nastavitve, pretakanje ob vklopu in PTP za DAQ-E. |
| `daq pool-set-network --sensor-id ID (--auto \| --static IP [--mask M] [--gateway G]) [--no-reboot] [--force]` | DHCP ali statični naslov; ponovni zagon in ponovna povezava senzorja. |
| `daq pool-set-auto-stream --sensor-id ID (--on \| --off)` | Pretakanje ob vklopu. |
| `daq pool-reboot --sensor-id ID` | Ponovni zagon naprave DAQ-E in ponovna povezava. |
| `daq pool-update-firmware --sensor-id ID` | Naložite firmware za napravo DAQ-E iz te različice na starejši senzor. |

***

## Odpravljanje težav pri prvi povezavi z DAQ-E

1. DAQ-E nima LED-a za stanje — preverite napajanje prek indikatorja PoE/povezave na stikalu ali vstopnem vtičniku in po vklopu počakajte nekaj sekund, da se naprava zagoni in priključi na omrežje.
2. Strežnik mora biti v **isti oddajni domeni** kot senzor — mDNS ne prehaja skozi usmerjevalnike.
3. Požarni zid mora dovoliti prehod mDNS (UDP 5353), podatkov DAQ-E (UDP 5002) in PTP (UDP 319/320). Windowsov namestitveni program za to doda pravilo, Linuxov `.deb` pa jih odpre v `ufw` / `firewalld`, ki je bil aktiven ob namestitvi; požarni zid tretje osebe ali tisti, ki je bil omogočen kasneje, mora imeti te protokole dovoljene.
4. Še vedno nič od `--eth`? Uporabite `--eth-host` z imenom gostitelja naprave (`daq-e-<id>.local`) ali njenim IP-naslovom, kar preskoči postopek odkrivanja.

***
{% hint style="info" %}
**Nasvet za AI-pomočnike.** Vsaka stran tega priročnika je na voljo v surovem formatu Markdown — dodajte `.md` k malemu URL-slugu strani (ta stran: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); indeks, ki ga lahko berejo stroji, je `https://mapir.gitbook.io/chloros/llms.txt`. Za popolno dokumentacijo na ravni zastavic za `chloros-cli daq` in vsako drugo družino ukazov si oglejte [Referenco CLI](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); pot v Pythonu je `chloros_sdk.connect_daq_sensor()` v [SDK Reference](../reference/sdk-reference.md).
{% endhint %}

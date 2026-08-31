# CLI Hitri začetek (pool-*)

Priloženi `chloros-cli` poganja senzorje DAQ prek družine ukazov **`daq pool-*`** — tanke odjemalce HTTP, ki upravljajo senzor prek trajnega sklopa senzorjev zadnjega dela Chloros. Backend upravlja s prenosom, zato si grafični vmesnik (GUI), skripti CLI in SDK delijo eno aktivno ročajno številko, namesto da bi se borili za vrata. Vse, kar stranka potrebuje, je dostopno prek `pool-*`: povezovanje, pretakanje, snemanje kalibriranih datotek `.daq` in zamenjava profilov kapic.

`pool-*` je tudi **edini** DAQ-vmesnik v izdanih različicah. `chloros-cli daq --help` prikazuje seznam podukazov `pool-*`, pri čemer se ob klicu podukaza DAQ za neposredno upravljanje strojne opreme v izdani različici program zaključi z izrecno napako, ki navaja manjkajoči paket in vas usmerja nazaj na `pool-*` — nič ne odpove tiho. (Ukazi za neposredno komunikacijo s strojno opremo delujejo le iz izvlečka izvorne kode MAPIR; tudi `pip install chloros-sdk` jih ne zagotavlja.)

***

## Predpogoji

* **Zadnji del Chloros mora teči** — ukazi `pool-*` so odjemalci HTTP, ne pa strojne gonilnike. Na Windows zaženite namizno aplikacijo Chloros (ta zažene backend). Na napravi Linux/Jetson brez zaslona omogočite storitev: `sudo systemctl enable --now chloros-backend.service`.
* **Prijava v Chloros+ (plačljiva stopnja)**: najprej zaženite `chloros-cli login`. Preverjanje veljavnosti poteka na strežniški strani – brez prijave ukazi z `401 AUTH_REQUIRED` ne uspejo; v brezplačni (Iron) stopnji ne uspejo z `403 PLAN_UPGRADE_REQUIRED`.
* Ukazi so privzeto namenjeni `http://127.0.0.1:5000`; družina `daq pool-*` upošteva spremenljivko okolja `CHLOROS_BACKEND_URL`, če vaš backend teče drugje.

***

## Petminutna seja

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)

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
| `--frame-avg N` / `-f N` | Število okvirjev, iz katerih se izračuna povprečje za vsak poročani spekter. |
| `--no-ae` | Onemogoči samodejno izpostavljenost (AE je privzeto vklopljena). |
| `--no-stream` | Poveži se brez zagona prenosa (kasneje nadaljuj s `pool-stream --start`). |
| `--cap-id CAP` | Profil popravka zgornje meje; privzeta nastavitev backenda je `sunshine_cosine`. Glej [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

{% hint style="warning" %}
**Opozorilo glede samodejnega odkrivanja `--eth`.** Na gostitelju z več omrežnimi vmesniki (več kot enim aktivnim omrežnim vmesnikom) se lahko *prvi* `pool-connect --eth` po zagonu izkaže za praznega, čeprav je senzor v dobrem stanju — iskanje senzorjev lahko spregleda senzorjev vmesnik, dokler je ARP-predpomnilnik prazen. Če `--eth` ne najde ničesar, poskusite znova ali pa iskanje v celoti preskočite z `--eth-host <ip-or-hostname>`, kar je zanesljiva pot na računalnikih z več omrežnimi vmesniki. Ime gostitelja DAQ-E je `daq-e-<id>.local` (npr. `daq-e-def330.local`); deluje tudi njegov navaden IP-naslov.
{% endhint %}

## `pool-list` — oglejte si, kaj je priključeno

Prikaže vse senzorje v skladišču v ozadju, vključno z `sensor_id`, ki ga potrebujejo vsa ostala ukaza:

| Model | Oblika `sensor_id` | Primer |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-oktetna z vezaji | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

## `pool-latest` — branje spektralnih okvirjev

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Vrne najnovejši okvir ali najnovejše okvire; `--recent N`; `--json` izpiše strojno berljiv izhod za skripte. Okvirji predstavljajo radiometrično kalibrirano spektralno sevanje (W/m²/nm) na 135-točkovni mreži v območju 340–1010 nm, pri čemer je že upoštevan profil pokrovčka senzorja. Za kvantitativne vrednosti sevanja izračunajte povprečje vsaj 15 sekund posnetkov – to je značilnost instrumenta, ne pa napaka.

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
| `--duration SEC` / `-d SEC` | `0` | Trajanje snemanja v sekundah; `0` pomeni, da se izvaja, dokler ne izdate `--stop`. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Izhodni imenik, določen **na računalniku, na katerem teče backend**. |
| `--device-name NAME` | — | Oznaka, shranjena skupaj s snemanjem. |
| `--stop` | — | Zaustavi snemanje, ki je v teku. |

{% hint style="info" %}
Snemanje poteka v strežniku, zato se datoteka `.daq` shrani v datotečnem sistemu **računalnika z backendom** — privzeto v `~/Documents/DAQ Live View/` tam, ne nujno tam, kjer ste zagnali CLI. Imena datotek vključujejo ID senzorja in časovni žig.
{% endhint %}

## `pool-set-cap` — določite nameščeno pokrovčko

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

ID pokrovčka izbere tovarniško izmerjeni korekcijski profil, ki se uporabi za vsak spekter, in **mora ustrezati pokrovčku, ki je fizično nameščen na senzorju** — niti senzor niti programska oprema ne moreta samostojno zaznati pokrovčka, izbira pa je vtisnjena v vsako datoteko `.daq`. Privzeta vrednost povsod je `sunshine_cosine` (vsak DAQ se dobavlja z nameščenim pokrovčkom Sunshine za kosinusno korekcijo, ki ima po zasnovi ~12-kratno slabljenje — nedeklarirana zamenjava pokrovčka povzroči napačno korekcijo spektrov približno za ta faktor).

| `--cap-id` | Na voljo za |
| --- | --- |
| `sunshine_cosine` (privzeto) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | samo DAQ-U |
| `none` | samo DAQ-E — glej opombo |

ID pokrovčka, ki ne spada v nabor senzorja, se ob povezovanju zavrne z jasno napako. `none` (DAQ-E) pomeni, da je pokrovček fizično odstranjen — še vedno se uporablja tovarniški geometrijski profil za vdolbeni stekleni difuzor DAQ-E, zato ne gre za brezdelovanje, goli DAQ-E pa je konfiguracija za laboratorijsko delo, ne pa podprta konfiguracija za uporabo na terenu. (Goli DAQ-U je resnično gol in ne potrebuje nobenega korekcijskega profila; DAQ-M se uporablja s pokrovčkom Sunshine.)

## `pool-disconnect` — sprostitev senzorjev

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

***

## Povzetek ukazov

| Ukaz | Namen |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Odpri senzor v bazenu backenda. |
| `daq pool-list` | Prikaži vse senzorje v sklopu z njihovimi `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Zadnjih N kalibriranih spektralnih okvirjev. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Nadaljuj / zaustavi pretakanje. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--stop]` | Začni / ustavi snemanje `.daq` (na strani backenda). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Zamenjava profila popravka omejitve med delovanjem. |
| `daq pool-disconnect --sensor-id ID [--all]` | Sprostitev enega senzorja ali vseh senzorjev. |

***

## Odpravljanje težav pri prvi povezavi z DAQ-E

1. DAQ-E nima LED-a za stanje — preverite napajanje prek indikatorja PoE/povezave na stikalu ali vstopnem vtičniku in po vklopu počakajte nekaj sekund, da se naprava zagoni in priključi na omrežje.
2. Strežnik mora biti v **isti domenski skupini** kot senzor – mDNS ne prehaja skozi usmerjevalnike.
3. Na napravi Windows ob prvem zagonu potrdite poziv požarnega zidu Defender (mDNS UDP 5353, podatki DAQ-E UDP 5002, PTP UDP 319/320).
4. Še vedno ni odziva od `--eth`? Uporabite `--eth-host` z imenom gostitelja naprave (`daq-e-<id>.local`) ali IP-naslovom – to je zanesljiva pot, zlasti na gostiteljih z več povezavami.

***{% hint style="info" %}**Nasvet za AI-pomočnike.** Vsaka stran tega priročnika je na voljo v surovem formatu Markdown – dodajte `.md` k malemu slugu strani URL (ta stran: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); indeks, ki ga lahko berejo stroji, je `https://mapir.gitbook.io/chloros/llms.txt`. Za popolno dokumentacijo na ravni zastavic za `chloros-cli daq` in vsako drugo družino ukazov prenesite [CLI Referenco](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); pot do Python je `chloros_sdk.connect_daq_sensor()` v [SDK Reference](../reference/sdk-reference.md).
{% endhint %}

# DAQ-E: omrežje in sinhronizacija časa

> Fizična namestitev omrežja za senzor – ožičenje, PoE, dodelitev IP-naslova in lastne omrežne nastavitve naprave – je opisana v **[uporabniškem priročniku za DAQ](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Ta stran obravnava stran Chloros: povezovanje, sinhronizacijo časa in kaj storiti, če iskanje ne prinese rezultatov.

DAQ-E je član družine DAQ, ki deluje prek omrežja Ethernet: napaja se prek PoE, odkrije se prek mDNS (storitev `_daq-e._tcp`), nasloviti pa ga je mogoče z imenom gostitelja, ki izhaja iz njegove identifikacijske številke senzorja — `daq-e-<6 hex>.local`, npr. `daq-e-def330.local`. Ta stran obravnava, kako naprava prenaša podatke po omrežju in kako sodeluje pri časovni sinhronizaciji PTP.

## Načini prenosa

| Način | Končna točka | Uporabniki | Opombe |
| --- | --- | --- | --- |
| **Multicast** (privzeto) | UDP `239.10.10.10:5002` | Vsak naslov v istem LAN-omrežju prejme isti tok | Vsak datagram je preverjen s CRC-16/CCITT |
| **Raw** | TCP vrata `5000` | Točno en odjemalec (izključno) | Neposredno združljiv na ravni bajtov z DAQ-U |

Chloros privzeto uporablja multicast, kar omogoča, da GUI, CLI in SDK hkrati spremljajo en senzor.

## Zahteve omrežja

* **Ista domena oddajanja.** Računalnik, na katerem teče Chloros, mora biti v istem segmentu omrežja L2 kot senzor — odkrivanje mDNS ne prehaja skozi usmerjevalnike.
* **Požarni zid.** V sistemu Windows namestitveni program v vsak omrežni profil doda pravilo požarnega zida **Chloros Backend (UDP-In)**za backend, ki zajema podatke DAQ-E (UDP 5002), mDNS (UDP 5353) in PTP (UDP 319/320), ter pravilo**»Chloros Backend (TCP-In)«**, omejeno na lokalno podomrežje, za povezavo, ki jo DAQ-E vzpostavi nazaj z računalnikom med posodobitvijo firmware-a. V sistemu Linux `.deb` odpre vrata UDP v `ufw` ali `firewalld`, če je ta požarni zid aktiven ob namestitvi. Za požarni zid tretje osebe ali tistega, ki je bil omogočen kasneje, je treba dovoliti iste vrata.
* **Napajanje prek PoE, brez LED-a za stanje.** Naprava DAQ-E nima lastnega LED-a – preverite napajanje prek indikatorja povezave/PoE na stikalu ali vstopni vratih injektorja ter po vklopu počakajte nekaj sekund, da se naprava zagoni in priključi na omrežje.

## Povezovanje

**GUI:** Zavihek Svetlobni senzorji → Poveži senzor → Vrsta naprave »DAQ-E (Ethernet)«. Iskanje poteka le, dokler je na zaslonu prikazano okno za povezovanje (brskanje po mDNS in ARP-skeniranje v sistemu Windows), in se ponavlja vsakih 15 sekund; gumb Osveži takoj ponovno izvede skeniranje. Odkriti senzorji se prikažejo v spustnem meniju; prvi zaznan senzor je samodejno izbran.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (teče backend):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

DAQ-E, ki je že v skupini, se prepozna po imenu gostitelja in po IP-naslovu: če ga ponovno povežete pod drugim naslovom, se prikaže obstoječi vnos v skupini, namesto da bi se enota odprla še enkrat.

### Gostitelji z več mrežnimi karticami in prva povezava po zagonu

Na gostiteljih z več kot enim aktivnim omrežnim vmesnikom se lahko **prva** povezava `pool-connect --eth` po zagonu izkaže za prazno, čeprav je senzor v dobrem stanju – iskanje senzorjev lahko spregleda vmesnik, na katerem se senzor nahaja, dokler je ARP-predpomnilnik še prazen. Zanesljiva rešitev je, da preskočite iskanje in naslov izrecno navedete:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` sprejme ime gostitelja mDNS ali IP-naslov, vedno cilja na pravi senzor in je priporočena oblika za skripte ter namestitve brez grafičnega vmesnika. V grafičnem vmesniku uporabite gumb »Osveži« v pogovornem oknu za povezavo in počakajte na nov krog iskanja.

## Nastavitve naprave in vgrajena programska oprema

Senzor sam hrani svoje omrežne nastavitve: DHCP (tovarniška nastavitev, ki se preklopi na lokalni naslov povezave 169.254.x.x, če noben strežnik DHCP ne odgovori) ali statični naslov. Hrani tudi nastavitev, ali naj začne s prenosom podatkov ob vklopu. Te nastavitve spremenite v grafičnem vmesniku (v nastavitvah senzorja z ikono zobnika, **Mreža**:**Spremeni…**,**Prenos ob vklopu**,**Ponovni zagon senzorja**; glej [Zavihek DAQ](gui.md)), ali prek CLI-ja in SDK-ja s komandami `pool-set-network`, `pool-set-auto-stream` in `pool-reboot` (`DAQSensorSession.set_network()`, `set_auto_stream()`, `reboot()`; glejte [Referenco za CLI](../reference/cli-reference.md#daq-e-device-settings)). Senzor se ponovno zažene, da se upošteva sprememba v omrežju, Chloros pa ga ponovno poveže. V omrežju računalnika, na katerem teče Chloros, mora biti statični naslov: Chloros zavrne tistega, do katerega po ponovnem zagonu ne more dostopati. ID senzorja in njegovo ime gostitelja `daq-e-<id>.local` se nikoli ne spremenita. Novost v različici 1.3.0.

**Posodobitve firmware-a so vgrajene v grafični uporabniški vmesnik (GUI).** Ko priključen DAQ-E uporablja starejšo različico firmware-a od tiste, ki je priložena vaši različici Chloros, se v vrstici senzorja prikaže rumena ikona **Posodobitev na voljo**, v oknu nastavitev z zobnikom pa se prikaže <version>gumb</version>

»Posodobi na<version>«. Posodobitev se prenese prek omrežja v približno 30 sekundah; senzor se samodejno ponovno zažene in ponovno poveže, prekinjen prenos pa ohrani trenutno firmware nespremenjeno. V CLI-ju ali SDK-ju ter v sistemu Linux, ki nima grafičnega vmesnika, zaženite `chloros-cli daq pool-update-firmware --sensor-id <id>` (`DAQSensorSession.update_firmware()`).

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## Sinhronizacija časa PTP

Firmware DAQ-E v1.2.0+ sodeluje v standardu IEEE 1588 PTPv2 kot običajna ura (izključno kot podrejeni). **Backend gostitelja Chloros je PTP grandmaster** — vsaka naprava DAQ-E in vsaka kamera LATTICE v omrežju LAN deluje kot podrejena ura v domeni 0, pri čemer se vsi časovni žigi naprav ohranjajo z odstopanjem ~1 ms. Prav ta skupna ura omogoča, da se časovni žigi odčitkov DAQ ujemajo z ekspozicijami kamer (glej [Snemanje in format .daq](recording.md)).

Preverite sinhronizacijo prek CLI-ja:

| Ukaz | Prikaže |
| --- | --- |
| `chloros-cli time-sync status` | Stanje glavnega časovnega generatorja gostitelja, prioritete BMCA, identiteta ure ter `health`: `OK` ali `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` z `health_detail`, ki navaja razlog, zakaj njegovi paketi ne dosegajo omrežja |
| `chloros-cli time-sync peers` | Vsi zaznani podrejeni elementi (senzorji DAQ-E + kamere LATTICE) |
| `chloros-cli time-sync cameras` | Stanje PTP po posameznih kamerah (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Ponovni zagon procesa glavnega vodje |

V grafičnem uporabniškem vmesniku (GUI) okno z nastavitvami DAQ-E prikazuje vrstico **PTP Sync** v realnem času s trenutnim stanjem PTP senzorja.**DAQ-E z izklopljenim PTP ostane izklopljen.** Enota shrani nastavitev vklopa/izklopa PTP v lastnem pomnilniku, zato enota, pri kateri je bil PTP izklopljen, ostane izklopljena tudi po ponovni povezavi ali ponovnem zagonu, njeni časovni žigi pa niso sinhronizirani s PTP. `chloros-cli daq pool-list` takšno enoto označi (`!! PTP is switched OFF on this unit …`) in izpiše ukaz, ki to popravi:

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Chloros enote nikoli samodejno ponovno ne vklopi, saj je lahko izklop nameren.

Podrobnosti za uporabnike, ki zahtevajo strogo usklajevanje:

* Vsak prenesen datagram vsebuje polje z oznakami; **bit 2 je nastavljen na okvirjih, katerih časovni žig je sinhroniziran s PTP**. Potoki, ki zahtevajo strogo usklajevanje kamere/DAQ, morajo delovati na podlagi tega bita.
* Pred sinhroniziranim zajemom preverite, ali se senzor pojavi v `chloros-cli time-sync peers` in ali `chloros-cli daq pool-list` ne označuje njegovega PTP kot izklopljenega. (Notranja orodja MAPIR za neposredno upravljanje strojne opreme lahko prav tako omejijo snemanje na PTP-zaklep z zastavico `--wait-ptp`, ki čaka do 15 s, da senzor doseže stanje SLAVE; ta orodja niso del priloženega CLI.)
* Medtem ko PTP aktivno deluje kot suženj, senzor zavrača ročno posredovanje takta („PTP zagotavlja takt“). To je namerno – zaupajte PTP.

## Opombe za Linux

* **PTP-vrata brez root-pravic.** PTP se veže na UDP 319/320. `.deb` zapiše `/etc/sysctl.d/60-chloros-ptp.conf` (`net.ipv4.ip_unprivileged_port_start = 319`), zato jih backend veže in izvaja kot vaš uporabnik. Od različice 1.3.0 paket ne nastavi nobene datoteke z zmogljivostjo (`setcap`) na backendu in ne potrebuje `libcap2-bin`; sami ne dodajajte zmogljivosti, saj se v tem primeru razpakirani backend ne bo zagnal. Če PTP ne more zavezati vrat 319, glejte [Namestitev v sistemu Linux](../linux/linux-installation.md#ptp-will-not-start--cannot-bind-port-319).

* **Jetson brez zaslona / gostiteljski računalnik s sistemom Linux:** paket namesti storitve backenda, vendar nobene ne omogoči. Za stalno delovanje PTP (in razpoložljivost DAQ), ne da bi kdo izvajal ukaz, omogočite storitev kot svoj uporabnik:

  ```bash
  sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
  ```

  Brez tega PTP deluje le, dokler je ozadje zagnal nekdo drug – ukaz `chloros-cli` ali skript SDK. Glejte [Namestitev v sistemu Linux](../linux/linux-installation.md#always-on-ptp-for-headless-hosts) za uporabnika root `chloros-backend.service` in navodila za preklop iz njega.

## Odpravljanje težav: »Ni najdenih naprav DAQ-E«

| Preverjanje | Podrobnosti |
| --- | --- |
| Napajanje | LED na senzorju ne sveti – preverite indikatorje PoE in povezave na stikalu/vstavišču; po vklopu počakajte nekaj sekund |
| Domena oddajanja | Gostitelj in senzor sta v istem segmentu L2; mDNS ne usmerja |
| Požarni zid | Windows: pravilo **Chloros Backend (UDP-In)** v namestitvenem programu omogoča pretok DAQ-E (UDP 5002), mDNS (5353) in PTP (319/320); požarni zid tretje osebe mora imeti enake nastavitve. Linux: `.deb` odpre ta vrata v `ufw` / `firewalld` le, če je bil požarni zid aktiven ob namestitvi — jih sami omogočite v enem, ki ga kasneje vklopite |
| Gostitelj z več mrežnimi karticami | Odkrivanje brska po mDNS z nadomestnim ARP-jem; `--eth-host <ip-or-hostname>` preskoči odkrivanje in se poveže neposredno |
| Ponovno skeniranje v grafičnem vmesniku | Odkrivanje poteka le, dokler je odprto okno za povezavo; uporabite gumb »Osveži« |</version>

# Mrežno povezovanje in sinhronizacija časa za DAQ-E

> Fizična namestitev omrežja za senzor — ožičenje, PoE, dodelitev IP-naslova in lastne omrežne nastavitve naprave — je opisana v **[uporabniškem priročniku za DAQ](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Ta stran obravnava stran Chloros: povezovanje, sinhronizacijo časa in kaj storiti, če iskanje ne prinese rezultatov.

DAQ-E je član družine DAQ, ki deluje prek omrežja Ethernet: napaja se prek PoE, odkrije se prek mDNS (storitev `_daq-e._tcp`), nasloviti pa ga je mogoče z imenom gostitelja, ki izhaja iz njegove identifikacijske številke senzorja — `daq-e-<6 hex>.local`, npr. `daq-e-def330.local`. Ta stran obravnava, kako naprava prenaša podatke po omrežju in kako sodeluje pri časovni sinhronizaciji PTP.

## Načini prenosa

| Način | Končna točka | Uporabniki | Opombe |
| --- | --- | --- | --- |
| **Multicast** (privzeto) | UDP `239.10.10.10:5002` | Vsak naslov v istem LAN-omrežju prejme isti tok | Vsak datagram je preverjen s CRC-16/CCITT |
| **Raw** | TCP-vrata `5000` | Točno en uporabnik (izključno) | Neposredno združljiv na ravni bajtov z DAQ-U |

Chloros privzeto uporablja multicast, kar omogoča, da GUI, CLI in SDK hkrati spremljajo en senzor.

## Zahteve omrežja

* **Ista domena oddajanja.** Računalnik, na katerem teče Chloros, mora biti v istem segmentu omrežja L2 kot senzor — odkrivanje mDNS ne prehaja skozi usmerjevalnike.
* **Poziv požarnega zidu Windows: sprejmite ga.** Ko Chloros prvič poveže multicast vtičnice, Windows Defender enkrat zaprosi za dovoljenje. Če to dovolite, se zajamejo podatki DAQ-E (UDP 5002), mDNS (UDP 5353) in PTP (UDP 319/320). Pri Linux se to ne prikaže.
* **Napajanje PoE, brez statusne LED-lučke.** DAQ-E nima lastne LED-lučke — preverite napajanje prek indikatorja povezave/PoE na stikalu ali vmesniku injektorja ter po vklopu počakajte nekaj sekund, da se naprava zagoni in priključi na omrežje.

## Povezovanje

**GUI:** Zavihek Svetlobni senzorji → Poveži senzor → Vrsta naprave »DAQ-E (Ethernet)«. Iskanje poteka le, dokler je pogovorno okno za povezavo na zaslonu (brskanje po mDNS in pregled ARP na Windows), in se ponavlja vsakih 15 sekund; gumb »Osveži« takoj ponovno izvede iskanje. Odkriti senzorji se prikažejo v spustnem meniju; prvi zaznan senzor je samodejno izbran.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (teče strežniški del):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

### Gostitelji z več omrežnimi karticami in prva povezava po zagonu

Na gostiteljih z več kot enim aktivnim omrežnim vmesnikom se lahko **prvi** `pool-connect --eth` po zagonu prikaže kot prazen, čeprav je senzor delujoč – iskanje lahko spregleda vmesnik, na katerem se nahaja senzor, dokler je ARP-predpomnilnik še prazen. Zanesljiva rešitev je, da preskočite iskanje in naslov izrecno navedete:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` sprejema ime gostitelja mDNS ali IP-naslov, vedno cilja na pravi senzor in je priporočena oblika za skripte ter namestitve brez grafičnega vmesnika. V grafičnem vmesniku uporabite gumb »Osveži« v pogovornem oknu za povezavo in omogočite nov cikel iskanja.

## Nastavitve naprave in vgrajena programska oprema

Senzor sam hrani omrežne nastavitve – statični IP naslov v primerjavi z DHCP + lokalnim naslavljanjem povezave, ime naprave, samodejni prenos ob zagonu, geslo za OTA. Te nastavitve na strani naprave niso na voljo kot ukazi v priloženem CLI; upravljajo se prek grafičnega vmesnika Chloros, kjer so prikazane, ali s podporo MAPIR.

**Posodobitve firmware-a so vgrajene v grafični vmesnik.**Ko priključena naprava DAQ-E uporablja firmware, ki je starejši od slike, priložene vaši različici Chloros, se v vrstici senzorjev prikaže rumena ikona**Na voljo posodobitev**, v oknu z nastavitvami pa se prikaže<version>

gumb</version> »Posodobi na<version>

«. Posodobitev se prenese prek omrežja v približno 30 sekundah; senzor se samodejno ponovno zažene in ponovno poveže, prekinjen prenos pa ohrani trenutno različico firmwareja nespremenjeno.

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## Sinhronizacija časa PTP

Firmware DAQ-E v1.2.0+ sodeluje v standardu IEEE 1588 PTPv2 kot običajna ura (izključno kot podrejena ura). **Zadnji del gostitelja Chloros je PTP grandmaster** — vsaka naprava DAQ-E in vsaka kamera LATTICE v lokalnem omrežju (LAN) mu sledi v domeni 0, pri čemer se vsi časovni žigi naprav ohranjajo z odstopanjem ~1 ms. Prav ta skupna ura omogoča, da se časovni žigi odčitkov DAQ ujemajo z ekspozicijami kamer (glej [Snemanje in format .daq](recording.md)).

Preverite sinhronizacijo iz CLI:

| Ukaz | Prikaže |
| --- | --- |
| `chloros-cli time-sync status` | Stanje glavnega urejevalnika gostitelja, prioritete BMCA, identiteta ure |
| `chloros-cli time-sync peers` | Vse zaznane podrejene naprave (senzorji DAQ-E + kamere LATTICE) |
| `chloros-cli time-sync cameras` | Stanje PTP po posameznih kamerah (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Ponovni zagon procesa grandmasterja |

V grafičnem uporabniškem vmesniku (GUI) okno z nastavitvami DAQ-E prikazuje vrstico **PTP Sync** v realnem času s trenutnim stanjem PTP senzorja.

Podrobnosti za uporabnike, ki zahtevajo strogo usklajevanje:

* Vsak prenesen datagram vsebuje polje zastavic; **bit 2 je nastavljen na okvirjih, katerih časovni žig je sinhroniziran s PTP**. Potoki, ki zahtevajo strogo usklajevanje kamere in DAQ, morajo delovati na podlagi tega bita.
* Pred sinhroniziranim zajemanjem preverite, ali se senzor pojavi v `chloros-cli time-sync peers`. (Notranja neposredna strojna oprema modela MAPIR lahko prav tako omeji snemanje na PTP-zaklep z zastavico `--wait-ptp`, ki čaka do 15 s, da senzor doseže stanje SLAVE; ta orodje ni del dobavljenega CLI.)
* Medtem ko je PTP aktivno v podrejenem stanju, senzor zavrača ročno nastavljanje takta („PTP zagotavlja takt“). To je namerno – zaupajte PTP.

## Opombe k Linux

* **PTP potrebuje `libcap2-bin` ob namestitvi.** Postinst `.deb` dodeli `cap_net_bind_service=+ep` na `/usr/lib/chloros/chloros-backend`, da se lahko veže na PTP-vrata 319/320 brez root-pravic. Če `libcap2-bin` manjka, se ta korak preskoči in PTP se ne bo uspel zagnati. Rešitev:

  ```bash
  sudo apt install libcap2-bin
  sudo apt reinstall chloros
  ```

* **Jetson / Raspberry Pi brez grafičnega vmesnika:** ob prvi namestitvi se ustvari enota systemd `chloros-backend.service`, vendar ni omogočena. Za stalno delujoč PTP (in razpoložljivost DAQ) brez grafičnega vmesnika:

  ```bash
  sudo systemctl enable --now chloros-backend.service
  ```

  Brez nje PTP deluje le, dokler je odprt grafični vmesnik Chloros.

## Odpravljanje težav: »Ni najdenih naprav DAQ-E«

| Preverjanje | Podrobnosti |
| --- | --- |
| Napajanje | LED na senzorju ne sveti — preverite indikatorje PoE in povezave na stikalu/vtočnem vmesniku; po vklopu počakajte nekaj sekund |
| Domena oddajanja | Gostitelj in senzor sta v istem segmentu L2; mDNS ne usmerja |
| Požarni zid Windows | Ob prvem zagonu sprejmite poziv programa Defender (UDP 5002, 5353, 319/320) |
| Gostitelj z več mrežnimi karticami | Pri prvem odkrivanju po zagonu senzor morda ne bo zaznan — povežite se z `--eth-host <ip-or-hostname>` |
| Ponovno skeniranje v grafičnem vmesniku | Odkrivanje poteka le, dokler je odprto okno za povezavo; uporabite gumb »Osveži« |</version>

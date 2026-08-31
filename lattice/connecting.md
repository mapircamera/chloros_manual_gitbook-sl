# Priključevanje kamer

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>Zavihek Kamere, preden je kaj priključeno</p></figcaption></figure>Chloros samodejno odkrije kamere LATTICE na povezavi — iz zavihka »Kamere« v grafičnem vmesniku, iz `chloros-cli lattice` ali iz Python SDK. Niz modela kamere določa vse nadaljnje korake: Chloros določi profil senzorja, razpored pasov in tovarniško kalibracijo na podlagi podatkov kamere `DeviceUserID` + `DeviceSerialNumber`, zato **ni treba ničesar nastavljati za posamezno kamero**.

Pred priključitvijo se prepričajte, da je omrežje gostitelja pravilno nastavljeno – naslavljanje link-local, jumbo okvirji ter, za matrice, nastavitve sprejemnega pomnilnika mrežne kartice. To je nastavitev na strani strojne opreme in je opisana v priročniku LATTICE: [**Nastavitev omrežja**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Povezovanje prek grafičnega vmesnika

Odprite zavihek **Kamere**v stranskem meniju Chloros (zaviheki za strojno opremo se prikažejo, ko se zaključi zagon strežnika) ali uporabite glavni meni →**Poveži s kamero**. Obe možnosti odpreta pogovorno okno**Poveži kamero(-e)**.

### Pogovorno okno »Poveži kamero(-e)«

Pogovorno okno takoj ob odprtju pregleda omrežje („Pregledovanje omrežja...“) in prikaže seznam vseh najdenih kamer. V vsaki vrstici so prikazani **model**kamere (npr. `LATT-M3M-L41-F550`),**serijska številka**in**IP-naslov**.

* **Kliknite vrstico, da jo izberete**(zelena označitev). Izberete lahko**več kamer** in jih povežete naenkrat — Chloros jih poveže zaporedno.
* Vrstice z oznako **»Povezano«** so že povezane in jih ni mogoče ponovno izbrati.
* Vrstice z oznako **»V nizu«** pripadajo trenutno povezanemu nizu kamer. Najprej prekinite povezavo niza, če želite to kamero uporabljati samostojno.
* **Poveži** — poveže izbrane kamere; gumb prikazuje število, npr. »Poveži (3)«, kadar je izbranih več kot ena kamera.
* **Ponovno skeniranje** — ponovno zažene iskanje.
* **Zapri** — zapre pogovorno okno.
* Če se iskanje konča brez rezultatov, pogovorno okno prikaže **»V omrežju ni bilo najdenih kamer«** — glejte [Odpravljanje težav](connecting.md#troubleshooting) spodaj.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>Pogovorno okno »Poveži kamero(-e)« — tukaj prikazano brez kamer v omrežju</p></figcaption></figure>### Prvo povezovanje: prenos kalibracijskega paketa

**Prvič**, ko se določena kamera poveže z računalnikom, Chloros prenese tovarniški kalibracijski paket kamere (\~3,8 MB) iz same kamere prek GigE. Medtem ko se to izvaja, pogovorno okno prikaže zeleni panel**»Prenos kalibracijskih podatkov iz kamere«**s serijsko vrstico napredka – pričakujte približno**70 sekund** na kamero. Paket se shrani v predpomnilnik na gostitelju, zato se pri poznejših priključitvah iste kamere prenos v celoti preskoči (in panel se nikoli ne prikaže).

### Analiza sistema

Gumb **Analiza sistema** v pogovornem oknu preveri gostiteljski računalnik in omrežje (med izvajanjem se prikaže napis »Analizira se...«) ter prikaže diagnostično poročilo:

* **Gostiteljski računalnik** — jedra procesorja in pomnilnik RAM; ime grafičnega procesorja in pomnilnik ali »GPU: ni zaznan«.
* **Omrežni vmesniki** — ime vsakega omrežnega vmesnika, hitrost povezave, MTU (z oznako »jumbo«, če je aktivna), stanje prenosa navzgor/navzdol in ali je vmesnik priključen na USB-vodilo.
* **Kamere**— serijska številka, model, IP in**na kateri mrežni kartici je priključena posamezna kamera**.
* **Zmogljivost** — trenutno število sličic na sekundo (fps) v primerjavi z idealnim za posamezno kamero glede na format pikslov, z zeleno vrstico »Potential: N× improvement possible«, kadar idealno število presega trenutno.
* **Opozorila in oštevilčena priporočila** — ali »Sistem deluje dobro za trenutno število kamer«, kadar ni ničesar za popraviti.

Program zaženite, kadar se odkrivanje ali pretakanje obnaša nepričakovano — prepozna večino težav na strani omrežnih kartic (napačna vrednost MTU, kamera na napačnem vmesniku, omejitve USB-adapterja), ne da bi zapustili pogovorno okno.

### Povezovanje niza

Če želite dve ali več kamer povezati v **sinhroniziran niz**, namesto tega uporabite čarovnika za povezovanje niza (**Poveži niz kamer**): vodnik vas popelje skozi izbiro glavne/podrejene kamere (predhodno izpolnjeno s sondo za ožičenje GPIO), izbiro načina prikazovanja (ločeni ali združeni kvadratki) ter nastavitve niza s projekcijo v realnem času dosegljivih sličic na sekundo (fps) in pasovne širine ožičenja, preden potrdite izbiro. Čarovnik in poteki dela z nizi so obravnavani v poglavju o nizih z več kamerami v tem priročniku; ekvivalent za CLI je »Postopek prvega priključitve kamere LATTICE« v [CLI Referenca](../reference/cli-reference.md).

## Povezovanje iz CLI in SDK

Za dostop do CLI in SDK je potreben plačljiv paket Chloros+ in prijava; to se izvaja na strani strežnika (`401 AUTH_REQUIRED`, če niste prijavljeni, `403 PLAN_UPGRADE_REQUIRED` na brezplačnem nivoju).

```bash
# List cameras on the network (vendor, model, serial, IP, MAC)
chloros-cli lattice info

# Single-camera smoke test: capture one frame (saves every applicable export type)
chloros-cli lattice capture -o output/

# Connect a synchronized array — same smart-prep flow as the GUI
chloros-cli lattice array-connect --serials 213800234,214000533
```

```python
import chloros_sdk

# Persistent live-camera session through the backend
with chloros_sdk.connect_camera("213800234") as cam:
    ...

# Array session (smart-prep: network probe, tier auto-pick, PTP, AE seeding, trigger config)
with chloros_sdk.connect_array(["213800234", "214000533"]) as array:
    ...
```

Celotni podpisi, možnosti in delovni tokovi zajemanja: [CLI Referenca](../reference/cli-reference.md) § `chloros-cli lattice`, [SDK Referenca](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Kako se kalibracija izvede ob povezavi

Vsaka kamera LATTICE ima tovarniški kalibracijski paket **vgrajena v kamero**, Chloros pa ob povezavi kamere preveri tudi oblak MAPIR:

| Situacija   | Kaj uporablja Chloros                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Spletno**|**Najnovejša kalibracija, objavljena za to serijsko številko** — kopija v oblaku prevlada nad kopijo v kameri. Kamera, ki jo je MAPIR ponovno kalibriral ali posodobil, se zato samodejno posodobi; ni potrebno nobeno ukrepanje uporabnika. |
| **Offline**|**Paket v kameri**, takšen, kakršen je. Delovni tokovi, ki potekajo popolnoma offline, še naprej delujejo; preprosto ne prevzamejo novejših kalibracij, dokler kamera ni vsaj enkrat online (ali dokler ni ponovno posodobljena s tovarniškimi nastavitvami).                                                  |

Ob zajemanju so dejansko uporabljeni koeficienti **zamrznjeni v metapodatkih XMP vsake slike**. Kasnejša posodobitev kalibracije nikoli tiho ne spremeni slik, ki ste jih že zajeli – ponovna obdelava starega posnetka uporablja koeficiente, vtisnjene v njegovih metapodatkih XMP, ne pa tistih, ki so danes najnovejši.

## Odpravljanje težav

* **»V omrežju ni najdenih kamer“**— preverite lokalno omrežno nastavitev v [Nastavitve omrežja](https://mapir.gitbook.io/lattice-camera/setup/network-setup): statični naslov gostiteljske mrežne kartice `169.254.x.x/16`, kamere na isti povezavi, DHCP/prehod ni predviden. Nato v pogovornem oknu za povezavo uporabite možnost**Analiza sistema**, da preverite, na kateri mrežni kartici je (ali ni) vidna posamezna kamera. Po vsaki spremembi ožičenja ali mrežne kartice izvedite**ponovno iskanje**.
* **Sistem, ki je prej deloval, se ne želi povezati** (vrata panelov z `FRAMES WILL DROP` / `Reduce ROI to enable`) — posodobitev gonilnika omrežne kartice je tiho ponastavila nastavitve sprejemnega obroča. Ponovno jih uporabite ali zaženite `chloros-cli lattice network --fix` iz terminala z upraviteljskimi pravicami; glejte [Nastavitev omrežja](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **Kamera prikazuje »In Array«** — pripada povezani seji niza. Odklopite niz, če želite kamero uporabljati samostojno.

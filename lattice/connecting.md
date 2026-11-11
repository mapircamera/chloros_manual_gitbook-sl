# Priključevanje kamer

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>Zavihek »Kamere«, preden je kaj priključeno</p></figcaption></figure>

Chloros
samodejno odkrije kamere LATTICE na povezavi — iz zavihka »Kamere« v grafičnem vmesniku, iz `chloros-cli lattice` ali iz Python
SDK
. Niz modela kamere določa vse nadaljnje postopke: Chloros
iz kamere `DeviceUserID` + `DeviceSerialNumber` določi profil senzorja, razpored pasov in tovarniško kalibracijo, zato **ni treba ničesar nastavljati za posamezno kamero**.

Pred priključitvijo se prepričajte, da je omrežje gostitelja pravilno nastavljeno — naslavljanje link-local, jumbo okvirji ter, za matrice, nastavitve sprejemnega pomnilnika mrežne kartice. To je nastavitev na strani strojne opreme in je opisana v priročniku LATTICE: [**Nastavitev omrežja**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Povezovanje prek grafičnega uporabniškega vmesnika

Odprite zavihek **Kamere** v stranskem meniju Chloros
a (zaviheki za strojno opremo se prikažejo, ko se zaključi zagon strežnika) ali uporabite glavni meni → **Poveži s kamero**. Obe možnosti odpreta pogovorno okno**Poveži kamero(-e)**.

### Pogovorno okno **Poveži kamero(-e)**Pogovorno okno takoj ob odprtju pregleda omrežje („Pregled omrežja...“) in prikaže seznam vseh najdenih kamer. Vsaka vrstica prikazuje**model**kamere (npr. `LATT-M3M-L41-F550`),**serijsko številko**in**IP-naslov**.

* **Kliknite vrstico, da jo izberete**(zelena označitev). Izberete lahko**več kamer** in jih povežete naenkrat — Chloros
jih poveže zaporedno.
* Vrstice z oznako **»Povezano«** so že povezane in jih ni mogoče ponovno izbrati.
* Vrstice z oznako **»V nizu«** pripadajo trenutno povezanemu nizu kamer. Najprej prekinite povezavo z nizom, če želite to kamero uporabljati samostojno.
* **Poveži** — poveže izbrane kamere; gumb prikazuje število, npr. »Poveži (3)«, če je izbranih več kamer.
* **Ponovno skeniranje** — ponovno zažene iskanje.
* **Zapri** — zapre pogovorno okno.
* Če se iskanje konča brez rezultatov, se v pogovornem oknu prikaže **»V omrežju ni najdenih kamer«** — glejte [Odpravljanje težav](connecting.md#troubleshooting) spodaj.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>Pogovorno okno »Poveži kamero(-e)« — tukaj prikazano brez kamer v omrežju</p></figcaption></figure>

### Prvo povezovanje: prenos kalibracijskega paketa

Ko se določena kamera **prvič** poveže z računalnikom, Chloros
prek GigE prenese tovarniški kalibracijski paket kamere (\~3,8 MB) neposredno iz kamere. Medtem ko se to izvaja, pogovorno okno prikaže zeleni panel **»Prenos kalibracijskih podatkov iz kamere«**s serijsko vrstico napredka — pričakujte približno**70 sekund** na kamero. Pri več kot šestih kamerah se vrstice napredka pomikajo znotraj panela. Paket se shrani v predpomnilnik na gostitelju, zato se pri poznejših priključitvah iste kamere prenos v celoti preskoči (in okno se nikoli ne prikaže).

### Analiza sistema

Gumb **Analiza sistema** v pogovornem oknu preveri gostiteljski računalnik in omrežje (med izvajanjem se prikaže napis »Analizira se...«) ter prikaže diagnostično poročilo:

* **Gostiteljski računalnik** — jedra procesorja in pomnilnik RAM; ime grafične kartice in njen pomnilnik ali »GPU: ni zaznano«.
* **Omrežni vmesniki** — ime vsakega omrežnega vmesnika, hitrost povezave, MTU (z oznako »jumbo«, če je aktivna), stanje prenosa (up/down) in ali je vmesnik priključen na USB-vodilo.
* **Kamere**— serijska številka, model, IP in**na katerem omrežnem vmesniku je priključena vsaka kamera**.
* **Zmogljivost** — trenutno število sličic na sekundo (fps) v primerjavi z idealnim za vsako kamero glede na format pikslov, z zeleno vrstico »Potential: N× improvement possible«, kadar idealno število presega trenutno.
* **Opozorila in oštevilčena priporočila** — ali »Sistem deluje dobro za trenutno število kamer«, kadar ni ničesar za popraviti.

Zagnajte ga, kadar se odkrivanje ali pretakanje obnaša nepričakovano — prepozna večino težav na strani omrežnih kartic (napačna MTU, kamera na napačnem vmesniku, omejitve USB-adapterja), ne da bi zapustili pogovorno okno.

### Povezovanje niza

Če želite povezati dve ali več kamer kot **sinhroniziran niz**, namesto tega uporabite čarovnika za povezovanje niza (**Poveži niz kamer**): ta vas vodi skozi izbiro glavne/podrejene kamere (vnaprej izpolnjeno s preizkusom sinhronizacijskega kabla — vsaka priključena kamera lahko postane glavna, kamera RGB
`-FRGB` pa je vnaprej izbrana, če je priključena s kablom), izbiro načina prikazovanja (ločene ali združene ploščice) ter prizor z nastavitvami niza s projekcijo v realnem času dosegljivih sličic na sekundo (fps) in pasovne širine povezave, preden potrdite izbiro. Čarovnik in delovni tokovi za nize so obravnavani v poglavju o nizih več kamer v tem priročniku; ustreznik v CLI
-ju je »LATTICE Camera First-Connect Workflow« v [CLI
Reference](../reference/cli-reference.md).

## Povezovanje prek CLI
in SDK
Dostop prek

CLI
in SDK
zahteva plačljivo stopnjo Chloros
+ in prijavo; to se uveljavlja na strani strežnika (`401 AUTH_REQUIRED`, če niste prijavljeni, `403 PLAN_UPGRADE_REQUIRED` na brezplačni stopnji).

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

Celotni podpisi, možnosti in delovni tokovi zajemanja: [Referenca CLI
](../reference/cli-reference.md) § `chloros-cli lattice`, [Referenca SDK
](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Kako se kalibracija izvede ob povezavi

Vsaka kamera LATTICE ima svoj tovarniški kalibracijski paket **vgrajena v kamero**, poleg tega pa Chloros
ob povezavi kamere preveri tudi oblak MAPIR
:

| Situacija   | Kaj uporabi Chloros
                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Spletno**|**Najnovejša kalibracija, objavljena za to serijsko številko** — kopija v oblaku prevlada nad kopijo na kameri. Kamera, ki jo je MAPIR
ponovno kalibriral ali posodobil, se zato samodejno posodobi; ni potrebno nobeno ukrepanje uporabnika. |
| **Offline**|**Paket v kameri**, tak kot je. Popolnoma offline delovni tokovi še naprej delujejo; preprosto ne prevzamejo novejših kalibracij, dokler kamera ni enkrat spet online (ali dokler ni ponovno posodobljena na tovarniške nastavitve).                                                  |

Ob zajemanju so dejansko uporabljeni koeficienti **vpisani v XMP-metapodatke vsake slike**. Kasnejša posodobitev kalibracije nikoli tiho ne spremeni že zajetih slik — pri ponovni obdelavi starega posnetka se uporabijo koeficienti, vpisani v njegovih XMP-metapodatkih, ne pa tisti, ki so danes najnovejši.

## Odpravljanje težav

* **»V omrežju ni najdenih kamer“**— preverite nastavitve za lokalno povezavo v [Nastavitve omrežja](https://mapir.gitbook.io/lattice-camera/setup/network-setup): statični naslov omrežne kartice gostitelja `169.254.x.x/16`, kamere na isti povezavi, DHCP/prehod ni predviden. Nato v pogovornem oknu za povezavo uporabite**Analiza sistema**, da preverite, na kateri mrežni kartici je (ali ni) vidna posamezna kamera. Po vsaki spremembi ožičenja ali mrežne kartice izvedite**ponovno iskanje**.
* **Naprava, ki je prej delovala, se ne želi povezati** (vrata plošče z `FRAMES WILL DROP` / `Reduce ROI to enable`) — posodobitev gonilnika mrežne kartice je tiho ponastavila nastavitve sprejemnega obroča. Ponovno jih uporabite ali zaženite `chloros-cli lattice network --fix` iz terminala z upraviteljskimi pravicami; glejte [Nastavitev omrežja](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **Kamera prikazuje »In Array«** — pripada povezani seji niza. Odklopite niz, če želite kamero uporabljati samostojno.

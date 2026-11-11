---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Prenos

Prenesite najnovejšo različico programa Chloros
, da se lotite obdelave multispektralnih slik.

### Sistemski zahtevi

#### Windows


| Zahteva          | Minimalna                                              | Priporočena                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Operacijski sistem** | Windows
10 (64-bitni)                                  | Windows
11 (64-bitni)                                  |
| **Procesor**        | Intel Core i5 ali enakovreden                          | Intel Core i7 ali boljši                              |
| **Pomnilnik (RAM)**     | 8 GB                                                  | 16 GB ali več                                         |
| **Grafična kartica**    | združljiva z DirectX 11                                | grafična kartica NVIDIA z 4 GB ali več VRAM                            |
| **Prostor za shranjevanje**          | 6 GB prostega prostora                                       | SSD z 10 GB ali več prostega prostora                            |
| **Zaslon**          | 1920x1080                                            | 2560x1440 ali več                                  |
| **Internet**         | Potreben za [neobvezno] aktivacijo licence Chloros
+ | Potreben za [neobvezno] aktivacijo licence Chloros
+ |

#### Linux
amd64 (x86_64)

| Zahteve       | Minimalne                    | Priporočene               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribucija**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Procesor**     | x86\_64 (Intel/AMD)        | Intel Core i7 ali boljši   |
| **Pomnilnik (RAM)**  | 8 GB                        | 16 GB ali več              |
| **Grafična kartica** | Ni potrebna (obdelava v procesorju)      | NVIDIA GPU z 4 GB+ VRAM |
| **Prostor za shranjevanje**       | 2 GB prostega prostora             | SSD z 10 GB+ prostega prostora       |
| **Python**        | Python
3.7+ (za SDK
)      | Python
3.10+              |

#### Linux
arm64 (NVIDIA Jetson)

| Zahteve      | Minimalne                      | Priporočene                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platforma**     | NVIDIA Jetson z JetPack 6 | Jetson Orin NX 16 GB ali AGX Orin |
| **Pomnilnik (RAM)** | 8 GB (skupni za grafično kartico in procesor)         | 16 GB+ skupnega pomnilnika                    |
| **Shranjevanje**      | 2 GB prostega prostora               | NVMe SSD z 10 GB+ prostega prostora        |
| **Python**       | Python
3.7+ (za SDK
)        | Python
3.10+                    |

{% hint style="info" %}
**Pospeševanje z grafičnim procesorjem (GPU)**: Uporabniki Chloros
+ z grafičnimi procesorji NVIDIA lahko uporabijo pospeševanje CUDA za znatno hitrejšo obdelavo. To deluje z namiznimi grafičnimi procesorji NVIDIA v sistemih Windows
in Linux
amd64 ter na NVIDIA Jetson z JetPack 6. Uporabniki Chloros
+ pridobijo tudi večnitno obdelavo za največjo hitrost.
{% endhint %}

***

## Prenos Chloros


### Najnovejša stabilna različica: različica 1.3.0 (11. november 2026)

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Prenos programa Chloros za sistem Windows (.exe)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Prenos programa Chloros za sistem Linux amd64 (.deb)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Prenos programa Chloros za Linux arm64 / Jetson (.deb)</a>


#### Namestitveni program za Windows
(GUI + CLI
+ Backend)

* **Vrsta datoteke**: .exe (namestitveni program za Windows
)

**Koraki namestitve:**

1. Prenesite zgornjo datoteko .exe
2. Dvakrat kliknite namestitveni program, da začnete namestitev
3. Sledite navodilom namestitvenega čarovnika
4. Izberite namestitveni imenik (privzeto: `C:\Program Files\Chloros\`)
5. Na strani **Izberite dodatne naloge**pustite obe možnosti**Sistemske nastavitve** označeni (priporočeno – glejte spodaj)
6. Zaključite namestitev in zaženite Chloros
ali Chloros
CLI
7. Prijavite se s svojim [MAPIR
Cloud Chloros
+ računom](https://cloud.mapir.camera/pricing) (ali nadaljujte z brezplačno različico)

**Možnosti sistemskih nastavitev** (obe sta privzeto označeni):

| Možnost, kot je navedena v namestitvenem programu | Kaj počne | Izključitev tihe namestitve |
| --- | --- | --- |
| **Omogoči dolge poti v sistemu Windows** (omogoča, da so mape projektov globlje od 260 znakov; priporočeno) | Nastavi sistemsko nastavitev `LongPathsEnabled` v Windows
u na vrednost 1. Windows
je privzeto izklopljen, zato nobena pot do datoteke ne sme biti daljša od 260 znakov; izdelki Chloros
se nahajajo do približno 130 znakov pod projektno mapo, zato se pri izklopljenih dolgih poteh globlja projektna mapa zavrne. Velja za programe, ki se zaženejo po namestitvi. Po odstranitvi programa ostane vklopljena, saj se nanjo morda zanašajo drugi programi. | `/MERGETASKS="!longpaths"` |
| **Izključi Chloros
ov backend iz skeniranja s programom Microsoft Defender** (precej hitrejše zajemanje s kamero; priporočeno) | Dodaja izključitev procesa v programu Microsoft Defender za `chloros-backend.exe` v namestitveni mapi, tako da zaščita v realnem času ne skenira vsake datoteke, ki jo backend zapiše, ne glede na to, kam jo zapiše. Pri visokih hitrostih zajemanja skeniranje porabi približno 30 % hitrosti zajemanja (77,7 proti 52,7 zajemov na sekundo pri neprekinjenem zajemanju z 9 kamerami). Noben drug program ni izključen, izključitev pa se ob odstranitvi programa odstrani. Če Defender ni nameščen ali je blokiran s pravilnikom, ta korak tiho ne uspe in namestitev se nadaljuje. | `/MERGETASKS="!defenderexclusion"` |

Pri tiho namestitvi (`/SILENT` ali `/VERYSILENT`) se uporabita obe možnosti, razen če se odločite za izključitev s prikazano vrednostjo `/MERGETASKS`.

{% hint style="success" %}
Namestitveni program enkrat doda mapo CLI
(`C:\Program Files\Chloros\cli\`) v vaš sistem `PATH` — ponovna namestitev ali nadgradnja ne doda več podvojenega vnosa in odstrani podvojene vnose, ki so jih dodali prejšnji namestitveni programi. Po namestitvi odprite nov terminal, da boste lahko uporabljali `chloros-cli`. Namestitveni program doda tudi dve pravili požarnega zidu Windows
a za backend v vsakem omrežnem profilu: **Chloros
Backend (UDP-In)**, za tokove kamer, PTP, mDNS in podatkovni tok DAQ-E, ter**Chloros
Backend (TCP-In)**, omejeno na lokalno podomrežje, za posodobitev firmware-a DAQ-E, pri kateri se senzor ponovno poveže s tem računalnikom, da zbira sliko. Obe se ob odstranitvi programa izbrišeta.
{% endhint %}

#### Linux
amd64 (paket .deb — CLI
+ backend)

* **Vrsta datoteke**: .deb (paket Debian/Ubuntu)
* **Arhitektura**: x86\_64 (amd64)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

#### Linux
arm64 — NVIDIA Jetson (paket .deb — CLI
+ Backend)

* **Vrsta datoteke**: .deb (JetPack 6)
* **Arhitektura**: aarch64 (arm64)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

Uporabite ime datoteke, ki ste jo dejansko prenesli, in ohranite `./`, da bo `apt` namestil to datoteko, namesto da bi iskal paket po imenu. Namestite z `apt install`, ne pa s `dpkg -i`: apt namesti tudi zahteve za Python
SDK
, ki jih paket priporoča, namesto da bi jih zahteval, `dpkg -i` pa jih preskoči. Od različice 1.3.0 naprej se backend in CLI
namestita razpakirana pod `/usr/lib/chloros/`, zato se zaženeta, ne da bi se najprej razpakirala.

Paket Jetson uporablja CUDA-različico PyTorcha iz JetPack 6, zato obdelava, vključno z vgrajenim Texture Aware debayerjem, poteka na grafičnem procesorju (GPU) naprave Jetson. Uporablja knjižnici CUDA in cuDNN, ki ju namesti JetPack 6 (`sudo apt install nvidia-jetpack` na sliki, na kateri ju ni).

Za podrobna navodila za nastavitev glejte [Namestitev v sistemu Linux
](linux/linux-installation.md), za navodila, specifična za Jetson, pa [Vodnik za NVIDIA Jetson](linux/nvidia-jetson-guide.md).

#### Python
SDK
(vse platforme)

Vsak namestitveni program vključuje ustrezen paket `chloros_sdk`, tako da se različica SDK
vedno ujema z nameščenim GUI/CLI
/backendom. V sistemu Windows
namestitveni program samodejno namesti paket v sistemski Python
. V sistemu Linux
namestitveni program `.deb` namesti SDK
za sam sistem `python3` (v `/usr/lib/python3/dist-packages`), tako da `import chloros_sdk` deluje takoj po namestitvi, brez pip in brez omrežja. Le virtualno okolje (virtualenv), ki ne vidi sistemskih paketov, potrebuje vgrajeno kolesce iz paketa:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Za gostitelje, ki uporabljajo le pip (na katerih ni nameščen paket Chloros
; SDK
v tem primeru poganja backend na drugem računalniku), je SDK
na voljo tudi na PyPI:

```bash
pip install chloros-sdk
```

Za dokumentacijo glejte [API
: Python
SDK
](api-python-sdk.md) in [SDK
Reference](reference/sdk-reference.md).

{% hint style="info" %}
**Uporabniki Linux
a**: Paket `.deb` namesti CLI
, backend in Python
SDK
. Za Linux
ni grafičnega vmesnika (GUI) — vsa interakcija poteka prek CLI
ali SDK
.
{% endhint %}

***

## Dodatni viri

### Python
SDK


Za razvijalce in avtomatizirane delovne tokove je Chloros
Python
SDK
priložen vsakemu namestitvenemu programu: namestitveni program za Windows
ga namesti v vaš Python
, namestitveni program za Linux
`.deb` pa ga namesti za sistem `python3`. Na računalniku brez paketa Chloros
(ki upravlja z backendom na drugem računalniku) ga namestite iz PyPI:

```bash
pip install chloros-sdk
```

**Dokumentacija**: [API
: Python
SDK
](api-python-sdk.md)

**Zahteve**: backend Chloros
, ki ga je treba upravljati (namestitveni program za Windows
ali paket `.deb` za Linux
ga namesti), ter prijava v Chloros
+ na plačljivi stopnji

***

## Kaj je vključeno

### Namestitveni program za Windows


* ✅ **GUI Chloros** – grafični vmesnik s polno funkcionalnostjo
* ✅ **CLI
Chloros
** – vmesnik ukazne vrstice (zahteva licenco Chloros
+)
* ✅ **Backend Chloros** – procesni motor
* ✅ **Python
SDK
** – namesti se v vaš Python
, če je ta prisoten (zahteva licenco Chloros
+)
* ✅ **Profili kamer** – vnaprej konfigurirane predloge kamer MAPIR


### Paket .deb za Linux


* ✅ **Chloros
CLI
** – vmesnik ukazne vrstice (zahteva licenco Chloros
+)
* ✅ **Chloros
Backend** – procesni motor
* ✅ **Python
SDK
** – nameščen za sistem `python3` (zahteva licenco Chloros
+)
* ✅ **Profili kamer** – vnaprej konfigurirane predloge kamer MAPIR

* ❌ Brez grafičnega vmesnika (GUI) — Linux
je brez grafičnega vmesnika, na voljo je le CLI
/SDK


### Python
SDK
(PyPI, za računalnike brez paketa Chloros
)

* ✅ **Chloros
SDK
** – Python
API
(zahteva licenco Chloros
+)

***

## Nadgradnja na Chloros
+

Odkleni napredne funkcije z naročnino na Chloros
+:

* 🚀 **Večnitna obdelava** – vzporedna obdelava slik
* ⚡ **Pospeševanje z GPU (CUDA)** – Izkoristite moč grafičnih procesorjev NVIDIA
* 💻 **Dostop prek CLI** – Avtomatizirajte z orodji za ukazno vrstico
* 🐍 **Python
SDK
** – Programski dostop do API
-ja
* 📱 **Več naprav** – Uporaba na 2–10+ napravah (odvisno od paketa)
* **🐻 Napredna metoda debayeringa, ki upošteva teksturo** – visokokakovosten debayering, ki upošteva robove, v kombinaciji z modelom za odstranjevanje šuma na podlagi umetne inteligence in strojnega učenja, ki odstrani skoraj ves šum pri debayeringu.
* 🧮 **Prilagojene formule** – Ustvarjanje prilagojenih multispektralnih indeksov

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Oglejte si načrte in cene za Chloros+</a></p>***

## Pomoč pri namestitvi

### Odpravljanje težav

**Namestitev se ne uspešno zaključi z naslednjim sporočilom o napaki:**

* Preverite, ali imate skrbniške pravice
* Začasno onemogočite protivirusno programsko opremo
* Preverite, ali izpolnjujete minimalne sistemske zahteve

**Aplikacija se ne zažene (sistem Windows

):**

* Preverite, ali je nameščen Windows

10/11 (64-bitni)
* Posodobite grafične gonilnike
* V pregledovalniku dogodkov Windows

preverite podrobnosti o napaki
* Obrnite se na podporo s dnevniki napak

**CLI

se ne zažene (Linux

):**

* Preverite, ali je paket `.deb` pravilno nameščen: `dpkg -l | grep chloros`
* Preverite dovoljenja: `sudo chmod +x /usr/bin/chloros-cli`
* Zaženite diagnostiko: `chloros-cli selftest`
* Preverite, ali manjkajo knjižnice: `ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"` (od različice 1.3.0 naprej je `/usr/lib/chloros/chloros-backend` majhen skript za zagon; sam program se nahaja v `/usr/lib/chloros/backend/`)

**Težave z aktivacijo licence:**

* Preverite, ali je internetna povezava aktivna
* Preverite poverilnice na [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Preverite, ali požarni zid ne blokira Chloros

* Podrobna navodila najdete na [Chloros

+ Prijava](chloros+-login.md)

### Pomoč

Potrebujete pomoč pri namestitvi ali nastavitvi?

* 📧 **E-pošta**: info@mapir.camera
* 🌐 **Spletna stran**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentacija**: [Prvi koraki](./)
* ❓ **Pogosta vprašanja**: [Pogosta vprašanja](faq.md)***

## Posodobitve programske opreme

Chloros

preveri, ali so na voljo posodobitve, obvesti vas, ko je na voljo nova različica, in prikaže povezavo do te strani za prenos — posodobitev izvedete z zagonom novega podpisanega namestitvenega programa. Vaše nastavitve, projekti in prijavni podatki ostanejo ohranjeni tudi po posodobitvi. V sistemu Linux

in na Jetsonu program `chloros-cli update` preveri, ali je na voljo novejša različica, ter ponudi prenos in namestitev ustrezne različice `.deb` (ta ukaz je na voljo le v sistemu Linux

).

***

## Seznam sprememb**Različica 1.3.0 (najnovejša, 11. november 2026)**— celoten seznam najdete v razdelku**Novosti v Chloros

u 1.3.0** na strani [Začetek](./).**Različica 1.2.0**— glejte**Novosti v Chloros

u 1.2.0** na strani [Začetek](./).

<details>

<summary>Različica 1.0.5</summary>

**Datum izdaje: 10. februar 2026**

**Nove funkcije*** **Metoda debayeringa, ki upošteva teksturo \[Samo Chloros

+] –** Texture Aware uporablja visokokakovosten debayering, ki upošteva robove, v kombinaciji z modelom za odstranjevanje šuma na podlagi umetne inteligence/strojnega učenja (AI/ML), ki odstrani skoraj ves šum pri debayeringu.
* **Podpora za kalibracijske tarče T4P*** **Hitrejša obdelava s pomočjo grafičnega procesorja (GPU) v Chloros

+, boljše upravljanje pomnilnika**

**Popravki napak*** Povsem nov uporabniški vmesnik (GUI), ki naj bi zdaj deloval na vseh računalnikih s sistemom Windows

.

</details>

<details>

<summary>Različica 1.0.4</summary>

**Datum izdaje: 5. januar 2026**

**Nove funkcije*** **Preklop med sliko in metapodatki**: V brskalniku datotek je bil dodan preklopnik za prikaz metapodatkov izbrane slike v tabeli namesto v mreži slik
* **Drsnik za povečavo mreže slik**: Nov drsnik v uporabniškem vmesniku za prilagajanje velikosti sličic (podpira tudi kombinacijo tipk CTRL + kolesce miške)
* **Gumbi za izvoz mreže slik**: Gumbi v zgornji vrstici za preklop sličic iz formata JPG na obdelane izvoze (Targets, Reflectance, Index, LUT)
* **Zavihek »Zemljevid«**: Nov interaktiven 2D-zemljevid, ki prikazuje oznake GPS-lokacij na slikah
  * Podpira Google Maps in kartografske ploščice ESRI (samodejno izbere najboljšo storitev ploščic glede na razpoložljivost stopnje povečave)
  * Predogled sličic ob preletu z miško nad označbami na zemljevidu

**Popravki napak*** Izboljšana podpora za namestitev programa Chloros na računalnikih, ki niso v angleškem jeziku

</details>

<details>

<summary>Različica 1.0.3</summary>

**Datum izdaje: 20. december 2025**

**Nove funkcije*** Prvi zagon

**Izboljšave*** Prvi zagon

**Popravki napak*** Prvi zagon

**Znane težave*** Prvi zagon

</details>

***

## Licenčna pogodba**Lastniška programska oprema** – Avtorske pravice (c) 2026 MAPIR Inc.

Neodobrena uporaba, distribucija ali spreminjanje je prepovedano.

**Brezplačna različica**: Na voljo za osebno in komercialno uporabo z omejenimi funkcijami**Chloros+**: Licenca na podlagi naročnine za napredne funkcije in komercialno uporabo

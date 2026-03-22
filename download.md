---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Prenos

Prenesite najnovejšo različico programa Chloros in se lotite obdelave multispektralnih slik.

### Sistemski zahtevi

#### Windows

| Zahteva          | Minimalna                                              | Priporočena                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Operacijski sistem** | Windows 10 (64-bitni)                                  | Windows 11 (64-bitni)                                  |
| **Procesor**        | Intel Core i5 ali enakovreden                          | Intel Core i7 ali boljši                              |
| **Pomnilnik (RAM)**     | 8 GB                                                  | 16 GB ali več                                         |
| **Grafična kartica**    | Združljiva z DirectX 11                                | NVIDIA GPU z 4 GB+ VRAM                            |
| **Shranjevanje**          | 6 GB prostega prostora                                       | SSD z 10 GB+ prostega prostora                            |
| **Zaslon**          | 1920x1080                                            | 2560x1440 ali več                                  |
| **Internet**         | Potreben za \[izbirno] aktivacijo licence Chloros+ | Potreben za \[izbirno] aktivacijo licence Chloros+ |

#### Linux amd64 (x86\_64)

| Zahteva       | Minimalna                    | Priporočena               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribucija**  | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+             |
| **Procesor**     | x86\_64 (Intel/AMD)        | Intel Core i7 ali boljši   |
| **Pomnilnik (RAM)**  | 8 GB                        | 16 GB ali več              |
| **Grafična kartica** | Ni potrebna (obdelava CPU)      | NVIDIA GPU z 4 GB+ VRAM |
| **Shranjevanje**       | 2 GB prostega prostora             | SSD z 10 GB+ prostega prostora       |
| **Python**        | Python 3.7+ (za SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Zahteva      | Minimalno                      | Priporočeno                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platforma**     | NVIDIA Jetson z JetPack 6 | Jetson Orin NX 16 GB ali AGX Orin |
| **Pomnilnik (RAM)** | 8 GB (skupni GPU/CPU)         | 16 GB+ skupnega                    |
| **Shranjevanje**      | 2 GB prostega prostora               | NVMe SSD z 10 GB+ prostega        |
| **Python**       | Python 3.7+ (za SDK)        | Python 3.10+                    |

{% hint style="info" %}
**Pospeševanje s GPU**: Uporabniki Chloros+ z grafičnimi karticami NVIDIA lahko uporabijo pospeševanje CUDA za bistveno hitrejše obdelovanje. To deluje tako na Windows (namizne grafične kartice) kot na Linux (namizne grafične kartice in NVIDIA Jetson). Uporabniki Chloros+ pridobijo tudi večnitno obdelavo za maksimalno hitrost.
{% endhint %}

***

## Prenesite Chloros

### Najnovejša stabilna različica (23. marec 2026): Različica 1.1.0

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Prenos Chloros za Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Prenos Chloros za Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Prenesite Chloros za Linux arm64 / Jetson (.deb)</a>

#### Namestitveni program Windows (GUI + CLI + Backend)

* **Vrsta datoteke**: .exe (namestitveni program Windows)**Koraki namestitve:**

1. Prenesite zgornjo datoteko .exe
2. Dvakrat kliknite namestitveni program, da začnete namestitev
3. Sledite navodilom namestitvenega čarovnika
4. Izberite namestitveni imenik (privzeto: `C:\Program Files\[USER]\Chloros\`)
5. Zaključite namestitev in zaženite Chloros ali Chloros CLI
6. Prijavite se s svojim [MAPIR Cloud Chloros+ računom](https://cloud.mapir.camera/pricing) (ali nadaljujte z brezplačno različico)

{% hint style="success" %}
Namestitveni program samodejno doda `chloros-cli` v sistemsko pot PATH za dostop prek ukazne vrstice.
{% endhint %}

#### Linux amd64 (paket .deb — CLI + Backend)

* **Vrsta datoteke**: .deb (paket Debian/Ubuntu)
* **Arhitektura**: x86\_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (.deb paket — CLI + Backend)

* **Vrsta datoteke**: .deb (JetPack 6)
* **Arhitektura**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Za podrobna navodila za namestitev glejte [Namestitev Linux](linux/linux-installation.md), za navodila, specifična za Jetson, pa [Vodnik za NVIDIA Jetson](linux/nvidia-jetson-guide.md).

#### Python SDK (vse platforme)

```bash
pip install chloros-sdk
```

Za dokumentacijo glejte [API : Python SDK](api-python-sdk.md).

{% hint style="info" %}
**Uporabniki Linux**: Paket `.deb` namesti CLI in backend. Python SDK se namesti ločeno prek pip. Za Linux ni grafičnega vmesnika – vsa interakcija poteka prek CLI ali SDK.
{% endhint %}

***

## Dodatni viri

### Python SDK

Za razvijalce in avtomatizirane delovne tokove namestite Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentacija**: [API: Python SDK](api-python-sdk.md)**Zahteve**: Chloros mora biti nameščen (namestitveni program Windows ali paket Linux `.deb`), potrebna je prijava z licenco Chloros+***

## Vsebina

### Namestitveni program Windows

* ✅ **Chloros GUI** – grafični vmesnik s polno funkcionalnostjo
* ✅ **Chloros CLI** – vmesnik ukazne vrstice (zahteva licenco Chloros+)
* ✅ **Chloros Backend** – procesni motor
* ✅ **Profil kamere** - Vnaprej konfigurirane predloge kamer MAPIR

### Paket Linux .deb

* ✅ **Chloros CLI** - Vmesnik ukazne vrstice (zahteva licenco Chloros+)
* ✅ **Chloros Backend** – Obdelovalni motor
* ✅ **Profil kamere** – Vnaprej konfigurirane predloge kamer MAPIR
* ❌ Brez grafičnega vmesnika — Linux je brez grafičnega vmesnika, samo CLI/SDK

### Python SDK (pip, vse platforme)

* ✅ **Chloros SDK** - Python API (zahteva licenco Chloros+)***

## Nadgradnja na Chloros+

Odkleni napredne funkcije z naročnino Chloros+:

* 🚀 **Večnitno obdelovanje** – vzporedna obdelava slik
* ⚡ **Pospešitev z GPU (CUDA)** – izkoristi moč grafičnih procesorjev NVIDIA
* 💻 **Dostop do CLI** – Avtomatizirajte z orodji za ukazno vrstico
* 🐍 **Python SDK** – Programski dostop do API
* 📱 **Več naprav** – Uporaba na 2–10+ napravah (odvisno od paketa)
* **🐻 Napredna metoda debayeringa z upoštevanjem teksture** – visokokakovosten debayering z upoštevanjem robov v kombinaciji z modelom za odstranjevanje šuma AI/ML, ki odstrani skoraj ves šum pri debayeringu.
* 🧮 **Prilagojene formule** – Ustvarjanje prilagojenih multispektralnih indeksov

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Oglejte si načrte in cene Chloros+</a></p>***

## Pomoč pri namestitvi

### Odpravljanje težav

**Namestitev se ne uspešno zaključi z naslednjim sporočilom o napaki:**

* Preverite, ali imate skrbniške pravice
* Začasno onemogočite protivirusno programsko opremo
* Preverite, ali izpolnjujete minimalne sistemske zahteve

**Aplikacija se ne zažene (Windows):**

* Preverite, ali je nameščen Windows 10/11 (64-bitni)
* Posodobite grafične gonilnike
* Preverite Windows pregledovalnik dogodkov za podrobnosti o napaki
* Obrnite se na podporo s dnevniki napak

**CLI se ne zažene (Linux):**

* Preverite, ali je paket `.deb` pravilno nameščen: `dpkg -l | grep chloros`
* Preverite dovoljenja: `sudo chmod +x /usr/bin/chloros-cli`
* Zaženite diagnostiko: `chloros-cli selftest`
* Preverite, ali manjkajo knjižnice: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Težave z aktivacijo licence:**

* Preverite, ali je internetna povezava aktivna
* Preverite poverilnice na [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Preverite, ali požarni zid ne blokira Chloros
* Podrobna navodila najdete na [Chloros+ Prijava](chloros+-login.md)

### Pomoč

Potrebujete pomoč pri namestitvi ali nastavitvi?

* 📧 **E-pošta**: info@mapir.camera
* 🌐 **Spletna stran**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentacija**: [Začetek](./)
* ❓ **Pogosta vprašanja**: [Pogosta vprašanja](faq.md)***

## Seznam sprememb

<details>

<summary>Različica 1.1.0 (najnovejša)</summary>

**Datum izdaje: marec 2026**

**Nove funkcije*** **Podpora za Linux** — Native CLI in SDK za Linux amd64 (x86\_64) in arm64 (NVIDIA Jetson JetPack 6). Namestite prek paketov `.deb`.
* **Podpora za NVIDIA Jetson** — Optimizirano obdelovanje za naprave Jetson Nano, Orin Nano, Orin NX in AGX Orin.
* **Dinamična prilagoditev računalniških zmogljivosti** — Samodejno zaznavanje strojne opreme in optimizacija strategije obdelave. Chloros se prilagaja vaši strojni opremi, od Jetson Nano do delovne postaje z več grafičnimi procesorji.
* **4-nitni procesni potek** — Sočasne niti za zaznavanje, kalibracijo, obdelavo in izvoz z dinamično dodelitvijo pomnilnika grafičnega procesorja.
* **Nove ukaze CLI** — `selftest` (sistemska diagnostika) in `update` (upravljanje posodobitev Linux).
* **Nove zastavice procesa CLI** — `--debayer` (standardno/upošteva teksturo), `--indices` (določi indekse), `--target` (za hitrejše zaznavanje najprej poišči ciljni podmapo).
* **Nove postavke v meniju GUI** — Dodaj datoteke, Dodaj mapo in Začni/ustavi obdelavo so zdaj dostopne v spustnem meniju glavnega menija.**Izboljšave**

* Samodejno zaznavanje večplatformnega ozadja (poti Windows in Linux)
* Izboljšano SDK `get_status()` s sledenjem napredka po posameznih niti
* Nove izjeme SDK: `ChlorosConfigurationError`, `ChlorosAuthenticationError`
* Upravljanje toplote in prilagodljivo omejevanje za NVIDIA Jetson
* Avtomatsko upravljanje pomnilnika z OOM preklopom na obdelavo GPU v obliki ploščic

</details>

<details>

<summary>Različica 1.0.5</summary>

**Datum izdaje: 10. februar 2026**

**Nove funkcije*** **Metoda Texture Aware Debayer \[Samo Chloros+] -** Texture Aware uporablja visokokakovostni debayer, ki upošteva robove, v kombinaciji z modelom za odstranjevanje šuma AI/ML, ki odstrani skoraj ves šum debayeringa.
* **Podpora za kalibracijske tarče T4P*** **Hitrejša obdelava s GPU v Chloros+, boljše upravljanje pomnilnika**

**Popravki napak*** Povsem nov uporabniški vmesnik (GUI), ki naj bi zdaj deloval na vseh računalnikih Windows.

</details>

<details>

<summary>Različica 1.0.4</summary>

**Datum izdaje: 5. januar 2026**

**Nove funkcije*** **Preklop slike/metapodatkov**: V brskalniku datotek je bil dodan preklopnik za prikaz metapodatkov izbrane slike v tabeli namesto v mreži slik
* **Drsnik za povečavo mreže slik**: Nov drsnik v uporabniškem vmesniku za prilagajanje velikosti sličic (podpira tudi CTRL + kolesce miške)
* **Gumbi za izvoz mreže slik**: Gumbi v zgornji vrstici za preklop sličic iz JPG v obdelane izvoze (Cilji, Odbojnost, Indeks, LUT)
* **Zavihek Zemljevid**: Nov interaktiven 2D zemljevid, ki prikazuje oznake GPS lokacij slik
  * Podpira Google Maps in ESRI zemljevidne ploščice (samodejno izbere najboljšo storitev ploščic glede na razpoložljivost stopnje povečave)
  * Predogled sličic ob preletu z miško nad oznakami na zemljevidu

**Popravki napak*** Izboljšana podpora za namestitev Chloros na računalnikih, ki niso v angleškem jeziku

</details>

<details>

<summary>Različica 1.0.3</summary>

**Datum izdaje: 20. december 2025**

**Nove funkcije*** Prvi zagon

**Izboljšave*** Prvi zagon

**Popravki napak*** Prvi zagon

**Znane težave*** Prvi zagon

</details>***

## Licenčna pogodba**Lastniška programska oprema** – Avtorske pravice (c) 2026 MAPIR Inc.

Neodobrena uporaba, distribucija ali spreminjanje je prepovedano.

**Brezplačna različica**: Na voljo za osebno in komercialno uporabo z omejenimi funkcijami**Chloros+**: Licenca na podlagi naročnine za napredne funkcije in komercialno uporabo

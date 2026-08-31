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
| **Grafična kartica**    | Združljiva z DirectX 11                                | NVIDIA GPU z 4 GB ali več VRAM                            |
| **Prostor za shranjevanje**          | 6 GB prostega prostora                                       | SSD z 10 GB ali več prostega prostora                            |
| **Zaslon**          | 1920x1080                                            | 2560x1440 ali več                                  |
| **Internet**         | Potreben za \[neobvezno\] aktivacijo licence Chloros+ | Potreben za \[izbirno] aktivacijo licence Chloros+ |

#### Linux amd64 (x86\_64)

| Zahteva       | Minimalna                    | Priporočena               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribucija**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Procesor**     | x86\_64 (Intel/AMD)        | Intel Core i7 ali boljši   |
| **Pomnilnik (RAM)**  | 8 GB                        | 16 GB ali več              |
| **Grafična kartica** | Ni potrebna (obdelava v procesorju)      | NVIDIA GPU z 4 GB ali več VRAM |
| **Prostor za shranjevanje**       | 2 GB prostega prostora             | SSD z 10 GB+ prostega prostora       |
| **Python**        | Python 3.7+ (za SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Zahteva      | Najmanjša                      | Priporočena                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platforma**     | NVIDIA Jetson z JetPack 6 | Jetson Orin NX 16 GB ali AGX Orin |
| **Pomnilnik (RAM)** | 8 GB (skupni GPU/CPU)         | 16 GB+ skupnega                    |
| **Shranjevanje**      | 2 GB prostega prostora               | NVMe SSD z 10 GB+ prostega prostora        |
| **Python**       | Python 3.7+ (za SDK)        | Python 3.10+                    |

{% hint style="info" %}
**Pospeševanje s GPU**: Uporabniki Chloros+ z grafičnimi procesorji NVIDIA lahko uporabijo pospeševanje CUDA za znatno hitrejšo obdelavo. To deluje tako na Windows (namizni grafični procesorji) kot tudi na Linux (namizni grafični procesorji in NVIDIA Jetson). Uporabniki Chloros+ pridobijo tudi večnitno obdelavo za največjo hitrost.
{% endhint %}

***

## Prenesite Chloros

### Najnovejša stabilna različica: različica 1.2.0

<!-- NOLAN: replace installer links + release date for 1.2.0 — the three download buttons below still point at the 1.1.0 Google Drive files, and the release date needs to be added to the heading above. -->



### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Prenesite Chloros za Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Prenos Chloros za Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Prenesite Chloros za Linux arm64 / Jetson (.deb)</a>

#### Namestitveni program Windows (GUI + CLI + Backend)

* **Vrsta datoteke**: .exe (namestitveni program Windows)**Koraki namestitve:**

1. Prenesite zgornjo datoteko .exe
2. Dvakrat kliknite namestitveni program, da začnete namestitev
3. Sledite navodilom namestitvenega čarovnika
4. Izberite namestitveni imenik (privzeto: `C:\Program Files\MAPIR\Chloros\`)
5. Zaključite namestitev in zaženite Chloros ali Chloros CLI
6. Prijavite se s svojim [računom MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (ali nadaljujte z brezplačno različico)

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

#### Linux arm64 — NVIDIA Jetson (paket .deb — CLI + Backend)

* **Vrsta datoteke**: .deb (JetPack 6)
* **Arhitektura**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Za podrobna navodila za namestitev glejte [Namestitev Linux](linux/linux-installation.md), za navodila, specifična za Jetson, pa [Vodnik za NVIDIA Jetson](linux/nvidia-jetson-guide.md).

#### Python SDK (vse platforme)

Vsak namestitveni program vsebuje ustrezno kolo `chloros_sdk`, zato se različica SDK vedno ujema z nameščenim GUI/CLI/backendom. Na Windows ga namestitveni program samodejno namesti v vaš sistem Python; na Linux pa `.deb` shrani kolo v `/usr/lib/chloros/sdk/` in izpiše ukaz za namestitev:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Za gostitelje, ki uporabljajo le pip (na katerih ni nameščen paket Chloros), je SDK na voljo tudi na PyPI:

```bash
pip install chloros-sdk
```

Glejte [API : Python SDK](api-python-sdk.md) ter [SDK Referenca](reference/sdk-reference.md) za dokumentacijo.

{% hint style="info" %}
**Uporabniki Linux**: Paket `.deb` namesti CLI in backend. Za Linux ni grafičnega vmesnika – vsa interakcija poteka prek CLI ali SDK.
{% endhint %}

***

## Dodatni viri

### Python SDK

Za razvijalce in avtomatizirane delovne tokove namestite Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentacija**: [API: Python SDK](api-python-sdk.md)**Zahteve**: Chloros mora biti nameščen (namestitveni program Windows ali paket Linux `.deb`), potrebna je prijava z licenco Chloros+***

## Vsebina paketa

### Namestitveni program Windows

* ✅ **Chloros GUI** – grafični vmesnik s polno funkcionalnostjo
* ✅ **Chloros CLI** – Vmesnik ukazne vrstice (zahteva licenco Chloros+)
* ✅ **Chloros Backend** – Procesni motor
* ✅ **Profil kamere** – vnaprej nastavljene predloge kamer MAPIR

### Paket .deb Linux

* ✅ **Chloros CLI** – vmesnik ukazne vrstice (zahteva licenco Chloros+)
* ✅ **Chloros Backend** – Obdelovalni motor
* ✅ **Profili kamer** – Vnaprej nastavljene predloge za kamere MAPIR
* ❌ Brez grafičnega vmesnika — Linux je brez grafičnega vmesnika, na voljo je le CLI/SDK

### Python SDK (PIP, vse platforme)

* ✅ **Chloros SDK** – Python API (zahteva licenco Chloros+)***

## Nadgradnja na Chloros+

Odkleni napredne funkcije z naročnino Chloros+:

* 🚀 **Večnitna obdelava** – vzporedna obdelava slik
* ⚡ **Pospeševanje z GPU (CUDA)** – izkoristite zmogljivost grafičnih procesorjev NVIDIA
* 💻 **Dostop do CLI** – Avtomatizirajte z orodji za ukazno vrstico
* 🐍 **Python SDK** – Programski dostop do API
* 📱 **Več naprav** – uporaba na 2–10+ napravah (odvisno od paketa)
* **🐻 Napredna metoda debayerja, ki upošteva teksturo** – visokokakovosten debayer, ki upošteva robove, v kombinaciji z modelom za odstranjevanje šuma na podlagi umetne inteligence in strojnega učenja, ki odstrani skoraj ves šum pri debayerju.
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
* V pregledovalniku dogodkov Windows preverite podrobnosti o napaki
* Obrnite se na podporo uporabnikom in priložite dnevnike napak

**CLI se ne zažene (Linux):**

* Preverite, ali je paket `.deb` pravilno nameščen: `dpkg -l | grep chloros`
* Preverite dovoljenja: `sudo chmod +x /usr/bin/chloros-cli`
* Izvedite diagnostiko: `chloros-cli selftest`
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
* 📚 **Dokumentacija**: [Prvi koraki](./)
* ❓ **Pogosta vprašanja**: [Pogosta vprašanja](faq.md)***

## Posodobitve programske opreme

Chloros preveri, ali so na voljo posodobitve, vas obvesti, ko je na voljo nova različica, in vas preusmeri na to stran za prenos — posodobitev izvedete z zagonom novega podpisanega namestitvenega programa. Vaše nastavitve in projekti ostanejo nespremenjeni tudi po posodobitvi. V Linux in Jetson `chloros-cli update` preveri, ali je na voljo novejša različica, ter ponudi prenos in namestitev ustreznega `.deb` (ta ukaz je na voljo le v različici Linux).

***

## Seznam sprememb**Različica 1.2.0 (najnovejša)**— za celoten seznam funkcij glejte**Novosti v Chloros 1.2.0** na strani [Začetek](./).

<details>

<summary>Različica 1.0.5</summary>

**Datum izdaje: 10. februar 2026**

**Nove funkcije*** **Metoda debayiranja z upoštevanjem teksture \[Samo za Chloros+] –** Metoda z upoštevanjem teksture uporablja visokokakovostno debayiranje z upoštevanjem robov v kombinaciji z modelom za odstranjevanje šuma na podlagi umetne inteligence/strojnega učenja, ki odstrani skoraj ves šum pri debayiranju.
* **Podpora za kalibracijske tarče T4P*** **Hitrejša obdelava s pomočjo grafičnega procesorja (GPU) za Chloros+, boljše upravljanje pomnilnika**

**Popravki napak*** Povsem nov uporabniški vmesnik (GUI), ki naj bi zdaj deloval na vseh računalnikih Windows.

</details>

<details>

<summary>Različica 1.0.4</summary>

**Datum izdaje: 5. januar 2026**

**Nove funkcije*** **Preklop med sliko in metapodatki**: V brskalniku datotek je bil dodan preklopnik za prikaz metapodatkov izbrane slike v tabeli namesto v mreži slik
* **Drsnik za povečavo mreže slik**: Nov drsnik v uporabniškem vmesniku za prilagajanje velikosti sličic (podpira tudi CTRL + kolesce miške)
* **Gumb za izvoz mreže slik**: Gumbi v zgornji vrstici za preklop sličic iz formata JPG na obdelane izvoze (cilji, odbojnost, indeks, LUT)
* **Zavihek »Zemljevid«**: Nov interaktiven 2D zemljevid, ki prikazuje oznake GPS-lokacij na slikah
  * Podpira Google Maps in kartografske ploščice ESRI (samodejno izbere najboljšo storitev ploščic glede na razpoložljivost stopnje povečave)
  * Predogled sličic ob preletu z miško nad oznakami na zemljevidu

**Popravki napak*** Izboljšana podpora za namestitev Chloros na računalnikih z neangleškim jezikom

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

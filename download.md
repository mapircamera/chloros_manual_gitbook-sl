---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Prenos

Prenesite najnovejšo različico Chloros, da začnete z obdelavo multispektralnih slik.

### Sistemski zahtevi

| Zahteva          | Minimalna                                              | Priporočena                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Operacijski sistem** | Windows 10 (64-bitni)                                  | Windows 11 (64-bitni)                                  |
| **Procesor**        | Intel Core i5 ali enakovreden                          | Intel Core i7 ali boljši                              |
| **Pomnilnik (RAM)**     | 8 GB                                                  | 16 GB ali več                                         |
| **Grafična kartica**    | Združljiva z DirectX 11                                | NVIDIA GPU z 4 GB+ VRAM                            |
| **Shranjevanje**          | 6 GB prostega prostora                                       | SSD z 10 GB+ prostega prostora                            |
| **Zaslon**          | 1920x1080                                            | 2560x1440 ali več                                  |
| **Internet**         | Potreben za \[neobvezno] aktivacijo licence Chloros+ | Potreben za \[neobvezno] aktivacijo licence Chloros+ |

{% hint style="info" %}
**Pospeševanje GPU**: Uporabniki Chloros+ z grafičnimi procesorji NVIDIA lahko uporabljajo pospeševanje CUDA za bistveno hitrejše obdelovanje. Uporabniki Chloros+ pridobijo tudi večnitno obdelovanje za največjo hitrost.
{% endhint %}

***

## Prenesite Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Prenesite Chloros tukaj</a>

### Najnovejša stabilna različica

**Chloros Namestitveni program za Windows*** **Različica**: 1.0.5
* **Datum izdaje**: 10. februar 2026
* **Velikost datoteke (prenos)**: 1,6 GB
* **Velikost datoteke (namestitev)**: 5,7 GB
* **Vrsta datoteke**: .exe (namestitveni program Windows)

#### **Koraki namestitve:**

1. Prenesite datoteko `CHLOROS INSTALLER - CURRENT VERSION.exe`
2. Dvakrat kliknite namestitveni program, da začnete namestitev
3. Sledite navodilom namestitvenega čarovnika
4. Izberite namestitveni imenik (privzeto: `C:\Program Files\[USER]\Chloros\`)
5. Zaključite namestitev in zaženite Chloros ali Chloros CLI
6. Prijavite se s svojim [MAPIR Cloud Chloros+ računom](https://cloud.mapir.camera/pricing) (ali nadaljujte z brezplačno različico).

{% hint style="success" %}
Namestitveni program samodejno doda `chloros-cli` v sistemsko pot PATH za dostop prek ukazne vrstice.
{% endhint %}

***

## Dodatni viri

### Python SDK

Za razvijalce in avtomatizirane delovne tokove namestite Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentacija**: [API: Python SDK](api-python-sdk.md)**Zahteve**: Chloros Desktop mora biti nameščen, Chloros+ licenca za prijavo je potrebna.***

## Kaj je vključeno

Namestitev Chloros vključuje:

* ✅ **Chloros** – grafični vmesnik (GUI) s polno funkcionalnostjo
* ✅ **Chloros CLI** – vmesnik ukazne vrstice (zahteva licenco Chloros+)
* ✅ **Chloros SDK** – Python API (zahteva licenco Chloros+)
* ✅ **Profil kamere** – vnaprej nastavljene predloge kamere MAPIR***

## Nadgradnja na Chloros+

Odkleni napredne funkcije z naročnino Chloros+:

* 🚀 **Večnitno obdelovanje** – vzporedna obdelava slik
* ⚡ **Pospeševanje GPU (CUDA)** – izkoristite moč grafičnega procesorja NVIDIA
* 💻 **Dostop do CLI** – avtomatizacija z orodji za ukazno vrstico
* 🐍 **Python SDK** – programski dostop do API
* 📱 **Več naprav** – uporaba na 2–10+ napravah (odvisno od načrta)
* **🐻 Napredna metoda Debayer z upoštevanjem teksture** – visokokakovostna metoda Debayer z upoštevanjem robov v kombinaciji z modelom AI/ML za odstranjevanje šuma, ki odstrani skoraj ves šum Debayer. 
* 🧮 **Prilagojene formule** – Ustvarjanje prilagojenih multispektralnih indeksov

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Oglejte si načrte in cene Chloros+</a></p>***

## Pomoč pri namestitvi

### Odpravljanje težav

**Namestitev ni uspela in se je pojavilo naslednje sporočilo o napaki:**

* Preverite, ali imate administratorske pravice
* Začasno onemogočite protivirusno programsko opremo
* Preverite, ali izpolnjujete minimalne sistemske zahteve

**Aplikacija se ne zažene:**

* Preverite, ali je nameščen Windows 10/11 (64-bit)
* Posodobite grafične gonilnike
* Preverite podrobnosti napake v Windows Event Viewer
* Obrnite se na podporo z dnevniki napak

**Težave z aktiviranjem licence:**

* Preverite, ali je internetna povezava aktivna.
* Preverite poverilnice na [https://cloud.mapir.camera](https://cloud.mapir.camera).
* Preverite, ali požarni zid ne blokira Chloros.
* Podrobna navodila najdete v [Chloros+ Prijava](chloros+-login.md)

### Pomoč

Potrebujete pomoč pri namestitvi ali nastavitvi?

* 📧 **E-pošta**: info@mapir.camera
* 🌐 **Spletna stran**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentacija**: [Začetek dela](./)
* ❓ **Pogosta vprašanja**: [Pogosta vprašanja](faq.md)***

## Seznam sprememb

<details>

<summary>Različica 1.0.5</summary>

#### **Datum izdaje**: 10. februar 2026**Nove funkcije*** **Metoda Debayer z upoštevanjem teksture \[Chloros+ Samo] -** Texture Aware uporablja visokokakovostni debayer z upoštevanjem robov v kombinaciji z modelom AI/ML za odstranjevanje šuma, ki odstrani skoraj ves šum debayeringa.
* **Podpora za kalibracijske cilje T4P*** **Hitrejša obdelava GPU Chloros+, boljše upravljanje pomnilnika**

**Popravki napak*** Povsem nov uporabniški vmesnik (GUI), ki naj bi zdaj deloval na vseh računalnikih Windows.

</details>

<details>

<summary>Različica 1.0.4</summary>

#### **Datum izdaje**: 5. januar 2026**Nove funkcije*** **Preklop slike/metapodatkov**: V brskalniku datotek je bil dodan preklop, ki omogoča ogled metapodatkov izbrane slike v tabeli namesto v mreži slik.
* **Drsljaj za povečavo mreže slik**: Nov drsljaj v uporabniškem vmesniku za prilagajanje velikosti sličic (podpira tudi CTRL + miškin kolesce)
* **Gumb za izvoz mreže slik**: Gumbi v zgornji vrstici za preklop miniatur iz JPG v obdelane izvoze (cilji, odbojnost, indeks, LUT)
* **Zavihek zemljevid**: nov interaktiven 2D zemljevid, ki prikazuje oznake GPS lokacije slike.
  * Podpira Google Maps in ESRI zemljevidske ploščice (samodejno izbere najboljšo ploščico glede na razpoložljivost stopnje povečave).
  * Predogled sličice ob preletu z miško nad oznakami na zemljevidu.

**Popravki napak*** Izboljšana podpora za namestitev Chloros na računalnikih, ki niso v angleškem jeziku.

</details>

<details>

<summary>Različica 1.0.3</summary>

#### **Datum izdaje**: 20. december 2025**Nove funkcije*** Prvi zagon

**Izboljšave*** Prvi zagon

**Popravki napak*** Prvi zagon

**Znane težave*** Prvi zagon

</details>***

## Licenčna pogodba**Lastniška programska oprema** – Copyright (c) 2026 MAPIR Inc.

Neodobrena uporaba, distribucija ali sprememba je prepovedana.

**Brezplačna različica**: Na voljo za osebno in komercialno uporabo z omejenimi funkcijami.**Chloros+**: Licenca na podlagi naročnine za napredne funkcije in komercialno uporabo.

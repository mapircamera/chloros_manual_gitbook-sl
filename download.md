---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Prenos

Prenesite najnovejšo različico Chloros za Windows, da se lahko začnete ukvarjati z obdelavo multispektralnih slik.

### Sistemski zahtevi

| Zahteva          | Minimalna                         | Priporočena                     |
| -------------------- | ------------------------------- | ------------------------------- |
| **Operacijski sistem** | Windows 10 (64-bitni)             | Windows 11 (64-bitni)             |
| **Procesor**        | Intel Core i5 ali enakovreden     | Intel Core i7 ali boljši         |
| **Pomnilnik (RAM)**     | 8 GB                             | 16 GB ali več                    |
| **Grafična kartica**    | Združljiva z DirectX 11           | NVIDIA GPU z 4 GB+ VRAM       |
| **Shranjevanje**          | 2 GB prostega prostora                  | SSD z 10 GB+ prostega prostora       |
| **Zaslon**          | 1920x1080                       | 2560x1440 ali več             |
| **Internet**         | Potreben za aktiviranje licence | Potreben za aktiviranje licence |

{% hint style=&quot;info&quot; %}
**Pospeševanje GPU**: Uporabniki Chloros+ z grafičnimi karticami NVIDIA (4 GB+ VRAM) lahko uporabijo pospeševanje CUDA za bistveno hitrejše obdelovanje.
{% endhint %}

***

## Prenesite Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Prenesite Chloros tukaj</a>

### Najnovejša stabilna različica

**Chloros Namestitveni program za Windows**

* **Različica**: 1.0.3
* **Datum izdaje**: december ?, 2025
* **Velikost datoteke**: 1,6 GB
* **Vrsta datoteke**: .exe (namestitveni program Windows)

#### **Koraki namestitve:**

1. Prenesite datoteko `CHLOROS INSTALLER - CURRENT VERSION.exe`.
2. Dvakrat kliknite namestitveni program, da začnete namestitev.
3. Sledite navodilom namestitvenega čarovnika.
4. Izberite namestitveni imenik (privzeto: `C:\Program Files\Chloros\`).
5. Zaključite namestitev in zaženite Chloros.
6. Prijavite se s svojim računom MAPIR Cloud Chloros+ (ali nadaljujte z brezplačno različico).

{% hint style=&quot;success&quot; %}
Namestitveni program samodejno doda `chloros-cli` v sistemsko pot PATH za dostop prek ukazne vrstice.
{% endhint %}

***

## Dodatni viri

### Python SDK

Za razvijalce in avtomatizirane delovne tokove namestite Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentacija**: [API: Python SDK](api-python-sdk.md)

**Zahteve**: Chloros Desktop mora biti nameščen, potrebna je licenca Chloros+.

***

## Kaj je vključeno

Namestitev Chloros vključuje:

* ✅ **Chloros Desktop GUI** – grafični vmesnik s polno funkcionalnostjo
* ✅ **Chloros (brskalnik)** – spletni vmesnik za sisteme z nižjimi specifikacijami
* ✅ **Chloros CLI** – vmesnik ukazne vrstice (zahteva licenco Chloros+)
* ✅ **Backend Engine** – potek obdelave slik
* ✅ **Profil kamere** - Vnaprej nastavljene predloge kamer MAPIR

***

## Nadgradnja na Chloros+

Odkleni napredne funkcije z naročnino Chloros+:

* 🚀 **Večnitna obdelava** – vzporedna obdelava slik
* ⚡ **Pospeševanje GPU (CUDA)** – izkoristite moč grafičnega procesorja NVIDIA
* 💻 **Dostop CLI** – avtomatizirajte z orodji za ukazno vrstico
* 🐍 **Python SDK** - Programski dostop API
* 📱 **Več naprav** - Uporaba na 2–10+ napravah (odvisno od načrta)
* 🧮 **Prilagojene formule** - Ustvarjanje prilagojenih multispektralnih indeksov

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Oglejte si načrte in cene Chloros+</a></p>***

## Pomoč pri namestitvi

### Odpravljanje težav

**Namestitev ni uspela in se je pojavila napaka:**

* Preverite, ali imate administratorske pravice.
* Začasno onemogočite protivirusno programsko opremo.
* Preverite, ali izpolnjujete minimalne sistemske zahteve.

**Aplikacija se ne zažene:**

* Poskusite z različico Chloros (brskalnik)
* Preverite, ali je nameščen Windows 10/11 (64-bit)
* Posodobite grafične gonilnike
* Preverite podrobnosti napake v pregledovalniku dogodkov Windows
* Obrnite se na podporo uporabnikom s protokoli napak.

**Težave z aktiviranjem licence:**

* Preverite, ali je internetna povezava aktivna.
* Preverite poverilnice na [https://cloud.mapir.camera](https://cloud.mapir.camera).
* Preverite, ali požarni zid ne blokira Chloros.
* Podrobna navodila najdete na [Chloros+ Prijava](chloros+-login.md)

### Pomoč

Potrebujete pomoč pri namestitvi ali nastavitvi?

* 📧 **E-pošta**: info@mapir.camera
* 🌐 **Spletna stran**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentacija**: [Začetek dela](./)
* ❓ **Pogosta vprašanja**: [Pogosta vprašanja](faq.md)

***

## Seznam sprememb

<details>

<summary>Različica 1.0.3</summary>

### **Datum izdaje**: december ?, 2025

#### Nove funkcije

* Prvi zagon

#### Izboljšave

* Prvi zagon

#### Popravki napak

* Prvi zagon

#### Znane težave

* Prvi zagon

</details>***

## Licenčna pogodba

**Lastniška programska oprema** – Avtorske pravice (c) 2025 MAPIR Inc.

Neodobrena uporaba, distribucija ali sprememba je prepovedana.

**Brezplačna različica**: Na voljo za osebno in komercialno uporabo z omejenimi funkcijami.

**Chloros+**: Licenca na podlagi naročnine za napredne funkcije in komercialno uporabo.

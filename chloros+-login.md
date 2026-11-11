# Prijava v Chloros+

## Prijava prek grafičnega vmesnika

Meni v stranskem pasu <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> vam omogoča prijavo v račun Chloros+ in dostop do dodatnih funkcij.

**Na vsakem računalniku se morate prijaviti le enkrat.** GUI, CLI in Python SDK uporabljajo isto shranjeno sejo – prijava prek namiznega GUI aktivira tudi CLI in SDK na tem računalniku (in obratno prek `chloros-cli login`).

Ko ste prijavljeni, se prikažejo podrobnosti vašega računa:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.3.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->

## Ravni naročnin

| Naročnina | `plan_id` | Vrsta |
| --- | --- | --- |
| Iron | `0` | Brezplačna |
| Copper | `1` | Plačljiv (Chloros+) |
| Bronze | `2` | Plačljiv (Chloros+) |
| Silver | `3` | Plačljiv (Chloros+) |
| Zlato | `4` | Plačljivo (Chloros+) |

Za informacije o vsebini posameznih plačljivih paketov glejte [pakete in cene](https://cloud.mapir.camera/pricing).

### Dostop do CLI-ja / SDK-ja zahteva plačljiv paket

Za dostop do CLI in Python SDK je potrebna **katera koli plačljiva stopnja Chloros+ (Copper ali višja)**. To se uveljavlja**na strani strežnika** — vsaka zahteva CLI/SDK mora vsebovati tako aktivno sejo kot tudi plačljiv načrt:

| Stanje HTTP | `error_code` | Pomen | Rešitev |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Niste prijavljeni na tem računalniku | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Prijavljen, vendar je raven paketa prenizka (brezplačna raven Iron) | Nadgradite na kateri koli plačljiv paket Chloros+ |

`chloros-cli status` ostaja dostopen v brezplačnem paketu, zato lahko vedno vidite svoj trenutni paket in razlog za zavrnitev dostopa.

### Omejitve povezane strojne opreme po paketih

Vsak paket omejuje število kamer LATTICE in svetlobnih senzorjev DAQ, ki jih je mogoče hkrati povezati v živo:

| Paket | Kamere LATTICE | Svetlobni senzorji DAQ |
| --- | --- | --- |
| Iron (brezplačen / niste prijavljeni) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## Prijava v CLI



Prijavite se s svojimi podatki za Chloros+, da omogočite obdelavo v CLI. V sistemu Linux (brez grafičnega vmesnika) je to edini način za aktivacijo vaše licence.

**Sintaksa:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Uporabniki SDK**: Python SDK ponuja tudi programsko metodo `logout()` za izbris shranjenih prijavnih podatkov iz predpomnilnika. Podrobnosti najdete v [referenčnem priročniku za SDK](reference/sdk-reference.md).
{% endhint %}

**Primer:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Posebni znaki**: Gesla, ki vsebujejo znake, kot so `$`, `!` ali presledke, obdajte z enojnimi narekovaji.
{% endhint %}

**Izhod:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.3.0"; capture a successful login with the current output format. -->

### Shranjevanje poverilnic

Shranjene poverilnice in konfiguracija so na **vseh platformah** shranjene v mapi `.chloros` v vašem domačem imeniku:

| Platforma | Pot do predpomnilnika poverilnic |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

Če namestite novejšo različico nad obstoječo, se vaši podatki za prijavo ohranijo v sistemu Windows (namestitveni program) in v sistemu Linux (mapa `.deb`, vključno z mapo `chloros-cli update`). Odstranitev programa Chloros — ali odstranitev paketa zLinux

a — izbriše shranjene prijavne podatke v predpomnilniku, zato se morate po novi namestitvi ponovno prijaviti.

### Potek veljavnosti naročnine in obdobje odloga brez internetne povezave

Potek veljavnosti naročnine v grafičnem vmesniku prikazuje, kdaj bo vaša licenca prenehala veljati. Pri ponavljajočih se mesečnih naročninah poteče veljavnost ob koncu meseca; pri letnih naročninah pa eno leto po začetku naročnine.

Program Chloros preverja veljavnost vaše licence prek spleta, vendar je delovanje brez spletne povezave podprto v okviru obdobja odloga:

* Uspešna preverjanja na strežniku se shranijo v predpomnilnik za **5 minut**, zato pri običajni uporabi pride do zelo malo zahtevkov za licenco.
* Podpisana licenca, vezana na posamezni računalnik, pokriva daljša obdobja brez internetne povezave: **30 dni za mesečne načrte**in**do datuma izteka vaše naročnine (največ 365 dni) za letne načrte**.
* Ko prehodno obdobje poteče, se paket spremeni v brezplačno stopnjo »Iron«, dokler naprava ne vzpostavi povezave z licenčnim strežnikom; dostop se ponovno vzpostavi ob naslednjem uspešnem preverjanju.

### Omejitev števila naprav

Vsak načrt Chloros+ ponuja različno število registriranih naprav. Vsaka naprava, na katero se prijavite z računom Chloros+, se šteje v število vaših registriranih naprav. Napravo lahko preimenujete in odstranite na strani svojega računa MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Načrt Chloros</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Podprte naprave</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>

Natančno število naprav, ki jih lahko uporabljate v okviru svojega računa, je prikazano na strani vašega računa v oblaku MAPIR. Odjava z naprave zanesljivo sprosti njeno mesto, naprava, ki je že registrirana, pa se lahko vedno ponovno prijavi, tudi če je račun dosegel omejitev števila naprav.

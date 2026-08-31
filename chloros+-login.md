# Chloros+ Prijava

## Prijava prek grafičnega vmesnika

Stranski meni uporabnika <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> vam omogoča prijavo v vaš račun Chloros+ in odklepanje dodatnih funkcij.

**Na vsakem računalniku se morate prijaviti le enkrat.** Grafični vmesnik, CLI in Python SDK delijo isto shranjeno sejo — prijava prek namiznega grafičnega vmesnika aktivira tudi CLI in SDK na tem računalniku (in obratno prek `chloros-cli login`).

Ko se prijavite, se prikažejo podatki o vašem računu:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.2.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->
## Ravni načrtov

| Načrt | `plan_id` | Vrsta |
| --- | --- | --- |
| Iron | `0` | Brezplačen |
| Copper | `1` | Plačljiv (Chloros+) |
| Bronze | `2` | Plačljiva (Chloros+) |
| Srebro | `3` | Plačljivo (Chloros+) |
| Zlato | `4` | Plačljivo (Chloros+) |

V [paketih in cenah](https://cloud.mapir.camera/pricing) si oglejte, kaj vključuje vsaka plačljiva stopnja.

### Dostop do CLI / SDK zahteva plačano stopnjo

Za dostop do CLI in Python ter SDK je potrebna **katera koli plačljiva stopnja Chloros+ (Copper ali višja)**. To se izvaja**na strežniški strani** — vsaka zahteva CLI/SDK mora vsebovati tako aktivno sejo kot tudi plačani paket:

| HTTP status | `error_code` | Pomen | Rešitev |
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

Prijavite se s svojimi poverilnicami za Chloros+, da omogočite obdelavo v CLI. V programu Linux (brez grafičnega vmesnika) je to edini način za aktivacijo vaše licence.

**Sintaksa:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Uporabniki SDK**: Python SDK ponuja tudi programsko metodo `logout()` za izbris podatkov za prijavo iz predpomnilnika. Za podrobnosti glejte [Referenco za SDK](reference/sdk-reference.md).
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
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.2.0"; capture a successful login with the current output format. -->
### Shranjevanje poverilnic

Shranjene poverilnice in konfiguracija so na **vseh platformah** shranjene v mapi `.chloros` v vašem domačem imeniku:

| Platforma | Pot do predpomnilnika poverilnic |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

### Potek veljavnosti naročnine in obdobje odloga za delo brez povezave

Datum poteka veljavnosti naročnine v grafičnem vmesniku prikazuje, kdaj bo vaša licenca prenehala veljati. Pri ponavljajočih se mesečnih naročninah je datum poteka ob koncu meseca; pri letnih naročninah pa eno leto po začetku naročnine.

Chloros preveri veljavnost vaše licence prek spleta, vendar je delo brez spletne povezave podprto v okviru obdobja odloga:

* Uspešne preveritve na strežniku se shranijo v predpomnilnik za **5 minut**, zato pri običajni uporabi pride do zelo malo zahtevkov za licenco.
* Podpisana licenca, vezana na posamezni računalnik, pokriva daljša obdobja brez internetne povezave: **30 dni za mesečne načrte**in**do datuma izteka vašega naročila (največ 365 dni) za letne načrte**.
* Ko prehodno obdobje poteče, se paket spremeni v brezplačno stopnjo Iron, dokler računalnik ne vzpostavi povezave z licenčnim strežnikom; dostop se ponovno vzpostavi ob naslednjem uspešnem preverjanju.

### Omejitev števila naprav

Vsak paket Chloros+ ponuja različno število registriranih naprav. Vsaka naprava, na kateri se prijavite z računom Chloros+, se šteje v število vaših registriranih naprav. Napravo lahko preimenujete in odstranite na strani svojega računa MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Paket Chloros+</th><th align="center">BAKROV</th><th align="center">BRONZE</th><th align="center">SREBRO</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Podprte naprave</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>Natančno število naprav, ki jih lahko uporabljate na svojem računu, je prikazano na strani vašega računa MAPIR Cloud. Odjava z naprave zanesljivo sprosti njeno mesto, naprava, ki je že registrirana, pa se lahko vedno ponovno prijavi, tudi če je račun dosegel omejitev števila naprav.

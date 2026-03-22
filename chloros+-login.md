# Prijava v Chloros+

## Prijava v Chloros in Chloros (spletni brskalnik)

Stranski meni <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> stranski meni vam omogoča prijavo v vaš račun Chloros+ in odklepanje dodatnih funkcij.

Ko se prijavite, se prikažejo podatki o vašem računu:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## Prijava v CLI

Prijavite se s svojimi poverilnicami za Chloros+, da omogočite obdelavo v CLI. V Linux (brez grafičnega vmesnika) je to edini način za aktivacijo vaše licence.

**Sintaksa:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Uporabniki SDK**: Python SDK ponuja tudi programsko metodo `logout()` za izbris podatkov za prijavo iz predpomnilnika. Za podrobnosti glejte [dokumentacijo Python SDK](api-python-sdk.md#logout).
{% endhint %}

**Primer:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Posebni znaki**: Gesla, ki vsebujejo znake, kot so `$`, `!` ali presledke, obdajte z enojnimi narekovaji.
{% endhint %}

**Izhod:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Shranjevanje poverilnic

Shranjene poverilnice so shranjene na lokaciji, specifični za platformo:

| Platforma | Pot do predpomnilnika poverilnic |
| --- | --- |
| **Windows** | `%APPDATA%\Chloros\cache\` |
| **Linux** | `~/.cache/chloros/` |

### Potek veljavnosti paketa

Potek veljavnosti paketa v grafičnem uporabniškem vmesniku prikazuje, kdaj bo vaša licenca prenehala veljati. Pri ponavljajočih se mesečnih naročninah poteče veljavnost ob koncu meseca. Pri letnih naročninah poteče veljavnost eno leto po začetku naročnine. Preverjanje licence zahteva mesečno internetno povezavo za potrditev, z 30-dnevnim odlogom.

### Omejitev naprav

Vsak načrt Chloros+ ponuja različno število registriranih naprav. Vsaka naprava, na katero se prijavite z računom Chloros+, se šteje v število vaših registriranih naprav. Napravo lahko preimenujete in odstranite na strani svojega računa MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Načrt Chloros+</th><th align="center">BRIZGA</th><th align="center">BRON</th><th align="center">SREBRO</th><th align="center">ZLATO</th></tr></thead><tbody><tr><td align="right">Podprte naprave</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>

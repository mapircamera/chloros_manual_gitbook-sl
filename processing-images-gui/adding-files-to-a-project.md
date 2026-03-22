# Dodajanje datotek v projekt

Ko ustvarite ali odprete projekt v programu Chloros, je naslednji korak dodajanje vaših multispektralnih slik, da lahko začnete z obdelavo. Zavihek »File Browser«<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> omogoča enostaven uvoz slik in upravljanje z vašim nizom podatkov.

## Dostop do brskalnika datotek

1. Odprite ali ustvarite projekt v Chloros
2. Kliknite ikono **Brskalnik datotek** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> v levem stranskem meniju
3. V oknu Brskalnik datotek se bo prikazal seznam datotek vašega projekta

{% hint style="info" %}
**Podprti tipi datotek**: Chloros podpira slikovne datoteke RAW+JPG in JPG iz kamer MAPIR, Survey3W in Survey3N. Priporočajo se le datoteke RAW+JPG.
{% endhint %}

***

## Dodajanje slik v projekt

Obstajata dva glavna načina za dodajanje slik v projekt:

### Metoda 1: Dodaj datoteke

To možnost uporabite za uvoz posameznih slikovnih datotek ali manjšega izbora datotek.

1. Kliknite gumb **»Dodaj datoteke«** <img src="../.gitbook/assets/image.png" alt="" data-size="line"> na vrhu okna brskalnika datotek
2. Poiščite mapo, ki vsebuje vaše slike
3. Izberite eno ali več slikovnih datotek (za izbiro več datotek držite tipko **Ctrl**)
4. Kliknite **&quot;Odpri&quot;**, da uvozite izbrane datoteke

### Metoda 2: Dodaj mapo

To možnost uporabite za uvoz vseh slik iz mape naenkrat.

1. Kliknite gumb **»Dodaj mapo«** <img src="../.gitbook/assets/image (1).png" alt="" data-size="line"> na vrhu okna brskalnika datotek
2. Poiščite in izberite mapo, ki vsebuje slike iz vaše seje zajemanja
3. Kliknite **&quot;Izberi mapo&quot;**, da uvozite vse podprte slike iz te mape***

## Razumevanje tabele brskalnika datotek

Ko so slike uvožene, se prikažejo v tabeli z naslednjimi stolpci:

### Ime datoteke

* Izvirno ime datoteke iz kamere
* Ohranja konvencijo poimenovanja kamere (npr. IMG\_0001.RAW)

### Časovni žig

* Datum in čas zajema slike
* Izvlečeno iz metapodatkov EXIF slike
* Uporablja se za sinhronizacijo PPK in zaznavanje kalibracijskih ciljev

### Model kamere

* Samodejno zaznana konfiguracija kamere in filtra
* Primeri: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Uporablja se za uporabo pravilnih profilov obdelave

### Stolpec ciljev (potrditveno polje)

* To polje označite za slike, ki vsebujejo kalibracijske cilje
* Zelo pospeši zaznavanje ciljev med obdelavo
* Za podrobnosti glejte [Izbira ciljnih slik](choosing-target-images.md)

### Ogled metapodatkov slike

S klikom na gumb za preklop v zgornjem desnem kotu nad tabelo se v območju mreže slik prikažejo metapodatki izbrane slike.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Upravljanje datotek v projektu

### Odstranjevanje datotek

Če želite iz projekta odstraniti neželene slike:

1. V tabeli brskalnika datotek izberite eno ali več slik
2. Kliknite gumb **&quot;Odstrani izbrano&quot;** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line"> .
3. Potrdite odstranitev (datoteke se ne izbrišejo z diska, ampak se le odstranijo iz projekta).

### Razvrščanje in filtriranje

* **Razvrščanje po stolpcu**: kliknite kateri koli naslov stolpca, da razvrstite slike.
* **Razvrščanje po časovnem žigu**: koristno za organiziranje kronoloških zaporedij posnetkov.
* **Filter po modelu kamere**: združite slike po vrsti kamere, če uporabljate več kamer.***

## Predogled slike

### Ogled celotne slike

Kliknite katero koli miniaturno sliko v brskalniku datotek, da jo prikažete v glavnem predoglednem področju:

1. Slika se prikaže v osrednjem predoglednem oknu
2. Uporabite gumbe za povečavo, da pregledate podrobnosti slike
3. Med slikami se premikajte s puščičnimi tipkami

### Hitra navigacija

* **Prejšnja slika**: Kliknite levo puščično tipko ali pritisnite tipko ←
* **Naslednja slika**: Kliknite desno puščico ali pritisnite tipko →
* **Povečanje/pomanjšanje**: Uporabite kolesce miške ali gumbe za povečavo
* **Premikanje**: Kliknite in povlecite po sliki, ko je povečana***

## Obravnavanje podvojenih datotek

Chloros samodejno zazna in prezre podvojene datoteke:

* Datoteke z identičnimi imeni se preskočijo
* Preprečuje naključno dvojno obdelavo
* Ob zaznanih podvojenih datotekah se prikaže opozorilno sporočilo

{% hint style="warning" %}
**Pomembno**: Pred uvozom ne preimenujte ali spreminjajte izvirnih slikovnih datotek. Chloros se za pravilno obdelavo zanaša na izvirna imena datotek in metapodatke.
{% endhint %}

***

## Mešani nizi podatkov kamer

Če vaš projekt vsebuje slike iz več kamer MAPIR:

1. Chloros samodejno zazna vsak model kamere
2. Vsak tip kamere se obdela z ustreznim kalibracijskim profilom
3. Brskalnik datotek prikaže model kamere v stolpcu Model kamere
4. Obdelava uporabi pravilne nastavitve za vsak tip kamere

**Primer scenarija**: Survey3W RGN + Survey3N OCN nastavitev z dvema kamerama***

## Najboljše prakse

### Uredite pred uvozom

* Shranite slike kalibracijskih ciljev v isti mapi kot slike meritev
* Ohranite originalno strukturo map iz vaše kamere/SD kartice
* Ne mešajte podatkovnih nizov iz različnih sej v enem projektu

### Poimenovanje datotek

* Ohranite originalna imena datotek kamere (IMG\_0001.RAW, itd.)
* Ne preimenujte datotek pred uvozom
* Izvirna imena vsebujejo pomembne metapodatke

### Slike kalibracijskih ciljev

* V vsako sejo vedno vključite 1–2 sliki kalibracijskih ciljev
* Cilje posnemite pred in po seji snemanja
* Cilje postavite v enake svetlobne pogoje kot območje snemanja
* Slike ciljev označite z uporabo potrditvenega polja Target, da pospešite obdelavo

***

## Pogoste težave in rešitve

### Slike se po uvozu ne prikažejo

**Možni vzroki:**

* Datotečni format ni podprt (samo RAW+JPG in JPG iz kamer MAPIR)
* Slike so iz kamer, ki niso MAPIR (glej [Podprte kamere](../supported-cameras.md))
* Poškodovana datoteka ali nepopoln prenos s kartice SD

**Rešitev**: Preverite združljivost formata datoteke in modela kamere

### Model kamere ni zaznan

**Možni vzroki:**

* Spremenjeni metapodatki EXIF
* Slike so bile urejene v zunanji programski opremi
* Nepopoln prenos datotek

**Rešitev**: Ponovno uvozite originalne, nespremenjene datoteke iz kamere/kartice SD

### Manjkajoči časovni žigi

**Možni vzroki:**

* Ura fotoaparata ni pravilno nastavljena
* Podatki EXIF so bili odstranjeni z zunanjo programsko opremo

**Rešitev**: Preverite, ali so bile nastavitve časa fotoaparata med zajemanjem pravilne***

## Naslednji koraki

Ko so datoteke uvožene:

1. **Preglejte seznam datotek** – Preverite, ali so bile vse slike pravilno naložene
2. **Preverite modele fotoaparatov** – Preverite, ali je fotoaparat pravilno prepoznan
3. **Označite ciljne slike** – Glejte [Izbira ciljnih slik](choosing-target-images.md)
4. **Prilagodite nastavitve** – Konfigurirajte možnosti obdelave v [Nastavitvah projekta](adjusting-project-settings.md)
5. **Začnite obdelavo** – glejte [Začetek obdelave](starting-the-processing.md)

Za podrobne informacije o konfiguraciji projekta glejte [Prilagajanje nastavitev projekta](adjusting-project-settings.md).

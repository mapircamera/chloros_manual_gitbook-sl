# Dodajanje datotek v projekt

Ko ustvarite ali odprete projekt v Chloros, je naslednji korak dodajanje multispektralnih slik, da lahko začnete z obdelavo. Z zavihkom Brskalnik po datotekah<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> omogoča enostavno uvažanje slik in upravljanje podatkovne zbirke.

## Dostop do brskalnika datotek

1. Odprite ali ustvarite projekt v Chloros
2. Kliknite ikono **Brskalnik datotek** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> v levem stranskem meniju
3. V oknu Brskalnik po datotekah se prikaže seznam datotek vašega projekta

{% namig style=&quot;info&quot; %}
**Podprti tipi datotek**: Chloros podpira datoteke RAW+JPG in JPG iz fotoaparatov MAPIR Survey3W in Survey3N. Priporočamo samo RAW+JPG.
{% endhint %}

***

## Dodajanje slik v projekt

Slike v projekt lahko dodate na dva načina:

### Način 1: Dodajanje datotek

To možnost uporabite za uvoz posameznih slikovnih datotek ali majhnega izbora datotek.

1. Kliknite gumb **„Dodaj datoteke“** <img src="../.gitbook/assets/image.png" alt="" data-size="line"> gumb na vrhu okna brskalnika datotek
2. Poiščite mapo, ki vsebuje vaše slike
3. Izberite eno ali več slikovnih datotek (za izbiro več datotek držite tipko **Ctrl**)
4. Kliknite **»Odpri«**, da uvozite izbrane datoteke

### Način 2: Dodajanje mape

To možnost uporabite za uvoz vseh slik iz mape naenkrat.

1. Kliknite gumb **„Dodaj mapo“** <img src="../.gitbook/assets/image (1).png" alt="" data-size="line"> na vrhu okna brskalnika datotek.
2. Poiščite in izberite mapo, ki vsebuje slike iz vaše snemalne seje.
3. Kliknite **»Izberi mapo«**, da uvozite vse podprte slike iz te mape.***

## Razumevanje tabele brskalnika datotek

Ko so slike uvožene, se prikažejo v tabeli z naslednjimi stolpci:

### Ime datoteke

* Izvirno ime datoteke iz kamere
* Ohranja konvencijo poimenovanja kamere (npr. IMG\_0001.RAW)

### Časovni žig

* Datum in čas zajema slike
* Izvlečeno iz metapodatkov EXIF slike
* Uporablja se za sinhronizacijo PPK in zaznavanje kalibracijskega cilja

### Model kamere

* Samodejno zaznavana konfiguracija kamere in filtra
* Primeri: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Uporablja se za uporabo pravilnih profilov obdelave

### Stolpec cilja (potrditveno polje)

* To polje potrdite za slike, ki vsebujejo kalibracijske cilje
* Močno pospeši zaznavanje ciljev med obdelavo
* Za podrobnosti glejte [Izbira ciljnih slik](choosing-target-images.md)

***

## Upravljanje datotek v projektu

### Odstranjevanje datotek

Če želite iz projekta odstraniti neželene slike:

1. Izberite eno ali več slik v tabeli brskalnika datotek
2. Kliknite gumb **»Odstrani izbrano«** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line"> .
3. Potrdite odstranitev (datoteke se ne izbrišejo z diska, ampak se samo odstranijo iz projekta).

### Razvrščanje in filtriranje

* **Razvrsti po stolpcu**: kliknite kateri koli naslov stolpca, da razvrstite slike.
* **Razvrsti po časovnem žigu**: koristno za organiziranje kronoloških zaporedij zajemanja.
* **Filter modela kamere**: združite slike po tipu kamere, če uporabljate več kamer.***

## Predogled slike

### Ogled celotne slike

Kliknite katero koli sličico slike v brskalniku datotek, da jo prikažete v glavnem predoglednem področju:

1. Slika se prikaže v osrednjem predoglednem oknu.
2. Uporabite gumbe za povečavo, da pregledate podrobnosti slike.
3. Med slikami se premikajte s puščičnimi tipkami.

### Hitro navajanje

* **Prejšnja slika**: kliknite levo puščico ali pritisnite tipko ←
* **Naslednja slika**: kliknite desno puščico ali pritisnite tipko →
* **Povečanje/pomanjšanje**: uporabite kolesce miške ali gumbe za povečavo
* **Premikanje**: kliknite in povlecite sliko, ko je povečana***

## Obravnavanje podvojenih datotek

Chloros samodejno zazna in prezre podvojene datoteke:

* Datoteke z enakimi imeni se preskočijo.
* Prepreči naključno dvojno obdelavo.
* Ob zaznavanju podvojenih datotek se prikaže opozorilno sporočilo.

{% hint style=&quot;warning&quot; %}
**Pomembno**: Pred uvozom ne preimenujte ali spreminjajte izvirnih slikovnih datotek. Chloros za pravilno obdelavo uporablja izvirna imena datotek in metapodatke.
{% endhint %}

***

## Mešani nizi podatkov kamere

Če vaš projekt vsebuje slike iz več kamer MAPIR:

1. Chloros samodejno zazna vsak model kamere.
2. Vsak tip kamere se obdela z ustreznim kalibracijskim profilom.
3. Brskalnik datotek prikaže model kamere v stolpcu Model kamere.
4. Obdelava uporabi pravilne nastavitve za vsak tip kamere.

**Primer scenarija**: Survey3W RGN + Survey3N OCN nastavitev z dvema kamerama.***

## Najboljše prakse

### Uredite pred uvozom

* Slike kalibracijskih ciljev shranite v isti mapi kot slike raziskave.
* Ohranite originalno strukturo map iz kamere/SD kartice.
* Ne mešajte podatkovnih nizov iz različnih sej v enem projektu.

### Poimenovanje datotek

* Ohranite originalna imena datotek kamere (IMG\_0001.RAW itd.).
* Datotek ne preimenujte pred uvozom.
* Originalna imena vsebujejo pomembne metapodatke.

### Slike za kalibracijo

* Vedno vključite 1–2 slike za kalibracijo na sejo.
* Posnemite cilje pred in po seji snemanja.
* Cilje postavite v enake svetlobne pogoje kot območje snemanja.
* Označite ciljne slike z izbirnim poljem Target, da pospešite obdelavo.

***

## Pogoste težave in rešitve

### Slike se po uvozu ne prikažejo

**Možni vzroki:**

* Datotečni format ni podprt (samo RAW+JPG in JPG iz kamer MAPIR)
* Slike so iz kamer, ki niso MAPIR (glej [Podprte kamere](../supported-cameras.md))
* Poškodovana datoteka ali nepopoln prenos s kartice SD

**Rešitev**: Preverite združljivost datotečnega formata in modela kamere.

### Model kamere ni zaznan

**Možni vzroki:**

* Spremenjeni metapodatki EXIF
* Slike, urejene v zunanji programski opremi
* Nepopoln prenos datotek

**Rešitev**: Ponovno uvozite originalne, nespremenjene datoteke iz kamere/SD kartice.

### Manjkajoči časovni žigi

**Možni vzroki:**

* Nepravilno nastavljena ura kamere
* Podatki EXIF odstranjeni z zunanjo programsko opremo

**Rešitev**: Preverite, ali so bile nastavitve časa kamere med zajemom pravilne.***

## Naslednji koraki

Ko so datoteke uvožene:

1. **Preglejte seznam datotek** – Preverite, ali so bile vse slike pravilno naložene.
2. **Preverite modele kamer** – Preverite, ali je kamera pravilno zaznana.
3. **Označite ciljne slike** – glejte [Izbira ciljnih slik](choosing-target-images.md)
4. **Prilagodite nastavitve** – konfigurirajte možnosti obdelave v [Nastavitvah projekta](adjusting-project-settings.md)
5. **Začnite obdelavo** – glejte [Začetek obdelave](starting-the-processing.md).

Podrobne informacije o konfiguraciji projekta najdete v [Prilagajanje nastavitev projekta](adjusting-project-settings.md).

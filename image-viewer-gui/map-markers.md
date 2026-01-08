# Oznake na zemljevidu

Zavihek Zemljevid prikaže vaše slike na interaktivnem 2D zemljevidu na podlagi njihovih GPS koordinat. To vam omogoča geografski pregled vaše zajemne seje in vam pomaga vizualizirati prostorsko pokritost. To je koristno tudi pri prvem uvozu slik, da lahko hitro odstranite slike, ki jih ne potrebujete za obdelavo.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Dostop do zavihka Zemljevid

1. Odprite ali ustvarite projekt v Chloros.
2. Uvozite slike, ki vsebujejo metapodatke GPS.
3. Kliknite zavihek **Zemljevid** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> v levem stranskem meniju.
4. Zemljevid bo prikazal označevalce na GPS lokaciji vsake slike.

{% hint style="info" %}
**Potreben GPS**: Na zemljevidu se bodo prikazale samo slike z vgrajenimi GPS-koordinatami v metapodatkih EXIF. Prepričajte se, da je med zajemanjem v vaši kameri omogočen GPS.
{% endhint %}

***

## Prilagajanje slik iz zavihka Zemljevid

Zavihek **Zemljevid** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> ima enake možnosti dodajanja  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  in odstranjevalna  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  kot [**Brskalnik datotek**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Prikaže tudi enak seznam projektnih datotek, vendar z drugačnimi naslovi stolpcev:

### Ime datoteke

* Izvirno ime datoteke iz kamere
* Ohranja konvencijo poimenovanja kamere (npr. IMG\_0001.RAW)

### Zemljepisna širina

* Zemljepisna širina slike

### Zemljepisna dolžina

* Zemljepisna dolžina slike

### Nadmorska višina

* Nadmorska višina slike

{% hint style="info" %}
S klikom na naslove stolpcev tabele se razvrstijo tudi podatki v vrsticah.
{% endhint %}

***

## Oznake slik

Vsaka slika z GPS-podatki je na zemljevidu označena z oznako:

### Prikaz oznak

* Oznake označujejo natančne GPS-koordinate, kjer je bila posneta vsaka slika.
* Oznake se lahko združijo v skupine, ko se slika pomanjša.
* Povečajte sliko, da vidite posamezne lokacije slik.

{% hint style="success" %}
SUPER-ZOOM: Ko dosežete največjo stopnjo povečave, ki jo omogoča ponudnik zemljevidnih ploščic, se ploščica pri nadaljnjem povečanju poveča, kar vam omogoča, da vidite oznake, ki so blizu skupaj.
{% endhint %}

### Predogled ob preletu

* **Preletite z miško** nad katerim koli označevalcem, da si ogledate predogled slike v obliki miniaturne slike.
* To omogoča hitro vizualno prepoznavanje, ne da bi zapustili pogled zemljevida.
* Koristno za iskanje določenih slik v obsežni seriji posnetkov.

***

## Ponudniki zemljevidnih ploščic

{% hint style="success" %}
**Samodejna izbira**: Chloros samodejno izbere storitev kartografskih ploščic, ki zagotavlja najboljšo stopnjo povečave za vašo trenutno lokacijo na zemljevidu. Po želji lahko ročno preklopite med ponudniki.
{% endhint %}

Zavihek Zemljevid podpira dva ponudnika kartografskih ploščic za slike zemljevida v ozadju:

### Google Maps

* Standardne satelitske in zemljevidske slike iz Googla.
* Najboljše za splošno pokritost po vsem svetu.

### ESRI

* Satelitske in letalske slike iz ESRI ArcGIS.
* Pogosto zagotavlja slike z višjo ločljivostjo v določenih regijah.

***

## Vrste kartografskih ploščic

Izberete lahko vrsto kartografske plasti (od leve proti desni):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="line">### Teren

Prikaže višinske profile in kartografske ploščice s podrobnostmi (ceste itd.)

### Zemljevid

Prikaže standardne (manjša pasovna širina) kartografske ploščice s podrobnostmi (ceste itd.)

### Satelit

Prikaže podrobne (večja pasovna širina) satelitske kartografske ploščice

### Hibrid

Prikaže satelitske kartografske ploščice z dodatnimi podrobnostmi (ceste itd.)

***

## Navigacija po zemljevidu

### Nadzorni elementi za povečavo

* **Povečava/pomanjšanje**: uporabite kolesce miške ali gumbe za povečavo
* **Celozaslonski način**: zemljevid v celozaslonskem načinu

### Nadzorni elementi za premikanje

* **Premikanje**: kliknite in povlecite, da se premikate po zemljevidu

***

## Primeri uporabe

### Vizualizacija letalske poti

* Ogled pokritega območja snemanja z dronom
* Prepoznavanje vrzeli v pokritosti slike
* Preverjanje izvedbe letalske poti

### Pregled zemeljskega pregleda

* Ogled prostorske porazdelitve zemeljskih posnetkov
* Lociranje kalibracijskih ciljnih slik glede na območje pregleda
* Načrtovanje dodatnih lokacij snemanja

### Nadzor kakovosti

* Hitro prepoznajte slike, posnete na nepričakovanih lokacijah.
* Preverite natančnost GPS v celotnem nizu podatkov.
* Primerjajte lokacije slik s terenskimi zapisi.

***

## Odpravljanje težav

### Oznake se ne prikažejo

**Možni vzroki:**

* Slike ne vsebujejo metapodatkov GPS.
* GPS je bil med snemanjem izklopljen na kameri.
* Podatki EXIF so bili odstranjeni z zunanjo programsko opremo.

**Rešitev**: Preverite, ali je GPS v kameri omogočen, in ponovno uvozite izvirne datoteke.

### Oznake na napačni lokaciji

**Možni vzroki:**

* GPS kamere je imel slabo satelitsko povezavo.
* GPS se je med zajemom premaknil.

**Rešitev**: To je običajno težava v času zajema; za natančne aplikacije razmislite o uporabi PPK/RTK GPS.

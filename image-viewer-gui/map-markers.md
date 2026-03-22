# Oznake na zemljevidu

Zavihek »Zemljevid« prikaže vaše slike na interaktivnem 2D-zemljevidu na podlagi njihovih GPS-koordinat. To vam omogoča geografski pregled nad vašo snemalno sejo in pomaga vizualizirati prostorsko pokritost. Uporaben je tudi ob prvem uvozu slik, saj omogoča hitro odstranitev vseh slik, ki jih ni treba obdelati.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Dostop do zavihka Zemljevid

1. Odprite ali ustvarite projekt v Chloros
2. Uvozite slike, ki vsebujejo metapodatke GPS
3. Kliknite zavihek **Zemljevid** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> v levem stranskem meniju
4. Zemljevid bo prikazal označevalce na GPS-lokaciji vsake slike

{% hint style="info" %}
**Potreben GPS**: Na zemljevidu se bodo prikazale le slike z vgrajenimi GPS-koordinatami v metapodatkih EXIF. Prepričajte se, da je med zajemanjem v vaši kameri omogočen GPS.
{% endhint %}

***

## Prilagajanje slik iz zavihka Zemljevid

Zavihek **Zemljevid**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> ima enako gumb za dodajanje  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  in odstranjevanja  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  kot jih ima zavihek [**Brskalnik datotek**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Prikazuje tudi isti seznam projektnih datotek, vendar z drugačnimi naslovi stolpcev:

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
S klikom na naslove stolpcev v tabeli se podatki v vrsticah tudi razvrstijo
{% endhint %}

***

## Oznake slik

Vsaka slika z GPS-podatki je na zemljevidu označena z oznako:

### Prikaz oznak

* Oznake označujejo natančne GPS-koordinate, kjer je bila posneta vsaka slika
* Zgoščene oznake se lahko združijo, ko se oddaljite
* Približajte, da vidite lokacije posameznih slik

{% hint style="success" %}
SUPER-ZOOM: Ko dosežete najvišjo stopnjo povečave od ponudnika kartografskih ploščic, se ploščica pri nadaljnjem povečanju še dodatno poveča, kar vam omogoča, da vidite oznake, ki so blizu skupaj.
{% endhint %}

### Predogled ob preletu

* **Preletite z miško** kateri koli označevalec, da si ogledate predogled slike v obliki sličice
* To omogoča hitro vizualno prepoznavanje, ne da bi zapustili pogled na zemljevid
* Koristno za iskanje določenih slik znotraj obsežne seje zajemanja

***

## Ponudniki kartografskih ploščic

{% hint style="success" %}
**Samodejna izbira**: Chloros samodejno izbere storitev ploščic, ki zagotavlja najboljšo stopnjo povečave za vašo trenutno lokacijo na zemljevidu. Po želji lahko ročno preklopite med ponudniki.
{% endhint %}

Zavihek Zemljevid podpira dva ponudnika ploščic za slike v ozadju zemljevida:

### Google Maps

* Standardne satelitske in zemljevidne slike iz Googla
* Najboljše za splošno pokritost po vsem svetu

### ESRI

* Satelitske in letalske slike iz ESRI ArcGIS
* V nekaterih regijah pogosto zagotavlja slike z višjo ločljivostjo

***

## Vrste kartografskih ploščic

Izberete lahko vrsto kartografske plasti (od leve proti desni):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="original">### Terén

Prikaže višinske profile in kartografske ploščice s podrobnostmi (ceste itd.)

### Zemljevid

Prikaže standardne (manjša pasovna širina) kartografske ploščice s podrobnostmi (ceste itd.)

### Satelit

Prikaže podrobne (večja pasovna širina) satelitske kartografske ploščice

### Hibrid

Prikaže satelitske kartografske ploščice z dodatnimi podrobnostmi (ceste itd.)

***

## Navigacija po zemljevidu

### Upravljalniki povečave

* **Povečaj/Pomanjšaj**: Uporabite kolesce miške ali gumbe za povečavo
* **Celozaslonski način**: Prikaži zemljevid v celozaslonskem načinu

### Upravljalniki premikanja

* **Premikanje**: Kliknite in povlecite, da se premikate po zemljevidu***

## Primeri uporabe

### Vizualizacija poti leta

* Ogled območja pokritosti posnetkov, zajetih z dronom
* Prepoznavanje vrzeli v pokritosti slike
* Preverjanje izvedbe poti leta

### Pregled terenskega pregleda

* Ogled prostorske porazdelitve posnetkov, zajetih s tal
* Lociranje kalibracijskih ciljnih slik glede na območje pregleda
* Načrtovanje dodatnih lokacij zajemanja

### Nadzor kakovosti

* Hitro prepoznajte slike, posnete na nepričakovanih lokacijah
* Preverite natančnost GPS v celotnem nizu podatkov
* Primerjajte lokacije slik s terenskimi zapisi

***

## Odpravljanje težav

### Oznake se ne prikažejo

**Možni vzroki:**

* Slike ne vsebujejo metapodatkov GPS
* Med snemanjem je bil GPS na kameri izklopljen
* Podatki EXIF so bili odstranjeni z zunanjo programsko opremo

**Rešitev**: Preverite, ali je GPS na vaši kameri omogočen, in ponovno uvozite izvirne datoteke

### Oznake na napačni lokaciji

**Možni vzroki:**

* GPS kamere je imel slabo satelitsko lokacijo
* Odklon GPS med zajemanjem

**Rešitev**: To je običajno težava v času zajemanja; razmislite o uporabi PPK/RTK GPS za natančne aplikacije

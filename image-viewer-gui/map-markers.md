# Oznake na zemljevidu

Zavihek »Zemljevid« prikaže vaše slike na interaktivnem 2D-zemljevidu na podlagi njihovih GPS-koordinat. Omogoča vam geografski pregled snemalne seje in je najhitrejši način, takoj po uvozu, da odstranite slike, ki jih ne želite obdelati.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Dostop do zavihka »Zemljevid«

1. Odprite ali ustvarite projekt v Chloros
2. Uvozite slike, ki vsebujejo metapodatke GPS
3. V levem stranskem meniju kliknite zavihek **Zemljevid** <img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line">
4. Na zemljevidu se na lokaciji GPS vsake slike prikaže označevalec

{% hint style="info" %}
**Potreben je GPS**: na zemljevidu se prikažejo le slike, ki imajo v svojih EXIF-metapodatkih GPS-koordinate. Slika brez koordinat je še vedno v projektu in se obdeluje normalno – preprosto nima označevalca.
{% endhint %}

***

## Prilagajanje slik na zavihku »Zemljevid«

Zavihek **Zemljevid**<img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> ima enake gumbe za dodajanje <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> in odstranjevanje <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line"> datotek kot zavihek [**Brskalnik datotek**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">. Prikazuje isti seznam projektnih datotek z geografskimi stolpci:

| Stolpec        | Vsebina                                                           |
| ------------- | ------------------------------------------------------------------ |
| **Ime**      | Ime datoteke, kot je bilo shranjeno iz fotoaparata                             |
| **Zemljepisna širina** | Decimalne stopinje, šest decimalnih mest                                |
| **Dolžina** | Decimalne stopinje, šest decimalnih mest                                |
| **Nadmorska višina** | Metri, eno decimalno mesto — `-`, če slika ne vsebuje podatka o nadmorski višini |

{% hint style="info" %}
Kliknite kateri koli naslov stolpca, da razvrstite po njem; kliknite še enkrat, da obrnete vrstni red.
{% endhint %}

{% hint style="warning" %}
**Nadmorska višina je višina nad morsko gladino, ne višina nad tlemi.** Vrednost izhaja iz EXIF-oznake slike `GPSAltitude`, ki se nanaša na povprečno morsko gladino. To ni višina leta nad terenom, zato Chloros iz te vrednosti ne bo izračunal razdalje med vzorci na tleh – nad poljem, ki leži 300 m nad morsko gladino, bo dron na višini 100 m AGL tukaj zabeležil približno 400 m. Stolpec uporabite za odkrivanje izjemnih vrednosti in potrditev dosledne višine leta, ne pa kot meritev AGL.
{% endhint %}

***

## Oznake slik

Vsaka slika z GPS-podatki dobi oznako na svojih koordinatah.

### Prikaz označevalcev

* Označevalci se nahajajo na natančnih koordinatah, zabeleženih za vsak posnetek
* Označevalci, ki so blizu drug drugega, se lahko vizualno prekrivajo, če sliko pomanjšate – povečajte sliko, da jih ločite
* Izbrani in poudarjeni označevalci so prikazani nad ostalimi

### Predogled ob preletu z miško

* **Preletite** kateri koli označevalec, da se prikaže miniatura te slike z imenom datoteke
* **Kliknite**na oznako, da izberete sliko in**pritrdite** pojavno okno – ostane odprto, dokler ne kliknete drugam. Medtem ko je pojavno okno pritrjeno, ga premikanje miške nad drugimi oznakami ne prekriva
* To je hiter način za iskanje posameznega posnetka v obsežni seji, ne da bi zapustili zemljevid

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption><p>Zavihek »Zemljevid« prikaže vse slike z geografskimi oznakami v projektu</p></figcaption></figure>### Super-zoom

{% hint style="success" %}
**SUPER-ZOOM**: ko dosežete največjo stopnjo povečave, za katero ponudnik ploščic ponuja slike, se s nadaljnjim povečanjem ploščice še naprej povečajo, namesto da bi se ustavile, tako da lahko ločite oznake, ki ležijo skoraj ena na drugi.
{% endhint %}

* Super-zoom se vklopi le, ko ste **na** največji stopnji povečave, ki jo ponudnik omogoča za to lokacijo, in ko se je nalaganje ploščic zaključilo. Pod to stopnjo povečave deluje povečava normalno
* Obseg je **od 1× do 32×** nad največjo stopnjo povečave, ki jo omogoča ponudnik
* Kazalnik v kotu prikazuje trenutno stopnjo super povečave v odstotkih, gumb **×** zraven pa vas z enim klikom vrne na običajno povečavo
* Zmanjševanje povečave se vedno prenese na sam zemljevid, zato se nikoli ne morete zatakniti v super povečavi
* Povečanje in premikanje med super-povečavo prenese nastali odmik nazaj na zemljevid, tako da območje, ki ni v središču in na katerega ste se premaknili, še naprej zahteva ploščice, namesto da bi postalo prazno
* Oznake so narisane kot vektorski elementi in ne kot rastrirane, zato ostanejo ostre na vsaki stopnji super-povečave

***

## Ponudniki kartografskih ploščic

{% hint style="success" %}
**Samodejna izbira**: Chloros izbere storitev ploščic, ki ponuja najboljšo stopnjo povečave za lokacijo, kjer se nahajajo vaše slike. Kadarkoli lahko ročno preklopite.
{% endhint %}

| Ponudnik        | Opombe                                                                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Google Maps** | Široko svetovno pokritje; podpira vse štiri vrste ploščic                                                                                                            |
| **Esri ArcGIS**| Pogosto zračne slike z višjo ločljivostjo v določenih regijah. Vrsta ploščic**Terrain** ni na voljo za Esri, zato je gumb za njo onemogočen, dokler je izbran Esri |***

## Vrste kartografskih ploščic

Izberite vrsto kartografske plasti s pomočjo gumbov (od leve proti desni):

![](<../.gitbook/assets/image (14).png>)

| Vrsta                 | Prikaže                                                                |
| -------------------- | -------------------------------------------------------------------- |
| **Teren**          | Senčenje višine z zemljevidnimi podrobnostmi (ceste, oznake). Samo Google       |
| **Zemljevid**              | Standardni zemljevidni ploščki – možnost z najmanjšo porabo pasovne širine              |
| **Satelit**        | Podrobne satelitske slike, brez oznak — možnost z največjo porabo pasovne širine |
| **Hibrid** (privzeto) | Satelitske slike s cestami in oznakami, narisanimi nanje                |

Zavihek »Zemljevid« se odpre v načinu **Hibridni**. Vaša izbira se prenese tudi na druge ponudnike, če ti to podpirajo.***

## Navigacija po zemljevidu

* **Povečava**: kolesce miške ali gumbi za povečavo na zemljevidu
* **Premikanje**: kliknite in povlecite
* **Celozaslonski način**: gumb za celozaslonski način razširi zemljevid na celotno okno***

## Primeri uporabe

### Pregled poti leta

* Na prvi pogled si oglejte območje pokritosti posamezne seje z dronom
* Odkrijte vrzeli, kjer je bil prelet izpuščen
* Preverite, ali je let potekal po načrtovani poti

### Pregled terenskega merjenja

* Oglejte si, kako so razporejene posnetke s tal
* Poiščite okvirje kalibracijskih ciljev glede na območje merjenja
* Odločite, kje so potrebni dodatni posnetki

### Nadzor kakovosti

* Poiščite posnetke, posnete na nepričakovanih mestih, in jih odstranite pred obdelavo
* Razvrstite po nadmorski višini, da odkrijete posnetek, posnet na napačni višini, ali takšnega, pri katerem je bila lokacija GPS slabo določena
* Primerjajte lokacije posnetkov s terenskimi zapisi

***

## Odpravljanje težav

### Oznake se ne prikažejo

**Možni vzroki**

* Slike ne vsebujejo metapodatkov GPS
* Med zajemanjem je bil GPS na kameri onemogočen
* Podatki EXIF so bili pred uvozom odstranjeni z drugo programsko opremo

**Kaj storiti**: preverite, ali je GPS na fotoaparatu omogočen, in ponovno uvozite izvirne datoteke. Ali ima določena datoteka koordinate, lahko preverite v tabeli datotek na zavihku »Zemljevid« – slika brez koordinat tam nima vrstice.

### Oznake so na napačnem mestu

**Možni vzroki**: slaba satelitska lokacija ob zajemanju ali odstopanje GPS-a med sejo.**Kaj storiti**: to je težava, povezana z zajemom, in ne nekaj, kar lahko Chloros popravi naknadno. Za natančno delo uporabite delovni tok PPK/RTK GPS – glejte nastavitev**Uporabi PPK popravke** v [Nastavitvah projekta](../project-settings/project-settings.md).

### Zemljevid je prazen ali se ploščice ne nalagajo več

Ponudniki ploščic so spletne storitve. Če se ploščice ne nalagajo več, preverite omrežno povezavo naprave, nato pa poskusite zamenjati ponudnika. Če ste močno povečali prikaz, pritisnite gumb za ponastavitev **×**, da se vrnete na normalno stopnjo povečave in omogočite zemljevidu, da ponovno zahteva ploščice.***

## Povezane strani

* [**Mreža slik**](image-grid.md) — isti nabor slik kot pri sličicah
* [**Odpiranje slike v polnem zaslonu**](opening-an-image-full-screen.md) — podrobno pregledovanje posamezne slike
* [**Dodajanje datotek v projekt**](../processing-images-gui/adding-files-to-a-project.md) — gumbi za dodajanje/odstranjevanje datotek, ki so na voljo na tem zavihku

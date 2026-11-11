# GUI: Navigacija

Ko prvič zaženete program Chloros, se zažene njegov procesni backend. Ko je backend pripravljen, se v zgornjem levem kotu prikaže ikona glavnega menija <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">, v levem stranskem meniju pa se odklenejo zavihki »Kamere« in »Svetlobni senzorji« (do takrat so sivi).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Zgornji naslovni trak od leve proti desni vsebuje:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Glavni meni

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>V glavnem meniju lahko:

* **Nov projekt**— ustvarite nov projekt. Če imate shranjene predloge projektov, se prikaže spustni meni**Izberi predlogo**, tako da se nov projekt začne z nastavitvami predloge.
* **Odpri projekt**— odprete obstoječi projekt. Seznam vključuje gumb**Odpri mapo projekta**, ki v vašem raziskovalcu datotek odpre mapo s projekti.
* **Podvojiti projekt** — kopirajte trenutno odprt projekt pod novim imenom (predlagano je prosto ime, npr. »MojProjekt (2)«) in odprite kopijo. _(vidno po odprtju projekta)_
* **Dodaj datoteke** — dodajte posamezne slikovne datoteke v trenutni projekt _(vidno po odprtju projekta)_
* **Dodaj mapo** — dodajte eno ali več map s slikami v trenutni projekt _(vidno po odprtju projekta)_
* **Začni obdelavo / Ustavi obdelavo** — zaženi ali ustavi potek obdelave slik _(omogočeno po dodajanju datotek)_
* **Poveži s kamero** — preide na [zavihek Kamere](lattice/), da poveže kamero ali niz kamer LATTICE. Deluje tudi brez odprtega projekta.
* **Poveži se s svetlobnim senzorjem** — preide na [zavihek Svetlobni senzorji](daq/), da se poveže s svetlobnim senzorjem DAQ. Deluje tudi brez odprtega projekta.

{% hint style="info" %}
**Samo za Windows**: Grafični uporabniški vmesnik Chloros

Desktop je na voljo v sistemu Windows

. Uporabniki Linux

a naj za obdelavo brez grafičnega vmesnika preberejo dokumentacijo o [CLI

](CLI.md) in [Python

SDK

](api-python-sdk.md).
{% endhint %}

### <img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">

Gumb za predvajanje/začetek

Ko je omogočen, gumb za začetek obdelave sproži potek obdelave slik.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line">

Kazalnik napredka <img src=".gitbook/assets/image (5).png" alt="" data-size="line">



V brezplačnem načinu Chloros

, ki vse datoteke obdeluje zaporedno, kazalnik napredka prikazuje 2 fazi: zaznavanje ciljev in obdelava.

V plačljivem načinu z licenco Chloros

+, ki vse datoteke obdeluje sočasno, kazalnik napredka prikazuje 4 faze: zaznavanje, analiziranje, kalibriranje, izvoz. Če z miškinim kazalcem preletite kazalnik napredka Chloros

+, se odpre razširjen panel s štirimi kazalniki napredka, da lahko spremljate potek. S klikom na zgornji kazalnik napredka zamrznete razširjeni panel, s ponovnim klikom pa ga sprostite.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Stranski meni

Levi stranski meni vsebuje različne ikone za interakcijo, v tem vrstnem redu od zgoraj navzdol:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Nastavitve projekta](project-settings/project-settings.md)

Zavihek »Nastavitve projekta« vam omogoča prilagajanje splošnih nastavitev projekta in nastavitev obdelave. Te nastavitve prilagodite, preden začnete obdelovati datoteke.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Brskalnik datotek

Dodajajte datoteke/mape in odstranjujte datoteke iz projekta. Podvojene datoteke se prezrejo. Označite polje v stolpcu »cilj« za katero koli ciljno sliko, pri čemer bo obdelava upoštevala le označene slike kot cilje, kar znatno pospeši čas obdelave. Z gumbom »Slika/Metapodatki« lahko preklapljate med prikazom mreže sličic izbrane slike in podrobno tabelo metapodatkov.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Pregledovalnik slik](image-viewer-gui/opening-an-image-full-screen.md)

Ko v glavnem pregledovalniku slik kliknete na sliko, se ta odpre v polnem zaslonu na zavihku »Pregledovalnik slik«.

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Pregledovalnik zemljevidov](image-viewer-gui/map-markers.md)

Slike si oglejte na interaktivnem 2D-zemljevidu na podlagi njihovih GPS-koordinat. Podpira ponudnike kart Google Maps in ESRI, pri čemer samodejno izbere najboljšo storitev za vašo lokacijo. Preletite z miško nad označevalci, da si ogledate predoglede sličic.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Kamere](lattice/)

Povežite in upravljajte kamere LATTICE v živo — po eno naenkrat ali kot sinhronizirane skupine več kamer. Zavihek prikazuje ploščice s predogledom v živo s prekrivnimi sloji in histogrami, nastavitve po posamezni kameri in po nizu ter nastavitve zajemanja, ki določajo, katere kamere in vrste izvoza ustvari funkcija »Zajemi vse«. Na voljo, ko bo strežniški del pripravljen; za celoten vodnik glejte [oddelek LATTICE](lattice/).

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Svetlobni senzorji](daq/)

Povežite svetlobne senzorje DAQ — DAQ-U (USB), DAQ-M (Bluetooth) in DAQ-E (Ethernet) — ter si oglejte njihove kalibrirane spektralne grafe v realnem času v W/m²/nm. Od tu lahko v odprt projekt snemate datoteke `.daq`, preimenujete senzorje, izberete profile korekcije kapacitivnosti in posodobite firmware DAQ-E. Na voljo bo, ko bo backend pripravljen; za celoten vodnik glejte [razdelek DAQ](daq/).

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Dnevnik odpravljanja napak

V primeru težav preglejte dnevnik za izpise odpravljanja napak. Kopirajte/prenesite dnevnik in ga pošljite [podpori MAPIR](https://www.mapir.camera/community/contact) za pomoč.

Novost v različici 1.3.0: ikona napake na vrhu zavihka z dnevnikom (**Prijavi napako**) pošlje podjetju MAPIR poročilo z vašim opisom, do 10 slikami, dnevniki Chlorosa in podrobnostmi o sistemu. Za pošiljanje morate biti prijavljeni. Glejte [Prijava napake iz zavihka z dnevnikom](processing-images-gui/monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Prijava uporabnika](chloros+-login.md)

Stranski pas za prijavo uporabnika vam omogoča prijavo v vaš račun Chloros+, da odklenete napredne funkcije. Prav tako lahko pregledate trenutno različico aplikacije ter prilagodite jezik prikazanega besedila v grafičnem vmesniku Chloros in v CLI-ju.

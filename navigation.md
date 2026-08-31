# GUI: Navigacija

Ko prvič zaženete program Chloros, se zažene njegov procesni backend. Ko je backend pripravljen, se v zgornjem levem kotu prikaže ikona glavnega menija <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">, v levem stranskem meniju pa se odklenejo zavihki »Kamere« in »Svetlobni senzorji« (do takrat so sivi).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Zgornji naslovni trak od leve proti desni vsebuje:

### Glavni meni »<img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">«

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

V glavnem meniju lahko:

* **Nov projekt**— ustvarite nov projekt. Če imate shranjene predloge projektov, se prikaže spustni meni**Izberi predlogo**, tako da se nov projekt začne z nastavitvami predloge.
* **Odpri projekt**— odprete obstoječi projekt. Seznam vključuje gumb**Odpri mapo projekta**, ki v vašem raziskovalcu datotek odpre mapo s projekti.
* **Podvajaj projekt** — kopirajte trenutno odprt projekt pod novim imenom (predlagano je prosto ime, kot je »MojProjekt (2)«) in odprite kopijo. _(vidno po odprtju projekta)_
* **Dodaj datoteke** — dodajte posamezne slikovne datoteke v trenutni projekt _(vidno po odprtju projekta)_
* **Dodaj mapo** — dodajte eno ali več map s slikami v trenutni projekt _(vidno po odprtju projekta)_
* **Začni obdelavo / Ustavi obdelavo** — zaženi ali ustavi potek obdelave slik _(omogočeno po dodajanju datotek)_
* **Poveži s kamero** — preide na [zavihek Kamere](lattice/), da poveže kamero ali niz kamer LATTICE. Deluje tudi brez odprtega projekta.
* **Poveži se s svetlobnim senzorjem** — preide na [zavihek Svetlobni senzorji](daq/), da se poveže z DAQ svetlobnim senzorjem. Deluje tudi brez odprtega projekta.

{% hint style="info" %}
**Samo Windows**: Namizni grafični vmesnik Chloros je na voljo na Windows. Uporabniki modela Linux naj si ogledajo dokumentacijo za [CLI](CLI.md) in [Python SDK](api-python-sdk.md) za obdelavo brez grafičnega vmesnika.
{% endhint %}

### Gumb »<img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">

« (Predvajaj/Začni)

Ko je omogočen, gumb za začetek obdelave sproži potek obdelave slik.

### Napredovalna vrstica »<img src=".gitbook/assets/image (4).png" alt="" data-size="line">

« ()<img src=".gitbook/assets/image (5).png" alt="" data-size="line">

V brezplačnem načinu Chloros, ki vse datoteke obdeluje zaporedno, napredovalna vrstica prikazuje 2 fazi: zaznavanje ciljev in obdelavo.

V plačljivem licenčnem načinu Chloros+, ki vse datoteke obdeluje sočasno, vrstica napredka prikazuje 4 faze: zaznavanje, analiziranje, kalibriranje in izvoz. Če z miškinim kazalcem preletite kazalec napredka Chloros+, se odpre razširjen panel s 4 kazalci napredka, da lahko spremljate potek. S klikom na zgornji kazalec napredka zamrznete razširjeni panel, s ponovnim klikom pa ga sprostite.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Stranski meni

Levi stranski meni vsebuje različne ikone za interakcijo, v tem vrstnem redu od zgoraj navzdol:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Nastavitve projekta](project-settings/project-settings.md)

Zavihek »Nastavitve projekta« vam omogoča prilagajanje splošnih nastavitev projekta in nastavitev obdelave projekta. Te nastavitve prilagodite, preden začnete obdelovati datoteke.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Brskalnik datotek

Dodajte datoteke/mape in odstranite datoteke iz projekta. Podvojene datoteke se prezrejo. Označite polje v stolpcu »cilj« za katero koli ciljno sliko, pri čemer bo obdelava upoštevala le označene slike kot cilje, kar znatno pospeši čas obdelave. Z gumbom »Slika/Metapodatki« lahko preklapljate med prikazom mreže sličic izbrane slike in podrobno tabelo metapodatkov.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Pregledovalnik slik](image-viewer-gui/opening-an-image-full-screen.md)

Ko v glavnem pregledovalniku slik kliknete na sliko, se ta odpre v polnem zaslonu v zavihku »Image Viewer«.

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Pregledovalnik zemljevidov](image-viewer-gui/map-markers.md)

Slike si oglejte na interaktivnem 2D-zemljevidu na podlagi njihovih GPS-koordinat. Podpira ponudnike kartografskih ploščic Google Maps in ESRI ter samodejno izbere najboljšo storitev za vašo lokacijo. Preletite z miško čez označevalce, da si ogledate predoglede sličic.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Kamere](lattice/)

Povežite in nadzorujte kamere LATTICE v živo — po eno naenkrat ali kot sinhronizirane skupine več kamer. Zavihek prikazuje ploščice s predogledom v živo s prekrivnimi sloji in histogrami, nastavitve za posamezno kamero in za sistem ter nastavitve zajemanja, s katerimi izberete, katere kamere in vrste izvoza ustvari funkcija »Capture All«. Na voljo, ko bo strežniški del pripravljen; za celoten vodnik glejte [oddelek LATTICE](lattice/).

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Svetlobni senzorji](daq/)

Povežite svetlobne senzorje DAQ — DAQ-U (USB), DAQ-M (Bluetooth) in DAQ-E (Ethernet) — ter si oglejte njihove kalibrirane spektralne grafe v realnem času v W/m²/nm. Od tu lahko v odprt projekt snemate datoteke `.daq`, preimenujete senzorje, izbirate profile korekcije pokrovčka in posodabljate firmware DAQ-E. Na voljo, ko bo backend pripravljen; za celoten vodnik glejte [razdelek DAQ](daq/).

#### Dnevnik odpravljanja napak »<img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line">«

V primeru težav preglejte dnevnik za izpise odpravljanja napak. Kopirajte/prenesite dnevnik in ga pošljite na [podporo MAPIR](https://www.mapir.camera/community/contact) za pomoč.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Prijava uporabnika](chloros+-login.md)

Stranski pas za prijavo uporabnika vam omogoča, da se prijavite v svoj račun Chloros+ in tako odklenete napredne funkcije. Prav tako lahko pregledate trenutno različico aplikacije ter nastavite jezik prikazanega besedila v grafičnem vmesniku Chloros in CLI.

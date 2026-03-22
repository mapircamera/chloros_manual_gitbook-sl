# GUI: Navigacija

Ko prvič zaženete Chloros in Chloros (Brskalnik), se bo zagnal njihov strežniški del. Ko bo pripravljen, se bo prikazala ikona glavnega menija v zgornjem levem kotu <img src=".gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

Zgornji naslovnik vsebuje od leve proti desni:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Glavni meni

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

V glavnem meniju lahko:

* **Nov projekt** — ustvarite nov projekt
* **Odpri projekt** — odprete obstoječi projekt
* **Odpri mapo projekta** — odprete mapo projekta v raziskovalcu datotek
* **Dodaj datoteke** — dodate posamezne slikovne datoteke v trenutni projekt _(vidno po odprtju projekta)_
* **Dodaj mapo** — dodajte mapo s slikami v trenutni projekt _(vidno po odprtju projekta)_
* **Začni obdelavo / Ustavi obdelavo** — začnite ali ustavite potek obdelave slik _(omogočeno po dodajanju datotek)_

{% hint style="info" %}
**Samo Windows**: Namizni grafični vmesnik Chloros je na voljo v Windows. Uporabniki Linux naj si ogledajo [CLI](CLI.md) in [Python SDK](api-python-sdk.md) za obdelavo brez grafičnega vmesnika.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1).png" alt="" data-size="line"> Gumb Predvajaj/Začni

Ko je omogočen, gumb za začetek obdelave sproži potek obdelave slike.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> Indikator napredka <img src=".gitbook/assets/image (5).png" alt="" data-size="line">V brezplačnem načinu Chloros, ki obdeluje vse datoteke zaporedno, kazalnik napredka prikazuje 2 fazi: zaznavanje cilja in obdelava.

V plačljivem načinu Chloros+, ki obdeluje vse datoteke hkrati, kazalnik napredka prikazuje 4 faze: zaznavanje, analiziranje, kalibriranje, izvoz. Če z miško premaknete kazalec nad kazalnik napredka Chloros+, se odpre razširjen panel s 4 kazalniki napredka, da lahko spremljate napredek. S klikom na zgornji kazalnik napredka zamrznete razširjeni panel, s ponovnim klikom ga sprostite.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Stranski meni

Levi stranski meni vsebuje različne ikone za interakcijo:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Nastavitve projekta](project-settings/project-settings.md)

Zavihek Nastavitve projekta vam omogoča prilagajanje splošnih nastavitev projekta in nastavitev obdelave projekta. Te nastavitve prilagodite pred začetkom obdelave datotek.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Brskalnik datotek

Dodajajte datoteke/mape in odstranjujte datoteke iz projekta. Podvojene datoteke se ignorirajo. Označite polje v stolpcu cilj za katero koli ciljno sliko, in obdelava bo upoštevala le označene slike za cilje, kar znatno pospeši čas obdelave. Uporabite stikalo Slika/Metapodatki za preklop med ogledom mreže sličic izbrane slike in podrobno tabelo metapodatkov.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Pregledovalnik slik](image-viewer-gui/opening-an-image-full-screen.md)

Ko v glavnem pregledovalniku slik kliknete na sliko, se ta odpre v polnem zaslonu v zavihku Pregledovalnik slik.

#### <img src=".gitbook/assets/image (7).png" alt="" data-size="line"> [Zemljevid](image-viewer-gui/map-markers.md)

Oglejte si slike na interaktivnem 2D zemljevidu na podlagi njihovih GPS-koordinat. Podpira ponudnike Google Maps in ESRI, pri čemer samodejno izbere najboljšo storitev za vašo lokacijo. Preletite z miško nad označevalci, da si ogledate predoglede sličic slik.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Dnevnik odpravljanja napak

V primeru težav preglejte dnevnik za izpise odpravljanja napak. Kopirajte/prenesite dnevnik in ga pošljite [MAPIR podpori](https://www.mapir.camera/community/contact) za pomoč.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Prijava uporabnika](chloros+-login.md)

Stranski trak za prijavo uporabnika vam omogoča prijavo v račun Chloros+, da odklenete napredne funkcije. Prav tako lahko pregledate trenutno različico aplikacije ter prilagodite jezik prikazanega besedila v grafičnem vmesniku Chloros in CLI.

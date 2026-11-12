# Dodajanje datotek v projekt

Ko v programu Chloros ustvarite ali odprete projekt, je naslednji korak dodajanje multispektralnih slik, da lahko začnete z obdelavo. Zavihek »File Browser«<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> omogoča enostaven uvoz slik in upravljanje z vašim naborom podatkov.

## Dostop do brskalnika datotek

1. Odprite ali ustvarite projekt v programu Chloros
2. Kliknite ikono **Brskalnik datotek** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> v levem stranskem meniju
3. V oknu »Brskalnik datotek« se bo prikazal seznam datotek vašega projekta

{% hint style="info" %}
**Podprte vrste datotek**:

* **Survey3W / Survey3N**: pari datotek RAW+JPG in slike JPG (priporočljivo RAW+JPG)
* **LATTICE**: posnetki `.tif` / `.tiff` — posneti s programom za nadzor kamere Chloros ali prek vozlišča LATTICE
* **Podatki svetlobnega senzorja**: posnetki `.daq` (DAQ-U/M/E) in dnevniki navzdolnega sevanja DAQ-M `.csv` — uvoženi skupaj s slikami za kalibracijo odbojnosti
{% endhint %}

***

## Dodajanje slik v projekt

Obstajata dva glavna načina za dodajanje slik v projekt:

### Metoda 1: Dodajanje datotek

To možnost uporabite za uvoz posameznih slikovnih datotek ali manjšega izbora datotek.

1. Kliknite gumb **»Dodaj datoteke«** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> na vrhu okna brskalnika datotek
2. Poiščite mapo, ki vsebuje vaše slike
3. Izberite eno ali več slikovnih datotek (za izbiro več datotek držite tipko **Ctrl**)
4. Kliknite **»Odpri«**, da uvozite izbrane datoteke

### Način 2: Dodajanje mape

To možnost uporabite za hkratni uvoz vseh slik iz mape. V enem pogovornem oknu lahko izberete **več map**.

1. Kliknite gumb **»Dodaj mapo«** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> na vrhu okna brskalnika datotek
2. Poiščite in izberite mapo(-e), ki vsebuje(-jo) slike iz vaše seje zajemanja
3. Kliknite **»Izberi mapo«**, da uvozite vse podprte slike

{% hint style="info" %}
**Poročanje o datotekah, ki se ne uspejo naložiti.** Če mapa vsebuje datoteke, ki jih program Chloros prepozna, vendar jih ne more naložiti, vas o tem obvesti opozorilo – slike ne izginejo iz mreže neopazno.
{% endhint %}

***

## Uvoz map zajemov LATTICE

Zajemi LATTICE se shranijo z **eno podmapo na raven izvoza** – na primer `raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` — z ustrezno datoteko `.daq` v korenski mapi:

```
output/
├── raw/           capture_<timestamp>_SN<serial>_raw.tif
├── debayered/     capture_<timestamp>_SN<serial>_debayered.tif
├── preview/       capture_<timestamp>_SN<serial>_display.tif
└── *.daq          the downwelling reading matched to the capture
```

**Nastavite »Dodaj mapo« na korensko mapo posnetkov** (`output/` zgoraj). Če izbrana mapa sama ne vsebuje slik, ampak ima podmap, se Chloros samodejno spusti v njih — podmap na tej ravni in korenska mapa `.daq` se zajamejo v enem zamahu.**Kako poteka uvoz posnetkov:*** Vsak posnetek se uvozi kot **posamezna slika**, združena po posnetku (ne en vnos na raven). Druge ravni istega posnetka se prikažejo kot načini pregleda te ene slike.
* **Obdelava se vedno začne z neobdelanim okvirjem.** Druge ravni so vidne, vendar se skozi obdelovalno verigo vedno posreduje le `raw` – ponovna obdelava že obdelanega izdelka bi pomenila dvojno uporabo popravkov, zato to Chloros zavrne. Ponovno uvožen izvoz nikoli ne more zasedati prostora za surovo sliko posnetka.
* Mapa s posnetki, shranjena **brez** surovih uvozov, se prikaže normalno, vendar jo obdelava preskoči in to navede v dnevniku. (Zastavica CLI `--input-level` lahko v tem primeru prisili vstopno točko — glej [referenco CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like).)
* Od različice 1.3.0 naprej lahko mapa zajetih posnetkov vsebuje tudi mapo `clamp_masks/` (maske na piksel za ravni sevanja in odbojnosti — glej [formate izhodnih slik](../output-image-formats.md#clamp-masks)). Maske niso slike prizora, zato jih program Chloros pri uvozu preskoči, ne glede na to, kje se nahajajo.

**Sejne mape LATTICE hub** se uvažajo na enak način: kliknite »Dodaj mapo« in izberite mapo seje, ki ste jo kopirali iz huba (vsebuje `raw/` in `previews/`), skupaj z morebitnim dnevnikom navzdolnega sevanja DAQ-M `.csv`. Če kalibracija kamere ali DAQ-ja še ni shranjena v predpomnilniku na vašem računalniku, jo Chloros ob uvozu samodejno pridobi na podlagi serijske številke (za to je enkrat potrebna internetna povezava).

***

## Razumevanje tabele brskalnika datotek

Ko so slike uvožene, se prikažejo v tabeli z naslednjimi stolpci:

### Ime datoteke

* Izvirno ime datoteke iz kamere
* Ohranja konvencijo poimenovanja kamere (npr. IMG\_0001.RAW ali capture\_20260816\_101500\_SN213800234\_raw.tif)

### Časovni žig

* Datum in čas zajema slike
* Izvlečen iz metapodatkov EXIF slike
* Uporablja se za ujemanje svetlobnih senzorjev, PPK-sinhronizacijo in načrtovanje kalibracijskih ciljev

### Model kamere

* Samodejno zaznana konfiguracija kamere in filtra
* Primeri Survey3: Survey3W_RGN, Survey3N_OCN, Survey3W_RGB
* Primeri LATTICE: LATT-M3M-L41-F550, LATT-M3C-L87-FRGN
* Uporablja se za uporabo pravilnih profilov obdelave

### Stolpec ciljev (potrditveno polje)

* To polje označite za slike, ki vsebujejo kalibracijske cilje
* Če je označena vsaj ena slika, se **skanirajo le označene slike** za iskanje ciljev
* Za podrobnosti glejte [Izbira ciljnih slik](choosing-target-images.md)

### Ogled metapodatkov slike

S klikom na gumb za preklop v zgornjem desnem kotu nad tabelo se v območju mreže slik prikažejo metapodatki izbrane slike.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Datoteke svetlobnih senzorjev v vašem projektu

* Datoteki `.daq` in `.csv` se prikažeta na seznamu brskalnika datotek, vendar nista sliki, na kateri lahko kliknete — zagotavljata navzdolno sevanje za kalibracijo odbojnosti.
* Vsaka uvožena datoteka `.daq`/`.csv` je navedena v **Nastavitve projekta → Svetlobni senzor DAQ**, kjer lahko pregledate korekcijo difuzorskega pokrova, ki velja za vsako datoteko. Glejte [Prilagajanje nastavitev projekta](adjusting-project-settings.md).
* Zapisi, ki jih naredite v zavihku **Svetlobni senzorji**, se samodejno dodajo v odprt projekt – ročni uvoz ni potreben.

***

## Upravljanje datotek v projektu

### Odstranjevanje datotek

Če želite iz projekta odstraniti neželene slike:

1. V tabeli brskalnika datotek izberite eno ali več slik
2. Kliknite gumb **»Odstrani izbrano«** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
3. Potrdite odstranitev (datoteke se ne izbrišejo z diska, temveč se le odstranijo iz projekta)

### Razvrščanje in filtriranje

* **Razvrščanje po stolpcu**: Kliknite kateri koli naslov stolpca, da razvrstite slike
* **Razvrščanje po časovnem žigu**: Koristno za urejanje kronoloških zaporedij posnetkov
* **Filter po modelu kamere**: Razvrstite slike po tipu kamere, če uporabljate več kamer***

## Predogled slike

### Ogled celotne slike

Kliknite katero koli miniaturno sliko v brskalniku datotek, da se prikaže v glavnem predoglednem področju:

1. Slika se prikaže v osrednjem predoglednem oknu
2. Uporabite gumbe za povečavo, da si ogledate podrobnosti slike
3. Med slikami se premikajte s puščičnimi tipkami

### Hitra navigacija

* **Prejšnja slika**: Kliknite puščico v levo ali pritisnite tipko ←
* **Naslednja slika**: Kliknite puščico v desno ali pritisnite tipko →
* **Povečanje/pomanjšanje**: Uporabite kolesce miške ali gumbe za povečavo
* **Premikanje po sliki**: Pri povečani sliki kliknite in povlecite po sliki***

## Obravnava podvojenih datotek

Program Chloros samodejno zazna in prezre podvojene datoteke:

* Datoteke z identičnimi imeni se preskočijo
* Prepreči naključno dvojno obdelavo
* Ob zaznavi podvojenih datotek se prikaže opozorilno sporočilo

{% hint style="warning" %}
**Pomembno**: Pred uvozom ne preimenujte ali spreminjajte izvirnih slikovnih datotek. Chloros se za pravilno obdelavo zanaša na izvirna imena datotek in metapodatke.
{% endhint %}

***

## Mešani nizi podatkov iz različnih kamer

Če vaš projekt vsebuje slike iz več kamer MAPIR:

1. Chloros samodejno zazna vsak model kamere — Survey3, LATTICE ali mešanico
2. Vsak tip kamere se obdeluje z ustreznim kalibracijskim profilom
3. Brskalnik datotek prikaže model kamere v stolpcu »Model kamere«
4. Vsaka kamera dobi ob obdelavi svojo drevesno strukturo izhodnih map

**Primeri scenarijev**: konfiguracija z dvema kamerama Survey3W RGN + Survey3N OCN ali niz LATTICE z glavno kamero RGB in več ozkopasovnimi moduli***

## Priporočene prakse

### Uredite pred uvozom

* Slike kalibracijskih ciljev shranite v isti mapi kot posnetke raziskave
* Datoteke svetlobnih senzorjev `.daq` / `.csv` za vsako sejo zajemanja shranite skupaj s slikovnim gradivom te seje
* Ohranite izvirno strukturo map iz kamere/SD-kartice/vozlišča
* V enem projektu ne mešajte naborov podatkov iz različnih sej

### Poimenovanje datotek

* Ohranite izvirna imena datotek kamere (IMG\_0001.RAW, capture\_..., itd.)
* Datotek pred uvozom ne preimenujte
* Izvirna imena vsebujejo pomembne metapodatke

### Slike kalibracijskih ciljev

* V vsako sejo vedno vključite 1–2 slike kalibracijskih ciljev (program Survey3; za LATTICE jih lahko nadomesti DAQ-posnetek — glejte [Izbira slik ciljev](choosing-target-images.md))
* Posnemite ciljne slike pred in po seji zajemanja
* Ciljne slike postavite v enake svetlobne pogoje kot območje zajemanja
* Označite ciljne slike z uporabo potrditvenega polja »Target«

***

## Pogoste težave in rešitve

### Slike se po uvozu ne prikažejo

**Možni vzroki:**

* Datotečni format ni podprt (glejte seznam podprtih tipov na vrhu te strani)
* Slike so posnete s kamerami, ki niso MAPIR (glejte [Podprte kamere](../supported-cameras.md))
* Poškodovana datoteka ali nepopoln prenos s kartice SD

**Rešitev**: Preverite združljivost formata datoteke in modela kamere ter v opozorilu o nalaganju datotek preverite, katere datoteke niso bile uspešno naložene

### Model kamere ni bil zaznan

**Možni vzroki:**

* Spremenjeni metapodatki EXIF
* Slike so bile urejene v zunanji programski opremi
* Nepopoln prenos datotek

**Rešitev**: Ponovno uvozite izvirne, nespremenjene datoteke iz kamere/kartice SD

### Manjkajoči časovni žigi

**Možni vzroki:**

* Ura kamere ni pravilno nastavljena
* Podatki EXIF so bili odstranjeni z zunanjo programsko opremo

**Rešitev**: Preverite, ali so bile nastavitve časa fotoaparata med zajemanjem pravilne

### Ponovno odprt projekt poroča o manjkajočih datotekah

Če so bile izvorne datoteke od zadnjega odprtja projekta premaknjene ali izbrisane, vam Chloros sporoči, **katere**datoteke manjkajo, namesto da bi se odprla prazna mreža. Če se je premaknila sama mapa projekta, Chloros preusmeri poti do izvornih datotek na mapo, v kateri se zdaj nahaja mapa `project.json`, in ohrani varnostno kopijo (`project.json.before-relink-<date>-<time>`). Če jih tudi tam ne najde, okno**Izvorne datoteke niso najdene**ponuja možnost**Brskanje** do nove lokacije, nato pa Nov projekt ali Izbriši projekt. V CLI uporabite `chloros-cli project open PATH --relink FOLDER`; v SDK pa `open_project(path, relink=...)`.

***

## Naslednji koraki

Ko so datoteke uvožene:

1. **Preglejte seznam datotek** – Preverite, ali so bile vse slike pravilno naložene
2. **Preverite modele kamer** – Preverite, ali so kamere pravilno prepoznane
3. **Označite ciljne slike** – glejte [Izbira ciljnih slik](choosing-target-images.md)
4. **Prilagodite nastavitve** – konfigurirajte možnosti obdelave v [Nastavitvah projekta](adjusting-project-settings.md)
5. **Začnite obdelavo** – glejte [Začetek obdelave](starting-the-processing.md)

Podrobne informacije o konfiguraciji projekta najdete v [Prilagajanje nastavitev projekta](adjusting-project-settings.md).

# Dodajanje datotek v projekt

Ko ustvarite ali odprete projekt v programu Chloros, je naslednji korak dodajanje multispektralnih slik, da lahko začnete z obdelavo. Zavihek »File Browser« (<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">) omogoča enostaven uvoz slik in upravljanje z naborom podatkov.

## Dostop do brskalnika datotek

1. Odprite ali ustvarite projekt v programu Chloros
2. V levem stranskem meniju kliknite ikono **Brskalnik datotek** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
3. V oknu Brskalnika datotek se bo prikazal seznam datotek vašega projekta

{% hint style="info" %}
**Podprte vrste datotek**:

* **Survey3W / Survey3N**: pari RAW+JPG in slike JPG (priporočljivo RAW+JPG)
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
3. Izberite eno ali več slikovnih datotek (za izbiro več datotek pridržite tipko **Ctrl**)
4. Kliknite **»Odpri«**, da uvozite izbrane datoteke

### Metoda 2: Dodaj mapo

To možnost uporabite za uvoz vseh slik iz mape naenkrat. V enem pogovornem oknu lahko izberete **več map**.

1. Kliknite gumb **»Dodaj mapo«** (<img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line">) na vrhu okna brskalnika datotek
2. Poiščite in izberite mapo(-e), ki vsebuje(-jo) slike iz vaše seje zajemanja
3. Kliknite **»Izberi mapo«**, da uvozite vse podprte slike

{% hint style="info" %}
**Prijavljene so datoteke, ki se ne uspejo naložiti.** Če mapa vsebuje datoteke, ki jih Chloros prepozna, vendar jih ne more naložiti, vas o tem obvesti opozorilo – slike iz mreže ne izginejo neopazno.
{% endhint %}

***

## Uvoz map z zajetimi podatki LATTICE

Zajeti podatki LATTICE se shranijo z **eno podmapo na raven izvoza** – na primer `raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` — z ustrezno datoteko `.daq` v korenski mapi:

```
output/
├── raw/           capture_<timestamp>_SN<serial>_raw.tif
├── debayered/     capture_<timestamp>_SN<serial>_debayered.tif
├── preview/       capture_<timestamp>_SN<serial>_display.tif
└── *.daq          the downwelling reading matched to the capture
```

**Nastavite »Dodaj mapo« na korensko mapo posnetkov** (`output/` zgoraj). Če izbrana mapa sama ne vsebuje slik, ampak ima podmap, se Chloros samodejno spusti v njih — podmap na tej ravni in korenska mapa `.daq` se zajamejo v enem koraku.**Kako poteka uvoz posnetkov:*** Vsak posnetek se uvozi kot **ena slika**, razvrščena po posnetku (ne en vnos na raven). Druge ravni istega posnetka se prikažejo kot načini pregleda te ene slike.
* **Obdelava se vedno začne z neobdelanim kadrom.** Druge ravni so vidne, vendar se skozi obdelovalno verigo posreduje le `raw` — ponovna obdelava že obdelanega izdelka bi pomenila dvojno uporabo popravkov, zato se Chloros zavrne. Ponovno uvožen izvoz nikoli ne more zasedati prostora za surovo sliko posnetka.
* Mapa posnetkov, shranjena **brez** uvoza surovih slik, se normalno prikaže, vendar jo obdelava preskoči in to navede v dnevniku. (Zastavica CLI `--input-level` lahko v tem primeru prisili vstopno točko – glej [Referenco CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like).)**Sejne mape LATTICE hub** se uvažajo na enak način: v polju »Dodaj mapo« izberite mapo seje, ki ste jo kopirali iz vozlišča (vsebuje `raw/` in `previews/`), skupaj z morebitnim dnevnikom DAQ-M `.csv`. Če kalibracija kamere ali DAQ-ja še ni shranjena v predpomnilniku na vašem računalniku, jo Chloros ob uvozu samodejno pridobi po serijski številki (za to je enkrat potrebna internetna povezava).***

## Razumevanje tabele brskalnika datotek

Ko so slike uvožene, se prikažejo v tabeli z naslednjimi stolpci:

### Ime datoteke

* Izvirno ime datoteke iz kamere
* Ohranja konvencijo poimenovanja kamere (npr. IMG\_0001.RAW ali capture\_20260816\_101500\_SN213800234\_raw.tif)

### Časovni žig

* Datum in čas zajema slike
* Izvlečen iz metapodatkov EXIF slike
* Uporablja se za ujemanje svetlobnih senzorjev, sinhronizacijo PPK in načrtovanje kalibracijskih ciljev

### Model kamere

* Samodejno zaznana konfiguracija kamere in filtra
* Primeri Survey3: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
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

* Datoteki `.daq` in `.csv` se prikažeta na seznamu brskalnika datotek, vendar nista sliki, na kateri lahko kliknete — zagotavljata navzdol usmerjeno sevanje za kalibracijo odbojnosti.
* Vsaka uvožena datoteka `.daq`/`.csv` je navedena v **Nastavitve projekta → Svetlobni senzor DAQ**, kjer lahko pregledate korekcijo difuzorskega pokrova, ki velja za vsako datoteko. Glejte [Prilagajanje nastavitev projekta](adjusting-project-settings.md).
* Zapisi, ki jih naredite v zavihku **Svetlobni senzorji**, se samodejno dodajo v odprt projekt – ročni uvoz ni potreben.***

## Upravljanje datotek v projektu

### Odstranjevanje datotek

Če želite iz projekta odstraniti neželene slike:

1. V tabeli brskalnika datotek izberite eno ali več slik
2. Kliknite gumb **»Odstrani izbrano«** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
3. Potrdite odstranitev (datoteke se ne izbrišejo z diska, ampak se le odstranijo iz projekta)

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
* **Povečajte/zmanjšajte**: Uporabite kolesce miške ali gumbe za povečavo
* **Premikanje po sliki**: Pri povečani sliki kliknite in povlecite po sliki***

## Obravnava podvojenih datotek

Chloros samodejno zazna in prezre podvojene datoteke:

* Datoteke z identičnimi imeni se preskočijo
* Prepreči naključno dvojno obdelavo
* Ob zaznanih podvojenih datotekah se prikaže opozorilno sporočilo

{% hint style="warning" %}
**Pomembno**: Pred uvozom ne preimenujte ali spreminjajte izvirnih slikovnih datotek. Chloros se za pravilno obdelavo zanaša na izvirna imena datotek in metapodatke.
{% endhint %}

***

## Mešani nizi podatkov iz različnih kamer

Če vaš projekt vsebuje slike iz več kamer MAPIR:

1. Chloros samodejno zazna vsak model kamere — Survey3, LATTICE ali mešanico
2. Vsak tip kamere se obdeluje z ustreznim kalibracijskim profilom
3. Brskalnik datotek prikaže model kamere v stolpcu »Model kamere«
4. Vsaka kamera po obdelavi dobi svojo drevesno strukturo izhodnih map

**Primeri scenarijev**: Survey3W RGN + Survey3N OCN konfiguracija z dvema kamerama, ali matrika LATTICE z glavno kamero RGB in več ozkopasovnimi moduli***

## Priporočene prakse

### Uredite podatke pred uvozom

* Slike kalibracijskih tarč shranite v isti mapi kot posnetke meritev
* Datoteke svetlobnih senzorjev `.daq` / `.csv` za vsako sejo zajemanja shranite skupaj s slikami te seje
* Ohranite prvotno strukturo map iz kamere/SD-kartice/razdelilnika
* V enem projektu ne mešajte naborov podatkov iz različnih sej

### Poimenovanje datotek

* Ohranite izvirna imena datotek kamere (IMG\_0001.RAW, capture\_..., itd.)
* Datotek pred uvozom ne preimenujte
* Izvirna imena vsebujejo pomembne metapodatke

### Slike kalibracijskih ciljev

* V vsako sejo vedno vključite 1–2 sliki kalibracijskih ciljev (Survey3; za LATTICE jih lahko nadomesti DAQ-posnetek – glej [Izbira ciljnih slik](choosing-target-images.md))
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

Če so bile izvorne datoteke od zadnjega odprtja projekta premaknjene ali izbrisane, vas Chloros obvesti, **katere** datoteke manjkajo, namesto da bi se odprla prazna mreža. Datoteke obnovite na njihovih prvotnih poteh ali odstranite manjkajoče vnose in jih ponovno uvozite.***

## Naslednji koraki

Ko so datoteke uvožene:

1. **Preglejte seznam datotek** – Preverite, ali so se vse slike pravilno naložile
2. **Preverite modele fotoaparatov** – Preverite, ali je prepoznavanje fotoaparatov pravilno
3. **Označite ciljne slike** – glejte [Izbira ciljnih slik](choosing-target-images.md)
4. **Prilagodite nastavitve** – konfigurirajte možnosti obdelave v [Nastavitvah projekta](adjusting-project-settings.md)
5. **Začnite obdelavo** – glejte [Začetek obdelave](starting-the-processing.md)

Podrobne informacije o konfiguraciji projekta najdete v [Prilagajanje nastavitev projekta](adjusting-project-settings.md).

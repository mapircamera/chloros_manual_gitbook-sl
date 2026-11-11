# GUI: Projekti Program

Chloros vam omogoča ustvarjanje projektov, ki jih lahko v prihodnosti ponovno odprete. Projekt je navadna mapa (znotraj vaše projektne mape), ki vsebuje:

* `project.json` — nastavitve projekta, seznam datotek in nastavitve prikaza
* `cameras.json` — kamere in matrike, priključene medtem, ko je bil projekt odprt, skupaj z njihovimi nastavitvami
* `sensors.json` — senzorji svetlobe DAQ, priključeni med odprtjem projekta, ter povezave med kamerami in senzorji
* vaše zajete slike, posnetki `.daq` in mape z obdelanimi izhodnimi podatki

Ni lastniškega formata projektnih datotek — mapa in njene JSON-datoteke predstavljajo projekt, kar omogoča tudi enostavno kopiranje, arhiviranje in upravljanje projektov prek [CLI](CLI.md) ali [Python SDK](api-python-sdk.md).

Če tega ne spremenite v [Nastavitvah projekta](project-settings/project-settings.md), je mapa projekta `Documents\MAPIR\Chloros Projects` v mapi Dokumenti (`~/Documents/MAPIR/Chloros Projects` v sistemu Linux).

{% hint style="warning" %}
**Windows: mape projektov naj bodo kratke, razen če so dolge poti omogočene.**V sistemu Windows so dolge poti privzeto onemogočene, zato nobena pot do datoteke ne sme biti daljša od 260 znakov. Obdelani izdelki se nahajajo do približno 130 znakov pod projektno mapo, zato Chloros od različice 1.3.0 naprej zavrne pregloboko projektno mapo – ob njenem ustvarjanju, odpiranju ali začetku obdelave – namesto da bi se zagnal in ne zapisal ničesar. Sporočilo navaja najdaljšo projektno mapo, ki ustreza. Uporabite krajšo projektno mapo ali vklopite dolge poti v sistemu Windows (možnost namestitvenega programa**Omogoči dolge poti v sistemu Windows**, ki je privzeto izbrana) in ponovno zaženite program Chloros. Glejte [Začetek obdelave](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).
{% endhint %}

## Nov projekt

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>V glavnem meniju izberite »Nov projekt« in vnesite edinstveno ime za svoj projekt.

Če ste shranili kakšne predloge projektov, se pod poljem za ime prikaže spustni meni **Izberi predlogo** — izbira ene od njih začne nov projekt z nastavitvami te predloge. Predloge shranite v [Nastavitvah projekta](project-settings/project-settings.md): v polje »Ime predloge projekta« vnesite ime in kliknite ikono za shranjevanje.

## Odpri projekt

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>V oknu »Odpri projekt« so navedeni vsi projekti v vaši projektni mapi, na dnu pa je možnost <strong>Odpri projektno mapo</strong></p></figcaption></figure>

Izberite »Odpri projekt«, da si ogledate seznam obstoječih projektov v mapi projektov. Če ni nobenega projekta, se stranski meni ne bo odprl. Na zgornji fotografiji so prikazani nekateri projekti, ustvarjeni prek grafičnega vmesnika (t1, t2, t3). Projekti DATE\_TIME so bili ustvarjeni prek CLI-ja z uporabo privzetega sistema poimenovanja projektov. S klikom na katero koli ime projekta ga boste odprli.

S klikom na gumb »Odpri mapo s projekti« se odpre raziskovalec datotek na vašem računalniku na poti do projekta. Pot do projekta lahko prilagodite v [Nastavitvah projekta](project-settings/project-settings.md).

Če so bile katere koli izvorne slikovne datoteke projekta od zadnjega odpiranja premaknjene ali izbrisane, Chloros namesto odpiranja prazne mreže prikaže pogovorno okno, v katerem so natančno navedene manjkajoče datoteke.

## Podvajanje projekta

Na voljo, ko je projekt odprt. Izberite »Podvajanje projekta«, da kopirate trenutni projekt pod novim imenom – Chloros predlaga naslednje prosto ime (npr. »MojProjekt (2)«) – in podvojeni projekt se takoj odpre.

## Dodaj datoteke

Ko je projekt odprt, izberite »Dodaj datoteke« iz glavnega menija, da v trenutni projekt dodate posamezne slikovne datoteke. To odraža funkcijo dodajanja v brskalniku datotek, vendar je zaradi večje priročnosti dostopna neposredno iz glavnega menija.

## Dodaj mapo

Ko je projekt odprt, v glavnem meniju izberite »Dodaj mapo«, da v trenutni projekt dodate mape s slikami. V enem koraku lahko izberete več map. Podvojene datoteke se prezrejo.

## Začni / Ustavi obdelavo

Ko so datoteke dodane v projekt, se v glavnem meniju prikaže možnost »Začni obdelavo«. To je enako kot klik na gumb Predvajaj/Začni v zgornjem naslovnem pasu. Med obdelavo se ta menijska postavka spremeni v »Ustavi obdelavo«, da lahko ustavite potek obdelave.

## Povezava s kamero / Povezava s svetlobnim senzorjem

Na dnu glavnega menija sta dve bližnjici za strojno opremo, ki sta na voljo ne glede na to, ali je projekt odprt ali ne:

* **Povezava s kamero** — odpre [zavihek »Kamere«](lattice/), kjer lahko priključite kamero ali niz kamer LATTICE.
* **Povezava s svetlobnim senzorjem** — odpre [zavihek Svetlobni senzorji](daq/), kjer lahko priključite DAQ svetlobni senzor.

Če strojne naprave priključite, medtem ko je projekt odprt, se te povezave shranijo v projekt (glejte spodaj). Če projekt ni odprt, so povezave veljavne le za tekočo sejo.

{% hint style="info" %}
Menijske možnosti »Dodaj datoteke«, »Dodaj mapo« in »Začni/ustavi obdelavo« so vidne ali omogočene le, če je projekt odprt in so bile dodane datoteke. Omogočajo hiter dostop do dejanj, ki so na voljo tudi prek stranskega pasu brskalnika datotek in gumbov v glavi.
{% endhint %}

## Projekti si zapomnijo vašo strojno opremo

Novost v različici 1.2.0: projekt ohrani strojno opremo, ki jo priključite, dokler je odprt. Kamere in matrike (z nastavitvami za posamezne kamere, imeni, barvami in razporeditvijo mreže) se posnamejo v `cameras.json`, svetlobni senzorji (z imeni, barvami in povezavami s kamerami) pa v `sensors.json` — samodejno, medtem ko delate.

Ko **ponovno odprete** projekt, Chloros takoj ne vzpostavi stika z nobeno strojno opremo. Vsaka polovica se ponovno poveže, ko prvič obiščete zavihek, ki ji pripada:

* Odpiranje zavihka **Kamere** ponovno poveže shranjene kamere in matrike ter ponovno uporabi njihove shranjene nastavitve.
* Odpiranje zavihka **Svetlobni senzorji**ponovno poveže shranjene DAQ-senzorje in ponovno uporabi njihove shranjene nastavitve, vključno s pokrovčkom. Senzor, ki je v meniju**Pokrovček**nastavljen na**Nič (gol senzor)**, ostane gol. Projekt, shranjen s prejšnjo različico programa Chloros, je goli senzor zabeležil kot edino možnost, zato se ponovno prikaže s korekcijo sončnega kozinusa; nastavite ga še enkrat na**Nič** in shranite projekt.

Tako odpiranje projekta zgolj za pregledovanje ali izvoz slik nikoli ne vklopi kamer v način pretakanja. Če shranjene naprave ni mogoče najti ob odprtju zavihka, vas pogovorno okno obvesti, katere naprave niso na voljo, da jih lahko ponovno povežete ali odstranite.

## DAQ-posnetki in datoteke .daq v projektu

* Posnetki `.daq`, narejeni medtem ko je projekt odprt (iz zavihka »Light Sensors« ali med zajemanjem), se **samodejno dodajo v projekt**.
* Uvožene datoteke `.daq` in vsi posnetki projekta so navedeni v razdelku **DAQ svetlobni senzor** v [Nastavitvah projekta](project-settings/project-settings.md), vsaka s svojim profilom korekcije kapice.
* Med obdelavo datoteke `.daq` projekta zagotavljajo osvetlitev od zgoraj za produkte odbojnosti — glej [Formati izhodnih slik](output-image-formats.md).

## Izvajanje shranjenega projekta brez grafičnega vmesnika

Shranjeni projekt je mogoče izvajati brez grafičnega vmesnika:

* **CLI**: `chloros-cli project open / connect / capture / sensor / align / run` deluje na poti do projektne mape — glejte [Referenco CLI](reference/cli-reference.md).
* **SDK**: `chloros_sdk.open_project(path)` vrne ročaj projekta; `connect_all()` vključi vse shranjene kamere, matrike in svetlobne senzorje z njihovimi shranjenimi nastavitvami (svetlobni senzorji prek zagnanega Chlorosovega backenda) — glej [SDK referenco](reference/sdk-reference.md).

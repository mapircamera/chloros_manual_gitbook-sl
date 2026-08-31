# GUI: Projekti

Chloros vam omogoča ustvarjanje projektov, ki jih lahko v prihodnosti ponovno odprete. Projekt je navadna mapa (znotraj vaše projektne mape), ki vsebuje:

* `project.json` — nastavitve projekta, seznam datotek in nastavitve prikaza
* `cameras.json` — kamere in matrike, priključene medtem, ko je bil projekt odprt, skupaj z njihovimi nastavitvami
* `sensors.json` — senzorji svetlobe DAQ, priključeni med odprtjem projekta, ter povezave med kamerami in senzorji
* vaše zajete slike, posnetki `.daq` in mape z obdelanimi izhodnimi podatki

Ni lastniškega formata projektnih datotek – mapa in njene datoteke JSON predstavljajo projekt, kar omogoča tudi enostavno kopiranje, arhiviranje in zagon projektov iz [CLI](CLI.md) ali [Python SDK](api-python-sdk.md).

## Nov projekt

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>V glavnem meniju izberite »Nov projekt« in vnesite edinstveno ime za svoj projekt.

Če ste shranili kakšne predloge projektov, se pod poljem za ime prikaže spustni meni **Izberi predlogo** — izbira ene od njih začne nov projekt z nastavitvami te predloge. Predloge shranite v [Nastavitvah projekta](project-settings/project-settings.md): v polje »Ime predloge projekta« vnesite ime in kliknite ikono za shranjevanje.

## Odpri projekt

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>V oknu »Odpri projekt« so navedeni vsi projekti v vaši projektni mapi, na dnu pa je gumb <strong>»Odpri projektno mapo«</strong></p></figcaption></figure>Izberite »Odpri projekt«, da si ogledate seznam obstoječih projektov v mapi projektov. Če ni nobenega projekta, se stranski meni ne bo odprl. Na zgornji fotografiji so prikazani nekateri projekti, ustvarjeni prek grafičnega vmesnika (t1, t2, t3). Projekti z imeni DATE\_TIME so bili ustvarjeni s programom CLI po privzetem sistemu poimenovanja projektov. S klikom na katerokoli ime projekta se ta odpre.

S klikom na gumb »Odpri mapo s projekti« se v vašem računalniku odpre raziskovalec datotek na poti do projekta. Pot do projekta lahko prilagodite v [Nastavitvah projekta](project-settings/project-settings.md).

Če so bile katere koli izvorne slikovne datoteke projekta od zadnjega odprtja premaknjene ali izbrisane, Chloros prikaže pogovorno okno, v katerem so natančno navedene manjkajoče datoteke, namesto da bi se odprla prazna mreža.

## Kopiranje projekta

Na voljo, ko je projekt odprt. Izberite »Duplicate Project«, da kopirate trenutni projekt pod novim imenom — Chloros predlaga naslednje prosto ime (npr. »MyProject (2)«) — in kopija se takoj odpre.

## Dodaj datoteke

Ko je projekt odprt, izberite »Dodaj datoteke« iz glavnega menija, da v trenutni projekt dodate posamezne slikovne datoteke. Ta funkcija je enaka funkciji dodajanja v brskalniku datotek, vendar je zaradi večje priročnosti dostopna neposredno iz glavnega menija.

## Dodaj mapo

Ko je projekt odprt, v glavnem meniju izberite »Dodaj mapo«, da v trenutni projekt dodate mape s slikami. V enem koraku lahko izberete več map. Podvojene datoteke se prezrejo.

## Začni / Ustavi obdelavo

Ko so datoteke dodane v projekt, se v glavnem meniju prikaže možnost »Začni obdelavo«. To je enako kot klik na gumb »Predvajaj/Začni« v zgornjem naslovnem pasu. Med obdelavo se ta menijska postavka spremeni v »Ustavi obdelavo«, kar vam omogoča, da ustavite potek obdelave.

## Povezava s kamero / Povezava s svetlobnim senzorjem

V spodnjem delu glavnega menija sta dve bližnjici za strojno opremo, ki sta na voljo ne glede na to, ali je projekt odprt ali ne:

* **Povezava s kamero** — odpre [zavihek »Kamere«](lattice/), kjer lahko priključite kamero ali niz kamer LATTICE.
* **Povezava s svetlobnim senzorjem** — odpre [zavihek Svetlobni senzorji](daq/), kjer lahko priključite DAQ svetlobni senzor.

Če strojne naprave priključite, medtem ko je projekt odprt, se te povezave shranijo v projekt (glejte spodaj). Brez odprtega projekta so povezave veljavne le za tekočo sejo.

{% hint style="info" %}
Menijske možnosti »Dodaj datoteke«, »Dodaj mapo« in »Začni/ustavi obdelavo« so vidne ali omogočene le, če je projekt odprt in so bile dodane datoteke. Omogočajo hiter dostop do dejanj, ki so na voljo tudi prek stranskega pasu brskalnika datotek in gumbov v glavi.
{% endhint %}

## Projekti si zapomnijo vašo strojno opremo

Novost v različici 1.2.0: projekt ohrani strojno opremo, ki jo priključite, dokler je odprt. Kamere in matrike (z nastavitvami za posamezno kamero, imeni, barvami in razporeditvijo mreže) se shranijo v `cameras.json`, svetlobni senzorji (z imeni, barvami in povezavami s kamerami) pa v `sensors.json` — samodejno, medtem ko delate.

Ko **ponovno odprete** projekt, Chloros takoj ne vzpostavi povezave z nobeno strojno opremo. Vsaka polovica se ponovno poveže, ko prvič obiščete zavihek, ki ji pripada:

* Odpiranje zavihka **Kamere** ponovno poveže shranjene kamere in matrike ter ponovno uporabi njihove shranjene nastavitve.
* Odpiranje zavihka **Svetlobni senzorji** ponovno poveže shranjene DAQ-senzorje.

Tako odpiranje projekta zgolj za pregledovanje ali izvoz slik nikoli ne vklopi kamer v način pretakanja. Če ob odpiranju zavihka ni mogoče najti shranjene naprave, vas pogovorno okno obvesti, katere naprave niso na voljo, da jih lahko ponovno povežete ali odstranite.

## DAQ-posnetki in datoteke .daq v projektu

* Posnetki `.daq`, narejeni medtem ko je projekt odprt (iz zavihka Svetlobni senzorji ali med zajemanjem), se **samodejno dodajo v projekt**.
* Uvožene datoteke `.daq` in vsi posnetki projekta so navedeni v razdelku **DAQ svetlobni senzor** v [Nastavitvah projekta](project-settings/project-settings.md), vsaka s svojim profilom korekcije osvetlitve.
* Med obdelavo datoteke `.daq` projekta zagotavljajo osvetlitev od zgoraj za produkte odbojnosti — glejte [Formati izhodnih slik](output-image-formats.md).

## Izvajanje shranjenega projekta brez grafičnega vmesnika

Shranjeni projekt je mogoče izvajati brez grafičnega uporabniškega vmesnika:

* **CLI**: `chloros-cli project open / connect / capture / sensor / align / run` deluje na poti do projektne mape – glej [CLI Referenca](reference/cli-reference.md).
* **SDK**: `chloros_sdk.open_project(path)` vrne identifikator projekta; `connect_all()` vključi vse shranjene kamere in senzorje v omrežje z njihovimi shranjenimi nastavitvami — glej [SDK Referenca](reference/sdk-reference.md).

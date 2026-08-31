# Uporaba Chloros z AI-pomočniki

Ta priročnik je namenjen dvema ciljnima skupinam: ljudem in AI-pomočnikom, s katerimi ljudje vedno pogosteje sodelujejo. Na vsaki strani so navedene natančne vrednosti, privzete nastavitve in ukazi, ki jih je mogoče kopirati in prilepiti, tako da lahko pomočnik (Claude, ChatGPT, Copilot, agent za programiranje, …) že ob prvem poskusu napiše delujočo avtomatizacijo Chloros.

Različica Chloros: **

1.2.0**. Platforme CLI/SDK: Windows 10/11 x64 in Linux (x86_64 / Jetson aarch64).

## Kaj predati pomočniku

| Vir | URL | Za kaj je namenjen |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Računalniško berljiv indeks vseh strani v tem priročniku. |
| **CLI Reference** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Celotna površina ukazov `chloros-cli`: vsak ukaz, zastavica, privzeta vrednost, izhodna koda in pravilo za izhodno mapo. Napisano za uporabo v sistemih LLM. |
| **SDK Referenca** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Celoten `chloros_sdk` Python API: razredi, podpisi, izjeme in praktični primeri. Napisano za uporabo v LLM. |
| **Katera koli stran kot surovi Markdown** | dodajte `.md` na stran URL | npr. `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` vrne stran v obliki surovega Markdowna — idealno za vstavljanje v okno konteksta ali pridobivanje iz agenta. |

Povezave znotraj priročnika: [CLI Referenca](reference/cli-reference.md) · [SDK Referenca](reference/sdk-reference.md).

{% hint style="info" %}
Obe referenčni strani sta samostojni: pomočnik, ki je prebral eno od njiju, ne potrebuje preostalega priročnika, da bi napisal pravilni skript.
{% endhint %}

## Predloge ukazov

Kopirajte, izpolnite `<placeholders>` in prilepite v svoj pomočnik.

### 1. Obdelava mape z leti v NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Skupinsko spremljanje imenika zajetih podatkov

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Priključite matriko LATTICE in zajemajte

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Snemanje spektrov svetlobnega senzorja DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
Pisanje skriptov za DAQ iz ukazne vrstice vedno poteka prek družine `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Drugi podukazi `daq`, ki jih vaš pomočnik morda izmisli, niso na voljo v dobavljenih različicah in se zaključijo z napako.
{% endhint %}

## Zakaj skripti, napisani z umetno inteligenco, dobro delujejo z Chloros

Vsako od teh je resnično, preverjeno obnašanje Chloros 1.2.0 — odpravljajo klasične napake avtomatizacije, ki jo pišejo stroji:

* **Brez zapletenega nastavljanja.**Pomočniki za pametno povezovanje programa SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) ter vstopne točke za obdelavo (`ChlorosLocal`, `process_folder`)**samodejno zaženejo lokalni backend**. Za zgenerirani skript ni potrebno, da je odprt grafični vmesnik ali ročno zagnan strežnik — potrebna je le namestitev paketa desktop/CLI.
* **Celoten potek obdelave poteka z enim samim klicem.** `chloros_sdk.process_folder("path", indices=["NDVI"])` izvede uvoz → kalibracijo → odbojnost → izvoz indeksa od začetka do konca. Manj površine, manj možnosti, da bi pri generiranem skriptu prišlo do napak.
* **Izvajanja brez izhodnih podatkov se samodiagnosticirajo.** Po `process()` se povzetek izvajanja priloži rezultatu, vsak namig o obdelavi (npr. *zakaj* izvedba ni prinesla izhodnih podatkov) ponovno posredovan kot Python `UserWarning` — tako da diagnozo prikaže tudi skript, ki nikoli ne pregleda slovarja rezultatov.
* **CLI se glasno ne uspe.**Izvedba `chloros-cli process`, ki je zahtevala rezultate, a ni zapisala nobenega, izpiše `Processing finished but wrote no image products.` in**se zaključi z vrednostjo, ki ni nič**, zato jo skripti v lupini in CI zaznavajo s preprostim preverjanjem izhodne kode. Uspešni izvajanji poročajo o `Image products written: N`.

Ena asimetrija, ki jo mora poznati pomočnik: `process()` v SDK namerno **ne** sproži izjeme pri izvedbi brez izdelkov — namesto tega poroča prek povzetka/namigov. Če se mora potek Python ustaviti ob izvedbi brez izhodnih podatkov, preverite povzetek (recept 2 to stori).

## Opozorila

* **Chloros+ zahteva prijavo.**CLI in SDK zahtevata**plačljivo** stopnjo Chloros+, kar se uveljavlja na strežniški strani: zahtevki z napako `401 AUTH_REQUIRED` ne uspejo, če niste prijavljeni, in z napako `403 PLAN_UPGRADE_REQUIRED` na brezplačni stopnji. Pred zagonom generiranih skriptov enkrat na vsakem računalniku zaženite `chloros-cli login`. Glejte [Chloros+ Prijava](chloros+-login.md).
* **Ukaz za zajem upravlja z dejansko strojno opremo.** Ukazi `lattice` / `daq` / `project` in objekti seje SDK vzpostavljajo povezavo, prenašajo podatke in sprožajo fizične kamere in senzorje. Pred prvim zagonom preglejte ustvarjeni skript in ga zaženite ob prisotnosti strojne opreme.
* **Naključno preverite izhodne podatke.** Pred objavo rezultatov preverite mape z izdelki in nekaj vrednosti pikslov. Zlasti TIFF-datoteke z odbojnostjo so prilagojene glede na vir – preberite XMP-oznako `Chloros:PixelScale` (LATTICE: 32768 = odbojnost 1,0; Survey3: 65535), namesto da predpostavljate delitelj. Obe referenčni strani to dokumentirata v razdelku »Branje pikslov odbojnosti«.
* **Majhne pasti, ki ovirajo generirano kodo:**`pool-record` piše v datotečni sistem**gostitelja backenda** (privzeto `~/Documents/DAQ Live View/`); na računalnikih z več omrežnimi vmesniki raje uporabite `daq pool-connect --eth-host <ip-or-hostname>` namesto samodejnega odkrivanja; in uporabite `http://127.0.0.1:5000` (nikoli `localhost`) povsod, kjer se pojavi backend URL.

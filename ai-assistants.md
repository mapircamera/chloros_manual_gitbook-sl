# Uporaba programa Chloros z umetno inteligenco

Ta priročnik je namenjen dvema ciljnima skupinam: ljudem in AI pomočnikom, s katerimi ljudje vedno pogosteje delajo. Vsaka stran vsebuje natančne vrednosti, privzete nastavitve in ukaze, ki jih je mogoče kopirati in prilepiti, tako da lahko pomočnik (Claude, ChatGPT, Copilot, programski agent, …) že ob prvem poskusu napiše delujočo avtomatizacijo v Chlorosu.

Različica Chloros: **

1.3.0**. Platforme CLI/SDK: Windows 10/11 x64 in Linux (x86\_64 / Jetson aarch64).

## Kaj predati pomočniku

| Vir | URL | Za kaj je namenjen |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Strojno berljiv indeks vseh strani v tem priročniku. |
| **Referenca CLI** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Celoten nabor ukazov `chloros-cli`: vsi ukazi, zastavice, privzete vrednosti, izhodne kode in pravila za izhodne mape. Napisano za uporabo v LLM. |
| **Referenca SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Celoten Python API za `chloros_sdk`: razredi, podpisi, izjeme in praktični primeri. Napisano za uporabo v sistemih LLM. |
| **Katera koli stran v surovem formatu Markdown** | dodajte `.md` k URL-ju strani | npr. `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` vrne stran v surovem formatu Markdown — idealno za vstavljanje v okno konteksta ali pridobivanje iz agenta. |

Povezave znotraj priročnika: [Referenca CLI](reference/cli-reference.md) · [Referenca SDK](reference/sdk-reference.md).

{% hint style="info" %}
Obe referenčni strani sta samostojni: pomočnik, ki je prebral eno od njiju, za pisanje pravilnega skripta ne potrebuje preostalega priročnika.
{% endhint %}

## Naj vaš pomočnik neposredno zažene program Chloros: `chloros-mcp`

Vse zgoraj navedeno predvideva, da vaš pomočnik *napiše* skript ali ukaz. Z `chloros-mcp` asistent, ki govori [Model Context Protocol](https://modelcontextprotocol.io) (MCP), kot so Claude Desktop, Claude Code ali Cursor, sam upravlja s programom Chloros. Vi vprašate v preprostem jeziku, on pa pokliče orodja strežnika:

* **Obdelava slikovnega gradiva:** ustvarite projekt, uvozite mapo posnetkov, izberite nastavitve, obdelajte in poročajte, kaj je izvozil izvedeni postopek.
* **Upravljanje strojne opreme:** priključite kamere LATTICE, matrike in svetlobne senzorje DAQ, si oglejte predogled, nastavite osvetlitev in barve, zajemajte, snemajte in posodobite vgrajeno programsko opremo.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

Orodja strežnika pokrivajo funkcionalnost [Python SDK](api-python-sdk.md). Namestijo se skupaj z SDK-jem, zato običajno ni treba namestiti ničesar drugega: v sistemu Windows namestitveni program Chloros shrani `chloros-mcp.exe` v mapo `Scripts` vašega Pythona, Linuxov namestitveni program `.deb` pa datoteko `chloros-mcp` namesti v mapo `/usr/bin`.

**Claude Code:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**to dodajte v `claude_desktop_config.json` (odprete ga v**Nastavitve &gt; Razvijalec &gt; Uredi konfiguracijo**), nato ponovno zaženite Claude Desktop.**Cursor** uporabi isti vnos v `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

Če odjemalec sporoči, da se strežnik ni uspel zagnati, `chloros-mcp` verjetno ni na seznamu `PATH`, ki ga vidi: namesto tega vnesite polno pot (`where chloros-mcp` v sistemu Windows, `which chloros-mcp` v sistemu Linux).

Na istem računalniku morate imeti nameščen program Chloros in se tam prijaviti, bodisi v aplikaciji bodisi prek `chloros-cli login`, na kateri koli plačljivi stopnji Chloros+ (Copper ali višji). Strežnik se ne more prijaviti namesto vas.

{% hint style="info" %}
Nastavitve odjemalca za vsako aplikacijo, možnosti, vsako orodje in delovanje izvajanja: [MCP Server (AI Assistants)](reference/sdk-reference.md#mcp-server-ai-assistants) v referenčnem priročniku SDK.
{% endhint %}

## Predloge za ukaze

Kopirajte, izpolnite `<placeholders>` in prilepite v svojega pomočnika.

### 1. Obdelava mape z letalskimi posnetki v NDVI

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

### 3. Povezava matrike LATTICE in zajem

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Snemanje spektrov svetlobnih senzorjev DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
Pisanje skriptov za DAQ iz ukazne vrstice vedno poteka prek družine `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Drugi podukazi `daq`, ki jih lahko izmisli vaš pomočnik, niso na voljo v dostavljenih različicah in se zaključijo z napako.
{% endhint %}

## Zakaj skripti, napisani z umetno inteligenco, dobro delujejo z Chlorosom

Vsak od teh primerov je resnično, preverjeno delovanje Chlorosa 1.3.0 – odpravljajo klasične napake avtomatizacije, napisane s stroji:

* **Ni zapletenega nastavljanja.**Pomočniki za pametno povezovanje v SDK-ju (`connect_camera`, `connect_array`, `connect_daq_sensor`) ter vstopne točke za obdelavo (`ChlorosLocal`, `process_folder`)**samodejno zaženejo lokalni backend**. Generirani skript ne potrebuje odprtega grafičnega vmesnika ali ročno zagnanega strežnika – potrebuje le nameščen paket za namizje/CLI.
* **Celoten potek obdelave poteka v enem klicu.** `chloros_sdk.process_folder("path", indices=["NDVI"])` od začetka do konca izvede uvoz → kalibracijo → odbojnost → izvoz indeksa. Manj površine, manj možnosti, da bi pri generiranem skriptu prišlo do napak.
* **Izvajanja brez izhodnih podatkov se samodiagnosticirajo.** Po `process()` se povzetek izvajanja priloži rezultatu, vsak namig o obdelavi (npr. *zakaj* izvedba ni prinesla izhodnih podatkov) ponovno izdan kot Pythonov skript `UserWarning` — tako da diagnozo prikaže tudi skript, ki nikoli ne pregleda slovarja rezultatov.
* **CLI se jasno ne uspe.**Izvedba `chloros-cli process`, ki je zahtevala izhodne datoteke, a jih ni zapisala, izpiše `Processing finished but wrote no image products.` in**se zaključi z izhodno kodo, ki ni nič**, tako da jo skripti v lupini in CI zaznajo s preprostim preverjanjem izhodne kode. Uspešni izvajanji poročajo o `Image products written: N`, skupaj z `(plus M clamp masks)`, če so bile zapisane maske. Ukazi za zajem prav tako navedejo vsako datoteko, ki je niso mogli zapisati, in zaključijo z `1`.
* **Relativne poti pomenijo tisto, kar skript pričakuje.** Pri lokalnem backendu se relativna pot izpisa (`-o out/` v CLI, `output_dir="out/"` v SDK) razreši glede na mapo, iz katere se izvaja ukaz ali skript.

Ena asimetrija, ki jo mora poznati pomočnik: `process()` v SDK namerno **ne** sproži izjeme pri izvedbi brez izhodnih podatkov – namesto tega o tem poroča prek povzetka/namigov. Če se mora Pythonov potek ustaviti ob praznem izvajanju, preverite povzetek (recept 2 to stori).

## Opozorila

* **Potrebna je prijava v Chloros+.**CLI in SDK zahtevata**plačljivo** stopnjo Chloros+, kar se uveljavlja na strani strežnika: zahtevki se ne uspešijo z napako `401 AUTH_REQUIRED`, če niste prijavljeni, in z napako `403 PLAN_UPGRADE_REQUIRED` na brezplačni stopnji. Pred izvajanjem ustvarjenih skriptov enkrat na vsakem računalniku zaženite `chloros-cli login`. Glejte [Prijava v Chloros+](chloros+-login.md).
* **Ukazi za zajem upravljajo z dejansko strojno opremo.** Ukazi `lattice` / `daq` / `project`, objekti seje SDK in orodja `chloros-mcp` vzpostavljajo povezavo s fizičnimi kamerami in senzorji, prenašajo podatke in jih sprožajo. Pred prvim zagonom preglejte ustvarjeni skript in ga zaženite ob prisotnosti strojne opreme.
* **Naključno preverite izhodne podatke.** Pred objavo rezultatov preverite mape z izdelki in nekaj vrednosti pikslov. Zlasti so TIFF-datoteke z odbojnostjo prilagojene glede na vir – preberite oznako XMP `Chloros:PixelScale` (datoteke LATTICE jo vsebujejo: 32768 = 1,0 odbojnost; datoteke Survey3 pa ne in uporabljajo 65535 = 1,0), namesto da predpostavljate delitelj. Obe referenčni strani to dokumentirata v razdelku »Branje pikslov odbojnosti«.
* **Majhne pasti, ki ovirajo generirano kodo:**`pool-record` piše v datotečni sistem**gostiteljskega strežnika** (privzeto `~/Documents/DAQ Live View/`); na računalnikih z več omrežnimi vmesniki raje uporabite `daq pool-connect --eth-host <ip-or-hostname>` namesto samodejnega odkrivanja; uporabite `http://127.0.0.1:5000` (nikoli `localhost`) povsod, kjer se pojavi URL strežnika; skript, ki prehaja po izhodnem drevesu projekta, naj preskoči mape `clamp_masks/` (njihove datoteke `*_clampmask.tif` so maske, ne slike — glej [Formati izhodnih slik](output-image-formats.md#clamp-masks)); ter v sistemu Windows, kjer so dolge poti onemogočene, poskrbite, da so projektne mape kratke – projektna mapa, ki je daljša od približno 130 znakov, bo zavrnjena še pred začetkom obdelave.

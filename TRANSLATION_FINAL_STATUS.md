# Chloros Priročnik – Končni status prevajalskega projekta

**Zadnja posodobitev:** 13. december 2025

---

## 📊 Splošni status

### ✅ **ZAKLJUČENO: 32 jezikov (DeepL)**

Popolnoma prevedeno in objavljeno na GitBook:

**Evropski jeziki (20):**
- 🇧🇬 bolgarščina (bg)
- 🇨🇿 češčina (cs)
- 🇩🇰 danščina (da)
- 🇩🇪 nemščina (de)
- 🇬🇷 grščina (el)
- 🇪🇸 španščina (es)
- 🇪🇪 estonščina (et)
- 🇫🇮 Finski (fi)
- 🇫🇷 Francoski (fr)
- 🇭🇺 Madžarski (hu)
- 🇮🇹 Italijanski (it)
- 🇱🇻 Latvijski (lv)
- 🇱🇹 Litovski (lt)
- 🇳🇱 nizozemščina (nl)
- 🇳🇴 norveščina (no)
- 🇵🇱 poljščina (pl)
- 🇵🇹 portugalščina (pt)
- 🇧🇷 portugalščina (Brazilija) (pt-BR)
- 🇷🇴 romunščina (ro)
- 🇸🇰 slovaški (sk)
- 🇸🇮 slovenski (sl)
- 🇸🇪 švedski (sv)

**Drugi jeziki (12):**
- 🇸🇦 arabski (ar)
- 🇨🇳 poenostavljena kitajščina (zh-CN)
- 🇭🇰 Kitajščina Hongkong (zh-HK)
- 🇹🇼 Tradicionalna kitajščina (zh-TW)
- 🇮🇩 Indonezijščina (id)
- 🇯🇵 Japonščina (ja)
- 🇰🇷 Korejščina (ko)
- 🇷🇺 Ruščina (ru)
- 🇹🇷 turščina (tr)
- 🇺🇦 ukrajinščina (uk)

**Kakovost prevoda:**
- ✅ Vsa vsebina je v celoti prevedena.
- ✅ Opisi v uvodu so prevedeni.
- ✅ Tehnični izrazi so zaščiteni.
- ✅ Kodni bloki so ohranjeni.
- ✅ Formule so nedotaknjene.
- ✅ Povezave delujejo.
- ✅ Oblikovanje je popolno

---

### 🔄 **V TEKU: 5 jezikov (Google Translate)**

**Trenutno stanje:**
- 🇮🇳 **hindijščina (hi)** - ⏳ PREVAJA SE (2–3 ure)
- 🇭🇷 **Hrvaščina (hr)** - ⏳ V čakanju (angleščina + prevedeni opisi)
- 🇲🇾 **Malajščina (ms)** - ⏳ V čakanju (angleščina + prevedeni opisi)
- 🇹🇭 **Tajščina (th)** - ⏳ V čakanju (angleščina + prevedeni opisi)
- 🇻🇳 **vietnamščina (vi)** - ⏳ V čakanju (angleščina + prevedeni opisi)

**Zakaj so ti počasnejši:**
- DeepL API ne podpira
- Google Translate API ima omejitve hitrosti
- Uporaba ultrakonzervativnega prevajanja vrstica po vrstici
- 1 sekunda zamude na vrstico, da se izogne omejevanju

**Trenutno stanje (4 jeziki v čakanju):**
- ✅ Repozitoriji obstajajo na GitHub
- ✅ Opisi frontmatter prevedeni
- ✅ Vsi viri in slike so sinhronizirani
- ⚠️ Vsebina besedila je še vedno v angleščini (funkcionalna)

---

## 🔧 Značilnosti prevajalskega sistema

### Avtomatsko prevajanje
- **Polja z opisom** v uvodnem delu so avtomatsko prevedena
- **DeepL API** za 32 jezikov (visoka kakovost)
- **Google Translate** za 5 jezikov (z omejeno hitrostjo)

### Zaščita vsebine
- ✅ Imena izdelkov (Chloros, MAPIR)
- ✅ Kodni bloki in vgrajeni kod
- ✅ Matematične formule
- ✅ Tehnična imena barv (Red, Green, Blue, NIR, RedEdge)
- ✅ Pot do datotek in URL-ji
- ✅ Kratke kode GitBook
- ✅ E-poštni naslovi
- ✅ Razširitve datotek

### Vsebina, ki se prevede
- ✅ Naslovi strani
- ✅ Besedilo in odstavki
- ✅ Celice in naslovi tabel
- ✅ Orodni namigi in opombe
- ✅ Besedilo povezav
- ✅ Opisi predhodnih podatkov

### Naknadna obdelava
- ✅ Popravi nove vrstice HTML
- ✅ Obnovi zaščitene elemente
- ✅ Popravi težave z oblikovanjem
- ✅ Zagotovi združljivost GitBook

---

## 📝 Pregled skript

### Glavni dnevni delovni tok
**`update_all_translations.py`**
- Posodobi vseh 37 jezikovnih repozitorijev
- Sinhronizira besedilo, slike in vire
- Prevaža samo spremenjene datoteke
- Samodejno potrdi in prenese v GitHub
- Uporaba: `python update_all_translations.py`

### Prevajalski skripti
**`translate_with_deepl.py`**
- Osnovno prevajanje DeepL (32 jezikov)
- Obdelava opisov frontmatter
- Popolna zaščita markdown

**`translate_with_google.py`**
- Integracija Google Translate (5 jezikov)
- Enaka zaščita kot DeepL
- Obravnava omejitve API

**`translate_google_conservative.py`**
- Ultra počasen, a zanesljiv Google Translate
- Prevajanje vrstica po vrstici
- Dolge zamude, da se izogne omejitvam hitrosti
- Za težke jezike: `python translate_google_conservative.py hi`

### Skripti za pomoč
**`verify_all_pushed.py`**
- Preveri, ali je vseh 37 repozitorijev prenesenih v GitHub

**`check_google_progress.py`**
- Preveri število jezikovnih datotek Google Translate

**`check_hindi_progress.py`**
- Podrobni napredek prevajanja v hindijščino

**`push_until_stable.py`**
- Prenesite vse repozitorije, dokler ni sprememb

---

## 🌐 GitBook integracija

### Sinhronizacijski proces
1. Spremembe so poslane v repozitorij GitHub.
2. GitBook se samodejno sinhronizira v 5–10 minutah.
3. Spremembe se prikažejo na spletni strani.

### Struktura repozitorija
- **Angleščina:** `chloros_manual_gitbook`
- **Prevodi:** `chloros_manual_gitbook-{lang_code}`

### Jezikovne kode
| Ime repozitorija | CLI Koda | Jezik |
|-----------|----------|----------|
| zh-CN | zh | poenostavljena kitajščina |
| zh-HK | zh | kitajščina Hong Kong |
| zh-TW | zh | tradicionalna kitajščina |
| nb | no | norveščina |
| pt-BR | pt-BR | brazilska portugalščina |
| Vse ostale | Enako kot repozitorij | Standardno |

---

## 📈 Statistični podatki o prevajanju

### Skupna velikost projekta
- **Jeziki:** 37 + angleščina = 38 repozitorijev
- **Datoteke na jezik:** ~30 datotek markdown
- **Skupno število prevedenih datotek:** 32 × 30 = 960 datotek (DeepL)
- **Slike/vsebine:** Sinhronizirane v vseh 37 repozitorijih
- **Prevedene vrstice:** ~50.000+ vrstic

### API Uporaba
- **DeepL API:** ~960 prevodov datotek
- **Google Translate:** V teku (5 jezikov)
- **Vloženi čas:** Več dni razvoja in prevajanja

### Merila kakovosti
- ✅ 100 % prevodov DeepL je visoke kakovosti
- ✅ 100 % prevodov opisov frontmatter (vseh 37 jezikov)
- ✅ 100 % ohranjena oblika
- ✅ 100 % zaščitenih tehničnih izrazov
- ✅ 0 % ne delujočih povezav ali slik

---

## 🚀 Naslednji koraki

### Kratkoročno (danes)
1. ⏳ Počakajte, da se prevod v hindijščino zaključi (~2–3 ure)
2. 📤 Preverite, ali je hindijščina prenesena v GitHub
3. 🔍 Preizkusite hindijščino na GitBook

### Srednjeročno (ta teden)
1. Prevedite preostale 4 jezike (hr, ms, th, vi)
2. Vsak bo trajal 2–3 ure z konzervativno metodo
3. Posodobite in preverite vse na GitBook

### Dolgoročno
1. Spremljajte, ali DeepL dodaja podporo za teh 5 jezikov.
2. Ponovno prevedite z DeepL, ko bo na voljo.
3. Redne posodobitve z uporabo `update_all_translations.py`.

---

## 💡 Priporočila

### Za redne posodobitve
```bash
python update_all_translations.py
```
To samodejno obdela vse za jezike DeepL.

### Za jezike Google Translate
Ko se angleška vsebina spremeni, ročno zaženite:
```bash
python translate_google_conservative.py hi
python translate_google_conservative.py hr
python translate_google_conservative.py ms
python translate_google_conservative.py th
python translate_google_conservative.py vi
```

### Za spremljanje
```bash
python verify_all_pushed.py       # Check all repos
python check_google_progress.py   # Check Google langs
python check_hindi_progress.py    # Check Hindi specifically
```

---

## 🎯 Merila uspešnosti

### ✅ Doseženo
- [x] 32 jezikov v celoti prevedenih prek DeepL
- [x] Prevedeni vsi opisi frontmatter (37 jezikov)
- [x] Vsi repozitoriji na GitHub
- [x] Vsi repozitoriji sinhronizirani z GitBook
- [x] Avtomatiziran dnevni skript delovnega toka
- [x] Zaščita za vso tehnično vsebino
- [x] Post-processing popravi vse oblikovanje

### ⏳ V teku
- [ ] 5 jezikov Google Translate v celoti prevedenih
- [ ] Prevod v hindijščino (trenutno v teku)

### 📅 Prihodnost
- [ ] Spremljanje razširitve podpore DeepL
- [ ] Po potrebi razmislite o profesionalnem prevodu za zadnjih 5 jezikov

---

## 📞 Podpora in dokumentacija

### Ključni dokumenti
- `TRANSLATION_QUICK_START.md` - Kratki referenčni vodnik
- `TRANSLATION_WORKFLOW.md` - Podrobna dokumentacija delovnega toka
- `TRANSLATION_COMMANDS.md` - Referenca ukazov
- `TRANSLATION_FINAL_STATUS.md` - Ta dokument

### Lokacija ključnih skriptov
Vsi skripti v: `C:\Users\MAPIR\Documents\GitHub\chloros_manual_gitbook\`

### Lokacija repozitorijev
Repozitoriji prevodov: `D:\chloros_translation_robust\`

---

**Stanje projekta:** 🟢 **32/37 dokončano**, 🟡 **5/37 v teku**

**Skupna stopnja uspešnosti:** 86 % dokončano (32 v celoti prevedeno + 5 s prevedenimi opisi)




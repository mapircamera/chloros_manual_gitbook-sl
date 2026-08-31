# Podprti jeziki

Chloros zagotavlja popolno podporo vmesnika v **38 jezikih po vsem svetu**, kar ga naredi dostopnega uporabnikom po vsem svetu. Jezike lahko takoj preklopite tako v namiznem grafičnem vmesniku kot tudi v CLI.

Chloros podpira naslednje jezike:

| # | Jezik | Izvirno ime | CLI koda |
|---|----------|-------------|----------|
| 1 | 🇺🇸 angleščina | English | `en` |
| 2 | 🇪🇸 španščina | Español | `es` |
| 3 | 🇵🇹 Portugalščina | Português | `pt` |
| 4 | 🇫🇷 francoščina | Français | `fr` |
| 5 | 🇩🇪 nemščina | Deutsch | `de` |
| 6 | 🇮🇹 italijanščina | Italiano | `it` |
| 7 | 🇯🇵 Japonščina | 日本語 | `ja` |
| 8 | 🇰🇷 Korejščina | 한국어 | `ko` |
| 9 | 🇨🇳 Kitajščina (poenostavljena) | 简体中文 | `zh` |
| 10 | 🇹🇼 Kitajščina (tradicionalna) | 繁體中文 | `zh-TW` |
| 11 | 🇷🇺 Ruščina | Русский | `ru` |
| 12 | 🇳🇱 Nizozemščina | Nederlands | `nl` |
| 13 | 🇸🇦 Arabščina | العربية | `ar` |
| 14 | 🇵🇱 Poljščina | Polski | `pl` |
| 15 | 🇹🇷 Turščina | Türkçe | `tr` |
| 16 | 🇮🇳 hindijščina | हिंदी | `hi` |
| 17 | 🇮🇩 Indonezijščina | Bahasa Indonesia | `id` |
| 18 | 🇻🇳 Vietnamščina | Tiếng Việt | `vi` |
| 19 | 🇹🇭 Tajski | ไทย | `th` |
| 20 | 🇸🇪 Švedski | Svenska | `sv` |
| 21 | 🇩🇰 Danski | Dansk | `da` |
| 22 | 🇳🇴 Norveški | Norsk | `no` |
| 23 | 🇫🇮 finščina | Suomi | `fi` |
| 24 | 🇬🇷 grščina | Ελληνικά | `el` |
| 25 | 🇨🇿 češčina | Čeština | `cs` |
| 26 | 🇭🇺 madžarščina | Magyar | `hu` |
| 27 | 🇷🇴 Romunščina | Română | `ro` |
| 28 | 🇺🇦 Ukrajinščina | Українська | `uk` |
| 29 | 🇧🇷 brazilska portugalščina | Português Brasileiro | `pt-BR` |
| 30 | 🇭🇰 kantonščina | 粵語 | `zh-HK` |
| 31 | 🇲🇾 Malajščina | Bahasa Melayu | `ms` |
| 32 | 🇸🇰 Slovakščina | Slovenčina | `sk` |
| 33 | 🇧🇬 Bolgarščina | Български | `bg` |
| 34 | 🇭🇷 Hrvaščina | Hrvatski | `hr` |
| 35 | 🇱🇹 Litovščina | Lietuvių | `lt` |
| 36 | 🇱🇻 Latvijščina | Latviešu | `lv` |
| 37 | 🇪🇪 Estonščina | Eesti | `et` |
| 38 | 🇸🇮 Sloveneščina | Slovenščina | `sl` |

## Kako spremeniti jezik

### V programu Chloros Desktop

1. Odprite nastavitve programa
2. Prejdite v meni za izbiro jezika
3. Iz seznama izberite želeni jezik
4. Vmesnik se bo takoj posodobil

### V Chloros CLI

Uporabite ukaz `language` za ogled ali spremembo jezika vmesnika CLI:

```bash
# View current language
chloros-cli language

# Change to Spanish
chloros-cli language es

# Change to Chinese (Simplified)
chloros-cli language zh

# Change to Brazilian Portuguese
chloros-cli language pt-BR

# List all available languages
chloros-cli language --list
```

Za več podrobnosti glejte [dokumentacijo CLI](CLI.md).

## Pokritost

Vseh 38 jezikov je v celoti podprtih v:

* **Chloros Desktop** – popoln prevod grafičnega uporabniškega vmesnika
* **Chloros CLI** – vmesnik ukazne vrstice in izhodna sporočila

Python SDK API in njegova [referenčna dokumentacija](reference/sdk-reference.md) sta na voljo v angleščini.

Podpora za različne jezike zagotavlja, da lahko uporabniki po vsem svetu učinkovito delajo v svojem maternem jeziku brez ovir.

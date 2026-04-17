# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Single-file browser tool (`index.html`). No build step, no package manager, no dependencies to install — open directly in any browser.

External dependency loaded via CDN: **SheetJS** (`xlsx-0.20.3`) for client-side Excel parsing.

## Architecture

Everything lives in `index.html`: inline CSS, HTML, and a `<script>` block. The logical sections are:

### i18n (`i18n` object)
Translations for PT / EN / ES. Every user-visible string lives here. Dynamic strings are functions `(arg) => string`. Applied via `t(key, ...args)` and `setLang(lang)` which updates all `[data-i18n]` elements and re-renders the guide.

### Global state (7 variables)
| Variable | Purpose |
|---|---|
| `currentLang` | Active language code |
| `camposDefinicao` | Array of `{nome, tamanho, alinhamento, tipo}` after field-def filtering |
| `dadosLinhas` | Raw row objects from the data spreadsheet |
| `outputTexto` | Generated fixed-width text (Step 3) |
| `outputCurl` | Generated `.sh` bash script (Step 4) |
| `outputPs1` | Generated `.ps1` PowerShell script (Step 4) |
| `authToken` | JWT/token obtained from the auth endpoint |

### Field definition filtering (Step 1)
Reads sheet `Campos Entrada` (falls back to first sheet). Headers expected on **row 2** (index 1), data from **row 3** (index 2). Keeps only rows where `Entrada = S` **and** `CampoPresenteTransacao = S`. Column positions are detected by header name via `findCol()` — order in the spreadsheet doesn't matter.

### Layout generation (Step 3 — `gerarLayout`)
Iterates `dadosLinhas`, pads/truncates each field to its defined size. `AlinhamentoCampo = D` → `padStart` (right-align); anything else → `padEnd` (left-align). Missing fields are silently filled with spaces.

### cURL / script generation (Step 4 — `gerarCurl`)
Calls `buildPayload(row, institution, eventType, now)` for each row, producing the fixed JSON structure `{institutionAcronym, eventTypeCode, map}`. `CURL_NUMERIC_FIELDS` drives type coercion — fields in that Set are cast to `parseFloat`, others to `String`. `recordCreationDate` is always the generation timestamp.

Produces three output formats simultaneously:
- **`.sh`** — bash script with `grep`-based token extraction
- **`.ps1`** — PowerShell script using `Invoke-RestMethod`
- **`.bat`** — self-contained double-click launcher; embeds the PS1 as UTF-16 LE Base64 passed to `powershell -EncodedCommand` (see `ps1ToBase64`)

### Authentication (`testarAuth` / `fetchToken`)
POST to the auth URL with `{username, password, institutionAcronym}`. Token extracted from response fields in order: `token → accessToken → access_token → id_token`.

## Adding a New Output Field to the cURL Map

1. Add the field name to `CURL_MAP_FIELDS` (preserves template order).
2. If numeric, add it to `CURL_NUMERIC_FIELDS`.
3. No other changes needed — `buildPayload` iterates `CURL_MAP_FIELDS` automatically.

## Adding a New Language

1. Add a new key block to the `i18n` object matching the structure of `pt`/`en`/`es`.
2. Add a `<button class="lang-btn" onclick="setLang('xx')">XX</button>` in the header.
3. Add the lang code to the `['PT','EN','ES']` array in `setLang`.

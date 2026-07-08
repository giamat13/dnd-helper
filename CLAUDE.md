# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

ארגז הכלים של ההרפתקן (D&D Adventurer's Toolkit) — a single-file, no-build, client-only D&D 5E character/session helper. The entire app (HTML, CSS, and JS) lives in one file: [index.html](index.html). There is no `package.json`, no bundler, no test suite, and no server-side code. The UI is in Hebrew with `dir="rtl"`.

## Running / developing

There is no build step. Just open `index.html` in a browser, or serve it with a static file server. The repo has `.vscode/settings.json` configured for the "Live Server" extension (port 5501) — right-click `index.html` → "Open with Live Server" is the expected dev loop.

Because the app fetches from `https://api.open5e.com` on load, features (class/race lists, encyclopedia) require network access; a local `file://` open works but CORS/fetch behavior is more reliable when served over `http://localhost`.

There are no lint or test commands — verify changes by reloading the page in a browser and exercising the relevant tab.

## Architecture

Everything is inside the single `<script>` block of [index.html](index.html). It follows a tiny hand-rolled "state + render" pattern (no framework):

- **`state`** (global object, ~line 426) holds all persisted app data: character info, spells, spell slots, "specials" (limited-use abilities), attack combos, dice history, HP/resources, gold/inventory, and the encyclopedia UI state.
- **`open5e`** (global object, ~line 502) holds live data fetched from the Open5e API (classes, races) plus a hardcoded `OPEN5E_FALLBACK_CLASSES`/`OPEN5E_FALLBACK_RACES` used when the API is unreachable.
- **Render functions** (`renderSetupTab`, `renderSpellsTab`, `renderFighterTab`, `renderDiceTab`, `renderResourcesTab`, `renderLootTab`, `renderEncyclopediaTab`) each fully re-render their `<section class="pane">` via `innerHTML` and then re-attach event listeners. There is no diffing — every state change re-renders the whole active pane by calling the tab's own render function (or `renderMain()` to swap tabs).
- **`TABS`** array (~line 635) drives the nav; tabs can be conditionally visible (`visible: ()=>...`) — e.g. the "לוחשות" (Spells) tab only shows for spellcasting classes and "לוחם" (Fighter) only for non-casters, decided by `currentClassIsCaster()` which checks `spellcasting_ability` from the Open5e class data.
- **Persistence**: `saveState()`/`loadState()` serialize `state` to `localStorage` (`STORAGE_KEY = 'dnd-helper-state'`) on every render. The `ency` (encyclopedia) branch is intentionally excluded from persistence except for the selected category, since its results are always re-fetched live.
- **Derived stats**: `applyLevelDerivedStats()` recomputes spell slot maxima from character level using `FULL_CASTER_SLOT_TABLE` whenever level or class changes; `proficiencyBonusForLevel()` and `estimateMaxHp()` are pure helpers used only for display/suggestions in the Setup tab (they don't overwrite user-entered values automatically — the user must click an "Apply" button).

### Tabs (in `visibleTabs()` order)

0. **Setup (הגדרת דמות)** — one-time character config: name, level, class/race (from Open5e), HP, hit die, spell slot maxima. Other tabs read from this but only update *current* state (damage taken, slots used, etc.), not maxima.
1. **Spells (לוחשות)** — spellbook with free-text tags (comma-separated, not fixed categories), per-spell "uses per day" counters (for at-will/innate-style spells that don't consume slots), a spell-slot pip tracker, and a "cheapest available spell for tag X" recommender (`recommendCheapest`).
2. **Fighter (לוחם)** — limited-use "specials" (rage, second wind, etc.) with short/long-rest reset tracking, a crit damage calculator, and a multi-attack "combo" builder/calculator.
3. **Dice (קוביות)** — dice roller with advantage/disadvantage, plus a 20,000-trial Monte Carlo simulator that renders a histogram.
4. **Resources (משאבים)** — current HP / temp HP tracking with damage/heal buttons.
5. **Loot (שלל וזהב)** — coin purse (gold/silver/copper) and inventory with a quick "sell" action.
6. **Encyclopedia (אנציקלופדיה)** — live browser for the full Open5e SRD dataset (spells, monsters, magic items, weapons, armor, races, classes, feats, conditions, backgrounds, planes, rules sections, spell lists, source documents). Supports search, pagination (`next` cursor), and a detail modal (`encyDetailBody` switches rendering per category). Spells/weapons/magic items can be added directly into the character's spellbook/inventory (`addSpellFromApi`, `addWeaponFromApi`, `addMagicItemFromApi`).

### Rest handling

`doShortRest()` and `doLongRest()` are global actions invoked from buttons on multiple tabs (Spells, Fighter, Resources) — a short/long rest always applies to the whole character (slots, specials, spell "uses per day", HP) regardless of which tab triggered it, not just the resource local to that tab.

### Open5e integration details

- `loadOpen5eData()` runs on load, fetches classes+races in parallel, filters races to `document__slug === 'wotc-srd'` only (official SRD content, no third-party), and falls back to the hardcoded lists on any failure (network error or non-OK response), setting `open5e.status` to `'ready'` or `'error'` accordingly (surfaced to the user in the Setup tab).
- The encyclopedia's markdown-ish description renderer (`fmtDesc`, `mdTable`, `splitGluedTableHead`) contains defensive parsing for known Open5e source-data quirks: tables sometimes have their header glued to preceding prose text with no newline, or are missing a closing `|` on trailing rows. Preserve this behavior if touching that code — it's handling real malformed upstream data, not speculative edge cases.

### Conventions specific to this file

- All user-facing strings are Hebrew; keep new UI text consistent with the existing RTL Hebrew style.
- `esc()` is used for all interpolated user/API content in `innerHTML` templates — always escape when adding new dynamic HTML to avoid XSS, since spell/monster/item names and descriptions come from a remote API.
- `uid()` generates local-only random IDs (not persisted server-side) for list items (spells, specials, combo rows, inventory).
- CSS custom properties in `:root` (`--gold`, `--combat`, `--nature`, `--utility`, etc.) define the color system; reuse these variables rather than hardcoding colors when adding UI.

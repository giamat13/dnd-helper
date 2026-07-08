# ארגז הכלים של ההרפתקן — D&D 5E Toolkit

A lightweight, client-only toolkit for managing D&D 5E characters during play. Track spells, combat abilities, resources, loot, and access the complete SRD reference — all in a single HTML file with no dependencies.

**Interface language:** Hebrew (RTL) | **No build required** | **No server needed**

## Features

- **Character Setup** — Define name, level, class, race, HP, and spell slots. Class/race data is fetched live from the [Open5e API](https://open5e.com) (with local fallback).
- **Spellbook** — Manage spells with free-form tags, track spell slots and "uses per day" counters, and get automatic recommendations for the cheapest available spell for a given tag.
- **Fighter Abilities** — Track limited-use class features (specials), calculate critical damage, and plan multi-attack combos in a single round.
- **Dice Roller** — Roll with advantage/disadvantage, apply modifiers, and run 20,000-trial Monte Carlo simulations to visualize probability distributions.
- **HP & Resources** — Track current/temporary HP with damage and healing buttons; apply short and long rests across the whole character.
- **Inventory & Loot** — Manage coins (gold, silver, copper) and loot with quick-sell actions.
- **Encyclopedia** — Browse the entire D&D 5E SRD: spells, monsters, items, weapons, armor, classes, races, feats, conditions, and more. Add spells/weapons/items directly to your character.

## Getting Started

### Quick Start

1. Clone or download the repo
2. Open `index.html` in a web browser
3. Start using — data saves to browser localStorage automatically

### With Live Server (Recommended for Development)

The repo includes VS Code Live Server configuration (port 5501):

```bash
# In VS Code:
# Right-click index.html → "Open with Live Server"
```

Or use any static file server:

```bash
# Python 3
python -m http.server 8000

# Node.js (npx http-server)
npx http-server

# Then open http://localhost:8000
```

## How It Works

- **No build step**, no Node.js, no package.json — the entire app is one `index.html` file (~2000 lines).
- **All client-side** — data lives in browser localStorage; no backend.
- **Live SRD data** — classes, races, and encyclopedia entries are fetched from [Open5e API](https://api.open5e.com) on load. The app includes hardcoded fallback data so it works offline.
- **Responsive design** — works on desktop and tablet; mobile-friendly layout at 640px and below.

## Tabs Overview

| Tab | Purpose |
|-----|---------|
| **הגדרת דמות** (Setup) | Character basics: name, level, class, race, HP, spell slots |
| **לוחשות** (Spells) | Spellbook with tags, slot tracking, per-spell "uses per day" (casters only) |
| **לוחם** (Fighter) | Special abilities, crit calculator, multi-attack combos (non-casters only) |
| **קוביות** (Dice) | Roller with advantage/disadvantage and probability simulator |
| **משאבים** (Resources) | HP, temporary HP, short/long rests |
| **שלל וזהב** (Loot) | Coins and inventory management |
| **אנציקלופדיה** (Encyclopedia) | Full SRD browser with search; add spells/items to your character |

## Persistence

- **Character data** is saved to browser `localStorage` every time you make a change.
- **Encyclopedia results** are fetched live and not persisted (only your selected category is remembered).
- **Clearing browser data** will erase your character. Back up important data if needed.

## Keyboard Shortcuts

- In the encyclopedia search, press **Enter** to search.
- In spell tag fields, press **Enter** to confirm tags.

## Offline Use

The app requires network access on first load to fetch class/race data from Open5e. After that, it stores fallback SRD data and can work offline, though encyclopedia features will not function without a connection.

## Development Notes

See [CLAUDE.md](CLAUDE.md) for architecture, code structure, and development guidance.

## License

This project is open-source. The data comes from [Open5e](https://open5e.com), which provides the D&D 5E SRD under the Open Game License.

## Troubleshooting

**Encyclopedia doesn't load?**
- Check your internet connection; Open5e API is required for catalog browsing.
- If the API is down, the app will still work with fallback class/race data for character creation.

**Data disappeared after closing the browser?**
- Check your browser's privacy settings; localStorage might be disabled or cleared on exit.

**Class/race list is incomplete?**
- This can happen if the Open5e API is unreachable. A limited fallback list will be used instead.

## Contributing

Found a bug or have a feature idea? Open an issue or a pull request.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

GetLando — a Formula 1 reskin of GetKRCS, itself a reskin of [GetPVI](https://github.com/getpvi/GetPVI), a [2048](http://gabrielecirulli.github.io/2048/) clone. Plain static site, no build tooling, no package manager, no test suite. Live at https://dcarley04.github.io/GetLando/, served directly from `master` via GitHub Pages.

## Commands

- **Run locally**: `python3 -m http.server 8642` from the repo root, then open `http://localhost:8642/`. No install step — it's static HTML/CSS/JS served as-is.
- **Lint JS**: `npx jshint js/*.js` (config in `.jshintrc`: 2-space indent, 80-char lines, ES-next, no unused vars). Not installed locally or wired into any script; run via `npx`.
- **No build step**: `style/main.css` is hand-edited directly and is what the page actually loads. `style/main.scss` is stale/unused leftover from upstream GetPVI — don't try to keep it in sync, and don't add a Sass build step for it.
- **Deploy**: push to `main`; GitHub Pages serves the repo root directly. No CI, no deploy script.

## Architecture

Vanilla JS, no framework, no modules (plain `<script>` tags loaded in dependency order in `index.html`). The whole game boots in `js/application.js`:

```js
new GameManager(4, KeyboardInputManager, HTMLActuator, LocalScoreManager);
```

`GameManager` (`js/game_manager.js`) is the hub — owns game state (score, won/over/keepPlaying) and orchestrates the other four constructor-injected pieces:
- **`Grid`** (`js/grid.js`) — the 4×4 cell array; pure data structure, no rendering.
- **`Tile`** (`js/tile.js`) — a single tile's position/value/merge-tracking; also pure data.
- **`KeyboardInputManager`** (`js/keyboard_input_manager.js`) — arrow keys, vim (hjkl), WASD, and touch swipes, all normalized to one `emit("move", direction)` event (0=up, 1=right, 2=down, 3=left). `GameManager` subscribes via `.on("move"/"restart"/"keepPlaying", ...)`.
- **`HTMLActuator`** (`js/html_actuator.js`) — the only piece that touches the DOM; renders `Grid` state into `.tile-container` and applies `tile-N` CSS classes per value.
- **`LocalScoreManager`** (`js/local_score_manager.js`) — best-score persistence via `localStorage`, with an in-memory fallback if storage is unavailable.

**Tile → driver image mapping** lives entirely in `style/main.css` (not JS, not `main.scss`): `.tile.tile-{2,4,8,...,2048} .tile-inner { background: url('../img/X.png'); }`, plus `.tile-super` for anything above 2048. To change which driver sits on which tile, edit the URL in that rule and swap the PNG in `img/` — nothing else needs to change. Current ladder (documented in `README.md`): 10 drivers ascending 2→1024, Lando Norris at 2048, Super Lando beyond.

**Header easter egg**: the Lando headshot in the title bar and the "Lando Norris!" text in the intro paragraph both toggle a single `body.show-teams` class (defined in `main.css`) that tiles `img/teams.png` (a 2×2 composite of the McLaren, Ferrari, Mercedes, and Red Bull logos) across the page background. Both triggers share that one class specifically so they can't desync into inconsistent states — don't reintroduce a second, independent inline-style toggle.

## Credits / licensing

MIT-licensed (see `LICENSE.txt`), forked from GetKRCS, itself forked from GetPVI by Declan Emery, inspired by GetMIT and Get Caltech!, descending from Gabriele Cirulli's 2048. Driver photos and team logos belong to Formula 1 and the respective teams, used non-commercially. Preserve this credit chain in `README.md` if it's touched.

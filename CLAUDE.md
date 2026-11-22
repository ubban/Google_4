# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up played in the browser on an HTML5 canvas.

- `index.html` — the entire game: markup, CSS, and JavaScript in one file (~1,670 lines).
- `README.md` — minimal project readme.

There is no build step, package manager, bundler, test suite, or linter. Do not add `package.json`, a framework, or a build pipeline unless the user asks for one.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

Verify changes by playing the game in a browser. There are no automated tests to run.

## Code layout inside `index.html`

- **`<style>`** (lines 10–359): all styling, including the HUD, overlays, touch controls, and the `.reduced-motion` class.
- **`<body>`** (lines 361–421): the canvas (`#gameCanvas`), HUD elements, overlay/menu, difficulty buttons, touch controls, and the `#srLive` ARIA live region.
- **`<script>`** (lines 422–1667): one IIFE (`'use strict'`) containing all game logic. Key sections, in order:
  - DOM references and `STORAGE_*` localStorage keys
  - Adaptive quality (`trackPerf`, `quality` 0–2) that reduces effects when frame times are high
  - Config tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`
  - Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` table (Web Audio, muteable)
  - HUD/visual effects: hearts, bombs, combo pop, stage banner, screen shake, flashes
  - Input: keyboard, touch joystick, fire/bomb buttons, and mouse on the canvas
  - Game state: `resetGame`, `update(dt)`, `draw()`, `loop(now)`
  - Enemies, bosses (`BOSS_PATTERNS` bullet patterns), power-ups, bombs, weapons, and scoring
  - Game flow: `loseLife`, `endGame`, `togglePause`, `startBtn` handler

## Conventions

- **Keep it a single file.** New game code goes in the existing `<script>` IIFE, and new styles go in the existing `<style>` block, unless the user asks for a split.
- **Data-driven tuning.** Balance changes (enemy stats, difficulty values, boss HP multipliers, stage pacing) belong in the config tables near the top of the script, not hard-coded in update logic.
- **Persisted settings use the `skyfighter.*` localStorage keys** (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Keep that prefix if adding new keys. Storage can throw in private browsing or when disabled, so guard new storage reads and writes.
- **Accessibility.** Respect `prefers-reduced-motion` (the in-game motion toggle reads and writes it). Announce important state changes through `announce()` so screen readers hear them via `#srLive`. Keep the viewport meta tag and touch-friendly controls working at phone widths.
- **Coordinate system.** Gameplay is designed on a fixed `BASE_W × BASE_H` (480 × 720) logical canvas, scaled by `resize()`. Position and size new entities in those units.
- **Time-based updates.** Movement and timers use `dt` (seconds, clamped to 0.05). Do not tie gameplay speed to frame count.
- **Pause and game-over.** Any new timed effect should respect `paused` and `running` so it freezes correctly when the game is paused or ends.
- **No new dependencies or external network calls** without the user's approval; the game is intended to work offline from a single file.

## Working in this repo

- Make small, focused changes; this is a single file, so keep diffs tidy and avoid reformatting unrelated code.
- After changing gameplay or UI, open the game in a browser and check the affected behavior (start, shooting, bombs, boss fights, pause, game over, mute, difficulty, and touch controls where relevant).
- Git: the working branch for this project is `claude/gifted-tesla-fjimug`. Commit and push there; do not push to other branches without permission. Do not open pull requests unless asked.

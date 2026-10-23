# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small static project containing one game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up.

- `index.html` — the entire game: markup, CSS, and JavaScript in a single file (~1,670 lines).
- `README.md` — a one-line title only.

There is no build system, package manager, bundler, test suite, linter, or CI configuration. Nothing needs to be installed.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000
```

There are no automated tests. Verify changes by playing the game in a browser (desktop and a touch-sized viewport), since most behavior is visual, timing-based, or audio-driven.

## Code layout inside `index.html`

- `<style>` (about lines 10–360): dark-scheme styles. Layout uses a fixed full-viewport `#container` with a centered `#gameCanvas`. The HUD (`#hud`), overlays (`#overlay`), touch controls (`#touchControls`), and the boss bar (`#bossBarWrap`) are absolutely positioned on top of the canvas.
- `<body>` markup (about lines 362–420): HUD, boss bar, top-bar buttons (mute, reduced motion, pause), touch controls, and the start/pause overlay with difficulty buttons. The control hints in `#controlsHint` describe the controls and should be kept in sync with the input code.
- `<script>` (about lines 422–1667): one IIFE that holds all game state. Sections are marked with `// ---------- … ----------` comments:
  - **Config constants**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
  - **Persistence**: `localStorage` keys prefixed `skyfighter.` (best score, mute, difficulty, reduced motion). Every read and write is a direct call, so it can throw in restricted contexts; keep new storage access consistent with that.
  - **Audio**: a WebAudio synth (`ensureAudio`, `beep`, `noiseBurst`, the `sfx` table). No audio assets are loaded.
  - **Canvas sizing**: `resize()` scales a fixed `BASE_W × BASE_H` (480 × 720) play area to fit the window, capped at 1.6×.
  - **Input**: keyboard (`keys` map), mouse (`mouseTarget`), and touch joystick plus FIRE/BOMB buttons.
  - **Game state and entities**: `resetGame`, spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), weapons (`fireWeapon`, `useBomb`), and the main `update(dt)`.
  - **Boss patterns**: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, driven by `updateBoss`.
  - **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
  - **Loop**: `loop(now)` runs `update` and `draw` only while `running && !paused`, with `dt` clamped to 50 ms.

## Game rules worth knowing

- **Stages**: a stage ends after `KILLS_PER_STAGE` (18) kills, which triggers a boss. Defeating a boss advances `stage` and cycles through `THEMES`. `BOSSES` has six entries and is indexed by stage.
- **Enemies**: `grunt` and `interceptor` appear from stage 1. `tank` appears from stage 2.
- **Power-ups**: `life`, `bomb`, and `weapon`. Weapon power-ups raise the level up to 3, then toggle between `vulcan` and `laser`.
- **Difficulty** changes spawn rate, enemy shot chance, starting lives and bombs, and boss HP multiplier. Lives and bombs are capped by `maxLives` and `maxBombs`.
- **Controls**: arrows or WASD to move, Space or hold mouse to fire, X or Shift to bomb, P or Esc to pause.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. The single-file, no-dependency design is deliberate.
- Match the existing style: 2-space indentation, single quotes in JavaScript, `const`/`let`, small named functions, and short section banner comments.
- Do not add external scripts, fonts, or assets. Audio is synthesized and graphics are drawn on the canvas.
- Keep accessibility hooks working: `#srLive` announcements via `announce()`, the reduced-motion toggle (`motionBtn`, which disables shake, flash, and pulse), and the `prefers-reduced-motion` check.
- Preserve mobile behavior: touch controls, `touch-action: none`, safe-area insets, and the no-scroll viewport.
- When changing a gameplay constant, check the HUD text and `#controlsHint` for anything that quotes it.

## Git

- Development happens on feature branches named `claude/<topic>`; `main` is the default branch.
- Write clear, descriptive commit messages that explain the gameplay or UI change.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file, `index.html`, with inline CSS and one inline `<script>` (no build step, no dependencies, no external network calls).

Files:

- `index.html` — the whole game: markup, styles, and game logic (~1,670 lines).
- `README.md` — minimal placeholder.
- `CLAUDE.md` — this file.

## Running and testing

- There is no build, package manager, bundler, linter, or test suite configured.
- To run the game, open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it.
- Verify changes by loading the page and playing: check the overlay/start flow, shooting, bombs, pause, and game over, and check the touch controls if you touched layout or input code.

## Code structure (inside `index.html`)

- **Markup**: `#container` holds `#gameCanvas`, the HUD (`#hud`: score, best, multiplier, weapon, hearts, bombs, boss bar), `#stageBanner`, `#comboPop`, `#topBar` (mute, motion, pause buttons), `#touchControls` (joystick zone and FIRE/BOMB buttons), and `#overlay` (title/start/game-over card).
- **Styles**: a single `<style>` block near the top. Colors are hard-coded hex values, with `color-scheme: dark`.
- **Script**: one IIFE. Major sections, in rough order:
  - Persistence keys (`STORAGE_*`, all prefixed `skyfighter.`) and difficulty tables (`DIFFICULTIES`).
  - Visual and content data: `THEMES` (stage backdrops), `ENEMY_TYPES`, `BOSSES`.
  - Audio: a WebAudio synth (`ensureAudio`, `beep`, `noiseBurst`, and the `sfx` map). No audio files.
  - Input: keyboard (`keydown`/`keyup`), mouse steering and click-to-fire on the canvas, and touch joystick/buttons.
  - Game state and loop: `resetGame`, `update(dt)`, `draw()`, `loop(now)`. `dt` is clamped to 50 ms.
  - Combat: `fireWeapon`, `applyPowerUp`, `useBomb`, enemy spawning, boss spawning and attack patterns (`BOSS_PATTERNS`), `killBoss`, `loseLife`, `endGame`.

## Game rules and constants (keep in sync with the code)

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn rates, enemy speed and fire chance, starting lives and bombs, and a boss HP multiplier.
- **Stages**: a stage ends after `KILLS_PER_STAGE` (18) kills, which triggers a boss. Stage N uses `BOSSES[(N-1) % BOSSES.length]` and `THEMES` cycles the same way.
- **Enemies**: `grunt`, `interceptor` (weaving), and `tank` (4 HP, appears from stage 2).
- **Weapons**: `vulcan` and `laser`, each with 3 levels. Weapon power-ups raise the level; past level 3 the weapon swaps.
- **Controls**: mouse to steer and click to fire; Arrows/WASD to move; Space to fire; X or Shift for a smart bomb; P or Esc to pause. Touch: drag on the left to fly, tap FIRE and BOMB.
- **Persistence**: `localStorage` stores best score, mute, difficulty, and reduced-motion preference. Every access is a plain `localStorage` call, so guard new reads/writes in a `try/catch` if you add them, since storage can be blocked.

## Conventions

- Keep the game in the single `index.html` file unless the user asks for a split. Don't add a build step or external dependencies without asking.
- Don't load external scripts, fonts, or stylesheets. The page is intentionally self-contained.
- Keep the game loop frame-rate independent: scale movement and timers by `dt`.
- Preserve the mobile and accessibility behavior: viewport scaling (`BASE_W` 480 × `BASE_H` 720 via `resize()`), safe-area insets, the reduced-motion toggle, and the `aria-live` announcer (`announce`).
- Keep the in-page help text (`#controlsHint`) and the meta description accurate when controls or features change.

## Git

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-8euaw5`). Don't push to other branches without permission.
- Don't open a pull request unless explicitly asked.

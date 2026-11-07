# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas game.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–360), and JavaScript (`<script>`, ~lines 422–1669).
- `README.md` — minimal project readme.

There is no build step, package manager, bundler, test suite, or linter. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000
```

Then visit `http://localhost:8000/index.html`.

## Code layout inside `index.html`

The script is one IIFE (`(function () { 'use strict'; ... })();`). Keep new code inside it; do not add globals.

Major sections, in file order:

| Area | Key identifiers |
| --- | --- |
| DOM references | `canvas`, `overlay`, `scoreEl`, `heartsEl`, `bombsEl`, `bossBarWrap`, ... |
| Persistence keys | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all prefixed `skyfighter.`) |
| Adaptive quality | `trackPerf`, `quality` (0 full / 1 medium / 2 low) |
| Tuning data | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS` |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `sfx` (Web Audio, synthesized — no audio files) |
| UI feedback | `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake` |
| Input | keyboard handlers, mouse on canvas, touch joystick (`joystickStart/Move/End`), FIRE and BOMB buttons |
| Game state | `resetGame`, `update(dt)`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause` |
| Rendering | `draw`, `drawPlane`, `drawBossShip`, `drawClouds` |
| Main loop | `loop(now)` — calls `trackPerf`, then `update` and `draw` while running and not paused; `dt` is clamped to 50 ms |

### Gameplay model

- **Difficulty** (`easy` / `normal` / `hard`) sets spawn rates, enemy speed and fire chance, starting lives and bombs, and a boss HP multiplier. Changes go in `DIFFICULTIES`.
- **Stages** advance every `KILLS_PER_STAGE` (18) kills. Each stage has a theme from `THEMES` and a boss from `BOSSES` (cycled).
- **Enemies**: `grunt`, `interceptor` (weaving), and `tank` (multi-hit, unlocks at stage 2). Defined in `ENEMY_TYPES`.
- **Weapons**: Vulcan and Laser, with 3 power levels, switched by power-ups (`applyPowerUp`, `fireWeapon`).
- **Bombs**: limited smart bombs (`useBomb`) that clear the screen and damage bosses.
- **Score**: base points per kill multiplied by a kill-chain multiplier (`addScore`, `breakChain`).
- **Persistence**: best score, mute, difficulty, and reduced-motion preference are stored in `localStorage` under `skyfighter.*` keys. Reads and writes are direct (no try/catch), so keep new keys under the same prefix and parse values defensively.

### Conventions

- Use `const`/`let`, arrow functions, and template strings to match the surrounding code. No `var`.
- Keep time-based logic in `dt` (seconds) so behaviour is frame-rate independent.
- Respect `reducedMotion`: any new screen shake, flash, or animation should be disabled or softened when it is on (see the `body.reduced-motion` CSS rules and `screenShake`).
- Route audio through the existing `sfx` helpers rather than creating `AudioContext` nodes ad hoc. Audio must stay muted when `muted` is true.
- Accessibility: announce important state changes through `announce()` (the `srLive` region).
- Mobile: all new controls must work with touch. Keep the joystick and FIRE/BOMB layout working at phone widths; the page uses `viewport-fit=cover` and `resize()` scales the canvas to `BASE_W` × `BASE_H` (480 × 720).
- Do not introduce external libraries, CDN scripts, or network calls. The game is self-contained.

## Testing and verification

There are no automated tests. To verify a change:

1. Open `index.html` in a desktop browser and play through at least one stage boss.
2. Check the mobile layout with browser devtools device emulation (touch controls, portrait and landscape).
3. Confirm the console has no errors.
4. Test each difficulty, mute/reduced-motion toggles, pause (P/Esc), and that best score persists across reloads.

If a headless browser is available, Playwright can load the page and capture screenshots for visual checks.

## Git workflow

- Develop on the branch the task specifies. Do not push to `main` directly.
- Write commit messages that describe the gameplay or code change in the imperative mood, with a short body when the change is non-trivial.
- Do not create a pull request unless explicitly asked.

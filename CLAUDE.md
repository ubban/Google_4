# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser as a single HTML5 canvas page.

- `index.html` — the entire game: markup, CSS, and JavaScript in one file (~58 KB).
- `README.md` — a one-line title; there is no build or setup documentation.

There is no package manager, bundler, test suite, or CI configuration. Do not add one unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server` from the repo root). No install step is needed.

## Code layout inside `index.html`

- **`<style>`** (approx. lines 10–362): all CSS, including the HUD, overlay card, touch controls, and reduced-motion rules. Colors and layout live here.
- **`<body>` markup** (approx. lines 363–421): canvas `#gameCanvas`, HUD elements (`#score`, `#best`, `#mult`, `#weaponLvl`, `#hearts`, `#bombs`), boss bar, stage banner, top buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), touch controls, and the start/title overlay (`#overlay`, `#startBtn`, `.diffBtn`).
- **`<script>`** (approx. lines 422–1670): one IIFE holding all game code. Major sections, in order:
  - Constants: `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
  - Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers (Web Audio, synthesized, no asset files).
  - HUD and effects: `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `flashHit`.
  - Input: keyboard handlers, touch joystick, fire/bomb buttons, mouse steering on the canvas.
  - Game state and logic: `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `update(dt)`, `applyPowerUp`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
  - Rendering: `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
  - Loop: `loop(now)` driven by `requestAnimationFrame`; start button wiring at the bottom.

## Game mechanics reference

- **Controls:** mouse moves and fires on click; Arrows/WASD move; Space fires; X or Shift uses a bomb; P or Esc pauses. Touch uses a left-side drag zone plus FIRE and BOMB buttons.
- **Weapons:** Vulcan / Laser, each with power levels (1–3), upgraded by power-ups.
- **Bombs:** limited count; clear the screen and damage bosses.
- **Stages:** each stage is `KILLS_PER_STAGE` kills, then a boss (from `BOSSES`, with patterns from `BOSS_PATTERNS`).
- **Difficulty:** `easy`, `normal`, `hard` via `DIFFICULTIES`. Spawn rate ramps up with time (`difficultyRamp`).
- **Score:** kill-chain multiplier (`addScore`, `breakChain`).

## Persistence

The game stores these keys in `localStorage`. Changing a key name resets that user's saved value, so keep them stable or migrate deliberately:

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | high score |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | last chosen difficulty |
| `skyfighter.reducedMotion` | `'1'`/`'0'`; when absent, falls back to `prefers-reduced-motion` |

All `localStorage` access should stay defensive, since it can throw in private or restricted browsing modes.

## Conventions

- Keep the game as a single self-contained `index.html` unless the user asks for a split. Do not add external network dependencies; audio is synthesized and graphics are drawn on canvas.
- Preserve the single IIFE structure and avoid new globals.
- Timed logic should use the `dt` passed to `update`. The `frame` counter is used only for coarse ramps (such as the spawn-rate ramp), so don't rely on it for precise timing.
- Respect the reduced-motion setting when adding screen shake, flashes, or pulses.
- Touch and mouse input must both keep working. Test changes to controls on both paths.
- Canvas uses a fixed logical size `BASE_W × BASE_H` (480×720) scaled by `resize()`. Draw in logical coordinates.
- Keep the HUD and overlay readable at phone widths; the page disables scrolling and zoom by design (`touch-action: none`, `user-scalable=no`).

## Verifying changes

There are no automated tests. For gameplay or UI changes:

1. Load `index.html` in a browser (or use the pre-installed Playwright Chromium in cloud sessions).
2. Start a game, exercise the changed path, and check the browser console for errors.
3. For touch changes, emulate a mobile viewport as well as desktop.

## Git

- Development branch for this session: `claude/gifted-tesla-mmarp7`. Push with `git push -u origin <branch>`.
- Commit messages follow the existing style: a short imperative subject line, optionally followed by a bulleted body describing the change.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game lives in a single self-contained file, `index.html` (~1,670 lines: inline CSS, markup, and one inline `<script>`).

- `README.md` — placeholder stub; no real project docs.
- `index.html` — the whole game.

There is no package manager, build step, bundler, test suite, linter, or CI configuration. Nothing needs to be installed.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static file server. No build is required.

## Architecture of `index.html`

The file is organized top to bottom as:

1. **`<style>`** — all CSS, including HUD, overlay, boss bar, and touch-control layouts.
2. **`<body>`** — a single `<canvas id="gameCanvas">` plus HUD/overlay DOM elements (score, best, weapon, hearts, bombs, boss bar, mute/motion/pause buttons, touch controls).
3. **One IIFE `<script>`** wrapping all game logic. Nothing is exported or attached to `window`.

Inside the script, the rough sections are:

| Area | Key identifiers |
|---|---|
| DOM refs | `canvas`, `ctx`, `overlay`, `scoreEl`, `startBtn`, … |
| Persistence | `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`) |
| Config tables | `DIFFICULTIES`, `THEMES` (per-stage backgrounds), `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE` |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `sfx` — Web Audio API, synthesized (no audio files) |
| Feedback | `flashHit`, `flashBomb`, `screenShake`, `showComboPop`, `showStageBanner`, `announce` (screen-reader live region) |
| Input | `keys` map, mouse/touch handlers, `joystick*` helpers, `useBomb` |
| Spawning & combat | `spawnEnemy`, `spawnBoss`, `fireWeapon`, `update(dt)`, boss patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`), `updateBoss`, `killBoss`, `loseLife`, `endGame` |
| Rendering | `drawClouds`, `draw()` |
| Loop | `trackPerf(dt)` adaptive quality, `loop(now)` driven by `requestAnimationFrame` |

Notes on behavior worth knowing before editing:

- **Game loop:** `loop` clamps `dt` to 50 ms, calls `update(dt)` then `draw()` only while `running && !paused`.
- **Adaptive quality:** `trackPerf` watches frame times and sets `quality` (0 full / 1 medium / 2 low) to reduce rendering cost on slow devices.
- **Stages:** after `KILLS_PER_STAGE` (18) kills in a stage, a boss from `BOSSES` spawns (after a short warning delay). Defeating it (`killBoss`) increments `stage`, and the background `THEMES` palette cycles via `(stage - 1) % THEMES.length`.
- **Difficulty:** `DIFFICULTIES` tunes spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier. `setDifficulty` saves the choice and updates the buttons; the button handler ignores changes while a run is active.
- **Fixed logical resolution:** the game is designed at `BASE_W × BASE_H` = 480 × 720 and scaled in `resize()` (capped at 1.6×) to fit the window.
- **Reduced motion:** honors `prefers-reduced-motion` on first load, and the user can override it with the Motion button; the choice persists in `localStorage`.

### Controls

- Keyboard: Arrows/WASD move, Space fire, X or Shift bomb, P or Esc pause.
- Mouse: move to steer, click to fire.
- Touch: drag on the left zone for a virtual joystick, tap FIRE, tap BOMB.

Keep these in sync with the on-screen `#controlsHint` text and the `keydown`/`keyup` handlers near the top of the script.

## Conventions

- **Keep it a single file.** The project's defining property is that it works by opening one HTML file. Do not split out modules, add a build step, or add npm dependencies without the user asking for it.
- **No external network dependencies at runtime.** The only external-looking asset is an inline SVG favicon in a `data:` URI. Keep it that way unless asked.
- **Match existing style:** 2-space indentation, single quotes in JS, `const`/`let` (no `var`), small helper functions, and config-driven tables for enemy and boss tuning rather than hard-coded numbers in logic.
- **Persistence keys** use the `skyfighter.` prefix. Read and write them through `localStorage` with the existing `Number(...) || default` pattern; wrap new storage access if it could throw in restricted browsers.
- **Accessibility:** announce important state changes through the `announce()` helper (the `srLive` region), and respect the reduced-motion setting when adding screen shake, flashes, or other motion effects.
- **Audio is synthesized.** Add new sounds via `beep`/`noiseBurst` and the `sfx` table; do not add audio files.
- **Mobile matters.** Any control change must work with the touch controls and the responsive `resize()` scaling.

## Verifying changes

There is no automated test suite. To verify a change:

1. Open `index.html` in a browser (or serve the folder statically) and check that the console has no errors.
2. Play through the affected path: start, fire, bomb, pause/resume, lose a life, and game over. For stage or boss changes, reach the relevant kill count or boss.
3. Check at least one desktop viewport and one narrow/mobile viewport, and confirm touch controls if input code changed.
4. Confirm the persisted settings (best score, mute, difficulty, motion) still load and save.

Describe in your summary what you actually exercised. If you could not run the game in a browser, say so rather than claiming it works.

## Git

- Development happens on the branch the session specifies. Commit with clear messages and push with `git push -u origin <branch>`.
- Do not force-push shared branches.

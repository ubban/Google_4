# CLAUDE.md

Guidance for working in this repository.

## What this is

A single-file browser game, **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game lives in `index.html`: markup (~lines 1–360), CSS (inline `<style>`), and one IIFE-wrapped `<script>` (starts ~line 422) that drives everything on a `<canvas>`. `README.md` is a placeholder.

There is no build step, no package manager, no bundler, and no test suite. Do not add one unless asked.

## Running it

Open `index.html` in a browser, or serve the directory with any static server, for example `python3 -m http.server`. Desktop and mobile layouts are both supported, so check touch layout changes at phone width as well as desktop.

## Code layout inside `index.html`

- **DOM refs** (top of the script): every element is looked up once by `id` and kept in a `const`. Add new UI the same way.
- **Storage keys**: `STORAGE_*` constants, all prefixed `skyfighter.`. Persisted values are best score, mute, difficulty, and reduced motion.
- **Tuning data** (plain object/array constants): `DIFFICULTIES`, `THEMES` (six stage backdrops), `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, `KILLS_PER_STAGE`. Balance changes usually belong here, not in the logic.
- **Audio**: a Web Audio synth (`beep`, `noiseBurst`, the `sfx` object). Audio is created lazily through `ensureAudio()`, which must run from a user gesture.
- **Game state and loop**: `resetGame()` initializes state. `update(dt)` advances the simulation, `draw()` renders, and `loop(now)` calls both each frame. `dt` is clamped to 50 ms.
- **Weapons and enemies**: `fireWeapon()` handles the two weapon families (`vulcan` spread and `laser`), each with levels 1–3. Enemies are spawned by `spawnEnemy()`, and bosses by `spawnBoss()` and `updateBoss()`, which pick from `BOSS_PATTERNS`.
- **Power-ups**: `maybeSpawnPowerUp()` drops `life`, `bomb`, or `weapon` pickups. `applyPowerUp()` applies them.
- **Adaptive quality**: `trackPerf()` watches frame time and steps a `quality` level (0–2) up or down. Rendering reads this level to reduce effects on slow devices.
- **Accessibility**: `announce()` writes to a live region. A reduced-motion toggle and the `prefers-reduced-motion` media query control screen shake and similar effects.

## Conventions

- Keep it a single file with no external dependencies. Inline all CSS and JS, and use `data:` URIs for icons, as the existing favicon does.
- Match the existing style: 2-space indent, single quotes in JS, `const` by default, and small `function` declarations inside the IIFE.
- Don't introduce globals. Everything stays inside the IIFE.
- `localStorage` is read once at startup and written on settings changes, with no `try`/`catch` today. New storage code should wrap access in `try`/`catch`, because some browser modes throw on access.
- Keep the mobile touch controls (`touchControls`, `moveZone`, `fireBtn`, `bombBtn`) working when you touch input code. Mouse, keyboard, and touch input all feed the same `keys`, `firing`, and `player` state.

## Controls (for testing)

- Keyboard: arrows or WASD to move, Space to fire, X or Shift for a bomb, P or Esc to pause.
- Mouse: move to steer, hold to fire.
- Touch: drag left to fly, tap FIRE, tap BOMB.

## Verification

Since there are no automated tests, verify changes by loading the page in a browser and playing through the affected path. For gameplay or rendering changes, check at least:

- Starting a run, firing, and taking a hit (lives decrease).
- Bomb use, and pause/resume.
- The stage transition after `KILLS_PER_STAGE` kills, and a boss fight.
- Game over, and the best-score update.
- The mobile layout at phone width.

Run the JavaScript through `node --check` after extracting the script, or open the page and watch the browser console for errors.

## Git

- Development branch for this work: `claude/gifted-tesla-t0x5js`.
- Keep commits focused. Write messages in the imperative mood and describe the gameplay or code effect.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small static web project containing **Sky Fighter**, a Raiden-style
vertical shoot-'em-up that runs in a browser canvas.

Files:

- `index.html` — the entire game: HTML, inline CSS in `<style>`, and one inline
  `<script>` IIFE. No build step, no dependencies, no external assets.
- `README.md` — minimal placeholder.

There is no package manager, no test suite, no linter, and no CI configuration.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static
server (e.g. `python3 -m http.server`). Nothing needs to be installed.

## Architecture of `index.html`

Everything runs inside one `(function () { ... })();` IIFE, so no globals leak.
Sections, in file order (approximate):

- **Styles** — `:root { color-scheme: dark }`, HUD pills, vignette, heart/bomb
  icons, `heartBeat` keyframes. The page is locked to the viewport
  (`overflow: hidden`, `touch-action: none`).
- **Markup** — `#container` holds `#gameCanvas`, `#hud`, overlay/menu, touch
  joystick and fire/bomb buttons, and an `aria-live` region for announcements.
- **Config constants** (near the top of the script):
  - `STORAGE_*` keys: `skyfighter.bestScore`, `skyfighter.muted`,
    `skyfighter.difficulty`, `skyfighter.reducedMotion`.
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed, enemy
    fire chance, starting lives, starting bombs, boss HP multiplier.
  - `KILLS_PER_STAGE = 18` — kills needed to advance a stage.
  - `THEMES` — six sky palettes cycled per stage.
  - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP, stage 2+).
  - `BOSSES` — six bosses (VANGUARD → TITAN), each with its own palette and HP mul.
  - `BOSS_PATTERNS` — bullet patterns used by bosses.
- **Adaptive quality** — `trackPerf()` samples frame time over 45 frames and
  steps `quality` (0 full / 1 medium / 2 low) to reduce glow, shadow blur, and
  particle counts when frames are slow.
- **Audio** — lazily created `AudioContext` (`ensureAudio`) producing
  synthesized `beep` and `noiseBurst` sounds; no audio files. Muted state is
  persisted.
- **Game state & logic** — `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`,
  `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon` and the weapon family
  (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`,
  `fireHomingSeeded`, `fireDoubleRing`), `useBomb`, `loseLife`, `killBoss`,
  `addScore` / `breakChain` (combo system), `endGame`, `togglePause`.
- **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`. Drawn on a
  canvas at a logical size of `BASE_W × BASE_H = 480 × 720`, scaled by `resize()`
  to fit the window.
- **Input** — keyboard, mouse, and touch. Touch uses an on-screen joystick
  (`joystickStart/Move/End`) and fire/bomb buttons. `canvasPointFromEvent`
  converts pointer coordinates to logical canvas space.
- **Main loop** — `loop(now)` via `requestAnimationFrame`, with `dt` clamped to
  50 ms. Update and draw are skipped while paused or not running.

### Controls

| Action | Keyboard | Touch / mouse |
| --- | --- | --- |
| Move | Arrow keys / WASD | Joystick |
| Fire | Hold `Space` (or hold the mouse on the fire button) | Hold fire button |
| Bomb | `X` or either `Shift` | Bomb button |
| Pause | `P` or `Escape` | Pause button (`#pauseBtn`) |

## Conventions when editing

- **Keep it a single file.** The project is deliberately dependency-free; do not
  introduce a bundler, framework, or npm dependencies unless the user asks.
- **Preserve the IIFE and its `'use strict'` directive.** Add new state as
  `let`/`const` inside the IIFE rather than on `window`.
- **Tune gameplay through the config tables** (`DIFFICULTIES`, `ENEMY_TYPES`,
  `BOSSES`, `THEMES`, `KILLS_PER_STAGE`) rather than hard-coding magic numbers
  into update logic.
- **Respect the quality tiers.** New visual effects that add glow, shadows, or
  many particles should be gated on `quality` the way existing code is.
- **Respect reduced motion.** `reducedMotion` gates screen shake and similar
  effects; new motion-heavy effects should check it too.
- **Persist settings with the existing `STORAGE_*` keys** and wrap any new
  storage access in the same pattern, since `localStorage` can throw in some
  browser privacy modes.
- **Keep touch working.** Any new control must be reachable on touch devices
  (`isTouchDevice`) as well as by keyboard.
- **Accessibility text** goes through `announce(text)` so screen readers get
  updates via the live region.

## Verifying changes

There are no automated tests. After editing `index.html`:

1. Open it in a browser (desktop and, if the change touches input or layout, a
   narrow/mobile viewport).
2. Play through at least one stage transition and one boss fight, since those
   exercise most of the state machine.
3. Check the browser console for errors.
4. Confirm pause/resume, bomb use, and game-over → restart still work.
5. If you changed persistence, reload the page and confirm best score, mute,
   difficulty, and reduced-motion settings survive.

A quick syntax sanity check for the inline script is possible with Node by
extracting the `<script>` block into a scratch file and running `node --check` on it.

## Git

- Development branch for this work: `claude/gifted-tesla-elu9uw`.
- Commit messages should be short and descriptive, in the imperative mood
  (existing history: "Add Sky Fighter: Raiden-style vertical shoot-em-up game").

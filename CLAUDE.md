# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a
single-page HTML5 canvas game. The whole game (markup, CSS, and JavaScript) lives in
`index.html`. `README.md` is a placeholder.

There is no build step, bundler, package manager, test suite, or linter. To run the
game, open `index.html` in a browser, or serve the directory with any static server.

## Repository layout

- `index.html` — the entire game:
  - `<style>` block (lines 10–359): HUD pills, hearts/bombs, glassmorphic
    overlay card, boss health bar, touch controls, and reduced-motion overrides.
  - Markup (lines 361–421): `#gameCanvas`, the HUD, overlay, pause/mute/motion
    buttons, touch joystick and FIRE/BOMB buttons, and a hidden `#srLive` region for
    screen-reader announcements.
  - One inline `<script>` (IIFE, `'use strict'`, lines 422–1667) with all game logic.
- `README.md` — placeholder, no setup instructions.
- `.git` — history. Default branch is `main`; development happens on feature branches.

## Architecture of the game script

The script is one IIFE. Its major parts, in file order:

1. **DOM refs and constants** — cached element lookups, `STORAGE_*` localStorage keys,
   `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES` (6 stage palettes), `ENEMY_TYPES`
   (`grunt`, `interceptor`, `tank`), and `BOSSES` (6 bosses, each with a `hpMul`).
2. **Persistence** — best score, mute, difficulty, and reduced-motion preference are stored
   in `localStorage` under the `skyfighter.*` keys. Every read/write is a plain
   `localStorage` call with no try/catch, so keep that in mind if you change storage.
3. **Adaptive quality** — `trackPerf` averages the last 45 frame times and steps `quality`
   (0 full, 1 medium, 2 low) up or down to hold roughly 40–56 fps.
4. **Audio** — `ensureAudio` lazily creates the `AudioContext` (must be triggered by a user
   gesture). Sound effects are synthesized with `beep` and `noiseBurst`; `panFor` pans by x.
5. **Canvas sizing** — the logical resolution is fixed at 480×720 (`BASE_W`/`BASE_H`);
   `resize` scales the CSS size to fit the window, capped at 1.6×.
6. **Input** — keyboard (`keys` map keyed by `e.code`), mouse/pointer via
   `canvasPointFromEvent`, and touch via the joystick (`joystickStart/Move/End`) and
   FIRE/BOMB buttons.
7. **Game state and loop** — `resetGame`, `update(dt)`, `draw()`, and the `requestAnimationFrame`
   loop. `dt` is clamped to 0.05 s. Update and draw run only when `running && !paused`.
8. **Combat** — `spawnEnemy`, `spawnBoss`, `fireWeapon` (Vulcan/Laser, 3 power levels),
   `useBomb` (screen clear, damages bosses), `maybeSpawnPowerUp`, `applyPowerUp`, and the
   kill-chain multiplier (`addScore`, `breakChain`).
9. **Boss patterns** — `BOSS_PATTERNS` is an array of bullet-pattern helpers
   (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`,
   `fireHomingSeeded`, `fireDoubleRing`), driven by `updateBoss` and `killBoss`.
10. **Lifecycle** — `loseLife`, `endGame`, `togglePause`, and the `startBtn` click handler.

## Conventions

- **Keep it one file.** The project is deliberately a single self-contained HTML file.
  Do not introduce a build step, npm dependencies, or separate asset files unless the
  user asks for that change.
- **Style.** Use 2-space indentation, single quotes in JS, and `const`/`let` (no `var`).
  Keep the existing IIFE and `'use strict'` wrapper. Don't add globals.
- **Constants over magic numbers.** Tunables belong in `DIFFICULTIES`, `ENEMY_TYPES`,
  `BOSSES`, or `THEMES`, so balance changes stay in one place.
- **Entities are plain objects in arrays** that get filtered out when they leave the screen
  or die. Follow the same pattern for new entities.
- **Rendering.** Draw everything onto `ctx` in logical 480×720 coordinates. Don't use CSS
  transforms to position game objects.
- **Accessibility.** Respect `reducedMotion` (screen shake, flashes, and similar effects
  should be gated on it). Announce important state changes through `announce()`.
- **Touch and desktop parity.** Any new control needs both a keyboard/mouse path and a
  touch path, and the on-screen `#controlsHint` text should reflect it.

## Testing and verification

There is no automated test setup. Verify changes by:

1. Loading `index.html` in a browser (or a headless Chromium) and checking the browser console
   for errors.
2. Playing through at least one stage transition and one boss fight after gameplay changes.
3. Checking touch controls with a touch-emulated viewport after input changes.
4. Checking reduced-motion behaviour after visual-effect changes.

Syntax-check the inline script if convenient, e.g. extract the `<script>` body and run
`node --check` on it.

## Git workflow

- Develop on the branch assigned for the task. Don't push to `main` directly.
- Write descriptive commit messages in the imperative mood, summarising the gameplay or
  code change first and listing details in the body.
- Keep commits focused. Don't commit unrelated formatting changes to `index.html`, since
  the file is large and diffs are hard to review.

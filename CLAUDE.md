# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up written as a single-page HTML5 canvas game.

Files:

- `index.html` (~1,670 lines): the entire game. CSS is in a `<style>` block (roughly lines 10–420), markup follows, and all game logic is in one IIFE `<script>` starting around line 422.
- `README.md`: a one-line placeholder. It is not a source of project documentation.

There is no build step, no package manager, no bundler, no framework, and no test suite. Do not add one unless the user asks.

## Running the project

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). No install is needed.

## Architecture of `index.html`

Everything runs inside one IIFE. Top-level state and helpers live in closure variables, not on `window`.

- **Setup and constants**: DOM lookups (`canvas`, `overlay`, HUD elements, touch controls), `STORAGE_*` keys, `DIFFICULTIES`, `THEMES` (one per stage, cycled), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), and `BOSSES` (six entries, one per boss fight).
- **Performance scaling**: `quality` (0 full, 1 medium, 2 low) is adjusted by `trackPerf()` from a rolling average of frame times.
- **Persistence**: `localStorage` keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Reads and writes are direct and not wrapped in try/catch, so keep that in mind if you change them.
- **Audio**: a lazily created Web Audio context (`ensureAudio`) with `beep()` and `noiseBurst()` helpers. Audio starts only after a user gesture.
- **Input**: keyboard (`keys[e.code]`), mouse, and touch. Touch uses a left-side joystick plus FIRE and BOMB buttons.
- **Game logic**: `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `update(dt)`, `updateBoss(dt)`, `endGame`, and `togglePause`.
- **Boss patterns**: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, and `fireDoubleRing`.
- **Rendering**: `draw()`, `drawPlane()`, `drawBossShip()`, `drawClouds()`, and `spawnExplosion()`. Drawing uses the 2D canvas context only.
- **Main loop**: `loop(now)` drives `update` and `draw` with a delta time.

### Gameplay reference

- Difficulty (`easy`, `normal`, `hard`) changes lives, starting bombs, spawn rates, enemy fire chance, and boss HP multiplier.
- Each stage lasts `KILLS_PER_STAGE` (18) kills, then a boss fight begins.
- Power-ups: `life`, `bomb`, and `weapon`. Weapon has three power levels.
- Controls: arrows or WASD to move, Space to fire, X or Shift to bomb, P or Esc to pause, mouse to steer and click to fire.

## Conventions

- Keep the game self-contained in `index.html`. Do not split out JS or CSS files, and do not add external CDN dependencies, unless the user asks for it.
- Match the existing style: 2-space indentation, `const` and `let`, `function` declarations inside the IIFE, and short comment banners such as `// ---------- Input ----------`.
- Game tuning values are inline constants in the tables near the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`). Change those tables instead of hard-coding numbers in logic.
- Preserve mobile behaviour when you change input or layout: `touch-action: none`, safe-area insets, and the `resize()` scaling of the fixed-size canvas.
- Preserve accessibility hooks: the `srLive` live region (`announce()`) and reduced-motion handling.

## Verifying changes

There are no automated tests or linters configured. To verify a change:

1. Open `index.html` in a browser, or use the pre-installed Playwright Chromium to load it headless.
2. Check the browser console for errors.
3. Exercise the change: start a game, fire, use a bomb, pause, reach a boss if the change touches boss logic, and check the HUD.

Say explicitly in your summary when you could only verify by reading the code.

## Git workflow

- Develop on the branch assigned for the session. Do not push to other branches without explicit permission.
- Write clear, descriptive commit messages that explain the why as well as the what.
- Do not open a pull request unless the user explicitly asks for one.

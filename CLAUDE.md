# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only app is **Sky Fighter**, a Raiden-style vertical shoot-'em-up written as a single self-contained HTML5 canvas page:

- `index.html` (~58 KB): all markup, CSS (`<style>`, starts ~line 10) and JavaScript (`<script>`, starts ~line 422) in one file.
- `README.md`: one-line title only.

There is no build system, package manager, test suite, linter, or backend. Nothing needs to be installed.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and browse to it.
- There are no automated tests. Verify changes by playing the game in a browser: start a game, check movement, firing, bombs, pause, stage transitions, boss fights, and game over. Check mobile layout with touch controls when touching HUD, overlay, or touch-control code.
- Keep the page working as a single file. Do not split it into modules or add a bundler unless the user asks.

## Code layout inside `index.html`

The game lives in one IIFE inside the `<script>` block. Key pieces, roughly in file order:

- **DOM references** (`const xxxEl = document.getElementById(...)`): every HUD, overlay, and touch element is looked up once at startup. Element IDs in the markup are load-bearing; renaming one requires updating the script.
- **Storage keys** (`STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`): all `localStorage` keys are prefixed `skyfighter.`. Keep that prefix for new keys and wrap access in try/catch if you add new storage use.
- **Tuning tables**:
  - `DIFFICULTIES`: easy/normal/hard values for spawn rate, enemy speed, shot chance, lives, starting bombs, and boss HP multiplier.
  - `KILLS_PER_STAGE`: kills needed to advance a stage.
  - `THEMES`: per-stage sky gradient, cloud and sun colors, and theme names.
  - `ENEMY_TYPES`: grunt, interceptor (weaving), tank (4 HP, from stage 2).
  - `BOSSES`: six boss designs with colors and `hpMul`.
- **Audio**: Web Audio API, created lazily by `ensureAudio()` (browsers require a user gesture first). Sound helpers are `beep()` and `noiseBurst()`. Respect the `muted` flag.
- **Performance scaling**: `trackPerf()` averages frame times and sets `quality` (0 full, 1 medium, 2 low). Effects should honor this value.
- **Accessibility**: a reduced-motion setting (`reducedMotion`, toggled by `motionBtn`) disables screen shake, flashes, and pulses. `announce()` writes to the `srLive` live region. Preserve both when adding effects.
- **Weapons and combat**: `fireWeapon()` (Vulcan/Laser, 3 power levels), `spawnEnemy()`, `spawnBoss()`, `spawnExplosion()`.
- **Boss attack patterns**: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, driven by `updateBoss(dt)`.
- **Main loop**: `update(dt)` (game logic) → `draw()` (rendering) → `loop(now)` scheduled with `requestAnimationFrame`. Time-based movement uses `dt`; keep it that way.
- **Input**: keyboard (`keys[e.code]`; arrows/WASD move, Space fire, X or Shift bomb, P or Esc pause), mouse (steer and click to fire), and touch (`moveZone` joystick, `fireBtn`, `bombBtn`).

## Conventions

- Keep the existing style: 2-space indentation, single quotes in JS, `const`/`let`, no frameworks, no external dependencies, and no CDN links. The page must work offline.
- Keep all CSS and JS inline in `index.html`. Inline SVG data URIs (such as the favicon) are fine.
- Prefer small, targeted edits. Do not reformat unrelated code.
- Use `dt`-based timing and the existing `quality` and `reducedMotion` flags for new visual effects.
- Keep the HUD and overlay responsive. The layout uses `clamp()` and `env(safe-area-inset-*)`; follow that pattern for new UI.
- Use plain-English comments for non-obvious game logic only.

## Git workflow

- Develop on the branch the session specifies (currently `claude/gifted-tesla-h3o2cu`). Do not push to `main` directly.
- Commit with clear, descriptive messages that explain the gameplay or UI change.
- Do not open a pull request unless the user explicitly asks for one.

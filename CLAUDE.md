# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, single-page browser game called **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game lives in one file, `index.html` (inline CSS, inline JavaScript, HTML5 `<canvas>`). There is no build step, no package manager, no test suite, and no external dependencies.

Other files:
- `README.md` — a one-line title only.
- `index.html` — the whole game (~1,670 lines).

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, e.g. `python3 -m http.server` and visit `http://localhost:8000`. No install step is needed.

## Code structure (`index.html`)

- **`<style>`** — dark color scheme, full-viewport layout, pill-style HUD, glassmorphic overlay card, touch joystick/fire controls.
- **`<body>`** — `#container` wrapping `#gameCanvas`, the HUD, overlays, and touch controls.
- **`<script>`** — all game logic, roughly in this order:
  - Constants: `BASE_W = 480`, `BASE_H = 720` (logical canvas size, scaled to fit the viewport), `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE = 18`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, `EXPLODE_SFX`.
  - Persistence: best score, mute, difficulty, and reduced-motion preference are stored in `localStorage` under `skyfighter.*` keys.
  - Audio: synthesized sound effects (`beep`, `noiseBurst`, `ensureAudio`) — no audio files.
  - Input: keyboard, mouse, and touch handlers; `keys[]` state; virtual joystick (`joystickStart/Move/End`).
  - Game systems: `update()` (simulation), `draw()` / `drawPlane` / `drawBossShip` / `drawClouds` (rendering), weapon patterns (`fireWeapon`, `fireRadial`, `fireSpiralBurst`, `fireHomingSeeded`, etc.), power-ups (`maybeSpawnPowerUp`, `applyPowerUp`), bombs (`useBomb`, `flashBomb`), bosses (`spawnBoss`, `updateBoss`, `killBoss`), combo/chain scoring (`breakChain`, `addScore`, `showComboPop`), stage flow (`showStageBanner`, `resetGame`, `endGame`), and the main `loop()`.

## Game rules and controls (keep docs in sync when changing these)

- **Keyboard:** arrow keys / WASD to move (via `keys[]`), Space to fire, `X` or Shift to use a smart bomb, `P` or Escape to pause.
- **Touch:** virtual joystick on the left, fire button; enabled automatically on touch devices.
- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. They set spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- **Stages:** each stage requires `KILLS_PER_STAGE` (18) kills, then a boss fight with a health bar and multiple attack patterns.
- **Weapons:** Vulcan / Laser with 3 power levels, gathered from power-ups.
- **Scoring:** kill-chain multiplier; best score persisted to `localStorage`.

## Conventions

- Keep the game **self-contained in `index.html`** unless the user asks for a split. Do not introduce a bundler, framework, or npm dependency without an explicit request.
- Preserve the existing style: plain modern JavaScript, `const`/`let`, small named functions, constants in ALL_CAPS. Match the surrounding code's semicolon usage, which is mixed.
- Tuning values (spawn rates, speeds, HP, scoring) live in the constants and `DIFFICULTIES` objects near the top of the script. Prefer changing those over hard-coding new numbers in logic.
- All persisted state must use the `skyfighter.*` key prefix and be read/written in a way that tolerates missing or invalid values.
- Keep the layout working on phone-sized viewports (`viewport-fit=cover`, safe-area insets, touch controls) as well as desktop.
- Respect the reduced-motion preference (`STORAGE_REDUCED`) when adding screen shake, flashes, or other motion effects.

## Verifying changes

There are no automated tests or linters. To verify a change:
1. Load `index.html` in a browser (or via a static server) and check the browser console for errors.
2. Play through at least one full stage and a boss fight on the affected difficulty.
3. Check pause (`P`/Escape), bomb (`X`/Shift), mute, and difficulty selection still work.
4. For layout changes, check both a desktop-sized and a phone-sized viewport.

## Git

- Development branch for this work: `claude/gifted-tesla-jex5s3`. Commit and push there; do not push to other branches without permission.
- Do not open a pull request unless explicitly asked.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game lives in a single self-contained file, `index.html` (~1,670 lines: inline `<style>`, inline HTML/markup, and one inline `<script>`). There is no build step, no package manager, no test suite, and no external assets. `README.md` is a one-line title.

Git: remote `origin` is `https://github.com/ubban/Google_4`. Development branch for this session: `claude/gifted-tesla-36vuyc`.

## Running the game

- Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`) and visit it.
- Nothing needs to be installed. Verify changes by loading the page in a browser and playing; there are no automated tests or linters configured.

## Architecture of `index.html`

Everything runs in one `<script>` block, organized as plain top-level functions and `let`/`const` state on a single canvas-driven game loop.

- **Layout / DOM**: `#container` holds `<canvas id="gameCanvas">`, a HUD (`#hud`, `#bossBarWrap`, `#topBar`), transient overlays (`#comboPop`, `#stageBanner`, `#flash`, `#vignette`), touch controls (`#touchControls` with `#moveZone`, `#joystickBase`, `#actionZone`), and a start/pause/game-over `#overlay` / `#overlayCard`.
- **Canvas sizing**: `BASE_W = 480`, `BASE_H = 720`; `resize()` scales the canvas to fit the viewport.
- **Config constants** (near the top of the script):
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rate, enemy speed, enemy shoot chance, starting lives, bombs, boss HP multiplier.
  - `KILLS_PER_STAGE = 18`: kills needed to advance a stage.
  - `THEMES`: six sky palettes cycled per stage.
  - `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`) and `BOSSES` (six bosses, each with its own colors and `hpMul`).
- **Persistence**: `localStorage` keys prefixed `skyfighter.` — `bestScore`, `muted`, `difficulty`, `reducedMotion`. Every access should keep working if storage is unavailable.
- **Audio**: Web Audio API, lazily created by `ensureAudio()`. Sound effects come from `beep()` and `noiseBurst()`; `EXPLODE_SFX` maps enemy types to explosion sounds. Respect the `muted` flag.
- **Input**:
  - Keyboard: arrows / WASD move, Space fires (also hold-to-fire), `X` / Shift fires a smart bomb (`useBomb()`), `P` / Esc pauses.
  - Touch: a virtual joystick (`joystickStart/Move/End`) and a fire button.
  - Mouse: pointer-follow via `canvasPointFromEvent()` and `mouseActive`.
  - Listeners are wired near lines 760–840. Keep `preventDefault` on touch handlers that need it (`{ passive: false }`).
- **Game loop**: `update(dt)` (around line 1148) handles movement, firing, spawning, collisions, and scoring; drawing helpers (`drawPlane`, `drawBossShip`, etc.) run each frame; `loop(now)` (line ~1640) drives `requestAnimationFrame`. `trackPerf(dt)` adjusts a `quality` level (full / medium / low) when frame times slip.
- **Spawning and combat**: `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`, `fireWeapon()`, `applyPowerUp()`, and a family of boss pattern generators (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`). `chain` / `multiplier` drive the combo system.
- **Accessibility**: `announce()` writes to the `#srLive` live region. `reducedMotion` defaults to the OS `prefers-reduced-motion` setting, can be toggled in-game, and is reflected as a `reduced-motion` class on `<body>`; new visual effects should respect it.

## Conventions

- Keep the game in a single file unless the user asks to split it. Do not introduce a bundler, framework, or package manager without being asked.
- Stay with vanilla JavaScript and CSS that already exist in the file. Match the existing style: two-space indentation, `const`/`let`, arrow functions for event handlers, short helper functions.
- Do not add external CDN scripts or fonts; the game is meant to work offline from a single file.
- Keep mobile support working: layout must fit phone widths, touch controls must stay usable, and `touch-action: none` / `user-select: none` on the body should be preserved.
- When changing gameplay numbers, adjust the `DIFFICULTIES`, `ENEMY_TYPES`, or `BOSSES` tables rather than hard-coding values inside the loop.
- Preserve the `skyfighter.*` storage key names so saved best scores and settings survive updates.

## Testing and verification

There is no automated test setup. For any change:

1. Open `index.html` in a browser (desktop and, if layout or input changed, a narrow mobile viewport).
2. Play through start, pause/resume, a bomb, a stage transition, a boss fight, and game over.
3. Check the browser console for errors, and confirm the best score, mute, difficulty, and reduced-motion settings persist across reloads.

## Git workflow

- Develop on the branch named for the session (currently `claude/gifted-tesla-36vuyc`). Do not push to other branches without explicit permission.
- Make small, focused commits with descriptive messages.
- Do not open a pull request unless the user explicitly asks for one.

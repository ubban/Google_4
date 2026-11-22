# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game (HTML, CSS, and JavaScript) lives in a single file, `index.html` (~1,670 lines). `README.md` is a stub with only the repo name.

There is no package manager, build step, bundler, test suite, linter, or CI configuration. Do not invent one unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser. Check at least: keyboard controls, touch controls (joystick and fire/bomb buttons), pause and mute, difficulty selection, and the game-over and best-score flow.
- For UI or layout changes, check both desktop and phone-sized viewports. The HUD and overlays are sized with `clamp()` and `vw`, and the canvas scales with `resize()`.

## Code layout (`index.html`)

- **`<style>` block:** all styling. The page is a fixed, non-scrolling full-screen layout (`overflow: hidden`, `touch-action: none`). Keep these properties in mind when adding elements.
- **`<body>` markup:** the `#container` holds `#gameCanvas` (the play field) plus HUD and overlay elements (`#hud`, `#overlay`, `#vignette`, `#flash`, `#touchControls`, `#bossBarWrap`, and others). Elements are looked up by ID at the top of the script.
- **Main `<script>`** (starts at line 422) wraps everything in one `(function () { 'use strict'; ... })();` block. Inside, the parts appear in this order:
  - DOM references and `STORAGE_*` keys.
  - Perf tracking (`trackPerf`) and a `quality` level (0 full, 1 medium, 2 low) that adapts to frame times.
  - Configuration constants: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (one per stage, cycling), `ENEMY_TYPES`, and `BOSSES`.
  - Audio: a lazily created `AudioContext` (`ensureAudio`) with synthesized `beep` and `noiseBurst` helpers and a `sfx` table. The context is created on first user input and resumed if suspended.
  - Input: keyboard (`keys`), a virtual joystick and buttons for touch, and mouse aiming on the canvas.
  - Entities and simulation: `spawnEnemy`, `spawnBoss`, `spawnExplosion`, `useBomb`, `fireWeapon`, the main `update(dt)`, and `updateBoss(dt)`. Boss attacks are pattern functions such as `fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, and `fireDoubleRing`.
  - Rendering: `draw()`, `drawPlane`, `drawBossShip`, and `drawClouds`.
  - Main loop: `loop(now)` drives `update` and `draw` each frame.

## Game design constants

Tune values in these tables rather than hard-coding them elsewhere:

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), and `hard`. Each sets spawn rates, enemy speed, enemy shoot chance, starting `lives`, starting `bombs`, and `bossHpMul`.
- **Stages:** every `KILLS_PER_STAGE` (18) kills advance the stage. Stage themes cycle through `THEMES`. Bosses come from `BOSSES`, each with its own `hpMul` and attack behavior.
- **Enemies** (`ENEMY_TYPES`): `grunt`, `interceptor` (weaving), and `tank` (4 HP, unlocks at stage 2). `minStage` controls when each type appears.

## Persistence

The game uses `localStorage` with these keys. Keep the names stable, because they hold players' saved state:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Reduced motion defaults to the OS `prefers-reduced-motion` setting until the user changes it. Respect this preference when adding visual effects such as screen shake, flashes, or vignettes.

## Accessibility

- Keep the `srLive` live region in use. The `announce()` helper feeds it for screen readers.
- Respect reduced-motion mode for new effects.
- Keep every control reachable by keyboard, mouse, and touch. Icon buttons are real `<button>` elements, but they currently rely on `title` attributes rather than `aria-label`. Add `aria-label` to new controls.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. Do not add a framework, bundler, or external dependencies without asking.
- Match the existing style: 2-space indentation, single quotes in JavaScript, `const` for constants, and `UPPER_SNAKE_CASE` for configuration tables.
- The `beep`/`noiseBurst` helpers and `sfx` table are the sanctioned way to add sounds. Do not load audio files.
- Keep the canvas logical size at `BASE_W` x `BASE_H` (480 x 720). Scale through `resize()` rather than changing the base size.
- Commit messages should be short and imperative, for example `Add Sky Fighter: ...`.

## Git

- Develop on the branch assigned for this session (currently `claude/gifted-tesla-maszz9`). Push only to that branch unless the user says otherwise.
- Do not open a pull request unless the user asks for one.

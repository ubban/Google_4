# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up that runs in a browser canvas.

- `index.html` — the entire game: markup, CSS, and JavaScript in one file (~1,670 lines).
- `README.md` — minimal placeholder; no setup instructions yet.

There is no build step, package manager, test suite, or external dependency. The page
loads no remote scripts, stylesheets, or fonts; the favicon is an inline SVG data URI.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server
(e.g. `python3 -m http.server`) and visit it. No install is required.

## File layout of `index.html`

1. `<head>` — viewport meta (`user-scalable=no`, `viewport-fit=cover`), theme color, title,
   inline favicon.
2. `<style>` (about lines 10–362) — all CSS, including HUD, touch controls, and layout.
3. `<body>` markup (from about line 363) — `<canvas id="gameCanvas">`, HUD elements,
   touch joystick/fire/bomb controls, and difficulty/mute/motion buttons.
4. One `<script>` block (from about line 422), wrapped in an IIFE, containing:
   - **Config constants**: `BASE_W = 480`, `BASE_H = 720` (fixed logical resolution, scaled
     by `resize()`), `KILLS_PER_STAGE = 18`, `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`,
     `BOSSES`, `BOSS_PATTERNS`.
   - **Persistence**: `localStorage` keys under the `skyfighter.*` prefix
     (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Reads are not currently
     wrapped in try/catch; `reducedMotion` falls back to `prefers-reduced-motion` when unset.
   - **Audio**: a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`). Audio is created
     lazily on first user gesture.
   - **Input**: keyboard (`keys[e.code]`), touch joystick, on-screen fire/bomb buttons,
     and mouse on the canvas.
   - **Game state and logic**: `resetGame`, `update(dt)`, `updateBoss(dt)`, spawning
     (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), weapons (`fireWeapon`, `useBomb`),
     scoring/combo (`addScore`, `breakChain`), and power-ups (`applyPowerUp`).
   - **Rendering**: `draw()` plus helpers such as `drawPlane`, `drawBossShip`.
   - **Main loop**: `loop(now)` drives `update` and `draw` through `requestAnimationFrame`,
     with adaptive quality (`trackPerf` samples 45 frames and steps `quality` 0–2 up or
     down based on average frame time).

## Conventions to follow

- **Keep it a single file.** Unless the user asks otherwise, put new code in `index.html`
  rather than introducing separate script/style files or a build tool.
- **Stay dependency-free.** Do not add npm packages, CDN scripts, or frameworks without
  an explicit request.
- **Preserve the IIFE.** Game code lives inside the existing `(function () { ... })()` to
  avoid polluting globals.
- **Use the existing constants and tables.** Add new enemies, bosses, themes, or
  difficulties by extending `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, `THEMES`, and
  `DIFFICULTIES` rather than hard-coding values in logic.
- **Use `dt`-based movement.** Update logic takes a delta time; keep speeds and timers
  frame-rate independent.
- **Respect reduced motion and mute.** Visual effects such as `screenShake` should honor
  the reduced-motion setting, and any new sound should go through the mute check.
- **Persist settings only via the `skyfighter.*` keys.** New storage reads and writes
  should be wrapped in try/catch, since `localStorage` can be blocked or unavailable.
- **Keep the layout touch-friendly.** Controls must work on phone-sized viewports with
  no horizontal scroll.

## Verifying changes

There is no automated test suite. For changes to `index.html`:

1. Load the page in a browser and confirm there are no console errors.
2. Play through at least one stage: move, fire, use a bomb, collect a power-up, and
   reach a boss.
3. Check the touch controls and a narrow (phone-width) viewport if the change touches
   layout or input.
4. Toggle mute, difficulty, and reduced motion, and confirm they persist across reloads.

## Git workflow

- Develop on the branch designated for the session (currently `claude/gifted-tesla-bvziq1`).
  Do not push to other branches without explicit permission.
- Do not create pull requests unless the user explicitly asks.
- Commit messages should be short, imperative summaries describing the behavior change.

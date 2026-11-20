# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is a single self-contained file, `index.html`, with inline CSS and JavaScript. There is no build step, no bundler, no package manager, and no test suite.

Repository contents:

- `index.html` — the entire game (markup, styles, and a single IIFE `<script>` at the end of the file).
- `README.md` — minimal project readme.
- `CLAUDE.md` — this file.

## Running the game

No install or build is needed. Open `index.html` directly in a browser, or serve the folder with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Code layout inside `index.html`

- **`<style>`** (top of file): all CSS, including HUD, overlays, boss bar, stage banner, touch controls, and reduced-motion overrides (`body.reduced-motion`).
- **`<body>`**: HUD elements (score, best, multiplier, weapon, hearts, bombs, boss bar, stage banner), the start/pause/game-over overlay, and touch controls (`#touchControls`, `#joystickBase`, `#fireBtn`, `#bombBtn`).
- **`<script>`**: one immediately-invoked function. Key sections, in file order:
  - DOM references and `STORAGE_*` keys (`localStorage` keys are prefixed `skyfighter.`).
  - Perf tracking (`trackPerf`, `quality` 0/1/2) that lowers effects quality when frame times are slow.
  - Config tables: `DIFFICULTIES`, `THEMES` (one per stage, cycled), `ENEMY_TYPES`, `BOSSES`. `KILLS_PER_STAGE` controls stage progression.
  - Audio: lazily created `AudioContext` via `ensureAudio()`, with `beep`, `noiseBurst`, and the `sfx` table. Respects the mute setting.
  - Input: keyboard (`keys` map keyed by `e.code`), mouse (steer and click-to-fire on the canvas), and touch (virtual joystick, fire and bomb buttons).
  - Gameplay: `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `update(dt)`, `updateBoss(dt)`, and boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
  - Rendering: `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`, and `spawnExplosion`.
  - Main loop: `loop(now)` driven by `requestAnimationFrame`, with a canvas sized by `resize()` to a base resolution of 480×720 (scaled to fit the window, capped at 1.6×).

## Conventions

- **Keep it a single file.** Add new CSS, markup, and JS to `index.html` unless the user explicitly asks for a split into multiple files. Do not introduce a build system, framework, or npm dependencies without asking.
- **Vanilla JS only.** Use the existing style: ES2015+ syntax, `const`/`let`, no transpilation, and no external CDN scripts.
- **Tune gameplay through the config tables.** Difficulty numbers, enemy stats, theme colors, and boss stats belong in `DIFFICULTIES`, `ENEMY_TYPES`, `THEMES`, and `BOSSES` rather than being hardcoded in logic.
- **Preserve accessibility and mobile support.** The game has a reduced-motion mode, an ARIA live region (`#srLive`, used by `announce()`), safe-area insets, and touch controls. Changes to input or HUD should keep these working.
- **Persisted settings use `localStorage`.** Keys are `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, and `skyfighter.reducedMotion`. Wrap new storage access in care for private-mode or blocked-storage errors, and keep the existing keys stable so saved data still loads.
- **Keep the visual style consistent.** Match existing colors, fonts (`Segoe UI`, Roboto, Arial), and the HUD look when adding UI.

## Verifying changes

There are no automated tests or linters configured. Verify changes by:

1. Opening `index.html` in a browser (desktop and, if relevant, a narrow mobile viewport).
2. Playing through the affected path: starting a game, firing, using a bomb, reaching a stage transition, fighting a boss, and reaching game over.
3. Checking the browser console for errors.
4. For input changes, testing keyboard, mouse, and touch where the change touches them.
5. Checking that `localStorage` persistence (best score, mute, difficulty, reduced motion) still works after a reload.

Use the `run` skill or a headless browser for repeatable checks when helpful.

## Git workflow

- Develop on the branch assigned for this work (currently `claude/gifted-tesla-icgwog`). Do not push to other branches without explicit permission.
- Commit only when asked, or when the task calls for it. Use clear, imperative commit messages that describe the gameplay or code change.
- Do not create pull requests unless explicitly requested.
- Do not commit secrets or local settings files.

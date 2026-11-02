# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small, dependency-free static web project. Its only product is **Sky Fighter**, a Raiden-style vertical shoot-em-up rendered on an HTML5 `<canvas>`.

| Path | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS (`<style>`, ~lines 10–359), and JavaScript (`<script>`, ~lines 422–1667). |
| `README.md` | Placeholder title only. |

There is no build step, package manager, test suite, linter, or CI configuration. Do not add tooling unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). No install step is needed.

## Architecture of `index.html`

Everything lives in one IIFE (`(function () { 'use strict'; ... })();`). Nothing is exported to `window`, so keep new code inside that closure.

Sections of the script, in order:

1. **DOM refs**: `const xxxEl = document.getElementById(...)`. Every HUD/overlay element is fetched once at startup. If you add UI, add its element ID to the markup and fetch it here.
2. **Storage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. These are the only persisted state.
3. **Adaptive performance** (`trackPerf`): averages the last 45 frame times and steps a `quality` level (0 full / 1 medium / 2 low) up or down. Drawing code should respect `quality` when adding costly effects.
4. **Tuning tables**:
   - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rate, enemy speed, enemy fire chance, starting lives and bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
   - `THEMES`: background palettes, cycled per stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP, unlocked at stage 2).
   - `BOSSES`: six bosses, one per boss fight, each with its own colors and `hpMul`.
5. **Audio**: a Web Audio synth. `ensureAudio()` must run from a user gesture before any sound plays. Use `beep()` and `noiseBurst()` and the `sfx` helpers; do not add audio files.
6. **Input**: keyboard (`keys[e.code]`), mouse (`mouseTarget` / `mouseActive`), touch joystick plus FIRE and BOMB buttons (`joystickStart`/`Move`/`End`).
7. **State and lifecycle**: `resetGame()`, `update(dt)`, `draw()`, `loop(now)`, `togglePause()`, `loseLife()`, `endGame()`. The game state is a set of module-level `let` variables (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `bombCount`, `stage`, `chain`, `multiplier`, etc.).
8. **Bosses**: `BOSS_PATTERNS` is an array of per-boss arrays of attack functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`). `updateBoss(dt)` picks the active pattern; `killBoss()` advances the stage.
9. **Rendering**: `draw()` plus helpers `drawPlane`, `drawBossShip`, `drawClouds`. Screen shake is handled by `screenShake()` and `shakeTime`.

### Key conventions

- **Fixed logical resolution**: the canvas is always `480 × 720` (`BASE_W`, `BASE_H`). CSS scales it to fit the window (`resize()`, capped at 1.6×). Coordinates in game code are in these logical pixels, never CSS pixels.
- **Delta-time movement**: `loop()` clamps `dt` to 0.05 s. All motion must multiply by `dt` so speeds are frame-rate independent.
- **Pure Canvas and DOM**: no frameworks, no libraries, no CDN scripts. Keep it that way.
- **Accessibility hooks**: announcements go through `announce(text)`, which writes to the `#srLive` live region. Respect `reducedMotion` (body class `reduced-motion`) when adding motion effects.
- **Persistence is best-effort**: `localStorage` reads and writes are used directly today. If you add new storage, wrap it in `try/catch`, because private windows and blocked storage can throw.
- **Input parity**: any new action needs a keyboard binding, a touch control, and a line in `#controlsHint` (the in-game help text, ~line 417).

## Editing guidelines

- Prefer small, targeted edits to `index.html`. Do not split it into multiple files or introduce a build step unless the user asks.
- Keep the single-file, no-dependency nature intact.
- When changing balance, edit the tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`) rather than hard-coding numbers in logic.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and short comment banners like `// ---------- Input ----------` to separate sections.
- Do not change the storage key names. Doing so silently resets players' saved best score, mute, difficulty, and motion settings.

## Verifying changes

There are no automated tests. Verify by:

1. Opening `index.html` in a browser (or a static server) and checking the browser console for errors.
2. Playing through the relevant path: start, move, fire, bomb, pause/resume, a stage transition, a boss fight, and game over.
3. For layout or touch changes, checking a narrow (phone-width) viewport and confirming the touch controls still appear and respond.

In a headless environment, Playwright with the pre-installed Chromium can drive the page for a smoke test. Do not install a new Playwright browser.

## Git

- Development branch for this work: `claude/gifted-tesla-3t81gj`.
- Write clear, imperative commit messages that describe the gameplay or code change.
- Do not open a pull request unless the user asks for one.

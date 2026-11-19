# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small, dependency-free static web project containing one game:
**Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on an HTML `<canvas>`.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS (`<style>`), and JavaScript (`<script>`) in a single file (~1,670 lines). |
| `README.md` | One-line project title plus a stray timestamp. Not documentation of the game. |

There is no build step, no package manager, no test suite, and no linter configuration. Do not add
any of these unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server, e.g.
`python3 -m http.server` and visit `http://localhost:8000`.

## Architecture of `index.html`

The file is organized top to bottom as:

1. **`<head>`**: meta tags, a data-URI SVG favicon, and an inline `<style>` block. The page uses
   `color-scheme: dark`, `overflow: hidden`, and `touch-action: none` because it is a fullscreen game.
2. **`<body>` markup**: a `#container` wrapping `#gameCanvas`, the HUD (`#hud`, `#bossBarWrap`,
   `#comboPop`, `#stageBanner`, `#topBar`), touch controls (`#touchControls`: joystick, fire and bomb
   buttons), and `#overlay` (title/pause/game-over card with difficulty buttons).
3. **One inline `<script>`** wrapping everything in an IIFE with `'use strict'`. There are no
   globals and no modules. Keep new code inside this IIFE.

### Main script sections (in order)

- **Setup and DOM refs**: `const xxxEl = document.getElementById(...)` at the top.
- **Persistence keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`,
  `skyfighter.reducedMotion`, all in `localStorage`. Every read is currently unguarded; if you add
  storage access, consider wrapping it in `try/catch` (private browsing can throw).
- **Adaptive quality**: `trackPerf` samples frame times over 45 frames and steps `quality` from 0
  (full) to 2 (low) when the frame budget is missed.
- **Config tables**:
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed, shoot chance, lives,
    bombs, boss HP multiplier.
  - `THEMES`: six sky palettes that cycle per stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaves), `tank` (4 HP, appears from stage 2).
  - `BOSSES`: six bosses (VANGUARD → TITAN) with colors and `hpMul`.
  - `KILLS_PER_STAGE = 18`.
- **Audio**: `ensureAudio()` lazily creates the `AudioContext`. `beep()` and `noiseBurst()` synthesize
  all sound effects; there are no audio files. Audio must start from a user gesture.
- **Input**: keyboard (`keys` map keyed by `e.code`), touch joystick, and fire/bomb buttons. Pointer
  input is also translated to canvas coordinates by `canvasPointFromEvent`.
- **Sizing**: `BASE_W = 480`, `BASE_H = 720` is the logical play field. `resize()` scales it to fit.
- **Game state and logic**: `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`,
  `fireWeapon`, `useBomb`, `applyPowerUp`, `update(dt)`, `loseLife`, `endGame`, `togglePause`.
- **Boss attacks**: `BOSS_PATTERNS` is an array of bullet-pattern functions (`fireRadial`,
  `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`,
  `fireDoubleRing`). `updateBoss` and `killBoss` drive them.
- **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`. Drawing is plain 2D canvas
  primitives; there are no image assets.
- **Main loop**: `loop(now)` clamps `dt` to 50 ms, calls `trackPerf`, then `update` and `draw` only
  while `running && !paused`.

### Controls (for reference when changing input)

- Move: arrow keys or WASD. Fire: Space (hold) or the fire button. Bomb: `X` or either Shift.
- Pause: `P` or `Escape`. Mute, reduced-motion and pause buttons are in `#topBar`.
- Touch: left-side virtual joystick, right-side fire and bomb buttons.

## Conventions

- **Keep it a single file** unless the user asks to split it. Styles, markup and script all live in
  `index.html`, and that is intentional.
- **Vanilla JS only.** No frameworks, no CDN dependencies. Match the existing style: `const`/`let`,
  arrow callbacks, two-space indentation, and section comments in the form `// ---------- Name ----------`.
- **Tune gameplay through the config tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`,
  `BOSS_PATTERNS`) instead of hard-coding magic numbers in update logic.
- **Respect `reducedMotion`** when adding screen shake, flashes, or particle effects. The existing
  effects (`screenShake`, `flashHit`, `flashBomb`) show the pattern.
- **Touch and keyboard parity.** Any new player action needs both a key binding and a touch control,
  and should update the on-screen hints if there are any.
- **Accessibility**: announce important state changes through `announce()`, which writes to the
  `aria-live` region `#srLive`.
- **Mobile first.** The layout must work at phone width without horizontal scrolling. Test with the
  viewport meta and `touch-action` in mind.

## Verifying changes

There are no automated tests. For any gameplay or rendering change:

1. Open `index.html` in a browser (or serve it locally) and play through at least one stage.
2. Check the console for errors. Most failures here surface as uncaught exceptions in `loop`.
3. For UI or layout changes, check both a desktop viewport and a phone-sized viewport.
4. For balance changes, test all three difficulties if practical.

Say explicitly in your summary what you verified and what you could not verify.

## Git and workflow

- Default development branch for this session: `claude/gifted-tesla-08s6w9`. Do not push to other
  branches without explicit permission.
- Commit only when asked. Do not open pull requests unless the user asks for one.
- Keep commits focused. Do not commit local editor or OS files.
- `localStorage` keys are user data on their own machines. Renaming a key silently resets players'
  best scores and settings, so treat key names as a compatibility surface.

## Things to avoid

- Do not split `index.html` into multiple files, add a bundler, or add dependencies without a request.
- Do not remove the `'use strict'` IIFE wrapper or leak variables to `window`.
- Do not replace canvas drawing with image or audio assets unless the user provides them.
- Do not change `BASE_W` / `BASE_H` casually. Collision bounds, spawn positions, and boss patterns
  are tuned to them.

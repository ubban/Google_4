# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small static web project. Its only game is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up that runs in a single HTML file on a 2D
`<canvas>`. There is no build step, no package manager, no test suite, and no
server-side code.

## Repository layout

| Path        | Purpose                                                                 |
|-------------|-------------------------------------------------------------------------|
| `index.html`| The entire game: markup (`<body>`), CSS (`<style>`, ~lines 10–359), and JavaScript (`<script>`, ~lines 422–1667). |
| `README.md` | Placeholder title only.                                                 |
| `CLAUDE.md` | This file.                                                              |

Open `index.html` directly in a browser to play. Nothing needs to be installed.

## Architecture of `index.html`

The script is one IIFE (`(function () { 'use strict'; ... })();`). Everything
lives inside it; no globals are exported.

**Markup (lines ~361–421):** `#container` holds the `#gameCanvas`, HUD
(`#hud`, `#bossBarWrap`, `#comboPop`, `#stageBanner`), the top-bar icon buttons
(`#muteBtn`, `#motionBtn`, `#pauseBtn`), touch controls (`#touchControls`:
joystick zone, FIRE, BOMB), and the `#overlay` start/game-over card with the
difficulty buttons (`.diffBtn[data-diff]`) and `#startBtn`.

**Script sections, in order:**

1. **DOM refs and constants** — element lookups, `STORAGE_*` localStorage keys
   (`skyfighter.bestScore`, `.muted`, `.difficulty`, `.reducedMotion`).
2. **Tuning data** — `DIFFICULTIES` (spawn rate, enemy speed/fire chance, lives,
   bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (six sky palettes
   cycled per stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), `BOSSES`
   (six bosses: VANGUARD → TITAN, with per-boss HP multipliers).
3. **Adaptive quality** — `trackPerf()` samples frame time over 45 frames and
   steps `quality` 0/1/2 (full/medium/low) to reduce effects when frames are slow.
4. **Audio** — `ensureAudio()` lazily creates an `AudioContext` (must be called
   from a user gesture). `beep()` and `noiseBurst()` synthesize all sound
   effects; there are no audio files.
5. **Rendering helpers** — `drawPlane()`, `drawBossShip()`, `drawClouds()`,
   `renderHearts()`, `renderBombs()`, and screen effects (`screenShake`,
   `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`).
6. **Input** — `keys` map keyed by `KeyboardEvent.code`; joystick and touch
   handlers; mouse/touch position mapped to canvas coordinates via
   `canvasPointFromEvent()`.
7. **Game state** — `resetGame()` initializes `player`, `bullets`, `enemies`,
   `enemyBullets`, `particles`, `powerUps`, `boss`, score, lives, and bombs.
8. **Entities and combat** — `spawnEnemy()`, `spawnBoss()`,
   `maybeSpawnPowerUp()` (drops `life`, `bomb`, `weapon`), `fireWeapon()`
   (`vulcan` or `laser`, levels 1–3), `useBomb()`, `addScore()` / `breakChain()`
   for the score multiplier combo.
9. **`update(dt)`** — the per-frame simulation: movement, firing, enemy and
   bullet motion, collisions (`rectsOverlap`), power-up pickup, stage progression.
10. **Boss patterns** — `BOSS_PATTERNS` is an array of three attack functions per
    boss, built from helpers (`fireRadial`, `fireAimedSpread`, `fireColumns`,
    `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
    `updateBoss(dt)` cycles through them; `killBoss()` advances the stage.
11. **Lifecycle** — `loseLife()`, `endGame()` (updates best score in
    localStorage and shows the overlay), `togglePause()`.
12. **`draw()`** — renders the frame; `loop(now)` is the `requestAnimationFrame`
    driver. `dt` is clamped to 50 ms. Update and draw run only when `running && !paused`.

## Controls (as implemented)

- Move: mouse, arrow keys, or WASD; touch uses a left-side drag joystick.
- Fire: Space (held), or hold mouse on the FIRE button, or touch FIRE.
- Bomb: `X` or Shift, or touch BOMB.
- Pause: `P` or `Esc`.
- Mute, reduced-motion toggle, and difficulty are persisted to localStorage.

## Conventions when editing

- **Keep it a single file** unless the user asks for a split. The project is
  deliberately dependency-free; do not introduce a bundler, framework, or CDN
  dependency without asking.
- **Stay inside the IIFE** and keep `'use strict'`. Do not add new globals.
- **Tuning changes belong in the data tables** (`DIFFICULTIES`, `ENEMY_TYPES`,
  `BOSSES`, `THEMES`, `BOSS_PATTERNS`) rather than hard-coded in logic.
- **Game coordinates** are in the fixed `BASE_W × BASE_H` (480 × 720) space and
  scaled by `resize()`. Use canvas-space values when adding entities.
- **Persisted state** uses the `skyfighter.*` localStorage keys. Changing a key
  silently resets that setting for existing players; avoid it unless intended.
- **Accessibility hooks** exist and should be preserved: `aria-live` announcements
  via `announce()`, `sr-only` text, `prefers-reduced-motion` handling, and
  `reduced-motion` body class.
- **Mobile**: `touchControls` is shown only on touch-capable devices. Test
  layout changes at phone width as well as desktop.

## Verifying changes

There are no automated tests or linters configured. To verify a change:

1. Open `index.html` in a browser (or serve the directory with any static server).
2. Start a game on each difficulty, check firing, bombs, pause, power-ups,
   stage transitions (every 18 kills), and each boss.
3. Check the browser console for errors.
4. For layout changes, check both desktop and a narrow touch-sized viewport.

If you add a test or lint setup, document the command here.

## Git

- Default branch: `main`. Feature work for this repo has been on `claude/*` branches.
- Commit with clear messages describing the gameplay or code change.

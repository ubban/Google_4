# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small, dependency-free web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up that runs as a single HTML5 canvas page.

| File        | Purpose                                                        |
|-------------|----------------------------------------------------------------|
| `index.html`| The entire game: inline CSS, markup, and one IIFE `<script>`.  |
| `README.md` | Placeholder title only.                                        |

There is no build step, no package manager, no test runner, and no CI config.
To run the game, open `index.html` in a browser (or serve the directory with any
static server).

## Layout of `index.html`

- **`<style>` (~lines 10–360):** dark color scheme (`:root { color-scheme: dark; }`),
  HUD pills, hearts/bombs indicators, vignette/flash overlays, boss bar, touch
  controls, and the start/pause overlay card.
- **`<body>` markup (~lines 362–421):** `#container` holds the `#gameCanvas`, HUD
  (`#hud`), boss bar, combo/stage banners, top bar (mute / reduced-motion / pause),
  an `aria-live` region (`#srLive`), touch controls, and `#overlay` with difficulty
  buttons and the start button.
- **`<script>` (~lines 422–1670):** one IIFE with `'use strict'`. Nothing is exported
  or attached to `window`; keep new code inside the IIFE.

Inside the script, the code is grouped roughly in this order:

1. **DOM references and `STORAGE_*` keys.** Persistence uses `localStorage` under
   the `skyfighter.*` prefix (best score, mute, difficulty, reduced motion).
2. **Adaptive performance** (`trackPerf`): a 45-frame rolling average of frame
   time steps `quality` between 0 (full), 1 (medium), and 2 (low).
3. **Config tables:** `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`,
   `BOSSES`. Most tuning lives here.
4. **Audio:** a lazily created `AudioContext` (`ensureAudio`) routed through a
   dynamics compressor. Helpers `beep`, `noiseBurst`, and `panFor` build the
   layered sounds. Sounds are only created after a user gesture.
5. **HUD and UI helpers:** `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`,
   `showStageBanner`, `screenShake`, `setDifficulty`, `announce` (screen reader text).
6. **Input:** keyboard (`keys` map), touch joystick and buttons, and mouse steering
   on the canvas. Keyboard and touch take priority over mouse steering.
7. **Game state:** `resetGame`, `update(dt)`, spawn functions (`spawnEnemy`,
   `spawnBoss`, `maybeSpawnPowerUp`), weapons (`fireWeapon`), bombs (`useBomb`),
   power-ups (`applyPowerUp`), scoring (`addScore`, `breakChain`), `loseLife`,
   `endGame`, `togglePause`.
8. **Bosses:** `BOSS_PATTERNS` (each boss design has its own attack set, e.g.
   `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireCross`, `fireHomingSeeded`),
   plus `updateBoss` and `killBoss`. Attacks are preceded by a telegraph flash.
9. **Rendering:** `draw()` with `drawPlane`, `drawBossShip`, `drawClouds`, and the
   parallax background. Themes cycle by stage.
10. **Main loop:** `loop(now)` clamps `dt` to 50 ms, updates and draws only when
    running and not paused, then calls `requestAnimationFrame` again.

## Conventions to follow

- **Plain JavaScript and canvas only.** Do not add frameworks, bundlers, or npm
  dependencies unless the user asks for them. Keep everything in `index.html`
  unless there is a clear reason to split it.
- **Match the existing style:** two-space indentation, `const`/`let`, arrow
  functions for callbacks, section headers written as `// ---------- Name ----------`,
  and short comments that explain intent rather than restate code.
- **Tuning goes in the config tables.** Prefer adding an entry to `ENEMY_TYPES`,
  `BOSSES`, `THEMES`, or `DIFFICULTIES` over hard-coding numbers in logic.
- **Respect accessibility settings.** Anything that shakes the screen, flashes, or
  pulses should go through `screenShake` / the `reduced-motion` class so the toggle
  keeps working. Announce important state changes through `announce()`.
- **Respect performance tiers.** Expensive effects (particle counts, `shadowBlur`)
  should consult `quality` so low-end devices stay smooth.
- **Persist only through the `skyfighter.*` keys,** and wrap any new storage use so
  it still works when storage is unavailable.
- **Audio starts only after a gesture.** Call `ensureAudio()` from user-input
  handlers, never on page load.

## Verifying changes

There is no automated test suite. Verify by:

1. Opening `index.html` in a browser and checking the console for errors.
2. Playing through at least one stage on each difficulty when changing gameplay,
   and checking that stage 1 → boss → stage 2 still advances.
3. Testing both keyboard and touch-style controls when changing input, and the
   mute and reduced-motion toggles when changing audio or effects.
4. For bigger changes, a headless Playwright run that fires weapons, triggers a
   bomb, and kills a boss is a good smoke test. Earlier commits in the history were
   verified this way.

## Git

- Write commit messages with a short imperative subject line and details in the
  body, matching the existing history.

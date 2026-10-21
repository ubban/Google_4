# CLAUDE.md

Guidance for working in this repository.

## Project overview

Google_4 is a small static web project containing **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser.

- `index.html` — the entire game: markup (`<body>`), styles (`<style>`, roughly lines 10–359), and game logic (one inline `<script>`, roughly lines 422–1667).
- `README.md` — placeholder title only.

There is no build step, package manager, bundler, test suite, or linter. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static file server. No install is needed.

## Code architecture (`index.html`)

The script is one IIFE (`(function () { ... })();`). Everything lives inside it; nothing is exported or attached to `window` beyond the event listeners.

Major sections, in the order they appear in the script:

1. **DOM references** — `const xxxEl = document.getElementById(...)` at the top.
2. **Storage keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all prefixed `skyfighter.`).
3. **Performance scaling** — `trackPerf(dt)` watches frame times and sets `quality` (0 full, 1 medium, 2 low). Particle counts and glow scale with it.
4. **Tuning tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`. Balance changes usually belong here, not in the logic.
5. **Audio** — Web Audio API, created lazily by `ensureAudio()` on the first user gesture. Effects are built from `beep()` and `noiseBurst()`; the `sfx` object maps named effects to them. Mute is stored in `STORAGE_MUTE`.
6. **UI helpers** — HUD rendering (`renderHearts`, `renderBombs`), screen effects (`flashHit`, `screenShake`, `showStageBanner`), and `announce()` for the `aria-live` region.
7. **Scaling and input** — `resize()` fits a 480×720 logical canvas (`BASE_W`/`BASE_H`) into the window, capped at 1.6×. Keyboard state is in `keys` (keyed by `e.code`). Touch uses a virtual joystick plus fire/bomb buttons. Mouse and touch drag set `mouseTarget`.
8. **Game state** — module-level `let` variables (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`, `chain`, `multiplier`, …). `resetGame()` reinitializes them.
9. **Weapons and bosses** — `fireWeapon()` for the player; `fireRadial`, `fireAimedSpread`, `fireCross`, etc. are boss attack helpers. `BOSS_PATTERNS[i]` lists the attack sequence for each boss design; `updateBoss(dt)` steps through them.
10. **Update and draw** — `update(dt)` advances the simulation; `draw()` renders it. `loop(now)` drives both via `requestAnimationFrame`, clamps `dt` to 0.05 s, and only updates and draws while `running && !paused`.
11. **Lifecycle** — `loseLife()`, `endGame()` (saves `bestScore` when beaten), `togglePause()`, `killBoss()`.

Game flow is a `running` / `paused` / `gameOver` state machine driven by the overlay (`overlay`, `startBtn`).

## Controls

| Action | Keyboard | Touch |
|---|---|---|
| Move | Arrow keys / WASD (via `keys`) | Virtual joystick |
| Fire | Space (hold) | Fire button |
| Bomb | X or Shift | Bomb button |
| Pause | P or Escape | Pause button |
| Mute | Mute button | — |
| Reduced motion | Motion button | — |

The mouse also steers the ship (`mouseTarget`). Touch controls are only shown when `isTouchDevice` is true.

Check the `keydown` handler (around line 769) before changing any binding; it is the source of truth.

## Persistence

Only `localStorage` is used, and every access must tolerate it being unavailable or throwing:

- `skyfighter.bestScore` — high score
- `skyfighter.muted` — `'1'` or `'0'`
- `skyfighter.difficulty` — `easy`, `normal`, or `hard`
- `skyfighter.reducedMotion` — `'1'` or `'0'`; if absent, the `prefers-reduced-motion` media query decides

## Accessibility

The project already handles several accessibility concerns. Keep them working when you change the UI:

- `#srLive` is an `aria-live` region; announce state changes through `announce()`.
- Reduced motion disables or softens screen shake, flashes, and similar effects.
- Touch controls are shown only on touch-capable devices (`isTouchDevice`).

## Conventions

- Vanilla JavaScript and CSS only. No frameworks, no external dependencies, no CDN links.
- Keep everything in `index.html` unless the user asks for a split. Do not introduce modules or a build step.
- Match the surrounding style: 2-space indentation, single quotes in JS, `const` by default, short arrow functions for listeners, and brief comments only where the intent is not obvious.
- Prefer tuning values in the tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`) over hard-coded numbers in logic.
- Preserve the fixed logical resolution (480×720). Coordinates and sizes are in logical pixels, and `resize()` handles scaling.

## Verifying changes

There are no automated tests. To check a change:

1. Open `index.html` in a browser and play a few stages on each difficulty.
2. Exercise the changed path specifically (for example, a new boss pattern, a weapon level, or a pause/resume cycle).
3. Check the browser console for errors.
4. If the change touches layout or touch input, check it at a phone-sized viewport as well as desktop.

## Git

- Develop on the branch assigned to the session, and push with `git push -u origin <branch>`.
- Write commit messages that say what changed and why, in the imperative mood.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single HTML5 canvas page.

Files:

- `index.html` — the entire game: markup, CSS, and one inline `<script>` (an IIFE with `'use strict'`).
- `README.md` — placeholder title only.

There is no build system, package manager, bundler, test suite, or linter. There is no `package.json`. Nothing needs installing.

## Running and checking changes

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- Verify gameplay changes by playing in a browser. There are no automated tests to run.
- Keep the file loadable without network access. Don't add external scripts, fonts, or CDN links unless the user asks for them.

## Code structure of `index.html`

The inline script is organized roughly in this order. Keep new code in the matching section.

1. **DOM references and storage keys** — `getElementById` lookups at the top; `STORAGE_*` constants (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`) for `localStorage`.
2. **Tuning tables** — `DIFFICULTIES` (spawn rates, enemy speed, shot chance, starting lives/bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (sky gradients and names per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (name, colors, HP multiplier).
3. **Audio** — a lazily created `AudioContext` (`ensureAudio`), with `beep` and `noiseBurst` helpers and a mute toggle. Audio must start only after a user gesture.
4. **UI helpers** — hearts/bombs rendering, flash/vignette effects, combo pop, stage banner, screen shake, and `announce()` for screen-reader live text.
5. **Input and layout** — `resize()` scales a fixed 480×720 logical canvas to the viewport (capped at 1.6×); keyboard, mouse/pointer, and touch joystick handling.
6. **Game state** — `resetGame()`, `update(dt)` (the main simulation step), power-ups (`applyPowerUp`), weapon firing (`fireWeapon`), bombs (`useBomb`), score and kill-chain multiplier (`addScore`, `breakChain`), lives (`loseLife`), and end/pause flow (`endGame`, `togglePause`).
7. **Bosses** — `spawnBoss`, `BOSS_PATTERNS` (bullet-pattern functions such as radial, spiral, cross, homing), `updateBoss`, `killBoss`.
8. **Rendering** — `draw()` renders everything with 2D canvas primitives (`drawPlane`, `drawBossShip`, parallax `drawClouds`). Rendering must not mutate game state.
9. **Main loop** — `loop(now)` drives `requestAnimationFrame`, computes `dt`, and calls `update` then `draw`.

Game flow: pick difficulty → fight through waves of enemies → each stage ends after `KILLS_PER_STAGE` kills with a boss → bosses have health bars and cycle through attack patterns → lose all lives to end the run. The best score persists in `localStorage`.

## Conventions

- **Single file.** Keep HTML, CSS, and JS in `index.html` unless the user asks for a split. Don't introduce a build step or framework without being asked.
- **Vanilla JS only**, ES2015+ syntax already in use (`const`/`let`, arrow functions, template literals). Keep the IIFE wrapper and `'use strict'`.
- **Tune through the tables.** Balance changes (speeds, HP, spawn rates, scoring) belong in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS`, not as magic numbers scattered through logic.
- **Fixed logical resolution.** Game coordinates use `BASE_W` × `BASE_H` (480×720). Don't hard-code screen pixel sizes.
- **Time-based updates.** Movement and timers use `dt`, so behavior stays consistent across frame rates.
- **Accessibility.** Respect `prefers-reduced-motion` (the `reducedMotion` flag and `reduced-motion` body class gate shake, flashes, and pulses). Keep `aria-live` announcements for important state changes. Keep the controls hint in sync with real key bindings.
- **Mobile-aware.** Preserve the touch joystick, FIRE and BOMB buttons, `touch-action: none`, and safe-area insets. Test layout at phone width.
- **Persistence.** Wrap `localStorage` access so the game still runs if storage is unavailable or blocked, as the current code does for best score and settings.
- **Keep the existing look.** The HUD is pill-style with heart lives, and the overlay is a glassmorphic card. Match the existing CSS variables, colors, and naming when adding UI.

## Controls (keep the hint text in `#controlsHint` accurate)

- Mouse: move to steer, click to fire.
- Keyboard: Arrows/WASD to move, Space to fire, X or Shift for bomb, P or Esc to pause.
- Touch: drag left zone to fly, tap FIRE, tap BOMB.

## Git and workflow

- Default branch work: develop on the branch the session names. Commit with clear messages, and push with `git push -u origin <branch>`.
- Don't open pull requests unless the user asks.
- Don't commit generated or local-only files.

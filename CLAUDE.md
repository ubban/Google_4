# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small repo containing **Sky Fighter**, a Raiden-style vertical shoot-'em-up built as a single-page HTML5 canvas game.

- `index.html` — the entire game: inline CSS, markup, and one inline `<script>` (an IIFE). Roughly 1,700 lines.
- `README.md` — a stub (only a title and a timestamp line).
- No `package.json`, build step, bundler, dependencies, or test suite. Nothing is compiled or installed.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests or linters configured. Verify changes by playing the game in a browser, checking desktop (keyboard/mouse) and mobile-style (touch) controls, and watching the browser console for errors.

## Architecture (index.html)

Everything lives inside one IIFE in the inline `<script>`. Key parts, in roughly file order:

- **Tuning constants** — `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier per `easy`/`normal`/`hard`), `KILLS_PER_STAGE` (18), `THEMES` (six sky palettes, cycled by stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), `BOSSES` (six bosses, cycled), and `BOSS_PATTERNS` (bullet patterns, e.g. radial, spiral, cross, homing).
- **Persistence** — `localStorage` keys prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`, `reducedMotion`. Every access is a direct call; keep that in mind before adding storage that might be unavailable (private mode, blocked storage).
- **Audio** — Web Audio synthesis (`beep`, `noiseBurst`) with a lazy `ensureAudio()`. No audio files are loaded.
- **Rendering** — drawing helpers such as `drawPlane`, `drawBossShip`, and the parallax/sun/cloud background. The logical play field is `BASE_W` × `BASE_H` = 480 × 720, scaled to fit the window by `resize()`.
- **Game loop** — `update(dt)` drives spawning, collisions (`rectsOverlap`), scoring (`addScore`, kill-chain multiplier via `breakChain`), power-ups (`applyPowerUp`), bombs (`useBomb`), and boss updates (`updateBoss`).
- **HUD/UI** — DOM overlays (`renderHearts`, `renderBombs`, stage banner, combo pop, vignette, flash). `announce()` feeds an ARIA live region for screen readers.
- **Input** — keyboard listeners on `window`, mouse on the canvas, and touch joystick plus FIRE/BOMB buttons for mobile.

### Gameplay facts worth keeping in sync

- Stages advance after `KILLS_PER_STAGE` kills, then a boss fight begins. Defeating the boss increments `stage`.
- Difficulty affects lives, bombs, enemy speed, shooting frequency, and boss HP.
- Pausing is `P` or `Esc`, or the pause button.

## Controls (must match the in-game hint at `#controlsHint`)

- Mouse: move to steer, click to fire.
- Arrows/WASD: move. Space: fire.
- X or Shift: bomb. P or Esc: pause.
- Mobile: drag left side to fly, tap FIRE, tap BOMB.

If you change a control, update the `#controlsHint` text in the same change.

## Conventions

- Keep the game self-contained in `index.html`. Don't add a build system, framework, or external runtime dependencies unless the user asks for it.
- Keep the existing style: `const` for tuning tables, UPPER_SNAKE_CASE for constants, camelCase for functions and variables, two-space indentation.
- Keep all tuning values in the constant tables near the top of the script rather than hard-coding them in logic.
- Preserve accessibility hooks: the reduced-motion toggle (`body.reduced-motion` CSS rules), the `announce()` live region, and `touch-action`/`user-select` settings that prevent accidental scrolling on mobile.
- Keep the layout working at phone widths; the canvas scales via `resize()`.
- Mind performance: `trackPerf()` implements adaptive quality, so avoid per-frame allocations in hot paths (`update`, drawing).

## Git

- Development branch for this work: `claude/gifted-tesla-n5zlja`. Commit and push there; don't push to other branches without permission.
- Write clear commit messages describing the gameplay or code change.
- Don't create pull requests unless explicitly asked.

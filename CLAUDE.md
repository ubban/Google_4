# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free static project. Its only real content is
**Sky Fighter**, a Raiden-style vertical shoot-'em-up written as a single
self-contained HTML5 canvas page.

| File        | Purpose                                                        |
|-------------|----------------------------------------------------------------|
| `index.html`| The entire game: inline CSS (`<style>`), markup, and JS (`<script>`). |
| `README.md` | Placeholder title only.                                        |

There is no build step, package manager, test suite, linter, or CI config.
Do not add tooling unless the user asks for it.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static
server (for example `python3 -m http.server`) and browse to it. Nothing needs
to be installed.

## Code layout inside `index.html`

- **CSS** (`<style>`, roughly lines 10–360): HUD pills, glassmorphic overlay
  card, touch controls, boss health bar, and the `.sr-only` helper for screen
  readers.
- **Markup** (`<body>`, roughly lines 361–421): `#container` holds the
  `<canvas id="gameCanvas">`, HUD elements, touch controls, and the `#overlay`
  start/pause/game-over card. Element IDs are the contract between markup and
  script; renaming one means updating every `getElementById` call.
- **Script** (`<script>`, roughly lines 422–1668): one IIFE. Sections, in order:
  1. DOM references and `STORAGE_*` keys for `localStorage`.
  2. Tuning tables: `DIFFICULTIES`, `THEMES` (per-stage backgrounds),
     `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`.
  3. Persistence (best score, mute, difficulty, reduced motion).
  4. Audio: a lazily created `AudioContext` plus `beep` and `noiseBurst`
     helpers, and named sound effects in `sfx`.
  5. HUD and effect helpers (hearts, bombs, combo pops, stage banner, shake,
     screen flash).
  6. Input: keyboard, touch joystick and buttons, and mouse steering.
  7. Game state and entity spawning: `resetGame`, `spawnEnemy`, `spawnBoss`,
     `spawnExplosion`.
  8. Weapons and boss patterns: `fireWeapon`, `applyPowerUp`, and the
     `fire*` pattern functions used by `updateBoss`.
  9. `update(dt)`, the per-frame simulation. `draw()` renders each frame.
     `loop(now)` drives both through `requestAnimationFrame`.
  10. `endGame` records the best score and shows the overlay.

## Conventions

- **Single file, no modules.** Keep CSS, markup, and JS in `index.html`. Do
  not split out separate files, bundlers, or frameworks unless asked.
- **Vanilla JS, ES2017+ style.** Use `const`/`let`, arrow functions for
  callbacks, and `addEventListener`. Match the existing naming: `camelCase`
  functions and variables, `UPPER_SNAKE_CASE` for constant tables, and
  `STORAGE_*` for `localStorage` keys (prefixed `skyfighter.`).
- **Tuning lives in data tables.** Add or change enemies, bosses, themes, and
  difficulty values by editing the tables near the top of the script rather
  than hard-coding numbers in logic.
- **Fixed logical resolution.** The game is laid out at `BASE_W × BASE_H`
  (480 × 720) and scaled in `resize()`. Coordinates in game logic use these
  base units.
- **Frame-rate independence.** Update functions take `dt`. Scale movement and
  timers by it instead of counting frames.
- **Adaptive quality.** `trackPerf` lowers the `quality` level when frames run
  slow. Expensive effects should respect it.
- **Accessibility.** Respect `reducedMotion` (user setting and the
  `prefers-reduced-motion` media query) for screen shake and flashes. Announce
  important state changes through the `announce()` helper, which writes to the
  `#srLive` live region.
- **Guard browser APIs.** Wrap `localStorage` access in `try`/`catch` (as the
  audio code does for `AudioContext`), because storage can be blocked. Create
  the `AudioContext` only after a user gesture.
- **Keep the mobile layout working.** The page uses `viewport-fit=cover`,
  `touch-action` rules, and on-screen joystick and buttons for touch devices.
  Check that changes to HUD or overlay CSS still fit a phone-sized viewport.

## Controls (for reference when changing input code)

- Move: arrow keys or WASD (`KeyW`/`KeyA`/`KeyS`/`KeyD`), or mouse hover over the canvas.
- Fire: hold `Space`.
- Bomb: `X` or either `Shift`.
- Pause: `P` or `Esc`.
- Mute, reduced-motion, and pause buttons are in the top bar (`#topBar`).
- Touch: left-side joystick, right-side fire and bomb buttons.

## Persistence keys

`localStorage` entries used by the game (keep these stable so players do not
lose saved data):

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

## Verifying changes

There are no automated tests. After editing `index.html`:

1. Load the page in a browser and confirm there are no console errors.
2. Start a run, then check movement, firing, a bomb, pausing, and game over.
3. Check that the best score persists across a reload.
4. For layout or CSS changes, check a narrow (phone-sized) viewport as well as
   a desktop one.

Headless browser checks are fine. Playwright is the usual choice if you want
to script this.

## Git workflow

- Develop on the branch the session specifies. Do not push to other branches
  without permission.
- Write commit messages in the imperative mood with a short summary line, and
  describe the why in the body when it is not obvious from the diff.
- Do not open pull requests unless the user asks for one.

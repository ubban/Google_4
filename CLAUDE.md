# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only app is **Sky Fighter**, a Raiden-style
vertical shoot-'em-up that runs in a single HTML5 canvas page.

- `index.html` — the entire game: markup, CSS (`<style>`), and JavaScript (`<script>`).
- `README.md` — one-line project title. Keep it short.

There is no build step, no package manager, no bundler, no test suite, and no linter.
Do not add any of these unless the user asks for them.

## Running locally

Open `index.html` directly in a browser, or serve the folder with any static server
(for example `python3 -m http.server` from the repo root) and visit it.

## Code layout inside `index.html`

The JavaScript is one IIFE (`(function () { 'use strict'; ... })();`) with no modules.
Sections are marked with `// ---------- Name ----------` comments. Keep that convention.

| Section | Purpose |
| --- | --- |
| DOM lookups | `document.getElementById` for every HUD/overlay element, captured once at the top |
| Storage keys | `skyfighter.*` keys in `localStorage` (best score, mute, difficulty, reduced motion) |
| Adaptive performance | `trackPerf` lowers/raises a `quality` level (0–2) from average frame time |
| Config tables | `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, `KILLS_PER_STAGE` |
| Audio | Lazily created `AudioContext` (`ensureAudio`), `beep` and `noiseBurst` helpers |
| HUD / feedback | hearts, bombs, combo pops, stage banner, screen shake, hit flash |
| Input | `keys` map from `keydown`/`keyup`, joystick and buttons for touch devices |
| Game state | `resetGame`, `update(dt)`, `draw()`, `loop(now)` driven by `requestAnimationFrame` |
| Weapons and bombs | `fireWeapon`, `useBomb`, `applyPowerUp` |
| Bosses | `spawnBoss`, `updateBoss`, `killBoss`, pattern functions (`fireRadial`, `fireCross`, ...) |

### Key bindings

- Move: arrow keys / WASD (via `keys`), or the on-screen joystick on touch devices
- Fire: hold Space / fire button
- Bomb: `X` or Shift / bomb button
- Pause: `P` or Escape

## Conventions

- **Vanilla JS and HTML only.** Do not introduce frameworks, npm dependencies, or external
  scripts. Everything stays in `index.html` unless the user asks for a split.
- **Canvas coordinates** use the fixed logical size `BASE_W = 480`, `BASE_H = 720`. `resize()`
  scales the canvas to the viewport. Write game logic in logical units, not CSS pixels.
- **Time-based updates.** `update(dt)` receives seconds. `dt` is clamped to 0.05 in `loop`.
  Use `dt` for movement and timers rather than per-frame counts.
- **Tuning lives in data tables.** Prefer adding or editing entries in `DIFFICULTIES`,
  `ENEMY_TYPES`, `BOSSES`, `THEMES`, or `BOSS_PATTERNS` over hard-coding numbers in logic.
- **Persistence** goes through the `STORAGE_*` constants. Wrap new storage reads and writes
  so the game still works if `localStorage` is unavailable (private mode, blocked storage).
- **Accessibility.** Announcements go through `announce()` into the `srLive` region. Respect
  `reducedMotion` for screen shake and flashes. Keep the `prefers-reduced-motion` default.
- **Mobile.** The page is designed for phone-width screens with touch controls. Check that
  layout changes still work at small widths and do not add horizontal page scroll.
- **Style.** Match the surrounding code: 2-space indent, single quotes, `const`/`let`,
  arrow functions for callbacks, and short `//` section comments. Comments explain intent,
  not mechanics that are already obvious from the code.

## Making changes

- Read the relevant section of `index.html` before editing. The file is large (~1,700
  lines), so search for the function or table you need instead of reading it whole.
- Test changes by loading the page in a browser. There are no automated tests, so manually
  check the affected flow: start, play, pause, bomb, boss, game over, and the high-score
  save if you touched scoring or storage.
- Keep commits focused. Use a short imperative subject line that describes the game change,
  for example `Tune BEHEMOTH boss HP on hard difficulty`.

## Git workflow

- Develop on the branch assigned for the session. Do not push to other branches without
  explicit permission.
- Do not open a pull request unless the user asks for one.
- Do not commit editor, OS, or dependency files. The repo currently has none of these, so
  add a `.gitignore` only when the user asks for it.

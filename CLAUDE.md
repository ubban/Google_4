# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single HTML5 canvas page.

- `index.html` — the whole game: markup, CSS (inline `<style>`), and JavaScript (inline `<script>`, wrapped in an IIFE). About 1,670 lines.
- `README.md` — a stub. Keep it minimal unless the user asks for more.

There is no build step, no package manager, no bundler, no test suite, and no linter configured. Do not add any of these unless the user asks.

## Running and checking changes

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser, and check the browser console for errors.
- For gameplay or UI changes, test at phone width as well as desktop width. The game supports touch controls.

## Code structure (inside `index.html`)

The script keeps all state in closure variables and runs one main loop (`loop(now)` → `update(dt)` → `draw()`) driven by `requestAnimationFrame`. Key areas, in roughly file order:

- **Persistence constants** (`STORAGE_*`): all `localStorage` keys are prefixed `skyfighter.` — `bestScore`, `muted`, `difficulty`, `reducedMotion`. Keep the prefix when adding keys. Wrap storage access so the game still works when storage is unavailable (private mode, blocked site data).
- **Tuning tables**: `DIFFICULTIES` (spawn rate, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`. Prefer changing these tables over hard-coding numbers in logic.
- **Audio**: a Web Audio synth (`beep`, `noiseBurst`, the `sfx` object). `AudioContext` is created lazily on the first user gesture (`ensureAudio`). Respect the mute flag.
- **Input**: keyboard (`keys[e.code]`), mouse/pointer steering and fire, and a touch joystick plus on-screen FIRE/BOMB buttons. Keyboard bindings: arrows/WASD move, Space fires, X or Shift uses a bomb, P or Esc pauses.
- **Game logic**: `resetGame`, `update`, `fireWeapon` (Vulcan and Laser, each with 3 power levels), `useBomb`, power-ups (`applyPowerUp`), the boss pattern functions (`BOSS_PATTERNS`, `fireRadial` and related), `loseLife`, and `endGame`.
- **Rendering**: `draw()` and helpers such as `drawPlane`, `drawBossShip`, `drawClouds`. Rendering is plain 2D canvas, no libraries.
- **UI/HUD**: DOM elements for hearts, bombs, stats, and overlays, updated by functions such as `renderHearts`, `renderBombs`, and `showStageBanner`.
- **Accessibility**: a reduced-motion toggle (`motionBtn`) disables screen shake, hit flash, and low-health pulse. It defaults to the `prefers-reduced-motion` media query and is stored after the user changes it.

## Conventions

- Keep the project as a single self-contained `index.html` unless the user asks for a split. Do not introduce external CDN scripts or fonts without a reason.
- Match the existing style: 2-space indentation, single quotes in JavaScript, `const` and `let` (no `var`), and small named functions inside the IIFE.
- Keep new globals out of the window scope; add them inside the existing IIFE.
- Preserve the touch and mobile experience. Anything bound to `keydown` should have a touch or pointer equivalent where the feature matters for play.
- Keep frame-rate-dependent logic based on the `dt` passed to `update`, not on raw frame counts, unless the existing code does otherwise for that specific value.
- Don't change `KILLS_PER_STAGE`, difficulty values, or the storage key names without saying so in the commit message, because they affect saved player data.

## Git workflow

- Develop on the branch the session specifies (currently `claude/gifted-tesla-fdxlhr`). Do not push to other branches without explicit permission.
- Write clear commit messages that describe the gameplay or code change, not just the file touched.
- Do not open a pull request unless the user asks for one.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its main content is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up that runs in the browser on a single HTML
canvas. There is no build step, no package manager, and no test suite.

Files:

- `index.html` (~1670 lines): the whole game. CSS is in a `<style>` block, markup
  in the body (canvas plus HUD and overlay elements), and all logic in one inline
  `<script>` wrapped in an IIFE.
- `README.md`: a placeholder title. Keep it accurate if you expand it.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static
  server (for example `python3 -m http.server`).
- There are no automated tests, linters, or build commands. Verify changes by
  playing the game in a browser, ideally at both desktop and phone widths.

## Architecture of `index.html`

Everything runs inside one IIFE, so most helpers are private to the script.

- **Configuration constants** near the top of the script:
  - `DIFFICULTIES` (easy / normal / hard): spawn rates, enemy speed, enemy fire
    chance, starting lives and bombs, and boss HP multiplier.
  - `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
- **Persistence**: `localStorage` holds the best score, mute setting, difficulty,
  and reduced-motion preference. Every access should stay guarded so the game still
  works when storage is unavailable.
- **Audio**: a WebAudio synth (`ensureAudio`, `beep`, `noiseBurst`, and the `sfx`
  object). It starts only after a user gesture, so call `ensureAudio()` from input
  handlers.
- **Game loop**: `requestAnimationFrame` drives `loop(now)`, which clamps `dt` to
  0.05 s and then calls `update(dt)` and `draw()`. `resetGame()` initializes state.
  State lives in module-level `let` variables.
- **Update logic** (`update`, `updateBoss`, `applyPowerUp`, `fireWeapon`,
  `useBomb`, `loseLife`, `endGame`): game rules, collisions (`rectsOverlap`),
  scoring (`addScore`, combo chain via `breakChain`), and stage progression.
- **Boss attacks**: a family of pattern helpers (`fireRadial`, `fireAimedSpread`,
  `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`,
  `fireDoubleRing`).
- **Rendering**: `draw()` and helpers such as `drawPlane`, `drawBossShip`, and
  `drawClouds`. Drawing is plain canvas 2D.
- **HUD and UI**: DOM elements updated from JS (`renderHearts`, `renderBombs`,
  `showStageBanner`, `showComboPop`, `announce`). `screenShake` and `flashHit`
  handle feedback effects.
- **Input**: keyboard, mouse on the canvas, and touch (virtual joystick in
  `moveZone`, plus fire and bomb buttons). Touch handlers call `preventDefault`,
  and `touch-action: none` is set on the page.
- **Performance**: `trackPerf` monitors frame times.

## Controls (for reference when changing input code)

- Move: arrow keys or WASD. Touch uses the joystick.
- Fire: Space (held), or the fire button.
- Bomb: X or Shift, or the bomb button.
- Pause: P or Escape.

## Conventions

- Keep the project as dependency-free static files. Do not add a build system or
  npm packages unless the user asks for one.
- Keep the game in a single file unless the user asks to split it. If you do
  split it, update this document.
- Add new tuning values to the `DIFFICULTIES` or constant objects rather than
  hardcoding them in update logic.
- Preserve the mobile layout. The page uses `viewport-fit=cover`, safe-area insets,
  `touch-action: none`, and `user-select: none`. Check phone widths after UI
  changes.
- Respect `prefers-reduced-motion` and the mute setting when adding effects or
  sounds.
- Guard all `localStorage` access in try/catch-equivalent logic (the current code
  reads it directly, so keep new reads defensive).
- Keep the code style consistent with the existing file: 2-space indent, single
  quotes, and small named helper functions.

## Git

- Development branch for this work: `claude/gifted-tesla-35xk4r`. Push there, and
  do not push to other branches without explicit permission.
- Commit messages should be short and imperative, describing the change.
- Do not create pull requests unless the user asks for one.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-'em-up that runs in the browser as a
single self-contained page. The whole game (markup, CSS, and JavaScript) lives in
`index.html`. `README.md` is a placeholder.

There is no build step, package manager, bundler, framework, or test suite. Do not
add one unless the user asks for it.

## Repository layout

- `index.html` — the entire game:
  - `<head>`: meta tags and title.
  - `<style>` (around lines 10–359): all CSS. Includes the HUD, the overlay card,
    touch controls, and the reduced-motion and "low quality" styles.
  - `<body>` (around lines 361–421): static DOM for the canvas, HUD, boss bar,
    top-bar buttons, touch controls, and the start/pause overlay.
  - `<script>` (around lines 422–1667): one IIFE with `'use strict'`. Nothing is
    exposed on `window`.
- `README.md` — project name only.

## Architecture of the game script

The script is organized as one IIFE with these sections, in roughly this order:

1. **DOM references**: `const` lookups by id at the top of the IIFE.
2. **Persistence keys**: `skyfighter.*` keys in `localStorage` (best score, mute,
   difficulty, reduced motion).
3. **Adaptive performance**: `trackPerf` samples frame time over 45 frames and steps
   a `quality` level (0 full, 1 medium, 2 low) up or down.
4. **Tuning tables** (plain data, the usual place to balance the game):
   - `DIFFICULTIES`: spawn rates, enemy speed and fire chance, starting lives and
     bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` (18) and `THEMES` (one per stage, cycled).
   - `ENEMY_TYPES`: grunt, interceptor, tank, with HP, speed, score, and the stage
     they first appear in.
   - `BOSSES`: six bosses, cycled by stage, with colors and an HP multiplier.
   - `BOSS_PATTERNS` (further down): the bullet patterns bosses cycle through.
5. **Audio**: a lazily created `AudioContext` (`ensureAudio`), with `beep` and
   `noiseBurst` helpers and an `sfx` object. Audio is created on the first user
   gesture. Do not create it at load time.
6. **HUD and effects**: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`,
   `showStageBanner`, `screenShake`.
7. **Layout and input**: `resize` (fits a `BASE_W × BASE_H` = 480×720 logical
   canvas to the viewport), keyboard handlers, the touch joystick, and mouse
   handlers on the canvas.
8. **Game state and simulation**: `resetGame`, `spawnEnemy`, `spawnBoss`,
   `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `applyPowerUp`, `update(dt)`,
   `updateBoss(dt)`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
9. **Rendering**: `draw()` and its helpers `drawPlane`, `drawBossShip`,
   `drawClouds`.
10. **Main loop**: `loop(now)` clamps `dt` to 50 ms, then calls `update` and `draw`
    while the game is running and not paused, then requests the next frame. The
    start button is wired near the end of the script.

Game state is held in module-level `let` variables inside the IIFE. Keep it there;
do not move it onto `window`.

## Conventions

- **Single file.** Keep HTML, CSS, and JS in `index.html` unless the user asks for a
  split. Use the existing `<style>` and `<script>` blocks.
- **Match the local style.** The script uses 2-space indentation, single quotes for
  JS strings, `const` by default, arrow functions for callbacks, and short section
  banner comments such as `// ---------- Name ----------`. Follow the same pattern
  for new sections, and avoid heavy commenting.
- **Naming.** Tuning constants are UPPER_SNAKE_CASE. Storage keys use the
  `STORAGE_*` prefix and the `skyfighter.` value prefix. DOM references use the
  element id with an `El` or `Btn` suffix where that helps (`scoreEl`, `startBtn`).
- **Frame-rate independence.** Game logic should scale by `dt` (seconds), as
  `update(dt)` does. Do not tie movement to frame count.
- **Accessibility.** Respect `prefers-reduced-motion`. The game's reduced-motion
  toggle controls screen shake, flashes, and the low-health pulse. Announcements for
  screen readers go through `announce()` into the `#srLive` region.
- **Storage access.** Keep `localStorage` reads and writes limited to the `skyfighter.*`
  keys. Wrap new storage use in a try/catch if it could be called in a restricted
  browsing context, since the game currently assumes storage is available.
- **Mobile.** Touch controls are shown for touch devices. Any new control needs a
  touch path as well as keyboard and mouse.

## Running and testing

- Open `index.html` directly in a browser. No server is required.
- There are no automated tests and no linter configured. Verify changes by playing:
  start a game, test each difficulty, collect power-ups, trigger a bomb, reach a
  boss (a boss warning fires after 18 kills in a stage), pause and resume, and
  check the game-over overlay and best-score persistence after a reload.
- For visual or layout changes, check both a desktop viewport and a narrow phone
  viewport (around 390 px wide), since the layout must not scroll horizontally.
- For a quick syntax check of the script without a browser, extract the `<script>`
  block to a scratch file and run `node --check` on it:
  `awk '/<script>/{f=1;next}/<\/script>/{f=0}f' index.html > game_script.js && node --check game_script.js`
  (write the scratch file outside the repo, or delete it afterward).

## Git

- Develop on the branch the session names (currently `claude/gifted-tesla-ihnu7k`).
  Do not push to other branches without explicit permission.
- Write descriptive commit messages that explain the gameplay or code change.
- Do not open a pull request unless the user asks for one.

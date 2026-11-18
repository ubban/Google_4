# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on an HTML5 canvas.

Files:

- `index.html` (~1,670 lines): the entire game. Inline `<style>` (lines ~10–359), markup (~360–421), and one inline `<script>` IIFE (~422–1667).
- `README.md`: a one-line title stub. Keep it short unless the user asks for more.
- No `package.json`, build step, bundler, test suite, linter, or CI config exists. Do not invent commands for them.

## Running and testing

- Open `index.html` directly in a browser, or serve the folder with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser. For UI or gameplay changes, use the `run` skill or Playwright (pre-installed Chromium) and check the result visually.
- Keep the file loadable as-is. Do not add a build step unless the user asks for one.

## Architecture of `index.html`

Everything runs inside one IIFE. Top-level game state lives in `let`/`const` variables in that closure, not on `window`.

Major sections of the script, in order:

1. **Persistence keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all prefixed `skyfighter.`). Every `localStorage` access must be wrapped so the game still works when storage is blocked or unavailable.
2. **Tuning data**: `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six boss designs with HP multipliers), and `BOSS_PATTERNS` (bullet patterns, defined near line 1372).
3. **Audio**: Web Audio helpers `ensureAudio`, `beep`, `noiseBurst`, and sound-effect maps such as `EXPLODE_SFX`. Audio starts only after a user gesture (`ensureAudio`).
4. **HUD and feedback**: `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`. HUD elements are DOM nodes overlaid on the canvas.
5. **Layout and input**: `resize()` scales the canvas to fit `BASE_W × BASE_H` (480 × 720). Keyboard, mouse, and touch handling includes a virtual joystick and fire and bomb buttons for mobile.
6. **Game logic**: `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `update(dt)`, `applyPowerUp`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
7. **Rendering**: `draw()` and helpers `drawPlane`, `drawBossShip`, `drawClouds`. Drawing is canvas-only.
8. **Main loop**: `loop(now)` drives `update` and `draw` with delta time. `trackPerf` adapts a `quality` level to frame time.

### Controls

- Move: arrow keys or WASD. Mouse moves the ship when the pointer is over the canvas.
- Fire: Space (also held mouse button).
- Bomb: X or Shift.
- Pause: P or Escape.
- Touch: virtual joystick on the left, fire and bomb buttons.

## Conventions

- Keep the game in **one self-contained HTML file** with inline CSS and JS, unless the user asks to split it. Do not add external scripts or stylesheets without being asked. Do not add CDN dependencies.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, small named functions, and comments only where the logic is not obvious.
- Tuning values (speeds, spawn rates, HP, scores) are data at the top of the script. Change them there rather than hardcoding magic numbers in the logic.
- Keep the layout responsive. The game must work at phone widths and use `touch-action: none` on the canvas and controls.
- Respect `prefers-reduced-motion` and the in-game reduced-motion toggle when adding screen effects. Screen shake and flashes are the existing examples.
- Do not break the mute toggle. All new sounds must go through the existing audio helpers so they respect mute.

## Git workflow

- Default development branch for this session: `claude/gifted-tesla-8jtte4`. Commit and push there. Do not push to other branches without explicit permission.
- Commit messages should describe what changed and why. Follow the style of the existing history (a short summary line, then a bulleted body for larger changes).
- Do not create a pull request unless the user explicitly asks for one.
- Never commit secrets. This repo currently has none.

## Things to avoid

- Do not add a framework, bundler, or package manager without a clear request.
- Do not move game state onto `window` or leak globals from the IIFE.
- Do not remove the touch controls or break mobile layout while changing desktop input.
- Do not change saved-data keys (`skyfighter.*`) without a migration plan, because they hold players' best scores and settings.

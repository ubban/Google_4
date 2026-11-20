# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up with stages, weapon power-ups, smart bombs, and boss fights.

- The whole game lives in one self-contained file: `index.html` (~1,670 lines). It contains the HTML, an inline `<style>` block, and one inline `<script>` block (starts around line 422). The game draws to a `<canvas id="gameCanvas">`.
- `README.md` is a stub (title plus a timestamp line). There is no build step, package manager, test suite, or CI configuration.
- There are no external assets. Sound is synthesized at runtime with the WebAudio API, and the favicon is an inline SVG data URI.

## Running and verifying

- There is nothing to install or compile. Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- Verify changes by playing the game in a browser. Checking for JavaScript syntax errors in the console is also worthwhile, since the script is one large inline block.
- Lint and test tooling does not exist yet. Do not add a `package.json`, build pipeline, or test framework unless the user asks for one.

## Code structure (inside `index.html`)

Roughly in file order:

1. **Head and CSS** (lines ~1–420): layout, HUD (`#hud`, `.stat` pills, hearts, bombs, weapon and multiplier readouts), overlays, vignette, and touch controls. Dark color scheme only (`color-scheme: dark`).
2. **Markup** (lines ~360–421): canvas, HUD, overlays for menu/pause/game over, the control hints, the header buttons (mute, difficulty, reduced motion), and the on-screen fire and bomb buttons for touch devices.
3. **Script** (starts ~line 422):
   - **Config constants:** `STORAGE_*` keys (`skyfighter.*` in `localStorage`), `DIFFICULTIES` (easy / normal / hard: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (six stage color palettes, cycled per stage), `ENEMY_TYPES`, `BOSSES`, `BASE_W`/`BASE_H` (480×720 logical play field).
   - **Audio:** `ensureAudio`, `beep`, `noiseBurst`. The audio context is created lazily on the first user gesture.
   - **Input:** keyboard handlers (`keydown`/`keyup`, populating `keys[e.code]`), a touch joystick (`joystickStart`/`joystickMove`/`joystickEnd`), and a fire button. Pointer and touch coordinates are mapped through `canvasPointFromEvent`.
   - **Layout:** `resize()` scales the canvas to the viewport while keeping the logical size.
   - **Game state:** `resetGame()` builds the `player` object and other state. `update(dt)` advances the simulation each frame. `draw()` renders everything. `loop(now)` drives the frame loop and calls `trackPerf(dt)`.
   - **Combat:** `fireWeapon` (vulcan and laser), `spawnEnemy`, `spawnBoss`, `useBomb`, `applyPowerUp`, `maybeSpawnPowerUp`, combo/chain scoring (`addScore`, `breakChain`).
   - **Bosses:** `BOSS_PATTERNS` lists bullet patterns (radial, aimed spread, columns, spiral, cross, homing, double ring). `updateBoss` and `killBoss` manage phases.
   - **Lifecycle:** `loseLife`, `endGame`, `togglePause`.
   - **Accessibility:** `announce()` writes to a screen-reader live region. Reduced motion defaults to the `prefers-reduced-motion` media query and can be toggled with the header button (`#motionBtn`); the choice is persisted as `skyfighter.reducedMotion`.

## Controls

- **Keyboard:** arrow keys or WASD to move, Space to fire, X or Shift to drop a smart bomb, P or Escape to pause.
- **Mouse:** move to steer, click to fire (`canvasPointFromEvent`).
- **Touch:** drag on the left side to fly (`joystickStart`/`joystickMove`), tap FIRE, tap BOMB.

## Conventions

- Keep the game in a single file unless the user asks to split it. Do not introduce external CDN scripts or assets without a reason. The project is designed to work offline.
- Match the existing style: two-space indentation, single quotes in JS, `const`/`let`, and section banner comments such as `// ---------- Audio ... ----------`.
- Tune gameplay through the constants in the config block (`DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `BOSSES`, `ENEMY_TYPES`) rather than hard-coding numbers in update logic.
- Persist player preferences only through the existing `skyfighter.*` `localStorage` keys. Read them defensively, since `localStorage` can be unavailable in some browsers or private modes.
- Keep the UI responsive: the HUD must remain usable at phone widths, and touch controls must not rely on hover.
- Respect reduced motion (`reducedMotion` flag, `reduced-motion` body class): screen shake, heart-beat animation, and flashes should be reduced or skipped when it is active.

## Git

- Development branch for this work: `claude/gifted-tesla-jwg4hn`. Commit and push there.
- Commit messages should be short and imperative, following the existing history (for example, "Add Sky Fighter: Raiden-style vertical shoot-em-up game").
- Do not open pull requests unless the user asks for one.

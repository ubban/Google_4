# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game (HTML, CSS, and JavaScript) lives in one file, `index.html`. There is no build step, no package manager, no test suite, and no external dependencies. `README.md` is a placeholder.

To run it, open `index.html` in a browser, or serve the directory with any static server.

## Repository layout

| Path         | Purpose                                                         |
|--------------|-----------------------------------------------------------------|
| `index.html` | The whole game: markup, inline `<style>`, and inline `<script>` |
| `README.md`  | Placeholder readme                                              |

## Architecture of `index.html`

The file has three parts: a `<style>` block (HUD, overlays, touch controls, reduced-motion and dark-scheme rules), the DOM markup (canvas, HUD, boss bar, stage banner, overlays, touch controls), and one `<script>` block containing the game. The script is organized in sections, marked with `// ---------- Name ----------` comments:

- **Storage keys and config**: `STORAGE_*` constants for `localStorage` (best score, mute, difficulty, reduced motion).
- **Tuning data**:
  - `DIFFICULTIES`: `easy` / `normal` / `hard`, with spawn rates, enemy speed, shoot chance, starting lives and bombs, and `bossHpMul`.
  - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
  - `THEMES`: six sky palettes, one per stage cycle.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), and `tank` (4 HP, from stage 2).
  - `BOSSES`: six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN), each with colors and an `hpMul`.
- **Audio**: `ensureAudio`, `beep`, and `noiseBurst`, built on the Web Audio API. Audio starts on the first user gesture.
- **HUD and feedback**: `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`.
- **Layout and input**: `resize` scales a fixed `BASE_W × BASE_H` (480 × 720) canvas to fit the viewport. Keyboard listeners, mouse steering and firing, and a touch joystick plus FIRE and BOMB buttons are handled here.
- **Game state**: `resetGame` initializes state. `update(dt)` is the per-frame simulation, `draw()` renders it, and `loop(now)` drives the `requestAnimationFrame` loop.
- **Entities and combat**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `addScore`, and the combo chain (`breakChain`).
- **Boss attacks**: `BOSS_PATTERNS` holds the bullet patterns (radial, aimed spread, columns, spiral, cross, homing, double ring). `updateBoss` selects and runs them, and `killBoss` handles defeat.
- **Lifecycle**: `loseLife`, `endGame`, `togglePause`.

### Controls

- Keyboard: arrows or WASD to move, Space to fire, X or Shift to use a bomb, P or Esc to pause.
- Mouse: move to steer, click to fire.
- Touch: drag left to fly, tap FIRE, tap BOMB.

## Conventions

- Keep the game self-contained in `index.html` unless the user asks for a split. Do not add a build system, bundler, or npm dependencies without asking.
- Keep tuning values in the existing data tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, `BOSS_PATTERNS`) rather than hard-coding numbers in logic.
- Preserve the `prefers-reduced-motion` handling (`body.reduced-motion`) when adding visual effects, and keep effects that shake or flash optional under it.
- Guard all `localStorage` access with try/catch. Storage can be blocked in private windows or sandboxed frames.
- Keep the layout usable on phones: the canvas scales with `resize`, and touch controls appear only on touch devices.
- Use the existing section comment style when adding new code.

## Testing and verification

There is no automated test suite. To verify a change:

1. Open `index.html` in a browser, or run a static server such as `python3 -m http.server` from the repo root.
2. Check the affected path in play: a new enemy, boss pattern, weapon, or power-up should be exercised through to its effect.
3. Check the console for errors. Also try the three difficulties and both keyboard and touch controls where relevant.
4. For visual changes, check a narrow (phone-width) viewport as well as desktop.

A headless browser (Playwright, with Chromium preinstalled in the cloud environment) can be used for smoke tests.

## Git

- Develop on the branch the session names, and do not push to other branches without permission.
- Write clear commit messages that describe the gameplay or code change.
- Do not open pull requests unless the user asks for one.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small static web game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game lives in one file, `index.html`, with inline CSS and JavaScript. `README.md` is a placeholder with no setup or usage notes.

## Layout

- `index.html`: the complete game (about 1,670 lines). Sections, top to bottom:
  - `<head>`: meta tags, an inline SVG favicon, and a single inline `<style>` block.
  - Body markup: the canvas, HUD, pause and start overlays, difficulty buttons, and touch controls.
  - One inline `<script>` that holds all game logic, audio, input, and rendering.
- `README.md`: placeholder.
- No `package.json`, build step, bundler, test suite, linter, or CI config exists.

## Running and checking changes

- There is nothing to install or build. Open `index.html` directly in a browser, or serve the folder with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser. Check the start, pause, and game-over flows, every difficulty, touch controls, and the mute and reduced-motion toggles.
- Keep the file as a single self-contained page. Do not add external scripts, stylesheets, or fonts unless the user asks for them.

## Code structure inside the script

Constants and config are near the top of the script:

- `DIFFICULTIES`: `easy`, `normal`, and `hard`. Each entry sets spawn timing, enemy speed, enemy fire chance, starting `lives`, starting `bombs`, and `bossHpMul`.
- `ENEMY_TYPES`: `grunt`, `interceptor`, and `tank`, each with `minStage` and `score`.
- `BOSSES`: six bosses (VANGUARD through TITAN). Each entry has colors and an `hpMul`. Their attack patterns are the `BOSS_PATTERNS` array and the `fire*` helpers.
- `THEMES`: six stage palettes. The theme changes as stages advance.
- `KILLS_PER_STAGE`: kills needed to advance a stage.

Persistence uses `localStorage` with keys prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`, and `reducedMotion`. Every access should be guarded, because storage can be unavailable in some browsers and private windows.

The main loop is `update(dt)` → `draw()`, driven by `requestAnimationFrame` in `loop(now)`. `trackPerf` lowers the `quality` level when frame times are slow.

## Controls (keep these in sync with the HUD and help text)

- Keyboard: arrow keys or WASD to move, Space to fire, `X` or Shift for a bomb, `P` or Escape to pause.
- Mouse: move over the canvas to steer, hold the mouse button to fire.
- Touch: on-screen joystick (left zone), FIRE, and BOMB buttons. Touch controls appear only on touch-capable devices.

## Conventions

- Keep changes small and in the existing style: plain JavaScript, no frameworks, no transpilation, and 2-space indentation.
- Preserve mobile layout. The page uses `touch-action: none`, safe-area insets, and a fixed full-viewport canvas. Check phone widths after CSS changes.
- Respect reduced motion. Screen shake, flashes, and pulses should be scaled down or skipped when the `reducedMotion` flag is on (see `screenShake`, `flashHit`, and `flashBomb`).
- Keep accessibility hooks working: the `srLive` live region is used by `announce()`, and buttons need `title` text.
- Audio is generated with the Web Audio API through `beep` and `noiseBurst`. It starts only after a user gesture (`ensureAudio`), so do not create the audio context earlier.

## Git

- Develop on the branch assigned for the task. Do not push to `main` without explicit permission.
- Write clear commit messages that describe the gameplay or code change.

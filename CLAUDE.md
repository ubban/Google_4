# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a single HTML5 canvas.

Files:

- `index.html` (~1,670 lines): the entire game. Inline CSS in `<style>`, markup, and one inline `<script>` that holds all game logic.
- `README.md`: a one-line placeholder.
- No build system, package manager, test suite, or CI configuration exists.

## Running and verifying

- There is no build step. Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser, or with a headless browser such as Playwright, and check the console for errors.
- Test on both desktop (keyboard) and touch layouts when you change input, HUD, or layout code. Touch controls are shown only on touch-capable devices.

## Code structure (inside `index.html`)

The script is one IIFE-style block that starts around line 426 and runs the game loop with `requestAnimationFrame`. Key sections, roughly in file order:

- **DOM references**: `canvas`, `overlay`, HUD elements, touch controls, and buttons are looked up once at the top.
- **Persistence keys**: `localStorage` keys use the `skyfighter.*` prefix (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Always wrap reads and writes so the game still works when storage is unavailable.
- **Config tables**:
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
  - `THEMES`: six stage sky palettes. Stage N uses `THEMES[(N - 1) % 6]`.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (fast, weaving), `tank` (4 HP, unlocks at stage 2).
  - `BOSSES`: six bosses. Stage N's boss is `BOSSES[(N - 1) % 6]`, with its own colors and HP multiplier.
- **Audio**: a lazily created `AudioContext` (`ensureAudio`) plus helpers `beep`, `noiseBurst`, and a `sfx` table. Sound is muted by a persisted flag.
- **Perf scaling**: `trackPerf` samples frame times and sets a `quality` level (0 full, 1 medium, 2 low) to reduce effects when frame rate drops.
- **Layout**: a fixed logical canvas of `BASE_W` × `BASE_H` = 480 × 720, scaled by `resize()` to fit the window (capped at 1.6×).
- **Input**:
  - Keyboard: arrow keys or WASD to move, Space to fire, `X` / Shift to drop a bomb, `P` / Escape to pause. Keys are tracked in the `keys` object by `e.code`.
  - Touch: a virtual joystick (`joystickStart` / `joystickMove`), a fire button, and a bomb button.
  - Mouse and pointer: the ship follows the pointer over the canvas (`mouseTarget`).
- **HUD and feedback**: hearts and bombs are rendered as HTML, with flash, vignette, screen shake, combo popups, stage banners, and a boss health bar. An `aria-live` region (`srLive`, via `announce`) reports important events to screen readers.
- **Main loop**: updates state, spawns enemies and bosses, resolves collisions, and draws. It uses `requestAnimationFrame(loop)`.

## Conventions

- Keep the game in a single file unless the user asks for a split. The project is intentionally dependency-free.
- Do not add external libraries, CDN scripts, or build tooling without asking first.
- Keep the existing style: two-space indentation, single quotes in JavaScript, `const` by default, and short helper functions.
- Tune gameplay through the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding numbers in the loop.
- Respect the existing reduced-motion setting: effects that shake or flash should check `reducedMotion` before running.
- Keep the layout responsive. The page uses `overflow: hidden`, `touch-action: none`, and safe-area insets, so new UI must not introduce page scrolling.
- Keep accessibility hooks working: announce important state changes through `announce()`, and keep the `aria-live` region and button labels accurate.

## Git workflow

- Develop on the branch assigned for the session. Do not push to other branches without permission.
- Write clear, descriptive commit messages.
- Do not open a pull request unless the user explicitly asks for one.

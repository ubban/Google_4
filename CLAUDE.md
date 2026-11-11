# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser on an HTML5 canvas. The entire game is one self-contained file, `index.html` (about 1,670 lines): inline `<style>`, markup, and a single inline `<script>` IIFE. There is no build step, no bundler, no package manager, no test suite, and no external dependencies.

Repository layout:

- `index.html`: the whole game (markup, CSS, and JS).
- `README.md`: a one-line placeholder.
- `CLAUDE.md`: this file.

## Running and checking

- **Run it:** open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). Nothing needs to be installed.
- **Syntax check the game script** (no test runner exists):
  ```bash
  f=$(mktemp --suffix=.js) && awk '/<script>/{f=1;next}/<\/script>/{f=0}f' index.html > "$f" && node --check "$f"; rm -f "$f"
  ```
  The extracted script goes to a temp file outside the repo.
- **Verify behavior** by playing it in a browser, ideally at a phone-sized viewport as well as desktop. Changes to input, layout, or audio must be checked in a real browser; a passing syntax check does not prove the game works.

## Architecture of `index.html`

The inline script is one IIFE. Key pieces, roughly in file order:

- **Constants** (around lines 463–516):
  - `STORAGE_*` keys, all prefixed `skyfighter.` (best score, mute, difficulty, reduced motion).
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed, shot chance, starting lives and bombs, and a boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
  - `THEMES`: six sky palettes that cycle with stage number.
  - `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`): size, HP, speed, score, and the stage each first appears on.
  - `BOSSES` (six entries, `VANGUARD` through `TITAN`): colors and an HP multiplier.
- **Persistence:** `localStorage` only, wrapped in the constants above. The best score is written on game over.
- **Audio:** a lazily created `AudioContext` (`ensureAudio`) with synthesized `beep` / `noiseBurst` helpers and a named `sfx` table. Audio is created on the first user gesture.
- **Input:** keyboard (arrows / WASD to move, Space to fire, X / Shift to bomb, P / Esc to pause), mouse on the canvas, and touch (joystick in `moveZone`, plus `fireBtn` and `bombBtn`).
- **Rendering:** a single `<canvas id="gameCanvas">` at a base resolution of 480×720 (`BASE_W`, `BASE_H`), scaled to fit the window by `resize()` (capped at 1.6×). Drawing functions include `drawPlane`, `drawBossShip`, and `drawClouds`.
- **Game loop:** `update(dt)` then `draw()`, driven by `requestAnimationFrame` through `loop(now)`. `dt` is clamped to 50 ms. `trackPerf` lowers a `quality` level (0 full, 1 medium, 2 low) when frame times are poor.
- **Weapons and power-ups:** `fireWeapon()` implements `vulcan` and `laser`; `applyPowerUp()` handles pickups. Bosses use a set of pattern helpers (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) driven from `updateBoss`.
- **Lifecycle:** `resetGame`, `spawnBoss`, `killBoss`, `loseLife`, `endGame`, and `togglePause`.
- **Accessibility:** a live region (`srLive`) for announcements via `announce()`, a reduced-motion toggle that also respects `prefers-reduced-motion`, and screen-shake and vignette effects that the reduced-motion setting turns down.

## Conventions

- Keep the game in a single file unless the user asks for a restructure. Do not add a build system, package.json, or framework without an explicit request.
- Prefer small, local edits. Match the existing style: two-space indentation, `const`/`let`, semicolons, and plain DOM APIs.
- Tune gameplay through the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding new numbers in the update loop.
- Keep touch and keyboard controls working together. Any new action needs a keyboard binding and a touch affordance, or a clear reason it does not.
- Preserve the `prefers-reduced-motion` and mute behavior when adding effects or sounds.
- Anything rendered on the canvas must still fit a phone-width viewport, and the page must not scroll horizontally (`overflow: hidden` on `html, body` is intentional).
- Do not add external scripts, fonts, or network calls. The game is fully offline.

## Git

- Development branch for this work: `claude/gifted-tesla-hzfpi3`. Commit and push there.
- Do not open a pull request unless the user explicitly asks for one.

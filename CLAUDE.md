# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The whole game lives in a single file, `index.html` (~1,670 lines: inline `<style>`, markup, and one IIFE `<script>` that starts near line 422).

Other files:
- `README.md` — a one-line placeholder title. Keep it accurate if the project description changes.

There is no build step, package manager, test suite, or linter configured.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

No install is required.

## Architecture (inside `index.html`)

- **Canvas rendering.** The game draws to `#gameCanvas` at a base logical size of 480×720 (`BASE_W`, `BASE_H`). `resize()` scales the canvas to fit the viewport. Coordinates in game logic use the base size.
- **Configuration constants** near the top of the script:
  - `DIFFICULTIES` — `easy`, `normal`, `hard`. Each sets spawn rates, enemy speed and fire chance, starting `lives`, `bombs`, and `bossHpMul`.
  - `KILLS_PER_STAGE` — kills needed to advance a stage.
  - `THEMES` — six sky palettes, cycled per stage.
  - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP, appears from stage 2).
  - `BOSSES` — six stage bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN), each with its own palette and `hpMul`.
- **Persistence.** Uses `localStorage` with keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every access should stay wrapped so the game still works when storage is unavailable or throws.
- **Audio.** Synthesized with the Web Audio API (`ensureAudio`, `beep`, `noiseBurst`). Audio is created lazily on the first user gesture; do not start it on page load.
- **Input.**
  - Keyboard: arrow keys/WASD move, Space fires, `X` / Shift uses a smart bomb, `P` / Escape pauses. Space's default is prevented to stop page scrolling.
  - Mouse: canvas move and click handlers, via `canvasPointFromEvent`.
  - Touch: a virtual joystick on the left (`moveZone`), a fire button, and a bomb button.
- **HUD and UI.** Hearts (lives), bomb count, score, weapon label, and stage banners are DOM elements updated from game state. Motion can be reduced via a toggle.

## Conventions

- **Keep it a single file.** Do not introduce a bundler, framework, or external dependencies unless the user asks. Inline CSS and JS stay in `index.html`.
- **Plain ES2015+ JavaScript.** Match the existing style: `const`/`let`, arrow functions, the IIFE wrapper, and no globals beyond what is already there.
- **Mobile-first.** The page is designed for touch as well as desktop: `touch-action: none`, safe-area insets, and `viewport-fit=cover`. Test any input or layout change at phone width.
- **Respect the reduced-motion setting** when adding screen shake, flashes, or other motion effects.
- **Keep the tuning in the constant tables.** Balance changes (spawn rates, HP, scores, difficulty) should go in `DIFFICULTIES`, `ENEMY_TYPES`, or `BOSSES` rather than being hard-coded in logic.
- **Do not add noisy logging** to the game loop.

## Verifying changes

There are no automated tests. For changes to gameplay, rendering, or input:
1. Open `index.html` in a browser (or serve it locally) and check the console for errors.
2. Play through at least one stage, including a boss, and exercise the keyboard, mouse, and touch paths you touched.
3. Check the layout at a narrow (phone) viewport and a desktop viewport.

If you cannot run a browser, say so explicitly rather than claiming the change was verified.

## Git

- Development branch for automated work: `claude/gifted-tesla-xqeg2f`. Do not push to other branches without explicit permission.
- Commit messages should be short and imperative and describe the gameplay or code change.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, browser-based Raiden-style vertical shoot-'em-up. There is no build step, package manager, test suite, or server dependency.

Files:

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` (~1,670 lines). Canvas-based rendering and game loop, Web Audio sound effects, touch controls.
- `README.md` — minimal placeholder (title only).
- `CLAUDE.md` — this file.

## Running the game

No install needed. Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it. Verify changes by playing in a real browser, not only by reading the code.

## Code layout inside `index.html`

The script is organized in commented sections. Keep new code in the matching section:

- **Storage keys** (`skyfighter.*`): `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`. Persisted via `localStorage`. Reads and writes are not wrapped in try/catch today; keep that in mind if you add more.
- **Adaptive performance** (`trackPerf`, `quality` 0–2): averages frame time over 45 frames and lowers effects quality when the frame rate drops.
- **Tuning data** (top-level `const`s):
  - `DIFFICULTIES` — `easy` / `normal` / `hard` spawn rates, enemy speed and fire chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` — kills needed before a stage boss warning.
  - `THEMES` — six stage sky palettes, cycled by stage.
  - `ENEMY_TYPES` — `grunt`, `interceptor`, `tank` (with `minStage` gating).
  - `BOSSES` — six bosses (VANGUARD → TITAN), each with colors and `hpMul`.
- **Input**: `keys` map keyed by `e.code`. Move with arrows or `A`/`D`; hold `Space` to fire; `P`/`Escape` pause; `X` or either `Shift` uses a bomb. Touch uses `#touchControls` (joystick zone, FIRE, BOMB).
- **Game loop**: `loop` → `update` (state changes, spawning, collisions, bosses) → `draw` (rendering, HUD). Weapon patterns are `fire*` functions dispatched from `fireWeapon`; power-ups are applied in `applyPowerUp`.
- **HUD and UI**: DOM elements such as `#hud`, `#bossBarWrap`, `#stageBanner`, `#comboPop`, `#overlay`, and the top-right icon buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`). `announce()` writes to the `#srLive` region for screen readers.
- **Audio**: `ensureAudio` lazily creates the `AudioContext` on first user input. Sound effects are procedural (`beep`, `noiseBurst`) with stereo panning via `panFor`. Respect `muted`.

## Conventions

- Keep the game as a single self-contained `index.html` unless the user asks for a split. Do not add a bundler, framework, or npm dependencies without a clear reason.
- No external network resources. The current page is fully self-contained; keep it that way.
- Preserve the existing style: 2-space indentation, single quotes in JS, `const`/`let` with small helper functions, and section comments using `// ---------- Name ----------`.
- Keep the game playable on phones: the layout uses `touch-action: none`, safe-area insets, and `resize()`. Check any new UI at phone width.
- Respect the reduced-motion option (`reducedMotion`) for any new screen shake, flashes, or pulses.
- Don't break the difficulty tuning: changing a `DIFFICULTIES` value changes game balance, so call it out in the summary.
- Accessibility: announce important state changes through `announce()`, and give new buttons a `title` or label.

## Making changes

- Read the relevant section of `index.html` before editing; the file is large, so search with `grep -n` for function names.
- After editing, open the game in a browser (or a headless Chromium) and play through at least one stage to confirm the change; for changes to visuals or HUD, take a screenshot.
- There is no linter or test runner configured. If you add one, document the command here.
- Keep commits small and focused. Develop on the branch assigned for the session; do not push to other branches without permission.

## Things to avoid

- Don't commit generated files, screenshots, or `node_modules`.
- Don't add analytics, tracking, or remote calls.
- Don't rename `skyfighter.*` storage keys without a migration, since that would reset players' saved best scores and settings.

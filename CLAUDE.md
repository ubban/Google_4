# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game is a single self-contained file, `index.html` (~1,670 lines: inline `<style>` and one inline `<script>`, roughly lines 422–1667). `README.md` is a placeholder.

There is no build step, no package manager, no test suite, and no linter configured. To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Repository layout

- `index.html` — the whole game: markup, CSS, and JavaScript.
- `README.md` — minimal project title only.
- `CLAUDE.md` — this file.

## Architecture of `index.html`

Everything in the script runs inside a single IIFE-style block (`<script>` at ~line 422). Key sections, roughly in file order:

- **DOM references** (~426–461): cached `getElementById` lookups for the canvas, HUD, overlays, touch controls, and buttons.
- **Persistence keys** (~463–466): `localStorage` keys prefixed `skyfighter.` — `bestScore`, `muted`, `difficulty`, `reducedMotion`. Every `localStorage` access must stay wrapped so the game still works if storage is unavailable.
- **Adaptive quality** (~474–483): `trackPerf` samples frame times and steps `quality` (0 full, 1 medium, 2 low) to reduce effects on slow devices.
- **Tuning tables** (~485–515): `DIFFICULTIES` (spawn rates, enemy speed/fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes; one per stage, cycling), `ENEMY_TYPES` (grunt, interceptor, tank with `minStage` gating), and `BOSSES` (six designs, cycled by stage).
- **Audio** (~543–680): lazily created `AudioContext` (`ensureAudio`), synthesized sound via `beep` and `noiseBurst`, and a `sfx` map of named effects. Audio is created only after a user gesture.
- **HUD and feedback** (~680–750): hearts/bombs rendering, hit and bomb flashes, combo pop, stage banner, screen shake, mute and difficulty toggles.
- **Layout** (~753–763): `BASE_W = 480`, `BASE_H = 720`; `resize()` scales the canvas to fit the viewport, capped at 1.6×.
- **Input** (~768–862): keyboard state in `keys` (keyed by `e.code`), a virtual joystick and fire/bomb buttons for touch, and mouse aiming/firing on the canvas.
- **Game state** (~864–912): module-level `let` variables (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`, `chain`, `multiplier`, etc.). `resetGame()` reinitialises them.
- **Rendering** (~913–1068, `draw` ~1520): plane and boss drawing, enemy spawning, power-up drops.
- **Simulation** (~1069–1500): `update(dt)` is the per-frame logic; weapon firing (`fireWeapon`), bomb use, boss patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`), `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
- **Main loop** (`loop(now)` ~1640): `requestAnimationFrame`-driven; computes `dt`, updates, draws.

### Controls (keep these in sync with the README/overlay text if you change them)

- Move: arrow keys or WASD; mouse movement over the canvas steers; touch joystick (drag left side).
- Fire: hold Space, hold the mouse button, or hold the touch FIRE button.
- Bomb: X or either Shift; touch bomb button.
- Pause: P or Escape.

## Conventions

- **Keep it a single file.** The project is intentionally zero-dependency and opens directly from disk. Do not add a bundler, framework, or external script/CSS dependency unless the user explicitly asks for one.
- **Match the existing style.** Two-space indentation inside the script and style blocks, `const`/`let` declarations, camelCase identifiers, and `SCREAMING_CASE` for constant tables.
- **Tune through the tables.** Balance changes (enemy stats, boss HP, difficulty numbers, stage progression) belong in the `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES` constants rather than being hard-coded in logic.
- **Respect the accessibility toggles.** Reduced-motion and mute are user-facing settings persisted in `localStorage`. New visual effects that involve shake, flashing, or rapid motion should honour `reducedMotion`, and new sounds should route through `sfx`/`masterGain` so mute applies.
- **Be mindful of mobile.** The layout uses `viewport-fit=cover`, safe-area insets, and touch controls. Verify changes to HUD or controls at phone widths.
- **Use `dt`-based movement.** Per-frame motion in `update` is scaled by the frame delta, so keep new movement consistent with that.

## Verifying changes

There are no automated tests. For any gameplay or UI change:

1. Open `index.html` in a browser (or run a simple static server such as `python3 -m http.server` from the repo root).
2. Play through at least one stage transition and a boss fight on the affected difficulty.
3. Check the HUD at a narrow (phone-sized) viewport and confirm touch controls still respond.
4. Confirm pause, mute, and the reduced-motion toggle still behave.
5. Check the browser console for errors.

If you add a non-trivial piece of logic, consider whether it is best verified with a quick scripted browser run rather than by reading alone.

## Git and workflow

- Develop on the branch specified by the session; do not push to other branches without explicit permission.
- Write clear, descriptive commit messages that explain *why* a change was made.
- Do not open pull requests unless the user explicitly asks for one.
- Do not commit generated artifacts or local scratch files to the repository.

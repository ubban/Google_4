# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up written as a single self-contained HTML5 canvas page.

- `index.html` (~1670 lines): all markup, CSS (in `<style>`), and JavaScript (in one inline `<script>` wrapped in an IIFE). No external assets, libraries, or network calls.
- `README.md`: a one-line placeholder title.

There is no build step, package manager, bundler, test suite, linter, or CI configuration. Do not invent one without being asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by loading the page and playing: check the title/overlay, a full game loop, pause, bombs, weapon power-ups, boss fights, game over, and the high-score reload.
- For mobile-related changes, check touch controls (joystick, fire, bomb) and the portrait/landscape resize behaviour. The canvas scales to fit the viewport.

## Architecture of `index.html`

Everything in the game lives inside one IIFE. Key sections, roughly in file order:

1. **Markup and CSS** (top of file): HUD pills, overlay card, touch controls, vignette/flash layers. Colors and layout are hard-coded; there are no CSS variables.
2. **DOM references**: `const xxxEl = document.getElementById(...)` block near the start of the script. Add new elements here.
3. **Storage keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all `skyfighter.*`). Persisted via `localStorage`.
4. **Tunables**:
   - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
   - `THEMES`: sky gradients per stage; cycles with stage number.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP, unlocked at stage 2). `minStage` gates spawning.
   - `BOSSES`: six designs cycled by `(stage - 1) % BOSSES.length`. HP is `70 + stage * 35`, scaled by `hpMul` and difficulty.
5. **Audio**: a lazily created `AudioContext` (`ensureAudio`) with `beep`, `noiseBurst`, and a `sfx` table. Audio starts only after a user gesture.
6. **Adaptive quality**: `trackPerf` moves `quality` between 0 (full), 1 (medium), and 2 (low) based on frame time. Effects such as `shadowBlur`, particle counts, and glow check `quality`. Keep new expensive effects behind this.
7. **Input**: keyboard (`keys` map keyed by `e.code`), mouse/pointer aim, and touch (`joystick*`, `fireBtn`, `bombBtn`). Keys: arrows/WASD to move, Space to fire, `X` or Shift for a bomb, `P` or Escape to pause.
8. **Game state and loop**: `resetGame`, `update(dt)`, and the draw functions (`drawPlane`, `drawBossShip`, and others), driven by `requestAnimationFrame(loop)`. `update` uses a `dt` value; keep gameplay timing frame-rate independent.
9. **Weapons and power-ups**: `fireWeapon` handles Vulcan (spread at levels 1–3) and Laser (lanes at level 2+). Power-up types are `life`, `bomb`, and `weapon`, applied in the pickup handler.
10. **Overlay and persistence**: the start/game-over overlay shows score, best, and stage. The best score is written to `localStorage` on a new record.

Accessibility features to preserve when touching related code:
- `prefers-reduced-motion` is detected at load and can be toggled from the HUD (`motionBtn`). Reduced motion disables screen shake and the hit/bomb screen flashes (`flashHit`, `flashBomb`, `screenShake`).
- A live region (`srLive`) is used via `announce()` to speak important state changes.
- Buttons carry `title` attributes. Keep these when adding controls.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. Adding files means also deciding how they are served.
- Match the existing style: 2-space indentation, `const`/`let`, single quotes in JavaScript, no frameworks, no transpilation.
- Do not add external CDN scripts, fonts, or network requests. The page is designed to work offline from a single file.
- Keep the page usable on phone-sized viewports and do not break the `touch-action: none` / `user-select: none` setup on the game body.
- When tuning balance, change the values in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `KILLS_PER_STAGE` rather than hard-coding numbers in the update logic.
- Be careful with `localStorage` access: wrap new reads/writes in `try`/`catch` if they might run in restricted browser contexts.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-0anny7`). Do not push to other branches without explicit permission.
- Do not create pull requests unless the user asks for one.
- Commit messages should describe the gameplay or code change in plain words. Keep the subject line short and imperative.

## Notes for future changes

- The project is a single-file game; prefer small, targeted edits to `index.html` over restructuring.
- If you add a new file or build step, update this document with the commands needed to run and verify it.

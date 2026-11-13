# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A single-file browser game, **Sky Fighter** — a Raiden-style vertical shoot-'em-up. There is no build step, no package manager, and no test suite.

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` (~1,670 lines).
- `README.md` — stub; currently only a title line.

Git: the default working branch is `claude/gifted-tesla-dzfpkk`. History is small (`Initial commit`, then `Add Sky Fighter ...`).

## Running it

Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`). Nothing needs to be installed.

## Code layout inside `index.html`

The script is plain JavaScript with no modules, no framework, and no external dependencies. Rough order of sections, by the function/constant names they define:

- **Storage keys / prefs** (`STORAGE_*`, `bestScore`, `muted`, `difficulty`, `reducedMotion`) — persisted via `localStorage` under the `skyfighter.*` namespace.
- **Perf scaling** (`quality`, `trackPerf`) — quality levels 0 (full) / 1 (medium) / 2 (low) adjusted from frame timing.
- **Tuning tables** — `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `THEMES`, `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES`, `KILLS_PER_STAGE`.
- **Audio** — lazily created `AudioContext` (`ensureAudio`), with `beep` and `noiseBurst` helpers. Sound is synthesized; there are no audio files.
- **HUD / feedback** — hearts, bombs, combo pop, stage banner, screen shake, hit/bomb flashes, `announce()` for screen readers.
- **Canvas & input** — fixed logical resolution `BASE_W × BASE_H` (480 × 720) scaled by `resize()`. Input comes from keyboard, a touch joystick + fire/bomb buttons, and mouse (`canvasPointFromEvent`).
- **Game state** — top-level `let`s (`score`, `lives`, `bombCount`, `stage`, `chain`, `multiplier`, `player`, …) reset by `resetGame()`.
- **Drawing** — `drawPlane`, `drawBossShip`, and related canvas routines.
- **Spawning / combat** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`, `fireWeapon`, `useBomb`, boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, `fireHomingSeeded`, …).
- **Main loop** — `update(dt)` handles movement, collisions, and scoring; the render/loop function drives per-frame drawing.

When changing behavior, look for the tuning table or the named function first rather than adding magic numbers inline.

## Controls (for reference when testing)

- Move: arrow keys or `WASD`
- Fire: `Space` (hold); mouse/touch on the canvas also aims and fires
- Bomb: `X` or `Shift`
- Pause: `P` or `Esc`
- Touch: left-side joystick, on-screen fire and bomb buttons

## Conventions

- Keep it a single self-contained file unless the user asks for a split. Don't introduce a build step or dependencies for small changes.
- Match the existing style: 2-space indent, `const`/`let`, single quotes, and small named helper functions.
- Keep the game playable at phone width — the viewport meta disables zoom and the canvas is `max-width/max-height: 100%`. Preserve `touch-action: none` on the canvas and controls.
- Respect the `reducedMotion` preference when adding new screen-shake, flashes, or particle effects.
- Any new persisted setting should use the `skyfighter.*` key prefix and be read inside a guard in case `localStorage` is unavailable.
- The current build has no external requests or CDN scripts. Keep it that way unless the user asks otherwise.

## Verification

There are no automated tests or linters configured. For changes to gameplay or UI:

1. Open `index.html` in a browser and play through at least one stage boss.
2. Check all three difficulties if you touched `DIFFICULTIES` or spawn/boss logic.
3. Check touch controls in a mobile-sized viewport if you touched input or layout.
4. Confirm there are no console errors on load or during play.

A headless Chromium/Playwright run is a reasonable way to smoke-test the page if one is needed.

## Git

- Develop on `claude/gifted-tesla-dzfpkk` and push there with `git push -u origin <branch>`.
- Write commit messages in the imperative mood, with a short summary line describing the gameplay or code change.

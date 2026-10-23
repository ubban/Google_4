# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only app is **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: markup, CSS (`<style>`, starts ~line 10), and JavaScript (`<script>`, starts ~line 422). Roughly 1,670 lines.
- `README.md` — placeholder title only.

There is no build step, bundler, package manifest, test suite, or linter configured. Nothing needs installing.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser: start a run, exercise the change, and check the console for errors.
- Test the touch controls as well as the keyboard and mouse controls when touching HUD, overlay, or input code. The touch joystick and FIRE/BOMB buttons live in `#touchControls`.

## Code structure inside `index.html`

The script is one IIFE (`(function () { 'use strict'; ... })();`). Keep new code inside it rather than adding globals.

Major sections, in file order:

1. **DOM references** — `const xEl = document.getElementById(...)` block at the top.
2. **Persistence keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`, all under the `skyfighter.*` prefix.
3. **Adaptive performance** — `trackPerf` moves a `quality` level (0–2) up or down based on a rolling frame-time average.
4. **Config tables** — `DIFFICULTIES` (easy/normal/hard), `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage), `ENEMY_TYPES`, `BOSSES`.
5. **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and an `sfx` object. Audio is created lazily on the first user gesture.
6. **HUD and effects** — `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`.
7. **Input** — keyboard (`keys` map), touch joystick and buttons, and mouse steering on the canvas.
8. **Game state and actions** — `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `addScore`, `breakChain`, `useBomb`, `fireWeapon`, `applyPowerUp`, `loseLife`, `endGame`, `togglePause`.
9. **Boss patterns** — `BOSS_PATTERNS` and the `fire*` helpers (radial, spread, columns, spiral, cross, homing, double ring).
10. **Update and draw** — `update(dt)` advances the simulation; `draw()` renders it; `drawPlane`, `drawBossShip`, `drawClouds` are the sprite and background helpers.
11. **Main loop** — `loop(now)` clamps `dt` to 50 ms, then calls `update` and `draw` only when running and not paused. The loop starts from the Start button handler at the bottom.

The game is driven by `dt` (seconds). Keep new movement and timers frame-rate independent by scaling with `dt`.

## Conventions

- **Style**: 2-space indent, single quotes, semicolons, `const` by default and `let` only for reassigned state. Match the existing section banners (`// ---------- Name ----------`) when adding a new section.
- **No new dependencies or external assets.** The game is intentionally one file that works offline. Audio is synthesized with WebAudio, not loaded from files.
- **Persisted settings** go through `localStorage` with a `skyfighter.` key, and every read and write should tolerate the value being absent or malformed. The existing reads use `Number(...) || 0` and `=== '1'` patterns.
- **Accessibility**: announce important state changes through `announce()` (the `srLive` region). Respect `reducedMotion`: skip or scale down screen shake, flashes, and pulses when it is on. The initial value comes from `prefers-reduced-motion` unless the user has set it.
- **Performance**: the game allocates per-frame objects in several places, so keep hot-path code (`update`, `draw`, particle and bullet loops) allocation-light. Check `quality` before adding expensive effects.
- Keep changes scoped to the task. Don't refactor unrelated parts of the file.

## Gameplay balance knobs

Most tuning lives in data tables, so prefer editing those over changing logic:

- `DIFFICULTIES` — spawn rate, enemy speed, enemy fire chance, starting lives and bombs, boss HP multiplier.
- `ENEMY_TYPES` — size, HP, speed multiplier, score, and the `minStage` each type first appears on.
- `BOSSES` — name, colors, and `hpMul` per boss.
- `KILLS_PER_STAGE` — kills needed to advance a stage.

## Git and branches

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-pyi680`). Don't push to `main` without explicit permission.
- Write commit messages in the imperative mood with a short summary line, followed by a body explaining what changed and why when it isn't obvious from the diff.
- Don't commit generated files or editor state; the repo currently has none.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a single HTML5 canvas.

| File | Purpose |
| --- | --- |
| `index.html` | The whole game: markup, CSS (`<style>`, ~lines 10–362), canvas and HUD elements, and all JavaScript (one IIFE in `<script>`, ~lines 422–1667). |
| `README.md` | Placeholder readme. |

There is no build step, no package manager, no bundler, no test suite, and no linter configuration. Do not add tooling unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). Nothing needs to be installed.

## Architecture of `index.html`

The JavaScript is a single `(function () { 'use strict'; ... })();` block. Everything is closure-scoped; nothing is exported or attached to `window`. Keep new code inside that block unless there is a clear reason not to.

Major sections, in order of appearance:

1. **DOM references**: `const x = document.getElementById(...)` for every HUD, overlay, and touch control element. Element IDs in the markup must match these lookups.
2. **Storage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Persisted via `localStorage`.
3. **Adaptive quality**: `trackPerf(dt)` samples the last 45 frame times and steps `quality` (0 full, 1 medium, 2 low) to keep frame time between ~1/56 s and ~1/40 s.
4. **Tuning data**, all declared as plain constants near the top:
   - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rate, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` (18): kills needed before a stage advances.
   - `THEMES`: six sky palettes that cycle with stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP, minimum stage 2).
   - `BOSSES`: six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with colors and HP multipliers. Their attack patterns are in `BOSS_PATTERNS`.
5. **Audio**: a lazily created `AudioContext` (`ensureAudio`), with `beep()` and `noiseBurst()` primitives and a named `sfx` table. Sound is muted by the mute button and by `muted`.
6. **Input**: keyboard (`keys` map keyed by `e.code`), touch joystick plus fire and bomb buttons, and mouse on the canvas. Touch controls are shown only when `isTouchDevice` is true.
7. **Game state and logic**: `resetGame()`, `update(dt)` (movement, firing, collisions, spawning, power-up pickup), `spawnEnemy`, `spawnBoss`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
8. **Rendering**: `draw()` plus helpers `drawPlane`, `drawBossShip`, `drawClouds`. Everything is drawn with Canvas 2D on a fixed `480 × 720` logical canvas that `resize()` scales to fit the window (capped at 1.6×).
9. **Main loop**: `loop(now)` clamps `dt` to 50 ms, updates and draws only when `running && !paused`, and calls `requestAnimationFrame`.

Conventions visible in the code:

- Game objects are plain objects in arrays (enemies, bullets, power-ups, explosions). Entities use `x`, `y`, `w`, `h` and collide with `rectsOverlap`.
- Time-based motion uses `dt` in seconds. Keep new motion frame-rate independent.
- Power-ups are `life`, `bomb`, and `weapon`. Weapons are `vulcan` and `laser`, toggled by the weapon power-up (`applyPowerUp`).
- Accessibility hooks exist and should be kept working: `announce()` writes to a live region (`srLive`). The reduced-motion setting defaults from `prefers-reduced-motion` and can be toggled. When it is on, `screenShake` is scaled down and capped, and `flashHit` / `flashBomb` return early.

## Conventions for changes

- **Keep it a single file** unless the user asks for a split. The project is intentionally dependency-free.
- **Tuning lives in the constant tables.** Prefer changing `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, or `THEMES` over hard-coding numbers in logic.
- **Persist only through the existing `STORAGE_*` keys** so behaviour stays consistent. Wrap new `localStorage` access in try/catch, because it can throw in private or restricted contexts.
- **Respect the 480×720 logical resolution.** Positions and sizes are in that space, not in CSS pixels.
- **Do not add external network dependencies** (CDN scripts, fonts, analytics) without asking.
- Match the existing style: 2-space indent, single quotes in JS, `const` by default, short section comments of the form `// ---------- Name ----------`.

## Verifying changes

There is no automated test. After editing `index.html`:

1. Open it in a browser (or serve it locally) and confirm the console has no errors.
2. Play through start, shooting, a bomb, a power-up pickup, a stage transition, a boss fight, game over, and restart.
3. Check pause (`P` / `Esc`), mute, difficulty switching, and the reduced-motion toggle.
4. For mobile-facing changes, check the touch joystick and fire/bomb buttons at a phone-sized viewport.

For a quick syntax check of the inline script, you can extract the `<script>` block and run `node --check` on it.

## Git

- Development branch for this work: `claude/gifted-tesla-ls7ed4`. Do not push to other branches without permission.
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Do not open a pull request unless the user asks for one.

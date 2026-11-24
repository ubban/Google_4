# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game lives in a single file, `index.html` (~1,670 lines): inline CSS in `<style>`, markup, and one IIFE-wrapped inline `<script>`. `README.md` is a placeholder with no setup instructions.

There is no build step, no package manager, no test suite, and no linter configured. To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Repository layout

```
index.html   # The whole game: CSS, HTML, and JS
README.md    # Placeholder title only
CLAUDE.md    # This file
```

## Code structure of `index.html`

- **Lines ~10–362:** inline `<style>`. Layout is a fixed full-viewport flex container with a `#gameCanvas` and an absolutely positioned `#hud`. Touch controls (joystick zone, FIRE and BOMB buttons) are styled here too.
- **Lines ~363–421:** markup for the canvas, HUD, overlays (start/pause/game over), and mobile controls.
- **Lines ~422–1669:** one `<script>` block wrapped in `(function () { ... })();`. Keep new code inside that IIFE so nothing leaks onto `window`.

Key sections inside the script, in rough order:

| Area | Notes |
| --- | --- |
| Constants (`STORAGE_*`, `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BASE_W/BASE_H = 480×720`) | Tuning knobs live here. Prefer editing these over hard-coding numbers in logic. |
| Audio (`ensureAudio`, `beep`, `noiseBurst`) | Web Audio synthesized sound. No audio files. Audio is only started after a user gesture via `ensureAudio()`. |
| HUD helpers (`renderHearts`, `renderBombs`, `showStageBanner`, `showComboPop`, `screenShake`) | DOM/CSS-driven feedback. |
| Resize and input (`resize`, keyboard listeners, joystick, `canvasPointFromEvent`) | Canvas is scaled to fit while keeping the 480×720 logical resolution. |
| Game state (`resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`) | Per-frame simulation with delta time `dt`. |
| Weapons and bombs (`fireWeapon`, `useBomb`, `applyPowerUp`) | |
| Boss patterns (`BOSS_PATTERNS`, `updateBoss`, `killBoss`) | Each boss cycles through bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring). |
| Rendering (`draw`, `drawPlane`, `drawBossShip`, `drawClouds`) | Canvas 2D only. |
| Loop (`loop(now)`) | `requestAnimationFrame` loop; calls `trackPerf(dt)` for adaptive performance. |

## Game rules and tuning

- **Stages:** 6 `THEMES` (sky palettes), each ending in a boss from `BOSSES` (6 bosses). A stage advances after `KILLS_PER_STAGE` (18) kills.
- **Enemies:** `grunt`, `interceptor` (weaving), `tank` (4 HP, appears from stage 2).
- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn rate, enemy speed, shoot chance, starting lives, starting bombs, and a boss HP multiplier.
- **Persistence:** `localStorage` keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Reads are not wrapped in try/catch today; if you add more storage use, guard it, since some browsers throw in private mode or when storage is blocked.
- **Reduced motion:** defaults to the `prefers-reduced-motion` media query and can be toggled in the UI. Respect the `reducedMotion` flag for any new screen shake, flashes, or particle effects.

## Controls

- **Keyboard:** Arrows or WASD to move; Space to fire (hold for continuous fire); X or Shift to drop a bomb; P or Escape to pause.
- **Mouse:** move to steer; click to fire.
- **Touch:** drag in the left zone to fly (virtual joystick); tap FIRE; tap BOMB.

Input state is tracked in a `keys` map keyed by `KeyboardEvent.code`, plus a `firing` flag for the touch and mouse fire controls. Pressing Space calls `preventDefault()` to stop page scrolling.

## Working conventions

- **Keep it one file.** The project is deliberately a single self-contained HTML file with no dependencies. Do not add a bundler, package.json, or external CDN scripts unless the user asks for it.
- **Match the existing style.** Two-space indentation inside the script and style blocks, single quotes in JS, `const`/`let` (no `var`), and short helper functions named in camelCase. Constants are UPPER_SNAKE_CASE.
- **Frame-rate independence.** Movement, timers, and cooldowns in `update(dt)` should use `dt`, not frame counts.
- **Canvas coordinates.** Game logic uses the logical `BASE_W × BASE_H` space. Don't use raw CSS pixels for gameplay positions.
- **Mobile first.** The layout uses `viewport-fit=cover` and safe-area insets, and `touch-action: none` is set on the body and canvas. Keep new UI usable at phone widths and don't break the touch controls.
- **Audio gating.** Anything that plays sound must call `ensureAudio()` first, or it will be blocked by browser autoplay rules.

## Verifying changes

There are no automated tests. After editing:

1. Open `index.html` in a browser and check the console for errors on load.
2. Start a game, then exercise movement, firing, a bomb, pausing, and game over.
3. Check at least one touch-style interaction if you changed input or layout (browser devtools device mode is enough).
4. Confirm the HUD still fits at narrow widths and that the best score persists after a reload.

Keep the JavaScript syntactically valid as a whole, since there's no build step to catch errors. A quick `node --check` on the extracted script works if you need one.

## Git

- Development branch for automated work: `claude/gifted-tesla-ddlef8`.
- Commit messages: short imperative summary line, then a body if the change needs context.
- Do not open pull requests unless the user asks for one.

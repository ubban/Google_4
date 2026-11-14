# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free web project. Its only app is **Sky Fighter**, a Raiden-style vertical shoot-'em-up written as a single HTML5 canvas page.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS (`<style>`, ~lines 10–361), and JavaScript (`<script>`, ~line 422 onward). |
| `README.md` | Minimal project readme. |

There is no build step, package manager, bundler, test suite, linter, or CI configuration. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

Verify changes by loading the page in a real browser and playing; there are no automated tests to run.

## Code structure of `index.html`

- **Single IIFE.** All game logic lives inside one `(function () { 'use strict'; ... })();` block in the inline script. Keep new code inside it; do not leak globals.
- **Rendering.** One `<canvas id="gameCanvas">` drawn with the 2D context. The logical play area is `BASE_W = 480`, `BASE_H = 720` (see `resize()`), scaled to fit the viewport.
- **Main loop.** `loop(now)` is driven by `requestAnimationFrame`, clamps `dt` to 0.05 s, calls `trackPerf(dt)`, then `update(dt)` and `draw()` while the game is running and not paused.
- **Adaptive quality.** `quality` (0 full, 1 medium, 2 low) is stepped up or down by `trackPerf` based on average frame time. Respect it when adding expensive effects.
- **Config tables** near the top of the script, which are the usual place to tune balance:
  - `DIFFICULTIES` — spawn rate, enemy speed, enemy fire chance, starting lives and bombs, boss HP multiplier (`easy` / `normal` / `hard`).
  - `KILLS_PER_STAGE` — kills needed before a boss stage.
  - `THEMES` — per-stage sky palettes, cloud and sun tints.
  - `ENEMY_TYPES` — `grunt`, `interceptor`, `tank` (size, HP, speed, score, `minStage`).
  - `BOSSES` — boss names, colours, and HP multipliers. `BOSS_PATTERNS` holds their bullet patterns.
- **Entities.** Player, enemies, bullets, enemy bullets, power-ups, and explosions are plain arrays of objects updated in `update(dt)` and drawn in `draw()`. Collision uses `rectsOverlap`.
- **Persistence.** `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every access must be wrapped so the game still works when storage is unavailable (private mode, blocked site data).
- **Audio.** Synthesised with the Web Audio API (`ensureAudio`, `beep`, `noiseBurst`). No audio files. `ensureAudio()` must be called from a user gesture.
- **HUD and overlay DOM.** Score, lives (hearts), bombs, weapon, boss bar, combo pop, stage banner, and the start/pause overlay are DOM elements updated from the script (`renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, and so on).

## Controls (keep the README/overlay in sync if these change)

- **Keyboard:** arrow keys or WASD to move, Space to fire, `X` / Shift to drop a smart bomb, `P` / Escape to pause.
- **Touch:** on-screen joystick (`#moveZone`), FIRE and BOMB buttons (`#touchControls`).
- **Mouse:** moving over the canvas steers the ship toward the cursor; holding the button fires (`mousemove` / `mousedown` handlers using `canvasPointFromEvent`).
- Top bar buttons: mute (`#muteBtn`), reduced motion (`#motionBtn`), pause (`#pauseBtn`).

## Conventions

- Keep the project as a single self-contained HTML file unless the user asks for a split. Do not introduce frameworks or npm dependencies.
- Use plain ES2015+ JavaScript (`const`/`let`, arrow functions, template strings) to match the existing style. Two-space indentation.
- Respect accessibility features already in place: the `#srLive` live region (`announce()`), the `reducedMotion` setting (gate screen shake, flashes, and pulses behind it), and `prefers-reduced-motion`.
- Keep the UI mobile-friendly. The page uses `viewport-fit=cover` and touch controls; check layout at phone width after CSS changes.
- Preserve existing balance constants unless the task is a balance change. Mention any tuning change explicitly in the summary.

## Git workflow

- Default working branch for this repo's automated sessions: `claude/gifted-tesla-0u4010`. Push there with `git push -u origin <branch>`.
- Commit messages should be short, imperative, and describe the gameplay or code change.
- Do not create pull requests unless explicitly asked.

## Working on this repo

- Read the relevant section of `index.html` before editing; the file is large (~1,670 lines), so prefer targeted searches over reading the whole file.
- After any gameplay or rendering change, load the page and confirm that the game starts, runs, pauses, and ends without console errors.
- Do not commit generated files, screenshots, or scratch scripts to the repository. Use the session scratchpad for those.

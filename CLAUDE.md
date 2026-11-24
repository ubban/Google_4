# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small, dependency-free static web project. Its only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a single HTML file.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS (`<style>`, ~lines 10–361), and JavaScript (`<script>`, ~lines 422–1668). |
| `README.md` | Placeholder project readme (title only). |

There is no build step, no package manager, no test suite, and no linter configuration. Do not add one unless asked.

## Running it

Open `index.html` directly in a browser, or serve the folder with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Architecture of `index.html`

- **Layout:** `#container` holds a `<canvas id="gameCanvas">` plus DOM overlays: `#hud` (score, best, multiplier, weapon, hearts, bombs), `#bossBarWrap`, `#comboPop`, `#stageBanner`, `#topBar` (pause/mute/motion buttons), `#touchControls` (joystick and fire/bomb buttons), and `#overlay` (start/pause/game-over card with difficulty buttons).
- **Single IIFE:** all script code lives inside `(function () { 'use strict'; ... })();`. Nothing is exported to `window`. Keep new code inside that closure.
- **Fixed logical resolution:** `BASE_W = 480`, `BASE_H = 720`. The canvas is scaled to fit the viewport in `resize()`; game coordinates always use the base size.
- **Main loop:** `requestAnimationFrame(loop)` → `trackPerf(dt)` → `update(dt)` → `draw()`. `dt` is clamped to 0.05 s. Updates only run while `running && !paused`.
- **Adaptive quality:** `trackPerf` watches a 45-frame average and steps `quality` between 0 (full), 1 (medium), and 2 (low) to keep frame time under ~40 fps. Drawing code should respect `quality` when adding effects.
- **Audio:** generated with the Web Audio API (`ensureAudio`, `beep`, `noiseBurst`, and a `sfx` object). No audio files are loaded. Audio is created lazily on the first user gesture (`ensureAudio()` is called from start, fire, and bomb input). Respect the `muted` flag.
- **Accessibility:** `announce(text)` writes to the `#srLive` ARIA live region. Use it for important state changes. `reducedMotion` (user toggle, defaulting to `prefers-reduced-motion`) should suppress or soften screen shake and flashes.

## Game data (tunable constants)

These are the main knobs. Prefer changing these constants over hard-coding new numbers in logic.

- `DIFFICULTIES` — `easy` / `normal` / `hard`: spawn timing, enemy speed, enemy shoot chance, starting `lives`, starting `bombs`, and `bossHpMul`.
- `KILLS_PER_STAGE = 18` — kills needed before a stage boss warning and fight.
- `THEMES` — six background palettes (sky gradient, cloud and sun tints, name), cycled by stage.
- `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), and `tank` (4 HP, unlocked at stage 2), with size, speed, color, and score value.
- `BOSSES` — six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with colors and `hpMul`.
- `BOSS_PATTERNS` — attack patterns (radial, aimed spread, columns, spiral, cross, homing, double ring) indexed by boss design.
- Power-ups (`maybeSpawnPowerUp`): `life`, `bomb`, and `weapon`. Weapon power-ups raise the level from 1 to 3, then switch between `vulcan` and `laser` and reset to level 1.

## Persistence

`localStorage` keys, all prefixed `skyfighter.`:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Every `localStorage` access in the game is a plain call, so if you add new reads or writes, consider wrapping them in `try/catch`, since storage can be blocked in some browser contexts.

## Controls

- **Keyboard:** arrow keys or WASD to move; Space to fire (held); `X` or Shift to bomb; `P` or Escape to pause.
- **Mouse:** mouse-move steers the ship; mouse-down fires.
- **Touch:** virtual joystick on the left (`#moveZone`), fire and bomb buttons on the right (`#actionZone`).

## Conventions for changes

- Keep everything in `index.html` unless there is a clear reason to split files. If you split, update this section.
- Match the existing style: 2-space indent, `const` by default, arrow functions for callbacks, and small named helper functions.
- Use the existing DOM references at the top of the IIFE rather than calling `getElementById` repeatedly.
- Do not add external dependencies, CDN scripts, or frameworks. The game is intentionally self-contained.
- Keep the layout usable at phone width. The page uses `viewport-fit=cover` and `user-scalable=no`, and touch controls must remain reachable.

## Verifying changes

There are no automated tests. For any gameplay or UI change:

1. Open the page in a browser (or serve it locally) and check the browser console for errors.
2. Play through a stage transition and at least one boss fight, since those paths touch most of the state.
3. Check pause, mute, reduced-motion, and difficulty toggles, and confirm that the best score persists after a reload.
4. Check the layout at phone width and with touch controls.

## Git

- Development branch for this work: `claude/gifted-tesla-xteva0`.
- Commit messages in this repo are short, imperative summaries (see `git log`, e.g. "Add Sky Fighter: Raiden-style vertical shoot-em-up game").

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-em-up that runs in the browser on a single HTML canvas.

- `index.html` (~1,670 lines): the entire game. HTML markup, CSS (`<style>`, roughly lines 10–360),
  and one inline `<script>` IIFE (roughly lines 422–1668). There are no external assets,
  dependencies, or network calls.
- `README.md`: placeholder; contains only the project title and a timestamp.

There is no `package.json`, build step, bundler, linter, or test suite. Do not add one
unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, e.g.
`python3 -m http.server 8000` and then visit `http://localhost:8000/`.

## Code structure inside `index.html`

The script is wrapped in an IIFE with `'use strict'`. Its sections, in order:

1. **DOM lookups**: all element references are grabbed once at the top by `id`.
2. **Storage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`,
   `skyfighter.reducedMotion` (all under `localStorage`).
3. **Adaptive performance** (`trackPerf`): samples frame times and steps a `quality` level
   (0 full, 1 medium, 2 low) to keep the frame rate up.
4. **Config tables**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage),
   `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS` (bullet-pattern functions).
5. **Audio**: a lazily created `AudioContext` (`ensureAudio`) with synthesized `beep` and
   `noiseBurst` sound effects. Nothing is loaded from disk.
6. **HUD helpers**: hearts, bombs, combo pops, stage banners, screen shake, and a
   screen-reader live region (`announce`).
7. **Input**: keyboard (`keys` map), touch joystick and fire/bomb buttons, and mouse
   aiming on the canvas.
8. **Game state and logic**: `resetGame`, `update(dt)`, spawning, collisions
   (`rectsOverlap`), scoring with a combo multiplier (`addScore`, `breakChain`), power-ups,
   bombs, weapons, boss behaviour, `loseLife`, `endGame`, and pause.
9. **Rendering**: `draw()` and its helpers (`drawPlane`, `drawBossShip`, `drawClouds`).
   Canvas is sized to a logical `480×720` (`BASE_W`/`BASE_H`) and scaled by `resize()`.
10. **Main loop**: `loop(now)` clamps `dt` to 50 ms, then calls `trackPerf`, `update`, and `draw`
    via `requestAnimationFrame`.

### Conventions

- Keep the file self-contained: inline CSS and JS, no external libraries or CDN links.
- Keep the IIFE and `'use strict'` wrapper. Do not leak globals.
- Put tunable numbers in the config tables near the top of the script rather than inside
  update logic.
- Game time is in seconds and speeds are in pixels per second, scaled by `dt`. Do not
  tie logic to frame counts.
- Respect the `reducedMotion` setting when adding screen shake, flashes, or other
  motion effects (the `reduced-motion` body class is toggled by `updateMotionBtn`).
- Keep the touch and keyboard paths in parity when adding a control.
- Any new persisted setting should use a `skyfighter.*` key. Storage access can throw in
  some browser privacy modes, so guard new reads and writes rather than assuming they work.

## Verifying changes

No automated tests exist. Verify changes by loading `index.html` in a browser and playing:

- Start, pause/resume, and game over (lives reach 0), then check the best-score update.
- Each difficulty (`easy`, `normal`, `hard`) and the mute and reduced-motion toggles.
- Stage progression (every `KILLS_PER_STAGE` kills) and boss encounters.
- Keyboard controls on desktop, and touch controls, or a touch emulator, on a phone-sized viewport.

For a UI or gameplay change, prefer a screenshot or short browser run over assuming it works.

## Git and branches

- Default working branch for this task series: `claude/gifted-tesla-k8bjnx`.
- Commit with clear messages and push with `git push -u origin <branch>`.
- Do not open a pull request unless explicitly asked.

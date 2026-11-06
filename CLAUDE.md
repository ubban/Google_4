# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs entirely in the browser. It is a single self-contained file:

- `index.html` (~1,670 lines) — all markup, CSS (`<style>`, starting around line 10), and game logic (one inline `<script>` IIFE, starting around line 422).
- `README.md` — minimal placeholder.

There is no build step, no package manager, no bundler, no test suite, and no external dependencies. Do not introduce any of these without the user asking for them.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, e.g.:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Code layout inside `index.html`

The inline script is wrapped in a strict-mode IIFE. Major sections, in source order:

1. **DOM references** — `getElementById` lookups for every HUD/overlay element. IDs are load-bearing; renaming an element requires updating the matching lookup.
2. **Storage keys** — `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` (all in `localStorage`, all read in a try-less fashion today).
3. **Adaptive performance** — `trackPerf` samples frame times and steps a `quality` level (0 full / 1 medium / 2 low).
4. **Tuning tables** — `DIFFICULTIES` (spawn rates, enemy speed/shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (six background palettes cycled by stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses cycled by stage).
5. **Audio** — a Web Audio synthesizer (`ensureAudio`, `beep`, `noiseBurst`, and named `sfx.*` effects). Audio is created lazily on the first user gesture.
6. **HUD helpers** — hearts, bombs, combo pop, stage banner, screen shake, flash/vignette, screen-reader announcements via `announce()`.
7. **Input** — keyboard (`keys` map), touch joystick + FIRE/BOMB buttons, and mouse steering/firing on the canvas.
8. **Game state and entities** — `resetGame`, `update(dt)`, spawners (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), weapon firing (`fireWeapon`, plus boss patterns `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`), bombs (`useBomb`), power-ups (`applyPowerUp`), and scoring with a combo multiplier (`addScore`, `breakChain`).
9. **Rendering** — `drawPlane`, `drawBossShip`, explosions, and the background/theme drawing, all onto a 2D canvas.
10. **Main loop** — `loop(now)` calls `update` and draw each frame via `requestAnimationFrame`. Delta time is clamped to 50 ms.

### Controls (keep README/hints in sync)

- Keyboard: arrows/WASD move, Space fires, X or Shift bombs, P or Esc pauses.
- Mouse: move to steer, click to fire.
- Touch: drag in the left zone to fly, tap FIRE, tap BOMB.

The on-screen hint text lives in `#controlsHint` (around line 417). Update it whenever bindings change.

## Conventions

- **Single file.** Keep HTML, CSS, and JS in `index.html` unless the user asks to split it.
- **Vanilla JS only.** No frameworks, libraries, or CDN includes.
- **Tuning belongs in the constant tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, `KILLS_PER_STAGE`) rather than inline magic numbers.
- **Frame-rate independence.** Movement and timers should scale with `dt`, which is in seconds.
- **Accessibility.** Respect the reduced-motion setting (`reducedMotion` gates screen shake and similar effects). Route important state changes through `announce()` so screen readers get them.
- **Persistence.** Store only the four `skyfighter.*` keys. Keep the names stable, because existing players' saved scores and settings depend on them.
- **Mobile.** The layout must keep working at phone widths and with `viewport-fit=cover`. `resize()` scales the canvas to the `480×720` base size (`BASE_W`/`BASE_H`).

## Verifying changes

There are no automated tests. For any gameplay or rendering change:

1. Open `index.html` in a browser and start a game.
2. Exercise the path you touched: firing, bombs, stage clear, boss fight, game over, and pause.
3. Check the difficulty buttons, mute, and reduced-motion toggles if you touched them.
4. For layout changes, check a narrow (phone-sized) viewport as well as desktop.

If you can't run a browser, say so explicitly in your summary rather than implying the change was tested.

## Git

- Develop on the branch the session specifies. Do not push to other branches without permission.
- Write clear commit messages describing the gameplay or code change.

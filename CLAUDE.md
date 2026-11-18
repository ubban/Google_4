# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs entirely in the browser. It is a single self-contained file, `index.html` (~1,670 lines), with inline `<style>` and an inline `<script>`. It draws to one `<canvas id="gameCanvas">`.

`README.md` is a placeholder and carries no project documentation.

There is no build step, no package manager, no bundler, no framework, and no test suite. Do not add one unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server:

```sh
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Repository layout

| Path | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS, and JavaScript |
| `README.md` | Placeholder |
| `CLAUDE.md` | This file |

## Code structure inside `index.html`

The inline script is organized in sections, roughly in this order:

1. **DOM references** (`document.getElementById` calls near the top of the script).
2. **Storage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Read and written via `localStorage`.
3. **Adaptive performance**: `trackPerf` moves a `quality` level (0 full, 1 medium, 2 low) based on a rolling frame-time average.
4. **Config tables**:
   - `DIFFICULTIES` (easy / normal / hard): spawn rates, enemy speed and fire chance, starting lives and bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` (18).
   - `THEMES`: six sky palettes, cycled by stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP, from stage 2).
   - `BOSSES`: six bosses, each with a name, colors, and HP multiplier.
   - `BOSS_PATTERNS`: bullet patterns (radial, aimed spread, columns, spiral, cross, homing, double ring).
5. **Audio**: a Web Audio API synth (`ensureAudio`, `beep`, `noiseBurst`). No audio files are loaded.
6. **Rendering helpers**: `drawPlane`, `drawBossShip`, `drawClouds`, and the main `draw()`.
7. **Game logic**: `resetGame`, `update(dt)`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`, `useBomb`, `fireWeapon`, `applyPowerUp`.
8. **Input**: keyboard (`keydown` / `keyup` on `window`), mouse on the canvas, and touch joystick plus FIRE and BOMB buttons.
9. **Main loop**: `loop(now)` drives `update` and `draw` each frame.

## Controls (keep this in sync with the in-game hint)

- Keyboard: arrows or WASD to move, Space to fire, X or Shift for a bomb, P or Esc to pause.
- Mouse: move to steer, click to fire.
- Touch: drag the left zone to fly, tap FIRE, tap BOMB.

The control hint lives in `#controlsHint` in the overlay markup. Update it whenever a control changes.

## Conventions

- **Keep it one file.** Put new markup, styles, and script in `index.html`. Do not split out modules or assets unless the user asks.
- **Plain JavaScript and CSS.** Target modern evergreen browsers. Do not add a transpiler or a dependency.
- **Use the existing config tables** (`ENEMY_TYPES`, `BOSSES`, `DIFFICULTIES`, `THEMES`) for tunable numbers rather than hard-coding them in logic.
- **Persist player settings only through the `skyfighter.*` localStorage keys.** Wrap storage access so the game still works when storage is unavailable (private mode, blocked site data). Do not rename existing keys, because that would reset players' saved best score and settings.
- **Respect reduced motion.** Screen shake, hit flashes, and the low-health pulse all go through `reducedMotion`. Any new visual effect with motion should do the same.
- **Keep the layout phone-friendly.** The canvas scales to the viewport (`resize()`, base size `BASE_W` × `BASE_H` = 480 × 720). Touch controls must work alongside keyboard and mouse.
- **Accessibility:** announce important state changes through `announce()` into the `#srLive` live region. Keep buttons labeled.
- **Keep frame logic time-based.** `update(dt)` takes a delta time, so preserve that when changing movement or timers.

## Testing and verification

There are no automated tests. To verify a change:

1. Load `index.html` in a browser and start a game on each difficulty you touched.
2. Exercise the path you changed: firing, bombs, power-ups, a stage transition, a boss fight, game over, pause and resume.
3. Check the mobile layout at a phone-sized viewport, and confirm the touch controls still respond.
4. Confirm the best score, mute, difficulty, and reduced-motion settings persist across a reload.

If you add a test harness, document how to run it here.

## Git workflow

- Default working branch for this session: `claude/gifted-tesla-1lfd88`. Push there with `git push -u origin <branch>`.
- Write clear commit messages in the imperative mood, with a short summary line and a body that explains the why when it isn't obvious.
- Do not create pull requests unless explicitly asked.
- Do not commit local or secret files.

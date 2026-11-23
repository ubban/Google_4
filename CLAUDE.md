# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small, dependency-free static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a browser on an HTML5 canvas.

## Repository layout

| Path        | Purpose                                                                 |
|-------------|-------------------------------------------------------------------------|
| `index.html` | The entire game: markup, CSS (`<style>`, ~lines 10–360), and JavaScript (`<script>`, ~lines 422–1667). Roughly 1,670 lines. |
| `README.md`  | Minimal placeholder (title only).                                       |
| `CLAUDE.md`  | This file.                                                              |

There is no `package.json`, build step, bundler, test suite, linter, or CI config. Nothing needs to be installed.

## Running and testing

- **Run:** open `index.html` directly in a browser, or serve the folder with any static server (for example `python3 -m http.server` from the repo root, then visit `http://localhost:8000`).
- **Tests:** none exist. Verify changes by playing the game in a browser. For automated checks, a Playwright script can load the page and drive it, but none is committed.
- **Lint/build:** none configured. Keep the file valid HTML and check JavaScript syntax by loading the page and watching the console for errors.

## Architecture of `index.html`

The game is a single IIFE (`(function () { ... })();`) inside the `<script>` tag. Key parts, in file order:

1. **DOM references:** `const` lookups by element id (`gameCanvas`, `overlay`, `hud` elements, touch controls, boss bar, and so on). Element ids must match the markup in the `<body>`.
2. **Storage keys:** `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. These are the only persisted state, read and written through `localStorage`.
3. **Tuning tables:**
   - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` (currently 18), which sets when a stage ends and its boss appears.
   - `THEMES`: six sky palettes cycled per stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor`, `tank`, each with `minStage` gating.
   - `BOSSES`: six stage bosses with colors and `hpMul`.
4. **Audio:** a lazily created `AudioContext` (`ensureAudio`) and procedural sound effects (`beep`, `noiseBurst`, and the `sfx` object). No audio files are loaded.
5. **Input:** keyboard (`keys` map), mouse (canvas steering and click-to-fire), and touch (joystick in `moveZone`, `fireBtn`, `bombBtn`).
6. **Game state:** module-level `let` variables (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`, `chain`, `multiplier`, and others). `resetGame()` reinitializes them.
7. **Update / draw:** `update(dt)` advances the simulation, `draw()` renders everything to the canvas. `loop(now)` runs on `requestAnimationFrame`, clamps `dt` to 0.05 s, and only updates and draws while `running && !paused`.
8. **Boss patterns:** `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, driven by `updateBoss`.
9. **Power-ups:** `maybeSpawnPowerUp` drops `life`, `bomb`, or `weapon` pickups; `applyPowerUp` applies them. Weapons are `vulcan` and `laser`, each with levels 1–3.
10. **Adaptive quality:** `trackPerf` watches frame times and sets `quality` (0 full, 1 medium, 2 low) to reduce effects on slow devices.

### Controls (also shown in the in-game hint)

- Move: mouse, arrow keys, or WASD. Fire: Space (hold) or mouse click. Bomb: `X` or Shift. Pause: `P` or Esc.
- Touch: drag the left zone to fly, tap FIRE, tap BOMB.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not introduce a build step or framework without being asked.
- **No dependencies.** Use vanilla JavaScript, the Canvas 2D API, and Web Audio. Do not add CDN links or npm packages without approval.
- **Tuning changes go in the tables.** Adjust numbers in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES` rather than hard-coding magic values in the update loop.
- **Canvas coordinates** use a fixed logical size, `BASE_W = 480` × `BASE_H = 720`. `resize()` scales the canvas to fit the window (capped at 1.6×). Place new entities in these logical units.
- **Respect reduced motion.** Effects such as screen shake and flashes check `reducedMotion`. New visual effects should do the same.
- **Keep the HUD in sync.** When state changes, update the matching DOM element (`scoreEl`, `heartsEl`, `bombsEl`, `weaponNameEl`, and so on) and announce important events through `announce()` for screen readers.
- **Mobile first.** The page sets `user-scalable=no` and uses `viewport-fit=cover`. Verify layout and touch controls at phone width when changing UI.

## Git workflow

- Develop on the branch named in the session instructions (for this session, `claude/gifted-tesla-ga1t1z`). Do not push to other branches without explicit permission.
- Commit messages follow the existing style: a short imperative summary line, then a body explaining what changed and why.
- Do not create pull requests unless explicitly asked.

## Things to watch for

- The game's state is entirely in module-level variables inside the IIFE. Do not expose them globally just to debug. Use the browser console with temporary logging instead, and remove it before committing.
- `localStorage` calls are unguarded in the current code. If you add new persistence, wrap access in `try/catch` so the game still runs in private windows or when storage is blocked.
- Keep the `id` attributes in the markup and the `getElementById` calls in sync. A mismatch fails silently on some paths and throws on others.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game called **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game lives in one file, `index.html` (~1,670 lines): inline CSS in `<style>`, markup, and one inline `<script>` that uses the Canvas 2D API and Web Audio. There is no build step, no package manager, no framework, and no test suite.

Files:

- `index.html` — the whole game (HUD, overlays, touch controls, canvas, game logic).
- `README.md` — a stub; only the project title.

## Running the game

No build is needed. Open the file directly, or serve the directory statically:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

Because there is no tooling, verification is manual: load the page in a browser, play through a stage, and check the affected behaviour. For visual or input changes, test both desktop (keyboard/mouse) and touch-sized viewports.

## Code structure (inside `index.html`)

The script is a single IIFE-style block that grabs DOM elements up front, then defines state and functions:

- **Persistence keys** — `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` in `localStorage`. Preserve these names so saved player settings keep working.
- **Config tables** — `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six stage bosses with HP multipliers), `KILLS_PER_STAGE` (18). Tune game balance here rather than in logic.
- **Audio** — `ensureAudio()` lazily creates the `AudioContext` (browsers require a user gesture first). `beep`, `noiseBurst`, and the `sfx` object generate all sound procedurally; there are no audio files.
- **Input** — keyboard (`keys` map, `keydown`/`keyup`), mouse on the canvas, and touch (virtual joystick in `moveZone`, `fireBtn`, `bombBtn`). Touch is detected via `isTouchDevice`.
- **Layout** — `BASE_W = 480`, `BASE_H = 720`; `resize()` scales the canvas to fit the viewport (capped at 1.6×).
- **Performance** — `trackPerf(dt)` watches frame times and steps a `quality` level (0 full, 1 medium, 2 low) to reduce effects when the frame rate drops.
- **Game loop** — `loop(now)` drives `update(dt)` (game state) and rendering, and re-schedules itself with `requestAnimationFrame`.
- **Gameplay functions** — `spawnEnemy`, `spawnBoss`, `updateBoss`, `fireWeapon`, `useBomb`, `spawnExplosion`, and the boss pattern helpers (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
- **UI helpers** — `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `flashHit`/`flashBomb`, `announce` (screen-reader live region `srLive`).
- **Accessibility** — `reducedMotion` (defaults from `prefers-reduced-motion`, user-toggleable via `motionBtn`) disables or softens shake/flash effects. Keep it honoured by any new visual effect.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split files. If you do split, update this document and make sure paths stay relative (no leading `/`) so the game still works when opened from disk.
- No external dependencies or CDN scripts. The game must run offline from a single file.
- Stay with vanilla JS and the existing style: 2-space indentation, `const`/`let`, single quotes, and small named functions.
- Mobile matters: the layout uses `viewport-fit=cover`, safe-area insets, `touch-action: none`, and touch controls. Check any HUD or control change at phone width.
- Any new user-facing text change should also update the `announce()` / `srLive` path where relevant.
- Keep the game's score, best-score, and settings behaviour intact; they are player-visible.

## Git workflow

- Develop and push on the designated session branch, `claude/gifted-tesla-bcgfos`. Do not push to other branches without explicit permission.
- Commit messages are short, imperative summaries (e.g. `Add Sky Fighter: ...`).
- Do not open a pull request unless explicitly asked.

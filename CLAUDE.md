# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-file, Raiden-style vertical shoot-'em-up that runs in the browser on HTML5 canvas. There is no build step, no package manager, and no test suite.

| File        | Purpose                                                              |
|-------------|----------------------------------------------------------------------|
| `index.html`| The entire game (~1,670 lines): markup and inline `<style>` (lines 1–421), inline `<script>` (lines 422–end) |
| `README.md` | Minimal project readme (title only)                                  |

Git: the initial commit is `4efdad1`; the game was added in `c4dc8f6`.

## Running the game

No tooling is required. Either:

- open `index.html` directly in a browser, or
- serve the directory statically, e.g. `python3 -m http.server 8000`, then visit `http://localhost:8000/`.

## Architecture of `index.html`

Everything runs inside one IIFE (`(function () { 'use strict'; ... })()`) in the inline script. Keep new code inside that scope rather than adding globals.

Major sections, in file order:

1. **DOM references** – `const xxxEl = document.getElementById(...)` block near the top of the script.
2. **Storage keys** – `skyfighter.*` keys in `localStorage` for best score, mute, difficulty and reduced motion. Every access to `localStorage` should keep working when storage is unavailable or throws.
3. **Adaptive performance** – `trackPerf()` samples frame time over 45 frames and steps a `quality` level (0 full, 1 medium, 2 low). Effects such as shadow blur, particle counts and glow check `quality`.
4. **Config tables** – `DIFFICULTIES`, `THEMES` (background palettes per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses), `KILLS_PER_STAGE` (18).
5. **Audio** – a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, the `sfx` object). No audio files are loaded. Audio is created lazily on the first user gesture.
6. **HUD helpers** – hearts, bombs, combo pop, stage banner, screen shake, flash, and an ARIA live region (`announce`) for screen readers.
7. **Input** – keyboard (`keys` map keyed by `e.code`), a touch joystick on the left (`moveZone`), tap-to-fire and bomb buttons, and mouse steering/firing on the canvas.
8. **Game state and loop** – `resetGame()`, `update(dt)`, `draw()`, `loop(now)`. `dt` is clamped to 50 ms. `update` runs only when `running && !paused`.
9. **Combat** – `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `applyPowerUp`, `maybeSpawnPowerUp`, and a family of boss bullet patterns (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, `fireHomingSeeded`, etc.) listed in `BOSS_PATTERNS`.
10. **Lifecycle** – `loseLife`, `killBoss`, `endGame`, `togglePause`.
11. **Rendering** – `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`. Everything is drawn on the 2D context; the logical size is `BASE_W × BASE_H` = 480 × 720, scaled by `resize()`.
12. **Start** – `startBtn` click handler, then `requestAnimationFrame(loop)`.

### Controls (keep the hint text in sync)

- Mouse: move to steer, click to fire.
- Arrows / WASD: move. Space: fire.
- X or Shift: bomb.
- P or Esc: pause.
- Touch: drag left side to fly, FIRE and BOMB buttons on screen.

The controls hint is the `#controlsHint` paragraph in the overlay markup. Update it whenever a control changes.

### Accessibility

- A reduced-motion mode (`reducedMotion`) defaults to the OS `prefers-reduced-motion` setting and can be toggled with the motion button. It suppresses screen shake and flash.
- Status changes are announced through the `#srLive` live region via `announce()`.
- Respect these when adding effects: anything that flashes or shakes should check `reducedMotion`.

## Conventions

- **Single file.** Do not split the game into modules or add a build step unless the user asks. Keep CSS in the `<style>` block and JS in the single `<script>`.
- **No dependencies.** Do not add libraries or CDN links without a request. The game uses only the canvas and Web Audio APIs.
- **Plain JS.** Match the existing style: `const`/`let`, arrow functions, two-space indentation, single quotes, semicolons.
- **Tunable values live in the config tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than being hard-coded in logic. Add new enemies, bosses and themes there.
- **Preserve mobile support.** Layout uses `viewport-fit=cover` and a fixed logical canvas size; test changes to HUD or controls at phone width.
- **Preserve keyboard, mouse and touch paths** when changing input handling.

## Testing and verification

There is no automated test suite or linter. Verify changes by:

1. Opening the game in a browser (or using the `run` skill / a headless Chromium) and playing through at least one stage.
2. Checking all three difficulty buttons and the mute, motion and pause toggles.
3. Confirming that the best score persists across reloads.
4. Checking touch controls at a phone-sized viewport if the change touches input or layout.

For a quick syntax check of the inline script, extract the `<script>` block and run `node --check` on it.

## Git workflow

- Develop on the branch designated for the session (currently `claude/gifted-tesla-lhbmd3`). Do not push to other branches without explicit permission.
- Do not open a pull request unless the user explicitly asks for one.
- Commit messages: short imperative summary line, followed by a body when the change needs explanation.

## Keeping this file current

Update this CLAUDE.md when the file layout changes, when a new control or storage key is added, or when a new workflow (tests, build, deployment) is introduced.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-page, Raiden-style vertical shoot-'em-up built on an HTML5 `<canvas>`. There is no build step, framework, or package manager.

Files:

- `index.html` — the entire game: markup, CSS (in a `<style>` block), and JavaScript (one IIFE in a `<script>` block, starting around line 422).
- `README.md` — minimal project title.

There is no `package.json`, no test suite, no linter config, and no CI workflow in the repo.

## Running the game

No install is needed. Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server` and visit `http://localhost:8000`). Test in a browser; nothing else can verify gameplay.

## Architecture (inside `index.html`)

The game script is a single IIFE. Major sections, roughly in file order:

- **DOM refs and constants** — element lookups, `STORAGE_*` localStorage keys (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`).
- **Config tables** — `DIFFICULTIES` (easy/normal/hard: spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six boss designs, cycled by stage).
- **Persistence and settings** — best score, mute, difficulty, and reduced-motion preference are read from and written to `localStorage`.
- **Audio** — lazily created `AudioContext` (`ensureAudio`) with synthesized `beep` and `noiseBurst` helpers; named effects live in the `sfx` map.
- **HUD and feedback** — hearts/bombs rendering, hit and bomb flashes, combo popups, stage banner, screen shake.
- **Layout and input** — `resize()` scales a 480×720 logical canvas to fit the window (max 1.6×). Keyboard, mouse, and touch (virtual joystick plus fire/bomb buttons) all feed the same state.
- **Game state** — module-level `let` variables (player, bullets, enemies, boss, score, lives, chain/multiplier, stage, etc.). `resetGame()` initializes them.
- **Rendering** — `drawPlane`, `drawBossShip`, and `draw()`.
- **Simulation** — `update(dt)` advances enemies, bullets, power-ups, and stage progression; `updateBoss(dt)` drives boss patterns and telegraphs.
- **Main loop** — `loop(now)` calls `update` and `draw` each frame via `requestAnimationFrame`. `trackPerf` lowers a `quality` level (full/medium/low) when frame times are poor.

### Controls

- Keyboard: arrow keys / WASD to move, Space to fire, `X` or Shift to use a bomb, `P` or Escape to pause.
- Power-up types currently spawned: `life`, `bomb`, and weapon upgrades (`vulcan` / `laser`).
- Mouse and touch: pointer drives the ship position; touch devices get on-screen controls (`#touchControls`).
- Mute, reduced-motion, and pause also have on-screen buttons.

### Gameplay rules worth knowing before editing

- Stages advance after `KILLS_PER_STAGE` kills; a boss spawns about 1.2 s later.
- Kill chains raise the score multiplier (`1 + min(9, floor(chain / 5))`); `breakChain()` resets the chain and multiplier.
- Bombs are limited per difficulty and damage bosses.
- Power-ups include weapon upgrades (Vulcan/Laser, 3 levels each), bombs, and extra life.

## Conventions

- Keep everything in `index.html` unless the user asks to split files. Don't add a build step, bundler, or dependencies without a clear request.
- Stick to vanilla JavaScript and CSS with no external libraries. The file currently loads no remote resources, and the favicon is an inline data URI.
- Keep the 480×720 logical resolution and the existing coordinate system when changing rendering or collision code.
- Preserve the `localStorage` key names; changing them would reset players' saved best score and settings.
- Some `localStorage` reads and writes are not wrapped in try/catch. When adding new storage access, wrap it so the game still runs in restricted browser contexts (private mode, blocked storage).
- Keep the code style consistent with the surrounding code: 2-space indentation, single quotes in JS, `const`/`let`, and small named helper functions.
- Respect `prefers-reduced-motion` and the in-game reduced-motion toggle when adding screen shake, flashes, or pulsing effects.
- Ensure new UI works at phone width and with touch input, since the game supports mobile controls.

## Making changes

- Read the relevant section of `index.html` before editing; the file is large (about 58 KB), so search by function name rather than reading all of it.
- Verify changes by opening the game in a browser and exercising the affected path (menu, gameplay, pause, game over, and any touched difficulty or stage). No automated tests exist, so say clearly in summaries when a change has only been checked by reading.
- Commit with clear messages describing the gameplay or UI change.
- Do not commit personal data, credentials, or generated files.

## Git

- Default branch: `main`. Feature work for this session is on `claude/gifted-tesla-jrd385`.
- Do not open pull requests unless the user asks for one.

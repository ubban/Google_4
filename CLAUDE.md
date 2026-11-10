# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, browser-based vertical shoot-'em-up in the style of Raiden. There is no build system, package manager, or test suite.

- `index.html` — the entire game: inline `<style>`, markup (HUD, overlays, touch controls), and one inline `<script>` wrapped in an IIFE with `'use strict'`. Roughly 1,670 lines.
- `README.md` — minimal placeholder.

To run the game, open `index.html` in a browser. No server or install step is needed.

## Architecture (inside `index.html`)

The script is organized as one closure around a 2D canvas (`#gameCanvas`) with a fixed logical resolution of `BASE_W = 480` x `BASE_H = 720`, scaled by `resize()`.

Key sections, roughly in file order:

- **Config constants** — `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky/cloud/sun palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six named bosses with colors and HP multipliers), and `BOSS_PATTERNS` (bullet-pattern generators such as radial, spiral, cross, homing).
- **Persistence** — `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). All reads/writes are direct and unguarded; keep it that way unless adding a failure path is part of the task.
- **Audio** — a lazily created Web Audio context (`ensureAudio`) with `beep` and `noiseBurst` helpers. Audio must be initialized from a user gesture; `ensureAudio()` is called from input handlers.
- **Input** — keyboard (`keys` map keyed by `e.code`), touch joystick (`moveZone`), fire and bomb buttons, and pointer/canvas handling (`canvasPointFromEvent`).
  - Move: arrow keys or `A`/`D` (via `keys`), or the on-screen joystick.
  - Fire: `Space` (held) or the fire button.
  - Bomb: `X` or either `Shift`, or the bomb button.
  - Pause: `P` or `Escape`.
  - Mute and motion toggles are on-screen buttons.
- **Game state and loop** — `resetGame()` initializes state; `update(dt)` advances simulation (movement, firing, collisions, spawning, power-ups, boss logic); `draw()` renders; `loop(now)` drives the frame cycle. `loseLife()`, `endGame()`, `killBoss()`, and `togglePause()` handle state transitions.
- **Rendering helpers** — `drawPlane`, `drawBossShip`, `drawClouds`, and HUD helpers (`renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `flashHit`, `flashBomb`).
- **Accessibility** — a screen-reader live region (`srLive`) via `announce()`, a reduced-motion setting that suppresses shake/flash effects, and `prefers-reduced-motion` detection.

Stages advance every `KILLS_PER_STAGE` kills; each stage cycles to the next theme and boss. Bosses are spawned by `spawnBoss()` and killed via `killBoss()`.

## Working conventions

- Keep the game self-contained in `index.html`. Do not introduce a bundler, framework, or external assets unless the user asks for it.
- Preserve the existing style: `const` for config, `camelCase` for functions and variables, `SCREAMING_SNAKE_CASE` for constant tables, and two-space indentation.
- Keep changes minimal and local. Gameplay numbers (difficulty tables, enemy stats, boss HP multipliers) are tuned values; change them only when the task calls for it.
- Maintain mobile support: the layout uses `viewport-fit=cover`, safe-area insets, and `touch-action: none`. Check touch behavior when changing controls.
- Respect the reduced-motion setting when adding visual effects (shake, flashes).

## Verifying changes

There are no automated tests or linters. To verify a change:

1. Open `index.html` in a browser (or serve the directory with any static server).
2. Play through at least one stage on each difficulty you touched, including a boss fight.
3. Check the HUD (score, best, hearts, bombs, weapon, boss bar), the pause/mute/motion toggles, and game-over/new-best flow.
4. For layout changes, test a narrow (phone-width) viewport as well as desktop.

For a quick syntax check of the inline script, extract the `<script>` block and run `node --check` on it.

## Git

- Development branch for this work: `claude/gifted-tesla-upxxyt`. Commit and push there.
- Write clear, descriptive commit messages that explain what changed and why.
- Do not open a pull request unless explicitly asked.

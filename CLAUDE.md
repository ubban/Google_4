# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small browser game, **Sky Fighter** — a Raiden-style vertical shoot-'em-up.
The entire game lives in one self-contained file, `index.html` (~1,700 lines):

- `<style>` (approx. lines 10–360): all CSS, including the HUD, overlays and touch controls.
- `<body>` (approx. lines 361–421): the canvas (`#gameCanvas`), HUD elements, overlays, and touch controls.
- `<script>` (approx. lines 422 onward): one IIFE containing all game logic, rendering, audio, and input.

`README.md` is a placeholder and carries no project documentation.

## Running the game

There is no build step, package manager, or dependency install. Open `index.html` directly in a browser,
or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it.

## Testing and tooling

- There is no test suite, linter, formatter, or CI configuration in the repo.
- Verify changes by loading the game in a browser and playing through the affected behavior.
  For headless checks, Playwright with Chromium is the practical option.
- Do not add a build system, framework, or npm dependencies unless the user asks for it.
  The project's appeal is that it is a single file with no tooling.

## Code structure inside `index.html`

Key areas of the script, in roughly the order they appear:

- **Config constants**: `STORAGE_*` localStorage keys, `DIFFICULTIES` (spawn rate, enemy speed,
  shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and the `EXPLODE_SFX` mapping. Audio is synthesized
  with the Web Audio API; there are no audio asset files.
- **Layout and input**: `BASE_W`/`BASE_H` (480×720 logical resolution), `resize`, joystick and pointer
  handling, and the keyboard handlers.
- **Game state**: `resetGame`, `update(dt)` (per-frame simulation), `applyPowerUp`, `fireWeapon`, `useBomb`.
- **Bosses**: `BOSS_PATTERNS` (bullet-pattern generators such as `fireRadial`, `fireCross`,
  `fireHomingSeeded`), `updateBoss`, `killBoss`.
- **Rendering**: `draw()` and helpers such as `drawPlane`, `drawBossShip`, `drawClouds`.
- **Main loop**: `loop(now)` drives `update` and `draw` with a delta-time value.

## Controls (for manual testing)

- Mouse: move to steer, click to fire.
- Keyboard: arrow keys / WASD to move, Space to fire, `X` or Shift to use a bomb, `P` or Escape to pause.
- Touch: drag on the left side to fly (joystick), tap FIRE, tap BOMB.
- Difficulty is chosen from the `.diffBtn` buttons (easy / normal / hard) and persisted in localStorage.

## Persistence and accessibility

- Stored in `localStorage` under the `skyfighter.*` keys: best score, mute state, difficulty,
  and reduced-motion preference. Read and write these defensively; storage may be unavailable.
- The game has a reduced-motion option (`motionBtn`) that tones down screen shake and flashes.
- Score and status updates are announced via the `#srLive` live region. Keep this working when adding
  new messages (use `announce(text)`).

## Conventions

- Keep the game self-contained in `index.html`. Put new CSS in the `<style>` block and new JS
  inside the existing IIFE, following the naming style already used (camelCase functions,
  UPPER_SNAKE_CASE constants).
- Use the existing `dt`-based update pattern for any new timed behavior so it stays frame-rate independent.
- Keep changes minimal and focused; don't refactor unrelated parts of the file.
- Preserve the mobile layout: the game should remain playable at phone width with touch controls.

## Git

- Development branch for this work: `claude/gifted-tesla-5mectz`.
- Commit messages should be short, imperative, and describe the gameplay or code change.

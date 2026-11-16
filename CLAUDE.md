# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only substantive content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a `<canvas>`.

- `index.html` — the entire game: markup, CSS, and JavaScript in one self-contained file (~1,670 lines).
- `README.md` — minimal placeholder.

There is no build step, no package manager, no bundler, no framework, and no test suite. Do not introduce any of these without being asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it. No install step is needed.

## Code layout inside `index.html`

- `<head>`: meta tags (viewport locked to the device, `theme-color` `#0b1d3a`) and a single inline `<style>` block.
- `<body>`: HUD elements (score, best, multiplier, weapon, hearts, bombs, boss bar), the `<canvas id="gameCanvas">`, the start/pause overlay, and on-screen touch controls (`#touchControls`: joystick, fire, bomb).
- A single inline `<script>` wrapped in an IIFE with `'use strict'`. Nothing is exposed on `window`; keep it that way unless there is a reason.

Major sections in the script, in file order (search for the comment headers and function names):

| Area | Key identifiers |
|---|---|
| Persistence | `STORAGE_*` constants (`localStorage` keys prefixed `skyfighter.`) |
| Adaptive quality | `trackPerf`, `quality` (0 full / 1 medium / 2 low) |
| Tuning data | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `sfx`, `EXPLODE_SFX` (Web Audio, synthesized; no audio files) |
| Layout / input | `resize`, `BASE_W = 480`, `BASE_H = 720`, `keys`, joystick and pointer handlers |
| Game state | `resetGame`, `update(dt)`, `loseLife`, `endGame`, `togglePause` |
| Weapons & bombs | `fireWeapon`, `applyPowerUp`, `useBomb` |
| Enemies & bosses | `spawnEnemy`, `spawnBoss`, `updateBoss`, `killBoss`, `BOSS_PATTERNS` and the `fire*` pattern helpers |
| Rendering | `draw`, `drawPlane`, `drawBossShip`, `drawClouds`, `spawnExplosion` |
| Loop | `loop(now)` driven by `requestAnimationFrame`; `dt` is clamped to 50 ms |

### Gameplay facts worth knowing before editing

- **Difficulty** (`easy` / `normal` / `hard`) sets spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and a boss HP multiplier.
- **Stages** advance every `KILLS_PER_STAGE` (18) kills, cycle through `THEMES`, and bring in a boss from `BOSSES`.
- **Weapons**: `vulcan` (spread grows with `player.level`) and `laser`. Power-ups toggle/upgrade them.
- **Controls**: arrows or WASD to move; Space or hold to fire; X / Shift for a bomb; P or Escape to pause. Touch uses the joystick and buttons.
- **Accessibility**: respects `prefers-reduced-motion` unless the player overrides it (persisted); has an `aria-live` region (`#srLive`) fed by `announce()`.

## Conventions

- Keep the game in the single `index.html` file unless the user asks for a split. Do not add build tooling.
- Match the existing style: 2-space indentation, `const`/`let`, arrow functions for callbacks, section comments with `// ---------- Name ----------`.
- Tuning values live in the data tables at the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`). Prefer editing those over hard-coding numbers in logic.
- Any new persisted setting should use a `skyfighter.`-prefixed `localStorage` key declared alongside the existing `STORAGE_*` constants, and be read inside a try/catch-tolerant path if it could be unavailable.
- Preserve mobile support: the viewport is non-scalable and touch controls must keep working at phone widths.
- Audio is generated with Web Audio; do not add binary sound assets without a reason.

## Verification

No automated tests exist. To verify a change:

1. Open `index.html` in a browser (or a static server) and check the console for errors.
2. Start a run, and exercise the changed system (movement, firing, bombs, stage transitions, bosses, pause, game over).
3. Check the best-score persistence across a reload if you touched scoring or storage.
4. For layout or rendering changes, check both desktop and a narrow phone-sized viewport, and the reduced-motion setting.

Say explicitly in the final report which of these you did and did not run.

## Git

- Development branch for this work: `claude/gifted-tesla-bvpntu`. Push there; do not push to other branches without explicit permission.
- Do not open pull requests unless the user asks.

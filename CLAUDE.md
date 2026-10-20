# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up written as a single self-contained HTML5 canvas page.

- `index.html` — the whole game: markup, CSS (`<style>`, ~lines 10–360), HTML overlay/HUD
  (~lines 362–420), and the game script (`<script>`, starts ~line 422).
- `README.md` — placeholder; has no project description yet.

There is no build step, no bundler, no package manager, no test suite, and no linter
configured. Do not add one unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server
  (e.g. `python3 -m http.server`) and visit it.
- Verify changes by loading the page and playing it. Check the browser console for errors
  after any change to the script.
- The game must keep working on desktop (keyboard + mouse) and touch devices. Touch
  controls are shown only when the device reports touch support.

## Code layout inside the game script

The script is one IIFE with `'use strict'`. Keep new code inside it; do not leak globals.
Sections appear in roughly this order, marked by `// ---------- Name ----------` comments:

- **DOM references and constants** — element lookups, `STORAGE_*` keys, `DIFFICULTIES`,
  `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
- **Adaptive performance** — `trackPerf` lowers visual `quality` (0/1/2) when frame time
  is high.
- **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` table. Audio is created
  lazily on the first user gesture; keep it that way.
- **Input** — keyboard (`keys`), joystick, fire/bomb buttons, and mouse steering.
- **Game state and flow** — `resetGame`, `update(dt)`, `draw()`, `loop(now)`,
  `togglePause`, `loseLife`, `endGame`.
- **Spawning and combat** — enemies, bosses (`BOSS_PATTERNS`, `updateBoss`, `killBoss`),
  power-ups (`maybeSpawnPowerUp`, `applyPowerUp`), bombs (`useBomb`), score and kill-chain
  multiplier (`addScore`, `breakChain`).
- **Rendering helpers** — `drawPlane`, `drawBossShip`, `drawClouds`, particles, HUD updates.

### Behaviour to preserve

- Time-based movement: `update(dt)` receives seconds, and `dt` is clamped to 0.05 in the
  loop. Movement speeds are per-second values, not per-frame.
- Canvas logical size is fixed at `BASE_W × BASE_H` (480 × 720); `resize()` only scales the
  CSS size. Coordinates in game code always use the logical size.
- A stage's boss spawns about 1.2 s after `stageKills` reaches `KILLS_PER_STAGE`. The boss
  design is `BOSSES[(stage - 1) % BOSSES.length]`, and its HP scales with stage and
  difficulty (`bossHpMul`).
- Weapon power-ups cycle: Vulcan level 1 → 2 → 3, then switch to Laser at level 1.
- Starting lives and bombs come from the selected entry in `DIFFICULTIES`.
- Pause is toggled with P or Esc. While paused, the overlay shows "Paused" and the
  Resume button.

## Persistence

`localStorage` keys (all prefixed `skyfighter.`):

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | Best score (number) |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | `'1'` / `'0'`; defaults to the OS `prefers-reduced-motion` setting |

Reads and writes must not break the game if storage is unavailable. Keep any new keys
under the `skyfighter.` prefix and document them in the table above.

## Conventions

- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let`,
  arrow functions for callbacks, short section comments with `// ---------- Name ----------`.
- Prefer small, local changes to the existing functions over new abstractions. The game is
  intentionally one file.
- Keep user-visible text consistent with the existing HUD and overlay copy. Update the
  `#controlsHint` text if controls change.
- Accessibility: announcements go through `announce()` into the `#srLive` region. Respect
  `reducedMotion` for screen shake, flashes, and similar effects.
- Do not introduce external dependencies, CDN scripts, or network calls.

## Git

- Develop on the branch assigned for the task (currently `claude/gifted-tesla-nljlwc`),
  not directly on `main`.
- Write descriptive commit messages that explain the change, not just the file touched.
- Do not open a pull request unless explicitly asked.

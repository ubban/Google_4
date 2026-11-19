# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project containing **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a browser canvas.

The entire game lives in one self-contained file:

- `index.html` (~58 KB) — markup, CSS (in a `<style>` block), and game logic (in a single IIFE `<script>` starting around line 422).
- `README.md` — a placeholder title only.

There is no build step, no package manager, no bundler, no tests, and no external dependencies. Audio is synthesized with the Web Audio API and persistence uses `localStorage`.

## Running the game

No build is needed. Open the file directly or serve the directory statically:

```bash
# Option 1: open directly
xdg-open index.html

# Option 2: local static server (useful for testing on another device)
python3 -m http.server 8000
# then visit http://localhost:8000/index.html
```

## Repository layout

```
.
├── index.html   # The whole game: HTML, CSS, and JS
├── README.md    # Title placeholder
└── CLAUDE.md    # This file
```

Keep the game in a single file unless there is a clear reason to split it. If you do split it, update this section and the "Running the game" section.

## Code architecture (`index.html`)

The script is wrapped in an IIFE with `'use strict'`. Key areas, in roughly the order they appear:

- **DOM references** (~line 425–460): all `getElementById` lookups are grabbed once at the top.
- **Storage keys** (`STORAGE_*`): `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`.
- **Tuning tables** — the main place to change game balance:
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills required to advance a stage.
  - `THEMES`: six sky palettes cycled per stage (`Daylight Front`, `Sunset Corridor`, `Night Approach`, `Storm Front`, `Arctic Wastes`, `Volcanic Ridge`).
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP). Each has a `minStage` gate.
  - `BOSSES`: six bosses (`VANGUARD` → `TITAN`), each with colors and an `hpMul`.
  - `BOSS_PATTERNS` (~line 1372): bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Audio** (`ensureAudio`, `beep`, `noiseBurst`): procedural sound effects. Audio context is created lazily on the first user gesture.
- **Input** (~line 769 onward): keyboard, mouse, pointer, and a virtual joystick/buttons for touch. See controls below.
- **Game state and loop**: `resetGame`, `update(dt)`, `draw()`, and `loop(now)`. `update` advances simulation with a delta-time value; `draw` renders the frame.
- **Weapons and power-ups**: `fireWeapon`, `applyPowerUp`, `maybeSpawnPowerUp`.
- **Bosses**: `spawnBoss`, `updateBoss`, `killBoss`.
- **Lifecycle**: `loseLife`, `endGame`, `togglePause`, `resize`.
- **HUD and accessibility**: `renderHearts`, `renderBombs`, `announce` (writes to an `aria-live` region), reduced-motion handling, and `screenShake`/`flashHit` effects that respect the reduced-motion setting.

### Conventions observed in the code

- Vanilla JavaScript only. No frameworks, no modules, no `import`/`export`.
- Game entities are plain objects in arrays; collisions use `rectsOverlap`.
- Time-based movement is driven by `dt`, so keep new movement frame-rate independent.
- Visual feedback is drawn with canvas 2D primitives (`drawPlane`, `drawBossShip`, `drawClouds`). Sprites are not loaded from files.
- Respect `reducedMotion` for any new screen shake, flash, or large animation.
- Use `announce()` for any new player-facing status change so screen readers receive it.

## Controls

| Action | Desktop | Touch |
| --- | --- | --- |
| Move | Mouse, Arrow keys, or WASD | Drag left side to fly (virtual joystick) |
| Fire | Hold Space, or click/hold mouse | Tap FIRE |
| Bomb | `X` or Shift | Tap BOMB |
| Pause | `P` or Esc | Pause button |
| Mute | Mute button | Mute button |

## Working in this repo

- **Verify changes in a browser.** There are no automated tests, so open `index.html` and play the affected path (menu → game → the relevant stage, boss, or difficulty). Check both desktop and a narrow/mobile viewport, since the layout is responsive and touch controls are shown on small screens.
- **Keep changes local.** Prefer editing the relevant tuning table or function over restructuring the IIFE.
- **Balance changes** belong in the tuning tables at the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`), not hard-coded in logic.
- **Don't add dependencies** (npm packages, CDN scripts, or assets) without a clear need. The current design is a single file that runs offline.
- **Preserve storage keys.** Changing a `STORAGE_*` value resets players' saved best score, mute, difficulty, and motion preferences.

## Git workflow

- Default working branch for this session: `claude/gifted-tesla-hff4gr`.
- Commit with clear, imperative messages describing the gameplay or code change.
- Do not force-push. Do not open a pull request unless explicitly asked.

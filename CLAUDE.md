# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, self-contained browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up rendered on an HTML5 `<canvas>`.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–362), canvas element, and all JavaScript (`<script>`, starting ~line 422). This is roughly 1,670 lines.
- `README.md` — a one-line title. No other docs.

There is no build step, no package manager, no test suite, no linter, and no CI configuration. Do not invent or assume any of these exist. If you add tooling, say so in the commit and update this file.

## Running the game

Open `index.html` directly in a browser. No server or install is required.

## Code architecture (`index.html`)

- All game code lives in a single IIFE inside the `<script>` tag. Nothing is exported to `window`; keep new code inside that closure.
- Main loop is `requestAnimationFrame(loop)` → `update(dt)` → `draw()`. `update` handles movement, firing, collisions, spawning, and boss logic; `draw` renders everything each frame.
- Canvas logical size is `BASE_W × BASE_H` (480 × 720). `resize()` scales it to the viewport.
- Audio is generated with the Web Audio API (`beep`, `noiseBurst`, `ensureAudio`); there are no audio asset files. `ensureAudio()` must run from a user gesture.

### Tunable game data

Balance and content are declared as constants near the top of the script. Prefer editing these over hard-coding values in logic:

- `DIFFICULTIES` — spawn rates, enemy speed and fire chance, starting lives and bombs, boss HP multiplier (`easy` / `normal` / `hard`).
- `KILLS_PER_STAGE` — kills needed to advance a stage (18).
- `THEMES` — per-stage sky, cloud, and sun palettes; cycled by stage.
- `ENEMY_TYPES` — `grunt`, `interceptor`, `tank`, with HP, speed, score, and the `minStage` each unlocks at.
- `BOSSES` — boss names, colors, and HP multipliers. Boss attacks are in `BOSS_PATTERNS` (`fire*` helpers).

### Power-ups

Spawned by `maybeSpawnPowerUp` with type `life`, `bomb`, or `weapon`, applied in `applyPowerUp`. Weapons have three levels (Vulcan / Laser).

## Persistence

`localStorage` keys, all prefixed `skyfighter.`:

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | High score |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | `easy` / `normal` / `hard` |
| `skyfighter.reducedMotion` | `'1'` / `'0'`; defaults to the `prefers-reduced-motion` media query when unset |

Reads and writes are not wrapped in try/catch. If you add more storage, keep the `skyfighter.` prefix and consider failure modes (private browsing, disabled storage).

## Controls

- **Move:** arrow keys or WASD (keyboard), or the on-screen joystick (touch).
- **Fire:** Space (hold), or the fire button / mouse press.
- **Bomb:** X or Shift (keyboard), or the bomb button. Bombs clear the screen and damage bosses.
- **Pause:** P or Escape.

Keep keyboard, touch, and mouse input paths in sync when changing controls.

## Conventions

- Plain HTML, CSS, and vanilla JavaScript (ES2015+ syntax as already used). Do not add frameworks or bundlers without a clear reason.
- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let`, camelCase names, `UPPER_SNAKE` for constant tables.
- Keep the game playable on both desktop and mobile widths. Touch controls must keep working when you change input or layout.
- Respect `reducedMotion`: new screen shake, flashes, or large animations should check it.
- Preserve the single-file structure unless the user asks for a split.

## Verification

There are no automated tests. To verify a change, open `index.html` in a browser and play through the affected path (menu → gameplay → pause → game over, and at least one boss or stage transition if the change touches combat or progression). Check both keyboard and touch controls where relevant.

## Git

- Development branch for this work: `claude/gifted-tesla-kl5n2t`.
- Keep commits focused. The existing history uses short imperative subjects with a descriptive body.

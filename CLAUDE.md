# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

Google_4 is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The whole game lives in one file, `index.html`. `README.md` is a placeholder.

There is no build step, no package manager, no test suite, and no linter configured. Do not invent one. To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Layout of `index.html`

| Lines (approx.) | Contents |
| --- | --- |
| 1–9 | Doctype, viewport and theme meta tags, `<title>` |
| 10–359 | `<style>`: HUD pills, glassmorphic overlay card, touch controls, boss bar, animations |
| 360–421 | Markup: `#gameCanvas`, HUD elements, overlay (title, difficulty buttons, start button), touch controls |
| 422–1667 | `<script>`: one IIFE containing all game logic |

Everything in the script is inside a single `(function () { ... })();`. Nothing is exported and there are no globals to preserve.

## Architecture of the script

Roughly in the order they appear:

1. **DOM refs and constants.** Element lookups are at the top. Storage keys are prefixed `skyfighter.` (`STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`).
2. **Config tables.** `DIFFICULTIES` (spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `THEMES` (one per stage, cycling), `ENEMY_TYPES` (grunt, interceptor, tank, with `minStage` gating), `BOSSES` (cycled by stage). `KILLS_PER_STAGE` controls wave length.
3. **Audio.** Web Audio API, created lazily by `ensureAudio()` on the first user gesture. Sounds are procedural (`beep`, `noiseBurst`) and grouped in `sfx`.
4. **Quality scaling.** `trackPerf()` averages frame time and sets `quality` (0 full, 1 medium, 2 low). Particle counts and glow respond to it.
5. **Input.** Keyboard (`keys` map by `e.code`), mouse (`mouseTarget`, follows the pointer over the canvas), and touch (virtual joystick, fire and bomb buttons). `firing` is shared by touch and mouse.
6. **Game state.** Module-level `let`s in `resetGame()`: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, plus `score`, `lives`, `bombCount`, `stage`, `chain`, `multiplier`, and so on.
7. **Update.** `update(dt)` moves the player, handles fire and spawns, runs collisions, and advances stage progression. `updateBoss(dt)` drives boss phases and patterns. Boss attacks are pure functions named `fire*` (for example `fireRadial`, `fireAimedSpread`), selected through `BOSS_PATTERNS`, which is indexed per boss design.
8. **Draw.** `draw()` renders sky gradient, parallax clouds (`drawClouds`), sun glow, enemies, bosses, bullets, power-ups, particles, and the screen-shake transform. Ship shapes come from `drawPlane` and `drawBossShip`.
9. **Loop.** `loop(now)` caps `dt` at 0.05 s, calls `update` and `draw` only while `running && !paused`, and reschedules with `requestAnimationFrame`.
10. **Persistence.** `localStorage` only, wrapped in nothing: `bestScore`, `muted`, `difficulty`, `reducedMotion`. Reads use `Number(...) || 0` style fallbacks.

## Controls

- Move: arrow keys or WASD (via `keys`), mouse pointer over canvas, or the touch joystick.
- Fire: `Space`, held, or the fire button.
- Bomb: `X` or `Shift`, or the bomb button. Bombs clear the screen and damage bosses.
- Pause: `P` or `Escape`.

## Game rules worth knowing

- Stages advance after `KILLS_PER_STAGE` kills, then a boss spawns. Defeating the boss increments `stage` and changes the theme.
- Enemy types unlock by stage through `minStage`.
- Score is multiplied by a kill-chain `multiplier` that resets via `breakChain()`.
- Power-ups (`applyPowerUp`) are extra life, bomb, and weapon. A weapon power-up raises `player.level` up to 3 (wider spreads). Past level 3 it switches `player.weapon` between `vulcan` and `laser` and resets the level to 1.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. Do not add a bundler, framework, or dependency for a change that fits in the existing file.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, short arrow functions for handlers, and terse one-line object literals for config tables.
- Comments are sparse. Add one only where the logic is non-obvious (for example, the `dt` cap or the quality thresholds).
- Keep user-facing text in the HTML/overlay copy and the `announce()` calls, which feed the screen-reader live region (`#srLive`). Keep new HUD elements accessible.
- Preserve mobile behavior. The layout scales to the viewport (`resize()`, `BASE_W` × `BASE_H` = 480 × 720) and must work at phone width without horizontal scroll.
- Respect `prefers-reduced-motion`. Screen shake, flashes, and particles are gated by `reducedMotion`. New visual effects should honor it.

## Verifying changes

There is no automated test. After a change:

1. Open `index.html` in a browser, or serve the directory statically.
2. Start a run on each difficulty and check that the HUD, combo popup, and stage banner update.
3. Reach a boss (or temporarily lower `KILLS_PER_STAGE` locally, without committing that change) and confirm the boss bar and patterns still work.
4. Check pause (`P`/`Escape`), mute, and the motion toggle, and reload to confirm `localStorage` persistence.
5. For layout changes, check a narrow viewport and the touch controls.

Do not commit temporary tuning values such as a reduced kill count.

## Git

- Develop on the branch assigned to the session. Current development branch: `claude/gifted-tesla-b7xp3q`. `main` is the base.
- Commit messages are imperative and descriptive, with a short body when the change is more than a tweak. The existing history uses a summary line followed by bullet points for feature commits.
- Do not open pull requests unless explicitly asked.

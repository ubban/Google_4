# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single HTML5 canvas page.

- `index.html` — the entire game: inline CSS, markup, and JavaScript (~1,670 lines).
- `README.md` — a one-line title. Keep it accurate if the project description changes.

There is no build step, no package manifest, no dependencies, and no test suite. Do not add a framework, bundler, or external script without the user asking for it.

## Running the game

Open `index.html` directly in a browser. No server is needed.

To check changes, load the page in a browser and play through the affected behavior. Also check the layout at phone width, because the game has touch controls. There are no automated tests to run.

## Controls

| Input | Action |
| --- | --- |
| Mouse move / touch drag | Steer the ship |
| Click / hold, Space, or FIRE button | Fire |
| Arrows / WASD | Move |
| X or Shift, or BOMB button | Smart bomb |
| P or Esc | Pause |

The control hint shown on the title screen is the `#controlsHint` element. Update it whenever a control changes.

## Code layout of `index.html`

- **`<style>`** — all styling. The HUD is pill-shaped, with heart-style lives and bomb indicators. The overlay is a glassmorphic card.
- **Markup** — `#gameCanvas` plus HUD and overlay elements, including `#bossName` and `#controlsHint`.
- **`<script>`** — all game logic, in one IIFE-style block, organized as:
  1. **Config constants** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS` (one attack pattern set per boss design).
  2. **Persistence** — `STORAGE_*` keys in `localStorage` (see below).
  3. **Audio** — `ensureAudio`, `beep`, and `noiseBurst`, built on the Web Audio API. Audio starts only after a user gesture.
  4. **Input** — keyboard listeners, plus joystick and button handling for touch.
  5. **Game state and loop** — `resetGame`, `update(dt)`, `updateBoss`, `draw()`, and the `requestAnimationFrame` `loop(now)`.

Game state is held in module-level `let` variables. The canvas is sized by `resize()` to a `480 × 720` base resolution (`BASE_W`, `BASE_H`).

## Gameplay systems to know about

- **Difficulty** (`easy` / `normal` / `hard`) sets spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier. Change balance in `DIFFICULTIES`, not in the spawn or update code.
- **Stages** advance after `KILLS_PER_STAGE` (18) kills. Each stage ends in a boss. `BOSSES` and `THEMES` cycle with `(stage - 1) % length`.
- **Weapons** — Vulcan and Laser, each with 3 power levels (`fireWeapon`, `weaponLabel`). Power-ups (`applyPowerUp`, `maybeSpawnPowerUp`) include weapon level, bomb, and extra life.
- **Bombs** clear the screen and damage bosses (`useBomb`).
- **Score** uses a kill-chain multiplier (`addScore`, `breakChain`).
- **Best score** is persisted through `endGame()`.

## Persistence

Only these `localStorage` keys are used. Reuse them rather than inventing new keys, and keep reads and writes defensive (the page should still work if storage is blocked):

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Reduced motion turns off screen shake and similar effects. Respect it in any new visual effect.

## Conventions

- Vanilla JavaScript and CSS only. Match the existing style: 2-space indentation, `const`/`let`, arrow functions for listeners, and short comments only where the logic is not obvious.
- Use the existing naming. Constants are `UPPER_SNAKE_CASE`, functions are `camelCase`, and storage keys are prefixed with `skyfighter.`.
- Keep the page mobile-friendly. Touch targets, the joystick, and the layout must work at phone width, and touch handlers call `preventDefault` where needed to avoid scrolling.
- Keep changes scoped. Balance tweaks belong in the config constants, and a new feature should fit the existing update/draw loop rather than adding a parallel one.

## Git

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-v76uhi`). Do not push to `main` without explicit permission.
- Write clear commit messages that describe the gameplay or UI change.
- Do not open a pull request unless the user asks for one.

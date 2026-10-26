# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free front-end project. Its only app is **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on a single HTML5 canvas.

- `index.html` is the entire game: markup, CSS, and JavaScript in one file (~1,670 lines).
- `README.md` is a placeholder with the project name only.

There is no build step, package manager, test suite, or linter configured.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- Nothing needs to be installed.

## Code layout inside `index.html`

- **`<style>` (about lines 10–359):** CSS for the HUD (pill-style hearts and bombs), overlay card, boss bar, combo and stage banners, touch controls, and the reduced-motion and low-quality modes. Colors and layout live here, not in JS.
- **`<body>` markup (about lines 362–420):** `#container` holds the `#gameCanvas` canvas, the HUD (`#hud`, `#topBar`, `#bossBarWrap`), pop-up layers (`#comboPop`, `#stageBanner`), touch controls (`#touchControls` with a joystick and fire/bomb buttons), and `#overlay` for the title, pause, and game-over screens.
- **`<script>` (about lines 422–1667):** one IIFE wrapped in `'use strict'`. Sections are marked with `// ---------- Name ----------` comments. Key areas, in file order:
  - Config constants: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
  - Persistence keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`.
  - Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers, built on the Web Audio API.
  - Input: keyboard listeners, joystick and touch handlers, and mouse handlers.
  - Game state and loop: `resetGame`, `update(dt)`, `draw()`, and `loop(now)`, driven by `requestAnimationFrame`.
  - Adaptive quality: `trackPerf` lowers effects quality when frame time is high.

## Gameplay reference

These values are defined in the script and are the first place to look when tuning:

| Area | Where | Notes |
| --- | --- | --- |
| Difficulty | `DIFFICULTIES` | `easy`, `normal`, `hard` set spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier. |
| Stages | `KILLS_PER_STAGE`, `THEMES` | Killing 18 enemies in a stage triggers a boss. Themes cycle with `(stage - 1) % THEMES.length`. |
| Enemies | `ENEMY_TYPES` | `grunt`, `interceptor` (weaving), and `tank` (4 HP, from stage 2). |
| Bosses | `BOSSES`, `BOSS_PATTERNS` | Six bosses, cycled by stage. Each has its own HP multiplier and attack patterns. |
| Power-ups | `maybeSpawnPowerUp`, `applyPowerUp` | Life (6%), bomb (8%), weapon (20%). Weapon upgrades go level 1→3, then toggle between Vulcan and Laser. |
| Weapons | `fireWeapon` | Vulcan spreads bullets by level. Laser fires paired lanes at level 2 and adds a piercing center lane at level 3. |

## Controls

- **Keyboard:** Arrow keys or WASD to move. Space to fire. `X`, `Shift` to use a bomb. `P` or `Escape` to pause.
- **Touch:** Joystick in the left zone, fire button, and bomb button.
- **Mouse:** Move over the canvas to steer. Hold to fire.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not add a build tool or npm dependency without asking.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, `camelCase` names, `UPPER_SNAKE_CASE` for constants, and short section-header comments.
- Put new tunable numbers in the config objects near the top of the script rather than inline in logic.
- Store user preferences only under the `skyfighter.*` localStorage keys. Wrap storage reads and writes so the game still works if storage is blocked.
- Preserve the accessibility hooks: the `#srLive` live region (`announce()`) and the `prefers-reduced-motion` handling.
- Keep the mobile layout working. The viewport meta disables user zoom, and touch controls must still be reachable at phone widths.

## Verification

No automated tests exist. For gameplay or UI changes:

1. Open `index.html` in a browser and start a run on each difficulty.
2. Check the HUD, pause, game over, and restart flow.
3. Exercise both keyboard and touch controls if the change touches input.
4. If you change boss or weapon logic, reach a boss fight, for example by temporarily lowering `KILLS_PER_STAGE` locally. Do not commit that change.

## Git workflow

- Feature work for this task goes on the designated branch, `claude/gifted-tesla-3b27ox`.
- Use clear, present-tense commit messages that describe the gameplay or code change.
- Do not open a pull request unless the user asks for one.

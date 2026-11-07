# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-em-up played in the browser on an HTML5 `<canvas>`.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS (`<style>`, ~lines 10–362), and one inline `<script>` (~lines 422–end). |
| `README.md` | Placeholder title only. |

There is no build system, package manager, test suite, or linter. Do not add one unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the folder with any static server (for example `python3 -m http.server`).
- Verify changes by playing the game in a browser. Check at least: title overlay, start, firing, a bomb, pause (`P`/`Esc`), game over, and the best-score update.
- Check mobile layout as well when touching UI or touch controls. The game has a touch joystick, fire and bomb buttons, and a responsive canvas (`resize()`).

## Code architecture (`index.html`)

The game script is a single IIFE (`(function () { 'use strict'; ... })();`). Everything is private to it; nothing is exposed on `window`.

Major sections, in the order they appear in the script:

1. **DOM references**: `const` lookups by id (`gameCanvas`, `overlay`, `hearts`, `bombs`, `bossBarFill`, and so on). Keep new element references here.
2. **Storage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`.
3. **Config tables**:
   - `DIFFICULTIES`: spawn rate, enemy speed, enemy shoot chance, starting `lives`, starting `bombs`, and `bossHpMul` for `easy`, `normal`, and `hard`.
   - `KILLS_PER_STAGE` (18): kills needed before a stage's boss.
   - `THEMES`: sky gradient, cloud, and sun colors, plus a name, cycled per stage.
   - `ENEMY_TYPES` and `BOSSES`: enemy and boss definitions.
   - `BOSS_PATTERNS`: bullet patterns. Each `fire*` helper (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) is one pattern.
4. **Audio**: `ensureAudio()` lazily creates the `AudioContext` (browsers require a user gesture). `beep()` and `noiseBurst()` synthesize all sound effects. There are no audio files.
5. **HUD and UI helpers**: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`, and `announce()` (screen-reader live region).
6. **Input**: keyboard handlers on `window`, pointer handling via `canvasPointFromEvent`, and touch joystick functions (`joystickStart`/`Move`/`End`).
7. **Game state**: `resetGame()` initializes all state. `update(dt)` advances the simulation. `draw()` renders each frame. `loop(now)` drives both through `requestAnimationFrame`, with `dt` clamped to 0.05 s.
8. **Game events**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `addScore`, `breakChain` (kill-chain multiplier), `useBomb`, `fireWeapon`, `loseLife`, `killBoss`, `endGame`, `togglePause`.

### Controls

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrow keys / WASD (via `keys[e.code]`) | Left-side joystick |
| Fire | `Space` (held) | Fire button |
| Bomb | `X` or `Shift` | Bomb button |
| Pause | `P` or `Esc` | — |

Keyboard state is tracked by `e.code`, not `e.key`, so keep bindings layout-independent.

### Game rules to keep consistent

- Weapons: `vulcan` (spread at levels 1–3) and `laser`. Weapon type is `player.weapon`; weapon level is `player.level`. A power-up toggles the weapon type.
- Lives are hearts; bombs clear the screen and damage bosses.
- Score uses a kill-chain multiplier that resets via `breakChain()`.
- A boss appears after `KILLS_PER_STAGE` kills; `stage` advances after the boss dies (`killBoss`).
- Best score is written to `localStorage` on game over, and only when the final score beats the stored best (`endGame`).

## Conventions

- **Single file.** Keep the game in `index.html`. Do not split out modules, bundlers, or external assets unless the user asks.
- **No dependencies.** Use plain JavaScript and browser APIs. Do not load CDN libraries.
- **Style.** Match the existing code: 2-space indentation, single quotes, `const`/`let`, arrow functions for callbacks, and `'use strict'` inside the IIFE.
- **Naming.** Use camelCase for functions and variables. Use `UPPER_SNAKE_CASE` for tuned constants and `skyfighter.*` for storage keys.
- **Tuning.** Put gameplay numbers in the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`) rather than hard-coding them in logic.
- **Accessibility.** Respect `reducedMotion` for screen shake and flashes. Announce important state changes through `announce()`.
- **Persistence.** Wrap new `localStorage` reads and writes so the game still works when storage is unavailable (private mode, blocked site data).

## Git workflow

- Develop on the branch assigned to the session (currently `claude/gifted-tesla-8cdla2`). Do not push to other branches without permission.
- Commit with clear, descriptive messages. Do not create a pull request unless explicitly asked.
- Before committing, confirm that the game still loads without console errors and that the changed behavior works.

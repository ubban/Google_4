# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, Raiden-style vertical shoot-'em-up browser game.

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` (canvas rendering, game loop, input, audio, HUD).
- `README.md` — minimal placeholder.

There is no build step, package manager, bundler, test suite, or linter. Do not add one unless asked. To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Repository layout

```
index.html   # complete game (CSS + HTML + JS)
README.md    # placeholder
CLAUDE.md    # this file
```

## Code structure (inside `index.html`)

Roughly in file order within the `<script>` block:

| Area | Key identifiers | Notes |
|------|-----------------|-------|
| Persistence | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` | `localStorage` keys prefixed `skyfighter.` |
| Performance | `trackPerf`, `quality` (0 full / 1 medium / 2 low) | Adapts visual detail to frame time |
| Tuning tables | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` | Main balance knobs |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `sfx` | Web Audio API, created lazily on first user gesture |
| HUD | `renderHearts`, `renderBombs`, `showStageBanner`, `showComboPop` | DOM-based HUD over the canvas |
| Layout | `BASE_W = 480`, `BASE_H = 720`, `resize()` | Canvas scales to fit viewport |
| Input | `keys` (keyed by `KeyboardEvent.code`), `joystick*`, `firing`, `canvasPointFromEvent` | Keyboard, touch joystick, fire button, mouse |
| State | `player`, `bullets`, `enemies`, `powerUps`, `score`, `lives`, `bombCount`, `stage`, `chain`, `multiplier` | Module-level `let` variables, reset by `resetGame()` |
| Simulation | `update(dt)`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `loseLife`, `endGame` | Frame-rate-independent via `dt` |
| Boss patterns | `BOSS_PATTERNS`, `updateBoss`, `killBoss`, `fire*` helpers | Each boss cycles through bullet patterns |
| Rendering | `draw()`, `drawPlane`, `drawBossShip`, `drawClouds` | Canvas 2D |
| Loop | `loop(now)` | `requestAnimationFrame` driver |

All game code lives inside a single `(function () { ... })();` wrapper in the `<script>` block, so nothing leaks to `window`. Keep new code inside that wrapper.

## Controls (for reference when changing input)

- **Move:** Arrow keys or WASD; touch joystick on the left side of the screen.
- **Fire:** Space (held), or the on-screen fire button / mouse button held.
- **Bomb:** X or Shift, or the on-screen BOMB button (touch and click).
- **Pause:** P or Escape.
- **Mute, reduced motion, difficulty:** on-screen buttons; mute, difficulty, and reduced motion are persisted to `localStorage`.

## Game design facts

- Difficulties: `easy`, `normal` (default), `hard` — they change lives, starting bombs, spawn rate, enemy speed, enemy fire chance, and boss HP multiplier.
- Each stage ends after `KILLS_PER_STAGE` (18) kills; the boss spawns about 1.2 s later. `BOSSES` and `THEMES` each have six entries.
- Enemy types: `grunt`, `interceptor` (weaving), `tank` (4 HP, unlocked at stage 2).
- Power-ups: `life`, `bomb`, `weapon`. Weapon upgrades go level 1→3, then toggle between `vulcan` and `laser`.
- Score uses a chain/combo multiplier that resets when the player takes a hit (`breakChain`).
- The best score is saved to `localStorage` at game end.

## Conventions and guidance for changes

- **Keep it a single file** unless the user asks for a split. Preserve the existing `<style>` / markup / `<script>` order.
- **Balance changes** belong in the tuning tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coded numbers in logic, where practical.
- **Frame-rate independence:** movement and timers should use `dt` (seconds) as the existing code does.
- **Touch and desktop both matter.** Any new control needs a keyboard path and a touch/pointer path, and must not break `touch-action: none` handling.
- **Accessibility:** respect the `reducedMotion` setting for new screen-shake, flash, or heavy animation. Announce important state changes through the existing `announce()` helper.
- **Audio** must be created only after a user gesture (`ensureAudio()`), and respect `muted`.
- **Responsive layout:** the game must work at phone widths. Keep the viewport meta and `touch-action` settings intact.
- **Persistence:** `localStorage` reads and writes currently are not wrapped in try/catch. If you touch them, consider guarding them so the game still runs when storage is blocked.
- **No external dependencies.** The game loads no scripts or stylesheets from CDNs; keep it that way.

## Verifying changes

There are no automated tests. For any gameplay or UI change:

1. Open `index.html` in a browser and check the affected behavior directly.
2. Check at least one desktop viewport and one narrow (phone-width) viewport.
3. Confirm the browser console has no new errors during play, pause, game over, and restart.
4. Check that the best score, mute, difficulty, and reduced-motion settings still persist across reloads.

## Git workflow

- Develop on the branch specified by the session/task. Do not push to other branches without permission.
- Write clear, descriptive commit messages that explain *why* the change was made.
- Do not open a pull request unless explicitly asked.

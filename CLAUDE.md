# CLAUDE.md

Guidance for working in this repository.

## Project overview

Google_4 contains **Sky Fighter**, a Raiden-style vertical shoot-em-up written as a single-page HTML5 canvas game. The whole game lives in `index.html`: inline CSS in `<style>`, markup in `<body>`, and one IIFE-wrapped `<script>` block. `README.md` is a placeholder.

There is no build step, package manifest, bundler, test suite, or linter. Do not add one unless asked.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- Verify changes by playing them in a browser. There are no automated checks to run.

## File layout

| Path | Purpose |
| --- | --- |
| `index.html` | The entire game: styles, HUD/overlay/touch markup, and game logic |
| `README.md` | Placeholder |
| `CLAUDE.md` | This file |

## Architecture of `index.html`

Structure of the script (all inside one `(function () { ... })();`):

1. **DOM refs and constants**: element lookups (`canvas`, HUD, overlay, touch controls), `STORAGE_*` keys, `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
2. **Persistence**: best score, mute, difficulty, and reduced-motion are stored in `localStorage` under `skyfighter.*` keys. All reads are wrapped as `Number(localStorage.getItem(...)) || default` style fallbacks; keep that pattern.
3. **Audio**: a lazily created `AudioContext` (`ensureAudio()`) with a compressor and master gain. Sounds are built from `beep()` (oscillators) and `noiseBurst()` (filtered noise) and exposed through the `sfx` object. Audio must start from a user gesture, so `ensureAudio()` is called on key, touch, and click handlers.
4. **Rendering helpers**: `drawPlane`, `drawBossShip`, cloud layers, sky gradients, particles, and `screenShake`/`flashHit` for feedback. Drawing goes through a 2D context at a fixed logical size (`BASE_W = 480`, `BASE_H = 720`); `resize()` scales the canvas to fit the window (capped at 1.6x).
5. **Input**: keyboard (`keys` map keyed by `e.code`), touch joystick plus fire/bomb buttons (`touchControls`), and mouse aim on the canvas (`mouseTarget`). Touch is detected with `isTouchDevice`.
6. **Game state**: module-level `let` variables (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`, `chain`, `multiplier`, ...). `resetGame()` initializes them.
7. **Update/draw loop**: `loop(now)` is driven by `requestAnimationFrame`. `dt` is clamped to 0.05s. `update(dt)` runs only when `running && !paused`, then `draw()`.
8. **Adaptive quality**: `trackPerf()` keeps a rolling frame-time average and steps `quality` (0 full, 1 medium, 2 low) up or down. Particle counts and glow effects read `quality`.

### Game systems

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal`, `hard`. Each sets spawn rate, enemy speed, enemy shoot chance, starting lives, starting bombs, and a boss HP multiplier.
- **Stages**: after `KILLS_PER_STAGE` (18) kills in a stage, the boss warning fires (`bossWarned`) and a boss spawns. Killing the boss (`killBoss()`) advances the stage. Each stage uses `THEMES[(stage - 1) % THEMES.length]`. Enemy types unlock by `minStage`.
- **Enemies**: `ENEMY_TYPES` (grunt, interceptor, tank) with HP, speed, color, and score value. Spawning is in `spawnEnemy()`.
- **Bosses**: `spawnBoss()` picks `BOSSES[(stage - 1) % BOSSES.length]`. Each boss has an attack pattern list in `BOSS_PATTERNS`, indexed by the same design index. `updateBoss()` cycles through them; `killBoss()` handles the reward.
- **Weapons**: `fireWeapon()` implements Vulcan and Laser at levels 1 to 3, with spread and lane offsets that depend on `player.level`.
- **Bombs**: `useBomb()` clears enemy bullets, destroys all on-screen enemies (small score bonus each), and deals 24 damage to the boss. Count is set per difficulty and capped by `maxBombs`.
- **Power-ups**: `maybeSpawnPowerUp()` drops `life`, `bomb`, or `weapon` pickups. `applyPowerUp()` applies them.
- **Scoring**: `addScore()` multiplies base points by `multiplier`, which grows with the kill `chain`. `breakChain()` resets it. `loseLife()` and `endGame()` handle damage and game over.

### Controls

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrow keys or `A`/`D`/`W`/`S` | Left-side joystick |
| Fire | `Space` (hold) | Fire button (hold) |
| Bomb | `X` or `Shift` | Bomb button |
| Pause | `P` or `Esc` | Pause button |
| Aim | n/a | Mouse over canvas moves the ship |

## Conventions

- **Single file.** Keep HTML, CSS, and JS in `index.html` unless the user asks for a split. Do not introduce frameworks or dependencies.
- **Style.** Match the surrounding code: 2-space indent, single quotes in JS, `const` by default, `let` for mutable game state, arrow functions for callbacks, and short helper functions with a leading comment only where the intent is not obvious.
- **Naming.** Constants in `UPPER_SNAKE_CASE` (`DIFFICULTIES`, `BOSS_PATTERNS`). Storage keys use the `skyfighter.` prefix.
- **New content goes in tables.** Add enemies, bosses, themes, or difficulty settings as entries in the existing `ENEMY_TYPES`, `BOSSES`, `THEMES`, and `DIFFICULTIES` objects. Add matching boss attack patterns to `BOSS_PATTERNS` at the same index as the boss.
- **Frame-rate independence.** Scale movement and timers by `dt`, as the existing code does. Do not hard-code per-frame increments.
- **Accessibility.** Respect `reducedMotion` for screen shake, flashes, and similar effects. Announce important state changes through the `announce()` helper, which feeds the `srLive` live region.
- **Mobile.** Keep the layout working at phone width and with touch controls. Test touch changes with touch emulation.
- **No secrets or external requests.** The game is self-contained. The only URL in the file is an inline SVG favicon data URI.

## Verification checklist

Before committing gameplay or UI changes, open the game in a browser and check:

1. Title overlay appears; Start begins a run; difficulty buttons persist across reloads.
2. Movement, firing, bombs, and pause work on keyboard. Touch controls work under touch emulation.
3. Weapon power-ups upgrade Vulcan through level 3 and Laser works.
4. After 18 kills the boss warning appears and a boss spawns; its health bar updates.
5. Killing the boss advances the stage (stage banner shows, theme changes).
6. Game over shows the score, stage, and best; a new best is saved to `localStorage`.
7. Mute and reduced-motion toggles persist and take effect.
8. No console errors.

## Git workflow

- Develop on the branch named in the session instructions (currently `claude/gifted-tesla-6lzmtc`). Do not push to `main` directly.
- Write clear commit messages that describe the gameplay or code change.
- Do not open a pull request unless explicitly asked.

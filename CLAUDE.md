# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The player flies a plane up a scrolling sky, switches weapons, drops smart bombs, and fights a boss at the end of each stage.

The entire game lives in one file, `index.html` (~1,670 lines): CSS in a `<style>` block, markup, and one `<script>` block. There is no build step, no package manager, no framework, no external assets, and no test suite. Audio is synthesized with the WebAudio API.

## Repository layout

| Path         | Purpose                                              |
| ------------ | ---------------------------------------------------- |
| `index.html` | The whole game: markup, styles, and game logic.      |
| `README.md`  | Minimal project readme (title only).                 |
| `CLAUDE.md`  | This file.                                           |

## Running the game

No build is required. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Code structure inside `index.html`

The `<script>` block is organized into labeled sections (marked with `// ---------- ... ----------` comments):

- **Persistence and settings**: `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`).
- **Adaptive performance**: `trackPerf(dt)` lowers a `quality` level (0 full, 1 medium, 2 low) when frame times are poor.
- **Config tables**: `DIFFICULTIES` (easy/normal/hard tuning for spawn rate, enemy speed, shot chance, lives, bombs, boss HP), `KILLS_PER_STAGE`, `THEMES` (six sky palettes cycled per stage), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six boss designs, indexed by `(stage - 1) % BOSSES.length`).
- **Audio**: `ensureAudio()`, `beep()`, `noiseBurst()`, and the `sfx` helpers. Audio is created lazily on the first user gesture.
- **HUD**: `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `flashHit`.
- **Canvas sizing**: `resize()` fits a `BASE_W x BASE_H` (480 x 720) logical canvas into the viewport.
- **Input**: keyboard (`keys[]`), touch joystick (`joystickStart/Move/End`), fire and bomb buttons, and mouse steering that glides the plane toward the pointer. Keyboard and joystick take priority over mouse steering.
- **Game state**: globals such as `score`, `lives`, `stage`, `chain`, `multiplier`, `player`, and the entity arrays. `resetGame()` restores the initial state.
- **Entities and combat**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`, `fireWeapon`, `useBomb`, `addScore`/`breakChain`.
- **Boss patterns**: `BOSS_PATTERNS` (one per boss) built from the `fire*` helpers (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`). `updateBoss`, `killBoss`.
- **Main loop**: `update(dt)` (logic), `draw()` (rendering), and `loop(now)` (requestAnimationFrame driver).

### Key game rules

- **Controls**: Arrow keys / WASD move; Space fires; `P` or `Escape` pauses; `X` or Shift drops a bomb. Touch and mouse controls are also supported.
- **Power-ups** (`applyPowerUp`): `life` (+1 life, capped), `bomb` (+1 bomb, capped), `weapon` (raises weapon level to 3, then toggles between `vulcan` and `laser` and resets to level 1).
- **Chain and multiplier**: kills build a chain that raises the score multiplier; the chain breaks when the player is hit.
- **Stages**: a stage ends after `KILLS_PER_STAGE` kills, followed by a boss fight.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. If a split is ever needed, update this file's layout section.
- Do not add external script, style, or font dependencies. The game is intentionally self-contained and works offline.
- Preserve the existing section-comment style and the naming pattern of the config tables (`UPPER_SNAKE_CASE` for constants, `camelCase` for functions and state).
- Keep the game playable on touch devices and small screens. The layout relies on `touch-action: none`, safe-area insets, and `clamp()` font sizes; don't break these.
- Respect the `reducedMotion` setting when adding new screen-shake, flash, or animation effects.
- Respect the `muted` setting for any new sound effects by routing them through the existing `beep` / `noiseBurst` helpers.
- Use the existing `DIFFICULTIES` table for any new tuning value that should vary by difficulty.

## Testing and verification

There is no automated test suite or linter. To verify a change:

1. Open `index.html` in a browser (desktop and, if possible, a narrow mobile viewport).
2. Start a game, and check movement, firing, bombs, power-ups, and the pause/resume flow.
3. Reach a boss fight (or temporarily lower `KILLS_PER_STAGE` locally) and confirm boss patterns and boss-defeat transitions.
4. Check the game-over flow and that the best score persists across reloads.
5. Confirm there are no errors in the browser console.

## Git

- Default development happens on the branch specified by the current task; do not push to other branches without permission.
- Write clear, descriptive commit messages that explain *why* a change was made.

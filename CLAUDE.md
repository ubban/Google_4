# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up. Switch weapons, drop smart bombs, and take down stage bosses.

The whole game lives in a single file:

- `index.html` (~1,670 lines): inline `<style>`, markup, and one inline `<script>` (an IIFE that holds all game state and logic). There is no build step, no package manager, and no test suite.
- `README.md`: a placeholder title only.

## Running the game

No build or install is needed. Either:

- open `index.html` directly in a browser, or
- serve the directory with any static server, e.g. `python3 -m http.server` and visit `http://localhost:8000`.

Touch devices get an on-screen joystick and fire/bomb buttons (detected via `ontouchstart` / `navigator.maxTouchPoints`).

## Controls

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrow keys / WASD (via `keys[e.code]`); mouse-move steers | Drag left side (joystick) |
| Fire | Hold Space; hold mouse button | Fire button |
| Smart bomb | `X` or either Shift | Bomb button |
| Pause | `P` or `Esc` | — |

## Code layout (inside `index.html`)

- **Persistence**: `localStorage` keys are prefixed `skyfighter.` — `bestScore`, `muted`, `difficulty`, `reducedMotion`. Every access should keep working if storage is unavailable or unset; the code already falls back to defaults.
- **Tunable config** (top of the script):
  - `DIFFICULTIES` — `easy` / `normal` / `hard`, controlling spawn rate (`spawnBase`, `spawnMin`), `enemySpeed`, `enemyShootChance`, starting `lives`, starting `bombs`, and `bossHpMul`.
  - `KILLS_PER_STAGE` — kills needed to advance a stage (currently 18).
  - `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, `EXPLODE_SFX` — stage art/palette, enemy stats, boss definitions, boss attack patterns, and explosion sound settings.
  - `BASE_W = 480`, `BASE_H = 720` — logical playfield size; `resize()` scales the canvas to fit the viewport.
- **Game loop**: a single `requestAnimationFrame` loop (`loop`). Delta time is clamped to 50 ms. Each frame runs `trackPerf(dt)`, then `update(dt)` and `draw()` when the game is running and not paused.
- **Player weapons**: `fireWeapon` fires either `vulcan` or `laser` (`player.weapon`), with fire delay 0.15 s / 0.1 s. Weapon power-ups in `applyPowerUp` raise `player.level` up to 3; past level 3 they toggle the weapon type and reset the level to 1. `weaponLabel` drives the HUD.
- **Boss attack patterns**: the helpers `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireDoubleRing`, and `fireHomingSeeded` are used by the `BOSS_PATTERNS` entries, not by the player.
- **Bosses**: `spawnBoss`, `updateBoss`, `drawBossShip`, `killBoss`. A boss warning starts once `stageKills` reaches `KILLS_PER_STAGE`; the next boss spawns about 1.2 s later.
- **Audio**: Web Audio API, created lazily in `ensureAudio()` (browsers require a user gesture first). Sounds are synthesized (`beep`, `noiseBurst`); `muted` toggles output.
- **HUD**: `renderHearts`, `renderBombs`, `weaponLabel`, `announce`, `showStageBanner`, `showComboPop`. Chain/combo logic lives in `addScore` and `breakChain`.
- **Feedback effects**: `screenShake`, `flashHit`, `flashBomb`, `spawnExplosion`. Respect the `reducedMotion` setting when adding new effects.
- **Lifecycle**: `resetGame` (new run), `loseLife`, `endGame`, `togglePause`, `setDifficulty`, `updateMuteBtn`, `updateMotionBtn`.

## Conventions

- Keep the game self-contained: no external libraries, CDNs, or build tooling unless the user asks for them.
- Keep everything in `index.html` unless there is a clear reason to split it; if it is split, update this file's layout section.
- Preserve the existing style: 2-space indentation inside the script, `const`/`let`, plain DOM APIs, and IIFE scoping (don't leak globals).
- When adding a tunable value, put it in the existing config objects (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, etc.) rather than hard-coding it in the loop.
- Any new input must work on both keyboard and touch, and any new stored setting should use the `skyfighter.` key prefix.
- Check the game in a real browser after changes to gameplay, rendering, or input, especially on a narrow (phone-sized) viewport, since the layout and touch controls are sized for it.

## Git

- Development branch for this environment: `claude/gifted-tesla-ycb2ua`. Commit and push there; don't push to other branches without explicit permission.
- Don't open a pull request unless the user asks for one.

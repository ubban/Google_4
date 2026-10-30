# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only app is **Sky Fighter**, a
Raiden-style vertical shoot-em-up written as a single self-contained HTML5
canvas page.

- `index.html` (~1,670 lines): all markup, CSS, and JavaScript for the game.
- `README.md`: a placeholder with no setup instructions.

There is no build step, no package manager, no bundler, and no test suite.
Nothing in the repo runs `npm`, `pip`, or similar. Changes are verified by
opening `index.html` in a browser.

## Repository layout

```
index.html   Game: <style> (~lines 10-360), <body> markup (~361-421), <script> (~422-1668)
README.md    Placeholder
```

Inside the `<script>`, everything lives in one IIFE with `'use strict'`. The
main sections, in file order, are:

1. **DOM references and storage keys** (`STORAGE_*` constants, `localStorage`).
2. **Adaptive performance** (`trackPerf` drops render quality when frames are slow).
3. **Config tables**: `DIFFICULTIES`, `THEMES` (stage backgrounds), `ENEMY_TYPES`, `BOSSES`.
4. **Audio** (`ensureAudio`, `beep`, `noiseBurst`, the `sfx` object). Audio is generated with WebAudio, with no audio files.
5. **HUD and effects** (`renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`).
6. **Input** (keyboard, touch joystick, fire and bomb buttons, mouse on the canvas).
7. **Game state** (`resetGame`), **spawning** (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), and **update** (`update(dt)`).
8. **Weapons and bosses** (`fireWeapon`, `applyPowerUp`, the `BOSS_PATTERNS` array of bullet patterns, `updateBoss`, `killBoss`).
9. **Rendering** (`draw`, `drawPlane`, `drawBossShip`, `drawClouds`) and the main `loop(now)` driven by `requestAnimationFrame`.

## Game rules to keep in mind

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets
  spawn timing, enemy shoot chance, starting lives, starting bombs, and a boss HP
  multiplier.
- **Stages**: after `KILLS_PER_STAGE` (18) kills in a stage, a boss is spawned
  (about 1.2s later, see `update`). Stage number selects the theme, and `tank` enemies
  appear from stage 2. Bosses cycle through `BOSSES`.
- **Scoring**: enemies have base scores. A kill-chain multiplier (`addScore`,
  `breakChain`) raises points for consecutive kills.
- **Weapons**: `VULCAN` and `LASER` (`weaponLabel`), with three power levels. Weapon
  power-ups raise the level.
- **Power-ups**: `life`, `bomb`, and `weapon`. They are created in `maybeSpawnPowerUp`
  and applied in `applyPowerUp`.
- **Bombs** clear the screen and damage bosses (`useBomb`). Bomb input is `X`,
  either Shift key, or the on-screen bomb button.

## Controls

| Action        | Desktop                                   | Touch                          |
|---------------|-------------------------------------------|--------------------------------|
| Move          | Arrow keys / WASD, or mouse to steer      | Drag in the left zone (joystick) |
| Fire          | Space (hold), or hold mouse button        | Tap FIRE                       |
| Bomb          | `X` or Shift                              | Tap BOMB                       |
| Pause         | `P` or `Escape`, or the HUD pause button  | Pause button in HUD            |

Touch controls appear only when `isTouchDevice` is true.

## Persistence

Values are stored in `localStorage` under these keys. Reads and writes are
synchronous, so keep them cheap.

| Key                        | Meaning                                  |
|----------------------------|------------------------------------------|
| `skyfighter.bestScore`     | Best score across sessions               |
| `skyfighter.muted`         | `'1'` when audio is muted                |
| `skyfighter.difficulty`    | Selected difficulty name                 |
| `skyfighter.reducedMotion` | `'1'`/`'0'`; falls back to the OS preference |

Changing a key's name or meaning breaks saved player data. Treat renames as
migrations.

## Conventions

- **Single file.** Keep the game in `index.html`. Do not split out JS or CSS
  files, and do not add a build step, unless the user asks for it.
- **No dependencies.** Use vanilla JS and the Canvas 2D and WebAudio APIs. Do not
  add CDN scripts or libraries without asking.
- **Match the local style.** The code uses 2-space indentation, single quotes,
  `const` by default, arrow functions for callbacks, and short `// ----------
  Section ----------` banners between sections. Follow the same pattern for new code.
- **Keep the game loop allocation-light.** `update` and `draw` run every frame. Avoid
  creating new objects or closures inside hot loops where a reused value works.
- **Respect reduced motion and quality levels.** `screenShake` and the hit/bomb flashes
  check `reducedMotion`. Particle counts, shadow blur, and glow check `quality`, which
  `trackPerf` adjusts at runtime. Do not add unconditional heavy effects.
- **Accessibility.** Status changes go through `announce()` for screen readers.
  New overlays or buttons need labels and keyboard access.
- **Mobile layout.** The canvas scales to the viewport via `resize()`, using a base
  of 480x720. Test new UI at phone-sized viewports.

## Working on this repo

- Open `index.html` directly in a browser to run the game. There is no dev server.
- Before committing, check the game manually: start a run, fire, use a bomb,
  reach a boss, pause and resume, and game over. Check the browser console for errors.
- Keep changes focused. Do not reformat unrelated code, since the file is large and a
  diff that touches many lines is hard to review.

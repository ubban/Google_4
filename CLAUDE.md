# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a
single self-contained HTML5 canvas page. There is no build step, no package manifest,
and no test suite.

- `index.html` — the entire game: inline `<style>`, markup, and one IIFE-wrapped `<script>`.
- `README.md` — minimal repo title only.

To run it, open `index.html` in a browser (or serve the directory with any static server).

## Repository layout

```
index.html   # all HTML, CSS, and JS (~58 KB)
README.md
CLAUDE.md
```

Keep the game as a single file unless the user asks for a split. Adding a bundler,
framework, or external dependency would change how the project is run and deployed.

## Architecture (inside `index.html`)

The script is one IIFE with `'use strict'`. Sections follow this order. Only some have a
`// ---------- Name ----------` banner comment, so search for the function names when in doubt:

| Section | What it covers |
| --- | --- |
| DOM refs and constants | `getElementById` lookups; `STORAGE_*` keys; `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` |
| Adaptive performance | `trackPerf()` steps a `quality` level (0 full, 1 medium, 2 low) from a 45-frame average |
| Audio | Web Audio helpers `ensureAudio`, `beep`, `noiseBurst`, and an `sfx` object |
| HUD and effects | `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake` |
| Settings | Mute, reduced motion, and difficulty toggles, persisted to `localStorage` |
| Canvas sizing | `BASE_W × BASE_H` is 480 × 720. `resize()` scales the canvas to fit the window, capped at 1.6× |
| Input | Keyboard, touch joystick, fire and bomb buttons, and mouse steering on the canvas |
| Game state and flow | `resetGame`, `update(dt)`, `draw()`, `loop(now)`, `endGame`, `loseLife`, `togglePause` |
| Drawing | `drawPlane`, `drawBossShip`, `drawClouds` |
| Spawning | `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp` |
| Weapons and bombs | `fireWeapon`, `useBomb`, `applyPowerUp` |
| Boss patterns | `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, and the `BOSS_PATTERNS` table |

The game loop is `requestAnimationFrame(loop)`. `loop` calls `update(dt)` and then
`draw()`. Start-up runs `resize()`, reads storage, and calls `setDifficulty()`.

## Game rules and tuning

Balance values are data-driven. Prefer changing these constants over changing logic.

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), and `hard` each set
  `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, and `bossHpMul`.
- **Stages**: a stage ends after `KILLS_PER_STAGE` (18) kills, which triggers a boss.
  Boss HP is `(70 + stage × 35) × hpMul`. Bosses cycle through `BOSSES` by
  `(stage - 1) % BOSSES.length`. Each stage's visual theme comes from `THEMES`.
- **Enemies** (`ENEMY_TYPES`): `grunt` (stage 1+), `interceptor` (stage 1+, weaves),
  `tank` (stage 2+, 4 HP).
- **Scoring**: a kill chain builds while kills land within `chainTimer` (2.2 s).
  `multiplier = 1 + min(9, floor(chain / 5))`. Score adds `base × multiplier`.
- **Power-ups** (`maybeSpawnPowerUp`): roughly 6% extra life, 8% bomb, and 20% weapon.
  Weapon power-ups raise the level from 1 to 3, then toggle between `vulcan` and `laser`.
- **Lives and bombs**: the player starts with `lives` and `bombs` from the difficulty.
  A bomb clears the screen and damages bosses.

## Persistence

All persistence uses `localStorage` with the `skyfighter.*` prefix:

| Key | Value |
| --- | --- |
| `skyfighter.bestScore` | Number |
| `skyfighter.muted` | `'1'` or `'0'` |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | `'1'` or `'0'`. If unset, it falls back to `prefers-reduced-motion` |

Storage access is currently not wrapped in `try/catch`. Private-mode or blocked-storage
browsers can throw. If you touch these lines, consider adding a guard.

## Controls

- **Desktop**: mouse to steer and click to fire. Arrow keys or WASD to move. Space to fire.
  X or Shift to bomb. P or Esc to pause.
- **Touch**: drag the left zone to fly, tap FIRE, and tap BOMB. Touch controls appear
  only when `'ontouchstart' in window` or `navigator.maxTouchPoints > 0`.

## Accessibility

- A `#srLive` `aria-live` region announces key events through `announce()`.
- Reduced motion turns off screen shake, flash, and pulse effects through the
  `body.reduced-motion` class.
- The overlay and controls use real `<button>` elements with `title` attributes.

## Conventions

- Match the existing style: two-space indent, single quotes in JS, `const` by default,
  and short section-banner comments.
- Keep DOM lookups at the top of the IIFE. Keep tuning values in the named constant
  objects, not inline.
- Keep the code dependency-free and inline.
- Run game changes in a browser before reporting them done. There are no automated tests.
- Keep `index.html` self-contained, with no external CDN links or assets.

## Git workflow

- Develop on the branch the session names. Do not push to `main` unless asked.
- Write commit messages that describe the gameplay or code change, not the file edit.
- Do not open pull requests unless the user asks.

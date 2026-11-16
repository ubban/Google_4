# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up.

- The entire app lives in one file: `index.html` (inline `<style>` and one inline `<script>`, roughly 1,650 lines).
- `README.md` is a placeholder with no setup instructions.
- There is no build step, package manager, test suite, or linter. No `package.json` exists.

## Running and verifying

- Open `index.html` directly in a browser, or serve the folder with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser, ideally on a desktop viewport and a phone-sized viewport, because touch controls are a separate code path.
- Plain JavaScript only. Do not add a bundler, framework, or dependency unless the user asks for one.

## Code layout (`index.html`)

- **`<head>`**: metadata, a data-URI favicon, and the dark `color-scheme`. The viewport meta disables zoom, since the game is touch-driven.
- **`<style>`**: full-screen flex container around `#gameCanvas`, the pill-style HUD (`#hud`, hearts and bombs), the glassmorphic overlay card, and combo and stage-banner styles.
- **`<body>`**: the canvas, the HUD, overlay screens, and mobile controls (joystick, FIRE, BOMB buttons).
- **`<script>`** (starts near line 422), in rough order:
  - Constants and persistence keys (`STORAGE_*`, `DIFFICULTIES`, `KILLS_PER_STAGE`).
  - Audio: `ensureAudio`, `beep`, `noiseBurst`, all built on Web Audio.
  - HUD helpers: `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`.
  - Input: keyboard (`keydown`/`keyup`), mouse, touch joystick, and `canvasPointFromEvent`.
  - Game state: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `applyPowerUp`, `useBomb`, `fireWeapon`.
  - Boss patterns: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, `updateBoss`, `killBoss`.
  - Rendering: `draw()` with `drawPlane`, `drawBossShip`, and parallax `drawClouds`.
  - Main loop: `loop(now)` drives `requestAnimationFrame`.

## Game mechanics to preserve

- **Controls**: mouse to steer and click to fire; arrows or WASD to move; Space to fire; X or Shift for a bomb; P or Esc to pause. On mobile, drag the left side to fly, and tap FIRE or BOMB. Keyboard and joystick input take priority when active, so they never fight.
- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier. Changing one should be deliberate, because it tunes the whole game.
- **Stages**: `KILLS_PER_STAGE` kills advance the stage. Each stage ends with a boss battle that has a health bar and multiple attack patterns.
- **Weapons**: Vulcan and Laser, each with 3 power levels. Power-ups include weapon level, bomb, and extra life.
- **Scoring**: a kill-chain multiplier (`addScore`, `breakChain`).
- **Bombs** clear the screen and damage bosses.
- **Lives**: `loseLife` triggers a hit flash, screen shake, and a 1.5 s invulnerability window (`player.invuln`). `endGame` runs at zero lives.
- **Adaptive quality**: `quality` (0 full, 1 medium, 2 low) is lowered automatically from measured frame times (`trackPerf`) to keep the frame rate up on slow devices.

## Persistence

The game uses `localStorage` under these keys. Keep the names stable, because changing them would reset players' saved data:

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | High score |
| `skyfighter.muted` | Audio mute (`'1'` or `'0'`) |
| `skyfighter.difficulty` | Selected difficulty |
| `skyfighter.reducedMotion` | Reduced-motion preference (screen shake and similar effects) |

All storage access should tolerate being unavailable, such as in private browsing. Keep that guard in place when editing.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. If you do split files, update this section.
- Match the existing style: 2-space indentation, `const` and `let`, small named functions, and comments that explain intent rather than restate code.
- Use `dt`-based timing in `update` so the game runs at the same speed on different refresh rates.
- Preserve accessibility hooks: the `announce()` function (writes to the `#srLive` polite live region) and the reduced-motion option.
- Keep the layout working on phones (`touch-action: none`, safe-area insets, and resize and orientation handling). Do not add page scrolling.

## Git

- Development branch for this work: `claude/gifted-tesla-g9gs6b`.
- Commit messages follow the existing style: a short imperative summary line, then bullet points for detail.
- Do not create pull requests unless the user asks.

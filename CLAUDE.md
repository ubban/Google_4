# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game ships as a single self-contained file, `index.html` (~1,670 lines: inline CSS, markup, and one IIFE-wrapped `<script>`). There is no build step, no package manager, no bundler, and no test suite.

`README.md` is a placeholder and contains no setup instructions.

## Repository layout

| Path | Purpose |
| --- | --- |
| `index.html` | The whole game: styles (lines ~10–359), DOM markup (~361–421), and game logic (~422–1667). |
| `README.md` | Minimal placeholder. |
| `CLAUDE.md` | This file. |

There are no subdirectories, assets, or external dependencies. Audio is synthesized with the Web Audio API and graphics are drawn with the 2D canvas, so nothing needs to be downloaded.

## Running the game

No install is needed. Open the file directly, or serve the directory with any static server:

```bash
# Option 1: open directly
xdg-open index.html        # Linux
open index.html            # macOS

# Option 2: static server (useful if a browser restricts file:// features)
python3 -m http.server 8000
# then visit http://localhost:8000/index.html
```

Verify changes by playing in a browser, not only by reading the code. The Playwright/Chromium setup in the cloud environment can drive it headlessly.

## Architecture of `index.html`

### Structure
- **`<style>`**: All CSS is inline. Layout is a fixed full-viewport `#container` centering a `<canvas id="gameCanvas">`. The HUD (`#hud`), boss bar, touch controls, top bar, and overlay menu are absolutely positioned on top of the canvas.
- **Markup**: HUD stats (score, best, multiplier, weapon, hearts, bombs), `#bossBarWrap`, `#comboPop`, `#stageBanner`, `#topBar` (mute, reduced-motion, pause), `#touchControls` (joystick + FIRE/BOMB), and `#overlay` (title, difficulty buttons, Start button, controls hint).
- **Script**: One `(function () { 'use strict'; ... })();` IIFE. Nothing is exposed on `window`, so keep new code inside it.

### Game-logic sections (in rough file order)
1. **State and persistence**: `STORAGE_*` keys, `announce()` for screen-reader updates, and adaptive quality (`trackPerf` lowers quality when frame time is high).
2. **Config tables**: `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES` (six sky palettes that cycle per stage), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six bosses).
3. **Audio**: `ensureAudio()`, `beep()`, `noiseBurst()`, and the `sfx` table. Audio is created lazily on the first user gesture, and it respects the mute setting.
4. **Input**: Keyboard (`keys[e.code]`), mouse (move to steer, click to fire), and touch (joystick, FIRE, BOMB) handlers.
5. **Resize**: `resize()` fits a logical `480 × 720` play field (`BASE_W`, `BASE_H`) into the window.
6. **Entities and spawning**: `resetGame()`, `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`, `spawnExplosion()`.
7. **Weapons and bombs**: `fireWeapon()` (vulcan or laser), `useBomb()`.
8. **Update**: `update(dt)` is the per-frame simulation. It handles movement, firing, collisions, stage progression, and power-up pickups.
9. **Power-ups**: `applyPowerUp()` for `life`, `bomb`, and `weapon`. Weapon pickups raise the level from 1 to 3, then toggle between vulcan and laser and reset to level 1.
10. **Boss patterns**: `BOSS_PATTERNS` is an array of bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring). `updateBoss()` picks among them.
11. **Lifecycle**: `loseLife()` (1.5 s invulnerability, breaks the combo chain), `killBoss()`, `endGame()`, `togglePause()`.
12. **Rendering**: `draw()` and helpers `drawPlane()`, `drawBossShip()`, `drawClouds()`.
13. **Main loop**: `loop(now)` clamps `dt` to 0.05 s, runs `update` then `draw`, and calls `requestAnimationFrame`. The game starts at the bottom of the file with `requestAnimationFrame(loop)`.

### Game rules worth knowing before changing balance
- **Difficulty** (`DIFFICULTIES`): sets lives, starting bombs, spawn rate, enemy speed, enemy fire chance, and boss HP multiplier. `normal` is the default.
- **Stages**: Killing 18 enemies (`KILLS_PER_STAGE`) triggers a boss, which spawns 1.2 s later. Defeating the boss increments `stage` and changes the theme. Tanks appear from stage 2.
- **Combo and multiplier**: `addScore()` increases the chain; `breakChain()` resets it when the player is hit.
- **Best score**: Saved to `localStorage` under `skyfighter.bestScore`.

### Persistence keys (`localStorage`)
| Key | Values |
| --- | --- |
| `skyfighter.bestScore` | number |
| `skyfighter.muted` | `'1'` or `'0'` |
| `skyfighter.difficulty` | `easy`, `normal`, `hard` |
| `skyfighter.reducedMotion` | `'1'` or `'0'` (defaults to the OS `prefers-reduced-motion` setting) |

Renaming these keys resets players' saved data, so change them only deliberately.

## Conventions

- **Keep it single-file.** The project is deliberately dependency-free. Do not add a bundler, framework, or `package.json` unless the user asks for one.
- **Style**: 2-space indentation, single quotes in JS, `const` by default, and `let` only for reassigned state. Existing code uses section comments like `// ---------- Name ----------`; follow that pattern.
- **Frame-rate independence**: Game updates should use `dt` (seconds), not per-frame constants. Speeds in the config tables are per-second values scaled by `dt`.
- **Accessibility**: Respect `reducedMotion` for screen shake, flashes, and pulses. Announce important state changes through `announce()`. Keep ARIA labels on icon buttons.
- **Mobile**: Preserve the touch controls and the `touch-action: none` / safe-area CSS. Test at phone width.
- **No secrets, no network calls.** The game is fully client-side.

## Making changes

1. Read the relevant section before editing; the file is large, so search for the function or constant name first.
2. Tune balance through the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`) rather than hard-coding numbers in `update()`.
3. When adding a power-up, update `maybeSpawnPowerUp()`, `applyPowerUp()`, and the HUD markup together.
4. When adding a boss pattern, add it to `BOSS_PATTERNS` and make sure it is reachable from `updateBoss()`.
5. After any change, load `index.html` in a browser and check: start, fire, bomb, take damage, pause/resume, game over, best-score update, and the mobile layout.
6. Do not introduce console errors. A quick check is to open the console while playing.

## Git

- Default development branch for this session: `claude/gifted-tesla-ej1x0t`.
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Do not open pull requests unless explicitly asked.

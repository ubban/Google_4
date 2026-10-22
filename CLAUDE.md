# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. Everything (markup, CSS, JavaScript) lives in `index.html`. `README.md` is a placeholder.

There is no build system, package manager, bundler, test suite, or linter. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the folder with any static server, for example:

```sh
python3 -m http.server 8000
```

Then visit `http://localhost:8000/`.

## File layout (`index.html`)

- Lines ~1–360: `<head>` with meta tags and one inline `<style>` block.
- Lines ~362–420: DOM for the HUD, touch controls, and the start/pause overlay. Element IDs here are referenced by the script.
- Lines ~422–1668: one inline `<script>`, wrapped in an IIFE with `'use strict'`. Nothing is exported to `window`.

Inside the script, the code is grouped under section comments such as `// ---------- Input ----------` and `// ---------- Canvas sizing ----------`. Keep new code in the matching section.

## Architecture

- **Rendering:** a single `<canvas id="gameCanvas">` at a fixed logical size of `BASE_W × BASE_H` (480 × 720). `resize()` scales the CSS size to fit the window, capped at 1.6×. Drawing uses the 2D context directly.
- **Loop:** `requestAnimationFrame` drives `loop(now)`. Each frame it clamps `dt` to 0.05 s, calls `trackPerf`, then runs `update(dt)` and `draw()` only while the game is running and not paused.
- **Adaptive quality:** `trackPerf` watches a rolling window of 45 frame times and steps `quality` (0 full, 1 medium, 2 low) down or up depending on average FPS.
- **State:** top-level `let` variables in the IIFE (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `running`, `paused`, `gameOver`, and so on). `resetGame()` reinitializes them.
- **Tuning data** lives in `const` objects near the top of the script:
  - `DIFFICULTIES`: spawn rates, enemy speed and fire chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE`: kills needed before advancing a stage.
  - `THEMES`: background palettes, cycled per stage.
  - `ENEMY_TYPES`: grunt, interceptor, tank (size, HP, speed, score, minimum stage).
  - `BOSSES`: six bosses with colors and HP multipliers.
  - `BOSS_PATTERNS`: bullet patterns (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Audio:** a Web Audio API wrapper (`ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object). Audio starts lazily on the first user gesture, because browsers block autoplay.
- **Persistence:** `localStorage` only, with keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every access must stay wrapped so the game still works if storage is unavailable or throws (for example in private browsing).
- **Accessibility:** `announce()` writes to the `#srLive` ARIA live region. Reduced motion is respected from `prefers-reduced-motion` and can be toggled in the UI; it disables screen shake, flashes, and pulses.

## Controls

| Input | Action |
|---|---|
| Mouse move over canvas | Steer |
| Mouse click | Fire (`mousedown` on canvas) |
| Arrow keys / WASD | Move |
| Space (held) | Fire |
| X or Shift | Bomb |
| P or Esc | Pause |
| Touch | Drag left zone to fly (virtual joystick), tap FIRE, tap BOMB |

Keyboard state is tracked in the `keys` map by `KeyboardEvent.code`.

## Gameplay rules to preserve

- Power-ups (`life`, `bomb`, `weapon`) drop from enemies. Weapon pickups upgrade the level 1→3, then switch between `vulcan` and `laser` and reset to level 1.
- Lives and bombs come from the difficulty table. Life and bomb pickups are capped at the max.
- Reaching `KILLS_PER_STAGE` kills spawns the stage boss. Defeating it (`killBoss`) advances `stage` and the background theme, which cycles through `THEMES` by `(stage - 1) % THEMES.length`.
- Score is multiplied by a chain-based multiplier (`1 + min(9, floor(chain / 5))`); a missed chain resets it.
- Best score is saved on game over (`endGame`).

When changing numbers, change the tables, not the logic that reads them.

## Conventions

- Match the existing style: 2-space indent, single quotes in JS, `const` by default, arrow functions for callbacks, and short section comments with `// ---------- Name ----------`.
- Keep everything in `index.html` unless the user asks to split files. Adding external scripts or stylesheets changes the no-dependency design and needs a reason.
- Don't add CDN dependencies. The game is meant to run offline from a single file.
- Keep touch and mouse input working when changing input code. `isTouchDevice` toggles the touch controls UI.
- Keep the layout usable on phones (`viewport` meta is set; the canvas scales to fit).

## Verifying changes

There are no automated tests. For a change, open the page in a browser and check:

1. The game starts from the overlay on each difficulty.
2. Keyboard, mouse, and touch controls respond (touch via browser device emulation).
3. Pause, mute, and reduced-motion toggles persist across a reload.
4. Game over shows the score and updates the best score.
5. The browser console has no errors.

If you change the numbers in `DIFFICULTIES`, `ENEMY_TYPES`, or `BOSSES`, play at least one full stage to check that the balance still works.

## Git

- Development branch for this work: `claude/gifted-tesla-y7k0j8`.
- Commit messages are short, imperative, and describe the gameplay or file change (for example, "Add Sky Fighter: Raiden-style vertical shoot-em-up game").

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs entirely in the browser. There is no build step, no package manager, and no dependencies.

- `index.html` — the whole game: markup, CSS (`<style>` block), and game logic (one inline `<script>` IIFE on canvas 2D).
- `README.md` — minimal project readme.

To play, open `index.html` directly in a browser, or serve the directory with any static server.

## Repository layout

```
index.html   # Entire game (≈1,670 lines)
README.md    # Project readme
CLAUDE.md    # This file
```

Inside `index.html`, the structure is:

1. `<head>` — meta tags, inline SVG favicon, `<title>`.
2. `<style>` — all styling, including the `body.reduced-motion` variant.
3. `<body>` — HUD (`#score`, `#best`, `#mult`, `#weaponName`, `#hearts`, `#bombs`), `<canvas id="gameCanvas">`, start/pause overlay (`#overlay`), touch controls (`#touchControls`), and boss bar (`#bossBarWrap`).
4. `<script>` — a single IIFE with `'use strict'`. Sections are marked with `// ---------- Name ----------` comments: Adaptive performance, Input, Audio, Game state, Update, Draw, and the main `loop`.

## Architecture notes

- **Game loop:** `requestAnimationFrame(loop)` calls `update(dt)` then `draw()` while `running && !paused`. `dt` is clamped to 50 ms.
- **Adaptive quality:** `trackPerf` averages the last 45 frame times and steps `quality` (0 full / 1 medium / 2 low) down when the frame rate drops below ~40 fps and back up above ~56 fps.
- **Difficulty:** `DIFFICULTIES` (`easy`, `normal`, `hard`) sets spawn timing, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier. `setDifficulty` applies the choice.
- **Content tables:** `THEMES` (background palettes, one per stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), `BOSSES` (six bosses, cycled by stage), and `BOSS_PATTERNS` (bullet patterns). `KILLS_PER_STAGE` controls stage progression.
- **Weapons and power-ups:** `applyPowerUp` handles `life`, `bomb`, and `weapon`. Weapons are `vulcan` and `laser`, each with levels 1–3. Taking a weapon pickup at max level switches weapon type and resets the level to 1.
- **Audio:** Web Audio API, synthesized in code (`beep`, `noiseBurst`, and the `sfx` object). `ensureAudio()` must run from a user gesture before sound plays.
- **Input:** keyboard, mouse (canvas aim), and touch (`joystick` on `#moveZone`, plus `#fireBtn` and `#bombBtn`). Touch devices are detected with `isTouchDevice`.
- **Rendering:** `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas to fit the viewport. Drawing uses the 2D context directly with no libraries.

### Controls

| Action        | Keyboard                         | Mouse / touch                       |
| ------------- | -------------------------------- | ----------------------------------- |
| Move          | Arrow keys or `WASD`             | Mouse moves over canvas / left-side joystick |
| Fire          | `Space`                          | Click canvas / hold FIRE button     |
| Bomb          | `X` or `Shift`                   | BOMB button                         |
| Pause         | `P` or `Escape`                  | ⏸ button                            |

### Persistence

The game stores these keys in `localStorage`. Changing a key name is a breaking change for existing players, so do not rename them casually.

| Key                         | Meaning                          |
| --------------------------- | -------------------------------- |
| `skyfighter.bestScore`      | Best score                       |
| `skyfighter.muted`          | `'1'` when muted                 |
| `skyfighter.difficulty`     | Selected difficulty name         |
| `skyfighter.reducedMotion`  | `'1'` / `'0'`; defaults to the OS `prefers-reduced-motion` setting |

Every `localStorage` access should stay wrapped so the game still works when storage is blocked (for example, in a private window).

### Accessibility

- A polite live region (`#srLive`) announces important events through `announce(text)`.
- Reduced motion turns off screen shake, flashes, and pulses. It is toggled by `#motionBtn` and the `body.reduced-motion` class.
- Keep new controls keyboard-reachable and labeled (`title` or `aria-label`).

## Conventions

- **Single file:** Keep the game in `index.html` unless the user asks for a split. Do not add a build step, bundler, or framework without asking.
- **No external dependencies:** The page loads nothing from a CDN. Keep it that way unless there is a clear reason.
- **Match local style:** 2-space indentation, single quotes in JS, `const` by default, and section banner comments (`// ---------- Name ----------`). Keep comment density close to the surrounding code.
- **Tables over branches for content:** New enemies, bosses, themes, or difficulty settings should be new entries in the existing tables rather than new code paths.
- **Keep gameplay constants together:** Put tunable numbers in the tables at the top of the script section, not scattered through update logic.

## Testing and verification

There is no automated test suite or linter. To verify a change:

1. Open `index.html` in a browser, or serve the folder (`python3 -m http.server` works).
2. Play through at least one stage on each difficulty you touched, and check the boss fight if you changed boss or stage logic.
3. Check pause, mute, reduced-motion, and difficulty persistence after a page reload.
4. Check touch controls with browser device emulation if you changed input or layout.
5. Confirm there are no console errors in the browser devtools.

Automated checks, if added later, should go in this section.

## Git workflow

- Develop on the branch specified by the task. Do not push to other branches without permission.
- Write clear, present-tense commit messages that describe the gameplay or code change.
- Do not open a pull request unless the user asks for one.

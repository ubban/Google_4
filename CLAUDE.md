# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only app is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up that runs in the browser as a single HTML5
canvas page.

| File        | Purpose                                                        |
|-------------|----------------------------------------------------------------|
| `index.html`| The entire game: markup, CSS, and JavaScript in one file (~1,670 lines) |
| `README.md` | Minimal project title                                          |
| `CLAUDE.md` | This file                                                      |

There is no build system, package manager, test suite, or server-side code.

## Running the game

No build step is needed. Open the file directly, or serve the directory with any
static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Architecture of `index.html`

The file has three parts, in order:

1. **`<head>`**: meta tags (viewport locked for mobile, `theme-color` `#0b1d3a`),
   an inline SVG favicon, and the page `<title>`.
2. **`<style>`**: all CSS. Layout is a fixed-aspect game container. The HUD is
   pill-styled, the overlay is a glassmorphic card, and touch controls are
   positioned over the canvas.
3. **`<body>` markup + one `<script>`**: a single IIFE (`'use strict'`) that holds
   all game state and logic. There are no modules and no globals to export.

### Logical game dimensions

The canvas uses a fixed logical resolution, `BASE_W = 480`, `BASE_H = 720`
(`index.html` near the `resize()` function). Game coordinates are always in these
units. `resize()` scales the canvas to the window. Do not hard-code screen pixels
into gameplay code.

### Main code sections (search for these names)

| Area | Identifiers to search for |
|------|---------------------------|
| Config tables | `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, `KILLS_PER_STAGE` |
| Persistence | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `EXPLODE_SFX`, `panFor` (Web Audio, synthesized, no asset files) |
| Input | `keys`, `keydown`/`keyup` listeners, `joystickStart/Move/End`, `canvasPointFromEvent` |
| Game loop | `resetGame`, `update(dt)`, `draw()`, `loop(now)` (`dt` is clamped to 0.05 s) |
| Combat | `fireWeapon`, `applyPowerUp`, `useBomb`, `spawnEnemy`, `spawnBoss`, `updateBoss`, `killBoss` |
| Feedback | `flashHit`, `flashBomb`, `screenShake`, `showComboPop`, `showStageBanner`, `announce` |
| Lifecycle | `loseLife`, `endGame`, `togglePause` |

## Controls (keep the in-game hint in sync)

- **Desktop:** mouse to steer and click to fire; Arrow keys or WASD to move;
  Space to fire; X or Shift for a bomb; P or Esc to pause.
- **Mobile:** left-side joystick zone to fly; FIRE and BOMB buttons on the right.

The desktop/mobile hint is the `#controlsHint` element. If you change a binding,
update that text and the key handlers together.

## Persistent state

Player settings and the best score live in `localStorage` under the
`skyfighter.*` keys (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every
access must tolerate storage being unavailable, so keep reads and writes defensive
and do not assume they succeed.

Reduced motion starts from the `prefers-reduced-motion` media query and can be
overridden by the user's saved choice.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split.
  Do not introduce a bundler, framework, or dependencies without a clear request.
- **No external assets.** Audio is synthesized with the Web Audio API and graphics
  are drawn on canvas. Do not add image or sound files without asking.
- **Keep the game loop allocation-light.** `update` and `draw` run every frame.
  Avoid creating objects or closures inside hot loops where a reused structure works.
- **Respect the `dt` model.** Movement and timers are scaled by the clamped `dt`,
  so changes to speeds or timers should be expressed per second.
- **Audio starts on user gesture.** Call `ensureAudio()` from input handlers rather
  than at load time, since browsers block autoplay.
- **Difficulty is data-driven.** Tune balance through the `DIFFICULTIES`, `ENEMY_TYPES`,
  and `BOSSES` tables instead of scattering magic numbers.

## Verifying changes

There are no automated tests or linters configured. To verify a change:

1. Open `index.html` in a browser (or serve it statically) and confirm there are
   no errors in the developer console.
2. Play through the start screen, a stage, a boss fight, a bomb, a pause/resume,
   and a game over. Check the score, best score, and hearts/bombs HUD.
3. For input changes, test both keyboard and touch (browser device emulation works).
4. For layout changes, check a narrow phone-sized viewport and a desktop viewport.

If a headless browser is available, a short Playwright script that loads the page
and checks for console errors is a reasonable smoke test, but it is not required.

## Git workflow

- Develop on the branch the task specifies (currently `claude/gifted-tesla-a0p9l0`).
- Write clear commit messages that describe the gameplay or code change.
- Do not open pull requests unless explicitly asked.

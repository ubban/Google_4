# CLAUDE.md

Guidance for working in this repository.

## What this is

`Google_4` is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. Everything lives in one file, `index.html`, which contains the HTML, CSS (`<style>`), and JavaScript (`<script>`) inline. There is no build step, no package manager, and no test suite.

Files:

- `index.html` — the whole game (~1,670 lines).
- `README.md` — a one-line placeholder.

## Running it

Open `index.html` in a browser. No server or install is needed.

There are no lint, build, or test commands. When you change game logic, verify it by loading the page and playing; say so explicitly if you could not do that.

## Code layout inside `index.html`

- **Markup and styles** (approx. lines 1–421): the `<canvas id="gameCanvas">`, the HUD (score, best, multiplier, weapon, hearts, bombs, boss bar), the overlay/menu, touch controls, and the CSS. Styles use a `reduced-motion` body class.
- **Script** (approx. lines 422–1667): one IIFE with `'use strict'`. Sections are marked with `// ---------- Name ----------` comments:
  - *Adaptive performance* — `trackPerf` drops visual `quality` (0 full, 1 medium, 2 low) when frame time is poor.
  - *Config tables* — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
  - *Audio* — a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, the `sfx` object). Audio starts only after a user gesture.
  - *Canvas sizing* — `BASE_W = 480`, `BASE_H = 720`, scaled by `resize()`.
  - *Input* — keyboard (`keys`), touch joystick, fire and bomb buttons, and mouse.
  - *Game state* — `resetGame()`, `update(dt)`, `draw()`, `loop(now)`. `loop` clamps `dt` to 0.05 s and only updates and draws while `running && !paused`.
  - *Weapons and bosses* — `fireWeapon`, `BOSS_PATTERNS` (bullet-pattern functions such as `fireRadial`, `fireCross`, `fireHomingSeeded`), `updateBoss`, `killBoss`.
  - *Persistence* — `localStorage` (see below).

Mechanics are plain objects in arrays (`bullets`, `enemies`, `powerUps`, and so on) updated each frame. Rectangle collision uses `rectsOverlap`.

## Game rules worth knowing

- **Difficulty** (`easy` / `normal` / `hard`) sets spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier. It can be changed only between games.
- **Stages**: after `KILLS_PER_STAGE` (18) kills, a boss spawns about 1.2 s later. Defeating the boss advances the stage, which changes the theme and resets the kill count. Enemy types unlock by `minStage` (`tank` from stage 2).
- **Power-ups** dropped by enemies: `life` (6%), `bomb` (8%), `weapon` (20%). A weapon pickup first raises the weapon level to 3, then toggles between `vulcan` and `laser` and resets the level to 1.
- **Score** uses a combo multiplier (`addScore`, `breakChain`).

## Controls

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrow keys / WASD (via `keys[e.code]`) | Left joystick |
| Fire | Hold Space | Fire button |
| Bomb | X or Shift | Bomb button |
| Pause | P or Escape | Pause button |
| Mouse | Move pointer to steer; click (hold) to fire | — |

## Persistence

`localStorage` keys, all prefixed `skyfighter.`:

- `bestScore` — high score, written in `endGame`.
- `muted` — `'1'` or `'0'`.
- `difficulty` — last chosen difficulty.
- `reducedMotion` — `'1'` or `'0'`. Defaults to the `prefers-reduced-motion` media query when unset.

Any change to these keys must keep existing saved values readable.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not add a build tool or framework without asking.
- Match the surrounding style: 2-space indent, single quotes in JS, short section banner comments, `const` by default, arrow callbacks for listeners.
- Game-balance numbers are tuned values in the config tables. Change them there, not scattered through the logic.
- Keep the UI usable on phones: the layout is viewport-based with `viewport-fit=cover`, and touch controls appear only on touch devices (`isTouchDevice`).
- Preserve accessibility hooks: the `srLive` live region is used by `announce()` for screen-reader status, and the reduced-motion toggle should keep scaling down screen shake (`screenShake`) and skipping the flash effects.

## Git

Develop on the branch the session specifies. Commit with clear messages and push with `git push -u origin <branch>`.

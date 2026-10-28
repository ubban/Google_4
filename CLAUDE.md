# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a single-page, Raiden-style vertical shoot-em-up built on an HTML5 canvas. The whole game lives in one file, `index.html`. There is no build step, no package manager, no bundler, and no external assets. Audio is synthesized with the WebAudio API.

`README.md` contains only a title. Treat `index.html` as the source of truth.

## Repository layout

```
index.html   The entire game: CSS in <style>, markup in <body>, JavaScript in one <script>
README.md    Project title only
```

Inside `index.html`:

- **`<style>`** (around lines 10–360): all styling, including the HUD pills, the glassmorphic overlay card, the boss health bar, the touch controls, and the reduced-motion rules.
- **`<body>`** (from around line 361): the `#container` with the canvas (`#gameCanvas`), HUD, boss bar, combo and stage banners, touch controls, and the start/pause overlay (`#overlay`).
- **`<script>`** (from around line 422): one `(function () { 'use strict'; ... })();` IIFE that holds all game logic.

## Running and checking changes

There is no test suite, linter, formatter, or build. To run the game, open `index.html` in a browser. To check the JavaScript for syntax errors without a browser, extract the script and run it through Node:

```sh
awk '/<script>/{f=1;next}/<\/script>/{f=0}f' index.html > /tmp/game.js && node --check /tmp/game.js
```

Syntax checks do not cover gameplay. For gameplay changes, play the game in a browser and check the affected path: start, fire, bomb, a boss fight, pause, game over, and the touch controls if the change touches input.

## Architecture of the game script

The script is organized into commented sections that start with `// ---------- Name ----------`. Keep new code inside the matching section, and add a new section header when a new area appears.

Main sections, in file order:

- **Setup**: DOM lookups at the top, then the `STORAGE_*` keys.
- **Adaptive performance** (`trackPerf`): steps a `quality` level (0 full, 1 medium, 2 low) from average frame time.
- **Tuning tables**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
- **Reduced motion** (`updateMotionBtn`).
- **Audio** (`ensureAudio`, `beep`, `noiseBurst`): synthesized sound effects.
- **HUD and effects** (`renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`).
- **Canvas sizing** (`resize`): the logical play field is `BASE_W` × `BASE_H` (480 × 720).
- **Input** (keyboard, joystick, touch buttons, mouse steering).
- **Game state** (`resetGame`, `update`, `updateBoss`, `loseLife`, `endGame`, `togglePause`).
- **Drawing** (`drawPlane`, `drawBossShip`, `drawClouds`, `draw`).
- **Weapons and boss patterns**: `fireWeapon`, `useBomb`, and the `BOSS_PATTERNS` array of six boss attack patterns (`fire*` helpers).
- **Main loop** (`loop(now)`), started with `requestAnimationFrame` at the end of the IIFE.

## Where to tune gameplay

Most balance numbers are in the constant tables near the top of the script, not inside the logic:

- `DIFFICULTIES`: spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier for `easy`, `normal`, and `hard`.
- `KILLS_PER_STAGE`: kills needed to advance a stage.
- `ENEMY_TYPES`: size, HP, speed, score value, and first stage each enemy appears in.
- `BOSSES`: name, colors, and HP multiplier. The attack logic for each boss is in `BOSS_PATTERNS`.
- `THEMES`: six sky palettes used for stage backgrounds.

## Persistence

The game stores four values in `localStorage`, all prefixed with `skyfighter.`:

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | Best score |
| `skyfighter.muted` | `'1'` when sound is muted |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | `'1'` or `'0'`. When unset, the `prefers-reduced-motion` media query decides the default |

The `localStorage` calls are not wrapped in `try`/`catch`. If you change storage code, consider that some browsers throw when storage is blocked, such as in private windows or with site data disabled.

## Controls

- **Move**: arrow keys or WASD. On a touch device, the joystick in the lower-left (`#moveZone`). With the mouse, the plane glides toward the pointer while it is over the canvas. Keyboard and joystick input take priority over mouse steering.
- **Fire**: Space (hold to fire continuously), or the fire button on touch devices.
- **Bomb**: `X`, or either Shift key, or the bomb button on touch devices.
- **Pause**: `P` or Escape.

## Conventions

- **Single file, no dependencies.** Keep all CSS and JavaScript in `index.html`. Do not add a framework, module system, or external asset without asking first. Loading resources from the network would also break the game when it is opened offline.
- **Match the existing style.** Use `const` and `let`, arrow functions for callbacks, and `function` declarations for named helpers. Constants are `UPPER_SNAKE_CASE`. DOM references use `camelCase` names that match their element IDs.
- **Accessibility.** Announce important state changes through `announce()`, which writes to the `#srLive` live region. Respect reduced motion: `flashHit` and `flashBomb` return early when it is on, and `screenShake` shrinks its magnitude and caps its duration.
- **Input and audio start on user gesture.** Call `ensureAudio()` from a click or touch handler, not on page load, because browsers block audio before a gesture.
- **Keep the play field in logical coordinates.** Game objects use the `BASE_W` × `BASE_H` (480 × 720) coordinate space. `resize()` keeps the canvas bitmap at that size and scales its CSS size to fit the window, capped at 1.6×.

## Git

- Develop on the branch named in the task. Do not push to other branches.
- Keep commits focused, and describe the gameplay or UI change in the message.

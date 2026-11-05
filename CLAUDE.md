# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-page HTML5 canvas game called **Sky Fighter**, a Raiden-style vertical shoot-'em-up.

- The entire app lives in one file, `index.html` (~58 KB): inline `<style>`, markup, and one `<script>` IIFE.
- No build step, no package manager, no dependencies, no tests. Open `index.html` in a browser to run it.
- `README.md` is a placeholder and contains only the title.
- Git history: the initial commit, then the commit that added Sky Fighter.

## Working on the code

- Keep the game in the single `index.html` unless the user asks for a split. Don't add a bundler, framework, or npm dependencies without being asked.
- Edit in place and keep the existing style: 2-space indentation, `const`/`let`, plain ES2015+ features, no transpilation.
- Verify changes by loading `index.html` in a browser (a headless Chromium or Playwright run is fine). There is no automated test suite, so say so explicitly if you could only verify by reading.
- Preserve the mobile and accessibility behavior described below when changing input, HUD, or overlay code.

## File layout (`index.html`)

| Section | Purpose |
| --- | --- |
| `<head>` | Meta tags, inline SVG favicon, and the full stylesheet (HUD pills, overlay card, touch controls, vignette, flash, boss bar, animations). |
| `<body>` markup | `#container` holds `#gameCanvas` and overlays: `#hud` (score, best, multiplier, weapon, hearts, bombs), `#overlay` (title, difficulty buttons, Start button, controls hint), `#touchControls` (joystick, FIRE, BOMB), icon buttons `#muteBtn` / `#motionBtn` / `#pauseBtn`, `#bossBarWrap`, `#comboPop`, `#stageBanner`, `#vignette`, `#flash`, and a screen-reader live region `#srLive`. |
| `<script>` | One IIFE with all game logic. Sections, in order: DOM refs, storage keys, `announce`, perf tracking, config tables, state and persistence, reduced motion, audio, SFX, HUD rendering, effects, canvas sizing, keyboard and touch input, mouse input, spawning, drawing, `update(dt)`, `updateBoss(dt)`, `draw()`, and `loop(now)`. |

## Game design constants

These live in the script and are the first place to look when tuning balance.

- **Difficulties** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn timing, enemy speed, enemy shoot chance, starting `lives`, starting `bombs`, and `bossHpMul`.
- **Stages**: `KILLS_PER_STAGE = 18`. Each stage has a theme from `THEMES`, which cycles through six sky palettes.
- **Enemies** (`ENEMY_TYPES`): `grunt` (1 HP), `interceptor` (1 HP, weaves, fast), `tank` (4 HP, from stage 2).
- **Bosses** (`BOSSES`): six named bosses (VANGUARD through TITAN), each with its own colors and `hpMul`. Bosses use `updateBoss` and `drawBossShip`, and show a health bar.
- **Weapons**: Vulcan and Laser, each with power levels 1-3, changed by power-ups.
- **Bombs**: limited smart bombs that clear the screen and damage bosses (`useBomb`).
- **Score**: a kill-chain multiplier shown in `#mult`.

## Persistence

`localStorage` keys (all prefixed `skyfighter.`):

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Every read and write should tolerate storage being unavailable (private mode, blocked site data). Keep the game playable when it is.

## Input

- **Keyboard**: arrows or WASD to move, Space to fire, X or Shift for bomb, P or Esc to pause. Input is tracked in the `keys` map by `e.code`.
- **Mouse**: move to steer, click the canvas to fire. Mousedown on `#fireBtn` also fires.
- **Touch**: drag in the left `#moveZone` for a virtual joystick (`joystickStart` / `joystickMove` / `joystickEnd`). Tap FIRE and BOMB. `touch-action: none` is set on the page, so don't add scrolling behavior.

## Accessibility and UX conventions

- Respect `prefers-reduced-motion`. The reduced-motion toggle (`#motionBtn`) turns off screen shake, flashes, and the low-health pulse.
- Announce important state changes through `announce()`, which writes to `#srLive`.
- Keep the HUD readable on phones. The layout uses `clamp()` font sizes and `max-width: 96vw`.
- Use `env(safe-area-inset-*)` for notched devices, as the existing HUD does.
- Canvas is a fixed 480x720 logical size (`BASE_W`, `BASE_H`), scaled by `resize()` up to 1.6x to fit the window. Draw in logical coordinates.

## Performance

- `trackPerf(dt)` samples frame times and sets `quality` (0 full, 1 medium, 2 low). Effects and particle counts should respect `quality` so low-end devices stay smooth.
- The loop is a single `requestAnimationFrame` loop (`loop(now)`). Don't add a second one.

## Git workflow

- Develop on the branch the session specifies. Don't push to other branches without permission.
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Don't open a pull request unless the user asks for one.

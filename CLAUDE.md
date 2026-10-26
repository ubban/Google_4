# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static project containing **Sky Fighter**, a Raiden-style
vertical shoot-'em-up that runs entirely in the browser.

- `index.html` — the whole game: markup, CSS, and JavaScript in one file (~1,670 lines).
- `README.md` — minimal project readme.

There is no build step, no package manager, no bundler, no external assets,
and no test suite. Audio is synthesized with the WebAudio API.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static
server (for example `python3 -m http.server` from the repo root) and visit it.

## Code layout inside `index.html`

- **`<style>`** (around lines 10–361): all styling, including the pill-style HUD,
  glassmorphic overlay card, and mobile touch controls.
- **`<body>` markup** (around lines 362–421): `#container` holds the canvas,
  HUD (`#hud`, `#topBar`, `#bossBarWrap`), popups (`#comboPop`, `#stageBanner`),
  touch controls (`#touchControls`), and the start/pause overlay (`#overlay`).
- **`<script>`** (around lines 422–end): a single IIFE with `'use strict'`.
  Sections are marked with `// ---------- Name ----------` comments:
  - Adaptive performance (`trackPerf`, quality levels 0–2)
  - Audio (`beep`, `noiseBurst`, `ensureAudio`)
  - Canvas sizing (`resize`, base resolution 480×720)
  - Input (keyboard, joystick, mouse steering, touch buttons)
  - Game state (`resetGame`, `update`, `draw`, `loop`)
  - Enemies, bosses (`BOSS_PATTERNS`, `updateBoss`, `killBoss`), power-ups, bombs

Keep new code in the same single-file structure unless the user asks for a split.

## Game configuration

Tunable values live in constants near the top of the script:

- `DIFFICULTIES` — `easy` / `normal` / `hard`, controlling spawn rate, enemy speed,
  enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- `KILLS_PER_STAGE` — kills needed to advance a stage (boss fight follows).
- `THEMES` — six sky/cloud/sun palettes cycled per stage.
- `ENEMY_TYPES` — `grunt`, `interceptor` (weaves), `tank` (4 HP, appears from stage 2).
- `BOSSES` — six bosses (VANGUARD → TITAN), each with a distinct colour scheme,
  HP multiplier, and attack pattern set in `BOSS_PATTERNS`.

When changing balance, prefer editing these constants over hard-coding numbers in
the update loop.

## Persistence

Settings and scores use `localStorage` with the `skyfighter.*` prefix:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Any new persisted setting should use the same prefix and be read/written
through the existing pattern so a missing or blocked storage does not break the game.

## Controls (for reference when testing)

- Keyboard: Arrows / WASD to move, Space to fire, X or Shift for bomb, P or Esc to pause.
- Mouse: move to steer, click to fire.
- Touch: drag left zone to fly, FIRE and BOMB buttons on the right.

## Conventions

- Plain HTML, CSS, and vanilla JavaScript. Do not add frameworks or dependencies
  without an explicit request.
- Match the existing style: 2-space indentation, single quotes in JS, short
  section-divider comments, and `const` by default.
- Keep the game playable on phone-sized viewports and with reduced-motion
  preferences respected (`reducedMotion` flag).
- Performance matters: the loop runs every frame on the canvas. Avoid per-frame
  allocations in hot paths where practical, and respect the adaptive `quality` level.

## Verifying changes

There are no automated tests or linters. To verify a change:

1. Open `index.html` in a browser (desktop and, if touch code changed, a
   mobile-sized viewport).
2. Start a game, check firing, bombs, power-ups, boss fights, and the game-over flow.
3. Confirm the pause overlay, mute, difficulty buttons, and reduced-motion toggle still work.
4. Check the browser console for errors.

## Git

- Development branch for the current work: `claude/gifted-tesla-az1p9i`.
- Write clear, descriptive commit messages that describe the change, not the file list.
- Do not open pull requests unless explicitly asked.

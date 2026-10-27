# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project containing **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a `<canvas>`.

- `index.html` — the entire game: markup, CSS, and JavaScript in one file (~1,670 lines).
- `README.md` — a one-line title. Nothing else.

There is no build step, package manager, bundler, test suite, linter, or CI configuration. Do not add tooling unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000
```

Then visit `http://localhost:8000/`.

## Code layout inside `index.html`

- **`<head>`** — viewport meta (locks zoom), theme color, and an inline SVG favicon.
- **`<style>`** (around line 10) — all CSS. Touch controls are hidden unless `#touchControls.enabled` is set. Mobile layout relies on `touch-action: none`.
- **`<body>`** (around line 362) — DOM for the HUD (`#hud`, `#topBar`, `#bossBarWrap`), transient popups (`#comboPop`, `#stageBanner`, `#flash`, `#vignette`), touch controls (`#touchControls`), and the start/pause overlay (`#overlay`).
- **`<script>`** (around line 422) — one IIFE wrapping all game logic, with `'use strict'`. Nothing is exposed on `window`. Keep new code inside the IIFE.

Within the script, the rough order is:

1. DOM lookups and `STORAGE_*` keys.
2. Adaptive quality (`trackPerf`, `quality` 0–2) that reduces effects when frame times are slow.
3. Tuning tables: `DIFFICULTIES`, `THEMES` (stage backgrounds), `ENEMY_TYPES`, `BOSSES`, and `KILLS_PER_STAGE`.
4. Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `EXPLODE_SFX` map. Audio is synthesized with WebAudio, with no audio files.
5. HUD helpers (hearts, bombs, combo pop, stage banner, screen shake).
6. Input: keyboard handlers, a virtual joystick and buttons for touch, and mouse steering.
7. Game state: `resetGame`, `update(dt)`, spawning, collisions, scoring, weapons, power-ups, and bombs.
8. Bosses: `BOSS_PATTERNS` is an array of bullet-pattern functions, and `updateBoss` / `killBoss` drive them.
9. Rendering: `draw()` and the `drawPlane` / `drawBossShip` / `drawClouds` helpers.
10. `loop(now)` — the `requestAnimationFrame` loop. `dt` is clamped to 50 ms.

## Controls

| Input | Action |
| --- | --- |
| Arrow keys or WASD | Move |
| Mouse move | Steer toward the pointer |
| Space, or mouse click | Fire |
| X or Shift | Smart bomb |
| P or Esc | Pause / resume |
| Drag left zone, tap FIRE, tap BOMB | Touch controls |

Pressing Space calls `preventDefault` so the page does not scroll.

## Gameplay constants

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn rate, enemy speed, shoot chance, starting lives, starting bombs, and boss HP multiplier.
- **Stages**: a stage ends after `KILLS_PER_STAGE` (18) kills and then spawns a boss. `THEMES` cycles the background palette.
- **Enemies**: `grunt` (1 HP, from stage 1), `interceptor` (weaves, from stage 1), and `tank` (4 HP, from stage 2).
- **Weapons**: `vulcan` and `laser`. A weapon power-up raises the current weapon's level from 1 to 3, then switches to the other weapon at level 1.
- **Score**: `addScore` multiplies base points by `1 + min(9, floor(chain / 5))`, so up to ×10. The chain resets after 2.2 s without a kill (`breakChain`).

When you change any of these, keep the tables as the single source of truth. Do not hard-code the same numbers elsewhere.

## Persistence

`localStorage` keys, all prefixed `skyfighter.`:

- `bestScore` — high score
- `muted` — `'1'` or `'0'`
- `difficulty` — `easy` / `normal` / `hard`
- `reducedMotion` — `'1'` or `'0'`. If unset, it follows `prefers-reduced-motion`.

Reads and writes are not wrapped in try/catch. If you add more storage, wrap it so the game still works when storage is blocked, for example in private browsing.

## Accessibility

- `#srLive` is an `aria-live` region. Use the `announce(text)` helper for important state changes.
- Reduced motion scales screen shake down to a short, weak shake and skips hit and bomb flashes. Keep new effects behind the `reducedMotion` flag.
- The viewport meta disables user zoom, so layouts must work at phone width without relying on zoom.

## Conventions

- Match the existing style: 2-space indentation, single quotes in JS, `const` / `let`, arrow functions for callbacks, and short section banners written as `// ---------- Name ----------`.
- Keep everything in `index.html` unless the user asks for a split. Do not introduce a framework or build step.
- Frame-based logic should use `dt` (seconds), not frame counts, so behavior stays consistent across refresh rates.
- Test changes by loading the page in a browser and playing through at least one stage and one boss. There are no automated tests.

## Git

- Development branch for this session: `claude/gifted-tesla-co7tcz`. Push there with `git push -u origin <branch>`.
- Do not open pull requests unless explicitly asked.

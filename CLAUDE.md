# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small static web game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game lives in a single self-contained file, `index.html`, which contains the HTML markup, inline CSS (`<style>`), and inline JavaScript (`<script>`). There is no build step, no bundler, no package manager, and no external dependencies.

Files:

- `index.html` (~1,670 lines): the whole game.
- `README.md`: currently a stub (title only). Treat it as the place for human-facing docs if they are added later.

## Running the game

No build is needed. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

There is no test suite, linter, or CI configuration in the repo. Verify changes by playing the game in a browser (desktop and, where relevant, a mobile-sized viewport).

## Code layout (inside `index.html`)

- **Markup**: `#container` wraps `<canvas id="gameCanvas">`, a HUD (`#hud`: score, best, multiplier, weapon, hearts, bombs), an overlay for the title/game-over screens (`#overlay`), touch controls (`#touchControls`: joystick, fire, bomb), and accessibility elements (`#srLive` live region).
- **Styles**: a single `<style>` block. Colors are hard-coded hex/rgba values; the page uses `color-scheme: dark`. A `.reduced-motion` body class disables motion effects.
- **Script**: one IIFE (`(function () { 'use strict'; ... })();`). Everything is private to the closure; nothing is exposed on `window`. Keep new code inside this IIFE unless there is a reason not to.

Key sections of the script, in rough order:

| Section | What it holds |
| --- | --- |
| DOM refs | `const xEl = document.getElementById(...)` at the top |
| Storage keys | `skyfighter.*` keys in `localStorage` (best score, mute, difficulty, reduced motion) |
| Adaptive performance | `trackPerf` lowers/raises a `quality` level (0–2) based on average frame time |
| Tuning data | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES` |
| Audio | Web Audio helpers: `ensureAudio`, `beep`, `noiseBurst`, `sfx` |
| Rendering helpers | `renderHearts`, `renderBombs`, `flashHit`, `screenShake`, `drawPlane`, `drawBossShip`, `drawClouds` |
| Input | keyboard (`keys`, Space fire, X/Shift bomb, P/Esc pause), mouse on canvas, touch joystick and buttons |
| Game state | `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb` |
| Bosses | `BOSS_PATTERNS` (bullet-pattern functions such as radial, spiral, cross, homing), `updateBoss`, `killBoss` |
| Update/draw | `update(dt)` (game logic), `draw()` (rendering), `loop(now)` (requestAnimationFrame driver, `dt` clamped to 50 ms) |

## Conventions

- **Vanilla JS only.** Do not add frameworks, bundlers, or npm dependencies unless the user asks for them.
- **Keep it a single file** unless the user explicitly asks to split it. If you split it, update the layout section of this file.
- **Tuning values are data.** Enemy stats, difficulty tables, boss HP multipliers, and palettes live in the constant tables near the top of the script. Prefer changing those tables over hard-coding numbers in logic.
- **Time is in seconds.** `update(dt)` uses a delta in seconds; keep new timers and speeds frame-rate independent by multiplying by `dt`.
- **Respect the adaptive quality level and reduced motion.** New particle or screen-effect code should degrade with `quality` and the `reducedMotion` flag, as existing effects do.
- **Accessibility.** Announce important state changes through `announce()` (which feeds `#srLive`), and keep touch targets and HUD text readable at phone widths. The viewport is locked (`user-scalable=no`) and the page sets `touch-action: none`; do not undo this without reason.
- **Persistence** goes through the `skyfighter.*` `localStorage` keys. Wrap new storage access in a way that tolerates storage being unavailable (private mode, blocked storage) if you add it, and add a new key with the same prefix.
- **Keep the existing style**: 2-space indentation, single quotes in JS, short comment headers (`// ---------- Section ----------`) to delimit sections.

## Making changes

- Read the relevant section before editing; the file is large, so prefer targeted edits over rewriting it.
- After changing gameplay, check the affected paths by actually playing: start, fire, bomb, pause/resume, lose a life, boss fight, game over, and best-score persistence across a reload.
- Check layout at phone width (the HUD wraps and the touch controls appear on touch devices).
- Do not commit `localStorage`-based test state or generated files; the repo contains only source.

## Git

- Default development branch for this workspace: `claude/gifted-tesla-2f7z58`. Push there unless told otherwise.
- Commit messages: short imperative summary line, then an optional body explaining why.
- Do not create pull requests unless explicitly asked.

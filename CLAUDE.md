# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser as a single-page HTML5 canvas app. There is no build step, no package manager, and no test suite.

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` (an IIFE, `'use strict'`).
- `README.md` — minimal placeholder.

Open `index.html` directly in a browser to play.

## Repository layout

| Path | Purpose |
| --- | --- |
| `index.html` | Full game (CSS ~lines 10–359, DOM ~361–421, script ~422–1667) |
| `README.md` | Placeholder |

Keep the game in one file unless the user asks otherwise. Do not introduce a bundler, framework, or npm dependencies without a request.

## Game architecture (inside `index.html`)

The script is organized by concern. Find these by name rather than line number, since lines drift.

- **Setup & DOM refs** — `document.getElementById(...)` lookups at the top of the IIFE.
- **Persistence** — `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every read and write should tolerate missing values; defaults apply when a key is absent.
- **Tuning tables** — `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, `BOSS_PATTERNS`, and `KILLS_PER_STAGE`. Balance changes usually belong here, not in the update logic.
- **Audio** — `ensureAudio()`, `beep()`, `noiseBurst()`. Web Audio is created lazily on first user gesture (browsers require this); `ensureAudio()` is called from input handlers.
- **Rendering** — `drawPlane`, `drawBossShip`, `draw()`, `drawClouds`. Drawing is canvas 2D only.
- **Simulation** — `update(dt)` drives spawning, movement, collisions (`rectsOverlap`), scoring (`addScore`, combo chain via `breakChain`), weapons (`fireWeapon`), bombs (`useBomb`), and power-ups (`applyPowerUp`).
- **Bosses** — `spawnBoss`, `updateBoss`, `killBoss`, with attack patterns in `BOSS_PATTERNS`.
- **Lifecycle** — `resetGame`, `loseLife`, `endGame`, `togglePause`. `loop(now)` is the `requestAnimationFrame` driver; `dt` is clamped to 50 ms.
- **Adaptive quality** — `trackPerf` lowers quality (0 full → 2 low) when average frame time is slow. Respect `quality` when adding expensive effects.
- **Input** — keyboard (`keys[e.code]`), pointer (`canvasPointFromEvent`), and touch joystick plus fire/bomb buttons. Keyboard: arrows/WASD move, Space fires, `X` / Shift bombs, `P` / Escape pause.
- **Accessibility** — `announce()` writes to the `srLive` aria-live region. `reducedMotion` gates screen shake, flashes, and pulses (also via `body.reduced-motion` CSS).

## Conventions

- Match the existing style: 2-space indent, single quotes in JS, `const` by default, arrow functions for callbacks, and short section comments using `// ---------- Name ----------`.
- Keep everything inside the existing IIFE. Do not add globals.
- Use `dt`-scaled movement (seconds), not per-frame constants, so the game stays frame-rate independent.
- New HUD elements need matching CSS in the `<style>` block and, if they change, an `announce()` call or ARIA update.
- Prefer editing existing tuning tables over hard-coding numbers in logic.

## Verifying changes

There is no test runner. Before committing a change to the game script:

1. Syntax-check the inline script. Extract the lines between `<script>` and `</script>` and run `node --check` on the result. This is the fastest correctness gate.
2. Open `index.html` in a browser (or via a local static server) and play through: start, move, fire, bomb, pause/resume, lose a life, game over, and reload to confirm the best score persists.
3. For visual or layout changes, check both desktop and a narrow touch viewport. Touch controls only appear on touch devices.

## Git workflow

- Develop on the branch the session specifies (currently `claude/gifted-tesla-c1hc4u`). Do not push to `main` unless explicitly asked.
- Write descriptive commit messages in the imperative mood, summarizing the gameplay or code change in the subject line.
- Do not open pull requests unless the user asks for one.

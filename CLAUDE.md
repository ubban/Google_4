# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-page HTML5 canvas game, **Sky Fighter**, a Raiden-style vertical shoot-'em-up.

- `index.html` — the entire game: markup, CSS (inline `<style>`), and JavaScript (inline `<script>` starting around line 422). No build step, no dependencies, no package manager.
- `README.md` — minimal placeholder.

Open `index.html` directly in a browser to play. There is no server, test suite, or linter configured.

## Structure of `index.html`

- **Markup (~lines 1–420):** `#container` wraps a `<canvas id="gameCanvas">`, a `#hud` overlay (lives, score, bombs, stage), touch buttons (`#bombBtn`, fire), and the title/pause/game-over overlay card. `#controlsHint` lists the controls.
- **Styles (inline `<style>`):** Dark color scheme, glassmorphic overlay, pill-style HUD. Keep UI styling here.
- **Script (inline, from ~line 422):**
  - Persistence keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` (all via `localStorage`, read with a `Number(...) || 0`-style fallback).
  - `DIFFICULTIES` table (~line 485) and `KILLS_PER_STAGE = 18` (~line 491) control pacing.
  - `resetGame()` (~line 875) initializes a run; `update(dt)` (~line 1148) advances simulation; `updateBoss(dt)` (~line 1411) drives boss patterns; `draw()` (~line 1520) renders; `loop(now)` (~line 1640) is the `requestAnimationFrame` game loop.
  - Keyboard input is tracked in a `keys` map via `keydown`/`keyup` on `window` (~line 769).

## Gameplay summary (for orientation when editing)

- Two weapon systems (Vulcan / Laser), each with 3 power levels.
- Limited smart bombs that clear the screen and damage bosses.
- Stages advance after `KILLS_PER_STAGE` kills, ending in a multi-pattern boss with a health bar (`STAGE N — <name>` banner).
- Kill-chain score multiplier and combo popups.
- Power-ups: weapon level, bomb, extra life.
- Pause (P / Esc), mute toggle, difficulty selector, reduced-motion toggle.

Controls: mouse steers and click fires; arrows/WASD move; Space fires; X or Shift bombs; P/Esc pauses. Touch: drag left to fly, tap FIRE, tap BOMB.

## Conventions and guidelines

- Keep the game **self-contained in `index.html`** unless the user asks to split it. Do not add a bundler, framework, or external runtime dependencies without discussion.
- Do not introduce external network calls or CDN scripts from the game.
- Preserve the `localStorage` key names above; changing them would orphan players' saved best score and settings.
- Wrap `localStorage` access in try/catch if you add new reads/writes, since storage can be unavailable in private windows or with blocked site data.
- Preserve touch support and the viewport meta / `touch-action: none` setup; the game is designed to work on phones.
- Frame-rate-dependent logic should use the `dt` passed into `update`, and respect the reduced-motion setting for screen shake and flashes.
- Verify gameplay changes by running the page in a browser (the Playwright/Chromium setup in the environment works) rather than relying on reading code alone.

## Git

- Development branch for this work: `claude/gifted-tesla-on9t0h`. Commit and push there; do not push to other branches without permission.
- Write descriptive commit messages that explain *why* a change was made.

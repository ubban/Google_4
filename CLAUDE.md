# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single-page HTML5 canvas game.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS (`<style>`), and JavaScript (one IIFE in a `<script>` block). |
| `README.md` | Placeholder title only. |

There is no build step, package manager, test suite, or linter configured.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Code layout (`index.html`)

- **Markup/CSS** (roughly lines 1–420): HUD (score, best, multiplier, weapon, hearts, bombs), overlay card, boss bar, touch controls, and a `#srLive` region for screen-reader announcements.
- **Script** (starts ~line 422), wrapped in `(function () { 'use strict'; ... })()`. Sections, in order:
  - DOM lookups and `STORAGE_*` keys (`skyfighter.*` in `localStorage`).
  - Adaptive quality (`trackPerf`, `quality` 0–2) that adjusts effects based on average frame time.
  - Tuning constants: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
  - Audio: WebAudio helpers `ensureAudio`, `beep`, `noiseBurst`. Sound is muted via `muted`.
  - Spawning and entities: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`.
  - Combat: `fireWeapon`, `useBomb`, `applyPowerUp`, boss attack patterns in `BOSS_PATTERNS` (`fireRadial`, `fireAimedSpread`, `fireCross`, etc.).
  - Game flow: `resetGame`, `update(dt)`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
  - Rendering: `draw`, `drawPlane`, `drawBossShip`, `drawClouds`.
  - Main loop: `loop(now)` drives `update` and `draw` via `requestAnimationFrame`. `dt` is clamped to 50 ms.

## Controls

- **Keyboard:** arrow keys / WASD-style movement via `keys[e.code]`; `Space` fires; `X` or `Shift` drops a bomb; `P` or `Escape` pauses.
- **Touch:** virtual joystick in the left move zone, fire and bomb buttons on screen.
- **Mouse:** pointer movement and mousedown on the canvas, plus a fire button.

## Conventions

- Keep the game as a single self-contained `index.html` unless the user asks for a restructure. Do not add a bundler or dependencies without asking.
- Tunable gameplay values belong in the constants at the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, `KILLS_PER_STAGE`), not hard-coded in update logic.
- Preserve the `localStorage` key names (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`). Renaming them resets players' saved data.
- Respect `prefers-reduced-motion` (the `reduced-motion` body class and `reducedMotion` flag) for any new screen-shake or flashing effects.
- Wrap any new `localStorage` access so the game still works when storage is unavailable.
- Accessibility: announce important state changes through `announce()` so they reach `#srLive`.

## Verification

No automated tests exist. After changing gameplay or rendering code:

1. Open `index.html` in a browser (or a headless Chromium via Playwright) and check the console for errors.
2. Start a game, play through pause/resume, a bomb, a power-up, and a life loss.
3. For mobile-facing changes, check the touch controls at a phone-sized viewport.

## Git

- Development branch for this session: `claude/gifted-tesla-s91oh7`. Push with `git push -u origin <branch>`.
- Write descriptive commit messages that explain the gameplay or code change.

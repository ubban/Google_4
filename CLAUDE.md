# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, single-file web project. Its only content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser as one HTML5 canvas page.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: inline CSS, markup, and one `<script>` IIFE. No build step, no dependencies. |
| `README.md` | Placeholder (title only). |

There is no package manager, test suite, linter, or bundler. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Code layout inside `index.html`

- **`<style>`** (around lines 10–360): all styling. Dark color scheme (`:root { color-scheme: dark; }`), HUD pill styles, overlay card, touch controls, and `.reduced-motion` overrides.
- **Markup** (around lines 362–420): `#container` holds the `#gameCanvas`, the HUD (`#hud`), boss bar, banners, top-right icon buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), touch controls (`#touchControls`), and the start/pause overlay (`#overlay`).
- **`<script>`** (starts around line 422), wrapped in `(function () { 'use strict'; ... })()`. Sections in order:
  - DOM references and `STORAGE_*` keys
  - Adaptive quality (`trackPerf`, `quality` 0–2) that lowers effect detail when frame time degrades
  - Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`
  - Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object. Audio is synthesized with WebAudio, with no audio files.
  - Input: keyboard handlers, touch joystick, fire/bomb buttons, mouse steering on the canvas
  - Game state and logic: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `useBomb`, `fireWeapon`, `applyPowerUp`, `loseLife`, `endGame`, `togglePause`
  - Boss attack patterns: `BOSS_PATTERNS` plus the `fire*` helpers, driven by `updateBoss`
  - Rendering: `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`
  - Main loop: `loop(now)` calls `requestAnimationFrame`

Keep new code in the matching section. Do not split the file into modules unless asked, because the project is intentionally a single self-contained page.

## Gameplay and controls (keep docs and UI text in sync)

- **Keyboard:** Arrows/WASD move, Space fires, X or Shift bombs, P or Esc pauses.
- **Mouse:** move to steer, click to fire.
- **Touch:** drag on the left side to fly, FIRE and BOMB buttons on the right.
- **Difficulty:** `easy`, `normal` (default), `hard`. Each sets spawn rate, enemy shot chance, starting lives and bombs, and boss HP multiplier.
- **Progression:** after `KILLS_PER_STAGE` (18) kills in a stage, a boss from `BOSSES` spawns. Defeating it advances to the next stage, whose sky palette comes from `THEMES[(stage - 1) % THEMES.length]`.
- **Weapons:** Vulcan and Laser, each with power levels. Bombs clear the screen and damage bosses.

If you change a control, update the `#controlsHint` text in the markup as well.

## Persistence

`localStorage` keys, all prefixed `skyfighter.`:

- `bestScore`: high score
- `muted`: `'1'` when muted
- `difficulty`: selected difficulty name
- `reducedMotion`: `'1'` or `'0'`. If unset, it follows `prefers-reduced-motion`.

Reads of `localStorage` are not wrapped in try/catch today. If you touch that code, consider guarding it, because blocked storage would throw at startup.

## Conventions

- Plain HTML, CSS, and JavaScript with no frameworks or external libraries. Keep it that way unless asked.
- Keep all code inside the existing IIFE and `'use strict'` scope. Do not add globals.
- Game-logic values are tuning constants in the config tables near the top of the script. Prefer changing those over hard-coding numbers in logic.
- Preserve accessibility hooks: `aria-live` announcements via `announce()` and the `srLive` region, plus the reduced-motion path. New screen shake, flashes, or pulses should respect `reducedMotion`.
- The page is responsive and must work on phones. The canvas scales to fit a `480×720` base (`BASE_W`, `BASE_H`) through `resize()`, and touch controls must stay usable.
- Do not add external CDN scripts or fonts. The page works offline as a single file.

## Verifying changes

There are no automated tests. For a change to `index.html`:

1. Open the page in a browser, or serve it locally, and check for console errors.
2. Play through start, pause/resume, bomb, a stage transition, and game over.
3. Check a narrow viewport, since the layout and touch controls are responsive.
4. For visual or layout changes, take a screenshot before and after.

## Git

- Development branch: `claude/gifted-tesla-wrzj8l`. Commit and push there.
- Commit messages are short and imperative, for example `Add Sky Fighter: ...`.

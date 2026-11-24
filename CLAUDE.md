# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on an HTML5 `<canvas>`.

Files:

- `index.html` — the entire game: markup, CSS (inline `<style>`), and JavaScript (inline `<script>`) in one file (~1,670 lines).
- `README.md` — a one-line title. It has no setup or usage instructions yet.

There is no build step, package manager, bundler, test suite, linter, or CI configuration. Do not invent one unless the user asks.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

No install step is required.

## Code layout inside `index.html`

- **`<head>`**: meta tags (viewport locked to device width, theme color `#0b1d3a`), an inline SVG favicon, and the `<style>` block (roughly lines 10–360). Styles cover the HUD, boss bar, stage banner, touch controls, and overlay.
- **`<body>`** (roughly lines 361–421): a `#container` holding the `#gameCanvas`, HUD elements (`#hud`, `#topBar`, `#bossBarWrap`, `#comboPop`, `#stageBanner`), touch controls (`#touchControls`), and the start/pause/game-over `#overlay`. `#srLive` is an aria-live region for screen readers.
- **`<script>`** (roughly lines 422 onward): a single IIFE that holds all game state and logic. Main pieces, in file order:
  - DOM lookups and `localStorage` keys (`skyfighter.*`) for best score, mute, difficulty, and reduced motion.
  - Config tables: `DIFFICULTIES` (easy/normal/hard tuning), `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six stage bosses), and `KILLS_PER_STAGE`.
  - Audio: a lazily created `AudioContext`, with `beep`, `noiseBurst`, and a `sfx` map. Audio is created only on user gesture.
  - Input: keyboard (`keys`), touch joystick and fire/bomb buttons, and mouse targeting.
  - Game flow: `resetGame`, `spawnEnemy`, `spawnBoss`, `update(dt)`, `updateBoss(dt)`, `draw()`, and `loop(now)` driven by `requestAnimationFrame`.
  - Adaptive quality: `trackPerf` samples the last 45 frame times and steps `quality` down (0 full → 2 low) when the average is slow, and back up when it is fast.
  - Accessibility: `announce()` feeds `#srLive`, and a reduced-motion toggle controls screen shake and flashes.

Everything is wrapped in an IIFE, so nothing is exposed on `window`. Keep new code inside the IIFE unless there is a reason not to.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not add a bundler, framework, or npm dependencies for this project.
- **No external assets or network calls.** The game uses only Canvas 2D, Web Audio, and `localStorage`. Do not add CDN links, remote fonts, or remote images without asking, because the game is meant to work offline when opened from disk.
- **Match the existing style.** Two-space indentation, single quotes in JavaScript, `const` by default, and `camelCase` identifiers. Constants that describe tables use `UPPER_SNAKE_CASE`.
- **Game tuning lives in the config tables.** Change difficulty, enemy stats, boss HP multipliers, and stage palettes in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES` rather than hard-coding numbers in the update loop.
- **Use the base resolution.** The canvas logic is built around `BASE_W = 480` and `BASE_H = 720`, scaled by `resize()`. Coordinates should be in base units.
- **Respect the time step.** `update(dt)` takes a delta time. New movement or timers should scale with `dt`, not frame count.
- **Keep the mobile layout working.** The viewport disables user scaling, and touch controls are shown on touch devices. Check any HUD or overlay change at phone width.
- **Keep accessibility hooks.** Announce meaningful state changes through `announce()` and honor `reducedMotion` for shake and flash effects.

## Persistence

The game stores four values in `localStorage`, all under the `skyfighter.` prefix:

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | Best score |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | `'1'` or `'0'`; absent means use the OS preference |

If you rename or change the format of a key, handle existing values in browsers that already have data. Wrap new storage access in `try/catch` if it might run in a private window or with storage blocked.

## Testing and verification

There is no automated test suite. To verify a change:

1. Open `index.html` in a browser (or serve it as above) and check the browser console for errors.
2. Play through at least one stage: move, fire, drop a bomb, collect power-ups if affected, and reach a boss if the change touches combat or stages.
3. Check each difficulty if you changed `DIFFICULTIES` or enemy stats.
4. Check the touch layout at phone width if you changed HUD, overlay, or controls.
5. Check mute, pause, and reduced-motion toggles if you changed audio, effects, or the overlay.

If you can drive a headless browser (Playwright/Chromium is usually available in this environment), a short scripted smoke test is a good way to confirm the page loads without console errors. Do not commit such a script unless the user asks for one.

Be explicit in your summary about what you actually ran and what you only checked by reading the code.

## Git and workflow

- Default working branch for this session: `claude/gifted-tesla-t73zhl`. Do not push to other branches without explicit permission.
- Make focused commits with clear messages that describe the gameplay or code change.
- Do not open a pull request unless the user asks for one.
- Do not commit secrets or local settings files (`.env`, `.claude/settings.local.json`, and similar).

## When making changes

- Prefer small, targeted edits to `index.html`. Use the Edit tool rather than rewriting the file.
- Do not reformat the whole file or reorder unrelated code; that makes diffs hard to review.
- If a change affects game balance, mention the before and after values in your summary so the user can judge the effect.
- Update this file when the structure, conventions, or workflow change. Keep it accurate rather than long.

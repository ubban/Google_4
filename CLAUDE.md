# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a single browser canvas.

| File | Purpose |
| --- | --- |
| `index.html` | The whole game: markup, inline CSS, and inline JavaScript (~1,670 lines). |
| `README.md` | One-line project title. |

There is no build system, package manager, test suite, or bundler. Nothing needs to be installed.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Code layout inside `index.html`

- **`<head>`**: viewport meta (no zoom, `viewport-fit=cover`), theme color `#0b1d3a`, and an inline SVG favicon. The page sets `color-scheme: dark`.
- **`<style>`**: all styles, including the HUD, overlay, stat row, boss bar, vignette/flash effects, and touch controls.
- **Markup**: `#container` wraps `#gameCanvas`, the HUD (`#score`, `#best`, `#mult`, `#weaponName`, `#weaponLvl`, `#hearts`, `#bombs`), the overlay (`#overlay`, `#startBtn`, `#finalScore`, etc.), boss bar (`#bossBarWrap`), touch controls (`#touchControls`, `#moveZone`, `#fireBtn`, `#bombBtn`), and the mute/motion/pause buttons.
- **`<script>`** (one IIFE-style block starting around line 426):
  - DOM lookups are grabbed once at the top.
  - Persistent settings use `localStorage` keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`.
  - Game tuning constants: `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES` (per-stage backgrounds), `ENEMY_TYPES`, `BOSSES`.
  - Audio is synthesized with the Web Audio API (`ensureAudio`, `beep`, `noiseBurst`, the `sfx` table). Audio context is created lazily on the first user gesture.
  - Adaptive quality: `trackPerf` samples frame times and steps `quality` (0 full, 1 medium, 2 low).
  - Input handlers: keyboard (`keydown`/`keyup`, arrows or WASD), touch joystick and fire/bomb buttons, and mouse on the canvas.
  - Game loop: `loop(now)` calls `update(dt)` and `draw()` each frame via `requestAnimationFrame`. Enemy and boss logic is in `spawnEnemy`, `spawnBoss`, and `updateBoss`; bombs in `useBomb`; rendering in `drawPlane`, `drawBossShip`, and `drawClouds`.

## Conventions

- **Keep it a single file.** Game logic, styles, and markup all live in `index.html`. Do not split out modules, a bundler, or a framework unless the user asks for it.
- **Vanilla JS and Canvas 2D only.** Do not add dependencies or CDN scripts without an explicit request.
- **Preserve the existing style**: 2-space indentation, `const`/`let`, single-quoted strings, and terse inline comments.
- **Respect the accessibility hooks.** Honor `prefers-reduced-motion` and the in-game reduced-motion toggle, keep the `srLive` announcements working, and keep touch controls usable on phones (`touch-action: none`, safe-area insets).
- **Mobile first.** The layout must work on phone-sized viewports; test responsive changes at narrow widths.
- **Don't break saved state.** Changing the `skyfighter.*` storage keys or their value formats resets players' best scores and settings. Make such changes deliberately.

## Verifying changes

There are no automated tests or linters. To verify a change:

1. Open `index.html` (or serve it) and start a run from the overlay.
2. Exercise the path you changed: movement and firing, bombs, weapon upgrades, stage transitions, boss fights, pause/mute, and difficulty selection.
3. Check the browser console for errors.
4. For layout changes, check both a desktop viewport and a narrow phone viewport.

If you can run a headless browser (Playwright/Chromium is typically available in this environment), a short scripted smoke test that loads the page and checks for console errors is a good substitute for manual testing.

## Git

- Development branch for this work: `claude/gifted-tesla-7valor`. Push there; do not push to other branches without explicit permission.
- Do not open pull requests unless the user asks for one.
- Keep commit messages short and descriptive, in the imperative mood.

## Notes for future changes

- The game's entire state is held in top-level `let` variables inside the script; new features should follow the same pattern rather than introducing global state elsewhere.
- Any new asset should be inlined (SVG data URIs, Web Audio) because the project is a single HTML file.

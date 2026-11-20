# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small static web project. The only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up, in a single self-contained file:

- `index.html` (~1,670 lines): all markup, CSS (`<style>` block), and JavaScript (one `<script>` IIFE) in one file.
- `README.md`: minimal placeholder.

There is no build step, no package manager, no bundler, no framework, no tests, and no external dependencies. The only external reference is an inline SVG favicon as a data URI. Do not add npm/Node tooling, CDN scripts, or extra asset files unless the user asks for them.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). There are no commands to build, lint, or test.

## Layout of `index.html`

Roughly in file order:

1. **`<head>` / `<style>`** (lines ~1–361): inline CSS. Responsive sizing, HUD, overlays, touch controls, and the `reduced-motion` body class.
2. **Markup** (lines ~362–420): `#container` holds the `<canvas id="gameCanvas">`, HUD (`#hud`, `#score`, `#best`, `#mult`, `#weaponName`, `#hearts`, `#bombs`), boss bar (`#bossBarWrap`), top buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), touch controls (`#touchControls`, `#fireBtn`, `#bombBtn`, joystick elements), and the start/title overlay (`#overlay`, `#startBtn`, `#diffRow`).
3. **Script** (lines ~422–1667): one IIFE with `'use strict'`. Sections, in order:
   - DOM lookups and constants: `STORAGE_*` keys, `announce()` for screen-reader text, `trackPerf()` adaptive quality (0 full / 1 medium / 2 low).
   - Tuning data: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage), `ENEMY_TYPES`, `BOSSES`.
   - Audio: `ensureAudio()`, `beep()`, `noiseBurst()`, `sfx.*` (Web Audio, synthesized, no audio files). `muted` is persisted.
   - HUD and FX helpers: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`.
   - Input: keyboard (`keys`), touch joystick and buttons, and mouse steering on the canvas.
   - Game state and loop: `resetGame()`, `update(dt)` (game logic), `draw()` (rendering), `loop(now)` (requestAnimationFrame; `dt` is clamped to 0.05 s).
   - Weapons and power-ups: `fireWeapon()`, `applyPowerUp()`, `maybeSpawnPowerUp()`, `useBomb()`.
   - Bosses: `BOSS_PATTERNS` (bullet-pattern functions such as `fireRadial`, `fireCross`, `fireHomingSeeded`), `spawnBoss()`, `updateBoss()`, `killBoss()`, `drawBossShip()`.
   - Flow: `loseLife()`, `endGame()`, `togglePause()`.

## Conventions

- **Keep it one file.** New game logic, styles, and markup go in `index.html` unless the user asks for a split.
- **Vanilla JS and Canvas 2D only.** Use the existing patterns: `const` by default, the `'use strict'` IIFE, and `dt`-based (time-scaled) movement.
- **Persisted settings use `localStorage`** with the `skyfighter.*` key prefix (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Wrap new storage access so the game still works if storage is unavailable.
- **Accessibility:** respect `prefers-reduced-motion` (the `reducedMotion` flag and `reduced-motion` body class). Announce important state changes through `announce()` / `#srLive`.
- **Performance:** the game adapts draw quality with `quality`. Keep per-frame allocations low in `update`/`draw`, since they run every frame.
- **Mobile:** the layout must work on touch devices (joystick + FIRE/BOMB buttons) and with the keyboard/mouse controls, which are listed in `#controlsHint`. Keep both in sync when controls change.
- **Tuning changes** (enemy stats, difficulty tables, boss HP multipliers) belong in the constant tables near the top of the script, not inline in logic.

## Git workflow

- Development branch: `claude/gifted-tesla-nqx7ad`. Commit and push there. Do not push to other branches without explicit permission.
- Do not create pull requests unless the user explicitly asks.
- Keep commits focused, with a short imperative subject (existing history example: `Add Sky Fighter: Raiden-style vertical shoot-em-up game`).

## Verification

With no tests or build, verify changes by opening `index.html` in a browser (or a headless Chromium) and playing through: start, movement, firing, a power-up, a boss fight, pause/resume, game over, and best-score persistence. For UI-only changes, also check the layout at phone width.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas app.

- `index.html` (~1,670 lines) is the whole game: markup, CSS, and one IIFE-wrapped `<script>` block. There is no build step, no package manager, and no test suite.
- `README.md` is a placeholder.

To run the game, open `index.html` in a browser, or serve the directory with any static server.

## Layout of `index.html`

- **`<head>`**: meta tags, an inline SVG favicon (data URI), and an inline `<style>` block (about lines 10–420). The styles cover the HUD pills, the glassmorphic overlay card, the touch controls, the vignette and flash effects, and the boss bar.
- **`<body>`**: `#container` holds the `#gameCanvas`, the `#hud`, the `#touchControls` (joystick, FIRE, BOMB), and the `#overlay` (start and game-over card with `#startBtn`, `#diffButtons`, and `#statsRow`). A visually hidden `#srLive` region is used for screen-reader announcements.
- **`<script>`** (starts about line 422): a single IIFE with these sections, in roughly this order:
  - DOM references and `STORAGE_*` keys
  - Perf monitor (`trackPerf`), which steps through three quality tiers
  - Config constants: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`
  - Settings persistence and reduced-motion handling
  - Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, panning)
  - Entity spawning (`spawnEnemy`, `spawnBoss`, `spawnExplosion`)
  - `update(dt)` and `updateBoss(dt)`, which hold the simulation
  - Rendering (`drawPlane`, `drawBossShip`, `drawClouds`, `draw()`)
  - `loop(now)`, the `requestAnimationFrame` driver, near the end of the file

Everything is in one file on purpose. Keep it that way unless the user asks for a split.

## Game design facts (keep in sync with code)

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. These set spawn rate, enemy fire chance, lives, starting bombs, and boss HP multiplier.
- **Enemies** (`ENEMY_TYPES`): `grunt` (stage 1+), `interceptor` (stage 1+, weaves), `tank` (stage 2+, 4 HP).
- **Bosses** (`BOSSES`): six designs that cycle by stage: Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan. Each has its own attack patterns and a telegraph flash before firing.
- **Stages**: advance after `KILLS_PER_STAGE` (18) kills, then a boss fight. Backgrounds cycle through `THEMES`.
- **Scoring**: kill-chain multiplier, `1 + min(9, floor(chain / 5))`.
- **Power-ups**: `life`, `bomb`, `weapon`. Weapons are Vulcan and Laser, each with 3 levels.
- **Smart bombs** clear the screen and damage bosses.

## Controls

- Keyboard: arrows or WASD to move, Space to fire, X or Shift for a bomb, P or Esc to pause.
- Mouse: moving over the canvas steers the plane (capped at 1.7× speed). Holding the button fires. Keyboard and touch input take priority when active.
- Touch: drag left to fly, tap FIRE, tap BOMB.

## Persistence

`localStorage` keys, all prefixed `skyfighter.`:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Reads and writes are wrapped where needed, but a storage failure should never break gameplay. Keep it that way when touching these paths.

## Conventions

- **Vanilla only.** No frameworks, no bundler, no npm dependencies. The only external resources are the inline favicon and system fonts.
- **Keep the single-file structure** unless asked otherwise.
- **Accessibility matters.** Honor `prefers-reduced-motion`. Announce important state changes through `#srLive`. Keep touch targets usable on phones, and keep the `touch-action: none` and `user-select: none` behavior on the game surface.
- **Performance.** Hot paths run every frame in `update` and `draw`. Avoid per-frame allocations where practical, and respect the quality tiers. Glow and shadow-blur effects are the first things the perf monitor drops.
- **Audio** goes through the shared `masterGain` and compressor. Create the `AudioContext` lazily on user gesture (`ensureAudio`).
- **Style.** Match the existing code: two-space indentation, single quotes in JS, short helper functions, and comments only where the intent isn't obvious.

## Verifying changes

There is no automated test suite. For gameplay or rendering changes:

1. Open `index.html` in a browser, or run a headless Playwright session. Playwright is preinstalled in the cloud environment. Use the existing Chromium and do not reinstall it.
2. Check the browser console for errors.
3. Exercise the paths you changed: firing, collisions, pickups, boss spawn and death, stage advance, pause, mute, reduced motion, and each difficulty.
4. Check the layout at phone width, since the HUD and touch controls must still fit.

Say explicitly in your summary which of these you verified and which you did not.

## Git

- Development branch for this work: `claude/gifted-tesla-3em1hf`. Push there, and don't push to other branches without explicit permission.
- Commit messages in this repo are descriptive and sometimes multi-paragraph. Follow the style of the existing history.
- Don't open a pull request unless the user asks for one.

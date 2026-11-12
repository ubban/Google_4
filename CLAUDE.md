# CLAUDE.md — Google_4 (Sky Fighter)

Guidance for Claude Code when working in this repository.

## What this repo is

A small static web project. Its only real content is **Sky Fighter**, a Raiden-style
vertical shoot-'em-up written as a single-page HTML5 canvas game.

| File        | Purpose                                                        |
|-------------|----------------------------------------------------------------|
| `index.html` | The entire game: markup, CSS (`<style>`), and JS (`<script>`) in one file (~1,670 lines). |
| `README.md`  | Placeholder. Currently contains only a title and a stray timestamp. |

There is no build system, no `package.json`, no bundler, no test suite, and no linter
configuration. Do not invent commands for those. To run the game, open `index.html` in a
browser (or serve the directory with any static server).

## Architecture of `index.html`

- **Layout:** a `<canvas id="gameCanvas">` (logical size `BASE_W=480 × BASE_H=720`,
  scaled by `resize()`), a HUD built from DOM elements (score, multiplier, weapon, hearts,
  bombs, boss bar), an overlay card for title/pause/game-over, and on-screen touch
  controls (joystick zone, fire button, bomb button).
- **JS:** one IIFE wrapped in `'use strict'`. Everything is private to the closure; nothing
  is exported to `window`. Keep it that way unless there is a concrete need.
- **Main loop:** `update(dt)` advances game state, `draw()` renders, and `loop(now)` ties
  them to `requestAnimationFrame`. Game start is wired at the bottom via `startBtn`.
- **Config tables** live near the top of the script and drive balance. Edit these rather than
  hard-coding numbers in logic:
  - `DIFFICULTIES` — `easy` / `normal` / `hard` (spawn rate, enemy speed, shoot chance,
    starting lives and bombs, boss HP multiplier).
  - `KILLS_PER_STAGE` — kills needed to advance a stage.
  - `THEMES` — sky palettes cycled per stage.
  - `ENEMY_TYPES` — `grunt`, `interceptor` (weaves), `tank` (4 HP, minimum stage 2).
  - `BOSSES` — boss names, colors, and HP multipliers; attack patterns are in `BOSS_PATTERNS`.
- **Power-ups** (`applyPowerUp`): `life` (+1 life, capped), `bomb` (+1 bomb, capped), and
  `weapon`, which raises `player.level` up to 3 and then switches between Vulcan and Laser,
  resetting to level 1. Drawn in `draw()` with fixed colors.
- **Adaptive quality:** `trackPerf()` lowers the `quality` level (0 full / 1 medium / 2 low)
  when frame time is high. Rendering code respects this level; preserve it when adding effects.

## Persistence and accessibility

- `localStorage` keys are namespaced with `skyfighter.`: `bestScore`, `muted`, `difficulty`,
  `reducedMotion`. Reuse the existing `STORAGE_*` constants and keep the prefix for new keys.
- Every `localStorage` access must tolerate being unavailable (private mode, blocked storage).
  Wrap new reads and writes in `try/catch` if you add them outside the existing pattern.
- Reduced motion defaults to the `prefers-reduced-motion` media query and can be toggled
  in the UI. Screen shake, flashes, and similar effects must honor it.
- Screen-reader announcements go through `announce()`, which writes to the `#srLive` region.
  Use it for new status messages instead of adding ad hoc live regions.

## Input

- **Keyboard:** arrows or WASD to move; key state is tracked in `keys[e.code]`.
- **Mouse:** pointer follows the canvas (`mousemove`), mousedown fires.
- **Touch:** joystick on the left (`moveZone`), fire and bomb buttons on the right.
  Touch handlers call `preventDefault` with `{ passive: false }`. Keep that when editing them.
- Audio is created lazily through `ensureAudio()`, which must run from a user gesture
  (browsers block autoplay). Keep calls to it inside input handlers.

## Working conventions

- **Keep it a single file** unless the user asks for a split. Changes should stay in
  `index.html` and should not add a framework or build step.
- **Match the existing style:** 2-space indentation, single quotes in JS, `const` by default,
  section comments like `// ---------- Name ----------`, and camelCase names.
- **Don't break the public-facing behavior** that the commit history describes: persistence,
  mobile controls, pause, difficulty, power-ups, bosses, and the kill-chain multiplier.
- **Verify changes in a browser.** There is no automated test, so after editing game logic,
  load the page and play through start, a stage transition, a boss fight if you touched bosses,
  pause/resume, and game over. Check the touch layout at phone width too.
- When adding a value to a config table, check every place that reads that table
  (for example, every `DIFFICULTIES[difficulty]` lookup) to make sure the new key is handled.

## Git

- Default development branch for this session: `claude/gifted-tesla-pfqmtd`. Do not push to
  other branches without explicit permission.
- Commit messages in history are descriptive, with a short summary line followed by bullet
  points for the changes.
- Do not open a pull request unless the user explicitly asks for one.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up written as a single-page HTML5 canvas game.

| File        | Purpose                                                        |
|-------------|----------------------------------------------------------------|
| `index.html`| The whole game: markup, inline CSS, and one inline `<script>`  |
| `README.md` | Minimal project readme                                         |

There is no build system, package manager, test suite, or backend. Nothing needs to be
installed. Open `index.html` directly in a browser, or serve the directory with any static
server (for example `python3 -m http.server`) to play it.

## Architecture of `index.html`

- **Single file, no dependencies.** Keep it that way unless the user asks otherwise. Do not
  add frameworks, bundlers, or CDN scripts without explicit approval.
- **Inline CSS** in `<style>` (roughly lines 10–360). HUD, overlay card, touch controls, and
  the boss bar are all styled here. Layout uses CSS `clamp()`, `env(safe-area-inset-*)`,
  and flexbox so it works on phones.
- **Markup** (roughly lines 362–420): `#container` holds the `<canvas id="gameCanvas">`, HUD
  pills (`#hud`), boss bar, banners, top-right icon buttons (mute, reduced motion, pause),
  touch controls (joystick zone plus FIRE and BOMB buttons), and the start/title `#overlay`.
- **Script** (roughly lines 422 to the end) is one IIFE wrapped in `'use strict'`. Its sections, in order:
  1. DOM references and `STORAGE_*` keys
  2. Adaptive performance (`quality` 0/1/2, driven by frame-time averages)
  3. Config tables: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`
  4. Audio: a lazily created `AudioContext` (`ensureAudio`), plus `beep` and `noiseBurst`
     synth helpers. Audio starts only after a user gesture.
  5. UI effects: hearts and bombs rendering, hit flash, combo pop, stage banner, screen shake
  6. Input: keyboard, joystick, touch buttons, and mouse on the canvas
  7. Game state: `resetGame`, `update(dt)`, `draw()`, `loop(now)` driven by `requestAnimationFrame`
  8. Gameplay: enemy spawning, power-ups, bombs, weapon firing, boss patterns (`BOSS_PATTERNS`),
     life loss, and game over
- **Game loop:** `update(dt)` advances state, then `draw()` renders it. Keep those two
  responsibilities separate. Do not put rendering side effects into `update`.
- **Canvas coordinates:** the game is authored at `BASE_W = 480`, `BASE_H = 720` and scaled
  by `resize()`. Positions and sizes in game code use these base units.

## Gameplay facts to preserve

- Difficulty (`easy` / `normal` / `hard`) sets spawn rate, enemy speed, enemy fire chance,
  lives, starting bombs, and boss HP multiplier.
- Stages advance every `KILLS_PER_STAGE` (18) kills. Each stage ends in a boss, and the
  stage cycles through `THEMES` and `BOSSES`.
- Enemy types: `grunt`, `interceptor` (weaving), `tank` (4 HP, minimum stage 2).
- Weapons: Vulcan and Laser, each with 3 power levels. Weapon power-ups raise the level.
- Power-ups are `life`, `bomb`, and `weapon`.
- Score uses a kill-chain multiplier (`addScore`, `breakChain`).
- Bombs (`useBomb`) clear enemy bullets and enemies on screen and damage bosses.

## Persistence

`localStorage` keys, all prefixed `skyfighter.`:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Every `localStorage` access should stay tolerant of failure (private mode, blocked storage).
If you add a new key, use the same prefix and document it here.

## Accessibility and mobile

- Keep the `prefers-reduced-motion` path working. It disables screen shake, flashes, and
  pulses. Toggling it writes `skyfighter.reducedMotion`.
- Keep the `aria-live` region (`#srLive`) updated through `announce()` for important events.
- Touch: `touch-action: none` and `user-select: none` are set deliberately. Preserve them
  on the game surface so gestures don't scroll or zoom the page.
- Controls must work on both keyboard/mouse and touch. If you add an input, add it to both.
  Documented controls are listed in `#controlsHint`.

## Conventions

- Plain ES2017+ JavaScript, `const`/`let`, arrow functions, no modules.
- Keep the code in the single IIFE. Do not leak globals.
- Match the existing style: two-space indent, single quotes in JS, small named functions,
  and config tables for tunable numbers instead of magic numbers in logic.
- Prefer adjusting values in the config tables over changing logic when tuning balance.
- Keep comments short and only where the intent is not obvious. Section headers use the
  `// ---------- Name ----------` style.

## Working in this repo

- **Verify in a browser.** Nothing here is unit-tested. After gameplay or UI changes, load
  the page and play through start, a stage transition, a boss, bomb use, pause, and game
  over. Check both a desktop viewport and a phone-sized viewport.
- **Keep changes focused.** The file is large, so make targeted edits and avoid reformatting
  unrelated code.
- **Git:** develop on the branch the session specifies (currently `claude/gifted-tesla-i6xhhp`).
  Do not push to other branches without explicit permission. Do not open a pull request
  unless the user asks for one.
- **Commit messages:** imperative summary line, then a short body explaining the gameplay or
  UI change, following the style of the existing commit history.
- Do not commit secrets. This repo has none today, so keep it that way.

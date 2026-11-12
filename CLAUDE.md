# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small, dependency-free static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on a single HTML5 `<canvas>`.

Files:

- `index.html` (~1,670 lines) — the entire game: markup, CSS in one `<style>` block (starts ~line 10), and all JavaScript in one `<script>` block (starts ~line 422). No build step, no bundler, no npm.
- `README.md` — placeholder title only.

There are no tests, no linter config, and no package manifest.

## Running it

Open `index.html` directly in a browser, or serve the directory statically (e.g. `python3 -m http.server`). Nothing needs to be installed.

## Code layout inside `index.html`

The script is one IIFE (`(function () { 'use strict'; ... })();`). Everything is closure-scoped; nothing is exposed on `window`. Keep it that way unless there is a reason.

Major sections, roughly in file order:

- **DOM references** — `document.getElementById(...)` lookups at the top of the IIFE. If you add a HUD element, add its lookup here.
- **Storage keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`, all prefixed `skyfighter.` in `localStorage`.
- **Tuning tables** — `DIFFICULTIES` (spawn rates, enemy speed, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky/cloud/sun palettes), `ENEMY_TYPES` (grunt / interceptor / tank), `BOSSES` (six bosses with colors and HP multipliers). Balance changes usually belong here, not in logic.
- **Audio** — Web Audio synthesis only (`ensureAudio`, `beep`, `noiseBurst`, `panFor`). No audio files. Audio is created lazily and must be unlocked by a user gesture (`ensureAudio()` is called from input handlers).
- **HUD / feedback** — `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (screen-reader live region).
- **Input** — keyboard (`keys[e.code]`), mouse (`canvasPointFromEvent`), and touch (virtual joystick on `moveZone`, `fireBtn`, `bombBtn`). Touch handlers call `preventDefault`, so keep `{ passive: false }` where it's set.
- **Rendering** — `drawPlane`, `drawBossShip`, plus background/particle drawing. Drawing uses a fixed logical canvas of `BASE_W = 480` × `BASE_H = 720`, scaled by `resize()`.
- **Game state & update** — `resetGame()` initializes state; `update(dt)` runs each frame; `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `addScore`/`breakChain` (kill-chain multiplier).
- **Boss patterns** — `BOSS_PATTERNS` plus helpers `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`. `updateBoss(dt)` drives the active boss.
- **Main loop** — `loop(now)` is driven by `requestAnimationFrame`, computes `dt`, and calls `trackPerf`. `startBtn` starts a run.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Don't introduce a build system, framework, or external CDN dependency without a clear request.
- **Vanilla JS, ES2015+ syntax** as already used (`const`/`let`, arrows, template literals). Match the existing style: 2-space indent, single quotes, semicolons, `'use strict'` inside the IIFE.
- **Keep state in closure variables.** Don't attach game state to `window`.
- **Time-based updates.** Movement and timers are driven by `dt` from the loop; don't hard-code per-frame increments that would break at other refresh rates.
- **Respect reduced motion.** The `reducedMotion` flag gates `screenShake()` (it scales magnitude down and caps duration), and the `reduced-motion` body class suppresses flashes, vignette pulses, and entrance animations in CSS. New visual effects that shake or flash should honor it.
- **Persist only via the existing `skyfighter.*` keys** in `localStorage`, and wrap new storage access in care for private-mode failures if you add it.
- **Accessibility.** Announce important state changes through `announce()` so the `srLive` region picks them up. Keep the mute, motion, pause, and difficulty controls working by keyboard and touch.
- **Mobile.** The layout must keep working on touch devices: the `#touchControls` panel is enabled only when `'ontouchstart' in window` or `navigator.maxTouchPoints > 0`, and `resize()` handles orientation changes.

## Making changes

- Read the surrounding section before editing; most logic is tightly coupled through shared closure variables.
- For balance tweaks, edit the tuning tables first (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, `KILLS_PER_STAGE`).
- Verify changes by actually running the game in a browser (play a run, check HUD, bombs, boss fight, pause/resume, mute, difficulty switch, and a mobile-sized viewport). There is no automated test suite to rely on.
- Keep commits focused and describe gameplay-visible changes in the commit message.

## Git

- Default branch: `main`. Feature work for this environment goes on the branch specified by the session setup (currently `claude/gifted-tesla-mkbjyk`).
- Don't open pull requests unless explicitly asked.

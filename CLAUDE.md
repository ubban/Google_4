# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small, dependency-free browser game, **Sky Fighter** (a Raiden-style vertical shoot-'em-up). The entire game lives in one file, `index.html`:

- `<style>` (~lines 10–361): all CSS, including the HUD, overlay cards, touch controls, and responsive layout.
- `<body>` markup (~lines 362–421): the `#container`, `<canvas id="gameCanvas">`, HUD elements, touch controls, and the start/pause overlay.
- `<script>` (~line 422 onward): one IIFE with `'use strict'` that holds all game logic and rendering.

`README.md` is a one-line placeholder. There is no build step, package manager, test suite, linter, or CI config.

## Running and verifying

- Open `index.html` directly in a browser. There is no server or build step required.
- There are no automated tests. Verify changes by playing the game in a browser (desktop keyboard/mouse and, where possible, a touch-sized viewport), checking the overlay, HUD, and touch controls.
- For browser-level checks, Playwright is pre-installed in the remote environment. Use `require('playwright')` and `chromium.launch()`; do not run `playwright install`.

## Code layout inside the script

Read these top-level sections before changing behavior. They are in source order:

| Area | What it holds |
|---|---|
| DOM references | `const xxxEl = document.getElementById(...)`. Element IDs must match the markup. |
| `STORAGE_*` keys | `localStorage` keys prefixed `skyfighter.` (best score, mute, difficulty, reduced motion). |
| Adaptive quality | `trackPerf` samples frame times and steps `quality` 0–2 to lower effects when frames run slow. |
| `DIFFICULTIES` | Per-difficulty tuning: spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier. |
| `THEMES`, `ENEMY_TYPES`, `BOSSES` | Data tables for background themes (changes per stage), enemy stats, and boss definitions. |
| Audio | Lazy `AudioContext` via `ensureAudio()`, with `beep` / `noiseBurst` helpers. Mute is persisted. |
| Input | Keyboard (`keydown` on `window`), mouse, pointer-based joystick and touch buttons. |
| `resetGame`, `update(dt)`, `draw()` | Game state reset, per-frame simulation, and canvas rendering. |
| Weapons and bosses | `applyPowerUp`, `fireWeapon`, `BOSS_PATTERNS` (bullet pattern functions), `updateBoss`, `killBoss`. |
| `loop(now)` | `requestAnimationFrame` loop. `dt` is clamped to 0.05s. Update and draw only run when `running && !paused`. |

Game state is plain module-level `let` variables inside the IIFE (score, lives, stage, bombs, enemy and bullet arrays, and so on). Nothing is exported and there are no globals to preserve.

## Conventions

- **Single file, vanilla JS, no libraries.** Do not introduce frameworks, bundlers, or CDN dependencies without an explicit request. Keep everything in `index.html` unless the user asks for a split.
- **Keep the IIFE and `'use strict'`.** New code belongs inside the existing IIFE, not in new globals.
- **Time-based movement.** Use `dt` (seconds) for movement and timers so behavior is frame-rate independent.
- **Data-driven tuning.** Put numbers in the `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES` tables rather than hard-coding them in logic.
- **Storage keys** use the `skyfighter.` prefix. Wrap new `localStorage` reads/writes so the game still works if storage is blocked.
- **Accessibility:** announcements go through `announce()` into the `#srLive` live region. Respect `prefers-reduced-motion` and the in-game reduced-motion toggle when adding screen shake, flashes, or pulses.
- **Responsive layout:** sizes use `clamp()` and viewport units. Canvas logical size is `BASE_W × BASE_H` (480×720) and is scaled by `resize()`. Keep the phone layout working with touch controls.
- **Keep changes minimal.** Match the existing style (2-space indent, single quotes in JS, short comments with `// ---------- Section ----------` headers).

## Git workflow

- Develop on the branch assigned for this session (currently `claude/gifted-tesla-iouttt`). Do not push to other branches without explicit permission.
- Commit only when asked. Do not open a pull request unless the user explicitly asks for one.
- Do not commit secrets or local settings files such as `.claude/settings.local.json`.

## Known gaps

- No tests, linting, or formatting tooling exists. Add them only if the user asks.
- `README.md` is a placeholder and does not describe the game. Update it if the user wants real project docs.

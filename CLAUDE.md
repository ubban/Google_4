# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free browser game called **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game lives in a single file, `index.html` (~1,670 lines), with inline `<style>` (lines ~10–361), markup (~362–421), and one inline `<script>` (starting ~line 422). There is no build step, no package manager, and no test suite.

Other files:
- `README.md` — a one-line title only.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There is no build, lint, or test command configured. Verify changes by loading the page and playing; for scripted checks, a headless Chromium with Playwright works.
- Keep the game working without any network access. Do not add external scripts, fonts, or CDN links unless the user asks for them.

## Architecture (inside `index.html`)

The script is a single `'use strict'` IIFE (starts at line ~423, closes at the end of the file) and is organized into sections marked by `// ---------- ... ----------` comments. Key pieces:

- **Persistence** — `localStorage` keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Keys are declared near lines 463–466 and read at startup (lines ~517–540); keep new persisted settings under the `skyfighter.` prefix.
- **Difficulty** — `DIFFICULTIES` (easy / normal / hard) sets spawn rates, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
- **Stages** — `THEMES` (6 background palettes, cycled per stage), `KILLS_PER_STAGE` (18), `ENEMY_TYPES` (grunt, interceptor, tank; `minStage` gates availability), and `BOSSES` (6 bosses, each with its own colors and `hpMul`).
- **Audio** — Web Audio API synthesized sound (`ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object). No audio files. Audio is created lazily on the first user gesture.
- **Input** — keyboard (`keys` map keyed by `e.code`), a touch joystick and FIRE/BOMB buttons (`touchControls`), and mouse/pointer aiming on the canvas (`canvasPointFromEvent`).
- **Game state** — module-level `let` variables (score, lives, bombs, stage, chain/multiplier, etc.) reset by `resetGame()`.
- **Update/draw** — `update(dt)` advances simulation; `draw()` renders everything to the canvas; `loop(now)` drives both via `requestAnimationFrame`. `dt` is in seconds.
- **Bosses** — `BOSS_PATTERNS` is an array of attack functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`). `updateBoss` runs the active pattern; `killBoss` handles defeat.
- **Performance** — `trackPerf(dt)` averages the last 45 frame times and steps `quality` (0 full, 1 medium, 2 low) down or up when the average crosses ~40 / ~56 FPS. Respect this when adding effects.
- **Accessibility** — `reducedMotion` (defaults from `prefers-reduced-motion`) gates screen shake, flashes, and pulses. `announce()` writes to the `#srLive` aria-live region. Any new motion effect should check `reducedMotion`.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Preserve the existing section comments and 2-space indentation.
- Use `const`/`let`, arrow functions for callbacks, and the existing naming style (camelCase functions, UPPER_SNAKE constants).
- Do not introduce frameworks, bundlers, or npm dependencies.
- Power-ups are `life`, `bomb`, and `weapon` (see `applyPowerUp`). Weapon upgrades affect `fireWeapon`.
- When tuning balance, change the values in `DIFFICULTIES`, `ENEMY_TYPES`, or `BOSSES` rather than hard-coding numbers in the update loop.
- Keep the layout working on phone-sized screens; the HUD and touch controls are sized with `clamp()` and `env(safe-area-inset-*)`.

## Git workflow

- Develop on the branch the session specifies (currently `claude/gifted-tesla-7rtbmb`).
- Commit with clear messages describing the gameplay or code change.
- Do not open a pull request unless the user explicitly asks for one.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-file, browser-based Raiden-style vertical shoot-'em-up. There is no build system, package manager, bundler, or test suite.

Files:

- `index.html` (~1,670 lines) — the entire game: markup, CSS (`<style>` block, ~lines 10–360), and JavaScript (`<script>` block, starting ~line 422).
- `README.md` — a placeholder title; it contains no setup instructions.

## Running the game

No install step is needed. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Verification

There are no automated tests, linters, or build steps configured. To verify a change:

1. Open the game in a browser and check the affected behavior manually.
2. For gameplay or layout changes, check both desktop (keyboard) and narrow/touch viewports, since touch controls and resize logic are separate code paths.
3. Check the browser console for errors on load and during play.

## Architecture

All game code lives in one IIFE (`(function () { 'use strict'; ... })();`) so nothing leaks into the global scope. Keep new code inside that IIFE unless there is a clear reason not to.

Major sections in the script, in source order:

- **DOM references** — `document.getElementById` lookups at the top. Add new element references here.
- **Persistence keys** — `STORAGE_*` constants (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`) stored in `localStorage`.
- **Adaptive performance** — `trackPerf` samples frame times and steps a `quality` level (0–2) up or down to shed effects on slow devices.
- **Tuning tables** — these hold most balance values; prefer editing them over hard-coding numbers:
  - `DIFFICULTIES` — spawn rates, enemy speed and shoot chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` — kills needed to advance a stage.
  - `THEMES` — per-stage sky/cloud/sun palettes and names.
  - `ENEMY_TYPES` — `grunt`, `interceptor`, `tank` (size, HP, speed, score, `minStage`).
  - `BOSSES` — boss names, colors, and HP multipliers, one per stage.
  - `BOSS_PATTERNS` — bullet-pattern functions for bosses (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, etc.).
- **Audio** — `ensureAudio()` lazily creates the `AudioContext` on the first user gesture (required by browser autoplay policy). Sound effects are built from `beep()` and `noiseBurst()`; `sfx` and `EXPLODE_SFX` map game events to them. Mute state is persisted.
- **Input** — keyboard (`keys` map, `keydown`/`keyup` on `window`), mouse (`canvasPointFromEvent`), and touch (virtual joystick in `moveZone`, plus `fireBtn` and `bombBtn`).
- **Game state** — `resetGame()` initializes state; `update(dt)` runs game logic; `draw()` renders. `loop(now)` drives both via `requestAnimationFrame`, clamps `dt` to 50 ms, and skips update/draw while paused or not running.
- **Accessibility** — `announce()` writes to an `aria-live` region (`srLive`). Reduced motion is honored from `prefers-reduced-motion` unless the user overrides it with the motion toggle (`body.reduced-motion` class).
- **Scaling** — `resize()` fits a fixed logical canvas (`BASE_W = 480`, `BASE_H = 720`) to the viewport. Game coordinates always use the logical size, not the CSS pixel size.

## Conventions

- Keep the single-file structure. Do not split the game into modules or add a bundler unless the user asks.
- Use `const` and `let`; the file runs in strict mode.
- Name tuning constants in the existing tables rather than adding magic numbers to logic.
- Guard every `localStorage` read and write. Storage can be unavailable (private mode, blocked site data), so the game must still run without it.
- Keep user-visible text and labels consistent with the existing style (short, uppercase boss names, title-case theme names).
- Preserve the existing indentation (2 spaces) and the section comment banners (`// ---------- Name ----------`).

## Gotchas

- **Delta time is capped** at 50 ms in `loop()`. Anything that depends on elapsed time should use `dt`, not wall-clock time, so pausing and tab-switching behave.
- **Audio must start from a user gesture.** Call `ensureAudio()` from input handlers before playing sounds.
- **Pause** is handled by `togglePause()`, which the Start button and the pause button both reach. Make sure any new timed effect respects `paused`.
- **Touch and mouse fire** are wired separately: `fireBtn` has `touchstart`/`touchend` and `mousedown` handlers, and `mouseup` is listened for on the window. Keyboard and canvas input set the same `firing` state. Check firing on both touch and mouse.
- **Boss HP** is scaled by both the difficulty `bossHpMul` and the boss's own `hpMul`.
- Best score is written to `localStorage` only when a run ends (`endGame`, around line 1477).

## Git and workflow

- Develop on the branch named in the task instructions (currently `claude/gifted-tesla-9m71ap`). Do not push to other branches without permission.
- Commit with clear, descriptive messages. Do not open a pull request unless the user asks for one.

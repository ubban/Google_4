# CLAUDE.md

Guidance for Claude Code working in this repository.

## What this repo is

A small static web project. It contains one game, **Sky Fighter**, a Raiden-style vertical shoot-'em-up, in a single self-contained file.

- `index.html` — the entire game: markup, CSS (`<style>` block), and JavaScript (one inline `<script>` IIFE). About 1,670 lines.
- `README.md` — a stub title. Nothing else.

There is no build step, package manifest, bundler, linter, or test suite. Do not add one unless asked.

## Running it

Open `index.html` in a browser. No server is needed. For a quick local server:

```bash
python3 -m http.server 8000
```

There are no automated tests. Verify changes by playing the game in a browser, and check both desktop (keyboard/mouse) and touch layouts when touching input or layout code.

## Code layout inside `index.html`

The script is one `(function () { 'use strict'; ... })();` block. Keep new code inside it. It is organized in these sections, roughly in file order:

1. **DOM references** — `document.getElementById` lookups at the top.
2. **Storage keys** — `STORAGE_*` constants for `localStorage` (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`).
3. **Adaptive performance** — `trackPerf` samples frame time and steps a `quality` level (0–2) up or down.
4. **Config tables** — `DIFFICULTIES`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`.
5. **Audio** — a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, the `sfx` object). Sound is synthesized, with no audio files.
6. **HUD and effects** — `renderHearts`, `renderBombs`, banners, screen shake, flash.
7. **Settings** — mute, difficulty, and reduced-motion toggles, each persisted to `localStorage`.
8. **Resize** — `resize()` keeps a fixed 480×720 logical canvas and scales it to fit the window (capped at 1.6×).
9. **Input** — keyboard (`keys` map keyed by `e.code`), mouse, a virtual joystick and buttons for touch.
10. **Game state** — `resetGame`, `update(dt)`, spawning, collisions, power-ups, bombs, weapons, and boss attack patterns (`BOSS_PATTERNS`).
11. **Rendering** — `draw()` and its helpers (`drawPlane`, `drawBossShip`, `drawClouds`).
12. **Main loop** — `loop(now)` clamps `dt` to 50 ms, runs `update` and `draw` only while running and not paused, then calls `requestAnimationFrame`.

## Controls (from the in-game hint)

- Mouse: move to steer, click to fire.
- Arrows or WASD: move. Space: fire.
- X or Shift: bomb. P or Esc: pause.
- Touch: drag on the left to fly, tap FIRE, tap BOMB.

## Game rules worth knowing

- Difficulty (`easy` / `normal` / `hard`) sets lives, bombs, spawn rate, enemy fire chance, and boss HP multiplier.
- Each stage ends after `KILLS_PER_STAGE` (18) kills. Then a boss spawns after a short delay. Bosses and themes are indexed with `(stage - 1) % length`, so they cycle after six stages.
- Enemy types are `grunt`, `interceptor` (weaving), and `tank` (4 HP, appears from stage 2).
- The best score persists in `localStorage`. Wrap storage access in a try/catch if you add new calls, because storage can be blocked in private windows.

## Conventions

- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and short section comments in the `// ---------- Name ----------` form.
- Keep the file self-contained. Do not add external scripts, fonts, or stylesheets without a clear reason.
- Keep accessibility hooks intact. Announcements go through `announce()`, which writes to the `srLive` live region. The reduced-motion setting (`reducedMotion`, which defaults to the OS preference) skips the hit and bomb flashes and dampens screen shake (`screenShake`). Any new visual effect should respect it.
- Game-balance numbers live in the config tables near the top of the script. Prefer editing those over hard-coding new values in logic.
- Use `dt` (seconds) for any time-based movement so the game runs the same at different frame rates.

## Git

- Work on the branch you were assigned. Do not push to `main` unless asked.
- Write clear, descriptive commit messages.

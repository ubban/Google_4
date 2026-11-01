# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-em-up built as a single self-contained HTML5 canvas page.

| File        | Purpose                                                              |
|-------------|----------------------------------------------------------------------|
| `index.html` | The entire game: markup, CSS (`<style>`), and JavaScript (`<script>`). ~1,670 lines. |
| `README.md`  | Placeholder. Contains only the title and a stray timestamp line.    |

There is no build step, no package manager, no bundler, and no test suite. There is no
`package.json`, so there are no dependencies to install.

## Running and verifying

- **Run:** open `index.html` directly in a browser, or serve the directory with any static
  server (e.g. `python3 -m http.server` and visit `http://localhost:8000`).
- **Verify changes:** there are no automated tests or linters configured. Verify game
  changes by loading the page in a browser and playing through the affected behavior.
  Check both desktop (keyboard/mouse) and mobile-sized viewports (touch controls) when
  touching input, layout, or HUD code.
- Do not introduce a build tool, framework, or dependency unless the user asks for it.
  The project's deliberate design is "one file, zero dependencies."

## Architecture of `index.html`

The file is organized top to bottom as:

1. **`<head>` / `<style>`**: all CSS inline. Includes the pill-style HUD, glassmorphic
   overlay card, combo/stage banners, screen-shake and hit-flash effects, and the
   `.reduced-motion` class that disables animation.
2. **`<body>`**: a single `<canvas id="gameCanvas">` plus HUD elements, the start/pause
   overlay, touch controls, and a hidden `aria-live` region (`#srLive`) for screen readers.
3. **`<script>`**: one IIFE (`(function () { 'use strict'; ... })();`). Nothing is exposed
   on `window`. Keep new code inside this IIFE.

### Key sections inside the script

| Area | Notes |
|------|-------|
| Config constants | `DIFFICULTIES`, `THEMES` (six stage backdrops), `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`, `BASE_W/BASE_H` (480×720 logical resolution). |
| Persistence | `localStorage` keys prefixed `skyfighter.*`: best score, mute, difficulty, reduced motion. All reads are unguarded, so keep them simple and never store anything large. |
| Audio | Web Audio API synthesis only (`beep`, `noiseBurst`). No audio files. `ensureAudio()` must run from a user gesture. |
| Input | Keyboard (`keys[e.code]`), mouse (steer + click-to-fire), and touch joystick plus FIRE/BOMB buttons. |
| Adaptive quality | `trackPerf` samples frame time and steps `quality` 0–2 to reduce effects on slow devices. |
| Entities | Player, enemies (grunt / interceptor / tank), bosses with multi-pattern attacks (`BOSS_PATTERNS`), bullets, power-ups (`life`, `bomb`, `weapon`), explosions. |
| Scoring | Kill-chain multiplier (`addScore`, `breakChain`), best score, per-stage progression. |
| Update / draw | `update(dt)` advances simulation; `draw()` renders. `loop(now)` drives both via `requestAnimationFrame`, clamps `dt` to 0.05 s, and skips update/draw while paused or not running. |

### Game-state flags

Global state lives as `let` variables inside the IIFE: `running`, `paused`, `firing`,
`quality`, `reducedMotion`, `muted`, `difficulty`, `bestScore`, and the per-run arrays and
counters that `resetGame()` initializes. When adding state, initialize it in `resetGame()`
if it should reset per run, and keep it out of `localStorage` unless it must persist.

## Conventions

- **Single file.** Add new CSS, markup, and JS to `index.html` unless the user explicitly
  requests splitting it up.
- **Match existing style:** 2-space indentation, single quotes in JS, `const` by default,
  short helper functions with descriptive names, and `// ----------` section headers.
- **Keep the IIFE closed.** Do not add globals or top-level `var`s.
- **Accessibility and motion:** respect `reducedMotion`. Any new screen shake, flash, or
  animation should be gated on it. Announce important state changes through `announce()`.
- **Mobile:** keep touch targets usable and preserve the `touch-action` / `passive: false`
  handling on touch listeners so the page does not scroll or zoom during play.
- **Performance:** the game loop runs every frame. Avoid allocating large arrays or
  objects per frame in `update`/`draw`, and respect `quality` when adding visual effects.

## Git workflow

- The default working branch for this task is `claude/gifted-tesla-wtmm7z`. Commit and
  push there; do not push to other branches without explicit permission.
- Use clear, descriptive commit messages that explain *why* a change was made.
- Do not open a pull request unless the user explicitly asks for one.

## Things to avoid

- Adding a build system, package manager, or external CDN dependency without being asked.
- Replacing the Web Audio synthesis with audio files, or adding network requests at runtime.
- Writing to `localStorage` in tight loops or storing per-frame data there.
- Changing the logical resolution (`BASE_W` / `BASE_H`) without checking how `resize()`
  scales the canvas on both desktop and mobile.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page.

| Path | Purpose |
| --- | --- |
| `index.html` | The entire game: inline CSS, HTML markup, and one inline `<script>` (~1,700 lines). |
| `README.md` | Placeholder. Contains only the repo title and a stray timestamp. |

There is no build system, package manager, test suite, linter, or backend. Nothing needs to be installed.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser, on both desktop and a phone-sized viewport.
- For headless checks, Playwright with the pre-installed Chromium works; see the environment notes for the browser path.

## Architecture of `index.html`

- **Single IIFE.** All game code sits in one `(function () { 'use strict'; ... })()` block. Nothing is exported or attached to `window`, so keep new helpers inside that block.
- **Rendering.** One `<canvas id="gameCanvas">` drawn with the 2D context. The main loop is `loop(now)` driven by `requestAnimationFrame`. `dt` is clamped to 0.05 s, so long frame gaps (tab switches, debugger pauses) do not cause large jumps.
- **Update/draw split.** `update(dt)` advances game state and `draw()` renders it. Keep new per-frame logic in `update` and new drawing in `draw` or the `draw*` helpers.
- **HUD and overlay.** The HUD (score, best, multiplier, weapon, hearts, bombs, boss bar, stage banner, combo pop) and the start/game-over overlay are DOM elements updated from JS, not drawn on the canvas. Helpers such as `renderHearts`, `renderBombs`, and `showStageBanner` own those updates.
- **Difficulty table.** `DIFFICULTIES` holds `easy`, `normal` (default), and `hard`. Each entry sets spawn timing, enemy speed, enemy fire chance, starting lives, bombs, and boss HP multiplier. Tune balance there rather than in scattered literals.
- **Stages.** `KILLS_PER_STAGE` (18) kills advance to a boss fight. Bosses use pattern helpers such as `fireRadial`, `fireAimedSpread`, `fireCross`, `fireHomingSeeded`, and `fireDoubleRing`, dispatched from `updateBoss`.
- **Weapons and power-ups.** Two weapons, `vulcan` and `laser`, each with power levels. Power-ups are applied in `applyPowerUp`.
- **Persistence.** `localStorage` keys are prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`, `reducedMotion`. Every access must be wrapped so the game still works when storage is unavailable (private mode, blocked site data).
- **Audio.** Procedural Web Audio (`beep`, `noiseBurst`). Audio is created lazily by `ensureAudio()`, which must run from a user gesture because browsers block autoplay.
- **Input.** Keyboard (arrows/WASD, Space to fire, X or Shift for bomb, P or Esc to pause), mouse (move to steer, click to fire), and touch (a joystick zone, FIRE, and BOMB buttons). Touch and mouse handlers use `preventDefault` where needed, and `touch-action: none` is set on the page.
- **Accessibility.** There is a `srLive` live region for screen-reader announcements (`announce`), a reduced-motion toggle that disables screen shake and similar effects, and a mute toggle.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. The single-file, zero-dependency design is intentional, so do not add frameworks, bundlers, or CDN dependencies without asking.
- Match the existing style: 2-space indentation, `const`/`let`, arrow functions for callbacks, and camelCase names. Constants are UPPER_SNAKE_CASE.
- Keep the visual theme consistent: the night-sky and blue palette in the CSS, and the `#0b1d3a` theme color.
- The page must work at phone width with no horizontal scroll. Test touch layouts when changing the HUD or controls.
- Do not introduce `innerHTML` with untrusted data. The current code uses `textContent` for dynamic text.

## Git workflow

- Develop on the branch named in the session instructions (for this environment, `claude/gifted-tesla-hh324g`). Do not push to other branches without explicit permission.
- Do not open a pull request unless the user asks for one.
- Commit messages in this repo use a short imperative subject line, optionally followed by a bulleted body that explains the change.

## Keeping this file current

Update this CLAUDE.md when the structure changes: new files, a split of `index.html`, a build step, tests, or new storage keys or difficulty settings. Keep it factual and short. Do not record secrets, personal details, or environment-specific paths.

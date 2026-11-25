# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small static web project. The only app is **Sky Fighter**, a Raiden-style vertical shoot-'em-up, contained entirely in `index.html` (inline `<style>` and a single inline `<script>` IIFE, ~1,670 lines). `README.md` is just a title line.

Files:

- `index.html` — the whole game: markup, CSS, and JavaScript.
- `README.md` — minimal placeholder.

There is no build system, package manager, bundler, test suite, linter, or CI config. Do not invent one; if you add tooling, say so in the change.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`). No install step.

## Architecture of `index.html`

- **Markup/CSS**: a `#container` wraps a `<canvas id="gameCanvas">` plus overlay, HUD (`#hud`, score/best/mult/weapon/hearts/bombs), touch controls (`#touchControls`, joystick, fire and bomb buttons), boss bar, and an `aria-live` region (`#srLive`). Colors and layout are inline; the page is `overflow: hidden` and `touch-action: none`.
- **Script**: everything lives inside one `(function () { 'use strict'; ... })();` closure. Nothing is exported to `window`; keep it that way unless there is a reason.
- **Game constants** (top of script): `DIFFICULTIES` (spawn rates, enemy speed/fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (six sky palettes, cycled per stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), `BOSSES` (six bosses, cycled), `BASE_W`/`BASE_H` = 480×720 logical resolution.
- **Main loop**: `loop(now)` clamps `dt` to 50 ms, calls `trackPerf` (adaptive `quality` 0–2), then `update(dt)` and `draw()` while `running && !paused`. Logic and rendering are separate functions; keep it that way.
- **Weapons**: `fireWeapon()` handles `vulcan` (spread grows with `player.level`) and `laser`. Power-ups (`life`, `bomb`, `weapon`) are spawned in `maybeSpawnPowerUp` and applied in `applyPowerUp`.
- **Bosses**: `BOSS_PATTERNS` holds the bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring); `updateBoss` and `killBoss` drive them.
- **Audio**: a hand-rolled Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, `sfx`). It is created lazily on first user gesture; do not start audio on load.
- **Persistence**: `localStorage` keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Any new key should use the `skyfighter.` prefix.
- **Accessibility**: respects `prefers-reduced-motion` (user-toggleable via `motionBtn`), announces state via `announce()` into `#srLive`, and provides keyboard, mouse, and touch input.

## Controls (for reference when changing input code)

- Move: arrow keys or WASD (tracked in `keys[e.code]`), mouse over canvas, or touch joystick.
- Fire: hold Space or the fire button.
- Bomb: `X` or Shift, or the bomb button.
- Pause: `P` or `Escape`.

Input handlers are registered near lines 763–861 and at the bottom of the script.

## Conventions

- Plain ES2015+ JavaScript, `const`/`let`, arrow functions, 2-space indentation, single quotes, semicolons. Match the surrounding code.
- Keep the file self-contained: no external scripts, fonts, or CDN links. The only external-looking reference is the inline SVG favicon as a data URI.
- Keep the `<meta name="viewport">` settings and touch-related CSS intact unless the change is specifically about mobile layout. The game must remain playable at phone widths with no horizontal scroll.
- When tuning gameplay, change the `DIFFICULTIES`, `ENEMY_TYPES`, or `BOSSES` tables rather than scattering magic numbers through the update logic.

## Verifying changes

No automated tests exist. For any gameplay or UI change:

1. Open `index.html` in a browser (or use the Playwright setup if one is present in the environment) and play through at least one stage.
2. Check the console for errors on load and during play.
3. Exercise the changed path on desktop (keyboard) and, if layout or input changed, at a phone-sized viewport.
4. Say in your final message exactly what you tested and what you could not test.

## Git

- Default development branch for automated work in this environment: `claude/gifted-tesla-lx2nl9`.
- Commit messages: short imperative summary; end with the attribution lines the session specifies.
- Do not open pull requests unless explicitly asked.

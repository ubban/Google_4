# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The whole project is one self-contained file:

- `index.html` — markup, CSS (`<style>`), and game logic (`<script>`), all inline.
- `README.md` — placeholder title only.

There is no build step, no package manager, no bundler, no framework, and no test suite. To run the game, open `index.html` in a browser (or serve the directory with any static file server).

## Repository layout

```
index.html   # The entire game
README.md    # Title placeholder
CLAUDE.md    # This file
```

## Code structure of `index.html`

- **Markup** (~lines 1–365): `#container` holds the `<canvas id="gameCanvas">`, a HUD (`#hud`: score, best, multiplier, weapon, hearts, bombs), boss bar (`#bossBarWrap`), combo/stage banners, top-bar buttons (mute, reduced motion, pause), touch controls (`#touchControls`: joystick zone, FIRE, BOMB), and the start/pause overlay (`#overlay`) with difficulty buttons.
- **Styles** (`<style>`): dark color scheme. Honor the `body.reduced-motion` class, which disables shake, flash, and pulse animations. Keep any new animation covered by that rule.
- **Script** (`<script>`, one IIFE with `'use strict'`): everything is local to the closure, so nothing is exposed on `window`.

Key sections inside the script, in order:

| Area | Notes |
| --- | --- |
| Sizing (`BASE_W = 480`, `BASE_H = 720`, `resize()`) | Fixed logical resolution, scaled to the viewport (max 1.6×). Game coordinates always use the 480×720 space. |
| Adaptive performance (`trackPerf`, `quality`) | Watches a 45-frame rolling average and steps `quality` 0–2 down or up. |
| Audio (`ensureAudio`, `beep`, `noiseBurst`, `sfx`) | Web Audio synthesis only; no audio files. Respects the mute toggle. |
| `DIFFICULTIES` | Per-difficulty spawn rates, enemy speed, shot chance, lives, bombs, and boss HP multiplier. |
| `KILLS_PER_STAGE = 18` | Kills needed before a boss spawns. |
| `THEMES` | Six sky palettes, cycled by stage (`(stage - 1) % THEMES.length`). |
| `ENEMY_TYPES` | `grunt`, `interceptor` (weaving), `tank` (4 HP, unlocks at stage 2). Gated by `minStage`. |
| `BOSSES` | Six boss designs, cycled by stage, each with an `hpMul`. HP is `70 + stage * 35`, scaled by difficulty and design. |
| `BOSS_PATTERNS` | Attack patterns per boss design (radial, aimed spread, columns, spiral, cross, homing, double ring). |
| `update(dt)` / `draw()` | Main simulation and render, called from `loop(now)` via `requestAnimationFrame`. `dt` is clamped to 50 ms. |
| `applyPowerUp(type)` | Power-ups: `life`, `bomb`, and `weapon`. Weapon upgrades go level 1 → 3, then swap between `vulcan` and `laser`. |
| Persistence | `localStorage` keys prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`, `reducedMotion`. Every access should tolerate storage being unavailable. |

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a restructure. Do not add a build system or dependencies without a clear request.
- **Match the existing style.** Use `const`/`let`, arrow functions for callbacks, 2-space indentation, single quotes in JS, and short section banners in the form `// ---------- Name ----------`.
- **Keep the logical resolution.** Gameplay positions, sizes, and speeds are in 480×720 units. Do not switch to CSS pixels for game logic.
- **Time-based movement.** Multiply per-frame motion by `dt` so behavior stays consistent across frame rates.
- **Accessibility.** Keep the `#srLive` announcements (`announce()`) in sync with important state changes, such as stage changes and game over. Respect reduced motion.
- **Touch and keyboard parity.** Any new control must work with the keyboard/mouse, the touch controls, or both, as the existing code does.

## Controls (for reference when editing input)

- Mouse: move to steer, click to fire.
- Keyboard: Arrows/WASD move, Space fires, X or Shift uses a bomb, P or Esc pauses.
- Touch: drag in the left zone to fly, tap FIRE, tap BOMB.

## Verifying changes

There are no automated tests or linters. To verify a change:

1. Open `index.html` in a browser, or serve the directory locally.
2. Start a game on each difficulty and confirm the HUD, weapons, bombs, and boss transitions behave correctly.
3. Check the browser console for errors.
4. Check at least one narrow (phone-sized) viewport and one desktop viewport.

If you add a test harness or tooling later, document the commands here.

## Git

- Develop on the branch specified by the session. Do not push to other branches without permission.
- Write clear, descriptive commit messages.

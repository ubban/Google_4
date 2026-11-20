# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up played in the browser on a `<canvas>`.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: HTML, CSS (inline `<style>`), and JavaScript (inline `<script>`), about 1,670 lines. |
| `README.md` | Minimal placeholder (title only). |

There is no build step, no package manager, no test suite, and no dependencies. Do not add a framework or bundler unless the user asks for one.

## Running the game

Open `index.html` directly in a browser, or serve the folder with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Architecture of `index.html`

- **Single IIFE.** All game code lives in `(function () { 'use strict'; ... })()`. Nothing is exposed on `window`; keep it that way.
- **DOM layer.** HUD, overlays, touch controls, and the boss bar are plain DOM elements (ids such as `#score`, `#hearts`, `#bombs`, `#overlay`, `#bossBarFill`) that are updated from JS. The canvas (`#gameCanvas`) draws only the playfield.
- **Main loop.** `loop(now)` is driven by `requestAnimationFrame`. `dt` is clamped to 0.05s. When the game is running and not paused, it calls `update(dt)` and then `draw()`.
- **Update / draw split.** `update(dt)` handles movement, spawning, collisions, weapons, and boss logic. `draw()` renders the frame, with helpers such as `drawPlane`, `drawBossShip`, and `drawClouds`.
- **Difficulty.** `DIFFICULTIES` (a config object keyed by name) sets spawn timing and related values. `setDifficulty(name)` persists the choice. Spawn rate ramps every 300 frames.
- **Boss patterns.** Boss attacks are composed from small helpers: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`. Add new patterns as helpers of the same shape rather than inlining them in `updateBoss`.
- **Weapons and power-ups.** `fireWeapon()` fires the current weapon; `applyPowerUp(type)` upgrades it. `weaponLabel()` feeds the HUD.
- **Audio.** Sound effects are synthesized with the Web Audio API (`beep`, `noiseBurst`) and only start after `ensureAudio()` runs on a user gesture.
- **Accessibility.** There is a reduced-motion toggle (screen shake, flash, and pulse), an `aria-live` region (`#srLive`, fed by `announce()`), and labelled buttons. Keep these working when adding effects: gate new screen shake or flashes behind the reduced-motion setting.
- **Input.** Keyboard (arrows/WASD to move, Space to fire, X or Shift for a bomb, P or Esc to pause), mouse (move to steer, click to fire), and touch (a virtual joystick on the left, FIRE and BOMB buttons on the right).

## Persistence

The game stores its state in `localStorage` under keys prefixed `skyfighter.`:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Any change to these keys should keep existing saved values readable. Reads and writes can throw in private browsing or with blocked storage, so wrap new storage access in `try/catch` if you add it.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Existing code uses 2-space indentation, single quotes in JS, and `const`/`let` (no `var`).
- Prefer small, named helper functions over long inline blocks, matching the existing style.
- Use the existing `DIFFICULTIES`, `STORAGE_*`, and DOM-reference constants at the top of the script instead of hard-coding new magic values or selectors.
- Do not introduce external CDN scripts, fonts, or network calls. The game is self-contained.
- Keep the layout working on phones (touch controls are shown on touch devices) as well as desktop.

## Verifying changes

There are no automated tests or linters configured. To check a change:

1. Open `index.html` in a browser (or serve it locally) and check the browser console for errors.
2. Start a game, then exercise the changed behavior: movement, firing, bombs, power-ups, boss phases, pause, mute, reduced-motion, and game over.
3. For layout changes, check at both desktop and phone widths.

If you cannot run a browser in the environment, say so explicitly and verify by reading the code instead.

## Git workflow

- Default working branch for this session: `claude/gifted-tesla-9xkudd`. Commit and push there with `git push -u origin <branch>`.
- Do not open a pull request unless the user asks for one.
- Commit messages should be short and imperative, describing the gameplay or code change (for example, `Add spiral boss pattern`).

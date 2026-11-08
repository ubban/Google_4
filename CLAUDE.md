# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only game is **Sky Fighter**, a
Raiden-style vertical shoot-em-up that runs as a single self-contained HTML5
canvas page.

| File        | Purpose                                                        |
|-------------|----------------------------------------------------------------|
| `index.html`| The entire game: inline CSS (`<style>`), markup, and one IIFE-wrapped `<script>` |
| `README.md` | Placeholder project readme                                     |

There is no build step, no package manager, no bundler, and no test suite.
Do not add one unless the user asks for it.

## Running the game

Open `index.html` directly in a browser, or serve the folder with any static
server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Code layout inside `index.html`

- **Markup/CSS** (lines ~10–420): HUD pills (score, best, multiplier, weapon,
  hearts, bombs), boss bar, stage banner, combo popup, overlay card, touch
  controls, and the `#vignette` / `#flash` effect layers.
- **Script** (starts ~line 422): one `(function () { 'use strict'; ... })();`
  wrapper. Nothing is exported to `window`; keep it that way.
  - `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`:
    tuning tables. Change balance here first.
  - `BOSS_PATTERNS` and the `fire*` helpers: boss bullet patterns.
  - `update(dt)`: game logic per frame. `draw()`: all rendering. `loop(now)`:
    the `requestAnimationFrame` driver, with `dt` clamped to 0.05 s.
  - Audio is synthesized with WebAudio (`beep`, `noiseBurst`); there are no
    audio files. The `AudioContext` is created lazily in `ensureAudio()`.
  - Input: keyboard (`keys[e.code]`), pointer/touch joystick and buttons.

### Persisted state

`localStorage` keys, all prefixed `skyfighter.`:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Reads and writes to `localStorage` are not wrapped in try/catch today. If you
touch them, consider guarding against storage being unavailable (private
windows, blocked site data).

## Controls (for reference when testing)

- Move: mouse (steer by pointer), arrow keys or WASD; touch joystick on mobile.
- Fire: hold Space, click/hold the mouse, or the on-screen FIRE button.
- Bomb: `X` or Shift (or the on-screen bomb button).
- Pause: `P` or Escape.

## Conventions

- Keep the game as a single HTML file unless the user asks to split it.
- Use plain ES2017-era JavaScript (`const`/`let`, arrow functions) to match
  the existing code. Do not introduce frameworks or CDN dependencies.
- Keep gameplay constants in the existing tables rather than hard-coding
  them in logic.
- Respect the reduced-motion setting. `screenShake()` already scales magnitude
  and duration down when `reducedMotion` is on; new shake/flash effects should
  follow the same pattern.
- Preserve the accessibility hooks already present (`#srLive` live region,
  `announce()` for screen-reader messages).

## Verifying changes

There are no automated tests. For gameplay or rendering changes:

1. Load `index.html` in a browser and start a run.
2. Exercise the changed path (for example, a boss pattern, a power-up, or a
   difficulty level), and check the pause, game-over, and restart flows.
3. Check the mobile layout at phone width if the HUD or controls changed.

## Git

- Default development happens on the branch named in the session
  instructions. Do not push to other branches without permission.
- Commit messages should be short imperative summaries, with details in the
  body when the change is non-trivial.

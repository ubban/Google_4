# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas game.

- Everything lives in one file: `index.html` (~1,670 lines). CSS is in a `<style>` block (starts around line 10), and all game logic is in one IIFE inside a `<script>` block (starts around line 422).
- `README.md` is a placeholder and has no run instructions.
- There is no build step, no package manager, no bundler, and no test suite. There is no `package.json`.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000
```

Then visit `http://localhost:8000/`.

## Verifying changes

There are no automated tests or linters configured. For changes to `index.html`:

1. Load the page in a browser and check the console for errors.
2. Play through at least one start, fight, and game-over cycle.
3. For UI changes, check both desktop and narrow (phone-width) viewports. The HUD and overlay are responsive, and touch controls only appear on touch devices.

## Code structure (inside `index.html`)

The script is organized as a single closure. Key pieces, roughly in file order:

- **DOM references**: `const xxxEl = document.getElementById(...)` at the top of the IIFE.
- **Persistence keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`. These are `localStorage` keys prefixed with `skyfighter.`. Changing a key name will reset saved player data for existing users, so treat it as a breaking change.
- **Performance scaling**: `trackPerf` adjusts a `quality` level (0–2) based on frame times.
- **Tuning tables**:
  - `DIFFICULTIES` (spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier)
  - `KILLS_PER_STAGE` (kills needed to trigger a boss)
  - `THEMES` (per-stage sky palettes, cycled by `(stage - 1) % THEMES.length`)
  - `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`, with `minStage` gating)
  - `BOSSES` (six bosses, cycled per stage, each with an `hpMul`)
- **Audio**: Web Audio API helpers `ensureAudio`, `beep`, and `noiseBurst`. Audio is created lazily on the first user gesture.
- **Input**: keyboard handlers near line 769, touch joystick and buttons, and a `keys` map.
- **Game loop**: `update(dt)` for logic, `draw()` for rendering, and `loop(now)` as the `requestAnimationFrame` driver.
- **Boss patterns**: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, driven by `updateBoss`.

## Controls (for reference when testing)

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrow keys or WASD | Joystick (left) |
| Fire | Space (hold) | Fire button |
| Bomb | X or Shift | Bomb button |
| Pause | P or Escape | Pause button |

Power-ups are `life`, `bomb`, and `weapon`. Weapon level and smart bombs are both limited.

## Conventions

- Keep the game self-contained in `index.html`. Do not add external script or stylesheet dependencies unless the user asks for them.
- Keep tuning values in the existing constant tables rather than hard-coding them inside update logic.
- Honor `reducedMotion`: any new screen shake, flash, or pulse effect should be gated on it.
- Announce important state changes through `announce()` so screen readers get them through the `srLive` region.
- Store any new persisted setting under the `skyfighter.` prefix, following the existing `localStorage` read/write pattern.
- Match the existing style: 2-space indent, single quotes in JS, `const`/`let`, and no transpilation.

## Git workflow

- Develop on the branch specified by the session (currently `claude/gifted-tesla-uc23u0`) and push with `git push -u origin <branch>`.
- Write descriptive commit messages that explain the gameplay or code change.
- Do not open pull requests unless the user asks for one.

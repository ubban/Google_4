# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free front-end project. Its only substantial content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on an HTML5 `<canvas>`.

- `index.html` (~1,670 lines) — the entire game: markup, CSS (`<style>` block), and JavaScript (one `<script>` IIFE). There is no build step, bundler, package manager, or framework.
- `README.md` — a stub title. Keep it minimal unless the user asks for more.

There is no `package.json`, test suite, linter, or CI configuration in the repo.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

Verifying changes: there are no automated tests. After editing the game, load it in a browser and play through the affected path (start, shooting, bombs, power-ups, boss, pause, game over). For visual or gameplay changes, a real browser check is the only verification available.

## Architecture of `index.html`

All game code lives inside a single `(function () { 'use strict'; ... })();` IIFE. Things to know before editing:

- **Config tables** near the top of the script drive most tuning:
  - `DIFFICULTIES` — `easy` / `normal` / `hard`, controlling spawn rate, enemy speed, enemy fire chance, starting `lives`, starting `bombs`, and `bossHpMul`.
  - `KILLS_PER_STAGE` — kills needed to advance a stage (currently 18).
  - `THEMES` — six sky palettes that cycle by stage.
  - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP, appears from stage 2).
  - `BOSSES` — six boss designs (name, colors, `hpMul`).
- **Persistence** uses `localStorage` under keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every read/write should tolerate missing or blocked storage the same way the existing code does.
- **Main loop**: `update(dt)` advances the simulation, `draw()` renders, and `loop(now)` drives both via `requestAnimationFrame`. Keep simulation logic in `update` and rendering in `draw`.
- **Boss fights**: patterns are functions in `BOSS_PATTERNS` (`fireRadial`, `fireAimedSpread`, `fireCross`, etc.), dispatched from `updateBoss`. `killBoss` and `loseLife` handle the transitions.
- **Input**:
  - Keyboard: arrows or WASD to move, Space to fire, X / Shift to bomb, P / Escape to pause.
  - Touch: on-screen joystick (`moveZone`), fire and bomb buttons.
  - Mouse: canvas pointer control.
- **Accessibility**: there is a `reducedMotion` setting and a live region (`srLive`) used by the `announce()` helper for screen readers. Preserve both when adding effects or messages.
- **Audio** is generated with the Web Audio API (`beep`, `noiseBurst`); `ensureAudio()` must run after a user gesture.
- **Canvas sizing**: logical size is `BASE_W × BASE_H` (480 × 720), scaled to fit by `resize()`. Coordinates in game logic use the logical size, not CSS pixels.

## Conventions

- Keep everything in `index.html` unless the user asks to split it. Do not introduce a build system, framework, or dependencies without an explicit request.
- Match the existing style: `const`/`let`, 2-space indentation, single quotes in JS, and small named functions.
- Prefer adding to the config tables over hard-coding new magic numbers in logic.
- When adding a new enemy, boss pattern, power-up, or theme, follow the shape of the existing entries in the corresponding table or function.
- Keep the game playable on phones: any new HUD element or control must work with the touch layout and a narrow viewport.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-1keaum`). Do not push to other branches without permission.
- Commit with clear, descriptive messages. Do not open a pull request unless the user asks for one.

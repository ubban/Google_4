# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small static project. It contains one game, **Sky Fighter**, a Raiden-style vertical shoot-em-up, implemented as a single self-contained HTML5 canvas page.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS (`<style>`, ~lines 10–360), and JavaScript (`<script>`, ~lines 422–1668). |
| `README.md` | Placeholder title only. |

There is no `package.json`, build step, bundler, test suite, or linter. Do not add tooling unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server, for example `python3 -m http.server`. Nothing needs to be installed.

## Code layout inside `index.html`

The script is one IIFE with `'use strict'`. Everything is private to it, and there are no modules or globals to export.

- **DOM references** (top of the script): every element is looked up once by id. Keep the ids in the markup and the `getElementById` calls in sync.
- **Persistence**: `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Reads and writes are not wrapped in try/catch, so keep that in mind if you touch them.
- **Tuning tables**, all plain object or array literals near the top of the script:
  - `DIFFICULTIES`: spawn rates, enemy speed and fire chance, starting lives and bombs, and boss HP multiplier for `easy`, `normal`, and `hard`.
  - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
  - `THEMES`: six sky palettes that cycle by stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), and `tank` (4 HP, from stage 2).
  - `BOSSES`: six bosses, each with colors, an `hpMul`, and a matching entry in `BOSS_PATTERNS`.
- **Bullet patterns**: the `fire*` helpers (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) build enemy bullets. `BOSS_PATTERNS` composes them into three-phase attack loops.
- **Game loop**: `update(dt)` handles simulation, `draw()` handles rendering, and `loop(now)` drives both through `requestAnimationFrame`. `trackPerf` adapts a `quality` level (0–2) when frame time is poor, and the draw code respects it.
- **Power-ups** (`applyPowerUp`): `life`, `bomb`, and `weapon`. The weapon cycles through three levels, then alternates between `vulcan` and `laser`.
- **Audio** is synthesized with the Web Audio API (`beep`, `noiseBurst`, `sfx.*`). `ensureAudio()` must run from a user gesture before any sound plays.
- **Accessibility**: `announce()` writes to the `srLive` live region. `reducedMotion` turns off screen shake and similar effects, and it defaults to the `prefers-reduced-motion` media query.

## Controls (keep the hint text in sync)

The on-screen hint is `#controlsHint`. It must match the handlers in the script.

- Keyboard: arrows or WASD to move, Space to fire, X or Shift to bomb, P or Esc to pause.
- Mouse: move to steer, hold to fire.
- Touch: drag the left zone to fly, tap FIRE or BOMB.

## Conventions

- Match the existing style: two-space indent, single quotes, `const`/`let`, short section comments in the `// ---------- Name ----------` form, and `addEventListener` for all wiring.
- Keep it a single file. Splitting it into modules or adding a framework would change the project's design, so ask first.
- Game balance lives in the tuning tables above. Prefer editing those over hard-coding numbers in logic.
- When you add a boss, theme, enemy, or power-up, update the matching table and any UI text that lists them.

## Verifying changes

There are no automated tests. Verify by loading `index.html` in a browser and checking the affected behavior, such as a new boss pattern, a difficulty change, or the touch layout at phone width. For logic-only changes, a quick Node syntax check is useful:

```bash
node -e "const s=require('fs').readFileSync('index.html','utf8'); new Function(s.split('<script>')[1].split('</script>')[0])"
```

## Git

- Develop on the branch assigned to the session. Do not push to `main` directly.
- Write descriptive commit messages that explain why a change was made.

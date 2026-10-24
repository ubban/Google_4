# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. It is a single-page HTML5 canvas game with no build step, no package manager, and no dependencies.

- `index.html` — the entire game: markup, CSS, and JavaScript in one file (~58 KB).
- `README.md` — minimal project title only.

There is no test suite, linter, or bundler configured.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000
```

## File layout inside `index.html`

| Section | Approx. lines | Contents |
| --- | --- | --- |
| `<head>` | 1–10 | Meta tags, theme color, inline SVG favicon |
| `<style>` | 10–361 | All CSS: HUD pills, hearts/bombs, overlay card, touch controls, `body.reduced-motion` rules |
| `<body>` markup | 362–421 | Canvas, HUD elements, overlay, touch controls, boss bar, `#srLive` live region |
| `<script>` | 422–1670 | The game, wrapped in a single IIFE with `'use strict'` |

Inside the script, the code is grouped by `// ----------` comment headers and by these major blocks:

- **Config tables** — `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, `KILLS_PER_STAGE`.
- **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object. Audio is synthesized with the Web Audio API; no sound files are loaded.
- **Input** — keyboard (`keydown`/`keyup` on `keys[e.code]`), mouse on the canvas, and touch joystick plus fire/bomb buttons.
- **Game state** — `resetGame`, `update(dt)`, `draw()`, and the `loop(now)` requestAnimationFrame driver.
- **Weapons, bombs, power-ups** — `fireWeapon`, `useBomb`, `applyPowerUp`.
- **Bosses** — `spawnBoss`, `updateBoss`, `killBoss`, and the pattern generators (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, and others).
- **Persistence** — `localStorage` keys (see below).
- **Adaptive quality** — `trackPerf` lowers the `quality` level when frame time is high.

## Game rules and tuning

- **Stages:** Kills advance the stage every `KILLS_PER_STAGE` (18). Each stage is themed from `THEMES`, which cycles through six sky palettes.
- **Enemies:** `grunt` (minStage 1), `interceptor` (minStage 1, weaves), and `tank` (minStage 2, 4 HP).
- **Bosses:** Six bosses in `BOSSES`, each with its own `hpMul` and attack patterns from `BOSS_PATTERNS`.
- **Difficulty:** `easy`, `normal` (default), and `hard` change spawn rate, enemy shot chance, starting lives, starting bombs, and boss HP multiplier.
- **Scoring:** A kill-chain multiplier (`addScore`, `breakChain`) raises score for consecutive kills.
- **Weapons:** Vulcan and Laser, each with power levels 1–3.

Balance numbers live in the config tables at the top of the script. Change them there rather than hard-coding values in logic.

## Controls

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrow keys or WASD | Left-side joystick |
| Fire | Hold Space | Fire button |
| Smart bomb | X or Shift | Bomb button |
| Pause | P or Escape | Pause button |
| Mute | Mute button | — |

Mouse movement on the canvas also steers the ship.

## Persistence

The game stores these keys in `localStorage`. Keep the prefix when adding new ones.

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | Best score |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | `'1'` or `'0'`; defaults to the `prefers-reduced-motion` media query when unset |

Reads of `localStorage` are not wrapped in `try/catch` in the current code. If you add storage use, guard it, since private browsing and blocked storage can throw.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not add a build step or npm dependencies without a clear reason.
- **No external resources.** The favicon is an inline data URI and audio is synthesized. Keep it that way so the game works offline.
- **Match local style.** The script uses 2-space indentation, `const`/`let`, arrow functions for callbacks, and short `// ----------` section headers. Follow that.
- **Accessibility.** Screen-reader announcements go through `announce()` and the `#srLive` region. Respect `body.reduced-motion` when adding animation, and add matching CSS rules.
- **Mobile.** The layout must work at phone width. The canvas scales via `resize()` against `BASE_W` × `BASE_H` (480 × 720).
- **Performance.** The main loop runs on `requestAnimationFrame` and clamps `dt` to 0.05 s. The `quality` level from `trackPerf` is the existing hook for scaling effects down on slow devices. Check it when adding costly visuals.

## Verifying changes

There is no automated test setup. To verify a change:

1. Open `index.html` in a browser (or serve it locally) and check the console for errors.
2. Play through at least one full stage and one boss fight on each difficulty you touched.
3. Check pause, mute, difficulty switching, and best-score persistence after a reload.
4. Check the layout at a phone-sized viewport and with reduced motion enabled.

## Git

- Development branch for this repo: `claude/gifted-tesla-lfigs2`.
- Commit messages should be short and describe the gameplay or code change.

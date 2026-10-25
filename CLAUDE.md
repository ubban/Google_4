# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-page HTML5 canvas game, **Sky Fighter** — a Raiden-style vertical shoot-em-up. The entire game lives in `index.html`:

- `<style>` block (~lines 10–361): HUD, overlays, touch controls, reduced-motion styles.
- Markup (~lines 362–421): canvas, HUD, top bar, touch controls, start/pause overlay.
- One inline `<script>` (~line 422 onward), wrapped in an IIFE with `'use strict'`.

`README.md` is a placeholder. There is no `package.json`, no build step, no bundler, no test suite, and no linter configured. Dependencies: none.

## Running and verifying

- Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser: start a run, check firing, bombs, power-ups, stage transitions, boss fights, pause (`P`/`Esc`), game over, and the best-score persistence.
- Check the mobile layout (touch joystick, FIRE/BOMB buttons) and the reduced-motion toggle when touching the HUD or effects code.
- For a quick syntax check of the inline script after edits, extract the `<script>` body and run `node --check` on it.

## Code layout inside the `<script>`

Sections appear in this order, marked with `// ---------- Name ----------` comments. Keep new code in the matching section:

1. **DOM references** — `const xxxEl = document.getElementById(...)` at the top.
2. **Constants** — `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, `BASE_W`/`BASE_H` (480×720 logical resolution).
3. **Persistence and accessibility** — best score, mute, difficulty, and reduced-motion state, all read from `localStorage` with a `skyfighter.*` key prefix.
4. **Audio** — `ensureAudio()`, `beep()`, `noiseBurst()`, and the `sfx` object. Audio is created lazily on the first user gesture (browsers block autoplay).
5. **HUD and effects** — `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`.
6. **Canvas sizing** — `resize()` scales the canvas to the window (capped at 1.6×).
7. **Input** — keyboard (`keys[e.code]`), mouse/pointer, and touch joystick/buttons.
8. **Game state** — `resetGame()`, `update(dt)`, spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), weapons (`fireWeapon`), power-ups (`applyPowerUp`), bomb (`useBomb`), kill-chain score multiplier (`addScore`, `breakChain`).
9. **Boss patterns** — `BOSS_PATTERNS` and the `fire*` helpers, driven by `updateBoss`. `killBoss` and `loseLife` handle transitions.
10. **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
11. **Main loop** — `loop(now)` with a fixed-ish `dt` and adaptive quality (`trackPerf`, `quality` 0–2).

### Game rules worth knowing

- Difficulty (`easy` / `normal` / `hard`) changes spawn rate, enemy speed, enemy shot chance, starting lives, starting bombs, and boss HP multiplier.
- Stages advance every `KILLS_PER_STAGE` (18) kills. Each stage has a theme from `THEMES` (cycled), and a boss from `BOSSES` (cycled).
- Enemy types: `grunt` (stage 1+), `interceptor` (stage 1+, weaving), `tank` (stage 2+, 4 HP).
- Weapons: Vulcan and Laser, each with power levels 1–3, raised by power-ups.
- The kill chain multiplies score; it breaks after a gap without kills (`breakChain`).
- Bombs clear the screen and damage bosses. Players can use keyboard (`X`, `Shift`), touch (`BOMB`), or the bomb button.

## Controls

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrows or `WASD`; mouse steers (pointer position) | Left-side joystick (drag) |
| Fire | Hold `Space`, or hold the mouse button | FIRE button (hold) |
| Bomb | `X` or `Shift` | BOMB button |
| Pause | `P` or `Esc` | ⏸ button |
| Mute | — (the 🔊 button only) | 🔊 button |
| Reduced motion | — (the 🎆 button only) | 🎆 button |

Fire is decided in `update()` as `keys['Space'] || firing`, where `firing` is set by the FIRE button and mouse/pointer press.

## Persistence

All saved state uses `localStorage` under the `skyfighter.` prefix:

- `skyfighter.bestScore` — high score (written at game over).
- `skyfighter.muted` — `'1'` or `'0'`.
- `skyfighter.difficulty` — `easy` / `normal` / `hard`.
- `skyfighter.reducedMotion` — `'1'` or `'0'`. If unset, it defaults to the `prefers-reduced-motion` media query.

Reads and writes currently run without try/catch. If you add new storage, consider that private browsing or blocked storage can make `localStorage` throw.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. Splitting changes the deploy story, and nothing here needs a build.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and short section banner comments (`// ---------- Name ----------`).
- Don't add dependencies, frameworks, or a build step without asking first.
- Keep the game loop allocation-light. Reuse existing arrays and objects where practical; `update()` and `draw()` run every frame.
- Respect the reduced-motion setting for any new screen shake, flash, or pulsing effect. Route new effects through the existing helpers (`screenShake`, `flashHit`) so the check stays in one place.
- Keep the HUD and overlays accessible: use `announce()` (the `#srLive` live region) for important state changes, and keep button `title` attributes set.
- Keep the layout working at phone width. The canvas scales, and touch controls sit below it.

## Git

- Development branch for this task: `claude/gifted-tesla-ikgldf`.
- Commit messages: short imperative summary, then a body explaining the why when it isn't obvious.
- Don't open pull requests unless asked.

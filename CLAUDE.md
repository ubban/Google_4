# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up.

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` (~1,670 lines). There is no build step, no package manager, and no test suite.
- `README.md` — a one-line title only.

To run it, open `index.html` in a browser (or serve the directory with any static server). Nothing needs to be installed.

## Repository layout

```
index.html   # whole game (CSS, HTML, JS)
README.md
CLAUDE.md    # this file
```

Keep the game self-contained in `index.html` unless the user asks for a split. Do not add a bundler, framework, or dependency without an explicit request.

## Code structure inside `index.html`

The script is one `(function () { 'use strict'; ... })();` block (lines ~422–1666) that runs on load. Major sections, in rough file order:

- **DOM refs and constants** (~line 426–466): `canvas`, HUD elements, touch controls, and `STORAGE_*` keys for `localStorage`.
- **Config tables** (~line 485–516):
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rate, enemy speed, shoot chance, lives, bombs, and boss HP multiplier.
  - `KILLS_PER_STAGE` (18) controls stage progression.
  - `THEMES` — six sky palettes, cycled by stage.
  - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP, stage 2+).
  - `BOSSES` — six boss designs, cycled by stage via `(stage - 1) % BOSSES.length`; HP scales with stage and `hpMul`.
- **Audio** (~line 543–680): lazily created `AudioContext` (`ensureAudio()`), synthesized SFX via `beep()` and `noiseBurst()`, and the `sfx` table. Audio must only start after a user gesture.
- **Input** (~line 768–861): keyboard (arrows/WASD move, Space fires, P/Escape pause, X/Shift bomb), touch joystick + fire/bomb buttons, and mouse drag on the canvas.
- **State** (~line 864–911): module-level `let` variables for the run (`player`, `bullets`, `enemies`, `boss`, `score`, `lives`, `stage`, `chain`/`multiplier`, etc.). `resetGame()` reinitializes them.
- **Rendering** (~line 913–1020, `drawPlane`, `drawBossShip`, `draw()` ~line 1520): all drawing is on a 2D canvas; no sprites or image assets.
- **Simulation** (~line 1022–1410): `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `update(dt)` (~line 1148), `useBomb()` (~line 1103), `updateBoss(dt)` (~line 1411).
- **Lifecycle**: `endGame()` (~line 1469), `togglePause()` (~line 1491), and `loop(now)` (~line 1640), which drives `requestAnimationFrame`.

Canvas base resolution is **480 × 720** (`BASE_W`, `BASE_H`); `resize()` scales it to fit the viewport, capped at 1.6×.

## Persisted settings

The game uses `localStorage` with these keys. Keep the names stable, since changing them resets players' saved data.

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | High score |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | `easy` / `normal` / `hard` |
| `skyfighter.reducedMotion` | `'1'` / `'0'`; defaults to the `prefers-reduced-motion` media query when unset |

All reads and writes are direct today. If you add more storage, wrap it in `try/catch`, since storage can throw in private windows or when site data is blocked.

## Working conventions

- **Make minimal, targeted edits.** The file is one large script; prefer changing the specific function or config row rather than restructuring.
- **Preserve the visual style and tuning.** Balance numbers (spawn timings, HP, speeds, scoring) are deliberate. Change them only when the task asks for it, and say so in the summary.
- **Keep it framework-free and offline-capable.** No external scripts or fonts are loaded today; don't add CDN dependencies without a request.
- **Mobile matters.** The layout uses `viewport-fit=cover`, safe-area insets, and touch controls. Check that any UI change still works at phone width and doesn't introduce horizontal scrolling. The page sets `overflow: hidden` on `html, body`.
- **Accessibility.** Keep the `srLive` announcements, the reduced-motion option, and the `aria`/labels on controls working when you touch the related code.
- **Dark theme only.** The page declares `color-scheme: dark`.

## Verifying changes

There are no automated tests or linters configured. To verify a change:

1. Open `index.html` in a browser, or serve the folder locally (for example, `python3 -m http.server` in the repo root).
2. Play through start, a stage transition (18 kills), a boss fight, a bomb, pause/resume, and game over.
3. Check the three difficulties, mute, and reduced-motion toggles, and confirm the high score persists across reloads.
4. For layout changes, test a narrow (phone-sized) viewport as well as desktop.

If you can run a headless browser (Playwright is typically available in this environment), use it to exercise the game and capture a screenshot. Otherwise say explicitly that the change was verified by reading only.

## Git

- Default branch work happens on the branch the session specifies; don't push to other branches without permission.
- Write clear, descriptive commit messages in the imperative mood.

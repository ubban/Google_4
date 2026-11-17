# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser.

- `index.html` — the entire game: inline CSS, inline JavaScript, and a `<canvas>`-based renderer (~1,670 lines). There is no build step, no bundler, and no external dependencies.
- `README.md` — minimal placeholder (title only).

There are no tests, no package manager files, and no CI configuration.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Code layout inside `index.html`

All game logic lives in a single IIFE (`(function () { ... })()`) starting around line 423. Key sections, in file order:

- **Storage keys** (`STORAGE_*`, ~line 463) — `localStorage` keys for best score, mute, difficulty, and reduced-motion.
- **Audio** (`ensureAudio`, `beep`, `noiseBurst`, `EXPLODE_SFX`) — Web Audio synthesized sound effects; no audio files.
- **Tuning tables** — `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES`, `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS` (bullet-pattern functions such as `fireRadial`, `fireCross`, `fireHomingSeeded`).
- **Input** — keyboard listeners (`keydown`/`keyup` on `window`), touch joystick and FIRE/BOMB buttons, and mouse steering/firing on the canvas.
- **Game state** — `resetGame()` initializes state; `update(dt)` runs one simulation step; `applyPowerUp`, `spawnEnemy`, `spawnBoss`, `useBomb`, `fireWeapon`, `loseLife`, `endGame`, `togglePause`.
- **Rendering** — `draw()` and helpers (`drawPlane`, `drawBossShip`, `drawClouds`); `resize()` scales a fixed `BASE_W × BASE_H` (480 × 720) logical canvas to the viewport.
- **Main loop** — `loop(now)` drives `update` and `draw` with a delta-time step.

Preserve this structure when editing. New constants belong in the tuning tables, not inline in logic.

## Controls (for reference when testing)

| Action | Desktop | Mobile |
| --- | --- | --- |
| Move | Arrow keys / WASD, or mouse | Drag left side |
| Fire | Space (hold), or hold mouse button | Tap FIRE |
| Bomb | X or Shift | Tap BOMB |
| Pause | P or Esc | — |

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not introduce a build system, framework, or npm dependencies without being asked.
- **Vanilla JS, ES2015+ syntax** as already used (`const`/`let`, arrow functions, template literals). Match the existing indentation (2 spaces) and the terse style of the surrounding code.
- **No external network requests.** The page is self-contained; keep it that way.
- **Mobile-first layout.** The viewport meta disables zoom and the canvas uses `touch-action: none`. Verify any UI change still works at phone width.
- **Accessibility.** Respect the existing reduced-motion setting and the `announce()` live-region helper when adding visible effects or messages.
- **Persisted state** uses `localStorage` keys prefixed `skyfighter.`. Wrap new storage reads/writes in try/catch, since storage can be unavailable in private browsing.

## Verification

No automated tests exist. To verify a change:

1. Load `index.html` in a browser and start a game.
2. Exercise the path you changed (weapons, bosses, bombs, pause, game over, difficulty switch).
3. Check the browser console for errors.
4. For layout or touch changes, test at a narrow (phone) viewport as well as desktop.

A headless Chromium with Playwright is available in the cloud environment and can be used for scripted smoke tests; do not run `playwright install`.

## Git

- Default work branch for cloud sessions: `claude/gifted-tesla-uve1o2`.
- Commit messages: short imperative summary line, followed by a body when the change needs explanation.
- Do not open pull requests unless explicitly asked.

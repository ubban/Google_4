# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page HTML5 canvas game called **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game lives in one file, `index.html` (~1,670 lines: inline CSS, markup, and one inline `<script>`). `README.md` is only a stub title.

There is no build step, no package manager, no bundler, no framework, and no test suite. Do not introduce any of these unless the user asks.

## Repository layout

| Path         | Purpose                                                        |
|--------------|----------------------------------------------------------------|
| `index.html` | The whole game: styles, HUD/overlay markup, and game script.   |
| `README.md`  | Minimal project readme.                                        |
| `CLAUDE.md`  | This file.                                                     |

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

No install step is needed.

## Code structure inside `index.html`

- **`<style>`** (approx. lines 10–360): CSS for the dark color scheme, pill-style HUD, glassmorphic overlay card, boss bar, stage banner, and low-health vignette. Reduced-motion styling is applied via `body.reduced-motion`.
- **`<body>`** (approx. lines 360–420): `#container` wrapping `<canvas id="gameCanvas">`, the HUD (`#hearts`, bombs, score, boss bar `#bossBarWrap`, `#stageBanner`, `#comboPop`, `#flash`, `#vignette`), the overlay card (`#overlayCard`, `#overlaySub`), touch controls (joystick move zone, fire button, bomb button), and the mute / pause / difficulty controls.
- **`<script>`** (approx. lines 422–1669), roughly in this order:
  1. **Config constants**: `STORAGE_*` localStorage keys (`skyfighter.bestScore`, `.muted`, `.difficulty`, `.reducedMotion`), `DIFFICULTIES` (easy / normal / hard: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18).
  2. **Audio**: `ensureAudio`, `beep`, `noiseBurst`, stereo `panFor`. Audio is synthesized with WebAudio; no audio files.
  3. **UI helpers**: `announce` (screen-reader live region), `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`, `setDifficulty`, `resize`.
  4. **Input**: `keydown` / `keyup` handlers on `keys[e.code]`, touch joystick (`joystickStart/Move/End`), fire and bomb buttons, mouse aiming on the canvas (`canvasPointFromEvent`).
  5. **Game state and lifecycle**: `resetGame`, `update(dt)`, `endGame`, `loseLife`, `togglePause`, `loop(now)` (the `requestAnimationFrame` loop). `trackPerf` monitors frame timing.
  6. **Entities and spawning**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `spawnExplosion`, `rectsOverlap` (AABB collision).
  7. **Weapons and bombs**: `fireWeapon` (Vulcan / Laser, 3 power levels), `useBomb` (clears screen, damages bosses), `addScore` / `breakChain` (kill-chain score multiplier).
  8. **Boss patterns**: `updateBoss`, `killBoss`, and pattern helpers `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`.
  9. **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds` (parallax layers), `draw()`. All drawing is 2D canvas; no sprites or image assets.

Game flow: the title overlay shows → `resetGame` → waves spawn → every `KILLS_PER_STAGE` kills a boss is warned and spawned → on boss kill `stage++` and a stage banner shows → on losing all lives `endGame` saves the best score.

## Conventions

- **Keep it a single file.** New code goes inside the existing `<style>` / `<script>` blocks of `index.html` unless the user asks to split it. Do not add external dependencies or CDN links; the game is intended to work offline from one file.
- **No frameworks or build tooling** without an explicit request.
- **Preserve the existing style**: plain ES2015+ JavaScript, `const`/`let`, arrow functions for callbacks, two-space indentation, single quotes in JS, and the existing naming (`camelCase` functions, `UPPER_SNAKE` constants, `STORAGE_*` prefixes for persisted keys).
- **Frame-based timing**: game logic uses the `dt` passed to `update(dt)`. Do not use wall-clock timers for gameplay.
- **Persist only through the existing `STORAGE_*` keys** and wrap any new localStorage access in try/catch, since storage may be unavailable (private windows, blocked site data).
- **Respect accessibility settings**: the reduced-motion toggle (`body.reduced-motion`) disables screen shake, flashes and pulses; new visual effects should honor it. Keep `announce()` calls for important state changes.
- **Touch and desktop parity**: any new control must work both with keyboard/mouse and with the touch controls. Keep `touch-action: none` and the `passive: false` handlers where `preventDefault` is needed.
- **Mobile layout**: the canvas scales via `resize()`; the page must not scroll and must respect `env(safe-area-inset-*)`.

## Testing and verification

There is no automated test suite. To verify a change:

1. Open `index.html` in a browser (or use the `run` skill / Playwright) and play through at least one full stage, including a boss fight.
2. Check all three difficulty levels if the change touches balance or spawning.
3. Exercise both keyboard (WASD/arrows to move, Space to fire, X or either Shift to bomb, P or Esc to pause) and touch/mouse controls if input code changed.
4. Check the layout at a phone-sized viewport and a desktop viewport.
5. Check the browser console for errors before reporting success.

Describe what was and was not verified in the final report.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-kd0kk2`). Do not push to other branches without permission.
- Commit only when asked or when the task calls for it, and keep commit messages descriptive, matching the existing history style (imperative summary line, optional bullet detail).
- Do not create pull requests unless the user explicitly asks for one.
- Do not commit secrets or local settings files such as `.claude/settings.local.json`.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is one self-contained file:

- `index.html` (~1,670 lines) — inline `<style>`, markup, and a single IIFE `<script>` that holds all game logic and rendering on a `<canvas id="gameCanvas">`.
- `README.md` — a placeholder title only.

There is no build step, package manager, bundler, test suite, linter, or framework. Do not introduce one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Code layout inside `index.html`

The script is organized as commented sections. Keep new code in the matching section:

| Section (comment header) | What lives there |
| --- | --- |
| DOM refs (top of IIFE) | `const xEl = document.getElementById(...)` lookups |
| `STORAGE_*` constants | `localStorage` keys, all prefixed `skyfighter.` |
| Adaptive performance | `trackPerf` lowers `quality` (0 full / 1 medium / 2 low) when frame time drifts |
| Config tables | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS` |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `sfx.*` — Web Audio, no asset files |
| HUD helpers | `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake` |
| Input | keyboard `keys[]` map, touch joystick + fire/bomb buttons, mouse aim on canvas |
| Game state | `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `applyPowerUp`, `useBomb`, `fireWeapon` |
| Boss attacks | `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, `updateBoss`, `killBoss` |
| Rendering | `drawPlane`, `drawBossShip`, `drawClouds`, `draw()` |
| Loop | `loop(now)` drives `update` + `draw` via `requestAnimationFrame`; `dt` is clamped to 50 ms |

### Key game data

- **Difficulty** (`DIFFICULTIES`): `easy` / `normal` / `hard` set spawn rate, enemy speed, enemy shot chance, starting lives, starting bombs, and boss HP multiplier. The selection persists in `localStorage`.
- **Stages**: a stage advances every `KILLS_PER_STAGE` (18) kills, cycling through `THEMES` for the background palette. Each stage ends in a boss from `BOSSES`.
- **Enemies** (`ENEMY_TYPES`): `grunt`, `interceptor` (weaving), `tank` (4 HP, appears from stage 2).
- **Power-ups** (`applyPowerUp`): `life`, `bomb`, and `weapon` (toggles `vulcan` ↔ `laser`).
- **Scoring**: base points per enemy, multiplied by a combo chain that resets via `breakChain()`. Best score is stored under `skyfighter.bestScore`.

### Controls

- Keyboard: arrow keys / WASD to move (via `keys[e.code]`), `Space` to fire, `X` or `Shift` for smart bomb, `P` or `Esc` to pause.
- Touch: joystick in `#moveZone`, `#fireBtn`, `#bombBtn` (shown when `ontouchstart` or `maxTouchPoints` is detected).
- Mouse: move to steer, click (or hold) to fire on the canvas.

## Conventions

- **Single file, no dependencies.** Keep all JS, CSS, and markup in `index.html`. Do not add external CDN scripts, fonts, or assets without a clear reason.
- **Match the existing style:** 2-space indentation, `const` by default, arrow functions for callbacks, `'single quotes'` for strings, and short `// ---------- Section ----------` banner comments to separate sections.
- **Keep the IIFE closed.** Game state is module-local; do not leak globals onto `window`.
- **Frame-rate independence.** Movement and timers use `dt` (seconds). New timers or speeds should be expressed per second and multiplied by `dt`.
- **Accessibility and motion:** respect the reduced-motion toggle (`reducedMotion`) when adding screen shake, flashes, or other motion effects. Announce important state changes through `announce()` for screen readers.
- **Touch and desktop parity:** any new control needs both a keyboard/mouse path and a touch path.
- **Persistence:** only write to `localStorage` through the `STORAGE_*` constants, and keep the `skyfighter.` prefix.
- **Canvas sizing:** the logical play field is `BASE_W` × `BASE_H` (480 × 720), scaled by `resize()`. Draw in logical coordinates.

## Testing and verification

There is no automated test suite. To verify a change:

1. Load `index.html` in a browser and check the browser console for errors.
2. Play through at least one stage on each difficulty you touched, including a boss fight.
3. Check pause/resume (`P`/`Esc`), game over, and the best-score update.
4. For UI or layout changes, check both a desktop viewport and a phone-sized viewport, since the game is designed to be playable on touch devices.
5. For audio changes, confirm the mute toggle still silences sound and that no errors occur before the first user gesture (`ensureAudio` runs on input).

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-47t38c`); do not push to `main` unless explicitly asked.
- Write clear, descriptive commit messages in the imperative mood, matching the existing history (for example, `Add Sky Fighter: Raiden-style vertical shoot-em-up game`).
- Do not open a pull request unless the user explicitly asks for one.

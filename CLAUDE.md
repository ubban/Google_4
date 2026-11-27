# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser.

- `index.html` — the whole game: inline `<style>`, markup, and one inline `<script>` (~1,670 lines). It is a single self-contained file with no build step, no bundler, no dependencies, and no network calls.
- `README.md` — a one-line title only.

There is no package manifest, test suite, linter, or CI configuration.

## Running the game

No build is needed. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Code architecture (`index.html`)

Everything lives in one IIFE (`(function () { 'use strict'; ... })()`). Main areas, in file order:

- **DOM references** — cached `getElementById` lookups at the top of the script.
- **Storage keys and config** — `STORAGE_*` keys (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`) and the `DIFFICULTIES` table (`easy` / `normal` / `hard`: spawn rates, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier). `KILLS_PER_STAGE` and `THEMES` (per-stage sky palettes) also live here.
- **Audio** — a WebAudio-based `sfx` layer (`ensureAudio`, `beep`, `noiseBurst`). Audio is created lazily on first user input.
- **Input** — `keys` map updated from `keydown`/`keyup`; touch joystick and fire/bomb buttons; pointer handling via `canvasPointFromEvent`. There is no device-motion/tilt input.
- **Game state and loop** — `resetGame`, `update(dt)`, `draw()`, and `loop(now)` driven by `requestAnimationFrame`. `update` and `draw` are kept separate; keep that split when adding features.
- **Weapons and bosses** — `fireWeapon` (Vulcan/Laser, 3 power levels), boss bullet patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`), `spawnBoss`, `updateBoss`, `killBoss`.
- **Progression and UI** — score multiplier chain (`addScore`, `breakChain`), smart bombs (`useBomb`), power-ups (`applyPowerUp`), HUD helpers (`renderHearts`, `renderBombs`, `showStageBanner`, `showComboPop`), overlay flow (`endGame`, `togglePause`).

### Controls

- Move: arrow keys / WASD (`KeyA`/`KeyD`/`KeyW`/`KeyS` are read from `keys`), or the on-screen joystick on touch devices.
- Fire: Space (hold), or the fire button on touch.
- Bomb: `X` or Shift, or the bomb button.
- Pause: `P` or Escape.
- Mute and reduced-motion toggles (buttons in the HUD) are persisted to `localStorage`. Reduced motion dampens screen shake and flashes; it is not a tilt-control setting.

## Conventions

- **Keep it a single file.** The project is deliberately dependency-free. Do not add a build system, package manager, or external CDN scripts unless the user asks for that change.
- **Preserve the existing style:** 2-space indentation, single quotes in JS, `const`/`let`, arrow functions for callbacks, and small named helper functions.
- **Frame-rate independence:** game logic takes a `dt` value; new timers and movement should use it rather than frame counts.
- **Mobile first:** the page sets `touch-action: none` and uses `viewport-fit=cover` and safe-area insets. Check that changes to the HUD or controls still work at phone widths.
- **Accessibility:** announcements go through the `srLive` live region (`announce`). Respect the reduced-motion setting when adding screen-shake, flashes, or other motion effects.
- **Persistence:** wrap any new `localStorage` access in try/catch. Private browsing and blocked storage can throw.
- **Git:** the default working branch for this checkout is `claude/gifted-tesla-9jx33q`. Commit only when asked, and do not open pull requests unless asked.

## Verifying changes

There are no automated tests. To verify a gameplay or UI change:

1. Serve or open `index.html` and play through at least one stage and a boss fight.
2. Check all three difficulty levels if you touched `DIFFICULTIES` or spawn logic.
3. Check the touch layout (for example, with browser device emulation) if you touched controls or HUD layout.
4. Confirm the console shows no errors during play, pause, game over, and restart.

Headless browser checks with Playwright are a reasonable way to automate step 4.

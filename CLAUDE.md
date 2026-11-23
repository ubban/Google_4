# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-em-up played in the browser on a 480×720 canvas.

## Repository layout

| Path | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS (`<style>`, ~lines 10–361), and JavaScript (`<script>`, ~lines 422–1668). |
| `README.md` | Placeholder title only. |

There is no build step, package manager, test suite, linter, or CI configuration. Do not add tooling unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). No dependencies need to be installed. The only external reference is an inline SVG favicon as a `data:` URI; nothing is loaded from a CDN.

## Code structure of `index.html`

Everything runs inside a single IIFE (`(function () { 'use strict'; ... })();`). Nothing is exposed on `window`, so functions are not reachable from the console.

- **DOM refs**: top-of-IIFE `getElementById` lookups for HUD, overlay, touch controls, and buttons.
- **Tuning tables**: `DIFFICULTIES` (spawn rates, enemy speed, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (sky/cloud/sun palettes cycled per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled by stage), `BOSS_PATTERNS` (bullet patterns).
- **Audio**: `ensureAudio()` lazily creates the `AudioContext` on first user gesture. `beep()` and `noiseBurst()` synthesize all sound effects; there are no audio files.
- **Input**: keyboard handlers on `window` (`keys` map keyed by `KeyboardEvent.code`), a mouse/pointer path via `canvasPointFromEvent`, and a touch joystick plus FIRE/BOMB buttons, shown only on touch devices.
- **Simulation**: `resetGame()` initializes state; `update(dt)` advances the world each frame; `updateBoss(dt)`, `applyPowerUp()`, `useBomb()`, `fireWeapon()`, and the `fire*` helpers implement combat.
- **Rendering**: `draw()` renders each frame, with helpers such as `drawPlane`, `drawBossShip`, and `drawClouds`.
- **Loop**: `loop(now)` is driven by `requestAnimationFrame`. `dt` is clamped to 50 ms. Updates and drawing run only when `running && !paused`.
- **Responsive scaling**: `resize()` fits the 480×720 base size to the viewport.

## Conventions to keep

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Keep CSS in the `<style>` block and JS in the IIFE.
- **Frame-rate independence.** Movement and timers use `dt` (seconds). Do not hard-code per-frame increments.
- **Keep the state machine intact.** Game flow is controlled by `running`, `paused`, and the overlay. Check that pausing, game over, and restart still work after changes.
- **Persisted settings use the `skyfighter.*` localStorage keys** (`STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`). Wrap storage access if you add more, because it can throw in private or restricted browsing modes. Changing a key's name resets players' saved data, so do not rename one casually.
- **Respect reduced motion.** Screen shake, flashes, and similar effects should check `reducedMotion`.
- **Accessibility.** Announcements go through `announce()` into the `srLive` live region. Keep the `controlsHint` text accurate when you change controls.
- **Keep the controls in sync.** Keyboard bindings are in the `keydown` handler, movement in `update()`, and the on-screen hint in `#controlsHint`. Update all three together. Current bindings: arrows/WASD move, Space fires, X or Shift bombs, P or Esc pauses.
- **Match the existing style.** Plain ES2015+ JavaScript, no frameworks, two-space indentation, `const` by default, and short helper functions.

## Testing and verification

No automated tests exist. After changing game logic, verify by loading `index.html` in a browser and checking:

1. Start, pause/resume, bomb, and game-over flows.
2. Each difficulty setting.
3. Stage progression (`KILLS_PER_STAGE`) and boss spawns.
4. Mobile layout, if you touched touch controls or CSS. Phone-width viewports matter here.

If a headless browser is available, a short Playwright script that loads the page and checks for console errors is a reasonable smoke test.

## Git

- Default development branch for this session: `claude/gifted-tesla-ui16f7`.
- Commit only when asked. Do not open pull requests unless explicitly requested.

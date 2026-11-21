# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game lives in one self-contained file, `index.html` (~1,670 lines): HTML markup, CSS in a `<style>` block, and JavaScript in a `<script>` block. There is no build step, no package manager, no framework, and no external dependencies.

Files:

- `index.html` — the whole game (markup, styles, script). This is the only file that matters for gameplay.
- `README.md` — a placeholder with only the title. Update it if you add setup or usage docs.

## Running the game

No build or install is needed. Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`) and visit it. Test changes by reloading the page.

There is currently no test suite, linter, or CI configuration. Verify changes by playing the game in a browser, paying particular attention to the area you touched.

## Code layout inside `index.html`

- **Head / styles** — Meta tags (viewport locked, `theme-color` `#0b1d3a`), an inline SVG favicon, and CSS for the canvas, HUD, overlay, boss bar, touch controls, and vignette/flash effects. Styles use `#id` selectors that the script queries directly.
- **DOM** — `#container` holds `#gameCanvas`, the HUD (`#hud`, with `#hearts`, `#bombs`, `#weaponStat`, `#multStat`), the boss bar (`#bossBarWrap`), `#topBar` (mute, reduced-motion, pause buttons), `#touchControls` (joystick plus FIRE/BOMB), and `#overlay` (title screen, pause, and game-over card with difficulty buttons `.diffBtn`).
- **Script** — a single IIFE-style block that grabs DOM references up front, then defines:
  - Persistence keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` (all in `localStorage`).
  - `DIFFICULTIES` (easy / normal / hard): spawn rates, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
  - `THEMES` — six sky palettes that cycle as the stage advances.
  - `ENEMY_TYPES` (grunt, interceptor, tank) and `BOSSES` (six bosses: VANGUARD through TITAN).
  - `KILLS_PER_STAGE = 18` — kills needed before a boss warning and boss fight.
  - Audio: a lazily created Web Audio context (`ensureAudio`), with `beep` and `noiseBurst` helpers and an `sfx` table. Audio starts only after a user gesture.
  - Input: keyboard (`keys` map keyed by `e.code`), touch joystick and buttons, and mouse steering with hold-to-fire.
  - Game state: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, plus score/lives/bombs.
  - Loop: `update(dt)` → `updateBoss(dt)` → `draw()`, driven by `requestAnimationFrame` through `loop(now)`. Includes an adaptive `quality` level (full / medium / low) based on frame-time tracking (`trackPerf`).

## Controls (for reference when testing)

- **Move:** arrow keys / WASD via `keys`, mouse hover over the canvas, or the touch joystick.
- **Fire:** hold Space (see `keys` handling) or hold the mouse button / FIRE button.
- **Bomb:** `X` or either Shift key, or the BOMB button.
- **Pause:** `P` or `Escape`, or the pause button.
- Keyboard and joystick input take priority over mouse steering when active.

## Conventions

- **Keep it a single file.** The project's design is one self-contained `index.html`. Do not introduce a build system, bundler, or separate asset files unless the user asks for that change.
- **Match the existing style.** Two-space indentation, single quotes in JS, camelCase identifiers, and `UPPER_SNAKE_CASE` for constant tables and storage keys.
- **Preserve accessibility hooks.** The page uses `aria-live` (`#srLive`) for announcements via `announce()`, and respects `prefers-reduced-motion`. Keep these working when changing UI or effects.
- **Preserve mobile behavior.** The viewport is fixed, `touch-action: none` is set, and touch controls appear only on touch devices (`isTouchDevice`). Check phone-sized layouts when changing HUD or overlay CSS.
- **Keep the canvas logical size.** The game renders at `BASE_W × BASE_H` (480 × 720) and is scaled by `resize()`. Coordinates should stay in base units.
- **Persist only the existing keys** unless a new one is needed; wrap new `localStorage` access so it does not throw in private browsing.

## Git

- Development branch: `claude/gifted-tesla-9z4h5z`. Commit and push there.
- Do not open a pull request unless explicitly asked.
- Commit messages should be short and imperative, describing what changed in the game (for example, "Add boss telegraph", "Fix HUD overflow on narrow screens").

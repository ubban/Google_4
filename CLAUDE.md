# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-em-up game built as a single self-contained HTML5 canvas page.

| File | Purpose |
| --- | --- |
| `index.html` | The whole game: markup, CSS (`<style>`, lines 10–359), canvas element, and all JavaScript (`<script>`, from line 422) |
| `README.md` | Placeholder title only |

There is no build step, package manager, bundler, test suite, or linter configured. Do not invent one. Open `index.html` directly in a browser (or serve the directory with any static server) to run the game.

## Repository layout and conventions

- Keep the game in one file. Do not split out CSS or JS files unless the user asks for it.
- No external dependencies. Everything is vanilla JS and the 2D canvas API, plus inline SVG for the favicon. Do not add CDN scripts or libraries without asking.
- The game logic lives in a single IIFE (`(function () { 'use strict'; ... })();`). Keep new code inside it so it does not leak globals.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, `let` when reassigned, and short section comments of the form `// ---------- Section name ----------`.
- The codebase is a flat set of top-level functions grouped by concern. Follow the same grouping when adding features.

## Architecture of `index.html`

### Markup and UI
- The HUD is a set of pill-style elements (score, best, multiplier, weapon, hearts for lives, bombs). Element IDs are looked up once at the top of the script (`document.getElementById(...)`). Add new IDs there, not inline in functions.
- The overlay card (`#overlay`) handles the title, game-over stats, and difficulty buttons (`.diffBtn`).
- Touch controls (`#touchControls`, joystick, fire and bomb buttons) are shown only when `isTouchDevice` is true (the `.enabled` class is added near line 779). Keyboard and mouse input are handled separately.
- `#srLive` is an ARIA live region. Use the `announce(text)` helper for screen-reader messages.

### Game constants
- `DIFFICULTIES` sets spawn timing, enemy speed, enemy shoot chance, starting lives and bombs, and boss HP multiplier for `easy`, `normal`, and `hard`.
- `KILLS_PER_STAGE` controls stage progression.
- `THEMES` holds the sky gradient, cloud, and sun colors for each stage. The theme index cycles with stage number.
- `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`) defines size, HP, speed multiplier, colors, score value, and the `minStage` at which each type appears.
- `BOSSES` defines the boss names, colors, and HP multipliers. `BOSS_PATTERNS` (near `fireRadial`) defines the attack patterns.
- `BASE_W` / `BASE_H` (480 × 720) is the logical play area. `resize()` scales the canvas to fit the window.

### Core loop
- `loop(now)` is driven by `requestAnimationFrame`. It clamps `dt` to 50 ms, calls `trackPerf` for adaptive quality, then runs `update(dt)` and `draw()` only when the game is running and not paused.
- `update(dt)` advances the simulation: player movement and firing, enemies, bullets, power-ups, collisions, bosses, and bombs.
- `draw()` renders everything. `drawPlane`, `drawBossShip`, and `drawClouds` are the sprite and background helpers.
- `trackPerf` adjusts a `quality` level (0 full, 1 medium, 2 low) based on frame timing. Respect `quality` when adding expensive effects.

### State and persistence
- `localStorage` keys are prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`, `reducedMotion`. Keep the prefix for any new key.
- `reducedMotion` defaults from `prefers-reduced-motion`, and the user can override it. Gate screen shake, flashes, and heavy animation on this flag.
- Audio is generated with WebAudio (`beep`, `noiseBurst`, `sfx`). `ensureAudio()` must run from a user gesture before any sound plays. Respect the `muted` flag.

### Game state lifecycle
- `resetGame()` initializes all per-run state. Add any new per-run variable there so restarts stay clean.
- `loseLife()`, `endGame()`, `togglePause()`, and `killBoss()` manage transitions. `endGame()` is where the best score is saved.

## Working in this codebase

- **Read before editing.** The file is large (~1,670 lines). Use `Grep` for the function or ID you need and read only that section.
- **Keep changes local.** Prefer adding or adjusting a function over restructuring the IIFE.
- **Verify in a browser.** There are no automated tests, so for gameplay changes, load `index.html` and check the affected behavior, including the touch layout if you touched UI.
- **Check for regressions in both input modes.** Keyboard, mouse, and touch paths are separate blocks of event listeners.
- **Be careful with the adaptive quality and reduced-motion paths** when adding visual effects.
- Do not commit generated or temporary files. The repo currently has no `.gitignore`, so check `git status` before committing.

## Git and delivery

- The default development branch for this work is `claude/gifted-tesla-7eabmm`. Commit and push there when the user asks for changes to be saved.
- Use clear, imperative commit messages that describe the change in the game or in the docs.
- Do not open a pull request unless the user explicitly asks for one.

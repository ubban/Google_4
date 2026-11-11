# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free front-end project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single HTML5 canvas page.

| File        | Purpose                                                                 |
|-------------|-------------------------------------------------------------------------|
| `index.html` | The entire game: CSS in `<style>` (starts ~line 10), markup in `<body>` (~line 363), and JS in one `<script>` IIFE (~line 422). ~1,670 lines total. |
| `README.md` | Placeholder (title only).                                               |

There is no build step, no package manager, no test suite, and no linter configured. Nothing needs to be installed.

## Running and testing

- **Run:** open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it.
- **Test:** no automated tests exist. Verify changes by loading the page and playing through the affected path (title screen → start → gameplay → game over). For gameplay or layout changes, check both desktop and a narrow/touch viewport, since touch controls and resizing are part of the feature set.

## Architecture of `index.html`

The file is organized top to bottom as:

1. **`<head>`** – viewport meta (no user scaling), theme color, an inline SVG favicon, and the `<title>`.
2. **`<style>`** – all CSS, including the HUD pills, glassmorphic overlay card, touch controls, and `.reduced-motion` styling.
3. **`<body>`** – a single `<canvas id="gameCanvas">`, the HUD elements, the overlay (title/start/game-over), touch controls (joystick, fire, bomb), and an `aria-live` region (`srLive`) for screen-reader announcements.
4. **`<script>`** – one `(function () { 'use strict'; ... })();` IIFE. Nothing is exposed on `window`; keep it that way unless there is a reason.

Major sections inside the script, in order:

- **DOM lookups** – every element is fetched once at the top.
- **Persistence** – `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). All reads/writes are direct; there is no schema or versioning.
- **Adaptive performance** – `trackPerf` samples frame times and steps a `quality` level (0–2) up or down.
- **Config tables** – `DIFFICULTIES` (spawn rates, enemy fire chance, lives, bombs, boss HP multiplier), `THEMES` (six stage backgrounds, cycled by stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), `BOSSES` (six bosses with colors and HP multipliers), `KILLS_PER_STAGE` (18).
- **Audio** – a lazily created `AudioContext` (`ensureAudio`) with `beep` and `noiseBurst` primitives and named SFX entries. Audio only starts after a user gesture.
- **Input** – keyboard (`keys` map keyed by `e.code`), touch joystick plus fire/bomb buttons, and mouse aiming on the canvas.
- **Game state and logic** – `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `applyPowerUp`, `addScore` / `breakChain` (kill-chain multiplier), `loseLife`, `endGame`, `togglePause`.
- **Boss patterns** – `BOSS_PATTERNS` is an array of bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring) selected by `updateBoss`.
- **Rendering** – `draw()` plus helpers `drawPlane`, `drawBossShip`, `drawClouds`, `renderHearts`, `renderBombs`, and HUD flash/shake/banner effects.
- **Main loop** – `loop(now)` clamps `dt` to 0.05 s, updates and draws only when running and not paused, then re-queues itself with `requestAnimationFrame`.

### Controls (for reference when changing input)

- Move: arrow keys or `WASD`; touch joystick on the left.
- Fire: hold `Space`, or the fire button / mouse button.
- Bomb: `X` or either `Shift`, or the bomb button.
- Pause: `P` or `Escape`.

## Conventions

- **Keep it a single file** unless the user asks to split it. Styles, markup, and script live together in `index.html` by design.
- **Style:** 2-space indentation, single quotes in JS, `const` by default, `let` only when reassigned. Match the surrounding code's style when editing.
- **Tuning values** belong in the config tables near the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`), not hard-coded in logic.
- **Logical canvas size** is `BASE_W × BASE_H` (480 × 720). `resize()` scales the canvas to fit the window; draw code should use logical coordinates.
- **Time-based movement:** update functions take `dt` in seconds. Use it instead of per-frame increments so gameplay stays consistent across frame rates.
- **Accessibility:** respect the reduced-motion toggle when adding screen shake, flashes, or other motion effects. Announce important state changes through `announce()` so they reach `srLive`.
- **Persistence:** wrap new `localStorage` keys with the `skyfighter.` prefix and read them with a sensible default.
- **Audio:** do not start an `AudioContext` outside a user gesture; go through `ensureAudio()`.

## Git and workflow

- Default branch work happens on the feature branch the session was started on (currently `claude/gifted-tesla-b6w3it`). Do not push to other branches without explicit permission.
- Commit messages in this repo are descriptive, with a short summary line followed by a body that explains the change (see `git log`).
- Do not open pull requests unless the user explicitly asks for one.
- Before committing, load `index.html` in a browser and exercise the changed feature.

## Things to be careful about

- The game loop and input handlers are all inside one closure; avoid introducing globals to work around ordering problems. Reorder or pass arguments instead.
- Boss pattern functions and enemy spawning run every frame during gameplay. Avoid allocating large arrays or creating new `AudioContext`/`Audio` objects inside them.
- Changing the `localStorage` key names will orphan existing players' saved best score and settings.

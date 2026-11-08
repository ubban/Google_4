# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` (no build step, no dependencies, no external assets).
- `README.md` — minimal project readme.

There is no package manager, bundler, test suite, or linter configured. Do not introduce one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, e.g. `python3 -m http.server` from the repo root, then visit `http://localhost:8000`.

## Code structure (`index.html`)

- **Head/CSS**: dark color scheme, full-viewport fixed container, canvas scaled to fit with `max-width`/`max-height`, pill-style HUD, glassmorphic overlay card, mobile touch controls. Keep `viewport` meta and `touch-action: none` intact for mobile play.
- **Script constants** (near the top of the script): `BASE_W`/`BASE_H` (480×720 logical playfield, scaled to fit the viewport), `STORAGE_*` localStorage keys (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`), `DIFFICULTIES` tuning table, `KILLS_PER_STAGE`.
- **Audio**: Web Audio API synthesis (`ensureAudio`, `beep`, `noiseBurst`), created lazily after a user gesture.
- **Game state & loop**: `resetGame()` initializes state; `update(dt)` advances the simulation each frame; rendering helpers draw planes, bosses, bullets, power-ups, and explosions on the canvas.
- **Weapons**: Vulcan/Laser with three power levels (`fireWeapon`, `applyPowerUp`).
- **Bosses**: multi-pattern bosses (`spawnBoss`, `updateBoss`, `killBoss`) using pattern helpers such as `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`.
- **Scoring**: kill-chain multiplier (`addScore`, `breakChain`), capped at 10x.
- **Bombs**: limited smart bombs clear the screen and damage bosses (`useBomb`).
- **Lifecycle**: `loseLife`, `endGame` (persists best score), `togglePause`.

## Controls (keep documented behavior in sync)

- Move: Arrow keys or WASD; mouse movement over the canvas and touch joystick also move the ship.
- Fire: Space (hold), or the on-screen fire button on touch.
- Bomb: X or Shift, or the on-screen bomb button.
- Pause: P or Escape.

## Conventions

- Keep everything in `index.html` unless the user asks for a split into multiple files.
- Use plain ES2015+ JavaScript and vanilla DOM/canvas APIs. Do not add frameworks or CDN dependencies without asking.
- Game-balance numbers live in the `DIFFICULTIES` table and related constants; prefer tuning there over hard-coding values in logic.
- Respect the `reducedMotion` setting when adding screen shake, flashes, or other motion effects.
- Any new persisted setting should use a `skyfighter.*` localStorage key, read and written in a try-safe manner consistent with existing code.
- Keep the layout working on phone-sized viewports (touch controls, safe-area insets) as well as desktop.

## Verifying changes

Since there are no automated tests:

1. Load `index.html` in a browser and play through at least one stage and a boss fight after gameplay changes.
2. Check the console for errors.
3. For UI or layout changes, check both a desktop-sized and a phone-sized viewport.

## Git workflow

- Develop on the branch designated for the task (currently `claude/gifted-tesla-890vgx`).
- Write clear, descriptive commit messages.
- Do not open pull requests unless explicitly asked.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up rendered on an HTML `<canvas>`.

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` (roughly lines 460–1669). No build step, no bundler, no dependencies, no external network requests.
- `README.md` — minimal project title stub.

There is no package.json, test suite, linter, or CI configuration.

## Running and checking changes

- Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`).
- There is no build or test command. To validate JavaScript syntax after editing, extract the inline script and parse it, for example:
  ```bash
  node -e "const s=require('fs').readFileSync('index.html','utf8');const m=s.match(/<script>([\s\S]*)<\/script>/);new Function(m[1]);console.log('ok')"
  ```
- For behavioral changes, play the game in a browser and check gameplay, HUD, audio, and mobile layout. Use the Playwright install already present in the environment if automation is needed; do not reinstall it.

## Code layout inside `index.html`

The script is organized roughly in this order:

1. **Storage keys** (`STORAGE_*`) — persisted via `localStorage` for best score, mute, difficulty, and reduced motion. All reads/writes should tolerate storage being unavailable.
2. **Tuning tables** — `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage palettes), `ENEMY_TYPES`, `BOSSES`. Prefer changing balance here rather than in logic.
3. **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers. Audio is created lazily after a user gesture.
4. **HUD and effects** — `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`, and the `announce` live-region helper for screen readers.
5. **Input and layout** — `resize` (fixed `480×720` base canvas, scaled to fit), keyboard handlers, mouse/touch handling, and a virtual joystick for touch.
6. **Game state** — `resetGame`, `spawnEnemy`, `spawnBoss`, power-ups, explosions, scoring/combo (`addScore`, `breakChain`), bombs (`useBomb`), and weapons (`fireWeapon`; vulcan and other types, upgraded by `player.level`).
7. **Update loop** — `update(dt)` for movement, collisions, and spawning; `updateBoss` with pattern functions in `BOSS_PATTERNS` (`fireRadial`, `fireAimedSpread`, `fireCross`, `fireHomingSeeded`, etc.); `loseLife`, `killBoss`, `endGame`, `togglePause`.
8. **Rendering** — `draw()` plus helpers `drawPlane`, `drawBossShip`, `drawClouds`. Rendering must not mutate game state.
9. **Main loop** — `loop(now)` drives `update` and `draw` with a delta-time step.

## Controls (for reference when changing input)

- Keyboard: Arrows/WASD move, Space fires, X or Shift bombs, P or Esc pauses.
- Mouse: move to steer, hold click to fire.
- Touch: drag left zone to fly, FIRE and BOMB buttons.

Input uses `e.code` (physical key), so keep bindings keyed on `KeyX`-style codes.

## Conventions

- Keep the game as a single self-contained HTML file unless the user asks for a split. Do not add npm dependencies or build tooling without being asked.
- Preserve accessibility hooks: keep `announce()` calls for stage changes and boss events, and respect the reduced-motion setting (`reducedMotion`, which defaults from `prefers-reduced-motion`).
- Keep the layout working at phone width (the page uses `touch-action: none` and `overflow: hidden`; avoid introducing scrolling).
- Keep balance changes in the tuning tables where possible, and mention balance impact in commit messages.
- Use `requestAnimationFrame`-driven timing via the existing `dt` pattern; do not introduce fixed-frame assumptions.

## Git

- Development branch for this task: `claude/gifted-tesla-a80fdv`. Commit and push there with `git push -u origin <branch>`.
- Write clear, descriptive commit messages. Do not open pull requests unless explicitly asked.

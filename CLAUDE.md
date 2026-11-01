# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

**Sky Fighter** — a single-file, browser-based, Raiden-style vertical shoot-'em-up. There is no build step, no package manager, no test suite, and no external assets: all markup, CSS, game logic, and audio are inline in one HTML file.

## Repository layout

| Path | Purpose |
| --- | --- |
| `index.html` | The entire game (~1,670 lines): HTML overlay/HUD, inline `<style>`, and one inline `<script>` IIFE. |
| `README.md` | Minimal project title. |

There are no subdirectories, bundlers, `package.json`, linters, or CI configuration.

## Running the game

- Open `index.html` directly in a modern browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There is nothing to install or compile.

## Verifying changes

- No automated tests exist. Verify by loading the page in a browser and playing through the affected behavior.
- For a quick syntax sanity check of the script block, extract the `<script>` contents and run `node --check` on them, e.g.:
  `sed -n '/<script>/,/<\/script>/p' index.html | sed '1d;$d' > /tmp/sf.js && node --check /tmp/sf.js` (use the scratchpad directory rather than `/tmp` if preferred).

## Code structure (inside `index.html`)

The script is wrapped in an IIFE. Major sections, in source order, are marked with `// ---------- Name ----------` comments:

- **Adaptive performance** — `trackPerf` samples frame times and adjusts a `quality` level (0–2).
- **Config constants** — `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (background palettes per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six boss definitions).
- **Persistence** — `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Reads/writes are not wrapped in try/catch; keep that in mind if changing them.
- **Audio** — WebAudio synth only (`ensureAudio`, `beep`, `noiseBurst`, `sfx.*`). No audio files.
- **HUD / effects** — `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`.
- **Canvas sizing** — fixed logical resolution `BASE_W = 480`, `BASE_H = 720`, scaled by `resize()`.
- **Input** — keyboard (`keys` map on `window`), mouse/pointer on the canvas (`canvasPointFromEvent`), and touch joystick + FIRE/BOMB buttons shown when a touch device is detected.
- **Game state** — `resetGame`, `player` object (`weapon: 'vulcan' | 'laser'`, `level` 1–3), `powerUps`, enemies, bullets.
- **Rendering** — `drawPlane`, `drawBossShip`, `draw`, `drawClouds`.
- **Spawning & combat** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `addScore`, `breakChain`, `rectsOverlap`.
- **Update** — `update(dt)` drives movement, collisions, and stage progression; `applyPowerUp` handles `life`, `bomb`, and `weapon` power-ups.
- **Bosses** — `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring), `updateBoss`, `killBoss`.
- **Game flow** — `loseLife`, `endGame`, `togglePause`, and the `loop(now)` requestAnimationFrame driver. `dt` is clamped to 0.05 s.

## Controls (as implemented)

- Move: arrow keys / WASD, or mouse/pointer on the canvas.
- Fire: Space (or hold the mouse button / FIRE button).
- Bomb: X or either Shift key (or BOMB button).
- Pause: P or Escape.
- Mute, reduced-motion, and difficulty are toggled from the HUD/menu.

## Conventions

- Keep the game self-contained in `index.html`. Do not add external scripts, stylesheets, or fonts unless the user asks; this is a deliberate constraint of the project.
- Match the existing style: 2-space indentation, `const`/`let`, compact one-line object literals for config tables, and section banner comments.
- When adding a new enemy, boss, theme, or power-up, extend the existing config tables and switch branches (`ENEMY_TYPES`, `BOSSES`, `THEMES`, `applyPowerUp`, `BOSS_PATTERNS`) rather than adding parallel code paths.
- Gameplay tuning lives in `DIFFICULTIES` and the config tables; prefer changing those over hard-coding numbers in `update`/`draw`.
- Preserve the mobile/touch behavior and the reduced-motion option (`reducedMotion`) when changing visual effects like screen shake, flashes, or pulses.
- Keep the page working without `localStorage` where practical; if you add storage access, wrap it in try/catch.

## Git

- Development branch for this task: `claude/gifted-tesla-4675ez`.
- Commit with clear messages; push with `git push -u origin <branch>`.
- Do not open pull requests unless explicitly asked.

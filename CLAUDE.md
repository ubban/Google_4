# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game lives in one file, `index.html` (~1,670 lines), containing inline `<style>`, markup, and a `<script>` block that uses the Canvas 2D API and the WebAudio API.

Files:

- `index.html` — the whole game (HTML, CSS, and JavaScript).
- `README.md` — a one-line title. Keep it accurate if the project description changes.

There is no package manager, build step, bundler, test suite, or linter configured. Nothing needs to be installed.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser. For gameplay or rendering changes, use the browser tooling available in the environment and check the actual canvas output rather than relying on reading the code alone.

## Code structure (inside `index.html`)

The script is organized by section comments. Major parts, in source order:

- **Constants and storage keys** (`STORAGE_*`, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BASE_W`/`BASE_H`). Persistence uses `localStorage` keys prefixed `skyfighter.`: best score, mute, difficulty, and reduced motion.
- **Audio** (`ensureAudio`, `beep`, `noiseBurst`, SFX tables). All sound is synthesized with WebAudio; the project ships no audio assets. Audio is created lazily on the first user gesture.
- **Rendering helpers** (`drawPlane`, `drawBossShip`, `drawClouds`, `draw`), HUD helpers (`renderHearts`, `renderBombs`, banners, combo pops), and screen shake.
- **Input** — keyboard (`keydown`/`keyup` on `window`), touch joystick and fire button, and mouse/pointer position mapped to canvas coordinates.
- **Game state and entities** — `resetGame`, spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), collision (`rectsOverlap`), scoring and combo chain (`addScore`, `breakChain`), bombs (`useBomb`), and weapons (`fireWeapon`, `applyPowerUp`).
- **Bosses** — `BOSS_PATTERNS` holds bullet-pattern generators (radial, spread, columns, spiral, cross, homing, double ring) used by `updateBoss`.
- **Game loop** — `update(dt)` for logic, `draw()` for rendering, and `loop(now)` driving `requestAnimationFrame`. Pause is handled by `togglePause`; game over by `endGame`/`loseLife`.

Keyboard controls currently bound: `Space` (fire), `KeyP`/`Escape` (pause), `KeyX`/`ShiftLeft`/`ShiftRight` (bomb). Preserve `e.preventDefault()` on `Space` so the page does not scroll.

## Conventions

- Keep the game as a single self-contained file unless the user asks for a split. Do not add npm dependencies, a build step, or external assets without a clear reason and the user's agreement.
- External resources are not needed. Keep the page working offline.
- Use the existing naming style: `camelCase` for functions and variables, `UPPER_SNAKE_CASE` for constants.
- Preserve mobile support: the page uses `touch-action: none`, a responsive canvas sized in `resize()`, and safe-area insets. Check that changes to layout or input still work on touch and at narrow widths.
- Respect the reduced-motion setting when adding visual effects (screen shake, flashes, pulses).
- Keep `STORAGE_*` keys stable; changing them would reset players' saved best score, mute, difficulty, and motion settings.

## Git

- Development branch for this workspace: `claude/gifted-tesla-umdpqr`. Do not push to other branches without explicit permission.
- Commit only when asked. Do not open a pull request unless the user asks for one.
- Commit messages are short imperative sentences, matching the existing history (for example, `Add Sky Fighter: Raiden-style vertical shoot-em-up game`).

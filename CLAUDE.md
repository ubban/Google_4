# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game lives in a single file, `index.html`, which contains inline CSS, inline JavaScript, and a `<canvas>`-based renderer. There is no build step, package manager, test suite, or server requirement.

Files:

- `index.html` — the whole game (HTML, CSS, and one `<script>` block starting around line 422).
- `README.md` — minimal project title.
- `CLAUDE.md` — this file.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`) and visit it. No install step is needed.

## Code structure (inside `index.html`)

- **Storage keys** (~line 463): `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. These are the only persisted state, all via `localStorage`.
- **Difficulty** (`DIFFICULTIES`, ~line 485): named presets (default `normal`) selected from the in-game UI and persisted.
- **Audio** (`ensureAudio`, `beep`, `noiseBurst`, ~line 545+): Web Audio synthesis only; no audio files. Audio is created lazily on the first user gesture.
- **HUD / effects** (~line 680–730): hearts, bombs, flashes, combo pops, stage banners, screen shake. Reduced-motion mode disables some of these.
- **Input** (~line 769–840): keyboard listeners on `window` (`keydown`/`keyup` on `e.code`), plus touch joystick and fire/bomb buttons for mobile.
- **Game state** (`resetGame`, ~line 875): reset of all entities and counters.
- **Rendering** (`drawPlane`, `drawBossShip`, `draw`, ~line 913+): all drawing is done with Canvas 2D primitives; there are no image assets.
- **Spawning and combat** (`spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `applyPowerUp`, boss attack patterns `fireRadial`/`fireAimedSpread`/`fireSpiralBurst`/etc., ~line 1022–1420).
- **Main loop** (`update(dt)`, `loop(now)` driven by `requestAnimationFrame`, ~line 1148 and ~line 1640).

## Controls (for reference when editing input code)

- Move: arrow keys or WASD
- Fire: Space (or touch fire button)
- Smart bomb: X or Shift (or touch bomb button)
- Pause: P or Escape

## Conventions

- Keep the game as a single self-contained HTML file unless the user asks for a restructure. Do not introduce a bundler, framework, or npm dependencies without explicit request.
- Use plain ES2015+ JavaScript and the existing style (2-space indent, `const`/`let`, camelCase identifiers, `STORAGE_*` constants for persisted keys).
- Any new persisted setting should get a `skyfighter.`-prefixed key and be wrapped where it is read so a blocked or unavailable `localStorage` does not throw.
- Keep the mobile layout working: the viewport meta disables zoom, `touch-action: none` is set on the canvas and body, and touch controls must keep working alongside keyboard input.
- Respect the reduced-motion setting when adding new screen shake, flashes, or large animations.

## Verification

There is no automated test suite or linter in the repo. After changing gameplay or rendering code, verify by loading `index.html` in a browser and playing through the affected path (movement, firing, bombs, boss phases, pause, game over, and best-score saving). For UI-only changes, check both desktop and a narrow mobile viewport.

## Git

- Default development branch for this session: `claude/gifted-tesla-00ktam`.
- Write clear, imperative commit messages describing the gameplay or code change.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser on a single HTML5 canvas. Players switch weapons, drop smart bombs, and fight stage bosses.

The entire game lives in **`index.html`** (~1,670 lines): inline CSS in `<style>`, markup, and one inline `<script>` (starts around line 422) that holds all game logic. `README.md` is a placeholder.

There is no build step, package manager, test suite, or linter. Nothing is installed or compiled.

## Running the game

- Open `index.html` directly in a browser, or serve the folder with any static server (for example, `python3 -m http.server`).
- There are no dev, build, or test commands. Verify changes by playing them in a browser.
- For headless checks, Playwright is pre-installed in the cloud environment. Launch Chromium with `chromium.launch()` and load the file over `file://` or a local server.

## Code layout (inside the `<script>` in `index.html`)

Sections are marked with `// ---------- Name ----------` comments. Rough order:

1. **DOM refs and constants**: element lookups (`canvas`, `overlay`, `hud`, touch controls) and the `STORAGE_*` keys.
2. **Tuning tables**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (one per stage, cycling background palettes), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), and `BOSSES`.
3. **Persistent state**: best score, mute, difficulty, and reduced motion, read from and written to `localStorage`.
4. **Audio**: a lazily created `AudioContext` (`ensureAudio`) with `beep` and `noiseBurst` helpers. `sfx` holds the named sound effects.
5. **HUD and effects**: `renderHearts`, `renderBombs`, flashes, combo pops, stage banners, and screen shake.
6. **Input**: keyboard handlers, touch joystick and fire/bomb buttons, and mouse drag on the canvas.
7. **Game flow**: `resetGame`, `resize`, `togglePause`, `endGame`, `loseLife`, and `killBoss`.
8. **Entities and combat**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, and the boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
9. **Update and draw**: `update(dt)` runs game logic, `draw()` renders, and `loop(now)` drives both from `requestAnimationFrame`.
10. **Adaptive quality**: `trackPerf` measures frame times and lowers `quality` (0 full, 1 medium, 2 low) when frames are slow.

Game state is kept in top-level `let` variables inside the script, not in classes or modules.

## Controls

| Input | Action |
| --- | --- |
| Arrow keys / WASD / mouse | Move (keyboard codes are read from `e.code`) |
| Space / hold mouse button | Fire |
| `X` or either Shift | Smart bomb |
| `P` or `Esc` | Pause |
| Touch | Virtual joystick (left), fire and bomb buttons (right) |

## Persistence

`localStorage` keys. Do not rename them, because existing players would lose their saved data:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Every read and write should be wrapped in try/catch, since storage can be unavailable in private windows or when site data is blocked. This is a suggestion for new code, not something the current code guarantees everywhere.

## Editing guidelines

- **Keep it a single file with no dependencies.** Do not add npm packages, bundlers, or external scripts unless the user asks for them.
- **Keep tuning in the tables.** Change difficulty, enemy stats, boss HP multipliers, and stage palettes in the constant objects near the top of the script rather than hard-coding numbers in logic.
- **Preserve touch support.** The game must stay playable on phones: keep the viewport meta, `touch-action: none`, and the on-screen controls working.
- **Respect reduced motion.** New screen shake, flashes, or particle effects should check the `reducedMotion` flag.
- **Keep the frame loop cheap.** `update` and `draw` run every frame. Avoid per-frame allocations in hot paths, and respect the `quality` level when adding effects.
- **Test in a browser.** There is no automated suite. After changes, play through at least one stage and check the HUD, a boss fight, game over, and the pause screen on both desktop and a narrow viewport.

## Git

- Work on the branch assigned for the session. Do not push to other branches without permission.
- Commit only when asked, and write clear commit messages that describe the gameplay or code change.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game lives in one file, `index.html` (about 1,670 lines), built on a single `<canvas>` with a synthesized WebAudio soundtrack. There are no external assets, no network calls, and no build step.

| File | Purpose |
| --- | --- |
| `index.html` | The whole game: markup, CSS, and JavaScript |
| `README.md` | Placeholder title only |

Repository layout of `index.html`:

- Lines ~10–359: `<style>` (CSS for HUD, overlay card, touch controls, boss bar, animations)
- Lines ~361–421: HTML markup (canvas, HUD, overlay, touch controls, icon buttons)
- Lines ~422–1667: `<script>` (all game logic, wrapped in an IIFE with `'use strict'`)

Line numbers drift as the file changes; search for the identifiers below rather than trusting numbers.

## Running and verifying

- There is no `package.json`, no bundler, no linter, and no test suite. Do not invent `npm` scripts or test commands.
- To run the game, open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- Verify changes by playing them in a browser. Check at least: keyboard controls, touch layout (a narrow viewport), pause/resume, game over, and the best-score persistence.
- For a syntax-only check of the script, extract the `<script>` block and run `node --check` on it.

## Architecture inside `index.html`

### Configuration tables (top of script)

- `DIFFICULTIES` — `easy` / `normal` / `hard`. Each sets `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, and `bossHpMul`.
- `KILLS_PER_STAGE` — kills needed before a boss is spawned (18).
- `THEMES` — six sky palettes, cycled per stage (`name` is shown in the stage banner).
- `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP, unlocks at stage 2).
- `BOSSES` — six bosses with hull/wing/core colors and `hpMul`.

When tuning balance, change these tables rather than hard-coding numbers in `update()`.

### Core state and loop

- Top-level game state is declared with `let` near the "game state" block: `score`, `lives`, `bombCount`, `stage`, `kills`, `stageKills`, `chain`/`multiplier`, `player`, and the entity arrays (`bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`).
- `resetGame()` initializes a fresh run from the active difficulty.
- `update(dt)` advances the simulation one step. It handles spawning, collisions, the stage/boss trigger, and pickups.
- `loop(now)` runs once per animation frame via `requestAnimationFrame`, calls `update` and the draw functions, and tracks frame time through `trackPerf()`.
- Boss patterns are separate functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) dispatched from `updateBoss(dt)`.
- `endGame()` and `togglePause()` handle the terminal and paused states.

### Progression

1. Each stage spawns enemies until `stageKills >= KILLS_PER_STAGE`.
2. The boss is then spawned about 1.2 s later (guarded by `bossWarned`).
3. Defeating the boss advances `stage`, resets stage counters, and shows the stage banner.

Power-ups (`powerUps`, applied in the pickup handler): `life` (red), `bomb` (amber), and `weapon` (green), which toggles between `vulcan` and `laser`. Weapon fire delay is 0.15 s for vulcan and 0.1 s for laser.

### Input

- **Keyboard:** arrow keys / WASD to move (tracked in the `keys` map by `e.code`), `Space` to fire, `X` / `Shift` to use a bomb, `P` / `Escape` to pause.
- **Mouse:** moving over the canvas steers the ship (`mouseTarget`); mouse-down on the canvas or on the fire button fires.
- **Touch:** a virtual joystick (`moveZone`, `joystickBase`, `joystickStick`) plus `fireBtn` and `bombBtn`. Touch handlers call `preventDefault` and are non-passive.

Keep all three input paths working when you change controls.

### Rendering and layout

- The logical play field is `BASE_W = 480` × `BASE_H = 720`. `resize()` scales the canvas to fit the window with a cap of 1.6×, so game coordinates stay fixed.
- HUD elements (score, best, multiplier, weapon, hearts, bombs, boss bar, stage banner, combo pop) are DOM elements layered over the canvas, not drawn on it.
- Visual effects (screen shake, hit flash, vignette pulse, bomb flash) respect `reducedMotion`. Any new effect should check it too.
- `quality` (0 full, 1 medium, 2 low) is lowered automatically by `trackPerf()` when frame times are poor.

### Audio

- All sound is synthesized with WebAudio (`beep`, `noiseBurst`, and the `sfx` map). There are no audio files.
- `ensureAudio()` must run from a user gesture (key, click, or touch) because browsers block audio before one.
- Respect `muted`. Audio nodes are not created when no `AudioContext` exists.

### Persistence

`localStorage` keys (defined as `STORAGE_*` constants; keep the `skyfighter.` prefix):

| Constant | Key | Value |
| --- | --- | --- |
| `STORAGE_BEST` | `skyfighter.bestScore` | number |
| `STORAGE_MUTE` | `skyfighter.muted` | `'1'` / `'0'` |
| `STORAGE_DIFF` | `skyfighter.difficulty` | `easy` / `normal` / `hard` |
| `STORAGE_REDUCED` | `skyfighter.reducedMotion` | `'1'` / `'0'` (falls back to `prefers-reduced-motion`) |

Storage can throw in private browsing or when blocked. Wrap new reads and writes in `try`/`catch` and keep the game working without them. The existing code does not yet do this everywhere, so match the current pattern only where you are confident it is safe.

### Accessibility

- A polite live region (`srLive`) is written to by `announce()`. Use it for important state changes (stage changes, pause, reduced-motion toggles).
- Icon buttons carry `title` attributes; keep them when adding controls.

## Conventions

- **Single file.** Keep everything in `index.html` unless the user asks for a split. Do not add frameworks, CDN scripts, or external fonts or images.
- **Match the local style.** Two-space indentation, single quotes in JS, `const` by default, and short section comments such as `// ---------- Audio ----------`.
- **Data-driven tuning.** Put new enemy, boss, difficulty, or theme values in the existing tables.
- **Keep the HUD DOM-based** and update it through the existing helpers (`renderHearts`, `renderBombs`, `showStageBanner`, and so on) rather than writing `innerHTML` in new places.
- **Do not use `alert`, `confirm`, or `prompt`.** The overlay card is the UI for menus and messages.
- **Mobile first.** Every change to layout or HUD must still fit a narrow portrait phone with the touch controls visible.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-m72bql`). Do not push to `main` directly.
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Do not open a pull request unless the user asks for one.

## Known gaps

- No automated tests, linting, or CI.
- No `package.json` or `.gitignore`.
- The README is a placeholder.

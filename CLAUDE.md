# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

Google_4 is a single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up with weapon swaps, smart bombs, stage bosses, and mobile touch controls.

- `index.html` — the entire game: markup, CSS (in a `<style>` block), and the game engine (in one `<script>` IIFE, starting around line 422). There is no build step, no bundler, no `package.json`, and no test suite.
- `README.md` — a stub with only the title.

There are no dependencies. Open `index.html` in a browser to run it. The `run` skill can launch it for a visual check.

## Running and verifying changes

- No install, build, lint, or test commands exist. Verify changes by loading `index.html` in a browser (or headless Chromium) and playing through the affected path.
- Quick syntax check for the script block, if Node is available (write the extracted file to a scratch directory, not the repo):
  ```sh
  sed -n '/<script>/,/<\/script>/p' index.html | sed '1d;$d' > "$SCRATCH/sf.js" && node --check "$SCRATCH/sf.js"
  ```
- Check both desktop (keyboard/mouse) and touch layouts when touching HUD, overlay, or control code. The layout must work at phone width with no horizontal scroll.

## Code layout in `index.html`

Order inside the `<script>` IIFE, top to bottom:

1. **DOM refs and storage keys** — `getElementById` lookups; `STORAGE_*` keys under the `skyfighter.*` prefix.
2. **Audio** — lazily created `AudioContext` (`ensureAudio`), `beep` / `noiseBurst` primitives, and the named `sfx` table. Sound is muted via `muted` and persisted.
3. **HUD and effects** — `renderHearts`, `renderBombs`, flash/vignette, combo pop, stage banner, screen shake.
4. **Config tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (one per stage, cycles), `ENEMY_TYPES`, `BOSSES` (one per stage, cycles).
5. **Input** — `keys` map for keyboard, a virtual joystick and fire/bomb buttons for touch, and mouse steering/firing on the canvas.
6. **Game state** — module-level `let`s: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, plus score, lives, bombs, stage, chain/multiplier.
7. **Drawing** — `drawPlane`, `drawBossShip`, `draw()`, `drawClouds`.
8. **Simulation** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `update(dt)`, `updateBoss(dt)`, and boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) gathered in `BOSS_PATTERNS`.
9. **Flow** — `resetGame`, `togglePause`, `loop(now)` (driven by `requestAnimationFrame`, `dt` clamped to 50 ms), and the start button handler.

Conventions in the existing code:
- Plain, non-module JavaScript (ES2015+ features such as arrow functions, template literals, `const`/`let`). No TypeScript, no framework.
- Naming: `camelCase` for functions and variables, `UPPER_SNAKE` for constants and config tables, `STORAGE_*` for localStorage keys.
- Comments are sparse. Add one only where the intent isn't obvious from the code (e.g., the `quality` levels or the 2-D perspective trick).
- Keep entities as plain objects in arrays; remove them when they leave the screen or die.
- Drawing uses the 2-D canvas API directly; use `quality` (0 full, 1 medium, 2 low) to gate expensive effects such as `shadowBlur`.

## Game-balance knobs

Most tuning lives in data tables, so prefer editing those over touching logic:

- `DIFFICULTIES` — spawn rate, enemy speed, enemy fire chance, starting lives and bombs, boss HP multiplier.
- `ENEMY_TYPES` — size, HP, speed, score, and `minStage` (when the type first appears).
- `BOSSES` — names, colors, `hpMul`. Boss base HP is set in `spawnBoss` (`70 + stage * 35`, scaled by `hpMul` and difficulty).
- `BOSS_PATTERNS` — the attack pattern list per boss, indexed by boss design.
- `KILLS_PER_STAGE` — kills needed before a boss warning.
- `THEMES` — background palettes per stage.

## Input and controls (for reference when editing)

- Move: Arrow keys or WASD. Fire: Space (hold), or hold the mouse button on the canvas.
- Bomb: `X` or Shift. Pause: `P` or Esc.
- Mouse: move to steer, click to fire.
- Touch: drag in the left zone to fly (virtual joystick), tap FIRE, tap BOMB.
- Space's default is prevented to stop page scrolling.

## Persistence

Stored in `localStorage`. Reads and writes are not wrapped in try/catch (only audio setup is), so a blocked-storage browser will throw. Keep that in mind if you touch these calls:

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | High score |
| `skyfighter.muted` | `'1'` when sound is off |
| `skyfighter.difficulty` | `easy` / `normal` / `hard` |
| `skyfighter.reducedMotion` | Explicit override; absent means follow `prefers-reduced-motion` |

Changing any key name breaks saved data for existing players, so treat the storage keys as a public contract.

## Accessibility and performance

- Respect reduced motion: `reducedMotion` toggles the `reduced-motion` body class and should gate shake, flashes, and heavy animation.
- Keep the `aria-live` region (`srLive`) updated via `announce()` for important state changes (stage, game over, pause).
- Frame-time monitoring (`trackPerf`) auto-adjusts `quality`. New visual effects should degrade at `quality >= 1`.
- Keep the game loop allocation-light: reuse arrays with filtering, avoid creating closures per frame in `update`/`draw`.

## Working in this repo

- Work on the branch named in the session instructions (currently `claude/gifted-tesla-1ybla6`). Don't push to `main` directly.
- Commit messages: short imperative summary in the style of the existing history (e.g., `Add Sky Fighter: ...`).
- Because everything is one file, keep diffs focused: edit the relevant section rather than reformatting unrelated code.
- Don't introduce a build step, package manager, or external script/CSS dependencies unless the user asks for it. The game is intentionally self-contained.

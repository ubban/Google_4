# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free static web project. Its only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs entirely in the browser.

Files:

- `index.html` — the entire game: inline CSS (`<style>`), markup, and inline JavaScript (`<script>`, wrapped in an IIFE). No build step, no packages, no external assets.
- `README.md` — one-line title only.
- `CLAUDE.md` — this file.

There is no `package.json`, test suite, linter, or CI configuration in the repo.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server` from the repo root) and visit `http://localhost:8000`.
- There are no automated tests. Verify changes by playing the game in a browser: start a run, fire, use bombs, collect power-ups, defeat a boss, lose all lives, and check the game-over screen.
- Check both desktop (keyboard/mouse) and touch layouts when changing controls, HUD, or the overlay. The canvas scales to fit the viewport, and the HUD and touch controls are positioned for phone widths.

## Code structure of `index.html`

The inline script is organized roughly in this order:

1. **Storage keys** (`STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`) — persisted to `localStorage` under the `skyfighter.*` prefix. Reads and writes are not wrapped in try/catch; keep that in mind if you change them, since storage can be unavailable in some private or sandboxed contexts.
2. **Tuning tables**:
   - `DIFFICULTIES` — `easy` / `normal` / `hard`, controlling spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
   - `KILLS_PER_STAGE` (18) — kills needed before a boss warning and fight.
   - `THEMES` — six sky palettes cycled by stage.
   - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP); `minStage` gates when each appears.
   - `BOSSES` — six bosses cycled by stage, each with colors and an `hpMul`.
   - `BOSS_PATTERNS` — per-boss attack patterns built from `fireRadial` and `fireAimedSpread`.
3. **Audio** — synthesized with the Web Audio API (`beep`, `noiseBurst`, and the `sfx` map). No audio files.
4. **HUD and effects** — `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`, and an `announce()` helper that feeds an `aria-live` region for screen readers.
5. **Input** — keyboard (`keydown`/`keyup` on `window`, keyed by `e.code`), mouse, and touch (virtual joystick, FIRE, and BOMB buttons). Canvas logical size is `BASE_W` × `BASE_H` (480 × 720), scaled by `resize()`.
6. **Game state and logic** — `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `applyPowerUp`, `addScore`/`breakChain` (combo multiplier), `update(dt)`, and stage/boss transitions.
7. **Rendering** — `drawPlane`, `drawBossShip`, and the frame drawing code.
8. **Loop** — `requestAnimationFrame` loop; `dt` is clamped to 0.05 s to avoid large jumps after tab switches.

Weapons: `vulcan` (spread bullets) and a laser weapon. Each has three levels, upgraded through weapon power-ups. Player lives, bombs, and weapon level are tracked as state variables in the script.

## Conventions

- Keep the game in a single self-contained `index.html` unless the user asks for a split. Do not add a build step or npm dependencies without being asked.
- Put new tuning values in the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding magic numbers in update or draw code.
- Respect the reduced-motion setting. Any new screen shake, flash, or pulse effect should go through the existing helpers, which already check `reducedMotion`.
- Keep the UI accessible: announce important state changes (stage start, boss defeated, game over) through `announce()`, and keep new buttons labeled with `title` or visible text.
- Preserve the mobile layout: safe-area handling (`env(safe-area-inset-*)`), `touch-action: none` on the game surface, and the touch control zones.
- Match the existing style: 2-space indentation, single quotes in JavaScript, `const`/`let`, and small named functions.

## Git workflow

- Develop on the feature branch assigned to the session (currently `claude/gifted-tesla-oztc35`). Do not push to other branches without explicit permission.
- Write clear, descriptive commit messages that explain the gameplay or code change.
- Do not open a pull request unless the user explicitly asks for one.

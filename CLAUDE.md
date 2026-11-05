# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up.

The entire game lives in a single file, `index.html` (~1,700 lines): HTML markup, an inline `<style>` block, and one inline `<script>` wrapped in an IIFE. There is no build step, no package manager, no framework, and no test suite.

- `README.md` — placeholder title only.
- `index.html` — the whole game.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests, linters, or build commands. Verify changes by playing the game in a browser (desktop and, where relevant, a mobile-sized viewport).
- Check the browser console for errors after any change to the script.

## Architecture

### Page structure
- `#container` holds `#gameCanvas` (480×720 logical size, scaled to fit the viewport up to 1.6×), the HUD (`#hud`: score, best, multiplier, weapon, lives as hearts, bombs), the top bar (mute, reduced-motion, pause buttons), touch controls (`#touchControls`: joystick zone, FIRE, BOMB), and `#overlay` (title / difficulty / start screen with stats after game over).
- Visual feedback layers: `#vignette` (low-health red edge), `#flash` (hit/bomb flash), `#comboPop`, `#stageBanner`, and a boss health bar (`#bossBarWrap`).
- `#srLive` is an `aria-live` region; use the `announce(text)` helper for screen-reader updates.

### Script layout (inside the IIFE)
The script is organized by section comments and `const`/`let` declarations. Key areas, roughly in file order:

1. **DOM references and storage keys** — `STORAGE_*` constants (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`).
2. **Adaptive performance** — `trackPerf(dt)` samples frame times and moves `quality` between 0 (full), 1 (medium), and 2 (low) to keep frame rate up.
3. **Game data tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six designs cycled by stage).
4. **Audio** — a lazily created `AudioContext` (`ensureAudio`) with `beep` / `noiseBurst` helpers and a `sfx` table. Audio is created only after a user gesture.
5. **HUD rendering and effects** — `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, flash helpers.
6. **Layout** — `resize()` computes canvas scale from `BASE_W`/`BASE_H`.
7. **Input** — keyboard (`keys` map), touch joystick (`joystickStart/Move/End`), fire/bomb touch buttons, and mouse steering/firing on the canvas.
8. **Game state and loop** — `resetGame()`, `update(dt)`, `draw()`, `loop(now)` driven by `requestAnimationFrame`. Shared state (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`, `chain`, `multiplier`, etc.) is module-level inside the IIFE.
9. **Spawning and combat** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `addScore` / chain multiplier (`breakChain`), `useBomb`, `fireWeapon`, `applyPowerUp`.
10. **Boss attack patterns** — `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, driven by `updateBoss` / `drawBossShip`.
11. **Lifecycle** — `killBoss`, `loseLife`, `endGame`, `togglePause`.

### Gameplay model (for reference when changing balance)
- **Difficulty** (`DIFFICULTIES`): easy / normal / hard change spawn cadence, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- **Stages**: advance after `KILLS_PER_STAGE` (18) kills; each stage ends with a boss. Boss HP scales with stage (`70 + stage * 35`, times the difficulty and boss multipliers). Enemy types unlock by `minStage`.
- **Power-ups**: `life` (+1 life, capped), `bomb` (+1 bomb, capped at `maxBombs`), `weapon` (raises weapon level 1→3, then toggles between `vulcan` and `laser`).
- **Weapons**: `vulcan` (spread shots, wider at higher levels) and `laser` (piercing lanes; level 3 pierces 2 targets).
- **Controls**: mouse steers and clicks to fire; arrows/WASD move; Space fires; X or Shift uses a bomb; P or Esc pauses. Touch: drag on the left to fly, tap FIRE, tap BOMB.

### Persistence and accessibility
- Only `localStorage` is used, under the `skyfighter.*` keys above. Changing a key name will reset players' saved best score, mute, difficulty, and motion settings, so treat those keys as a compatibility contract.
- Reduced motion defaults to the `prefers-reduced-motion` media query unless the player has set it explicitly; it gates screen shake, flashes, and pulses via the `reduced-motion` body class.
- Mute, reduced-motion, and pause are exposed as buttons with `title` attributes; keep these accessible when editing the UI.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split files. Do not introduce a build tool, bundler, or npm dependencies without asking.
- Stay in the existing style: `const`/`let`, 2-space indentation, single quotes in JS, section comments with `// ---------- Name ----------`, and data-driven tables for tunable values.
- Put tunable numbers (spawn rates, HP, speeds, drop chances, colors) in the existing data tables rather than scattering magic numbers through logic.
- Preserve mobile behavior: `touch-action: none` on the game surface, safe-area insets in the HUD, and the touch control layout. Test at phone-sized viewports after UI changes.
- Keep the game loop frame-rate independent by using the `dt` passed to `update`.
- Avoid adding `console.log` noise or external network requests (CDNs, analytics) to the game.

## Git workflow

- Develop on the branch specified for the task (currently `claude/gifted-tesla-acoy8c`). Do not push to other branches without permission.
- Write clear, descriptive commit messages in the imperative mood.
- Do not open a pull request unless explicitly asked.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game lives in one self-contained file, `index.html`, which has inline CSS and an inline `<script>`. There is no build step, no package manager, and no external dependencies. `README.md` is a placeholder.

## Running the game

- Open `index.html` directly in a browser. No server is required.
- There are no automated tests, linters, or build scripts. Verify changes by playing the game in a browser. Check at least the affected path on desktop (mouse and keyboard) and, when input or layout changes, on a touch-sized viewport.

## File layout (`index.html`)

- **Lines ~1–360:** `<style>` block. HUD, overlay card, touch controls, and the `.reduced-motion` class.
- **Lines ~361–421:** `<body>` markup. The `#container` holds the `<canvas id="gameCanvas">`, the HUD, the boss bar, the top-bar buttons, the touch controls, and the `#overlay` start/pause/game-over card.
- **Lines ~422–1669:** one `<script>` containing a single IIFE (`'use strict'`). Everything is private to the closure. Nothing is exposed on `window`.

Inside the IIFE, the code is grouped by `// ---------- Section ----------` comments:

1. **DOM refs and constants:** element lookups, `STORAGE_*` keys, `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
2. **Adaptive performance:** `trackPerf` adjusts a `quality` level from average frame time.
3. **Audio:** a WebAudio `sfx` object built from `beep` and `noiseBurst`. `ensureAudio` lazily creates the context on the first user gesture.
4. **HUD and feedback:** `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (for screen readers).
5. **Settings:** mute, reduced motion, and difficulty, each persisted to `localStorage`.
6. **Canvas sizing:** `resize` keeps a fixed 480×720 logical canvas and scales it to fit the window.
7. **Input:** keyboard, mouse, touch joystick, and fire/bomb buttons. All of them write into `keys`, `mouseActive`, `joyVec`, and `firing`.
8. **Game state and rules:** `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `addScore`, `breakChain`, `loseLife`, `endGame`, `togglePause`.
9. **Boss patterns:** `BOSS_PATTERNS` and `updateBoss`. Each pattern is a bullet-pattern function such as `fireRadial`, `fireCross`, or `fireHomingSeeded`.
10. **Rendering:** `draw`, `drawPlane`, `drawBossShip`, `drawClouds`. Rendering is plain 2D canvas calls, with no sprites or image assets.
11. **Main loop:** `loop(now)` clamps `dt` to 0.05 s and runs `update(dt)` followed by `draw()`. It only runs while `running && !paused`. `requestAnimationFrame(loop)` is kicked off at the end of the IIFE.

## Game rules and tuning

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), and `hard`. Each sets spawn timing, enemy speed, enemy shoot chance, starting lives and bombs, and boss HP multiplier. `setDifficulty` only stores the choice, so it takes effect when the next game starts. Lives and bombs are read in `resetGame`. Spawn and enemy values are read each frame in `update`, and boss HP is read at spawn.
- **Stages:** the player advances after `KILLS_PER_STAGE` (18) kills. That triggers a boss, which spawns 1.2 s later. The background theme cycles through `THEMES` by stage.
- **Enemies:** `grunt`, `interceptor` (weaves), and `tank` (4 HP, appears from stage 2). Defined in `ENEMY_TYPES`.
- **Bosses:** `BOSSES[(stage - 1) % BOSSES.length]`. Each has a name, colors, and an `hpMul`, which scales with the difficulty's `bossHpMul`.
- **Power-ups** drop from destroyed enemies (`maybeSpawnPowerUp`): about 6% life, 8% bomb, and 20% weapon. Weapon pickups upgrade the current weapon to level 3. Past level 3 they swap between `vulcan` and `laser`.
- **Scoring:** `addScore` applies a chain multiplier. `breakChain` resets it when the chain times out or the player is hit.

## Persistence

`localStorage` keys, all prefixed `skyfighter.`:

| Key | Values | Purpose |
| --- | --- | --- |
| `skyfighter.bestScore` | number | Best score, written in `endGame` |
| `skyfighter.muted` | `'1'` / `'0'` | Mute toggle |
| `skyfighter.difficulty` | `easy` / `normal` / `hard` | Selected difficulty |
| `skyfighter.reducedMotion` | `'1'` / `'0'` | Overrides the `prefers-reduced-motion` default |

Every access is a bare `localStorage` call with no try/catch. If you add more persistence, follow the same pattern and prefix. Also consider guarding against environments where storage is blocked.

## Conventions

- **Keep it one file.** New code belongs in `index.html`, in the existing IIFE and section structure. Don't add modules, a build step, or external dependencies unless the user asks for them.
- **Match the local style.** Use 2-space indentation, single quotes, `const` and `let` (no `var`), arrow functions for callbacks, and `// ---------- Section ----------` dividers for new sections.
- **Keep the fixed logical resolution.** Game coordinates use `canvas.width` (480) and `canvas.height` (720). Don't hard-code screen pixels, and don't change `BASE_W`/`BASE_H` without updating layout assumptions in both CSS and the touch controls.
- **Respect reduced motion.** Effects that shake the screen or flash should check the `reducedMotion` flag, as the existing ones do.
- **Keep audio gated.** Create audio only through `ensureAudio()`, which runs after a user gesture. Browsers block audio before one.
- **Keep screen-reader output.** Route important game-state changes through `announce()`, as `endGame` does.
- **Keep the controls hint in sync.** If you change or add controls, update both the `#controlsHint` text and the touch controls.

## Working in this repo

- Work on the branch the task names. Commit with clear messages.
- The working tree is a single file, so review diffs carefully. Changes to the game loop or `update` affect everything.
- Playtest any gameplay change. The game has no tests, so the browser is the only check.

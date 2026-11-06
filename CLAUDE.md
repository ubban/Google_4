# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game lives in one file, `index.html`: inline CSS in `<style>`, markup for the HUD and overlays, and one inline `<script>` block containing the game loop, rendering, audio, input, and boss logic. There is no build step, no package manager, no framework, and no test suite.

`README.md` is a placeholder and contains no setup instructions.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no build, lint, or test commands. Verify changes by loading the page and playing: check the title overlay, difficulty selection, weapon and bomb pickups, boss fights, pause (`P`/`Esc`), mute, and game over.
- Check touch controls and the small-screen layout when you change HUD or overlay CSS. The page targets phones as well as desktop, and it sets `user-scalable=no`.

## Architecture

### Coordinate system and scaling
- Logical play field: `BASE_W = 480`, `BASE_H = 720` (portrait). `resize()` scales the canvas to fit the window, capped at `1.6x`.
- Game logic uses the logical coordinates. Keep new positions and sizes in that space.

### Main loop
- `loop(now)` runs on `requestAnimationFrame`, computes `dt`, and calls `update(dt)` then `draw()`.
- `resetGame()` initializes all state (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, score, lives, bombs, stage). Add any new per-run state there.
- `frame` is a global frame counter. `difficultyRamp` is derived from it every 300 frames.

### Stages, enemies, and bosses
- `KILLS_PER_STAGE = 18`. Killing that many enemies in a stage spawns the stage boss.
- `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`) define size, HP, speed, color, score, and `minStage`. `spawnEnemy()` picks types whose `minStage` is at or below the current stage.
- `BOSSES` lists six designs cycled by `(stage - 1) % BOSSES.length`. `BOSS_PATTERNS[i]` holds the attack patterns for boss `i`, and `updateBoss(dt)` cycles through them. Pattern helpers (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) are shared across bosses.
- `THEMES` sets the sky gradient and cloud/sun colors. The theme is `(stage - 1) % THEMES.length`.

### Difficulty
- `DIFFICULTIES` (`easy`, `normal`, `hard`) sets spawn rates, enemy speed, shot chance, starting lives and bombs, and boss HP multiplier. Read values through `DIFFICULTIES[difficulty]`. Don't hard-code them.

### Power-ups and weapons
- `maybeSpawnPowerUp` drops `life`, `bomb`, or `weapon` pickups. `applyPowerUp` handles them.
- `player.level` (1–3) controls the spread of `fireWeapon()`. `player.weapon` switches between the normal gun and `laser` (faster fire delay).
- `useBomb()` clears enemy bullets, damages enemies and the boss, and triggers a screen shake and flash.

### Input
- Keyboard: arrow keys/WASD for movement (`keys[e.code]`), `Space` to fire, `X` or `Shift` for a bomb, `P` or `Esc` to pause.
- Mouse: follows the pointer over the canvas (`mouseTarget`) and fires while pressed.
- Touch: a virtual joystick (`moveZone`, `joystickBase`), a fire button, and a bomb button. `isTouchDevice` gates the touch UI.
- Keyboard listeners are registered on `window`. Touch listeners use `{ passive: false }` where they call `preventDefault`.

### Audio
- Web Audio API only, created lazily by `ensureAudio()`. Browsers require a user gesture, so call `ensureAudio()` from input handlers (as the existing code does), not at load time.
- Sound effects are defined in the `sfx` object using `beep()` and `noiseBurst()`. Mute state is stored and toggled through `muteBtn`.

### Rendering and quality
- Canvas 2D only. `draw()` renders in layers: sky, clouds, particles, enemies, bullets, and HUD-like effects.
- `quality` (0 = full, 1 = medium, 2 = low) is adjusted by `trackPerf(dt)` based on the average frame time. It reduces particle counts (`spawnExplosion`) and disables glow when not at full quality. Keep new effects compatible with this.
- `reducedMotion` (user setting, defaults to the OS `prefers-reduced-motion` value) disables the hit and bomb flashes and damps screen shake (`screenShake` uses 15% of the magnitude and caps the duration at 0.1s). Respect it in any new visual effect that moves the screen or flashes.

### Persistence (localStorage)
Keys are namespaced with `skyfighter.`:
- `skyfighter.bestScore` (`STORAGE_BEST`)
- `skyfighter.muted` (`STORAGE_MUTE`)
- `skyfighter.difficulty` (`STORAGE_DIFF`)
- `skyfighter.reducedMotion` (`STORAGE_REDUCED`)

Do not rename these keys without a migration. Existing players would lose their saved settings and best score.

### Accessibility
- `announce(text)` writes to the `#srLive` `aria-live` region. Use it for important game events (life lost, stage change, game over).
- Overlays use `hidden` attributes and real `<button>` elements. Keep both when adding UI.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. The single-file layout is intentional.
- Match the existing style: 2-space indent, `const` by default, short helper functions, and `SCREAMING_SNAKE_CASE` for config tables.
- Put tunable numbers in the existing config objects (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than scattering magic numbers through the logic.
- Don't add external dependencies, CDN scripts, or build tooling without an explicit request. The game is meant to load instantly from a single file.
- Use `dt`-scaled movement (pixels per second times `dt`) for anything time-based. Frame-count-based timing is only acceptable for the existing pattern timers.

## Git

- Development branch for this work: `claude/gifted-tesla-fjxkg0`.
- Write clear commit messages that describe the gameplay or code change.

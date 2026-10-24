# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is a single self-contained file, `index.html` (about 1,670 lines): inline `<style>`, the markup for the HUD and overlays, and one inline `<script>` that holds all game logic. `README.md` is a placeholder and has no project documentation.

There is no build step, no package manifest (`package.json`), no bundler, no framework, and no test suite. Do not add any of these unless the user asks. Open `index.html` directly in a browser, or serve the directory with any static file server, to run the game.

Assets are generated at runtime. Sound effects come from the WebAudio API (`beep`, `noiseBurst`, and the `sfx` table) and graphics are drawn on a `<canvas>` with the 2D context. Do not add external images, audio files, or CDN scripts without a reason.

## Repository layout

| Path | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, styles, and script |
| `README.md` | Placeholder |

## Code map (`index.html`)

Sections in the `<script>` are marked with `// ---------- Name ----------` comments. Use them to navigate.

- **Adaptive performance** (`trackPerf`, `quality`): samples frame times and lowers rendering quality (0 full, 1 medium, 2 low) when the frame rate drops.
- **Tuning tables**: `DIFFICULTIES` (spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (six sky palettes that cycle by stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses that cycle by stage).
- **Persistence**: `localStorage` keys prefixed `skyfighter.` store the best score, mute state, difficulty, and reduced-motion preference.
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`, `sfx`, `EXPLODE_SFX`. Audio is created lazily on the first user gesture.
- **Canvas sizing**: `resize` scales a fixed `480 × 720` logical playfield (`BASE_W`, `BASE_H`) to fit the window, capped at 1.6×.
- **Input**: keyboard (`keys`), touch joystick and fire/bomb buttons, and mouse steering. Keyboard and joystick take priority over the mouse so they never fight.
- **Game state**: the module-level `let` variables (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `bombCount`, `stage`, `stageKills`, `chain`, `multiplier`, and others).
- **Spawning**: `spawnEnemy`, `spawnBoss`, `spawnExplosion`.
- **Weapons**: `fireWeapon` for the player, and `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing` for boss patterns. Each boss has a pattern set in `updateBoss`, with comments naming them.
- **Simulation and rendering**: `update(dt)` advances the simulation, `draw()` renders the frame, and `loop(now)` drives both with `requestAnimationFrame`. Ship drawing lives in `drawPlane` and `drawBossShip`.
- **End of game**: `endGame()` records the best score and shows the results overlay.

## Game flow

- The player starts with 3 lives and 2 bombs on Normal difficulty. Easy gives 4 lives and 3 bombs, and Hard gives 3 lives and 2 bombs with tougher enemies and bosses.
- Each stage needs `KILLS_PER_STAGE` kills. When the count is reached, a boss is scheduled after a 1.2 s delay. Defeating it advances `stage`, which changes the theme and the next boss.
- Boss index is `(stage - 1) % BOSSES.length`.
- Power-ups (`type: 'life' | 'bomb' | 'weapon'`) drop from enemies. Weapon power-ups raise `player.level`, which the HUD shows.
- Controls: arrow keys or WASD to move, Space to fire, X or Shift for a bomb, P or Escape to pause. Touch devices use the on-screen joystick and buttons.

## Conventions

- **Keep it in one file.** New code goes in the existing `<script>` or `<style>` blocks unless the user asks for a split. Match the style already in the file: `const` and `let`, plain functions, two-space indentation, single quotes in JavaScript, and short section comments.
- **Tune through the tables.** Balance changes belong in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES`, not in scattered literals.
- **Respect reduced motion.** Effects that shake or flash should check the `reducedMotion` flag. The flag also toggles a `reduced-motion` class on `<body>`, and it defaults to the OS `prefers-reduced-motion` setting until the user overrides it.
- **Stay accessible.** Status text goes through `announce()` into the `srLive` live region. Keep the `viewport` meta tag as it is.
- **Mobile first.** Verify changes at phone width and in touch mode. Do not introduce layouts that scroll horizontally.
- **No secrets, no network calls.** The game is fully offline. Do not add analytics, remote fetches, or external dependencies without asking.

## Verification

There are no automated tests or linters. To check a change:

1. Open `index.html` in a desktop browser and play a full stage through to its boss to confirm the loop, scoring, and boss transition still work.
2. Check the difficulty buttons, pause and resume, mute, and the reduced-motion toggle.
3. Check the layout and controls at phone width, using the device emulator or a touch device.
4. Confirm the browser console shows no errors.

If a change touches the game loop, collision code, or boss patterns, play at least one full boss fight before reporting it done.

## Git

- Develop on the branch named in the task. The default working branch for this repository is `claude/gifted-tesla-d6xfcs`.
- Write commits in the imperative mood with a short subject line that names the change (for example, `Add Sky Fighter: ...`).
- Do not open pull requests unless the user asks for one.

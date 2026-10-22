# CLAUDE.md

Guidance for working in this repository.

## What this is

`Google_4` is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The whole game lives in one file, `index.html` (~1,670 lines), with no build step, no dependencies, and no package manager. `README.md` is a placeholder.

## Layout of `index.html`

- `<style>` (lines ~10–359): all CSS, including the HUD, overlays, touch controls, and the `prefers-reduced-motion` handling.
- `<body>` (lines ~361–420): static markup. Key IDs are `#gameCanvas`, `#overlay`, `#startBtn`, `#touchControls`, `#moveZone`, `#fireBtn`, `#bombBtn`, and the HUD elements (`#score`, `#hearts`, `#bombs`, `#weaponName`, `#bossBarWrap`, ...).
- `<script>` (lines ~422–1667): one IIFE holding all game logic. Nothing is exported to `window`.

Inside the script, the sections run roughly in this order:

1. **DOM refs and constants**: `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
2. **Persistence and settings**: best score, mute, difficulty, and reduced motion, all stored in `localStorage` under the `skyfighter.*` keys.
3. **Audio**: a lazily created `AudioContext` (`ensureAudio`), with synthesized SFX built from `beep()` and `noiseBurst()`. The `sfx` table maps event names to sounds.
4. **HUD and feedback helpers**: hearts, bombs, combo pop, stage banner, screen shake, and flash.
5. **Layout and input**: `resize()` scales a fixed 480×720 logical canvas. Keyboard, mouse, and touch input (virtual joystick plus fire and bomb buttons) all feed the same state.
6. **Game state**: `resetGame()` sets up the mutable globals (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`, and so on).
7. **Rendering**: `drawPlane`, `drawBossShip`, `draw()`, and cloud and background helpers.
8. **Spawning and combat**: `spawnEnemy`, `spawnBoss`, `fireWeapon`, the boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`), and `updateBoss`.
9. **Simulation**: `update(dt)` advances everything. It handles collisions with `rectsOverlap`, scoring through `addScore` and the chain/multiplier logic (`breakChain`), power-ups (`applyPowerUp`), and bombs (`useBomb`).
10. **Lifecycle**: `loseLife`, `killBoss`, `endGame`, and `togglePause`.
11. **Main loop**: `loop(now)` clamps `dt` to 50 ms, calls `trackPerf`, runs `update` and `draw` only while `running && !paused`, and reschedules itself with `requestAnimationFrame`.

## Conventions

- **Keep it one file.** Put new CSS in `<style>`, markup in `<body>`, and code in the existing IIFE. Don't add a bundler, a framework, or extra files unless the user asks for them.
- **Match the local style.** The code uses 2-space indentation, single quotes in JS, `const` by default with `let` for mutable state, arrow functions for callbacks, and `camelCase` names. Constants that are tables use `UPPER_SNAKE_CASE`. Comments are sparse and short.
- **Tune with data, not branches.** Enemy stats, boss designs, themes, and difficulty numbers live in the constant tables above the state section. Prefer adding an entry to a table over adding a new `if` path.
- **Respect the time step.** Movement and timers are multiplied by `dt` (seconds). Don't introduce per-frame counters for anything that should be frame-rate independent. `frame` is only used for visual animation phase.
- **Respect the performance tier.** `quality` (0 full, 1 medium, 2 low) is adjusted by `trackPerf` based on frame times. New particle or effect code should check `quality` and reduce or skip work at the lower tiers.
- **Respect reduced motion.** When `reducedMotion` is on, skip or soften screen shake, flashes, and other large motion effects.
- **Keep accessibility hooks working.** Announcements go through `announce()` into the `#srLive` live region. Controls keep their `aria-*` and `hidden` state in sync with the game state.
- **Persist only through the `skyfighter.*` keys.** If you add a setting, add a new `STORAGE_*` constant and wrap reads and writes the way the existing ones do.

## Running and testing

- There is no build, lint, or test tooling. To run the game, open `index.html` in a browser, or serve the directory with any static server, for example `python3 -m http.server` and then visit `/index.html`.
- Verify gameplay changes by playing them. Check at least: start, pause and resume, a boss fight, losing all lives and the game-over overlay, bombs, and the difficulty and mute toggles. Also check the layout at a phone-sized viewport, since touch controls appear there.
- Do a quick syntax check after editing the script. For example, extract the `<script>` body and run `node --check` on it.

## Git

- Development happens on the branch the session specifies, currently `claude/gifted-tesla-k32bg4`. Don't push to `main` directly.
- Write commit messages in the imperative mood, with a short subject line that names the gameplay or file area changed.

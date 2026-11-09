# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser.
The entire game is a single self-contained file, `index.html` (~1,670 lines):
inline `<style>`, inline `<script>` wrapped in one IIFE, no build step, no
dependencies, no external assets. `README.md` is a placeholder.

To play, open `index.html` directly in a browser (or serve the directory with any
static server). There is no package manager, test suite, linter, or CI config in
the repo as of this writing.

## Repository layout

```
index.html   # The whole game: markup, CSS, and JavaScript
README.md    # Placeholder title only
CLAUDE.md    # This file
```

Keep the game in one file unless the user asks for a split. Do not add a build
system, bundler, or framework without being asked.

## Architecture of `index.html`

- **Markup** (`<body>`, ~lines 63–120): `#container` holds the `#gameCanvas`
  canvas, a damage `#vignette`, a `#flash` overlay, the `#hud` (score, best,
  multiplier, weapon, hearts, bombs), the boss bar (`#bossBarWrap`), combo and
  stage banners, top-bar icon buttons (mute, reduced motion, pause), touch
  controls (`#touchControls`), and the start/game-over `#overlay` with difficulty
  buttons (`.diffBtn`).
- **Styles** (~lines 10–420): plain CSS using `:root { color-scheme: dark }`.
  The HUD is responsive via `clamp()` and `env(safe-area-inset-*)`.
- **Script** (~lines 422–1669): a single `(function () { ... })()` IIFE. Nothing
  is exposed on `window` except the event listeners it registers. Keep new code
  inside that IIFE.

Major sections of the script, in source order:

| Area | Notes |
|---|---|
| DOM refs and constants | `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE = 18`, `THEMES`, `ENEMY_TYPES`, `BOSSES` |
| Persistence | `localStorage` for best score, mute, difficulty, reduced motion |
| Performance | `trackPerf()` averages frame times and sets `quality` (0 full, 1 medium, 2 low) |
| Audio | Lazily created `AudioContext` (`ensureAudio`) and `sfx.*` helpers built on `beep()` and `noiseBurst()` |
| Input | `keys[]` for keyboard, joystick for touch, mouse steering via `canvas` events |
| Game state | `resetGame()` initializes the player, arrays of bullets/enemies/power-ups, and stage counters |
| Drawing | `drawPlane`, `drawBossShip`, `draw()`, `drawClouds` |
| Spawning | `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp` |
| Combat | `fireWeapon`, `fireRadial` / `fireAimedSpread` / `fireColumns` / `fireSpiralBurst` / `fireCross` / `fireHomingSeeded` / `fireDoubleRing` (boss patterns), `useBomb`, `loseLife`, `killBoss` |
| Loop | `update(dt)` for simulation, `draw()` for rendering, `loop(now)` driven by `requestAnimationFrame` |
| Flow | `togglePause`, `endGame`, the start button handler |

### Game rules and tuning

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets
  spawn timing, enemy speed, enemy fire chance, starting lives, starting bombs,
  and a boss HP multiplier.
- **Enemies** (`ENEMY_TYPES`): `grunt`, `interceptor` (weaving, fast), and `tank`
  (4 HP, unlocked at stage 2).
- **Stages**: a stage ends after `KILLS_PER_STAGE` kills, then a boss fight
  starts. Each stage cycles the visual `THEMES` and the `BOSSES` list.
- **Weapons**: `vulcan` (spread grows with `player.level`, up to 3) and `laser`.
  Each weapon power-up raises the level from 1 to 3. At level 3, the next weapon
  power-up switches to the other weapon and resets the level to 1.
- **Power-ups**: `life`, `bomb`, and `weapon`.
- **Bombs**: clear enemy bullets and destroy all regular enemies on screen, and
  deal 24 damage to the active boss. Triggered with X or Shift, or the BOMB touch
  button.
- **Score**: enemy kills feed a combo multiplier (`addScore`, `breakChain`).
  The best score persists to `localStorage` under `skyfighter.bestScore`.

### Persisted keys

These `localStorage` keys are user-facing state. Do not rename them without a
migration plan:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

All reads and writes should tolerate storage being unavailable (private mode,
blocked storage). Keep this in mind when adding new persisted state.

## Controls (keep the in-game hint in sync)

- **Keyboard**: arrows or WASD to move, Space to fire, X or Shift for bomb,
  P or Esc to pause.
- **Mouse**: move to steer, click to fire.
- **Touch**: drag on the left to move (virtual joystick), FIRE and BOMB buttons on
  the right.

The text in `#controlsHint` describes these controls. Update it whenever a
control changes.

## Conventions

- **Plain JavaScript (ES2018+ style)**, `const`/`let`, arrow functions, 2-space
  indentation, single quotes in JS, semicolons. Match the surrounding code.
- **No external dependencies.** No CDN scripts, fonts, or libraries. Audio and
  graphics use the Web Audio API and Canvas 2D.
- **Respect reduced motion.** New screen shake, flashes, or pulsing effects
  should go through the existing `reducedMotion` checks (`screenShake`,
  `flashHit`, `flashBomb`, etc.).
- **Accessibility.** Announce important state changes through `announce()`,
  which writes to the `#srLive` live region. Keep buttons labeled.
- **Mobile first.** The layout must work at phone width, with safe-area insets
  and `touch-action: none` on the game surface. Do not introduce page scroll.
- **Performance.** The game loop runs every frame. Avoid allocations in hot
  paths when practical, and respect the `quality` tier when adding effects.

## Working on this repo

- Test changes by loading `index.html` in a browser. There are no automated
  tests, so describe what you verified manually.
- When changing game balance, update the relevant table in `DIFFICULTIES` or
  `ENEMY_TYPES` rather than hard-coding numbers elsewhere.
- Develop on the branch the session names. Do not push to `main` or other
  branches without explicit permission.
- Commit messages should be short, imperative, and descriptive, following the
  existing history (for example, "Add Sky Fighter: Raiden-style vertical
  shoot-em-up game").

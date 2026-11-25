# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game is a single file, `index.html` (~1,670 lines), containing inline CSS, markup, and one inline `<script>` that drives everything on a `<canvas>`.

- `index.html` — the whole game (HTML, CSS, JS).
- `README.md` — a one-line title placeholder; not authoritative.

There is no build step, package manager, bundler, test suite, linter, or CI configuration. Do not invent commands for these.

## Running the game

Open `index.html` directly in a browser. No server is required.

## Code layout inside `index.html`

The inline script is organized roughly in this order:

1. **Config constants** — `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
2. **Persistent state** — loaded from `localStorage` at startup: best score, mute, difficulty, reduced-motion.
3. **Audio** — `ensureAudio()`, `beep()`, `noiseBurst()`, and an `sfx` object. Audio is lazily initialized on the first user gesture.
4. **UI helpers** — HUD rendering (`renderHearts`, `renderBombs`), flashes, combo pops, stage banners, screen shake, `announce()` for screen readers.
5. **Input** — keyboard (`keys` map), touch joystick and fire/bomb buttons, and mouse on the canvas.
6. **Game state & lifecycle** — `resetGame()`, `update(dt)`, `loseLife()`, `endGame()`, `togglePause()`.
7. **Rendering** — `drawPlane()`, `drawBossShip()`, and related drawing helpers.
8. **Spawning & combat** — `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`, `fireWeapon()`, `useBomb()`, `addScore()`, combo chain logic.
9. **Bosses** — `BOSS_PATTERNS` (bullet patterns such as radial, spiral, cross, homing) driven by `updateBoss()`; `killBoss()` advances the stage.

Keep new code in the matching section rather than appending at the bottom.

## Game rules worth knowing

- **Difficulty** (`easy` / `normal` / `hard`) controls spawn rates, enemy speed and fire chance, starting lives and bombs, and boss HP multiplier. Change balance values in `DIFFICULTIES`, not inline.
- **Stages** advance after `KILLS_PER_STAGE` (18) kills, which triggers a boss fight. Stages cycle through `THEMES` and `BOSSES`.
- **Enemies**: `grunt`, `interceptor` (weaves), and `tank` (4 HP, appears from stage 2). Stats live in `ENEMY_TYPES`.
- **Power-ups**: `life`, `bomb`, and `weapon` (player weapon level goes up to 3).
- **Controls**: arrows or WASD to move; Space or hold-to-fire to shoot; X, Shift to bomb; P or Escape to pause. Touch devices use the on-screen joystick and buttons.

## Persistence

Only these `localStorage` keys are used; keep their names stable because players' saved data depends on them:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Every `localStorage` access should tolerate failure (private mode, blocked storage). Wrap new reads and writes in `try/catch` if you add them.

## Conventions

- **Single-file, no dependencies.** Do not add external libraries, CDN scripts, or a build system unless the user explicitly asks. Keep everything inline in `index.html`.
- **Canvas coordinates** use the base resolution `BASE_W = 480`, `BASE_H = 720`; `resize()` scales the canvas to the viewport. Write game logic in base coordinates.
- **Time-based updates**: `update(dt)` takes a delta time. Scale movement and timers by `dt` rather than assuming a fixed frame rate.
- **Mobile-first layout**: the page must work at phone width without horizontal scrolling, and it uses `touch-action: none` on the game surface. Preserve safe-area insets (`env(safe-area-inset-*)`) in HUD positioning.
- **Accessibility**: respect the reduced-motion toggle when adding screen shake, flashes, or other motion effects, and route important status changes through `announce()`.
- **Styling**: dark color scheme (`color-scheme: dark`). Reuse the existing palette in the CSS when adding UI.

## Working in this repo

- Make focused edits; the file is large, so search for the relevant section before editing and avoid reformatting unrelated code.
- Because there are no automated tests, verify gameplay changes by opening `index.html` in a browser (desktop and a phone-sized viewport) and playing through the affected path: start, fire, bomb, pause, life loss, boss, and game over.
- Git: development happens on feature branches named `claude/...`. Commit with clear messages describing the gameplay or code change.

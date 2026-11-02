# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a
single-file HTML5 canvas game. There is no build step, no package manager, and no
test suite.

| File        | Purpose                                                        |
|-------------|----------------------------------------------------------------|
| `index.html` | The entire game: markup, CSS (`<style>`), and JS (`<script>`) |
| `README.md`  | Placeholder readme (currently just a title and a timestamp)   |

Run the game by opening `index.html` in a browser. No server is required.

## Architecture of `index.html`

The file (~1670 lines) has three sections:

1. **`<style>`** (roughly lines 10–359): CSS for the HUD (pill-style top bar,
   heart lives, bomb icons), glassmorphic overlay card, boss health bar, combo
   and stage banners, low-health vignette, hit flash, and touch controls.
   Supports a `.reduced-motion` body class.
2. **`<body>` markup** (roughly lines 360–421): `#container` wraps the
   `#gameCanvas` canvas plus HUD elements (`#score`, `#best`, `#mult`,
   `#weaponName`, `#weaponLvl`, `#hearts`, `#bombs`, `#bossBarWrap`,
   `#comboPop`, `#stageBanner`), touch controls (`#touchControls`, joystick,
   fire and bomb buttons), and the `#overlay` start/game-over/pause card.
3. **`<script>`** (roughly lines 422–1667): a single IIFE with `'use strict'`.
   All game state is module-private inside the IIFE.

### Key sections inside the script

| Area                         | Notes                                                                 |
|------------------------------|-----------------------------------------------------------------------|
| Config constants             | `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES` |
| Persistence                  | `localStorage` keys: `skyfighter.bestScore`, `.muted`, `.difficulty`, `.reducedMotion` |
| Audio                        | Web Audio via `ensureAudio()`, `beep()`, `noiseBurst()`, named SFX in `sfx`/`EXPLODE_SFX` |
| Adaptive quality             | `trackPerf()` samples frame times and steps `quality` 0–2 to reduce effects on slow devices |
| Input                        | Keyboard listeners, touch joystick, fire/bomb buttons, mouse on canvas |
| Game state                   | `resetGame()` initializes player, enemies, bullets, power-ups, boss, etc. |
| Spawning                     | `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`                  |
| Core loop                    | `update(dt)` (logic), `draw()` (rendering), `loop(now)` (rAF driver)  |
| Weapons and bombs            | `fireWeapon()` (Vulcan/Laser, 3 levels), `useBomb()` (clears screen, damages bosses) |
| Boss                         | `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring), `updateBoss()`, `killBoss()` |
| Scoring                      | `addScore()` applies the kill-chain multiplier; `breakChain()` resets it |
| Lifecycle                    | `loseLife()`, `endGame()`, `togglePause()`                            |

### Gameplay facts worth knowing

- **Difficulty** (`easy` / `normal` / `hard`) changes spawn rates, enemy speed,
  enemy shoot chance, starting lives, starting bombs, and boss HP multiplier.
- **Enemies:** `grunt` (1 HP), `interceptor` (fast, weaving, 1 HP, stage 1+),
  `tank` (4 HP, slow, stage 2+).
- **Stages:** after `KILLS_PER_STAGE` (18) kills, a boss is spawned. Defeating it
  increments `stage` (see `killBoss()`), which selects the next `THEMES` entry.
- **Power-ups:** `life`, `bomb`, and `weapon` (upgrades weapon level or toggles
  Vulcan/Laser).

## Conventions

- Keep the game in **one HTML file**. Do not introduce a bundler, framework, or
  external runtime dependency unless the user explicitly asks for it.
- Keep all new code inside the existing IIFE; do not add new globals.
- Match the existing style: 2-space indentation, `const`/`let`, arrow functions
  for callbacks, single quotes in JS, section headers written as
  `// ---------- Name ----------`.
- Use `localStorage` only through the `STORAGE_*` constants so keys stay
  consistent. Wrap new storage access in try/catch if it could be blocked.
- Time-based logic uses the `dt` passed into `update(dt)`; do not rely on frame
  counts for anything that should be frame-rate independent (`frame` exists for
  periodic patterns only).
- Respect `reducedMotion` for new screen shake, flashes, or large particle
  effects, and respect `quality` for expensive draw work.
- Any new on-screen state change should also be announced through `announce()`
  for screen readers.

## Testing and verification

There is no automated test suite. To verify a change:

1. Open `index.html` in a desktop browser and play through at least one full
   stage, including a boss fight.
2. Check touch controls with browser device emulation (joystick, fire, bomb).
3. Confirm the game-over flow updates `skyfighter.bestScore` and the overlay.
4. Toggle mute, difficulty, and reduced motion, then reload to confirm they persist.
5. Check the console for errors.

## Git workflow

- Default branch: `main`. Feature work is developed on `claude/*` branches.
- Write descriptive commit messages in the imperative mood, with a summary line
  followed by bullet points for details, matching the existing history.
- Do not open pull requests unless explicitly asked.

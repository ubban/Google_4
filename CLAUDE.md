# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser.

- The entire game lives in a single self-contained file: `index.html` (~1,670 lines). It contains the HTML, an inline `<style>` block, and an inline `<script>` block. There are no external assets, libraries, or build steps.
- `README.md` is a placeholder and contains no project documentation.
- Git history: an initial commit, then "Add Sky Fighter: Raiden-style vertical shoot-em-up game".
- There is no test suite, linter, package manager, or CI configuration.

## Running the game

No build step is needed. Open `index.html` directly in a browser, or serve the directory with any static file server (for example, `python3 -m http.server` from the repo root) and visit it.

## Code layout inside `index.html`

The script is plain JavaScript with no modules. Major sections, roughly in file order:

| Area | What it covers |
| --- | --- |
| `<style>` (top of file) | HUD, overlays, boss bar, stage banner, vignette, reduced-motion overrides, and responsive/mobile layout |
| Markup (`<body>`) | Canvas, HUD elements (`#hud`, hearts, bombs, weapon and multiplier stats), boss bar, stage banner, start/game-over overlay, mute and reduced-motion buttons, mobile joystick and fire/bomb buttons |
| Constants (~lines 463–516) | `localStorage` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` |
| Audio (~543–680) | Lazily created `AudioContext` (`ensureAudio`), `beep` and `noiseBurst` synthesis, per-enemy explosion sounds |
| UI helpers (~680–750) | HUD rendering, flashes, combo pop, stage banner, screen shake, mute/difficulty setters |
| Input and layout (~753–865) | `resize()` (fixed 480×720 base, scaled to fit), keyboard handler, touch joystick, mouse steering and firing |
| Game state and flow (~865–1150) | `resetGame`, drawing of player/boss ships, spawning (enemies, bosses, power-ups), scoring and multiplier chain, bombs, `fireWeapon` |
| `update(dt)` (~1148–1305) | Per-frame simulation: player movement and firing, enemies, bullets, collisions, power-up pickups |
| Bosses (~1304–1490) | `applyPowerUp`, boss projectile patterns (`BOSS_PATTERNS`), `updateBoss`, `killBoss`, `loseLife`, `endGame`, pause |
| Rendering (~1507–1640) | `draw()` for background clouds, sprites, HUD-adjacent effects |
| Main loop (~1640 onward) | `loop(now)` with delta-time handling and adaptive `quality` levels driven by `trackPerf` |

Keep the single-file structure unless the user asks for a split. If you refactor, keep the file runnable by opening it directly.

## Game mechanics reference

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn rates, enemy speed, enemy shoot chance, starting lives, starting bombs, and boss HP multiplier.
- **Enemies** (`ENEMY_TYPES`): `grunt` (stage 1+), `interceptor` (stage 1+, weaving), `tank` (stage 2+, 4 HP).
- **Stages**: advance every `KILLS_PER_STAGE` (18) kills. Each stage has a boss from `BOSSES`, shown in the boss bar.
- **Power-ups**: `life`, `bomb`, and `weapon` (toggles between `vulcan` and `laser`).
- **Scoring**: a kill chain builds a multiplier (`chain`, `chainTimer`, `multiplier`); breaking the chain resets it.
- **Lives and bombs**: lives come from `maxLives`; smart bombs are `bombCount` / `maxBombs`.

## Controls

- Mouse: move to steer, hold click to fire.
- Keyboard: Arrow keys or WASD to move, Space to fire, X or Shift for bomb, P or Escape to pause.
- Touch: drag on the left zone for the virtual joystick, tap FIRE, tap BOMB.

## Persistence

The game stores these keys in `localStorage`. Read and write them through the existing constants, and keep the names stable so saved player data keeps working:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

## Conventions for changes

- Keep the code dependency-free: vanilla HTML, CSS, and JavaScript only, with no CDN links or bundlers, unless the user asks otherwise.
- Match the existing style: 2-space indentation, `const`/`let`, single quotes in JavaScript, and small named functions.
- Use the existing constants and tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`) when adding content, rather than hard-coding values in the update loop.
- Respect the `reducedMotion` setting for any new screen shake, flashes, or pulsing effects, and the `muted` setting for any new sounds.
- Keep the mobile layout working: the game uses `touch-action: none`, safe-area insets, and a canvas scaled to the viewport.
- The game is a fixed 480×720 logical canvas. Coordinates and sizes in game logic should use that space.

## Verifying changes

There are no automated tests. To verify a change:

1. Open `index.html` in a browser (or serve it locally) and check the console for errors.
2. Play through at least one stage on each difficulty you touched, including a boss fight if the change affects combat, bosses, or power-ups.
3. Check both keyboard and touch controls if input code changed, and the mute and reduced-motion toggles if audio or effects changed.

## Git workflow

- Default development branch for this session: `claude/gifted-tesla-3m5fkg`.
- Commit only when asked. Do not open pull requests unless the user explicitly requests one.

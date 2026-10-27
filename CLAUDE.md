# CLAUDE.md

Guidance for working in this repository.

## What this is

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs entirely in the browser on an HTML5 `<canvas>`. Players switch weapons, collect power-ups, drop smart bombs, and defeat a boss at the end of each stage.

## Repository layout

| Path         | Purpose                                                                 |
|--------------|-------------------------------------------------------------------------|
| `index.html` | The entire game: inline `<style>`, markup, and one `<script>` IIFE.     |
| `README.md`  | Placeholder.                                                            |

There is no build step, package manifest, test suite, linter, or external dependency. Audio is synthesized with the Web Audio API, and there are no image or sound assets. Keep it that way unless the user asks for a change in structure.

## Running it

Open `index.html` directly in a browser. No server is needed.

## Verifying changes

There are no automated tests. After a change, load the page in a browser and check the paths you touched. Useful checks:

- **Desktop input:** keyboard (arrows/WASD, Space, X/Shift for bomb, P/Esc for pause) and mouse steering/firing.
- **Touch input:** joystick, FIRE, and BOMB. Touch controls appear only on touch-capable devices.
- **Resize:** the canvas is scaled to fit the window (see below), so check a narrow/phone viewport for horizontal scroll or clipped HUD.
- **Game flow:** start, all three difficulties, pause/resume, death and game over, stage transition at 18 kills, and boss fight.

## Code structure (inside `index.html`)

The script is wrapped in an IIFE. Its sections are marked with `// ---------- Name ----------` banners, in this order:

1. **Adaptive performance**: `trackPerf` samples frame time and steps a `quality` level (0–2) up or down.
2. **Audio**: lazy `ensureAudio()` creates the `AudioContext` on the first user gesture; `beep`, `noiseBurst`, and the `sfx` table build every sound.
3. **Canvas sizing**: `resize()` keeps the logical canvas at `BASE_W × BASE_H` (480 × 720) and scales it for display, capped at 1.6×.
4. **Input**: keyboard, touch joystick/buttons, and mouse steering. Keyboard and joystick take priority over the mouse when active.
5. **Game state**: `resetGame()` initializes everything. Entities are plain objects in arrays (`bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`) plus a single `boss`.
6. **Drawing**: `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`.
7. **Spawning and combat**: `spawnEnemy`, `spawnBoss`, `useBomb`, `fireWeapon`, `update(dt)`, the boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireCross`, etc.), and `updateBoss`.
8. **Loop**: `loop(now)` caps `dt` at 0.05 s, calls `update` then `draw` while running and not paused, and re-queues itself with `requestAnimationFrame`.

## Game rules and tuning

Keep these in mind when changing balance. They live as constants near the top of the script:

- **Difficulty** (`DIFFICULTIES`): sets spawn timing, enemy speed, enemy fire chance, starting lives and bombs, and a boss HP multiplier. The choice is saved to `localStorage`.
- **Stages**: a stage ends after `KILLS_PER_STAGE` (18) kills. A boss spawns 1.2 s later. Each stage index cycles through `BOSSES` (6 designs) and `THEMES` (6 sky palettes) using `% length`.
- **Boss HP**: `70 + stage × 35`, scaled by the difficulty multiplier and the boss design's `hpMul`.
- **Bombs**: clear all enemy bullets and enemies (+5 score each) and deal 24 damage to the boss.
- **Scoring chain**: the multiplier is `1 + min(9, floor(chain / 5))`, so it tops out at ×10. Getting hit breaks the chain.
- **Invulnerability**: 1.5 s after losing a life.
- **Power-up drops** (`maybeSpawnPowerUp`): life 6%, bomb 8%, weapon upgrade 20%.
- **Weapons**: Vulcan (spread shots) and Laser (piercing lanes). Each has levels 1–3, and the level sets how many bullets or lanes fire.

## Persistence and accessibility

- **`localStorage` keys** (all prefixed `skyfighter.`): `bestScore`, `muted`, `difficulty`, `reducedMotion`. The existing code reads and writes these directly without guards, so keep new keys under the same prefix.
- **Reduced motion** is on if the OS requests it (`prefers-reduced-motion`) unless the player has toggled it. When on, `screenShake` is scaled down and capped, and CSS flash/pulse effects are suppressed via the `reduced-motion` body class.
- **Screen readers**: use `announce(text)` to push status changes (stage changes, boss defeat) into the `#srLive` region.

## Conventions

- Match the existing style: 2-space indentation, single quotes in JS, `const` for tables, and section banners for new blocks.
- Add new visuals and sounds procedurally. Don't introduce image, audio, or font files, or CDN scripts, without asking.
- Keep changes inside `index.html` unless there is a clear reason to split it out.
- Respect the canvas coordinate space: positions and sizes are in the 480 × 720 logical units, not CSS pixels.

## Git

- Develop on the branch the session specifies, and push with `git push -u origin <branch>`.
- Write commit messages in the imperative mood with a short summary line, such as `Add Sky Fighter: ...` in the existing history.

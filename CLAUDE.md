# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-file browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up.

- `index.html` — the entire game: inline CSS, markup, and one inline `<script>` IIFE. No build step, no bundler, no dependencies, no tests.
- `README.md` — minimal placeholder.

To run it, open `index.html` in a browser (or serve the directory statically). Nothing needs installing.

## Repository layout

```
index.html   # game: <style> (~lines 1–361), markup (~362–421), <script> (~422–1668)
README.md
```

Inside the script, the code is wrapped in an IIFE and organized roughly as:

| Area | Key identifiers |
|---|---|
| DOM refs | `canvas`, `overlay`, `hudEls`, touch controls (`moveZone`, `fireBtn`, `bombBtn`) |
| Persistence keys | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (`skyfighter.*` in `localStorage`) |
| Tuning tables | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `sfx` (WebAudio, synthesized — no audio files) |
| Input | `keys` (keyboard), joystick + touch handlers, `firing`, `canvasPointFromEvent` |
| Simulation | `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause` |
| Rendering | `draw()`, `drawPlane`, `drawBossShip`, `drawClouds` |
| Loop | `loop(now)` driven by `requestAnimationFrame`; `trackPerf` adjusts `quality` (0 full / 1 medium / 2 low) |

The game renders to a fixed logical canvas of **480×720** (`BASE_W`, `BASE_H`) and `resize()` scales it to fit the window (max scale 1.6).

## Game rules and tuning (current values)

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn rate, enemy speed, enemy fire chance, starting lives/bombs, and boss HP multiplier.
- **Stages**: after `KILLS_PER_STAGE` (18) kills in a stage, a boss warning fires and the boss spawns (`spawnBoss`). Stage themes cycle through `THEMES` (6 palettes). Enemy `tank` requires stage 2+ (`minStage`).
- **Enemies**: `grunt`, `interceptor` (weaving), `tank` (4 HP).
- **Bosses**: `BOSSES` has 6 entries (VANGUARD → TITAN), spawned in order; each has its own HP multiplier and attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
- **Power-ups** (`maybeSpawnPowerUp`): `life` (6%), `bomb` (8%), `weapon` (20%). Weapon power-ups raise weapon level to 3, then toggle between `vulcan` and `laser`.
- **Scoring**: kill points plus a combo chain (`addScore`, `breakChain`) and a multiplier shown in HUD.
- **Bombs**: triggered by `X`, `ShiftLeft`, `ShiftRight`, or the touch bomb button (`useBomb`).

## Controls

| Input | Action |
|---|---|
| Arrow keys / WASD | Move |
| Space (or hold fire button) | Fire |
| X / Shift | Bomb |
| P / Escape | Pause |
| Touch: left zone joystick | Move |
| Touch: fire / bomb buttons | Fire / Bomb |

Player preferences (best score, mute, difficulty, reduced motion) persist in `localStorage` under `skyfighter.*`. Reduced motion defaults to the `prefers-reduced-motion` media query unless the user has set it.

## Conventions

- Keep the game **dependency-free and single-file** unless the user asks otherwise. Do not add a bundler, framework, or package manager without discussion.
- Tuning values live in the constant tables near the top of the script. Prefer changing those over hard-coding numbers in the simulation.
- Gameplay is frame-rate independent via `dt` passed to `update(dt)`; keep new timers in seconds/`dt`-scaled units.
- Respect the accessibility hooks: announcements go through `announce()` (`#srLive`), and visual effects like screen shake and flashes should honor `reducedMotion`.
- Keep the layout working on phones: the HUD and touch controls are sized with `clamp()`/`vw`, and `touch-action: none` is set on the page. Preserve `viewport-fit=cover` and safe-area insets.
- Keep the code style consistent with the existing file: 2-space indent, single quotes in JS, `const`/`let`, semicolons at line ends.

## Working on this repo

- There is no test suite or linter. Verify changes by loading `index.html` in a browser and playing through: start, fire, bomb, pause, a stage transition, a boss fight, game over, and the difficulty and mute toggles.
- When changing gameplay numbers, check the HUD (`#hud`, `#hearts`, `#bombs`) and difficulty sections still read correctly.
- Git: develop on the branch assigned for the session and commit with clear messages. Do not open pull requests unless asked.

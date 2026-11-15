# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game lives in one file:

- `index.html` (~1,670 lines): inline `<style>`, markup for the HUD/overlays/touch controls, and one inline `<script>` (starts around line 422) containing all game logic, rendering, audio, and input.
- `README.md`: a one-line title. Nothing else.

There is no build step, no package manager, no bundler, no tests, and no linter configuration. Git history has two commits: `Initial commit` and `Add Sky Fighter: Raiden-style vertical shoot-em-up game`.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by running the game in a browser and playing through the affected path (menu → difficulty → play → pause/resume → game over). For headless checks, Playwright with the pre-installed Chromium works; do not run `playwright install`.
- Check both desktop (keyboard + mouse) and touch layouts when changing HUD, overlay, or control code. The canvas scales to fit the viewport.

## Code layout (inside `index.html`)

Roughly in file order:

| Area | Approx. lines | Notes |
|---|---|---|
| CSS | 1–421 | HUD pills, hearts/bombs, vignette, overlays, touch controls, reduced-motion styles |
| DOM refs | ~426–461 | `getElementById` lookups; all element IDs are referenced here |
| Storage keys | ~463–466 | `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` |
| Accessibility | ~468 | `announce()` writes to the `srLive` live region |
| Perf scaling | ~474–483 | `trackPerf()` moves `quality` (0 full / 1 medium / 2 low) based on average frame time |
| Tuning tables | ~485–515 | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` |
| Audio | ~543–678 | Lazily created `AudioContext` (`ensureAudio`), `beep`/`noiseBurst` primitives, `sfx` presets |
| HUD helpers | ~680–752 | `renderHearts`, `renderBombs`, flashes, combo/stage banners, screen shake, `setDifficulty` |
| Canvas sizing | ~753–766 | Fixed logical size `BASE_W=480 × BASE_H=720`, scaled by `resize()` (capped at 1.6×) |
| Input | ~768–865 | Keyboard, joystick + fire/bomb touch buttons, mouse targeting |
| Game state | ~875–912 | `resetGame()` |
| Drawing | ~913–1072, ~1507–1639 | `drawPlane`, `drawBossShip`, `draw()`, `drawClouds` |
| Spawning | ~1022–1072 | `spawnEnemy`, `spawnBoss` (boss = `BOSSES[(stage-1) % BOSSES.length]`) |
| Weapons & bombs | ~1103–1147 | `fireWeapon` (vulcan/laser), `useBomb` |
| Update | ~1148–1323 | `update(dt)`: movement, firing, collisions, scoring, stage progression |
| Boss patterns | ~1324–1410 | `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing` |
| Boss update | ~1411–1506 | `updateBoss(dt)`, stage advance (`stage++` near line 1449) |
| Game flow | ~1469–1505 | `endGame()`, `togglePause()` |
| Main loop | ~1640–1665 | `loop(now)` driven by `requestAnimationFrame` |

### Game rules worth knowing

- **Difficulty** (`easy` / `normal` / `hard`) sets spawn rate, enemy speed, enemy fire chance, lives, starting bombs, and a boss HP multiplier. Changing a number there changes balance everywhere.
- **Stages**: each stage requires `KILLS_PER_STAGE` (18) kills, then a boss spawns after a short delay. Bosses cycle through the `BOSSES` array. The theme (`THEMES`) cycles with stage.
- **Enemies** are `grunt`, `interceptor` (fast, weaving), and `tank` (multi-hit, appears from stage 2) per `ENEMY_TYPES`.
- **Weapons**: the player toggles between `vulcan` and `laser` (`player.weapon`).
- **Persistence** is `localStorage` only, and every access must stay wrapped so the game still works when storage is blocked or throws. The only persisted values are the four `skyfighter.*` keys above.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split files. If you do split, update this document.
- Plain JavaScript, ES2017-style, no modules, no transpilation. Match the existing style: 2-space indent, `const`/`let`, single quotes, semicolons.
- Tuning values live in the constant tables near the top of the script. Prefer editing those over hard-coding magic numbers in update/draw code.
- Preserve accessibility hooks: announcements go through `announce()`, and motion effects respect `reducedMotion`.
- Preserve mobile behavior: `touch-action: none` on the canvas and the touch control layer are intentional.
- Keep the game playable at phone width; the canvas is scaled with `resize()`, not via page scrolling (`overflow: hidden` on `html, body` is deliberate).

## Git and delivery

- Default working branch for this session: `claude/gifted-tesla-tzi2wz`. Commit and push there; do not push to other branches without explicit permission.
- Do not open pull requests unless asked.
- Keep commits focused. Describe gameplay changes in terms of player-visible behavior (difficulty, stages, controls).

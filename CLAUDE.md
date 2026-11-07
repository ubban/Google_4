# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` (~1,670 lines).
- `README.md` — a one-line title; there is no other documentation.
- No build system, package manager, bundler, test suite, or CI config exists.

## Running and testing

- There is nothing to install or compile. Open `index.html` directly in a browser, or serve the
  directory with any static server (for example `python3 -m http.server` from the repo root).
- There are no automated tests or linters. Verify changes by playing the game in a browser.
  The most recent commit was checked with a headless Playwright run covering firing, collisions,
  pickups, boss patterns, and stage advance, with no console errors. Do the same for gameplay changes.
- Keep it dependency-free. Do not add npm packages, CDN scripts, or a build step unless the user asks.

## Code layout inside `index.html`

The script is one IIFE (`'use strict'`) with no modules. Sections, roughly in file order:

1. **DOM refs and constants** — element lookups, `STORAGE_*` localStorage keys, `DIFFICULTIES`,
   `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
2. **Audio** — `ensureAudio`, `beep`, `noiseBurst`, plus named SFX. Audio is created lazily on the
   first user gesture because browsers block autoplay.
3. **HUD and FX helpers** — hearts/bombs rendering, hit flash, combo pop, stage banner, screen shake.
4. **Layout and input** — `resize()` scales a fixed `BASE_W × BASE_H` (480×720) logical canvas.
   Keyboard state is kept in `keys[e.code]`; mouse, touch joystick, and on-screen FIRE/BOMB buttons
   are also wired here.
5. **Game state** — `resetGame()` initialises the run. Entities are plain objects in arrays.
6. **Drawing** — `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`, which renders each frame.
7. **Spawning and combat** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`,
   `addScore` / kill-chain multiplier (`breakChain`).
8. **Bosses** — `BOSS_PATTERNS` is an array of bullet-pattern functions (radial, spiral, cross,
   homing, and so on). `updateBoss` picks among them; `killBoss` advances the stage.
9. **Update and lifecycle** — `update(dt)`, `applyPowerUp`, `loseLife`, `endGame`, `togglePause`.
10. **Main loop** — `loop(now)` clamps `dt` to 50 ms, updates adaptive quality, then calls `update` and `draw`.

### Conventions to keep

- **Time-based motion.** Movement and timers use `dt` (seconds). Do not tie gameplay speed to frame count.
- **Adaptive quality.** `trackPerf` lowers `quality` (0 full, 1 medium, 2 low) when frame time is high.
  Visual effects should respect `quality`, so the game stays playable on weak devices.
- **Reduced motion.** `reducedMotion` (from the OS preference or the user's saved setting) disables
  screen shake and similar effects. New motion effects should check it.
- **Persistence.** Only four keys are stored in `localStorage`, all prefixed `skyfighter.`:
  `bestScore`, `muted`, `difficulty`, `reducedMotion`. Wrap any new storage use in try/catch, since
  storage can be blocked in private windows.
- **Accessibility.** Announcements go through `announce()` into the `#srLive` live region.
  Keep the `controlsHint` text in sync when controls change.
- **Single file.** Keep CSS in the `<style>` block and JS in the single `<script>`. Do not split the
  game into multiple files unless the user asks.

### Balancing knobs

Most tuning is in data objects near the top of the script:

- `DIFFICULTIES` — spawn rate, enemy speed, enemy fire chance, starting lives and bombs, boss HP multiplier.
- `ENEMY_TYPES` — size, HP, speed multiplier, score, and the stage at which each type first appears.
- `BOSSES` — boss names, colours, and HP multiplier. Bosses cycle by stage.
- `KILLS_PER_STAGE` — kills needed to trigger a boss.

Change these tables before changing logic.

## Controls (keep in sync with the in-game hint)

- Desktop: mouse to steer and click to fire, or Arrows/WASD to move. Space fires. X or Shift drops a bomb.
  P or Esc pauses.
- Mobile: drag the left zone to fly, tap FIRE, tap BOMB.

## Git and workflow

- Develop on the branch the session specifies. Do not push to `main` or any other branch without
  explicit permission.
- Write clear commit messages that say what changed and why.
- Do not open a pull request unless the user asks for one.
- Before finishing gameplay work, run the game in a browser and check the console for errors.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style
vertical shoot-'em-up. The entire game lives in a single self-contained file,
`index.html` (inline `<style>` and one inline `<script>`, drawn on a `<canvas>`).

- `index.html` — the whole game: markup, CSS, audio, rendering, input, game loop.
- `README.md` — minimal project readme.

There is no build step, no package manager, no test suite, and no bundler. Open
`index.html` in a browser to run it.

## Running and verifying changes

- Run locally: open `index.html` directly, or serve the folder with any static
  server (e.g. `python3 -m http.server` from the repo root, then visit
  `http://localhost:8000`).
- There are no automated tests or linters configured. After changes, verify by
  loading the game in a browser and playing through the affected path. For
  gameplay changes, check both keyboard and touch input where relevant.
- Keep the file loadable without network access. Do not add external CDN
  scripts, fonts, or stylesheets unless the user asks for them.

## Code layout inside `index.html`

The script is plain, framework-free JavaScript organized roughly as:

1. **Persistence keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`,
   `STORAGE_REDUCED` (all prefixed `skyfighter.`) read/write `localStorage`.
   Wrap any new storage access in the same defensive pattern used today.
2. **Accessibility/announcements** — `announce()` writes to the `#srLive`
   polite live region; `reducedMotion` (user setting, defaulting to the OS
   `prefers-reduced-motion` value) toggles a `reduced-motion` body class. Respect
   it for any new motion or flash effects.
3. **Performance** — `trackPerf()` adjusts a `quality` level (0 full, 1 medium,
   2 low) based on frame timing.
4. **Tuning tables** — `DIFFICULTIES`, `THEMES` (stage backdrops),
   `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled per
   stage), `KILLS_PER_STAGE`. Prefer changing balance here rather than in logic.
5. **Audio** — Web Audio via `ensureAudio()`, `beep()`, `noiseBurst()`. Audio
   is created lazily on the first user gesture; keep it that way.
6. **HUD** — `renderHearts`, `renderBombs`, combo/stage banners, screen shake.
7. **Input** — keyboard (`keys[e.code]`), a virtual joystick and fire/bomb
   buttons for touch, and mouse/pointer control on the canvas. `resize()`
   scales the canvas to a fixed logical size (`BASE_W` × `BASE_H` = 480 × 720).
8. **Game state and update** — `resetGame()`, `update(dt)`, `spawnEnemy()`,
   `spawnBoss()`, `maybeSpawnPowerUp()`, `applyPowerUp()`, `useBomb()`,
   `fireWeapon()`, and the boss attack patterns (`fireRadial`, `fireCross`,
   `fireSpiralBurst`, etc.).
9. **Rendering** — `drawPlane()`, `drawBossShip()`, and the background/particle
   drawing, all on the 2D canvas context.
10. **Main loop** — `loop(now)` driven by `requestAnimationFrame`, using `dt`.

### Controls (for reference when changing input)

- Move: arrow keys or WASD; touch joystick; mouse on canvas.
- Fire: Space (held) or the fire button.
- Bomb: X or Shift, or the bomb button.
- Pause: P or Escape.
- Weapons toggle between `vulcan` and `laser` via a power-up.

## Conventions

- Keep everything in `index.html` unless the user asks to split the project.
  Do not introduce a build step, framework, or package manager without a request.
- Match the existing style: 2-space indentation, `const`/`let`, single quotes in
  JS, small named functions, and constants in UPPER_SNAKE_CASE.
- Gameplay numbers belong in the tuning tables above, not hard-coded in update
  or draw logic.
- Preserve the touch and keyboard paths together. Any new control must work on
  both, and `touch-action`/`user-select` settings should not be loosened.
- Preserve the `resize()` logical-size scaling so the game looks right on phone
  and desktop viewports (the viewport meta disables user zoom on purpose).

## Git workflow

- Develop on the branch specified for the task (currently
  `claude/gifted-tesla-juj3gz`). Do not push to other branches without permission.
- Commit with clear, descriptive messages and push with `git push -u origin <branch>`.
- Do not open pull requests unless explicitly asked.

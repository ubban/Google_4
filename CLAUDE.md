# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only game is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up that runs in the browser.

- `index.html` — the entire game: markup, inline `<style>`, and one inline
  `<script>` (an IIFE). There is no build step, no bundler, and no dependencies.
- `README.md` — a one-line title. Keep it short.

There is no package.json, test suite, linter, or CI configuration in the repo.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any
  static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser:
  start a run, check the affected behavior, and confirm nothing else regressed.
  Test on both desktop (keyboard/mouse) and touch-sized viewports when the change
  touches input, HUD, or layout.

## Architecture of `index.html`

Layout of the file (approximate line ranges, will drift as the file changes):

| Section | What it holds |
| --- | --- |
| `<style>` (~10–359) | Dark-only styling (`color-scheme: dark`), HUD, overlay, touch controls |
| Markup (~361–421) | `#container` with `#gameCanvas`, `#hud`, `#bossBarWrap`, `#topBar`, `#touchControls`, `#overlay` |
| Script (~422–1667) | Single IIFE containing all game logic |

Key pieces inside the script, in roughly the order they appear:

- **Persistence** — `localStorage` keys prefixed `skyfighter.`:
  `bestScore`, `muted`, `difficulty`, `reducedMotion`. Every read and write
  should stay wrapped so a missing or blocked storage does not break the game.
- **Difficulty** — `DIFFICULTIES` (`easy` / `normal` / `hard`) sets spawn
  timing, enemy speed, enemy shoot chance, starting lives and bombs, and boss HP
  multiplier. `KILLS_PER_STAGE` (18) controls stage progression.
- **Stage themes** — `THEMES` supplies sky gradients, cloud and sun colors, and
  a name per stage. Stages cycle through the list.
- **Enemies** — `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank`
  (4 HP, appears from stage 2).
- **Bosses** — `BOSSES` (name, colors, `hpMul`) paired with `BOSS_PATTERNS`,
  an array of attack-pattern functions (radial, aimed spread, columns, spiral,
  cross, homing, double ring) built from the `fire*` helpers.
- **Player and weapons** — `fireWeapon()` and `applyPowerUp()`. Weapon types are
  `vulcan` and `laser`, each with levels 1–3. Pickups are `life`, `bomb`, and
  `weapon`. Extra weapon pickups past level 3 toggle to the other weapon type.
- **Scoring** — `addScore()` applies a chain multiplier; `breakChain()` resets it.
- **Main loop** — `update(dt)` advances state, `draw()` renders, `loop(now)`
  drives both via `requestAnimationFrame`.
- **Adaptive quality** — `trackPerf()` steps `quality` (0 full, 1 medium, 2 low)
  based on frame time. Shadow blur, particle counts, and glow check `quality`.
  Keep new visual effects consistent with this so they degrade on slow devices.
- **Input** — keyboard (arrows/WASD move, Space fire, X or Shift bomb, P/Esc
  pause), mouse (move to steer, click to fire), and touch (left-side joystick,
  FIRE and BOMB buttons). Touch listeners use `passive: false` where they call
  `preventDefault()`.
- **Accessibility** — `announce()` writes to the `#srLive` polite live region.
  A reduced-motion toggle (`#motionBtn`) gates screen shake, flashes, and pulses.
- **Audio** — Web Audio API, created lazily by `ensureAudio()` on the first user
  gesture. Browsers block audio before a gesture, so do not create the context
  earlier. Sound effects are built from `beep()` and `noiseBurst()`.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it.
  Splitting into multiple files would change how the project is served, so
  ask before doing it.
- Match the existing style: 2-space indentation, `const`/`let`, single quotes in
  JS, and the same naming pattern for constants (`UPPER_SNAKE` for config
  tables, `camelCase` for functions and state).
- Add new tunable numbers to the existing config tables (`DIFFICULTIES`,
  `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Do not add external scripts, fonts, or CDN links without a reason. The game
  currently ships fully self-contained.
- Keep the layout working at phone width and on landscape orientation. The
  canvas scales through `resize()` using `BASE_W` × `BASE_H` (480 × 720).

## Git workflow

- Develop on the branch assigned for the task. Do not push to other branches.
- Write commit messages that say what changed and why, in the imperative mood
  (e.g. "Add boss pattern for WRAITH", "Fix HUD overflow on narrow screens").
- Do not open a pull request unless explicitly asked.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small static web project. Its only real content is **Sky Fighter**, a Raiden-style
vertical shoot-'em-up that runs in a browser canvas.

- `index.html` — the entire game: inline CSS (`<style>`), markup, and one inline
  `<script>` block (an IIFE). No external assets, libraries, or network calls.
- `README.md` — placeholder; contains a stray timestamp line, not documentation.

There is no `package.json`, bundler, framework, test suite, linter, or CI config.

## Running and verifying

- Run it by opening `index.html` in a browser, or serve the directory with any static
  server (e.g. `python3 -m http.server`) and visit it. No install step is needed.
- No automated tests or build exist. When changing gameplay or UI, verify by playing
  the game in a browser (keyboard, and touch controls via device emulation if the change
  touches the touch UI). Say so explicitly if you could not do this.
- Do not invent build, lint, or test commands. If you add tooling, document the
  commands here at the same time.

## Code layout inside `index.html`

The script is organized into commented sections; keep new code in the matching section:

- **Setup / DOM refs** — `getElementById` lookups at the top of the IIFE.
- **Storage keys** — `STORAGE_*` constants (`skyfighter.*` prefix) for persisted settings.
- **Adaptive performance** — `trackPerf` lowers the `quality` level (0/1/2) when frame
  time is high; drawing code reads `quality`.
- **Tuning tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes),
  `ENEMY_TYPES`, `BOSSES`. Balance changes usually belong here, not in the logic.
- **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object. Audio is created
  lazily on the first user gesture; keep it that way (browsers block autoplay).
- **Canvas sizing** — fixed logical resolution `BASE_W × BASE_H` (480 × 720), scaled to
  fit the window via CSS. Draw in logical coordinates only.
- **Input** — keyboard (`keys` map keyed by `e.code`), touch joystick and buttons, and
  mouse/pointer aiming on the canvas.
- **Game state and logic** — `resetGame`, `update(dt)`, spawning, collisions, power-ups,
  bombs, weapons (`fireWeapon`), and the boss pattern table `BOSS_PATTERNS`.
- **Drawing** — `draw()` and helpers (`drawPlane`, `drawBossShip`, `drawClouds`).
- **Loop** — `loop(now)` runs `update` then `draw` each frame with `dt` clamped to 50 ms.

## Conventions

- Plain ES2017+ JavaScript, no modules, no build step. Keep it that way unless the user
  asks for a change in architecture.
- Keep everything in `index.html` unless the file becomes unmanageable; if you split it,
  update this file and the layout section above.
- Use `const` by default; the existing code uses `let` for mutable game state.
- Match the existing naming: `UPPER_CASE` for tuning constants and storage keys,
  `camelCase` for functions and state.
- Persisted values go through `localStorage` with the `skyfighter.` key prefix and must
  tolerate missing or malformed values (the code falls back to defaults).
- Accessibility features already exist and should not regress: the `srLive` live region
  (via `announce`), the reduced-motion toggle (`prefers-reduced-motion` respected), and
  keyboard controls. Keep new HUD text and alerts announced where appropriate.
- Respect the mobile layout: viewport meta disables user scaling, and touch controls are
  shown for touch devices. Check a phone-sized viewport after UI changes.

## Gameplay quick reference

- Move: arrow keys or WASD; touch joystick on mobile.
- Fire: Space (hold), or the fire button.
- Bomb: X or Shift; or the bomb button. Bomb count depends on difficulty.
- Pause: P or Escape.
- Difficulty (`easy` / `normal` / `hard`) cannot be changed during active play; it can be
  changed from the menu or while paused (`setDifficulty` guards on `running && !paused`).
- Stages are endless. Each stage ends after `KILLS_PER_STAGE` (18) kills and a boss fight.
  Stage N uses `THEMES[(N-1) % 6]` and `BOSSES[(N-1) % 6]`, so the cycle repeats.
- Touch controls are enabled when `ontouchstart` or `maxTouchPoints` indicates a touch device.

## Git

- Develop on the branch the session specifies. Do not push to other branches without
  explicit permission.
- Do not create pull requests unless the user asks for one.

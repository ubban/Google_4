# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser.
The whole game is a single self-contained file, `index.html`, with inline CSS and
JavaScript. There is no build step, no bundler, no package manager, no external
assets, and no test suite.

Repository layout:

- `index.html` — the entire game (markup, styles, and game script).
- `README.md` — minimal project readme.
- `CLAUDE.md` — this file.

## Running the game

Open `index.html` directly in a browser, or serve the folder with any static server
(for example `python3 -m http.server` from the repo root) and visit it. Nothing needs
to be installed.

## Verifying changes

There are no automated tests or linters configured. To verify a change:

1. Reload `index.html` in a browser and check the browser console for errors.
2. Play through the affected path: start a game, fire, use a bomb, pick up a
   power-up, reach a stage boss, lose a life, and reach game over.
3. For layout or HUD changes, check both desktop and a narrow phone-sized viewport.
   The page uses `touch-action: none` and has touch controls, so test touch input
   when input code changes.

## Code structure inside `index.html`

The `<script>` block is organized by `// ---------- Section ----------` comments.
Keep new code in the matching section:

- **Config constants** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky
  palettes), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six bosses:
  VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN).
- **Adaptive performance** — `quality` and `trackPerf(dt)` lower render quality when
  frame times slip.
- **Audio** — a WebAudio synth (`ensureAudio`, `beep`, `noiseBurst`). No audio files
  are used; keep it that way.
- **Canvas sizing** — `BASE_W = 480`, `BASE_H = 720`, scaled by `resize()`.
- **Input** — keyboard, mouse steering, joystick, and touch buttons. Keyboard and
  joystick input take priority over mouse steering.
- **Game state** — `resetGame()` owns the initial values of all run state. Add new
  per-run state there so restarts stay correct.
- **Update / draw** — `update(dt)` advances the simulation, `draw()` renders, and
  `loop(now)` drives both via `requestAnimationFrame`.
- **Bosses** — `BOSS_PATTERNS` holds each boss's bullet pattern functions, and
  `updateBoss` / `killBoss` drive their phases.

## Conventions

- **Persistence** uses `localStorage` with keys prefixed `skyfighter.` (`bestScore`,
  `muted`, `difficulty`, `reducedMotion`). Read and write these through the existing
  `STORAGE_*` constants, not raw strings. Wrap new storage access in `try/catch`,
  because storage can be unavailable in private browsing.
- **Accessibility** — `announce(text)` writes to the `#srLive` live region. Use it for
  meaningful state changes such as stage starts, bomb use, and game over. Respect the
  reduced-motion setting (`reducedMotion`) for screen shake, flashes, and pulses.
- **Dependencies** — do not add external scripts, fonts, or assets. Everything stays
  inline in `index.html`, so the game works offline from a single file.
- **Style** — match the existing code: 2-space indentation, single quotes in JS,
  `const`/`let` (no `var`), and comments only where they explain intent.
- **Balance changes** — tune numbers in the config constants rather than hard-coding
  values in update logic.

## Git workflow

- Develop on the branch assigned to the session. Do not push to other branches without
  explicit permission.
- Do not create pull requests unless the user asks for one.
- Keep commits focused, with messages that describe the gameplay or code change.

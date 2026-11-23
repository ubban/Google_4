# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single self-contained HTML5 canvas page.

- `index.html` (~1670 lines): the entire game. CSS is in a `<style>` block (~lines 10–360), markup in `<body>` (~361–421), and all JavaScript in one IIFE `<script>` block (~422–end).
- `README.md`: placeholder, contains only the title and a timestamp line. Not documentation.

There is no `package.json`, build step, bundler, test suite, linter, or CI config. Nothing needs to be installed.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by playing in a browser: start a run, exercise the feature you changed, and check the console for errors.
- For visual or interaction checks, Playwright with the pre-installed Chromium works well (see the environment notes if running remotely).

## Code layout inside `index.html`

The script is organized by section comments and `const` tables near the top of the IIFE:

- **Config tables**: `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (six stage color palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, each with colors and an `hpMul`), and `BOSS_PATTERNS` (one array of attack functions per boss).
- **Persistence**: `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Reads and writes are direct, with no try/catch. Keep new keys under the same prefix.
- **Audio**: `ensureAudio()` lazily creates the AudioContext (browsers require a user gesture). `beep()`, `noiseBurst()`, and the `sfx` object produce all sounds procedurally; there are no audio files.
- **Input**: keyboard (`keys[e.code]`; arrows/WASD to move, Space to fire, X/Shift for bomb, P/Escape to pause), a touch joystick plus fire/bomb buttons, and mouse (`canvasPointFromEvent`).
- **Game state and loop**: `resetGame()` initializes state; `update(dt)` runs game logic; `draw()` renders; `loop(now)` drives them via requestAnimationFrame. Logic is frame-time (`dt`) based.
- **Rendering helpers**: `drawPlane`, `drawBossShip`, `drawClouds` (parallax layers), plus explosions and screen shake.
- **Canvas scaling**: fixed logical size `BASE_W` x `BASE_H` (480x720), scaled to fit by `resize()`.
- **UI overlay**: the DOM HUD (hearts, bombs, score, boss bar, combo pop, stage banner) is updated from the game loop. The overlay card hosts start, pause, and difficulty controls.
- **Accessibility**: `announce()` writes to the `#srLive` live region. Reduced motion is a user toggle (`reducedMotion`, stored under `skyfighter.reducedMotion`) that defaults to the OS `prefers-reduced-motion` setting and dampens screen shake and flashes; respect it in any new visual effect.

## Conventions

- Keep everything in the single `index.html` unless the user asks for a split. Do not introduce a build tool, framework, or npm dependency without a clear reason.
- Match the existing style: 2-space indentation, `const` for tables, camelCase names, `SCREAMING_CASE` for config constants, and short comments that name the section or pattern.
- Put new tunable numbers in the config tables near the top rather than hard-coding them in logic.
- Any new visual effect should check `reducedMotion`. Any new sound should go through `beep`/`noiseBurst` and respect `muted`.
- Keep touch and keyboard controls working together. A feature that is keyboard-only is a regression on mobile.
- Test on a narrow viewport (phone width) as well as desktop; the layout must not scroll horizontally.

## Git

- Development branch for this session: `claude/gifted-tesla-6cfw0n`. Push there with `git push -u origin <branch>`.
- Commit messages: a short imperative summary line, then a body explaining the why when it isn't obvious.
- Do not open pull requests unless explicitly asked.

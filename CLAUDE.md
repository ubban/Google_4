# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: markup, CSS, and JavaScript in one file (~1,670 lines).
- `README.md` — a one-line title stub. Keep it minimal unless asked to expand it.

There is **no build step, no package manager, no test suite, and no linter**. Do not add a `package.json`, bundler, or framework unless the user asks for one.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by loading the page and playing: check the title overlay, a full run, pause/resume, bomb use, boss fights, and game-over with the best score saved.
- For a quick syntax check of the inline script, extract the `<script>` block and run `node --check` on it.
- Test on a narrow viewport as well as desktop. The canvas scales to fit the window, and touch controls appear on touch-capable devices.

## File layout inside `index.html`

| Region (approx. lines) | Contents |
|---|---|
| 1–10 | `<head>`, inline SVG favicon, `<title>` |
| 10–360 | `<style>`: HUD pills, hearts/bombs, overlay card, touch controls, reduced-motion rules |
| 361–421 | `<body>`: `#gameCanvas`, HUD, overlay, difficulty buttons, touch controls, live region |
| 422–1669 | One IIFE `<script>` containing all game logic |

Inside the script, the sections are marked with `// ---------- Section ----------` comments. Follow that convention when adding new sections.

Key constants and tables near the top of the script:

- `DIFFICULTIES` — spawn rate, enemy speed, enemy fire chance, starting lives and bombs, boss HP multiplier for `easy` / `normal` / `hard`.
- `KILLS_PER_STAGE` — kills needed before a boss stage.
- `THEMES` — background palettes, cycled by stage.
- `ENEMY_TYPES` — `grunt`, `interceptor` (weaves), `tank` (multi-hit, appears from stage 2).
- `BOSSES` — six bosses with colors and HP multipliers.
- `BOSS_PATTERNS` — bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, `fireHomingSeeded`, etc.) that bosses cycle through.
- `BASE_W` / `BASE_H` — the fixed logical resolution (480×720). All game coordinates use this space; `resize()` only changes CSS size.

Core loop: `loop(now)` clamps `dt` to 50 ms, runs `trackPerf(dt)` for adaptive quality, then calls `update(dt)` followed by `draw()` while the game is running and not paused.

## Conventions

- **Keep it one file.** New game code goes in the existing IIFE `<script>`. Styles go in the existing `<style>` block. Do not split out separate JS/CSS files unless asked.
- **Match the surrounding style.** The code uses 2-space indentation, single quotes in JS, `const`/`let` (no `var`), arrow functions for callbacks, and short comments that name a section or explain a non-obvious choice.
- **Persistence goes through the `skyfighter.*` localStorage keys.** The existing keys are `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, and `skyfighter.reducedMotion`. Any new persisted setting should use the same prefix and store values as strings. The current code does not guard storage calls with try/catch; if you add guards, add them to the existing calls too.
- **Accessibility.** Announce important state changes through `announce()`, which writes to the `#srLive` region. Respect `prefers-reduced-motion` through the existing `reducedMotion` flag and the `body.reduced-motion` class when adding screen shake, flashes, or similar effects.
- **Audio** is synthesized with the Web Audio API (`beep`, `noiseBurst`). Call `ensureAudio()` from user-gesture handlers before playing sound. Do not add audio asset files.
- **Input** must work on keyboard (arrows/WASD, Space to fire, X or Shift for bomb, P or Esc for pause), mouse, and touch. If you add a control, update the `#controlsHint` text and the touch controls.
- **No external resources.** The page uses no CDNs, fonts, or network requests. Keep it that way unless the user asks otherwise.
- **Performance.** The game uses an adaptive quality level (`quality` 0–2) driven by frame times. Keep per-frame allocations low in `update` and `draw`, since they run every frame.

## Git workflow

- Develop on the branch designated for the session (currently `claude/gifted-tesla-81mox2`). Do not push to `main` directly.
- Write commit messages in the imperative mood with a short summary line. Match the style of the existing history (for example, `Add Sky Fighter: Raiden-style vertical shoot-em-up game`).
- Do not open a pull request unless the user asks for one.

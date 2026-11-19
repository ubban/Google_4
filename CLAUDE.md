# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser.

- `index.html` — the entire game: markup (~lines 1–420), CSS (inline `<style>`, ~lines 10–362), and JavaScript (inline `<script>`, ~lines 422–1669).
- `README.md` — a stub with only a title. Keep it accurate if you expand it.

There is no build step, package manager, bundler, or test suite. There is no `package.json`, linter config, or CI.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- No install is required.
- Verify changes by playing the game in a browser. Check the start overlay, all three difficulty buttons, weapons, bombs, pause, boss fights, and the game-over screen with a new best score.
- For automated checks, a headless Chromium with Playwright works well for loading the page and catching console errors. Do not add a test framework or dependencies unless asked.

## Code structure inside `index.html`

The script is one IIFE. Keep new code inside it so globals stay out of `window`.

Key sections, in the order they appear:

1. **DOM refs** — `getElementById` lookups at the top of the script.
2. **Persistence keys** — `STORAGE_*` constants. All use the `skyfighter.` prefix (`bestScore`, `muted`, `difficulty`, `reducedMotion`).
3. **Tuning tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (one per stage), `ENEMY_TYPES`, `BOSSES`. Balance changes usually belong here, not in the update loop.
4. **Audio** — a lazily created `AudioContext` (`ensureAudio`), plus `beep`, `noiseBurst`, and the `sfx` map. Audio starts only after a user gesture.
5. **Feedback** — HUD helpers (`renderHearts`, `renderBombs`), screen effects (`flashHit`, `screenShake`, `showStageBanner`, `showComboPop`).
6. **Input** — keyboard (`keys` map keyed by `e.code`), mouse, touch joystick, and on-screen buttons. Shared controls must work on all of them.
7. **Game state** — top-level `let` variables (`score`, `lives`, `stage`, `chain`, `multiplier`, and others), reset by `resetGame()`.
8. **Entities and rendering** — `drawPlane`, `drawBossShip`, and `draw()`. Drawing uses canvas 2D with a logical `BASE_W × BASE_H` (480 × 720) space scaled by `resize()`.
9. **Spawning and bosses** — `spawnEnemy`, `spawnBoss`, and the boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireCross`, `fireHomingSeeded`, and others) called from `updateBoss`.
10. **Update and loop** — `update(dt)` advances game state, `loop(now)` drives `requestAnimationFrame`. `trackPerf` lowers the `quality` level when frame times are slow.

## Conventions

- **Vanilla only.** Use plain HTML, CSS, and JavaScript with no frameworks or npm dependencies. The game must keep working from a file opened directly in the browser.
- **Keep it in one file** unless the user asks for a split. Do not add build tooling.
- **Match the existing style:** 2-space indentation, single quotes, `const` by default, and small named functions.
- **Balance lives in the tables.** Change `DIFFICULTIES`, `ENEMY_TYPES`, or `BOSSES` rather than adding magic numbers to the update code. Name any new tunable constant.
- **Respect the accessibility features:**
  - `reducedMotion` should turn off or soften screen shake and flashes.
  - `announce()` feeds the `srLive` live region for screen readers. Use it for important state changes.
  - Keep the `prefers-reduced-motion` check working.
- **Support touch and keyboard.** Any new control needs a keyboard binding and a touch or on-screen equivalent. Update the `controlsHint` text when controls change.
- **Persist with `localStorage` only** and always under the `skyfighter.` prefix. Access to `localStorage` can throw in some private or restricted browser modes, so guard new reads and writes if you add them.
- **No external network calls, analytics, or third-party assets** unless the user asks. The page is self-contained, and the favicon is an inline data URI.

## Git workflow

- Develop on the branch the session names (currently `claude/gifted-tesla-r8q9w5`). Do not push to other branches without permission.
- Commit only when asked. Keep commits focused on one change.
- Do not open a pull request unless the user asks for one.

## Things to avoid

- Do not split `index.html` into modules or add a bundler without an explicit request.
- Do not remove the `preventDefault` on `Space`, which stops the page from scrolling during play.
- Do not add a `window`-level global for game state. Keep it inside the IIFE.
- Do not change the `skyfighter.*` storage key names. Renaming them would silently reset players' saved best scores and settings.

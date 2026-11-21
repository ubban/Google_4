# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-em-up written as a single-page HTML5 canvas game.

The repository is intentionally minimal:

| Path        | Purpose                                                                 |
|-------------|-------------------------------------------------------------------------|
| `index.html` | The entire game: markup, CSS (`<style>`), and JavaScript (`<script>`). ~1,670 lines. |
| `README.md`  | Placeholder (title only). Not yet a real project description.           |
| `.git/`      | Git history. Branches: `claude/gifted-tesla-giyi1s` is the active dev branch. |

There is **no build system, package manager, bundler, test suite, linter, or CI config**.
Do not invent `npm`/`pip` commands or test runners; none exist.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, e.g.:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

No install step is needed. Verification is manual: load the page, play, and check the
behavior you changed. Browser automation (Playwright with the pre-installed Chromium) is
available if an automated check is needed.

## Code layout inside `index.html`

- **`<style>`** (around lines 10–362): all styling, including the pill-style HUD, the
  glassmorphic overlay card, and the animated title/overlay entrance.
- **Markup** (around lines 363–421): a `<canvas id="gameCanvas">`, HUD elements, overlay
  cards, and on-screen touch controls (joystick zone, FIRE, BOMB).
- **Script** (starts around line 422): one IIFE `(function () { ... })()` that holds all
  game state and logic. Nothing is exported and no globals are added. Keep new code inside
  this IIFE.

Key sections of the script, in rough order:

| Area | Notes |
|------|-------|
| Constants | `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES` (6 sky palettes cycled per stage), `ENEMY_TYPES` (grunt / interceptor / tank), `BOSSES` (6 bosses) |
| Persistence | `localStorage` keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` |
| Audio | Lazily created `AudioContext` (`ensureAudio`), synthesized SFX via `beep()` / `noiseBurst()`. No audio files. |
| Input | Keyboard (`keys[e.code]`), mouse steering and click-to-fire on the canvas, touch joystick + FIRE/BOMB buttons |
| Sizing | Fixed logical resolution `BASE_W = 480`, `BASE_H = 720`, scaled by `resize()` |
| Game loop | `update(dt)` for logic, `draw()` for rendering, `loop(now)` driven by `requestAnimationFrame` |
| Weapons / bombs | `fireWeapon()` (Vulcan/Laser, 3 power levels), `useBomb()` (limited, clears the screen, damages bosses) |
| Bosses | `BOSS_PATTERNS` are bullet-pattern functions; `updateBoss` / `killBoss` manage the fight |
| Lifecycle | `resetGame()`, `loseLife()`, `endGame()`, `togglePause()` |

### Controls (as implemented)

- Move: Arrow keys or WASD; mouse hover steers on desktop; touch-drag on the left zone.
- Fire: Space (hold), or hold the mouse button, or tap FIRE on touch.
- Bomb: `X` or Shift, or tap BOMB on touch.
- Pause: `P` or Esc.

### Difficulty

`easy`, `normal` (default), `hard`. Each sets spawn rates, enemy shoot chance, starting lives
and bombs, and a boss HP multiplier. Difficulty is changed through `setDifficulty(name)`, which
also persists the choice.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not add a
  build step, framework, or dependency without an explicit request.
- **No external network dependencies.** The game is self-contained. Don't add CDN scripts or
  fonts without asking.
- **Match the existing style:** 2-space indentation, `const`/`let`, plain ES2015+ JavaScript,
  and the existing naming (`camelCase` functions, `UPPER_SNAKE` constants, `STORAGE_*` keys).
- **Preserve accessibility hooks.** Respect the `reducedMotion` setting (it gates screen shake and
  similar effects), keep the `announce()` live-region usage, and keep touch controls working.
- **Preserve persistence keys.** Renaming a `skyfighter.*` localStorage key silently resets
  players' saved best score, mute, difficulty, and motion settings.
- **Keep the game loop deterministic-ish and frame-rate independent.** Logic uses the `dt` passed
  into `update(dt)`; don't reintroduce fixed per-frame speeds without scaling by `dt`.
- Keep changes focused. Don't reformat unrelated parts of `index.html`; the file is large and a
  diff-noise change makes review hard.

## Verification checklist

Because there are no automated tests, before reporting a change as done:

1. Load the page with no console errors.
2. Start a run on each affected difficulty if the change touches balance or spawning.
3. Check the path you changed: movement, firing, bombs, a boss fight (after 18 kills per stage),
   pause, game over, and restart.
4. Check touch controls if the change touches input, and a narrow viewport if it touches layout.

## Git

- Active development branch: `claude/gifted-tesla-giyi1s`.
- Commit messages in this repo use a short imperative summary line followed by a bullet-list
  body describing the change (see `c4dc8f6`).
- Don't push to other branches or open pull requests unless explicitly asked.

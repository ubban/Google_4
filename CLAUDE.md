# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only app is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a `<canvas>`.

- `index.html` — the entire game: markup, CSS (inline `<style>`), and JavaScript (inline `<script>`). Roughly 1,670 lines.
- `README.md` — placeholder (title only).
- No build step, no package manager, no test suite, no external dependencies. The page loads no external scripts, stylesheets, or fonts (the favicon is an inline SVG data URI).

## Running and checking changes

- Open `index.html` directly in a browser, or serve the folder with any static server (for example `python3 -m http.server` from the repo root) and visit `http://localhost:8000`.
- There are no automated tests or linters configured. Verify changes by playing the game in a browser, including the mobile layout where it matters (the game uses touch controls and a viewport meta tag that disables zoom).

## Code structure inside `index.html`

The script is one `(function () { ... })();` block, so state lives in `const`/`let` variables scoped to it. Main pieces:

- **Persistence keys** (`STORAGE_*`): `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` in `localStorage`. Keep these names stable so saved player data keeps working.
- **Difficulty table** (`DIFFICULTIES`): `easy`, `normal`, `hard`, each setting spawn rates, enemy speed, enemy fire chance, starting lives, starting bombs, and a boss HP multiplier. `KILLS_PER_STAGE` controls stage progression.
- **Content tables**: `THEMES` (stage backgrounds), `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS` (bullet-pattern functions such as `fireRadial`, `fireSpiralBurst`, `fireHomingSeeded`).
- **Audio**: Web Audio helpers (`ensureAudio`, `beep`, `noiseBurst`, `panFor`). Audio is created lazily on the first user gesture.
- **Game loop**: `resetGame` initialises state, `update(dt)` advances the simulation, `draw()` renders, and `loop(now)` drives both via `requestAnimationFrame`. Keep update and draw logic separate.
- **Input**: keyboard (`keydown`/`keyup` on `e.code`), touch joystick and buttons (`touch*` handlers), and mouse/pointer on the canvas. Input state lives in `keys`, `firing`, and the joystick variables.
- **Layout**: `resize()` fits a fixed logical canvas (`BASE_W = 480`, `BASE_H = 720`) into the viewport.
- **Accessibility**: `announce()` writes to the `#srLive` live region. Reduced motion (`reducedMotion`) disables screen shake, flashes, and pulses.

### Controls (for reference when editing input code)

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrow keys or WASD | Left-side joystick |
| Fire | Space (hold) | FIRE button |
| Smart bomb | X or Shift | BOMB button |
| Pause | P or Escape | Pause button (top bar) |
| Mute | — | Mute button (top bar) |

## Conventions

- Keep the project as a single self-contained HTML file unless the user asks to split it. Do not add a bundler, framework, or npm dependency without asking.
- Match the existing style: 2-space indentation, `const` by default, small named functions, comments only where the logic is not obvious.
- Tuning values (speeds, spawn rates, HP, bullet counts) are intentionally inlined in the tables and functions above. Change them in place rather than adding new config layers.
- Do not add external CDN scripts or fonts unless requested. Keep the game working offline.
- Any new user-facing setting that should persist needs a new `skyfighter.*` key and must degrade gracefully if `localStorage` is unavailable or throws.

## Git

- Development branch for automated work: `claude/gifted-tesla-2yl2qb`. Do not push to other branches without explicit permission.
- Commit messages should be short and descriptive in the imperative mood (existing history: "Add Sky Fighter: Raiden-style vertical shoot-em-up game").
- Do not open pull requests unless the user asks for one.

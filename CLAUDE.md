# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free front-end project. Its only substantive content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas app.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: inline `<style>`, canvas markup, and one inline `<script>` (~1,670 lines). |
| `README.md` | Minimal project readme (title only). |

There is no build step, package manager, test suite, or linter configured.

## Running the project

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Architecture of `index.html`

- **Rendering:** a single `<canvas id="gameCanvas">` driven by `requestAnimationFrame` (`loop(now)` → `update(dt)` → `draw()`). Logical play area is `BASE_W = 480` × `BASE_H = 720`; `resize()` scales it to the viewport.
- **Input:** keyboard via `keys[e.code]`, plus touch/pointer handling (`joystickStart/Move/End`, `canvasPointFromEvent`) for mobile.
- **Audio:** WebAudio synth helpers (`ensureAudio`, `beep`, `noiseBurst`) — no audio asset files. Mute is persisted.
- **Game data:** tuning lives in constant tables near the top of the script — `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, `BOSS_PATTERNS`, and `KILLS_PER_STAGE`. Prefer changing these tables over hard-coding numbers in logic.
- **Persistence:** `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). All reads/writes are best-effort; keep them safe if storage is unavailable.
- **Accessibility:** respects `prefers-reduced-motion` (with a user override), and `announce()` provides screen-reader text.

### Controls (for reference when testing)

- Move: Arrow keys or WASD; touch joystick on mobile.
- Fire: Space (hold for continuous fire) or pointer/touch.
- Bomb: X or Shift (limited per difficulty).
- Pause: P or Escape.

## Conventions

- Keep the game as a **single self-contained HTML file** unless the user asks for a split. Do not introduce a bundler, framework, or npm dependencies without asking.
- Plain modern JavaScript (`const`/`let`, arrow functions) in the existing style; 2-space indentation.
- Put new tunable values into the existing config tables rather than inline magic numbers.
- Any new gameplay state must be reset in `resetGame()`.
- Keep the mobile layout working: the viewport meta disables user scaling, so verify canvas sizing and touch controls after UI changes.

## Working in this repo

- Default development branch: `claude/gifted-tesla-vx8qm7`. Commit and push there; do not push to other branches without explicit permission.
- Do not create pull requests unless explicitly asked.
- Before finishing a change to gameplay or UI, load `index.html` in a browser (or run a headless check) and confirm the game starts, runs, and reaches a game-over without console errors.

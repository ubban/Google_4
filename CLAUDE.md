# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free web project. Its only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up, implemented as a single self-contained file:

- `index.html` — the entire game: inline `<style>`, markup, and a single inline `<script>` that draws everything on a `<canvas>`.
- `README.md` — minimal title-only readme.

There is no build step, package manager, test suite, or framework. Do not introduce one unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server, e.g.:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Code layout inside `index.html`

- **Constants (around line 460–510):** `STORAGE_*` localStorage keys (`skyfighter.*`), `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
- **Persisted settings:** best score, mute, difficulty, and reduced-motion are stored in `localStorage`. Reads and writes are direct; keep that in mind when changing defaults or keys, since existing players have saved values.
- **Input (around line 760–870):** keyboard (`keydown`/`keyup` on `e.code`), touch joystick plus fire/bomb buttons, and mouse on the canvas. Any new control must be wired for both keyboard and touch.
- **Game state:** `resetGame()` (~875) initializes state; `update(dt)` (~1148) advances the simulation; `updateBoss(dt)` (~1411) drives boss attack patterns (`BOSS_PATTERNS`, ~1372).
- **Rendering:** `draw()` (~1520) and helpers such as `drawPlane`, `drawBossShip`, `drawClouds`. Rendering is pure canvas 2D.
- **Main loop:** `loop(now)` (~1640) runs on `requestAnimationFrame` with a delta-time `dt`. Logic should use `dt`, not frame counts, so the game behaves the same at different frame rates.
- **Layout:** the canvas uses a fixed logical size (`BASE_W = 480`, `BASE_H = 720`, ~line 753) and scales to fit the viewport. Keep HUD and overlay positions relative to that logical size.

## Conventions

- **Single file:** keep CSS, markup, and JS inside `index.html` unless the user asks for a split. Do not add external CDN dependencies for the game.
- **Plain JavaScript:** no transpilation, no modules, no TypeScript. Match the existing style (2-space indent, `const`/`let`, arrow callbacks, `camelCase` identifiers, `UPPER_SNAKE` for constants).
- **Mobile-first:** the page is used on phones (`touch-action: none`, `viewport-fit=cover`, safe-area insets). Verify touch paths when changing input or layout.
- **Accessibility and settings:** respect the reduced-motion and mute settings when adding effects or sounds.
- **Storage keys:** new persisted values should use the `skyfighter.` prefix. Do not rename existing keys without a migration.

## Verifying changes

There are no automated tests. To verify a change:

1. Load `index.html` in a browser (desktop and, if input changed, a touch-emulated viewport).
2. Play through at least one stage start to a boss, using keyboard, and check the HUD, scoring, and best-score persistence after a reload.
3. Check the browser console for errors.

A Playwright-driven smoke test is reasonable for larger changes; the project does not currently include one.

## Git

- Development branch for this work: `claude/gifted-tesla-mh4afx`.
- Commit messages should be short and descriptive in the imperative mood, matching existing history (e.g. "Add Sky Fighter: ...").
- Do not open pull requests unless explicitly requested.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

Google_4 is a small static web project containing **Sky Fighter**, a Raiden-style vertical shoot-'em-up played in the browser on a `<canvas>`.

- `index.html` — the entire game: inline CSS, markup, and a single inline `<script>`. About 1,670 lines.
- `README.md` — minimal project stub.

There is no build system, no package manager, no test suite, and no external assets. Audio is synthesized with WebAudio. Do not add a bundler, framework, or dependencies unless the user asks for one.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```
python3 -m http.server 8000
```

Then visit `http://localhost:8000/`.

## Code layout inside `index.html`

The script is one IIFE-style block. Roughly in order:

1. **DOM references** (`canvas`, HUD elements, overlay, touch controls, boss bar).
2. **Storage keys** — `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. All `localStorage` access must tolerate failure (private mode, blocked storage).
3. **Adaptive performance** — `trackPerf` lowers `quality` (0 full / 1 medium / 2 low) when frame times slip.
4. **Config tables** — `DIFFICULTIES`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`.
5. **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` map. Audio is created lazily on first user gesture.
6. **HUD and feedback** — hearts, bombs, combo pop, stage banner, screen shake, hit/bomb flashes.
7. **Canvas sizing** — fixed logical size `BASE_W × BASE_H` (480 × 720), scaled to fit the window by `resize()`.
8. **Input** — keyboard, touch joystick plus fire/bomb buttons, and mouse aiming on the canvas.
9. **Game state and rendering** — `resetGame`, `drawPlane`, `drawBossShip`, `spawnEnemy`, etc.
10. **Update / loop** — `update(dt)` runs the simulation; `loop(now)` drives it with `requestAnimationFrame`.

When changing game balance, prefer editing the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) over hard-coding numbers in logic.

## Controls (for reference when testing)

- Move: arrow keys or WASD. Mouse moves the ship when the pointer is over the canvas.
- Fire: Space (hold). Mouse-down on the canvas also fires.
- Smart bomb: X or Shift.
- Pause: P or Escape.
- Touch: left-side joystick, right-side fire and bomb buttons.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. Preserve the existing style: 2-space indentation, single quotes in JS, `const`/`let`, and short section comments of the form `// ---------- Name ----------`.
- Keep the game responsive: it must render correctly at phone width with the viewport meta and `touch-action: none` already in place. Do not introduce page scrolling.
- Respect `reducedMotion` for any new screen shake, flashes, or large animations.
- Guard every new `localStorage` call with try/catch.
- Gate per-frame allocations and expensive effects on `quality` where practical.

## Verification

There are no automated tests. For changes to gameplay or rendering:

1. Load the page in a browser (a headless Chromium with Playwright works well) and check the browser console for errors.
2. Start a run, exercise the changed path, and confirm the HUD updates and the game over overlay still appears when lives reach zero.
3. Check both a desktop-sized and a phone-sized viewport if layout or touch code changed.

If you cannot run the game in this environment, say so explicitly and limit claims to what was verified by reading the code.

## Git

- Development branch for this repo: `claude/gifted-tesla-9n6etw`. Commit and push there; do not push to other branches without explicit permission.
- Do not create pull requests unless the user asks.
- Commit messages: short imperative subject, optional body explaining *why*.

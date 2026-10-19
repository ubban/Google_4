# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a single HTML5 canvas page.

Files:

- `index.html` — the entire game: markup, CSS (`<style>` block), and JavaScript (one inline `<script>` IIFE). About 1,670 lines.
- `README.md` — a placeholder title only.
- No build system, package manager, tests, or linter exist yet.

## Running the game

No build step. Open `index.html` directly in a browser, or serve the folder with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

There is no test suite. Verify changes by playing the game in a browser. Check at least: the start overlay, a full stage, a boss fight, pause/resume, game over and the best-score update, and the mobile touch controls (use a narrow viewport).

## Architecture of `index.html`

Everything lives in one IIFE inside the inline `<script>`. Keep new code inside that IIFE and avoid globals.

Major sections, in file order:

1. **DOM references** — `const` lookups by element id (`scoreEl`, `heartsEl`, `bossBarFill`, `srLive`, etc.). Any new HUD element needs an id here.
2. **Storage keys** — `skyfighter.*` in `localStorage` (`bestScore`, `muted`, `difficulty`, `reducedMotion`).
3. **Adaptive performance** — `trackPerf(dt)` samples the last 45 frame times and steps `quality` between 0 (full), 1 (medium), and 2 (low). Shadows, glow, and particle counts read `quality`.
4. **Config tables** — `DIFFICULTIES` (spawn rate, enemy speed, shot chance, lives, bombs, boss HP multiplier), `THEMES` (background stages), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six boss designs), `BOSS_PATTERNS` (attack patterns), and `KILLS_PER_STAGE` (18).
5. **Audio** — a lazily created `AudioContext` (`ensureAudio()`), with synthesized effects via `beep()`, `noiseBurst()`, and a `sfx` object. Mute state is persisted.
6. **UI feedback** — hearts/bombs rendering, hit and bomb flashes, combo popups, stage banners, screen shake, and `announce()` for screen-reader text.
7. **Input** — keyboard (`keys` map via `e.code`), touch joystick plus fire/bomb buttons, and mouse drag on the canvas. Bomb keys are `KeyX` and Shift.
8. **Game state** — `resetGame()`, `update(dt)` (movement, spawning, collisions, scoring), `updateBoss(dt)`, `killBoss()`, `loseLife()`, `endGame()`, `togglePause()`.
9. **Rendering** — `draw()` plus helpers `drawPlane`, `drawBossShip`, `drawClouds`. Rendering is pure drawing and should not change game state.
10. **Main loop** — `loop(now)` clamps `dt` to 0.05 s, calls `trackPerf`, then `update` and `draw` while running and not paused.

### Conventions

- Time-based motion: multiply by `dt` (seconds). Do not assume a fixed frame rate.
- Logical resolution is fixed at 480×720 (`BASE_W`, `BASE_H`). `resize()` scales the canvas CSS size only; draw in logical coordinates.
- Respect `reducedMotion`: skip or shrink screen shake and flashes, as the existing helpers do.
- Use `quality` to scale expensive effects rather than adding fixed counts.
- Persist user settings through the `skyfighter.*` keys with direct `localStorage` calls, as the existing code does.
- Match the file's style: 2-space indentation, single quotes, arrow functions in callbacks, section comments written as `// ---------- Name ----------`.

## Making changes

- Keep the project a single self-contained HTML file unless the user asks for a split. Do not add dependencies or a bundler without being asked.
- When tuning balance, change the `DIFFICULTIES`, `ENEMY_TYPES`, or `BOSSES` tables instead of hard-coding numbers in the loop.
- Keep changes focused. Do not reformat unrelated code.

## Git workflow

- Develop on the branch named by the session. Do not push to other branches without explicit permission.
- Commit messages are descriptive, in the imperative mood, with a short summary line followed by a body that explains the why.
- Do not open a pull request unless the user asks for one.

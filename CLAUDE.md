# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up.

- Entire game lives in a single file: `index.html` (~1,670 lines: inline `<style>`, markup, and one inline `<script>`).
- `README.md` is a placeholder (title only).
- No build step, no package manager, no tests, no external assets. Audio is synthesized with WebAudio; graphics are drawn on a 2D `<canvas>`.

To run it, open `index.html` in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).

## Repository layout

| Path | Purpose |
| --- | --- |
| `index.html` | The whole game: CSS, HTML overlay/HUD/touch controls, and game logic. |
| `README.md` | Minimal project readme. |

## Architecture of `index.html`

The inline script is organised by `// ---------- Section ----------` comments. Key sections, in file order:

1. **DOM references and constants** – cached element lookups; `STORAGE_*` localStorage keys (`skyfighter.*`).
2. **Adaptive performance** – `trackPerf` samples frame times and sets `quality` (0 full, 1 medium, 2 low).
3. **Tuning data** – `DIFFICULTIES` (easy/normal/hard), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled by stage).
4. **Settings state** – best score, mute, difficulty, and reduced-motion preference, persisted to localStorage.
5. **Audio** – `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` table. Audio context is created lazily on first user gesture.
6. **HUD and effects** – hearts/bombs rendering, flash/vignette, combo pop, stage banner, screen shake.
7. **Canvas sizing** – fixed logical resolution `BASE_W × BASE_H` (480 × 720), scaled to fit the window (capped at 1.6×).
8. **Input** – keyboard (`keydown`), mouse, and touch (virtual joystick on the left, FIRE and BOMB buttons on the right).
9. **Entities and drawing** – `drawPlane`, `drawBossShip`, cloud/background drawing.
10. **Game update** – `update(dt)` for the player, enemies, bullets and collisions; `updateBoss(dt)` for boss phases.
11. **Render** – `draw()`.
12. **Main loop** – `loop(now)` driven by `requestAnimationFrame`.

Gameplay facts worth knowing before editing:
- Three lives, two bombs on Normal (difficulty changes these and spawn rates, enemy speed, shot chance, and boss HP multiplier).
- Weapon power-ups change the weapon name shown in the HUD; score multiplier is shown separately.
- Stage progression is driven by kill count: a boss warning fires after `KILLS_PER_STAGE` (18) kills. Stage N uses `BOSSES[(N-1) % 6]` and `THEMES[(N-1) % 6]`, so both cycle every six stages.
- Controls: mouse steers and clicks to fire; arrows/WASD move; Space fires; X or Shift bombs; P or Esc pauses. Touch uses the on-screen joystick and buttons.

## Conventions

- **Keep it a single file.** Do not introduce a bundler, framework, or external dependencies unless the user explicitly asks. The game is meant to open directly from disk.
- **No external network assets.** Keep audio synthetic and graphics procedural.
- **Match the existing style:** 2-space indentation, single quotes in JS, `const`/`let` (no `var`), small named functions, and section banners in the `// ---------- Name ----------` form.
- **Persist settings only via the existing `STORAGE_*` keys** and always guard `localStorage` access in case it is unavailable.
- **Respect accessibility hooks:** keep the `prefers-reduced-motion` check and the `srLive` announcer working when adding visual or audio feedback.
- **Keep touch and keyboard parity.** Any new action should work from both the keyboard and the touch controls.
- Prefer editing existing functions over adding parallel ones; tuning values belong in the data tables at the top of the script.

## Working on this repo

- Default branch for development: `claude/gifted-tesla-rlhtt2`. Do not push to other branches without permission.
- There is no test suite or linter. Verify changes by loading `index.html` in a browser (a headless Chromium with Playwright is fine) and playing a few seconds, checking for console errors.
- Check both desktop (keyboard/mouse) and narrow/touch layouts when touching HUD, overlay, or control CSS; the layout must not scroll horizontally.
- Keep changes focused. Don't reformat unrelated parts of `index.html`, since the file is one large diff surface.

## Git

- Commit messages: short imperative summary line, optional body explaining *why*.
- Do not create pull requests unless explicitly asked.

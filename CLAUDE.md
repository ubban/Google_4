# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only substantial content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a single HTML file, `index.html` (~1,670 lines). There is no build step, no package manager, no test suite, and no server-side code. `README.md` is a placeholder.

## Repository layout

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` (an IIFE wrapping all game logic).
- `README.md` — minimal project title.
- `.git/` — git metadata only.

## Running the game

No build is needed. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```
python3 -m http.server 8000
```

Then visit `http://localhost:8000/index.html`.

## Architecture of `index.html`

All game code lives in one IIFE inside the `<script>` block. Rough order of sections:

1. **DOM lookups** — every element is fetched once at the top by ID (`gameCanvas`, `overlay`, `hud` items, touch controls, boss bar, etc.). Keep IDs in sync between markup and script.
2. **Storage keys** — `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` in `localStorage`. Every read/write should tolerate missing or unavailable storage.
3. **Adaptive performance** — `trackPerf` samples frame time and steps a `quality` level (0 full, 1 medium, 2 low) to reduce effects when frames run slow.
4. **Tuning tables** — `DIFFICULTIES` (spawn rates, enemy speed/fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled by stage). Balance changes should usually start here.
5. **Audio** — a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, the `sfx` object). Audio is created lazily on first user gesture; respect the `muted` flag.
6. **Input** — keyboard (`keys` map keyed by `e.code`), mouse (steer and click-to-fire on the canvas), and touch (a left-side joystick zone, plus FIRE and BOMB buttons). Pause is `P` or `Esc`; bomb is `X` or `Shift`; fire is `Space` or held mouse/touch.
7. **Game state and loop** — `resetGame`, `update(dt)`, `draw()`, and `loop(now)` driven by `requestAnimationFrame`. `update` handles spawning, collisions, scoring, combos, power-ups, and stage progression. `draw` renders everything to the 2D canvas.
8. **Weapons and power-ups** — two weapon families, `vulcan` (spread shots) and `laser` (piercing lanes), each with 3 levels. Power-up types are `life`, `bomb`, and `weapon`; the `weapon` pickup levels up first, then toggles weapon family.
9. **Bosses** — `BOSS_PATTERNS` are bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring). `updateBoss` picks patterns; `killBoss` advances the stage.
10. **Flow control** — `loseLife`, `endGame`, `togglePause`, and the start button handler. Game over updates `bestScore` in storage.

The canvas is a fixed logical size of **480 × 720** (`BASE_W`, `BASE_H`), scaled by `resize()` to fit the viewport. Keep game coordinates in that space.

## Conventions

- Plain HTML, CSS, and JavaScript using modern syntax already in use (arrow functions, `const`/`let`). Do not introduce a bundler, framework, or npm dependencies unless the user asks.
- Keep everything in the single IIFE; do not add globals. Keep the file self-contained so it still works when opened from disk.
- Preserve accessibility hooks: the `srLive` live region (`announce()`), `prefers-reduced-motion` handling, and the `reduced-motion` body class.
- Keep touch support working. Controls use `touchstart`/`touchmove`/`touchend` with `passive: false` where `preventDefault` is needed.
- Stay within the existing visual style (inline `<style>` block, dark `color-scheme`).
- Match the existing indentation (2 spaces) and the terse comment style (section headers like `// ---------- Name ----------`).

## Testing and verification

There is no automated test suite or linter configured. To verify changes:

- Load `index.html` in a browser and play through at least one stage, a boss fight, a life loss, and game over.
- Check all three difficulties, pause/resume (`P`/`Esc`), mute and reduced-motion toggles, and that best score persists across reloads.
- For touch changes, test with device emulation or a touch-enabled browser.
- If a headless browser is used, Playwright is pre-installed in the remote environment; do not reinstall it.

## Git workflow

- Develop on the branch assigned to the session. Do not push to other branches without explicit permission.
- Do not open a pull request unless the user asks for one.
- Commit messages should be short and imperative, describing the gameplay or code change (see existing history: "Add Sky Fighter: ...").

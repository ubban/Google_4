# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small, single-page HTML5 game called **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on a `<canvas>`.

- `index.html` — the entire game: markup, CSS (in `<style>`), and JavaScript (in one `<script>` IIFE). ~1,670 lines.
- `README.md` — placeholder title only.

There is no build step, no bundler, no `package.json`, no test suite, and no dependencies. The game runs by opening `index.html` in a browser.

## Running it

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server` and visit `http://localhost:8000`).
- No install step is required.

## Code layout inside `index.html`

Everything lives in the single IIFE in the `<script>` block. Key sections, roughly in file order:

- **DOM references** — `getElementById` lookups for HUD, overlay, touch controls, and buttons.
- **Storage keys** — `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. These are read from `localStorage`; keep the names stable so saved player settings survive.
- **Tuning constants**
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rate, enemy speed, enemy shot chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (currently 18).
  - `THEMES` — six stage backdrops that cycle by stage.
  - `ENEMY_TYPES` — `grunt`, `interceptor` (weaves), `tank` (4 HP). `tank` is gated to stage 2+ via `minStage`.
  - `BOSSES` — six boss designs (Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan) that cycle by stage, each with its own `hpMul`.
  - `BASE_W = 480`, `BASE_H = 720` — the logical play field, scaled to fit the window (capped at 1.6x).
- **Audio** — a lazily created `AudioContext` (`ensureAudio`) feeding a master gain and dynamics compressor. Sounds are built from `beep` (oscillators) and `noiseBurst`, grouped in the `sfx` object. Audio is initialized on the first key press or interaction.
- **Input** — keyboard (`keys` map keyed by `KeyboardEvent.code`), mouse steering and click-to-fire on the canvas, and a touch joystick plus FIRE/BOMB buttons. Keyboard and joystick input take priority over mouse steering.
- **Update / draw** — `update(dt)` advances the simulation, `updateBoss(dt)` handles boss patterns, `draw()` renders everything, and `loop(now)` drives both via `requestAnimationFrame`.
- **Performance tiers** — a rolling frame-time monitor (`trackPerf`) steps quality down (fewer particles, no shadow-blur glow) on slow devices and back up when frame rate recovers.
- **Accessibility** — a reduced-motion toggle that defaults to `prefers-reduced-motion`, and an `aria-live` region (`srLive`, via `announce()`) for life loss, boss defeats, and game over.

## Controls (as documented in the in-game hint)

- Mouse: move to steer, hold/click to fire.
- Keyboard: Arrows or WASD to move, Space to fire, X or Shift to bomb, P or Esc to pause.
- Touch: drag left side to fly, tap FIRE, tap BOMB.

If you change a control, update the `controlsHint` text in the markup to match.

## Conventions and guidance

- **Keep it a single file.** The project is intentionally one self-contained `index.html` with no external dependencies. Don't introduce a build system, frameworks, or CDN scripts without an explicit request.
- **Preserve the style of the existing code.** Use `const`/`let`, the existing naming (`camelCase` functions, `UPPER_SNAKE` constants), and the existing section comments/ordering.
- **Respect the tuning tables.** Balance changes should go through `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES` rather than hard-coded numbers in update logic.
- **Respect accessibility settings.** New screen shake, flashes, or pulsing effects should honor the reduced-motion flag, and new player-facing state changes should be announced through `announce()` where appropriate.
- **Don't break mobile.** The layout uses `touch-action: none`, safe-area insets, and a joystick overlay. Test touch-related changes at phone width.
- **Keep `localStorage` keys stable** (see above) or add a migration if they must change.

## Verification

There is no automated test suite in the repo. Previous changes were verified with headless Playwright runs (driving the page, firing, checking collisions, boss spawns, and the console for errors). For gameplay changes:

1. Load `index.html` in a browser (or a headless Chromium) and start a run.
2. Exercise the affected path (e.g. weapons, bombs, boss transitions, pause, mute, reduced motion).
3. Confirm there are no console errors.

Check at least one difficulty and one touch-width viewport for changes that affect layout or controls.

## Git

- Default branch: `main`. Feature work is done on topic branches (for example `claude/...`).
- Write descriptive commit messages that explain what changed and how it was verified.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style
vertical shoot-'em-up. The entire game lives in a single file, `index.html`
(inline `<style>` and one inline `<script>` IIFE, no external assets or libraries).

Files:

- `index.html` — the complete game (markup, CSS, JavaScript).
- `README.md` — minimal project title.
- `CLAUDE.md` — this file.

## Running it

There is no build step, package manager, or test suite. Open `index.html` directly
in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Architecture of `index.html`

- **Layout:** a full-viewport flex container (`#container`) centers a `<canvas id="gameCanvas">`.
  HUD elements (`#hud`, `#weaponStat`, `#bombs`, `#bossBarWrap`, `#stageBanner`, etc.) are
  absolutely positioned DOM overlays on top of the canvas. Overlay screens (`#overlay`) handle
  the title, difficulty selection, pause, and game-over states.
- **Script:** one `(function () { ... })();` IIFE holds all state and logic. Keep new code inside
  it so nothing leaks to `window`.
- **Game loop:** `requestAnimationFrame`-driven; frame timing is tracked by `trackPerf`.
- **Configuration constants** (top of the script, near `DIFFICULTIES`):
  - `DIFFICULTIES` — `easy` / `normal` / `hard`, controlling spawn rate, enemy speed,
    enemy fire chance, starting lives and bombs, and boss HP multiplier.
  - `KILLS_PER_STAGE` — kills required to advance a stage.
  - `THEMES` — per-stage visual themes.
  - `ENEMY_TYPES` — `grunt`, `interceptor`, `tank`.
  - `BOSSES` — stage bosses, which are shown in `#bossBarWrap` while active.
- **Audio:** Web Audio API, lazily created by `ensureAudio()` on first user gesture. Sound effects
  are defined in the `sfx` object using `beep()` and `noiseBurst()` helpers. Mute is persisted.
- **Persistence:** `localStorage` only, with keys prefixed `skyfighter.`:
  `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`.
- **Accessibility:** `announce()` writes to a live region (`#srLive`) for screen readers. A
  reduced-motion mode is honoured from `prefers-reduced-motion` and can be toggled in the UI; it
  disables screen-flash effects (`body.reduced-motion #flash.bomb`).
- **Input:** keyboard (arrows/WASD move, Space fire, X or Shift bomb, P/Esc pause), mouse
  (move to steer, click to fire), and touch controls (`#touchControls`: drag-to-fly joystick,
  FIRE and BOMB buttons).

## Conventions

- Keep the game as a single self-contained HTML file unless the user asks for a restructure.
  Do not add npm dependencies, bundlers, or CDN scripts without asking.
- Match the existing style: 2-space indentation, `const`/`let`, single quotes in JS,
  `camelCase` identifiers, `SCREAMING_CASE` for tuning constants, and `#kebab-or-camel` element IDs.
- Put tuning values in the constant objects rather than hard-coding them in game logic.
- Preserve the `skyfighter.*` localStorage keys; changing them would reset players' saved data.
- Any new UI must work on phone-width screens (the page uses `touch-action: none` and
  `viewport-fit=cover`, and respects `env(safe-area-inset-*)`).
- Any new animation or flashing effect must respect the reduced-motion setting.
- Any new user-facing state change should be announced via `announce()`.

## Verifying changes

Because there are no automated tests, verify by actually running the game:

1. Open `index.html` in a browser (or serve it locally as above).
2. Start a game on each difficulty you touched, and check the HUD updates (score, hearts,
   bombs, weapon level, stage banner).
3. Exercise pause/resume, mute, reduced-motion toggle, and game over → restart.
4. For touch-related changes, test with a mobile viewport in browser devtools.

A headless browser (Playwright/Chromium) is a good way to take screenshots or smoke-test the
page without a display.

## Git

- Default development branch for this session: `claude/gifted-tesla-rmq27p`.
- Write clear, present-tense commit messages describing the gameplay or code change.
- Do not open pull requests unless explicitly asked.

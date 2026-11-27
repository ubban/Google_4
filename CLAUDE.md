# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small, dependency-free front-end project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a single HTML5 `<canvas>`.

| File        | Purpose                                                          |
|-------------|------------------------------------------------------------------|
| `index.html`| The entire game: inline CSS, markup, and one inline `<script>`. |
| `README.md` | Minimal project readme (title only).                             |

There is no build system, package manager, test suite, or linter. Do not add one unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

There is nothing to install or compile.

## Architecture of `index.html`

Everything lives in one file, in this order:

1. **`<head>`**: viewport meta (no zoom, `viewport-fit=cover`), theme color, and an inline SVG favicon. The page is full-screen, non-scrolling, and uses `touch-action: none`.
2. **`<style>`**: CSS for the canvas container, HUD (`#hud`, `.stat`, hearts, bombs, weapon and multiplier stats), vignette/flash overlays, and the pause and menu screens. Colors are hard-coded hex values.
3. **Markup**: the canvas (`#gameCanvas`), HUD elements, on-screen touch controls (joystick zone, fire button, bomb button), and the menu/pause/game-over overlays.
4. **`<script>`** (starts around line 422), organized by section comments:
   - **Persistence keys**: `STORAGE_*` constants in `localStorage` under the `skyfighter.*` namespace (best score, mute, difficulty, reduced motion).
   - **Adaptive performance**: `trackPerf()` samples the last 45 frame times and steps a `quality` level down or up. Check it before adding expensive per-frame effects.
   - **Tunables**: `DIFFICULTIES` (spawn rate, enemy speed, enemy fire chance, starting lives and bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`. Most balance changes belong here rather than in logic.
   - **Audio**: a lazily created `AudioContext` (`ensureAudio()`), with `beep()` and `noiseBurst()` helpers. Audio starts only after a user gesture.
   - **Input**: keyboard (`keys[]` map keyed by `e.code`), touch joystick, touch and mouse fire, and mouse aim on the canvas.
   - **Game state**: `resetGame()` initializes state. `update(dt)` is the per-frame simulation. `draw()` renders. `loop(now)` drives both via `requestAnimationFrame`.
   - **Gameplay systems**: enemy and boss spawning, power-ups (`applyPowerUp`), weapons (`fireWeapon`), bombs (`useBomb`), score and combo chain (`addScore`, `breakChain`), and lives (`loseLife`, `endGame`).
   - **Boss attacks**: `BOSS_PATTERNS` is a list of bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, `fireHomingSeeded`, and others). `updateBoss()` cycles through them.

The game loop runs `update(dt)`, then `draw()`. `dt` is clamped in the loop, so keep per-frame logic frame-rate independent by scaling with `dt`.

## Conventions

- **Keep it a single file** unless the user asks for a split. Do not introduce a bundler, framework, or npm dependency.
- **Vanilla JS only**, no modules, and no external CDN scripts or fonts. The page works offline as-is.
- **Match the existing style**: 2-space indentation, `const` by default, `let` for mutable state, arrow functions for event handlers, and section headers written as `// ---------- Name ----------`.
- **Tune numbers in the tunables block**, not inline, so balance changes stay in one place.
- **Preserve mobile support.** The layout must work at phone width, with touch controls and no page scroll. Test any HUD or layout change at small viewports.
- **Respect the reduced-motion setting.** It defaults to the OS `prefers-reduced-motion` value and can be toggled in-game. `screenShake()` already scales down when `reducedMotion` is set, so route new shake, flash, and similar effects through the same check.
- **Respect mute.** Route all sound through the existing helpers so the `muted` flag applies.
- **Accessibility:** `announce()` writes to the `srLive` live region for screen readers. Use it for important state changes such as stage start or game over.

## Verifying changes

There are no automated tests. For any gameplay or UI change:

1. Open `index.html` in a browser and play a few stages, covering both keyboard and touch paths if the change touches input.
2. Check the console for errors.
3. Check the affected difficulty levels, since most numbers vary by difficulty.
4. For layout changes, check a narrow phone-sized viewport as well as desktop.

If you have a headless browser available, a Playwright script that loads the page and checks for console errors is a reasonable smoke test. Do not commit such a script unless asked.

## Git workflow

- Develop on the branch assigned for the session. Do not push to other branches without permission.
- Do not open a pull request unless explicitly asked.
- Keep commits focused. Write messages in the imperative mood, describing the gameplay or UI change.

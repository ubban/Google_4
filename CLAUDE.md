# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs entirely in the browser. It is a single self-contained file:

- `index.html` — all markup, CSS (inline `<style>`), and JavaScript (inline `<script>`). There is no build step, no bundler, no package manager, and no external assets. Audio is synthesized with WebAudio.
- `README.md` — a stub. Not a source of truth for behavior.

There is no test suite, linter, or CI configured. Verify changes by loading `index.html` in a browser.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, e.g. `python3 -m http.server` and visit `http://localhost:8000/`. Nothing needs to be installed.

For automated checks, Playwright/Chromium is available in the cloud environment. Load the file with `file://` or a local server and drive it with the keyboard; confirm the canvas renders and the HUD updates.

## Code layout inside `index.html`

- **Head / CSS** (lines 1–359): theme colors, HUD (`#hud`, `#hearts`, `#bombs`, `#weaponStat`, `#multStat`), overlays (`#overlay`, `#overlayCard`), touch controls (`#touchControls`, `#joystickBase`, `#fireBtn`, `#bombBtn`), stage banner (`#stageBanner`), and a `body.reduced-motion` block that disables shake/flash/pulse effects.
- **Markup** (lines 361–421): the `<canvas id="gameCanvas">`, HUD, touch controls, and the start/pause/game-over overlay.
- **Script** (lines 422–1667): one block with these sections, in order:
  - DOM references and `STORAGE_*` keys (`skyfighter.bestScore`, `.muted`, `.difficulty`, `.reducedMotion`).
  - Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES` (6 sky palettes, cycled by stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), `BOSSES` (6 bosses, `VANGUARD` through `TITAN`), `BOSS_PATTERNS`.
  - Canvas sizing: `BASE_W = 480`, `BASE_H = 720`, scaled to fit the viewport (max 1.6×).
  - Input: `keydown`/`keyup`, touch joystick, fire and bomb buttons, mouse aim/fire on the canvas.
  - Audio (`ensureAudio`, `beep`, `noiseBurst`, `EXPLODE_SFX`).
  - Game state and loop: `resetGame`, `update`, `draw`, `loop` (driven by `requestAnimationFrame`), plus `trackPerf`, which adapts a `quality` level based on frame time.
  - Gameplay systems: enemy spawning and behavior, weapon firing (`fireWeapon` and the `fire*` pattern helpers), power-ups, bombs (`useBomb`), boss spawn/update/kill, combo chain (`addScore`, `breakChain`), stage progression, `endGame`.

Use `grep -n "function <name>" index.html` to navigate; line numbers drift as the file changes.

## Gameplay facts (for reasoning about changes)

- **Controls:** Arrow keys or WASD move (tracked via `keys[e.code]`); `Space` fires; `X` or either `Shift` drops a smart bomb; `P` or `Escape` pauses. Mouse moves the ship on the canvas and click fires. On touch devices, a floating joystick (drag left) moves the ship and dedicated Fire and Bomb buttons appear.
- **Difficulty** (`easy` / `normal` / `hard`) sets starting lives (4/3/3), starting bombs (3/2/2), spawn cadence, enemy speed, enemy fire chance, and a boss HP multiplier. It is persisted in `localStorage`.
- **Weapons:** two types, `vulcan` (default) and `laser`, toggled by collecting `weapon` power-ups. Fire delay is 0.15s for vulcan and 0.1s for laser.
- **Stages:** reaching `KILLS_PER_STAGE` (18) kills in a stage spawns that stage's boss (after a 1.2s delay). Killing the boss awards 500 points, increments `stage`, resets `stageKills`, and shows the next stage banner. The sky theme is `THEMES[(stage - 1) % THEMES.length]`. Bosses use `BOSSES[...]` for names/colors/HP and `BOSS_PATTERNS` for attacks.
- **Score:** kills award points by enemy type (`ENEMY_TYPES[...].score`), boss kills add 500, and the best score is persisted.
- **Persistence:** best score, mute, difficulty, and reduced-motion preference live in `localStorage`. Every access should stay safe if storage is unavailable.
- **Accessibility:** `reduced-motion` (user toggle, defaulting to the `prefers-reduced-motion` media query) suppresses screen shake, flashes, and pulse animations. Announcements go to the `#srLive` live region.
- **No device motion input** is implemented; the 🎆 button toggles reduced motion, not gyro controls.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not introduce a build step, framework, or npm dependency without an explicit request.
- Put tuning values in the existing constant tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Match the surrounding style: 2-space indentation, single quotes in JS, `const`/`let` (no `var`), and section comments in the form `// ---------- Name ----------`.
- Any new visual effect must respect `reducedMotion` (add a `body.reduced-motion` CSS rule or check the flag in JS).
- Any new user-facing status change should call `announce(...)` so screen readers get it.
- Keep the game playable on phone-sized viewports with touch controls; test touch paths when changing input.
- Keep `localStorage` access inside the existing `STORAGE_*` keys and wrap new access in try/catch if it could throw.

## Git and workflow

- Default working branch for this session is `claude/gifted-tesla-fwo67m`; push with `git push -u origin <branch>`.
- Do not open a pull request unless explicitly asked.
- Commit messages: imperative, descriptive subject line.

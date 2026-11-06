# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up written as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: inline `<style>`, markup, and one IIFE `<script>` (~1,670 lines). No build step, no bundler, no external assets (the favicon is an inline SVG data URI).
- `README.md` — a one-line title. Nothing else is documented there yet.

There is no `package.json`, test suite, linter, or CI configuration.

## Running and testing

- To play, open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests or lint commands. Verify changes by loading the page and playing it. Check desktop (mouse/keyboard) and, where relevant, touch layouts.
- Keep the file working without a build step. Do not introduce npm dependencies, frameworks, or a bundler unless the user asks for them.

## Architecture of `index.html`

The script runs inside a single IIFE. Key pieces, in source order:

1. **DOM references** — `document.getElementById` lookups are grouped at the top of the IIFE. Add new element references there.
2. **Storage keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all prefixed `skyfighter.`). Persistence goes through `localStorage`.
3. **Tuning tables** — these hold most of the game balance and are the first place to look for changes:
   - `DIFFICULTIES` — `easy` / `normal` / `hard`: spawn timing, enemy speed, shoot chance, starting `lives` and `bombs`, and `bossHpMul`.
   - `KILLS_PER_STAGE` — kills needed to advance a stage (18).
   - `THEMES` — six sky palettes, cycled per stage.
   - `ENEMY_TYPES` — `grunt`, `interceptor`, `tank`, with size, HP, speed, score, and the `minStage` each one first appears in.
   - `BOSSES` — six boss designs, chosen by `(stage - 1) % BOSSES.length`, each with a palette and an `hpMul`.
4. **Audio** — a lazily created `AudioContext` (`ensureAudio`), with `beep`, `noiseBurst`, and a named `sfx` map. Audio is created only after a user gesture.
5. **Input** — keyboard (`keys` map, keyed by `e.code`), mouse steering and fire on the canvas, and touch controls (a virtual joystick on the left, plus FIRE and BOMB buttons).
6. **Game state** — `player`, enemies, bullets, power-ups, and bosses are plain objects and arrays reset by `resetGame()`.
7. **Update / draw** — `update(dt)` and `updateBoss(dt)` advance the simulation. `draw()` renders everything, and `loop(now)` drives both through `requestAnimationFrame`. `dt` is clamped to 0.05 s.
8. **Resize** — the canvas uses a fixed logical size (`BASE_W` = 480, `BASE_H` = 720) and is scaled to fit the window, capped at 1.6×.

### Gameplay facts worth knowing

- **Weapons**: `vulcan` and `laser`, toggled by a weapon power-up. Each has a level (`player.level`).
- **Power-ups**: `life`, `bomb`, and `weapon`, dropped by enemies with `maybeSpawnPowerUp`.
- **Bombs**: `useBomb()` clears the screen and damages bosses. It is triggered by X, Shift, or the on-screen BOMB button.
- **Score**: uses a kill-chain multiplier (`mult`). The best score is persisted.
- **Pause**: P or Escape.
- **Accessibility**: a reduced-motion setting, persisted, that defaults to the user's `prefers-reduced-motion` media query. Screen-reader announcements go through `announce()` and the `srLive` region. Keep both working when you add UI.

## Conventions

- **Single file**: keep the game in `index.html`. Put new tuning values in the existing tables rather than hard-coding them in logic.
- **Style**: match the existing 2-space indentation, `const`/`let`, arrow callbacks, and the naming of existing helpers (`spawnX`, `renderX`, `flashX`, `showX`).
- **Mobile first**: the layout must still work with touch controls and the safe-area insets. Do not add `hover`-only interactions without a touch equivalent.
- **Performance**: the loop runs every frame. Avoid allocating large objects or arrays per frame in hot paths. `trackPerf` already monitors frame time.
- **Persistence**: wrap new `localStorage` access in the same pattern the file already uses. Guard against it being unavailable.

## Git workflow

- Develop on the branch named in the session instructions (currently `claude/gifted-tesla-lcwtmm`). Do not push to `main` directly.
- Write clear, descriptive commit messages in the imperative mood, with a short subject line and a body explaining the "why" when it isn't obvious.
- Do not open a pull request unless the user explicitly asks for one.

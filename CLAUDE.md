# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-file HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up.

- The entire app lives in `index.html`: markup (`<body>`), CSS (one `<style>` block), and game logic (one inline `<script>` wrapped in an IIFE).
- `README.md` is a placeholder and has no setup instructions.
- There is no build step, package manager, bundler, test suite, or linter. Do not add one unless asked.

## Running and checking changes

- Open `index.html` directly in a browser to run the game. No server is required.
- Syntax-check the game script after edits. The inline script has no imports, so this works:
  ```bash
  S=$(mktemp -d)
  sed -n '/^<script>$/,/^<\/script>$/p' index.html | sed '1d;$d' > "$S/game.js"
  node --check "$S/game.js"
  ```
- For behavior changes, play the affected path in a browser (desktop and, if touch-related, a touch emulation). There are no automated tests to rely on.

## Code layout inside `index.html`

The script is grouped by comment headers and roughly follows this order:

1. **DOM references**: `const` lookups by id at the top of the IIFE. Element ids must match the markup in `<div id="container">`, `#hud`, `#overlay`, and `#touchControls`.
2. **Storage keys**: `STORAGE_*` constants prefixed `skyfighter.` (best score, mute, difficulty, reduced motion). Preserve these names so saved player data keeps working.
3. **Tuning tables**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS`. Most balance changes belong here rather than in logic.
4. **Audio**: a Web Audio `sfx` object built from `beep` and `noiseBurst`. Audio is created lazily through `ensureAudio()`, which browsers require to start after a user gesture.
5. **Input**: keyboard (`keys` map plus P/Esc pause, X/Shift bomb), touch joystick and buttons, and mouse steering on the canvas.
6. **Game state and loop**: `resetGame`, `update(dt)`, `draw()`, and `loop(now)`. `dt` is clamped to 0.05 s. `trackPerf` adapts a `quality` level (0–2) to frame times.
7. **Boss logic**: `spawnBoss`, `BOSS_PATTERNS` (each boss has its own bullet pattern), `updateBoss`, and `killBoss`.

Key game facts:

- Canvas base size is 480×720. `resize()` scales the canvas with CSS to fit the viewport, up to 1.6×.
- Weapons: `vulcan` and `laser`, each with power levels 1–3. Collecting a weapon power-up past level 3 switches weapon type and resets to level 1.
- Lives and bombs depend on difficulty (`lives`, `bombs`, and `bossHpMul` in `DIFFICULTIES`).
- Stages: the stage number selects a theme and boss via `(stage - 1) % length`. A stage ends after `KILLS_PER_STAGE` kills and spawns a boss.
- Score uses a kill-chain multiplier (`chain`, `chainTimer`, `multiplier`) that `breakChain()` resets.
- Best score is saved to `localStorage` only when a game ends with a new high score (`endGame`).

## Conventions

- **Keep it a single file.** New styles go in the existing `<style>` block and new logic goes in the existing IIFE. Do not split out modules or external assets unless asked.
- **Match the surrounding code.** Use the same style as the code you touch: `const`/`let`, arrow functions for callbacks, 2-space indentation, single quotes in JS, and `// ---------- Section ----------` headers for new sections.
- **Keep the game loop allocation-light.** `update` and `draw` run every frame. Avoid creating new objects or closures there when a reused value works, and respect the `quality` level for expensive effects.
- **Accessibility.** Preserve the `prefers-reduced-motion` handling (`reducedMotion`, `body.reduced-motion`), the `aria-live` announcements via `announce()`, and the `sr-only` status region. Keyboard and touch controls must keep working.
- **Mobile.** Touch input is enabled only when `isTouchDevice` is true. Keep `touch-action: none` and the `passive: false` handlers that call `preventDefault`.
- **Comments.** Keep comments short and explain why, not what. Only add a comment where the logic is not obvious.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-514371`). Do not push to `main` without explicit permission.
- Make focused commits with descriptive messages that say what changed and why.
- Do not open a pull request unless explicitly asked.

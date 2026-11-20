# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free browser game called **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game lives in a single file, `index.html` (inline CSS, inline JavaScript, Canvas 2D rendering, Web Audio sound effects). There is no build step, no package manager, and no test suite.

Files:

- `index.html` — the whole game (markup, styles, and the game script inside one IIFE).
- `README.md` — minimal project readme.
- `CLAUDE.md` — this file.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests or linters configured. Verify changes by playing the game in a browser. For UI or gameplay changes, take a screenshot or drive it with Playwright (Chromium is preinstalled in the cloud environment; see the environment notes for the launch path).

## Code structure (inside `index.html`)

The game script is one `(function () { ... })();` block. Key sections, roughly in file order:

- **DOM references** — `document.getElementById` lookups for the HUD, overlays, touch controls, and buttons.
- **Persistence keys** — `localStorage` keys prefixed `skyfighter.` for best score, mute, difficulty, and reduced motion.
- **Tuning tables** — `DIFFICULTIES` (spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six boss designs, cycled by stage).
- **Audio** — lazily created `AudioContext` (`ensureAudio`), with `beep`, `noiseBurst`, and a `sfx` table. Respect the `muted` flag.
- **HUD rendering** — hearts, bombs, score, multiplier, weapon name and level, boss health bar.
- **Game loop and entities** — player, bullets, enemies, bosses, power-ups, explosions, and particles, updated and drawn each frame.
- **Input** — mouse, keyboard (arrows/WASD, Space fire, X or Shift bomb, P/Esc pause), and touch (joystick-style drag to fly, FIRE and BOMB buttons).

When adding content, prefer extending these tables (`DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`) over hard-coding values in the loop.

## Conventions

- Keep the project a single self-contained HTML file unless the user asks for a split. Do not add npm dependencies, bundlers, or external CDN scripts without asking.
- Match the existing style: 2-space indentation, `const` by default, `let` where reassignment is needed, and descriptive camelCase names. Tuning values are plain object literals at the top of the script.
- Preserve accessibility and responsiveness: the page sets `touch-action: none`, supports `prefers-reduced-motion` (`reducedMotion` flag, `body.reduced-motion` class), and uses an `aria-live` region (`announce()`) for screen-reader messages. Keep these working when touching related code.
- Keep mobile layout working. The HUD and touch controls use `env(safe-area-inset-*)` and `clamp()` sizes; check phone-width output when changing layout.
- Do not reintroduce dark-on-dark or low-contrast text; the palette is dark-themed (`color-scheme: dark`).

## Git

- Default development branch for this work: `claude/gifted-tesla-ytni5v`. Commit and push there; do not push to other branches without explicit permission.
- Do not open a pull request unless the user explicitly asks for one.
- Commit messages should be short and imperative, following the existing history (for example, "Add Sky Fighter: Raiden-style vertical shoot-em-up game").

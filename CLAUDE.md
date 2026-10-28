# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only game is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up that runs entirely in one file:

- `index.html` — the whole app: inline `<style>`, markup, and one inline
  `<script>` IIFE that owns all game state, rendering, input, audio, and storage.
- `README.md` — a placeholder; there is no other documentation yet.

There is no build step, no package manager, no bundler, no test suite, and no
linter configuration. Open `index.html` in a browser to run it.

## Repository layout

```
.
├── index.html   # Sky Fighter (CSS + HTML + JS, all inline)
└── README.md
```

Keep it flat unless there is a clear reason to split. If the game grows, prefer
separate `.js`/`.css` files loaded with plain `<script src>` / `<link>` tags so
the project still runs by opening `index.html` directly, with no build step.

## Running and verifying changes

- Run: open `index.html` in a browser (or serve the directory with any static
  server, e.g. `python3 -m http.server`).
- There are no automated tests. For any gameplay or UI change, load the page
  and exercise the affected path by hand: start a game, fire, use a bomb, pause,
  lose a life, and check the game-over overlay. Check both desktop (keyboard and
  mouse) and a narrow viewport (touch controls appear).
- Check the browser console for errors after your change; the game should run
  without exceptions.

## Code architecture (inside `index.html`)

The script is wrapped in an IIFE with `'use strict'`. Keep new code inside that
wrapper so it does not leak globals.

Major sections, in the order they appear in the script:

1. **DOM lookups** — every element is fetched once by `id` at the top. Add new
   references there, not inside functions.
2. **Storage keys** — `skyfighter.bestScore`, `skyfighter.muted`,
   `skyfighter.difficulty`, `skyfighter.reducedMotion`. Persisted values are
   read with `localStorage.getItem` and written with `setItem`.
3. **Adaptive performance** — `trackPerf` samples frame times and steps a
   `quality` level (0 full, 1 medium, 2 low) up or down to hold frame rate.
4. **Tuning tables** — `DIFFICULTIES`, `THEMES` (per-stage sky palettes),
   `ENEMY_TYPES`, `BOSSES`, plus `KILLS_PER_STAGE`. Balance changes belong here
   rather than in logic.
5. **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers.
   Audio is created lazily on the first user gesture (browsers block autoplay).
6. **UI feedback** — hearts/bombs rendering, combo pops, stage banner, screen
   shake, hit and bomb flashes, and the `announce` helper for screen readers.
7. **Input** — keyboard (`keys` map keyed by `KeyboardEvent.code`), mouse
   steering and click-to-fire on the canvas, and touch joystick plus FIRE/BOMB
   buttons. Touch handlers call `preventDefault` with `{ passive: false }`.
8. **Game state** — `resetGame`, `update(dt)`, `draw()`, and `loop(now)`.
   `loop` clamps `dt` to 50 ms and calls `update` then `draw` only while
   `running && !paused`.
9. **Gameplay systems** — weapons (`fireWeapon`, power-up levels), bombs
   (`useBomb`), scoring with kill-chain multiplier (`addScore`, `breakChain`),
   enemy and boss spawning, and the boss attack patterns in `BOSS_PATTERNS`.
10. **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, and the main
    `draw`. Rendering is canvas 2D only; there are no sprite assets.
11. **Lifecycle** — `loseLife`, `endGame`, `togglePause`, and the start button
    handler at the bottom.

Game coordinates use a fixed logical size of `BASE_W × BASE_H` (480 × 720);
`resize()` scales the canvas to fit the window. Gameplay code should work in
logical coordinates and never read the CSS size directly.

## Conventions

- **Style:** match the existing code. Two-space indentation, single quotes in
  JavaScript, semicolons, `const` by default, `let` only when reassigned, and
  arrow functions for callbacks. Keep comments short and use the existing
  `// ---------- Section ----------` divider style for new sections.
- **Naming:** `UPPER_SNAKE_CASE` for tuning constants and storage keys,
  `camelCase` for functions and variables, and `STORAGE_*` for keys that
  persist to `localStorage`.
- **Persistence:** any new user setting must use a `skyfighter.*` key and
  should default sensibly when the key is missing. Wrap nothing extra in
  try/catch unless you have a concrete reason, since the existing code does not.
- **Accessibility:** keep `aria-live` announcements (`announce`) for important
  state changes, and respect `reducedMotion` for any new screen shake, flash,
  or pulse effect.
- **Mobile:** new interactive controls need touch handlers as well as mouse
  and keyboard ones, and must not conflict with the joystick zone.
- **Balance:** change numbers in the tuning tables instead of hard-coding them
  in update or draw logic.

## Working in this repo

- Work on the branch you were asked to use. Do not push to `main` unless
  explicitly told to.
- Keep commits focused. Describe the gameplay or UI effect in the commit
  message, not just the mechanism.
- Do not add dependencies, frameworks, or a build step without asking first;
  the single-file, zero-build design is intentional.
- Do not commit generated files, editor state, or `node_modules`.

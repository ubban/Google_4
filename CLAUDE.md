# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small static web project. Its only real content is **Sky Fighter**, a Raiden-style
vertical shoot-'em-up, implemented as a single self-contained page: `index.html`
(about 1,670 lines: inline CSS in `<style>`, inline JavaScript in one `<script>` IIFE).
`README.md` is a placeholder and has no project documentation.

There is no build step, no package manager, no test suite, no linter, and no server
code. Nothing needs to be installed.

## Running and checking changes

- Open `index.html` directly in a browser, or serve the directory with any static
  server (for example `python3 -m http.server`). No build is needed.
- There are no automated tests. Verify gameplay changes by playing in a browser.
  Check both desktop (keyboard) and touch layouts, since the UI has touch controls and
  a phone-width layout.

## Architecture of `index.html`

Everything lives inside `(function () { 'use strict'; ... })()`, so nothing is exposed
on `window`. Keep new code inside that IIFE.

- **DOM references** are grabbed once at the top by element ID (`gameCanvas`,
  `overlay`, `startBtn`, `fireBtn`, `bombBtn`, `pauseBtn`, `muteBtn`, `motionBtn`,
  `bossBarWrap`, and so on). If you rename or remove an ID in the markup, update the
  matching `getElementById` call too.
- **Persistence** uses `localStorage` with the keys `skyfighter.bestScore`,
  `skyfighter.muted`, `skyfighter.difficulty`, and `skyfighter.reducedMotion`. Keep these
  keys stable, because they hold players' saved settings and high score. Reads and
  writes are not wrapped in try/catch; if you touch them, consider what happens when
  storage is blocked.
- **Config tables** near the top of the script drive most of the balance:
  - `DIFFICULTIES`: spawn timing, enemy speed, shoot chance, starting `lives` and `bombs`,
    and `bossHpMul`.
  - `THEMES`: the six sky palettes, one per stage. Stages cycle through them.
  - `ENEMY_TYPES`: `grunt`, `interceptor`, `tank`, with size, HP, speed, score, and
    `minStage` (which stage each type first appears in).
  - `BOSSES`: six bosses (VANGUARD through TITAN), each with colors and an `hpMul`.
    Bosses and themes both repeat by stage using `(stage - 1) % length`.
  - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
  - `BOSS_PATTERNS`: the bullet-pattern functions bosses use (radial, aimed spread,
    columns, spiral, cross, homing, double ring).
- **Game loop**: `loop(now)` runs every animation frame. It clamps `dt` to 0.05 s, calls
  `trackPerf`, then `update(dt)` and `draw()` while the game is running and not paused.
  `update` handles input, movement, firing, collisions, and spawning. `draw` renders
  everything to the 2D canvas.
- **Adaptive quality**: `trackPerf` watches frame times over 45 frames and steps a
  `quality` level (0 full, 1 medium, 2 low) up or down. Drawing code should respect it,
  and so should any new per-frame effects.
- **Canvas sizing**: the logical play area is `BASE_W` × `BASE_H` (480 × 720). `resize()`
  scales the canvas to fit the viewport. Coordinates in game logic use the logical size,
  not CSS pixels.
- **Audio**: a Web Audio context is created lazily by `ensureAudio()`, which must run
  from a user gesture (browsers block audio otherwise). Sound effects go through
  `beep` / `noiseBurst` and the `sfx` helpers. Respect `muted`.
- **Reduced motion**: `reducedMotion` is read from storage, falling back to the
  `prefers-reduced-motion` media query. Screen shake, flashes, and similar effects should
  check it.
- **Accessibility**: `announce()` writes to the `srLive` live region for screen readers.
  Use it for important state changes such as stage starts, boss arrival, and game over.

## Controls (keep in sync with the UI text)

- Move: arrow keys or WASD, or drag the on-screen joystick / the mouse on the canvas.
- Fire: Space (held), or the fire button.
- Smart bomb: X or Shift, or the bomb button.
- Pause: P or Escape, or the pause button.

Space's default browser action is suppressed in `keydown`, so the page does not scroll.

## Conventions

- Plain HTML, CSS, and ES2015+ JavaScript. No frameworks, libraries, or CDN loads.
  Keep it that way unless the user asks otherwise.
- Keep the file self-contained. Add new styles to the existing `<style>` block and new
  logic inside the existing IIFE.
- Use `const` by default and `let` where reassignment is needed. Match the surrounding
  style: two-space indents, single quotes, and section comments like
  `// ---------- Name ----------`.
- Keep the layout working at phone width with no horizontal page scroll. The page
  uses `viewport-fit=cover` and a dark-blue `theme-color`.

## Git

- Development branch: `claude/gifted-tesla-dq3z7z`. Commit and push there.
- The git history is short (two commits); there are no branch-protection or release
  conventions to follow.

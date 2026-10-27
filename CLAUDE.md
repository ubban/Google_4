# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only game is **Sky Fighter**, a Raiden-style
vertical shoot-'em-up that runs as a single-page HTML5 canvas app.

- `index.html` (~1,670 lines) — the entire game: inline `<style>`, markup, and one
  IIFE `<script>` containing all game logic, rendering, input, and audio.
- `README.md` — a one-line title. There is no other documentation.

There is no build step, package manager, bundler, test suite, or linter configuration.
Do not add tooling unless the user asks for it.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server
(for example `python3 -m http.server`). No dependencies are fetched from a CDN.

## Verifying changes

There are no automated tests. For behavior changes, verify in a real browser:

- Load the page and check the console for errors.
- Playwright is the practical option for headless checks: launch Chromium, load
  `file://…/index.html` (or a local server), drive the game with keyboard or mouse
  events, and read the DOM and canvas state. The previous feature work was validated
  this way (firing, collisions, boss patterns, stage advance, game over).
- If you cannot run a browser, say so explicitly and limit claims to what you verified
  by reading the code.

## Code layout inside `index.html`

The script is one IIFE. Keep new code inside it so it stays out of the global scope.

Top-to-bottom, the script is organized roughly as:

1. **DOM refs** — `getElementById` lookups for HUD, overlay, touch controls, and buttons.
2. **Storage keys** — `skyfighter.bestScore`, `skyfighter.muted`,
   `skyfighter.difficulty`, `skyfighter.reducedMotion`. Changing a key orphans saved
   player data, so treat these as a stable interface.
3. **Adaptive performance** — `trackPerf` steps a `quality` tier (0 full, 1 medium,
   2 low) from a rolling frame-time average. Lower tiers trim particles and disable
   shadow-blur glow.
4. **Tuning tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`,
   `BOSSES`. Most balance changes belong here rather than in the logic.
5. **Audio** — a lazily created `AudioContext` (`ensureAudio`) feeding a master gain
   and compressor. Sounds are built from `beep` (oscillator) and `noiseBurst`
   helpers with optional stereo pan from on-screen x position.
6. **Settings** — mute, reduced motion (defaults to `prefers-reduced-motion`), and
   difficulty, each persisted to `localStorage`.
7. **Entities and spawning** — `spawnEnemy`, `spawnBoss`, `spawnExplosion`,
   `fireWeapon`.
8. **Update and boss attacks** — `update(dt)` runs game logic; `updateBoss` and the
   `fire*` pattern helpers (radial, aimed spread, columns, spiral, cross, homing,
   double ring) implement boss bullet patterns.
9. **Game flow** — `resetGame`, `endGame` (saves best score), `togglePause`, `useBomb`.
10. **Rendering** — `draw()` and helpers (`drawPlane`, `drawBossShip`, `drawClouds`).
    Reduced motion and quality tier gate visual effects here.
11. **Input** — keyboard (`keys` map, `KeyP`/`Escape` pause, `KeyX`/`Shift` bomb),
    mouse steering and fire on the canvas, and touch joystick and buttons.
12. **Main loop** — `loop(now)` clamps `dt` to 50 ms, then calls `update` and `draw`
    while running and not paused.

## Game rules worth knowing

- Difficulty sets lives, bombs, spawn rate, enemy shot chance, and boss HP multiplier.
- A stage ends after `KILLS_PER_STAGE` kills, then a boss fights. Bosses cycle through
  the `BOSSES` roster by stage, and backgrounds cycle through `THEMES`.
- Enemy types are gated by `minStage` (`tank` appears from stage 2).
- Score uses a kill-chain multiplier.
- Accessibility: `aria-live` region `srLive` announces life loss, boss defeats, and
  game-over. Anything new the player should hear should go through `announce()`.

## Conventions

- Match the existing style: 2-space indentation, single quotes in JS, `const` by default,
  short section banners in the form `// ---------- Name ----------`.
- Keep the file self-contained. No new external scripts, stylesheets, or fonts unless
  the user asks.
- Touch-friendly layout matters. The page sets `touch-action: none` and uses
  `safe-area-inset` padding. Check phone-sized viewports when changing HUD or overlay
  layout.
- Preserve the `prefers-reduced-motion` and quality-tier paths when adding visual
  effects. New shake, flash, or pulse effects must respect `reducedMotion`.

## Git

- Develop on the branch the task names. Do not push to `main` without explicit permission.
- Commit messages in this repo are descriptive, with a summary line and body
  paragraphs explaining the why and what was verified.

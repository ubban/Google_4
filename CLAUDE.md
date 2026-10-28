# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a single HTML5 canvas.

- `index.html` — the entire game: markup, CSS, and JavaScript in one file (~1,670 lines).
- `README.md` — minimal project title; no setup instructions yet.

There is no build step, package manager, bundler, test suite, or linter. The game runs by opening `index.html` directly in a browser.

## Layout of `index.html`

1. **`<style>` (lines ~10–360)** — all styling, including HUD, overlay, touch controls, and the responsive canvas layout.
2. **`<body>` (lines ~361–421)** — static DOM: the `#gameCanvas`, HUD elements (`#hud`, `#score`, `#best`, `#mult`, `#hearts`, `#bombs`, weapon labels), boss bar (`#bossBarWrap`), stage banner, touch controls (`#touchControls`, `#fireBtn`, `#bombBtn`), and the start/pause/game-over overlay (`#overlay`).
3. **`<script>` (lines ~422–1668)** — one IIFE wrapping all game logic. Key sections, in order:
   - DOM references and `localStorage` keys (`skyfighter.*`)
   - Adaptive quality (`trackPerf`, `quality` 0–2) that lowers effects when frame time is high
   - Config tables: `DIFFICULTIES`, `THEMES` (stage backgrounds), `ENEMY_TYPES`, `BOSSES`
   - Audio: lazily created `AudioContext` (`ensureAudio`), `beep`, `noiseBurst`
   - Input: keyboard (`keys` map by `e.code`), virtual joystick, touch fire/bomb buttons, mouse on canvas
   - Game state and loop: `resetGame`, `update(dt)`, `draw()`, `loop(now)`
   - Spawning and combat: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `addScore` / combo chain
   - Boss attack patterns: `BOSS_PATTERNS` and `fire*` helpers, driven by `updateBoss`
   - Rendering helpers: `drawPlane`, `drawBossShip`, `drawClouds`

## Controls (for reference when testing)

| Action | Keyboard | Touch |
|---|---|---|
| Move | Arrow keys or WASD | Left-side joystick |
| Fire | Space (hold) | Fire button (hold) |
| Bomb | X or Shift | Bomb button |
| Pause | P or Escape | Pause button |

Difficulty (`easy` / `normal` / `hard`), mute, and reduced-motion settings persist in `localStorage`, as does the best score.

## Conventions

- **Keep it one file.** The project is deliberately dependency-free. Do not add a build tool, framework, or external script unless the user asks for it.
- **Match the existing style.** Use `const`/`let`, arrow functions for callbacks, two-space indentation, and single quotes in JS. Keep the section banners (`// ---------- Name ----------`) for new sections.
- **Add new game content through the config tables.** New enemies, bosses, themes, and difficulty tweaks belong in `ENEMY_TYPES`, `BOSSES`, `THEMES`, and `DIFFICULTIES`, not hard-coded in logic.
- **Keep the frame loop cheap.** `update` and `draw` run every frame. Avoid per-frame allocations in hot paths, and respect `quality` and `reducedMotion` when adding visual effects.
- **Wrap storage access sensibly.** `localStorage` reads currently aren't guarded with try/catch. If you touch that code, consider making it resilient to private-mode or blocked-storage errors.
- **Keep the page usable on phones.** Any new HUD or control must work with the touch layout and the `BASE_W` × `BASE_H` (480 × 720) scaling in `resize()`.
- **Accessibility.** Announce important state changes through the `#srLive` region via `announce()`, and keep the reduced-motion path working.

## Verifying changes

Since there are no automated tests:

1. Open `index.html` in a desktop browser and play through at least one stage to a boss.
2. Check touch controls in a mobile-width viewport (browser dev tools device mode).
3. Confirm pause, mute, difficulty switching, and game over → restart all work.
4. Confirm best score persists after a reload.

If you add logic that is easy to get wrong, consider extracting it into a pure function so it can be checked in isolation in the browser console.

## Git

- Work on the branch you were given; don't push to other branches.
- Commit messages are short imperative summaries (see `git log`, e.g. "Add Sky Fighter: Raiden-style vertical shoot-em-up game").

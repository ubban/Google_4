# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up built as a single-page HTML5 canvas game.

- `index.html` — the entire game: markup, CSS (inline `<style>`), and JavaScript (inline `<script>`). Roughly 1,670 lines.
- `README.md` — minimal project readme.

There is no build step, package manager, bundler, test suite, or server requirement. Open `index.html` directly in a browser, or serve the directory with any static server.

## Working in this repo

- Keep the game as a single self-contained file unless the user asks for a split. Do not introduce a build tool, framework, or npm dependencies without explicit approval.
- Do not add external CDN scripts or fonts; the game currently relies only on system font stacks and inline data URIs.
- Preserve the existing code organization inside the `<script>` block: constants (`DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`) are grouped near the top, state is declared with `let`, and game logic is organized as named functions (`update`, `resetGame`, `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `applyPowerUp`, draw helpers such as `drawPlane` and `drawBossShip`).
- Game loop logic lives in `update(dt)` and the render path; keep frame-rate-independent timing based on `dt`.

## Game architecture notes

- **Canvas sizing:** logical resolution is `BASE_W × BASE_H` (480 × 720). `resize()` scales the canvas to fit the viewport, including on `orientationchange`.
- **Stages:** each stage ends after `KILLS_PER_STAGE` (18) kills, which triggers a boss from `BOSSES`, each with attack patterns in `BOSS_PATTERNS`. Stage number feeds into `bossNameEl` and the stage banner.
- **Difficulty:** selected via `DIFFICULTIES` entries; the active one is read in spawn, update, and reset paths via `DIFFICULTIES[difficulty]`.
- **Persistence (localStorage):** keys are namespaced `skyfighter.*`:
  - `skyfighter.bestScore`
  - `skyfighter.muted`
  - `skyfighter.difficulty`
  - `skyfighter.reducedMotion`
  
  Any new persisted setting should follow this `skyfighter.<camelCaseName>` convention. Storage reads and writes should tolerate unavailable storage (private mode, blocked site data).
- **Audio:** Web Audio synthesized via `beep()` and `noiseBurst()`; audio context is created lazily in `ensureAudio()`, which must run from a user gesture. Respect the mute setting.
- **Input:** keyboard (`keys` map keyed by `e.code`), mouse on the canvas, and touch controls (virtual joystick in the move zone, fire and bomb buttons). Touch handling uses `preventDefault` with `{ passive: false }`.
- **Accessibility:** the reduced-motion setting defaults to the `prefers-reduced-motion` media query and can be toggled in-game. It scales down screen shake (`screenShake`) and flashes, and sets a `reduced-motion` class on `<body>`. Respect it in any new visual effects.

## Conventions

- Plain HTML, CSS, and vanilla JavaScript (ES2015+ syntax is already in use). No transpilation.
- Match the existing indentation (2 spaces) and quoting style (single quotes in JS).
- Keep the mobile layout working: the page uses `viewport-fit=cover`, safe-area insets, and `touch-action: none`. Test layout changes at phone width.
- Commit messages should be short and imperative; describe gameplay or UI changes in the body when they are non-trivial.

## Testing and verification

There is no automated test suite. To verify a change:

1. Open `index.html` in a desktop browser and play through at least one full stage, including the boss.
2. Check the mobile/touch layout with browser device emulation (joystick, fire, and bomb buttons).
3. Confirm persisted settings (mute, difficulty, reduced motion, best score) survive a page reload.
4. Check the browser console for errors.

## Git

- Development branch for this work: `claude/gifted-tesla-v9t3oh`. Commit and push there; do not push to other branches without explicit permission.
- Do not open pull requests unless the user asks.

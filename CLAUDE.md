# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: markup, CSS (`<style>`), and JavaScript (`<script>`). No build step, no dependencies, no package manager.
- `README.md` — minimal project title.

There is no test suite, linter, or bundler configured. Verify changes by opening `index.html` in a browser (or serving the directory with any static server) and playing through the affected behavior.

## Repository layout

```
index.html   # Full game (≈1,670 lines): HTML overlay/HUD, <style>, and one IIFE <script>
README.md
```

Inside `index.html`, the script is wrapped in a single `(function () { 'use strict'; ... })();` IIFE. Keep new code inside that wrapper; do not add globals.

## Code structure (inside the `<script>` block)

Sections are marked with `// ---------- Name ----------` comments. Follow that convention when adding a new section.

- **DOM references** — `const xEl = document.getElementById(...)` at the top.
- **Storage keys** — `STORAGE_*` constants under `skyfighter.*` in `localStorage`.
- **Adaptive performance** — `quality` (0 full / 1 medium / 2 low) is adjusted from rolling frame times.
- **Tuning tables** — `DIFFICULTIES`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), `BOSSES` (six boss skins), `KILLS_PER_STAGE`. Prefer adding or adjusting values here rather than hard-coding numbers in logic.
- **Audio** — `ensureAudio()`, `beep()`, `noiseBurst()`, and a `sfx` object of named sounds. Audio is created lazily on the first user gesture; keep it that way for autoplay policies.
- **Input** — keyboard (`keys` map, `keydown`/`keyup`), mouse (canvas steering and click-to-fire), and touch (`joystick*` for movement, `fireBtn`, `bombBtn`).
- **Game state and logic** — `resetGame()`, `update(dt)`, spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), weapons (`fireWeapon`, `useBomb`), scoring with a kill-chain multiplier (`addScore`, `breakChain`), and lives (`loseLife`, `endGame`).
- **Bosses** — `BOSS_PATTERNS` are bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring) selected per boss phase in `updateBoss`.
- **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`. Drawing is pure canvas 2D; keep it free of game-state mutation.
- **Main loop** — `loop(now)` drives `update` and `draw` with a delta-time clamp; pause is handled by `togglePause()`.

## Game rules to preserve

- **Lives and bombs** come from the difficulty table. Easy: 4 lives / 3 bombs; Normal and Hard: 3 lives / 2 bombs.
- **Weapons**: Vulcan and Laser, each with power levels 1–3, upgraded by `weapon` power-ups.
- **Power-up drop odds** (rolled once per destroyed enemy in `maybeSpawnPowerUp`): life ≈ 6%, bomb ≈ 8%, weapon ≈ 20%. Keep the type strings `life`, `bomb`, `weapon` in sync between spawn, `applyPowerUp`, and the render color lookup in `draw()`.
- **Stages**: once `stageKills` reaches `KILLS_PER_STAGE` (18), a boss is announced and spawned about 1.2 s later. Defeating the boss increments `stage` (see `killBoss`) and the boss health bar (`bossBarWrap`) is shown while a boss is alive.
- **Persistence**: best score, mute, difficulty, and reduced-motion preference are saved to `localStorage`. Every storage access is currently unguarded; if you add more, keep the same key prefix (`skyfighter.`).
- **Accessibility**: `announce()` writes to the `srLive` live region for screen readers. Use it for important state changes (stage start, boss, game over). Respect `reducedMotion`: `screenShake()` scales magnitude down and caps duration, and the hit/bomb flash helpers return early.
- **Responsive scaling**: the logical play field is `BASE_W × BASE_H` (480 × 720), scaled by `resize()`. Coordinates in game logic use this logical space, not CSS pixels.

## Conventions

- Plain ES2015+ JavaScript, `const` by default, arrow functions for callbacks, `'use strict'` on.
- Match the existing style: 2-space indentation, single quotes in JS, semicolons on statements, and short `//` section comments.
- Keep everything in one file unless there is a clear reason to split it. If you do split, update this file's layout section.
- Do not introduce frameworks, bundlers, or external CDN dependencies without asking first. The game is intentionally a zero-build single page.
- Keep the HTML `<head>` meta tags (viewport, theme-color, description) and the inline SVG favicon intact.
- Preserve mobile behavior: touch controls appear only when `isTouchDevice` is true, and the viewport disables user scaling.

## Testing and verification

No automated tests exist. Before committing a gameplay change:

1. Open `index.html` in a desktop browser and play one full run on Normal.
2. Check the touched system directly: for example, weapon levels for weapon changes, boss phases for boss changes, and the HUD for UI changes.
3. Check the mobile layout at a phone-sized viewport (touch controls visible, HUD not overlapping the play field).
4. Check the console for errors.

If you cannot run a browser, say so explicitly in your summary rather than claiming the change was tested.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-c2lr7p`); do not push to `main` directly.
- Write descriptive commit messages in the imperative mood, with a short summary line and a body that explains the gameplay or UI effect.
- Do not open a pull request unless explicitly asked.

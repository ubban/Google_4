# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a 2D `<canvas>`. The whole game lives in one file, `index.html`: markup (~lines 1–421), CSS (~lines 10–359), and one inline `<script>` IIFE (~lines 422–1667). `README.md` is a placeholder.

There is no build step, no package manager, no test suite, and no linter. Open the file in a browser to run it.

## Running and checking changes

- **Run:** open `index.html` directly, or serve the folder with any static server (for example `python3 -m http.server` and visit `localhost:8000`).
- **Verify:** there are no automated tests. Play the game in a browser after any change to gameplay, rendering, input, or audio. Check both desktop (keyboard and mouse) and touch layouts when you touch UI or input code.
- Keep it dependency-free. Don't add a bundler, framework, or npm packages unless the user asks for one.

## Code layout inside `index.html`

The script is one IIFE. Top-to-bottom it contains:

1. **DOM references** (`const xEl = document.getElementById(...)`): cached once at startup.
2. **Storage keys** (`STORAGE_*`): `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`.
3. **Tuning tables**: `DIFFICULTIES` (spawn rate, enemy speed and fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (sky palettes cycled per stage), `ENEMY_TYPES` (grunt, interceptor, tank, with `minStage` gating), and `BOSSES` (six designs, cycled by stage).
4. **Persistence and settings**: `localStorage` reads and writes for best score, mute, difficulty, and reduced motion.
5. **Audio**: a lazily created `AudioContext` (`ensureAudio`), with `beep` and `noiseBurst` primitives and a named `sfx` table. Audio starts only after a user gesture.
6. **Input**: keyboard state map `keys`, touch joystick and buttons, and mouse aiming. Touch controls are enabled only when a touch device is detected.
7. **Game state**: module-level `let` variables (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`, and so on), reset by `resetGame()`.
8. **Systems**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `applyPowerUp`, `addScore` and the combo chain (`chain`, `multiplier`), `loseLife`, `endGame`, `togglePause`.
9. **Boss attacks**: `BOSS_PATTERNS` is an array of bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring) that `updateBoss` picks from.
10. **Rendering**: `draw()` and its helpers (`drawPlane`, `drawBossShip`, `drawClouds`). Rendering reads the current state and never mutates it.
11. **Main loop**: `loop(now)` clamps `dt` to 50 ms, calls `trackPerf`, then runs `update(dt)` and `draw()` while the game is running and not paused.

### Conventions to follow

- Match the existing style: 2-space indentation, single quotes, `const` by default, `let` for mutable state, and compact one-line object literals for table entries.
- Add new enemy, boss, or theme data by extending the existing tables rather than adding special-case branches.
- Keep game logic in `update*` / `spawn*` functions and drawing in `draw*` functions. Don't mix them.
- Use `Math.random()` for gameplay randomness, consistent with the rest of the file.
- Don't add comments that narrate what the code does. Add a short comment only where the reason is non-obvious.

## Controls

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrow keys or WASD | Left-side joystick |
| Fire (hold) | Space | Fire button |
| Bomb | X or Shift | Bomb button |
| Pause | P or Escape | Pause button |
| Mute | Mute button | Mute button |

Mouse aiming moves a crosshair target on the canvas. The player follows it while the mouse is active.

## Accessibility and UX requirements

- Respect `prefers-reduced-motion`. The game reads this media query on startup, and the player can override it with the motion toggle. Screen shake and flashes must honor the reduced-motion setting.
- Announce important state changes through the `srLive` live region (`announce(text)`). Keep the `aria-live` region in sync with on-screen messages.
- Keep the HUD, overlay, and touch controls usable at phone-sized viewports. Scaling is handled in `resize()` against `BASE_W` × `BASE_H` (480 × 720).
- Use `touch-action: none` on the game surface and touch controls. Don't remove it, or touch play will scroll the page.

## Performance

- `trackPerf` samples frame times and sets a `quality` level (0 full, 1 medium, 2 low). Quality 0 enables expensive `shadowBlur` glow effects. Any new glow effect should check the `glow` flag so it degrades on slow devices.
- Keep per-frame allocations low in `update` and `draw`. Reuse existing arrays and filter them in place where possible.

## Git workflow

- Develop on the branch the session names (currently `claude/gifted-tesla-pbe2l3`). Don't push to `main` without explicit permission.
- Write clear commit messages that describe the gameplay or UI change.
- Don't create a pull request unless the user asks for one.

## Things to avoid

- Don't split `index.html` into multiple files, or add a build step, without the user asking.
- Don't change the stored key names (`skyfighter.*`). Changing them resets players' saved best scores and settings.
- Don't store secrets or external URLs in the page.

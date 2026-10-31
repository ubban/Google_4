# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. It is a single self-contained file:

- `index.html` (~1,670 lines): all markup, CSS, and JavaScript. There are no external assets, no bundler, no framework, and no package manager.
- `README.md`: a placeholder, currently only a title.

There is no build step, no test suite, no linter, and no CI configuration in the repo. To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Repository layout

Inside `index.html`:

| Lines (approx.) | Section |
| --- | --- |
| 10–362 | `<style>`: layout, HUD, overlay, touch controls, vignette/flash effects, keyframes |
| 363–421 | Markup: canvas, HUD, overlay/menus, touch controls, boss bar, live region |
| 422– | `<script>` wrapped in an IIFE; the sections are marked with `// ---------- … ----------` comments |

Major script sections, in file order:

1. **Adaptive performance**: `trackPerf` / `quality` (0 full, 1 medium, 2 low) based on recent frame times.
2. **Config tables**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
3. **Persistence and settings**: `localStorage` keys, mute, difficulty, reduced motion.
4. **Audio**: a WebAudio synth (`beep`, `noiseBurst`, `sfx`). No audio files are loaded.
5. **HUD and effects**: hearts, bombs, combo pop, stage banner, screen shake, flashes.
6. **Canvas sizing**: `BASE_W = 480`, `BASE_H = 720`, scaled to fit the window (max scale 1.6).
7. **Input**: keyboard, mouse steering, and touch (joystick plus fire/bomb buttons).
8. **Game state**: `resetGame`, the player, bullets, enemies, power-ups, and the boss.
9. **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, `draw`.
10. **Simulation**: `update(dt)` (player, spawns, collisions, power-ups), `updateBoss(dt)`.
11. **Main loop**: `loop(now)`, started with `requestAnimationFrame`.

### Game rules to keep in mind

- Difficulties (`easy`, `normal`, `hard`) change spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- Enemies: `grunt`, `interceptor` (weaving), `tank` (4 HP, appears from stage 2).
- Each stage ends after `KILLS_PER_STAGE` (18) kills and then brings a boss. Bosses cycle through `BOSSES` and telegraph attacks.
- Power-ups: `life`, `bomb`, and `weapon`. The weapon power-up toggles between `vulcan` and `laser`.
- Weapon level and combo multiplier are tracked on the player and in the HUD.
- Controls: arrow keys/WASD and mouse or touch to move; Space to fire; X or Shift to drop a smart bomb; P or Escape to pause.

## Conventions

- **Keep it one file.** Put new code, styles, and markup in `index.html` unless the user asks for a split. Do not add a build system, package manager, or framework without an explicit request.
- **No external dependencies or network assets.** Sound is synthesized and graphics are drawn on canvas. Keep it that way. The only external-looking content is the inline SVG favicon data URI.
- **Match the existing style.** The script is vanilla ES2015+ inside one IIFE with 2-space indentation, `const`/`let`, single-quoted strings, and section banner comments. Follow the same patterns.
- **Preserve the timing model.** Gameplay uses the `dt` passed to `update`. Keep frame-rate-independent movement when you add or change entities.
- **Respect accessibility settings.** Reduced motion (`skyfighter.reducedMotion`, which also respects `prefers-reduced-motion`) must keep disabling or softening screen shake, flashes, and similar effects. Keep ARIA live announcements (`announce`) working for important state changes.
- **Respect the adaptive quality level.** New particle or background effects should scale down with `quality` rather than always running at full cost.
- **Mobile first.** The layout uses `viewport-fit=cover`, safe-area insets, `touch-action: none`, and joystick/button touch controls. Test changes to HUD or controls at phone widths.
- **Persisted state.** The keys are `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, and `skyfighter.reducedMotion`. Wrap any new storage access so the game still works when storage is unavailable, and namespace new keys with `skyfighter.`.

## Verifying changes

There is no automated test suite. After a change:

1. Open `index.html` in a browser (or serve the folder statically) and check the console for errors.
2. Play through at least one full stage and one boss on `normal`. Check that the HUD updates for score, hearts, bombs, weapon, and multiplier.
3. Check pause (P/Escape), mute, difficulty switching, and the reduced-motion toggle.
4. For layout or input changes, check a narrow viewport and touch emulation.

If you add tests or tooling, document the commands here.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-xh4jju`). Do not push to `main` without explicit permission.
- Write clear, descriptive commit messages.
- Do not open a pull request unless the user asks for one.

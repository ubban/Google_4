# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on an HTML5 `<canvas>`.

Files:

- `index.html` — the entire game: markup, CSS (`<style>`), and JavaScript (`<script>`, wrapped in one IIFE). About 1,670 lines.
- `README.md` — a one-line title. Nothing else.

There is no build step, no package manager, no bundler, no test suite, and no linter configuration. Do not add any of these unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

Verify changes by playing the game in a browser. Check at least the title overlay, a full run through a stage, the boss fight, and the game-over screen. Also check a narrow (phone-sized) viewport, since touch controls are part of the layout.

## Architecture of `index.html`

Everything lives inside a single `(function () { ... })()` block. Do not introduce globals or modules unless there is a clear reason.

Major sections of the script, in file order:

1. **DOM references** — `getElementById` lookups for the canvas, HUD, overlay, buttons, and touch controls.
2. **Storage keys** — `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Persist user settings only through these keys. Wrap new `localStorage` access in `try/catch` if it can run in private or restricted browsing contexts.
3. **Tuning tables** —
   - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rate, enemy speed, enemy fire chance, lives, bombs, and boss HP multiplier.
   - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
   - `THEMES`: six sky palettes, one per stage, cycling with `(stage - 1) % 6`.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), and `tank` (4 HP, appears from stage 2).
   - `BOSSES`: six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN), each with its own colors and HP multiplier.
4. **Audio** — a lazily created `AudioContext` (`ensureAudio`), with helpers `beep`, `noiseBurst`, and a `sfx` map. Respect the mute setting.
5. **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`. Parallax clouds and the sun glow are drawn every frame.
6. **Spawning and combat** — `spawnEnemy`, `spawnBoss`, `spawnExplosion`, `fireWeapon`, `useBomb`, and the boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`), driven by `updateBoss`.
7. **Game loop** — `update(dt)` for simulation, `draw()` for rendering, and `loop(now)` scheduled with `requestAnimationFrame`. Time step `dt` is passed to `update`; keep simulation frame-rate independent.
8. **Input** — keyboard (`keys` map, `keydown`/`keyup` listeners), mouse steering and click-to-fire, and touch (joystick on the left, FIRE and BOMB buttons).
9. **UI state** — overlay, HUD (score, multiplier, weapon level, hearts, bombs), stage banner, combo pop-ups, boss health bar, pause, and a screen-reader live region (`srLive`) fed by `announce()`.

### Controls (keep docs in sync when changing them)

| Action | Keyboard | Mouse | Touch |
| --- | --- | --- | --- |
| Move | Arrows / WASD | Move cursor | Drag left-side joystick |
| Fire | Space | Click | Tap FIRE |
| Bomb | X or Shift | — | Tap BOMB |
| Pause | P or Esc | — | — |

The same text appears in the `#controlsHint` paragraph in the markup. Update both if controls change.

### Gameplay rules worth knowing

- Lives and bombs depend on difficulty (see `DIFFICULTIES`).
- Weapons have three power levels (Vulcan/Laser); power-ups raise weapon level, grant bombs, or add an extra life.
- Smart bombs clear all enemy bullets and enemies on screen and deal a fixed 24 damage to the boss (`useBomb`).
- The score multiplier comes from a kill chain (combo).
- The best score is saved to `localStorage` on game over.

## Conventions

- **Single file.** Keep HTML, CSS, and JS in `index.html` unless the user asks for a split. Do not add npm dependencies or a build step.
- **No external resources.** The page currently loads nothing from a CDN. The favicon is an inline SVG data URI. Keep it that way unless asked.
- **Style.** Match the existing code: two-space indentation, single quotes in JS, `const` by default, `let` when reassigned, and small named functions. Keep constants in the uppercase tuning tables rather than hard-coding magic numbers in the loop.
- **Accessibility.** Preserve the reduced-motion toggle (`prefers-reduced-motion` and the in-game motion button), keyboard controls, and the `aria-live` announcements. Keep text readable against both the sky gradients and the HUD pills.
- **Mobile.** The page uses `touch-action: none`, `user-scalable=no`, and safe-area insets. Do not break these when editing layout CSS.
- **Performance.** Avoid allocating objects inside the per-frame loop where practical. The loop already tracks frame times (`trackPerf`) for adaptive quality; check it before adding expensive effects.

## Git workflow

- Develop on the branch designated for the task. At the time of writing this is `claude/gifted-tesla-i4j317`. Never push to a different branch without explicit permission.
- Commit with clear, descriptive messages that explain *why* a change was made.
- Push with `git push -u origin <branch>`.
- Do not open a pull request unless the user explicitly asks for one.

## When editing this file

Keep `CLAUDE.md` accurate. If you change the file layout, add or remove a major section of the game script, change controls, or change the difficulty or stage tables, update the relevant part of this document in the same commit.

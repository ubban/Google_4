# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-file browser game, **Sky Fighter** — a Raiden-style vertical shoot-'em-up built on an HTML5 canvas.

- Entry point: `index.html` (≈58 KB). All markup, CSS (`<style>`), and JavaScript (`<script>`) live in this one file.
- `README.md` is a placeholder with no setup instructions.
- There is no build step, package manager, test suite, or linter. Open `index.html` in a browser to run the game.

## Repository layout

```
index.html   # The whole game: HUD/overlay markup, styles, and the game loop
README.md    # Placeholder
```

Inside `index.html`, the layout is roughly:

| Section | Approximate location | Contents |
|---|---|---|
| `<head>` / `<style>` | lines 1–362 | Meta tags, inline favicon, CSS for HUD, overlay, touch controls, vignette/flash effects |
| Markup | lines 363–421 | `#gameCanvas`, `#hud`, boss bar, stage banner, top bar, touch controls, start overlay |
| Script | lines 422–end | One IIFE (`'use strict'`) holding all state and logic |

Line numbers drift as the file changes; search for the identifiers below rather than trusting the numbers.

## Key constants and configuration (inside the script)

- **Difficulty** — `DIFFICULTIES` (easy / normal / hard): controls `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, and `bossHpMul`.
- **Stage pacing** — `KILLS_PER_STAGE = 18`. Reaching the quota triggers a boss; defeating it advances `stage`.
- **Themes** — `THEMES` cycles per stage (sky gradient, cloud/sun tints, display name).
- **Enemies** — `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP, stage 2+). Each has a `minStage` gate.
- **Bosses** — `BOSSES` cycles by `(stage - 1) % BOSSES.length`. Boss HP is `(70 + stage × 35) × hpMul × difficulty.bossHpMul`. Attack patterns are the `fire*` helpers (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Power-ups** — dropped by `maybeSpawnPowerUp`:
  - `life` (6%), `bomb` (8%), `weapon` (20%) — applied in `applyPowerUp`.
  - Weapon power-up raises the level 1→3, then toggles between `vulcan` and `laser`.
- **Persistence** (`localStorage`, all prefixed `skyfighter.`): `bestScore`, `muted`, `difficulty`, `reducedMotion`. Every access is a plain `localStorage` call — keep it that way unless adding error handling for private-mode browsers.

## Controls

- **Desktop:** mouse to steer and click to fire; Arrow keys or WASD to move; Space to fire; X or Shift for a bomb; P or Esc to pause.
- **Touch:** drag in the left zone to fly (virtual joystick); tap FIRE; tap BOMB.
- Keyboard input is tracked in the `keys` map via `keydown`/`keyup` on `e.code`.

## Architecture notes

- **Main loop:** `loop(now)` → `update(dt)` → `draw()`. `dt` (seconds) is capped at 0.05 per frame; `update` advances entities, collisions, spawns, and stage logic.
- **Entities** are plain arrays/objects inside the IIFE (enemies, bullets, power-ups, explosions, clouds). Reset in `resetGame()`.
- **Adaptive quality:** `trackPerf` averages the last 45 frame times and steps `quality` (0 full → 2 low) when FPS drops below ~40 or recovers above ~56.
- **Audio:** WebAudio synthesized via `beep` and `noiseBurst`; no audio files. Created lazily by `ensureAudio()` on first user gesture.
- **Accessibility:** `announce()` writes to the `#srLive` aria-live region. Reduced-motion mode disables screen shake, flash, and pulse effects.
- **Responsive canvas:** `resize()` fits the canvas to the viewport. `isTouchDevice` is used to decide whether touch controls are shown.

## Conventions

- Keep the game as a **single self-contained HTML file** with no external dependencies unless the user explicitly asks to change that. Do not add network requests.
- Keep all code inside the existing IIFE; don't add globals.
- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let` (no `var`), arrow functions for callbacks.
- Put new tunable numbers in the existing config objects (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- If a new `localStorage` key is added, prefix it with `skyfighter.` and declare it alongside the existing `STORAGE_*` constants.
- Preserve the `prefers-reduced-motion`-style toggle: any new screen-shake, flash, or flashing effect should respect `reducedMotion`.

## Verifying changes

There is no automated test or lint command. To verify a change:

1. Open `index.html` directly in a desktop browser and check the console for errors.
2. Play through at least one stage boss on each difficulty if the change touches combat, scoring, or pacing.
3. Check the touch layout at phone width (or with device emulation) if the change touches HUD, overlay, or controls.
4. Confirm the best score and settings persist across a reload.

## Git

- Development branch for this work: `claude/gifted-tesla-0ghj89`.
- Commit messages in this repo are imperative and descriptive (e.g., "Add Sky Fighter: …"), with a body describing what changed and why.

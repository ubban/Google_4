# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page HTML5 canvas game called **Sky Fighter**, a Raiden-style vertical shoot-'em-up.

- The whole game lives in `index.html`: markup, CSS in one `<style>` block, and vanilla JavaScript in one `<script>` block wrapped in an IIFE with `'use strict'`.
- No build step, no bundler, no package manager, no dependencies. Open `index.html` in a browser to run it.
- `README.md` is a placeholder and has no setup instructions.

## Repository layout

```
index.html   # The entire game (≈1,700 lines)
README.md    # Placeholder
CLAUDE.md    # This file
```

Inside `index.html`, the `<script>` section is organized by comment headers such as `// ---------- Adaptive performance ----------`. Keep new code in the matching section.

## Running and testing

- Run: open `index.html` directly, or serve the folder with any static server (for example `python3 -m http.server`).
- There is no test suite, linter, or CI configured. Verify changes by playing the game in a browser, at both desktop and phone-sized viewports.
- Syntax-check the script after edits, for example by extracting the `<script>` block and running `node --check` on it.

## Architecture notes

- **Game loop:** `loop(now)` calls `update(dt)` and `draw()` through `requestAnimationFrame`. `update` handles input, spawning, collisions, and boss patterns. `draw` is canvas-only.
- **Canvas sizing:** the logical play field is `BASE_W = 480` by `BASE_H = 720`. `resize()` scales it to fit the window.
- **Adaptive quality:** `trackPerf` averages recent frame times and steps `quality` between 0 (full), 1 (medium), and 2 (low). Respect this when adding expensive effects.
- **Difficulty:** the `DIFFICULTIES` table (`easy`, `normal`, `hard`) controls spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- **Content tables:** enemies (`ENEMY_TYPES`), bosses (`BOSSES`, with attack patterns in `BOSS_PATTERNS`), and stage themes (`THEMES`) are plain data objects. Add new content there first.
- **Stages:** `KILLS_PER_STAGE` kills trigger a boss. Defeating a boss advances the stage and the theme.
- **Weapons and power-ups:** the weapon has three levels (`player.level`), and `fireWeapon` picks the spread and lane patterns from that level. Power-ups (`life`, `bomb`, `weapon`) are spawned by `maybeSpawnPowerUp` and applied in `applyPowerUp`.
- **Accessibility:** screen-reader text goes through `announce()` into the `srLive` live region. Respect `reducedMotion`, which defaults to the `prefers-reduced-motion` media query and can be overridden by the user.

## Controls (for manual testing)

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrow keys or WASD | Left joystick zone |
| Fire | Space (hold) | Fire button (hold) |
| Smart bomb | X or Shift | Bomb button |
| Pause | P or Escape | Pause button |

Mouse input also moves the ship on the canvas.

## Persistence

`localStorage` keys, all prefixed `skyfighter.`:

- `bestScore`: high score
- `muted`: audio mute flag (`'1'` or `'0'`)
- `difficulty`: selected difficulty name
- `reducedMotion`: reduced-motion override (`'1'` or `'0'`)

Keep these keys stable. Renaming one resets that player's saved value. Wrap any new storage access so the game still works when storage is blocked.

## Conventions

- Plain ES2017+ JavaScript and CSS. No frameworks or external libraries. Do not add CDN dependencies without discussing it first.
- Match the existing style: 2-space indentation, single quotes in JavaScript, `const` by default, and short helper functions with descriptive names.
- Keep the game self-contained in `index.html` unless the user asks for a split.
- Keep the mobile layout working. The page uses `touch-action: none`, safe-area insets, and a fixed touch-control overlay.

## Git

- Development branch for this work: `claude/gifted-tesla-andnpd`.
- Write clear, descriptive commit messages in the imperative mood, matching the existing history.
- Do not open a pull request unless the user asks for one.

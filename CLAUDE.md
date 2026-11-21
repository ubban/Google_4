# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up.

- `index.html` — the entire game: markup, CSS (inline `<style>`, about lines 10–359), and JavaScript (inline `<script>`, starting about line 422). It is roughly 1,670 lines.
- `README.md` — a stub containing only the repo title.

There is no build step, no package manager, no test suite, and no external assets. Everything is self-contained.

## Running and verifying

- Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`).
- Verify changes by playing the game in a real browser (keyboard and, where relevant, touch), not just by reading the code. Check the start screen, a full stage progression, boss fights, pause, and game over.
- Test on a phone-sized viewport as well as desktop, since the layout and touch controls are responsive.

## Code structure (inside `index.html`)

The script is one IIFE. Key sections, in rough order:

1. **DOM references** — `getElementById` lookups for HUD, overlay, and touch controls.
2. **Constants** — `STORAGE_*` localStorage keys, `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES` (one per stage, cycling), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses with per-boss HP multipliers).
3. **Persisted settings** — best score, mute, difficulty, and reduced motion are read from and written to `localStorage`.
4. **Audio** — a lazily created `AudioContext` (`ensureAudio`), with `beep` and `noiseBurst` primitives and a `sfx` table. Audio starts only after a user gesture.
5. **HUD and effects** — hearts, bombs, combo pop, stage banner, screen shake, and flash helpers.
6. **Resize** — `resize()` scales a fixed `BASE_W × BASE_H` (480 × 720) canvas to fit the window, capped at 1.6×.
7. **Input** — keyboard (`keydown`/`keyup` on `e.code`), mouse/pointer targeting on the canvas, and a virtual joystick plus fire/bomb buttons for touch.
8. **Game state and loop** — `update(dt)` handles movement, firing, enemies, bosses, collisions, and pickups. `draw()` renders. `loop(now)` drives both via `requestAnimationFrame`, clamps `dt` to 0.05 s, and runs only when `running && !paused`.
9. **Adaptive quality** — `trackPerf` watches frame times and steps the `quality` level (full / medium / low) when the frame rate drops.

### Controls

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrow keys or WASD | Joystick |
| Fire | Space (hold) | Fire button |
| Smart bomb | X or Shift | Bomb button |
| Pause | P or Escape | Pause button |
| Aim | Mouse / pointer | — |

### Gameplay rules worth knowing

- **Difficulty** (`DIFFICULTIES`) sets spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
- **Weapons** have two types, `vulcan` and `laser`, each with levels 1–3. A weapon pickup first raises the level. At level 3 it switches to the other weapon and resets to level 1.
- **Stages** advance after `KILLS_PER_STAGE` kills, which triggers a boss. Stage themes and bosses both cycle with the stage number.

## Conventions

- **Keep it a single file** unless the user asks for a restructure. Do not add a bundler, framework, or package.json without a clear reason.
- **No external network dependencies.** The only external-style reference is an inline SVG data URI for the favicon. Do not add CDN scripts or fonts without asking.
- **Preserve the persisted-setting keys** (`skyfighter.*`). Changing them silently resets players' saved best score and preferences.
- **Preserve accessibility hooks:** `aria-live` (`srLive`), the reduced-motion toggle, and the `prefers-reduced-motion` default. New visual effects such as shake or flash should respect the reduced-motion setting.
- **Keep the game playable on touch.** Don't rely on keyboard-only input for anything new.
- **Match existing style:** two-space indentation, single quotes in JS, `const` by default, and short helper functions in the existing naming style.
- Prefer small, targeted edits over large rewrites of the inline script.

## Git

- Development branch for this work: `claude/gifted-tesla-fm5u3a`.
- Commit with a clear, descriptive message. Do not open pull requests unless asked.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up built as a single-file HTML5 canvas game.

- `index.html` — the entire game: markup, CSS (`<style>` in the head), and JavaScript (one IIFE in a `<script>` block near the end). There is no build step, no bundler, and no package manager.
- `README.md` — currently a placeholder; keep it accurate if you expand it.

## Running the game

No install is needed. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Architecture notes (`index.html`)

The JavaScript is organized as sections inside one IIFE:

- **Config constants** — `DIFFICULTIES` (spawn rates, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six boss designs).
- **Persistence** — `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Always wrap storage access so the game still works if storage is unavailable (private mode, blocked site data).
- **Audio** — Web Audio synthesis (`beep`, `noiseBurst`); no audio files. `ensureAudio()` must run from a user gesture before sound plays.
- **Input** — keyboard (`keys[e.code]`), touch joystick and fire/bomb buttons for mobile, and mouse/pointer on the canvas.
- **Game loop** — `update(dt)` advances state, `draw()` renders, and `loop(now)` drives them with `requestAnimationFrame`.
- **Combat** — weapons (`fireWeapon`, 3 power levels, Vulcan/Laser), smart bombs (`useBomb`), kill-chain score multiplier (`addScore`, `breakChain`), and power-ups (`life`, `bomb`, `weapon`) applied in `applyPowerUp`.
- **Bosses** — `BOSS_PATTERNS` are bullet-pattern functions selected in `updateBoss`; `killBoss` advances the stage.
- **Accessibility** — reduced-motion mode disables screen shake, hit flash, and the low-health pulse. It defaults to the `prefers-reduced-motion` media query until the player toggles it.

### Controls

- Move: arrow keys or WASD. Fire: Space (held) or the on-screen fire button.
- Bomb: X or Shift. Pause: P or Escape.

## Conventions

- Keep the game as a single self-contained `index.html` unless the user asks for a restructure. Do not add a build system or dependencies without asking.
- Use `const`/`let`, the existing 2-space indentation, and the existing naming style (camelCase functions, UPPER_SNAKE constants).
- Tune gameplay through the config constants rather than hardcoding magic numbers in update logic.
- Keep the layout working at phone width and preserve the touch controls.
- No external network requests at runtime; the game is self-contained.

## Testing

There is no automated test suite. To verify changes:

1. Open `index.html` in a browser and play through at least one stage and one boss.
2. Check all three difficulty levels, pause/resume, mute, and the reduced-motion toggle.
3. Confirm best score persists across reloads.
4. Check the touch layout at a phone-sized viewport (for example via browser devtools device emulation).

## Git

- Development branch: `claude/gifted-tesla-sdkiwf`. Commit and push there; do not push to other branches without explicit permission.
- Do not open pull requests unless the user asks.

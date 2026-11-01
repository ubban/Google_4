# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up.

- `index.html` — the entire game: markup, CSS (in a `<style>` block), and JavaScript (one inline `<script>` wrapped in an IIFE with `'use strict'`). Roughly 1,670 lines.
- `README.md` — placeholder title only.

There is no build step, package manager, bundler, test suite, or linter configured. Do not add one unless asked.

## Running and checking

- **Run:** open `index.html` directly in a browser. No server is required.
- **Syntax check** (the only automated check available): extract the inline script and run it through Node:
  ```bash
  awk '/<script>/{f=1;next}/<\/script>/{f=0}f' index.html > "$SCRATCH/sf_check.js" && node --check "$SCRATCH/sf_check.js"
  ```
  Use the scratchpad directory for `$SCRATCH`, not the repo. This currently passes.
- For behavior changes, play-test in a browser (desktop and a touch/phone-width viewport). Say explicitly in the final message if you could not do this.

## Architecture of `index.html`

All game logic lives inside one IIFE. Key sections, in source order:

| Section | Purpose |
|---|---|
| `<style>` / HTML body | HUD (`#hud`), boss bar, combo/stage banners, top bar (mute, reduced motion, pause), touch controls, start/pause overlay (`#overlay`) |
| Storage keys | `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` (all in `localStorage`) |
| Adaptive performance (`trackPerf`) | Rolling 45-frame average adjusts a `quality` level (0 full → 2 low) |
| `DIFFICULTIES` | Per-difficulty spawn rate, enemy speed, enemy fire chance, lives, starting bombs, boss HP multiplier |
| `THEMES` | Six background palettes; the theme cycles with the stage number (`(stage - 1) % 6`) |
| `ENEMY_TYPES` / `BOSSES` | Enemy stats (`grunt`, `interceptor`, `tank`) and six boss definitions (`VANGUARD` → `TITAN`) |
| Audio (`ensureAudio`, `beep`, `noiseBurst`, `sfx`) | Web Audio synthesis only; no audio files |
| Input | Keyboard (`keys` map), mouse on canvas, joystick + FIRE/BOMB buttons for touch |
| `resize()` | Scales a fixed `BASE_W × BASE_H` (480 × 720) logical canvas to the viewport |
| `resetGame`, `update(dt)`, `draw()`, `loop(now)` | Core state reset, simulation step, rendering, and the `requestAnimationFrame` loop |
| `fireWeapon`, `applyPowerUp`, `BOSS_PATTERNS` | Player weapons (VULCAN / LASER, levels 1–3), power-ups, and boss bullet patterns |
| `endGame`, `togglePause` | Game-over and pause flows, including the `#overlay` card |

Notable game rules:
- Kills advance the stage every `KILLS_PER_STAGE` (18) kills; each stage ends in a boss.
- Power-ups: `life`, `bomb`, and `weapon` (weapon power-ups raise the level, then toggle VULCAN ↔ LASER and reset to level 1).
- Delta time is clamped to 50 ms per frame in `loop`, so long pauses in the tab don't teleport objects.

## Conventions

- **Keep it a single file.** Add new CSS, markup, and JS to `index.html` unless the user asks for a split. Don't introduce external dependencies or CDN links without asking.
- **Stay inside the IIFE.** Don't add globals; expose nothing on `window` unless required.
- **Use the existing naming and style:** `camelCase` for functions and variables, `UPPER_SNAKE_CASE` for constants, two-space indentation, single quotes for strings, and the `// ---------- Section ----------` comment banners.
- **Storage keys** use the `skyfighter.` prefix. Wrap any new `localStorage` access so the game still works when storage is blocked.
- **Accessibility:** respect `prefers-reduced-motion` (the `reducedMotion` flag gates screen shake, flashes, and pulses). Keep the `#srLive` live region updated for important announcements via `announce()`.
- **Balance numbers** live in the `DIFFICULTIES`, `ENEMY_TYPES`, and `BOSSES` tables. Prefer editing those tables over hard-coding values in logic.
- **Touch support matters:** keep `touch-action: none` on the game surface and test that changes still work with touch controls.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-lotojj`). Don't push to other branches without explicit permission.
- Make small, focused commits with descriptive messages.
- Do not open a pull request unless the user asks for one.
- Before changing this file or `index.html`, check `git status` so you don't overwrite the user's in-progress edits.

## Keeping this file current

Update this CLAUDE.md when the structure changes: new files, a build or test step being added, new storage keys, or new major game systems. Keep it factual and short; remove entries that no longer match the code.

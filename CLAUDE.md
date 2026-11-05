# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs entirely in the browser. It is a single self-contained file:

- `index.html` — all markup, CSS, and JavaScript (inline `<style>` and one `<script>` block). The game is drawn on a `<canvas id="gameCanvas">` at a logical resolution of **480×720** (`BASE_W`/`BASE_H`), scaled to fit the viewport.
- `README.md` — placeholder; currently only a title.

There is no build step, package manager, bundler, test suite, or linter. Do not introduce one unless asked.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser: start a run, check the affected behavior, and check both desktop (keyboard/mouse) and touch layouts when touching HUD, overlay, or input code.
- For a quick syntax check of the inline script, extract the `<script>` block and run `node --check` on it.

## Code layout inside `index.html`

The script is one IIFE-style block with these sections, roughly in order:

1. **DOM references** — `getElementById` lookups for HUD, overlay, touch controls, and buttons.
2. **Persistence keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all `skyfighter.*` in `localStorage`).
3. **Config tables** — `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES` (name, colors, `hpMul`).
4. **Audio** — lazily created `AudioContext` (`ensureAudio`), `beep`/`noiseBurst` primitives, and the `sfx` table. Audio is only started after a user gesture.
5. **Rendering helpers** — hearts/bombs HUD, hit/bomb flashes, combo popups, stage banner, screen shake.
6. **Input** — keyboard (`keys` map, `KeyP`/`Escape` pause, `KeyX`/`Shift` bomb, `Space` fire), mouse/pointer target, and touch (virtual joystick in `moveZone`, `fireBtn`, `bombBtn`).
7. **Performance scaling** — `trackPerf` adjusts a `quality` level (0 full, 1 medium, 2 low) from average frame time.
8. **Game loop** — update and draw for player, enemies, bullets, power-ups, bosses, particles, and parallax clouds.

### Gameplay facts worth knowing

- Stages advance every `KILLS_PER_STAGE` kills; each stage ends in a boss from `BOSSES`, which has a health bar (`bossBarWrap`).
- Weapons: Vulcan and Laser, each with 3 power levels. Smart bombs clear the screen and damage bosses.
- A kill-chain multiplier raises score (`mult` HUD element).
- Best score is saved to `localStorage` under `skyfighter.bestScore`.
- Reduced-motion is honored from `prefers-reduced-motion` unless the user has set an explicit preference via the motion toggle, which is then stored.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. If you split out a file, update every reference and tell the user why.
- Match the existing style: 2-space indentation, `const` by default, DOM lookups grouped at the top, tuning numbers kept in the config tables rather than scattered through the loop.
- Any new persisted setting should use a `skyfighter.*` key and be read with a fallback default, as the existing ones are.
- Preserve mobile behavior: `touch-action: none`, safe-area insets, and the `user-select: none` on the page. Don't break the viewport meta tag.
- Keep accessibility hooks working: the `srLive` live region is used by `announce()` for screen-reader messages.
- Do not add external network requests, CDN scripts, or third-party assets. The game is intentionally self-contained and uses inline data-URI favicon and canvas-drawn graphics.

## Git

- Development branch for this task: `claude/gifted-tesla-rgbqzu`. Commit and push there; do not open a pull request unless explicitly asked.
- Write commit messages in the imperative mood with a short summary line, followed by a body that explains *why* when it isn't obvious.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up played in the browser. It is a single self-contained file:

- `index.html` (~1,670 lines) — all markup, CSS (inline `<style>`), and game logic (inline `<script>`, wrapped in an IIFE). No build step, no bundler, no dependencies.
- `README.md` — placeholder; currently only a title.

There is no package manager, test suite, linter, or CI configuration in the repo.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser (or with Playwright, per the environment's pre-installed Chromium) and checking the affected behavior, including on a narrow/mobile viewport.
- Syntax-check the inline script after edits if you can't run the game, e.g. extract the `<script>` block and run `node --check`.

## Code structure (inside `index.html`)

The script is one IIFE with these major sections, roughly in file order:

1. **DOM references** — `getElementById` lookups for HUD, overlay, touch controls, and buttons.
2. **Storage keys** — `skyfighter.*` keys in `localStorage` for best score, mute, difficulty, and reduced motion. All persistence goes through these constants.
3. **Config tables** — `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six boss designs, cycled by stage).
4. **Input** — keyboard (`keys` map, keydown/keyup), mouse steering and click-to-fire, and touch joystick + FIRE/BOMB buttons.
5. **Entities and spawning** — `spawnEnemy`, `spawnBoss`, `spawnExplosion`, `fireWeapon` (Vulcan/Laser, 3 power levels each).
6. **Update / draw** — `update(dt)` advances the simulation, `updateBoss(dt)` runs boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`), `draw()` renders everything, and `loop` drives `requestAnimationFrame`.
7. **Audio** — a lazily created `AudioContext` (`audioCtx`), muted via the mute toggle.

Gameplay rules worth knowing before changing things:

- Score uses a kill-chain multiplier (`mult`) shown in the HUD.
- Stages advance after `KILLS_PER_STAGE` kills and end in a boss fight with a health bar.
- Bombs (`X` or `Shift`) clear enemy bullets and on-screen enemies and deal 24 damage to a boss; the count is set per difficulty.
- Power-ups include weapon level, bombs, and extra life.
- Pause is on `P`/`Esc`; the reduced-motion toggle disables screen shake, flashes, and the low-health vignette pulse. It defaults from `prefers-reduced-motion` unless the user has set it.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split files; the single-file, no-build design is intentional.
- Match the existing style: 2-space indentation, `const` by default, small named functions, config values in the top-level tables rather than hard-coded in logic.
- Use the `skyfighter.` prefix for any new `localStorage` key and read/write it through a constant.
- Keep the accessibility hooks intact: the `srLive` live region (`announce()`), `aria`/`title` attributes on icon buttons, and the `prefers-reduced-motion` handling.
- Keep the layout working at phone widths: the page uses `touch-action: none`, safe-area insets, and touch controls that appear on touch devices. Don't add scrollable page content.
- Do not add external script or stylesheet dependencies without a clear need; the game currently loads nothing from the network.

## Git

- Development branch for this session: `claude/gifted-tesla-vlhw7k`. Commit and push there; don't push to other branches without explicit permission.
- Commit messages in this repo are short imperative summaries, with an optional body of bullet points describing the change.
- Don't open a pull request unless asked.

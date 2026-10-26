# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small, dependency-free browser game, **Sky Fighter** (a Raiden-style vertical shoot-em-up), delivered as a single self-contained page.

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` (~1,670 lines).
- `README.md` — placeholder; it contains only a title and a stray timestamp.

There is no build step, no package manager, no bundler, no test suite, and no linter configuration. Do not add tooling unless asked.

## Running it

Open `index.html` directly in a browser. No server is required. Everything is loaded from the page itself, so do not introduce external CDN scripts or fonts without a reason.

## Code layout inside `index.html`

The inline script is one `(function () { 'use strict'; ... })();` block with these sections, roughly in file order:

1. **DOM references** — `getElementById` lookups for every HUD element, overlay, and touch control.
2. **Persistence keys** — `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every read/write is wrapped to tolerate storage being unavailable; keep it that way.
3. **Adaptive quality** — `trackPerf` samples frame times and steps `quality` (0 full / 1 medium / 2 low) to reduce effects on slow devices.
4. **Tuning tables** — the main place to change game balance:
   - `DIFFICULTIES` — spawn rates, enemy speed and fire chance, starting lives and bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` — kills needed before a boss warning.
   - `THEMES` — per-stage sky palettes; the stage index cycles through them.
   - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP). Each has a `minStage` gate.
   - `BOSSES` — six boss designs (name, colors, `hpMul`), selected by stage.
5. **Audio** — a lazily created `AudioContext` (`ensureAudio`) with WebAudio `beep` and `noiseBurst` helpers. Sound effects are named in the `sfx` table. Audio must only start after a user gesture.
6. **HUD and feedback** — hearts/bombs rendering, combo popups, stage banner, screen shake, hit flash, and an `aria-live` announcer (`announce`) for screen readers.
7. **Layout scaling** — the canvas is a fixed `BASE_W × BASE_H` (480×720) logical size, scaled to fit the window by `resize()`.
8. **Input** — keyboard (`keys` map keyed by `KeyboardEvent.code`), touch joystick plus fire/bomb buttons, and mouse. Bombs are bound to `X` and `Shift`; pause to `P` and `Escape`; fire to `Space` or held fire button.
9. **Game state** — module-level `let`s (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `bombCount`, `stage`, `chain`, `multiplier`, …). `resetGame()` reinitializes them.
10. **Drawing** — `drawPlane`, `drawBossShip`, and the rest of the canvas rendering, all in `draw()`.
11. **Simulation** — `update(dt)` runs the spawning, collision, boss, power-up, and scoring logic, and is driven by `loop(now)` via `requestAnimationFrame`. `dt` is clamped so pauses and tab switches do not cause jumps.

Power-ups are `life`, `bomb`, and `weapon` (see `maybeSpawnPowerUp` and `applyPowerUp`). A weapon pickup raises the weapon level from 1 to 3. At level 3, the next weapon pickup switches between Vulcan and Laser and resets the level to 1.

## Conventions

- **Single file.** Keep HTML, CSS, and JS in `index.html` unless the user asks for a split. Do not add a framework.
- **Match the local style.** Two-space indentation, single quotes in JS, `const` by default, short section comments, and camelCase names. Constants that describe tuning are UPPER_SNAKE_CASE.
- **Keep the game loop allocation-light.** Avoid creating new arrays or objects in per-frame hot paths beyond what the existing code already does. Gate expensive effects on `quality`.
- **Respect accessibility hooks.** Honor `reducedMotion` for shake and flash effects, and keep `announce()` calls for important state changes such as boss warnings and game over.
- **Pointer and touch safety.** Use `{ passive: false }` plus `preventDefault()` only where the existing code does, on controls that must not scroll the page.

## Making changes

- Balance changes belong in the tuning tables above, not scattered through the logic.
- When adding an enemy, boss, or theme, follow the existing object shape, and check that any `minStage` gating still produces sensible early waves.
- Test by opening `index.html` in a browser and playing through at least one stage transition and one boss fight. Check all three difficulties if you touched `DIFFICULTIES`. Also check touch controls with browser device emulation when you change input code.
- A quick syntax check of the inline script is `node --check` on the extracted `<script>` body (for example, saved to a file in the scratchpad directory).

## Git

- Development happens on feature branches. The current working branch is `claude/gifted-tesla-597s8x`.
- Commit messages in this repo are imperative and descriptive, with a short subject line and bullet points for detail (see the `Add Sky Fighter` commit for the house style).
- Do not open pull requests unless the user asks.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser.

- `index.html` — the entire game: inline `<style>`, inline `<script>`, and a `<canvas>` element. About 1,670 lines.
- `README.md` — a one-line title. Nothing else.

There is no build step, package manager, test suite, or linter. Do not add one unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). No install step is needed.

## Code layout inside `index.html`

The script is a single inline block with no modules. Major sections, in rough file order:

- **Constants and storage keys** (around line 463): `STORAGE_*` keys for best score, mute, difficulty, and reduced motion. `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
- **Persisted settings** (around line 517): read from `localStorage` at load time, written on change.
- **Audio** (around line 543): a lazily created `AudioContext` (`ensureAudio`), with `beep` and `noiseBurst` helpers.
- **HUD and feedback**: `renderHearts`, `renderBombs`, combo pop, stage banner, screen shake.
- **Input** (around line 753): `resize` scales a fixed `BASE_W × BASE_H` (480×720) logical canvas to the viewport. Keyboard, touch joystick, fire and bomb buttons, and mouse all feed the same `keys`, `joyVec`, and `firing` state.
- **Game state and loop**: `resetGame`, `update(dt)`, the draw functions (`drawPlane`, `drawBossShip`), spawning, collisions, power-ups, bombs, and boss bullet patterns (`fireRadial`, `fireCross`, `fireHomingSeeded`, and others). The main loop is driven by `requestAnimationFrame`.

Controls: arrow keys or WASD to move, Space or the fire button to shoot, X / Shift / bomb button to drop a smart bomb, P or Escape to pause. Touch devices get a joystick on the left and fire and bomb buttons on the right.

## Conventions

- Keep the game as a single self-contained HTML file with inline CSS and JS unless the user asks for a restructure. Do not introduce a bundler, framework, or external CDN dependency.
- Use the existing style: `const` for config tables, `camelCase` identifiers, `UPPER_SNAKE_CASE` for constants, and `skyfighter.*` prefixes for `localStorage` keys.
- Use `dt`-based updates in `update()` so movement stays frame-rate independent. Keep the `BASE_W × BASE_H` logical coordinate space when adding entities.
- Respect the reduced-motion setting when adding screen shake, flashes, or particle effects. Route audio through the existing helpers and the mute flag.
- Test input changes on both keyboard and touch paths. Layout must work at phone width with no horizontal page scroll.

## Git

- Default development branch for this session: `claude/gifted-tesla-xvtjy9`. Do not push to other branches without explicit permission.
- Commit messages are short, imperative, and describe the gameplay or file change (for example, `Add Sky Fighter: Raiden-style vertical shoot-em-up game`).
- Do not create pull requests unless the user explicitly asks.

## Verification

Because there are no automated tests, verify changes by loading `index.html` in a browser and playing through the affected path. For game-logic changes, check at least: movement, firing, a bomb, a stage transition, a boss, game over, and the best-score update in `localStorage`.

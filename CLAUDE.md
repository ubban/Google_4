# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a single `<canvas>`.

- `index.html` — the entire game: inline CSS (`<style>`, ~lines 10–421), HTML overlay/HUD/touch controls (~lines 360–420), and one inline IIFE in `<script>` (~line 422 to end) holding all game logic.
- `README.md` — a one-line placeholder. Keep it accurate if you change how the project is described.

There is no build step, package manager, test suite, or linter. Nothing is compiled or bundled.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- Verify changes by playing the game in a browser. There are no automated tests to run.
- Syntax-check the inline script after edits, for example by extracting the `<script>` block into a scratch file and running `node --check` on it. Keep scratch files out of the repo.

## Architecture of `index.html`

The script is one IIFE. Key pieces, roughly in file order:

- **DOM references and storage keys.** Elements are looked up once at the top. `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Reads and writes go straight to `localStorage`, so wrap any new access in try/catch in case storage is blocked.
- **Config tables.** These drive balance and should be the first place to tune gameplay:
  - `DIFFICULTIES` — `easy` / `normal` / `hard`, with spawn rates, enemy speed, shoot chance, lives, bombs, and boss HP multiplier.
  - `KILLS_PER_STAGE` — kills needed to advance a stage.
  - `THEMES` — sky palettes cycled per stage.
  - `ENEMY_TYPES` — `grunt`, `interceptor`, `tank`, with `minStage` gating when each appears.
  - `BOSSES` — six bosses (VANGUARD → TITAN), cycled by stage, each with an `hpMul`.
- **Audio.** Web Audio API only, no audio files. `ensureAudio()` lazily creates the context on the first user gesture, which browsers require. Sound helpers are `beep`, `noiseBurst`, and the `sfx` object.
- **Performance scaling.** `trackPerf` averages frame times and sets `quality` (0 full, 1 medium, 2 low) to reduce effects when frames are slow.
- **Entities and state.** The `player` object holds position, cooldowns, `weapon` (`vulcan` or `laser`), `level`, and `invuln`. Enemies, bullets, enemy bullets, power-ups, and particles are plain arrays.
- **Game loop.** `update(dt)` handles movement, firing (`fireWeapon`), spawning, collisions, and stage progression. `updateBoss(dt)` drives boss patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`). `draw()` renders the frame. `loop(now)` calls them each animation frame via `requestAnimationFrame`.
- **Input.** Keyboard (`keys` map keyed by `KeyboardEvent.code`), mouse on the canvas, and touch (joystick in `moveZone`, `fireBtn`, `bombBtn`). `Space` fire, `X`/`Shift` bomb, `P`/`Esc` pause. Mute, motion, and pause also have on-screen buttons.
- **Accessibility.** `announce()` writes to the `srLive` live region. Respect `reducedMotion`, which defaults from `prefers-reduced-motion` and can be overridden by the user.

## Conventions

- Keep the game self-contained in `index.html`. Do not add build tooling, frameworks, or external dependencies unless the user asks for them.
- Use `const`/`let` and the existing style (2-space indent, single quotes in JS, semicolons).
- Put new balance numbers in the existing config tables, not inline in logic.
- Any new UI must work on both desktop and touch. Check the mobile layout (`viewport-fit=cover`, safe-area insets, `touch-action: none`) when changing the HUD or overlays.
- Guard new `localStorage` and audio calls so the game still runs when storage or audio is unavailable.

## Git

- Develop and push on the branch the session specifies. Do not push to other branches without permission.
- Commit messages are short and imperative, matching existing history (for example, `Add Sky Fighter: Raiden-style vertical shoot-em-up game`).
- Do not open a pull request unless the user asks for one.

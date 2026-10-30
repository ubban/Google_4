# CLAUDE.md

Guidance for working in this repository.

## Project overview

Google_4 is a small static project containing **Sky Fighter**, a Raiden-style vertical shoot-em-up
built as a single-page HTML5 canvas game.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–360), and JavaScript (`<script>`, ~lines 422–1669).
- `README.md` — placeholder title only.

There is no build step, package manager, test suite, linter, or CI configuration.

## Running the game

Open `index.html` directly in a browser. No server is required.

For a quick local server, from the repo root:

```bash
python3 -m http.server 8000
```

## Architecture of `index.html`

The script is one IIFE with `'use strict'`. Code is grouped into sections marked with comment banners
such as `// ---------- Adaptive performance ----------`. Keep new code in the matching section, or add a
new banner for a new concern.

Key areas, in file order:

- **DOM references and constants** — `const` element lookups at the top; `STORAGE_*` keys.
- **Adaptive performance** — `trackPerf` adjusts a `quality` level (0–2) from frame times.
- **Tuning tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
- **Audio** — lazy `ensureAudio()` on first user gesture, then `beep` / `noiseBurst` synthesis (no audio files).
- **Input** — keyboard listeners, a touch joystick (`joystickStart/Move/End`), and fire/bomb buttons.
- **Game state** — `resetGame()` initializes state; `update(dt)` runs simulation; `draw()` renders.
- **Weapons and bombs** — `fireWeapon`, `useBomb`, `applyPowerUp`.
- **Bosses** — `BOSS_PATTERNS` are bullet-pattern functions (`fireRadial`, `fireSpiralBurst`, …); `updateBoss`, `killBoss`.
- **Lifecycle** — `loseLife`, `endGame`, `togglePause`.
- **Loop** — `loop(now)` caps `dt` at 0.05s and runs `update` and `draw` only while `running && !paused`.

## Controls

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrow keys or WASD | Left-side joystick |
| Fire | Space (hold) | Fire button |
| Bomb | X or Shift | Bomb button |
| Pause | P or Escape | ⏸ button in the top bar (all devices) |

## Persistence

`localStorage` keys, all prefixed `skyfighter.`:

- `bestScore`
- `muted` (`'1'` / `'0'`)
- `difficulty` (`easy` / `normal` / `hard`)
- `reducedMotion` (`'1'` / `'0'`; defaults to the `prefers-reduced-motion` media query when unset)

Wrap any new storage access in care for private-browsing and disabled-storage cases, and treat the
stored value as untrusted input (validate or fall back to a default, as the existing code does).

## Conventions

- **Vanilla JS, no dependencies.** Do not add frameworks, bundlers, or CDN scripts without a clear reason.
- **Single file.** Add new game code to `index.html` unless the user asks for a split.
- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let` (no `var`), arrow
  functions for callbacks, and short section-banner comments.
- Tuning values live in the constant tables near the top of the script. Prefer adding a value there over
  hard-coding a magic number deep in a function.
- Keep the game accessible: respect `reducedMotion` for screen shake and flashes, and keep the
  `srLive` live region updated through `announce()` for important events.
- Keep the mobile layout working. Touch controls are enabled only when `isTouchDevice` is true (see
  `#touchControls.enabled` in the CSS), so test both the desktop and touch paths when changing input or layout.

## Testing

There is no automated test suite. Verify changes by opening the game in a browser and playing through:

- Start, pause/resume, and game over (score and best-score update).
- Each difficulty, and a stage transition at `KILLS_PER_STAGE`.
- Boss fight and boss defeat.
- Bomb use, weapon power-ups, and extra life.
- Mute and reduced-motion toggles, and that they persist across reloads.
- Touch controls, using browser device emulation.

Also check the browser console for errors. Report any check you could not run.

## Git workflow

- Develop on the branch designated for the task (currently `claude/gifted-tesla-5dfbml`). Do not push to other branches without permission.
- Write clear, descriptive commit messages that explain why the change was made.
- Do not open a pull request unless the user asks for one.

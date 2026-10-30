# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a
single self-contained HTML5 canvas game.

- `index.html` — the entire game: markup (~lines 1–360), inline CSS (`<style>`,
  ~lines 10–359), and inline JavaScript (`<script>`, ~lines 422–1667).
- `README.md` — placeholder; no build or setup instructions.

There is no build step, no bundler, no package manager, and no test suite. The
game runs by opening `index.html` in a browser.

## Working on the game

- Keep everything in `index.html` unless the user asks for a split. Do not add
  a framework, a build tool, or external dependencies without asking.
- All game logic lives in one IIFE with `'use strict'`. Don't introduce globals.
- Match the existing style: `const` / `let`, arrow callbacks, `function`
  declarations for named helpers, and short section banners of the form
  `// ---------- Name ----------`.
- Tuning data is declared as plain object/array constants near the top of the
  script: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`,
  `KILLS_PER_STAGE`, `BASE_W` / `BASE_H`. Prefer changing these over hard-coding
  numbers in update/draw code.
- Frame loop: `loop(now)` → `update(dt)` → `draw()`. `dt` is clamped to 0.05 s.
  Keep per-frame work allocation-light; the adaptive `quality` level
  (`trackPerf`) already throttles effects when frames run slow.
- Canvas is drawn at a fixed logical size (`BASE_W` × `BASE_H`, 480 × 720) and
  scaled by `resize()`. Use logical coordinates in game code.

## Persistence and browser APIs

`localStorage` keys are namespaced with `skyfighter.`:

| Key                       | Meaning                          |
| ------------------------- | -------------------------------- |
| `skyfighter.bestScore`    | Best score                       |
| `skyfighter.muted`        | `'1'` when audio is muted        |
| `skyfighter.difficulty`   | `easy` / `normal` / `hard`       |
| `skyfighter.reducedMotion`| `'1'` / `'0'`; defaults to the OS preference |

Reuse these keys and the existing helpers (`setDifficulty`, `updateMuteBtn`,
`updateMotionBtn`) rather than writing new ones. Audio is synthesized with the
Web Audio API (`ensureAudio`, `beep`, `noiseBurst`); don't add audio files.

## Controls (for testing)

- Desktop: arrow keys / WASD to move (via `keys[e.code]`), `Space` to fire,
  `X` / `Shift` for a bomb, `P` / `Esc` to pause.
- Touch: joystick in `moveZone`, `fireBtn` (hold), `bombBtn`.
- Mouse: moving the pointer over the canvas steers the ship; `mousedown` on the
  canvas fires.

## Accessibility and UX conventions

- Respect `prefers-reduced-motion`: effects that shake or flash must honor
  `reducedMotion`. `screenShake` scales its magnitude down, and `flashHit` skips
  entirely.
- Announce important state changes through `announce()` (the `#srLive`
  live region) so screen readers hear them.
- The game must stay playable at phone width. Keep touch controls working when
  changing layout or the overlay.

## Verifying changes

No automated tests or linters exist yet. For any change:

1. Open `index.html` directly in a browser (or serve the folder with any static
   server) and check the console for errors.
2. Play through at least: title → start → a few kills → a power-up → a bomb →
   pause/resume → game over → restart. Check the best score persists across a reload.
3. Toggle difficulty, mute, and reduced-motion, and confirm each persists.
4. If the change touches layout or input, check it at a phone-sized viewport.

If you add tests or tooling, document the commands here.

## Git

- Develop on the branch the session specifies (currently `claude/gifted-tesla-sb5j23`).
- Write commit messages in the imperative mood with a short summary line and a
  body that explains the gameplay or code change.
- Do not open a pull request unless explicitly asked.

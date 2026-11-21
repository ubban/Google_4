# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up that runs in a single HTML file.

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>`
  (~1,670 lines). No build step, no dependencies, no bundler.
- `README.md` — placeholder (title only).

There is no `package.json`, test suite, linter, or CI configuration in the repo.

## Running the game

No build is needed. Open the file directly, or serve the directory with any static server:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Architecture of `index.html`

The inline script is one IIFE-style block built around a single `requestAnimationFrame`
loop (`loop(now)`, around line 1640). Major sections, in file order:

- **DOM references** (~lines 426–461): every element is looked up once by id. Keep these
  in sync with the markup if you rename or remove elements.
- **Persistence** (~lines 463–466): `localStorage` keys prefixed `skyfighter.` — best
  score, mute, difficulty, and reduced-motion preference. Reads are not wrapped in
  try/catch; the game currently assumes storage is available.
- **Accessibility helpers**: `announce()` writes to a live region (`#srLive`) and
  `reducedMotion` gates screen-shake and flashes.
- **Performance scaling**: `trackPerf()` samples frame times and sets `quality`
  (0 = full, 1 = medium, 2 = low) to reduce effects on slow devices.
- **Tuning tables**:
  - `DIFFICULTIES` (~line 485): `easy` / `normal` / `hard`, controlling spawn rate,
    enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
  - `KILLS_PER_STAGE` (18) — kills required to advance a stage.
  - `THEMES` (~line 493), `ENEMY_TYPES` (~line 502), `BOSSES` (~line 508).
- **Audio**: a lazily created Web Audio context (`ensureAudio()`), with `beep()` and
  `noiseBurst()` building blocks and a `sfx` map of named sounds. Audio must be started
  from a user gesture; `ensureAudio()` is called on input.
- **Input**: keyboard (`keys` map keyed by `KeyboardEvent.code`), mouse (steer and
  click-to-fire on the canvas), and touch (virtual joystick in `#moveZone`, fire and
  bomb buttons).
- **Rendering**: a fixed logical canvas of `BASE_W` × `BASE_H` (480 × 720), scaled to fit
  the window by `resize()` up to 1.6×.

### Controls (as implemented)

| Action        | Keyboard                     | Other                          |
|---------------|------------------------------|--------------------------------|
| Move          | Arrows / WASD                | Mouse move; touch joystick     |
| Fire          | Space (hold)                 | Mouse button (hold); fire button |
| Bomb          | X or Shift (`useBomb()`)     | Bomb button                    |
| Pause         | P or Esc (`togglePause()`)   | Pause button                   |

Keep the on-screen `#controlsHint` text (around line 417) in sync when controls change.

## Conventions

- **Keep it a single file.** Styles and script stay inline in `index.html` unless the
  user asks for a split. Do not introduce a bundler, framework, or package manager
  without a request.
- **Use the existing style**: 2-space indentation, single quotes in JS, `const`/`let`,
  and the `skyfighter.*` prefix for any new storage keys.
- **Tune in the tables, not in the loop.** New difficulty values, enemy types, bosses,
  and themes belong in the constant tables above the game logic.
- **Respect accessibility hooks.** New visual shake, flash, or motion effects should
  check `reducedMotion`. Announce significant state changes through `announce()`.
- **Mobile matters.** The layout uses `touch-action: none` and safe-area insets. Verify
  touch paths when changing input code.
- **Don't add dependencies** for things the browser already provides.

## Verifying changes

There are no automated tests. For any change to `index.html`:

1. Load the page in a browser and start a game from each difficulty.
2. Exercise keyboard, mouse, and (if touched) touch input, plus pause, bomb, and game over.
3. Check the browser console for errors.
4. Resize the window to confirm the canvas still scales and the HUD still wraps.

When a headless browser is available, a short Playwright script that loads the page and
checks for console errors is a useful smoke test. Do not add it to the repo unless asked.

## Git and delivery

- Default working branch for this task: `claude/gifted-tesla-lzi7m9`.
- Commit with clear, imperative messages describing the gameplay or code change.
- Do not open a pull request unless the user explicitly asks for one.
- Do not commit local secrets or editor/OS junk files.

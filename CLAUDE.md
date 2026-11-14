# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The player switches weapons, collects power-ups, detonates smart bombs, and defeats stage bosses.

The entire game lives in one file, `index.html`, which contains inline CSS (`<style>`, roughly lines 1–362), the HTML markup (canvas, HUD, overlays, touch controls), and one inline `<script>` block (roughly lines 422–1667). There is no build step, no bundler, no package manager, and no external assets. Audio is synthesized with the WebAudio API.

`README.md` contains only the project title.

## Repository layout

```
index.html   # The whole game: markup, styles, and script
README.md    # Title only
```

There is no `package.json`, test suite, linter, or CI configuration. Do not invent build or test commands. If you add tooling, document the commands here.

## Running the game

No build is needed. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Code map (`index.html` script)

The script is organized into commented sections. Search for these headers when navigating:

| Section | Purpose |
| --- | --- |
| `Adaptive performance` | `trackPerf(dt)` tunes effects based on frame timing |
| `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` | Tunable game data |
| `Audio (WebAudio synth...)` | `ensureAudio`, `beep`, `noiseBurst`, and per-event sound effects |
| `Canvas sizing` | `BASE_W = 480`, `BASE_H = 720`; `resize()` scales the canvas |
| `Input` | Keyboard, mouse, and touch joystick handling |
| `Game state` | `resetGame()`, entity spawning, `update(dt)`, weapons, power-ups |
| `BOSS_PATTERNS` | Boss bullet-pattern functions (`fireRadial`, `fireCross`, etc.) |
| `draw()` | All rendering to the canvas |
| `loop(now)` | Main `requestAnimationFrame` loop |

Persistent settings use `localStorage` under keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Keep that prefix for any new keys.

## Controls

- **Move:** Arrow keys or WASD. Mouse: move to steer. Touch: drag on the left side to fly.
- **Fire:** Space, or mouse click (hold). Touch: tap the FIRE button.
- **Bomb:** `X` or Shift. Touch: tap the BOMB button.
- **Pause:** `P` or Escape

Keyboard input is handled in `window` `keydown`/`keyup` listeners and stored in the `keys` object by `e.code`. Space's default action is suppressed to avoid page scrolling.

## Conventions

- **Keep the single-file structure** unless the user asks for a split. Do not add a framework, bundler, or external dependency without discussing it first.
- **No external network requests.** The game must run fully offline from `index.html`. Do not add CDN scripts, fonts, or images.
- **Tune gameplay through the data tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding magic numbers in update or draw logic.
- **Respect the reduced-motion setting** when adding screen shake, flashes, or other motion effects. Check the existing `reducedMotion` flag and follow the pattern used by `screenShake` and `flashHit`.
- **Support both keyboard and touch.** Any new control needs a desktop input path and a touch path. Touch controls are shown only when `isTouchDevice` is true.
- **Keep the layout phone-friendly.** The viewport meta disables user scaling, and the canvas is sized to fit the screen. Verify changes at narrow widths.
- **Match the existing style:** 2-space indentation, single quotes in JavaScript, `const` and `let` (no `var`), and short section comments using `// ---------- Name ----------`.
- **Accessibility:** interactive controls are `<button>` elements. Announcements go through the existing `announce()` helper.

## Verifying changes

Because there are no automated tests, verify changes manually:

1. Open `index.html` in a browser and check the browser console for errors.
2. Play through at least one stage, including a boss fight, to confirm your change works.
3. Test each difficulty level if you changed tuning values.
4. Test pause, mute, and the reduced-motion toggle if your change touches them.
5. Check the touch layout with browser device emulation if you changed input or HUD code.

If you can run a browser in this environment, prefer checking the real page over reasoning from code alone.

## Git workflow

- Develop on the branch assigned for the session. Do not push to other branches without permission.
- Keep commits focused. Write commit messages that describe the change and why it was made.
- Do not open a pull request unless the user asks for one.

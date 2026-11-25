# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free static web project. Its only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs entirely in the browser.

| File | Purpose |
| --- | --- |
| `index.html` | The whole game: inline CSS, markup, and one inline `<script>` (~1,670 lines) |
| `README.md` | Minimal placeholder; no setup instructions yet |

There is no build step, no package manager, no test suite, and no external script or stylesheet dependencies. Do not add any of these unless the user asks.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Architecture of `index.html`

- **Styles**: one `<style>` block in `<head>`. The page is a fixed full-viewport container that centers a `<canvas id="gameCanvas">`. The dark color scheme is set via `color-scheme: dark`.
- **Markup**: HUD, overlays (start/pause/game over), touch controls (move zone, FIRE, BOMB), and a stage banner.
- **Script**: a single IIFE-style block of plain JavaScript. Sections are marked with `// ---------- Name ----------` comments, including Input, Audio, Game loop, and rendering. Keep that convention when adding code.
- **Logical resolution**: `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas to fit the viewport, capped at 1.6x.
- **Game loop**: `loop` drives `update` and `draw` each frame.
- **Persistence**: `localStorage` keys are namespaced `skyfighter.*` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). All reads and writes should tolerate storage being unavailable.
- **Tunables**: `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES`, `ENEMY_TYPES`, and `BOSSES` are the main constants to edit for balance and content. Boss attack patterns live in `BOSS_PATTERNS`, and weapon fire functions are named `fire*` (`fireRadial`, `fireSpiralBurst`, `fireHomingSeeded`, and so on).
- **Audio**: synthesized in code (`beep`, `noiseBurst`, `ensureAudio`), so there are no audio asset files. Audio starts only after a user gesture.

## Controls (for reference when changing input code)

- Keyboard: Arrows or WASD to move, Space to fire, X or Shift for a smart bomb, P or Esc to pause.
- Mouse: move to steer, click to fire.
- Touch: drag the left zone to fly, tap FIRE, tap BOMB. Touch controls appear only on touch-capable devices.

## Conventions

- Keep the project as a single self-contained HTML file unless the user asks to split it. If you split it, update this file.
- Use plain ES2015+ JavaScript and no frameworks or bundlers.
- Preserve mobile behavior: the layout must work at phone width with no horizontal scroll, and touch handlers use `{ passive: false }` where `preventDefault` is needed.
- Respect the reduced-motion setting when adding visual effects (`body.reduced-motion` class and the `reducedMotion` flag).
- Do not add analytics, network calls, or third-party assets.

## Verifying changes

There are no automated tests or linters configured. To verify a change:

1. Open `index.html` in a browser (or use the Playwright browser available in the cloud environment) and check the console for errors.
2. Play through at least one stage, including a boss, and confirm pause, bomb, and game-over flows still work.
3. For layout or input changes, check both a desktop viewport and a phone-sized viewport.

If you cannot run the game, say so explicitly and describe what you verified by reading the code.

## Git

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-lobqsz`). Do not push to other branches without explicit permission.
- Do not open pull requests unless the user asks.
- Keep commits focused, and describe the gameplay or file change in the subject line.

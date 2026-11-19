# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a single HTML5 canvas.

## Repository layout

- `index.html` — the entire game: inline `<style>`, markup (HUD, overlays, touch controls, canvas `#gameCanvas`), and one inline `<script>` IIFE containing all game logic, rendering, audio, and input.
- `README.md` — minimal project readme.
- No `package.json`, build step, bundler, test suite, or linter exists. Do not introduce one unless asked.

## Running and testing

- Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by loading the page and playing: check start, pause (`P`/`Esc`), bombs (`X`/`Shift`), stage transitions, boss fights, game over, and the best-score/mute/difficulty persistence.
- For visual or interaction checks, use the Playwright setup pre-installed in the environment (`require('playwright')`, `chromium.launch()`).

## Code structure and conventions (index.html)

- Everything lives in one IIFE; keep new code inside it so globals stay private.
- Constants are UPPER_SNAKE_CASE near the top of the script: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, `KILLS_PER_STAGE`, and `BASE_W`/`BASE_H` (the logical 480×720 playfield).
- Logical coordinates are always `BASE_W × BASE_H`; `resize()` scales the canvas for display. Don't hard-code screen pixels in game logic.
- Game loop: `loop(now)` → `update(dt)` → `draw()`. `dt` is in seconds and clamped to 0.05 s. Use `dt`-scaled movement, not per-frame increments.
- Persistence uses `localStorage` under keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Wrap storage access in try/catch, since storage can be unavailable in private windows.
- Audio is Web Audio synthesized in code (`beep`, `noiseBurst`); no audio asset files. Audio starts only after a user gesture via `ensureAudio()`.
- Boss attacks are data-driven through `BOSS_PATTERNS` and `fire*` helpers. Add new patterns there rather than inlining bullet logic in `updateBoss`.
- Respect the `reducedMotion` setting for any new screen shake, flashes, or particle effects.
- Input: keyboard (arrows/WASD move, Space fires, `X`/`Shift` bomb, `P`/`Esc` pause), mouse/pointer, and touch joystick + on-screen fire/bomb buttons. Any new control must work on both desktop and touch.

## Mobile and layout

- The page is designed for phones: keep the viewport meta, `touch-action: none`, safe-area insets, and no page scrolling.
- Test changes at phone width as well as desktop width.

## Style

- Plain HTML/CSS/JS, ES2015+ syntax, no frameworks or external CDN dependencies. Keep it that way unless the user asks otherwise.
- Match the existing 2-space indentation and single-quoted JS strings.
- Keep changes minimal and local; the file is large, so prefer targeted edits over rewrites.

## Git workflow

- Develop on the branch assigned to the session (currently `claude/gifted-tesla-xrhqdq`). Do not push to other branches without explicit permission.
- Do not create pull requests unless explicitly asked.

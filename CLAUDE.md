# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up written as a single-page HTML5 canvas game.

- `index.html` — the entire game: markup, CSS (inline `<style>`), and JavaScript (inline `<script>`, wrapped in an IIFE). About 1,670 lines.
- `README.md` — placeholder title only.

There is no build step, no package manager, no bundler, no test suite, and no external dependencies. The file runs by opening it in a browser.

## Running and testing

- Run: open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- Tests: none exist. Verify changes by playing the game in a browser. For changes that touch rendering or input, check both desktop (keyboard) and touch layouts.
- Do not add a framework, build tool, or dependency unless the user asks for one. Keeping the game as one self-contained file is intentional.

## Code structure inside `index.html`

The script is organized into sections, roughly in this order:

1. **DOM references** — `const xxxEl = document.getElementById(...)`. IDs are referenced directly, so renaming an element in the markup means updating the script too.
2. **Persistence keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`, all prefixed `skyfighter.`. Keep the prefix when adding new keys.
3. **Tuning tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (one per stage), `ENEMY_TYPES`, `BOSSES`. Balance changes usually belong here rather than in the logic.
4. **Audio** — a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, the `sfx` object). Audio is created lazily on the first user gesture.
5. **HUD and effects** — hearts, bombs, combo pops, stage banners, screen shake, hit flash.
6. **Input** — keyboard (`keys[e.code]`), touch joystick and fire/bomb buttons, and mouse on the canvas.
7. **Game state and logic** — `resetGame`, `update(dt)`, spawning, power-ups, weapon firing, bombs, boss patterns (`BOSS_PATTERNS`, `updateBoss`), and life/game-over handling.
8. **Rendering** — `draw()` and its helpers (`drawPlane`, `drawBossShip`, `drawClouds`). `draw` runs once per frame from `loop(now)`.
9. **Startup** — the start button handler and the initial `requestAnimationFrame(loop)`.

Notes on the code:

- Everything lives in one closure. Do not expose globals unless there is a reason to.
- Game time uses a `dt` value passed into `update`. Keep new movement and timers frame-rate independent.
- An adaptive quality level (`quality`, 0–2) is driven by `trackPerf`. New expensive effects should respect it.
- A reduced-motion setting exists (`reducedMotion`, stored and also read from `prefers-reduced-motion`). New screen shake, flashing, or large motion effects should be gated by it.
- Accessibility: announcements go through `announce()` into the `srLive` live region. Use it for important state changes.

## Controls (for reference when testing)

- Move: arrow keys or WASD. Touch: left-side joystick.
- Fire: hold Space. Touch: fire button.
- Bomb: `X` or Shift. Touch: bomb button.
- Pause: `P` or Escape.

## Conventions

- Match the existing style: 2-space indentation, single quotes in JavaScript, `const` by default, and small named functions.
- Keep the HTML, CSS, and JS in `index.html` unless the user asks for a split. If you do split files, update this document.
- Commit messages should be short and describe the change. Work on the branch the session specifies; do not push to other branches without permission.

## Keeping this file current

Update this document when you change the project's structure, add files, add a build or test step, or change controls or storage keys. Keep it factual and short.

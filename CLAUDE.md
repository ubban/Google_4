# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

`Google_4` is a single-page, browser-only game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game lives in one file, `index.html` (inline `<style>` and one inline `<script>`, drawn on a `<canvas id="gameCanvas">`). `README.md` is a stub.

There is no build step, no package manager, no framework, no external assets, and no test suite. Audio is synthesized with WebAudio; graphics are drawn with the 2D canvas API.

## Running it

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There is nothing to install or compile. Do not add a bundler, package.json, or framework unless the user asks.

## Repository layout

```
index.html   # the whole game: markup, CSS, and JS
README.md    # minimal project readme
CLAUDE.md    # this file
```

Inside `index.html`, the `<script>` is organized by `// ---------- Section ----------` comment banners. Use those banners to find things:

- **Adaptive performance**: `trackPerf` and related frame-budget logic.
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`, `panFor`. Synth only, no audio files.
- **Canvas sizing**: `resize`.
- **Input**: keyboard, mouse steering, touch joystick, and fire/bomb buttons.
- **Game state**: `resetGame`, `update`, `draw`, `loop`, `endGame`, `togglePause`.
- **Enemies, bosses, weapons**: `spawnEnemy`, `spawnBoss`, `updateBoss`, `killBoss`, and the `fire*` pattern functions (`fireRadial`, `fireSpiralBurst`, `fireHomingSeeded`, etc.). `BOSS_PATTERNS` maps each boss to its attack set.

Tuning data is declared as `const` tables near the top of the script: `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, and `KILLS_PER_STAGE`. Prefer changing these tables over hard-coding numbers in logic.

## Conventions

- **Keep it one file.** New game code goes in the existing `<script>`, styles in the existing `<style>`, and markup in `<body>`. Splitting into modules or files is a larger decision; ask before doing it.
- **No dependencies.** Use vanilla JS and the browser APIs already in use. Do not pull in libraries from a CDN without an explicit request.
- **Match the existing style.** 2-space indentation, single quotes in JS, `const`/`let` (no `var`), arrow functions for callbacks, and the section banner comments.
- **Data-driven tuning.** Add new enemies, bosses, difficulty settings, or themes as entries in the existing tables rather than new special-case branches.
- **Stage/boss flow.** When `stageKills` reaches `KILLS_PER_STAGE` (18), a boss warning is shown and `spawnBoss()` runs about 1.2 s later. Each boss's attacks come from `BOSS_PATTERNS[designIndex]`, falling back to index 0. Keep the warning, spawn, and pattern lookup consistent when changing any of them.

## Persistence (`localStorage`)

The game stores four keys. Keep these names stable; changing one silently resets player data.

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | Best score (number) |
| `skyfighter.muted` | `'1'` when audio is muted |
| `skyfighter.difficulty` | `'easy'`, `'normal'`, or `'hard'` |
| `skyfighter.reducedMotion` | `'1'` / `'0'`; if unset, falls back to the `prefers-reduced-motion` media query |

All reads and writes should tolerate missing or malformed values (the existing code uses `|| default` fallbacks).

## Controls (for manual testing)

- **Move:** Arrow keys or WASD; mouse hover steers the plane; touch uses an on-screen joystick.
- **Fire:** Hold `Space`, or hold the fire button on touch.
- **Bomb:** `X` or `Shift`, or the bomb button.
- **Pause:** `P` or `Escape`.
- Keyboard and joystick input take priority over mouse steering.

## Testing and verification

There is no automated test suite. When you change gameplay, verify by:

1. Opening `index.html` in a browser (or a local static server) and playing through at least one stage and one boss fight.
2. Checking the console for errors on load and during play.
3. Testing each difficulty and both mute and reduced-motion states if the change touches them.
4. Checking the layout at phone width if you touch the CSS or touch controls (the page uses `viewport-fit=cover` and safe-area insets).

Say plainly in your summary when something was not verified in a browser.

## Git

- Default branch work should happen on a feature branch; do not commit directly to the default branch.
- Commit messages: short imperative subject line, optional body explaining the why.
- Only commit or push when asked, unless the task explicitly calls for it.

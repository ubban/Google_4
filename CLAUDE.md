# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The whole game lives in one file, `index.html` (~1,670 lines, inline `<style>` and inline `<script>`). There is no build step, package manager, test suite, or external asset.

Files:

- `index.html` — the entire game (markup, CSS, JavaScript, canvas rendering, audio, input).
- `README.md` — placeholder title only.
- `CLAUDE.md` — this file.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Code layout inside `index.html`

- **`<style>`** — dark color scheme (`color-scheme: dark`), full-viewport fixed `#container`, a centered `#gameCanvas`, HUD elements (`#hud`, `#weaponStat`, `#bombs`, `#bossBarWrap`, `#stageBanner`, `#flash`), touch controls (`#fireBtn`, `#bombBtn`), and a `body.reduced-motion` class that disables banner/flash animations.
- **Markup** — HUD, boss bar, stage banner, and overlay screens (title, pause, game over, and so on). `#controlsHint` documents the controls.
- **`<script>`** — a single IIFE-style block that holds:
  - DOM lookups (`document.getElementById(...)`) grouped near the top.
  - Tuning tables: difficulty presets (`easy` / `normal` / `hard`) with spawn rates, enemy speed, enemy fire chance, lives, bombs, and boss HP multiplier; `KILLS_PER_STAGE`; enemy definitions (`grunt`, `interceptor`, `tank`) with size, HP, speed, score, and `minStage`; and the `BOSSES` list.
  - Persistent settings via `localStorage` keys (best score, mute, difficulty, reduced motion). Every access is a plain `localStorage` call, so keep that in mind if you change storage behavior.
  - Audio: a small `beep`-based sound set (shots, bomb, boss hit/telegraph/down, stage clear) — no audio files.
  - Game loop driven by `requestAnimationFrame`, with update and draw steps on a `<canvas>`.

When changing behavior, find the relevant tuning table or function with `Grep` first rather than reading the whole file.

## Gameplay facts to keep consistent

- Weapons upgrade via power-ups; the HUD shows the current weapon name (starts as `VULCAN`) and level.
- Smart bombs are shown as icons in `#bombs`; the count depends on difficulty.
- Each stage ends after `KILLS_PER_STAGE` (18) kills, which spawns that stage's boss. Bosses cycle through the `BOSSES` list by `(stage - 1) % BOSSES.length`, and the boss HP bar is shown in `#bossBarWrap`.
- Controls (from the in-game hint): mouse to steer and click to fire; arrows or WASD to move; Space to fire; X or Shift for bomb; P or Esc to pause. Mobile: drag left to fly, tap FIRE, tap BOMB.

## Conventions

- Keep the game as a **single self-contained HTML file** unless the user asks for a split. Do not add a bundler, framework, or npm dependency without asking.
- Use plain ES2015+ JavaScript and the existing style: `const`/`let`, the same naming scheme, and the existing tuning-table pattern for new enemies, bosses, or difficulty values.
- Preserve mobile support: the page sets `touch-action: none` and `user-select: none`, and the viewport meta disables zoom. Test touch changes at phone width.
- Preserve the `reduced-motion` path when adding new animations.
- Do not introduce external network requests, CDN scripts, or remote fonts; the game is meant to run offline from one file.

## Verification

There are no automated tests or linters configured. To verify a change:

1. Open `index.html` in a browser (or serve it as shown above) and check the console for errors.
2. Exercise the changed path: start a game, fire, bomb, pause/resume, let a stage advance, reach a boss, and game over.
3. For layout changes, check the HUD and controls at both desktop and phone-sized viewports.

If you add a test harness or tooling later, document the commands here.

## Git

- Default branch work happens on the branch the session specifies; do not push to other branches without permission.
- Keep commits focused. Commit messages should describe the gameplay or code change, not the process.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small static web project containing one game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up.

- `index.html` — the entire game: markup, CSS (inline `<style>`), and JavaScript (inline `<script>`, one IIFE). About 1,700 lines.
- `README.md` — a one-line title. No other docs.

There is no build step, no package manager, no dependencies, no test suite, and no linter configuration. Do not add any of these unless the user asks.

## Running the game

Open `index.html` directly in a browser. No server is required.

For a quick automated check, a headless Chromium with Playwright works (see the environment notes for the pre-installed browser). Load the file with `file://`, start a game, and check the browser console for errors.

## Code layout inside `index.html`

Roughly in order of appearance:

1. **`<head>`** — meta tags, an inline SVG favicon, and the `<style>` block. Colors are literal values (no CSS custom properties): the blue sky gradient plus HUD colors.
2. **`<body>`** — the `#container` with `#gameCanvas`, the HUD (`#hud`: score, best, multiplier, weapon, hearts, bombs, stage), overlays (start/pause/game-over), touch controls (`#touchControls`, `#moveZone`, `#fireBtn`, `#bombBtn`), and an ARIA live region (`#srLive`).
3. **Script IIFE** — all game logic lives in one `(function () { ... })();` block. Key sections, in source order:
   - DOM references (`const xEl = document.getElementById(...)`).
   - Storage keys and persisted state.
   - Audio: lazily created `AudioContext` (`ensureAudio`), with `beep` and `noiseBurst` helpers. Sound is synthesized; there are no audio files.
   - Performance tracking (`trackPerf`) that steps a `quality` level (0 full, 1 medium, 2 low) when frame times slip.
   - Config tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (one per stage, cycling), `ENEMY_TYPES`, `BOSSES`.
   - Input: keyboard (`keys[e.code]`), mouse, and touch handlers.
   - Game loop (`requestAnimationFrame(loop)`), entity updates, collision, rendering, HUD updates.

## Controls and settings (keep in sync with the overlay text)

- Keyboard: arrows/WASD move, Space fires, X or Shift uses a bomb, P or Esc pauses.
- Mouse: move to steer, click to fire.
- Touch: drag left side to fly, tap FIRE, tap BOMB.
- Difficulty (`easy` / `normal` / `hard`) sets lives, bombs, spawn rate, enemy fire chance, and boss HP multiplier.

The control hint text lives in `#controlsHint` and the overlay copy in `#overlaySub`. If you change a key binding, update both.

## Persisted state

Stored in `localStorage` under these keys. The reads and writes are not wrapped in try/catch today, so private-browsing or blocked-storage environments can throw. If you touch this code, consider adding guards.

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | Best score |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | Selected difficulty |
| `skyfighter.reducedMotion` | Reduced-motion override (absent means follow the OS preference) |

Renaming a key silently resets that player's saved value. Do not rename keys without a migration.

## Conventions

- **Keep it a single file.** Game code, styles, and markup stay in `index.html` unless the user asks for a split. Do not introduce a bundler, framework, or CDN dependency.
- **Vanilla JS, ES2015+.** Match the existing style: `const`/`let`, arrow functions for callbacks, semicolons, two-space indentation.
- **Respect the accessibility hooks.** Reduced-motion mode toggles `body.reduced-motion`, which turns off the flash, vignette, combo, banner, and heart-pulse animations. Announcements for screen readers go through `announce()` into `#srLive`. New visual effects should check `reducedMotion`.
- **Keep mobile working.** The layout uses `viewport-fit=cover`, safe-area insets, and `touch-action: none`. Test any HUD or control change at phone width.
- **Preserve the quality tiers.** New particle or draw-heavy effects should respect `quality` so the game degrades on slow devices instead of stuttering.
- **Balance numbers live in the config tables.** Prefer adjusting `DIFFICULTIES`, `ENEMY_TYPES`, or `BOSSES` over hard-coding values in logic.

## Working in this repo

- Make focused edits. Do not reformat the whole file, and do not reorder sections just to tidy them.
- After a gameplay change, play a short run in a browser to confirm there are no console errors and the change behaves as intended. Say so plainly if you could not run it.
- Git: the default working branch for automated sessions is `claude/gifted-tesla-k5i6qu`. Do not push to other branches without permission, and do not open a pull request unless asked.

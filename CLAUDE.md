# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up built as a single-page HTML5 canvas game. The entire game (markup, CSS, and JavaScript) lives in `index.html`. `README.md` is a placeholder and has no project documentation.

There is no build step, package manager, bundler, framework, or test suite. There is no `package.json`.

## Repository layout

```
index.html   # The whole game: <style> (~lines 10-361), HTML overlay/HUD (~362-421), <script> (~422-1668)
README.md    # Placeholder
```

Inside the `<script>`, everything sits in one IIFE with `'use strict'`. Sections are marked with `// ---------- Name ----------` comments: Adaptive performance, Input, Audio, HUD, Resize, Touch controls, Game logic, Drawing, Main loop.

## Running and verifying

- Open `index.html` directly in a browser. No server is required.
- There are no automated tests or linters configured. Verify changes by playing the game in a browser, especially the affected system (weapons, bosses, HUD, touch controls, audio).
- For UI or gameplay changes, check both desktop (keyboard/mouse) and touch layouts. The touch controls appear only when `'ontouchstart' in window` or `navigator.maxTouchPoints > 0`.
- Check the layout at phone width as well as desktop width.

## Game design reference

Keep these values in sync with the code if you change them:

- **Base resolution:** `BASE_W = 480`, `BASE_H = 720`, scaled to fit the viewport in `resize()`.
- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- **Enemies** (`ENEMY_TYPES`): `grunt`, `interceptor` (weaving), `tank` (4 HP, appears from stage 2).
- **Stages:** `KILLS_PER_STAGE = 18`. Each stage advances the background theme (`THEMES`, six themes that cycle). Stages end in a boss from `BOSSES` (six bosses), each with its own attack patterns in `BOSS_PATTERNS`.
- **Weapons:** `vulcan` (spread shot, up to 3 levels) and `laser` (piercing lanes, up to 3 levels). Power-up `weapon` increases level, then switches weapon type after level 3.
- **Power-ups:** `life`, `bomb`, `weapon`. Dropped by `maybeSpawnPowerUp()`.
- **Bombs:** clear the screen and damage bosses. Triggered with `X`, `Shift`, or the touch bomb button.
- **Score:** kill-chain combo multiplier (`addScore`, `breakChain`).

## Controls

| Action | Keyboard | Touch |
|---|---|---|
| Move | Arrow keys / WASD (via `keys[e.code]`) | Left-side virtual joystick |
| Fire | Hold `Space` (see `firing` state) | Fire button (hold) |
| Bomb | `X` or `Shift` | Bomb button |
| Pause | `P` or `Esc` | Pause button |
| Mouse | Move pointer over canvas to steer; click to fire | n/a |

## Persistence

`localStorage` keys, all prefixed `skyfighter.`:

| Key | Meaning |
|---|---|
| `skyfighter.bestScore` | Best score |
| `skyfighter.muted` | Audio muted (`'1'`/`'0'`) |
| `skyfighter.difficulty` | Selected difficulty |
| `skyfighter.reducedMotion` | Reduced-motion toggle; defaults to the `prefers-reduced-motion` media query when unset |

Storage access is not wrapped in try/catch. If you add new persisted state, follow the same prefix and keep it simple.

## Conventions

- **Single file:** Keep the game in `index.html` unless the user asks for a split. Do not add a build system, dependencies, or external CDN scripts without asking.
- **Audio:** All sound is synthesized with the Web Audio API through helpers (`beep`, `noiseBurst`, the `sfx` object). Do not add audio files.
- **Accessibility:** Announce important state changes through `announce()` (`#srLive`). Respect `reducedMotion`. Preserve the `aria-live` region and the reduced-motion class on `<body>`.
- **Performance:** The loop caps `dt` at 0.05s and adapts `quality` (0/1/2) based on frame time. New visual effects should respect this, and effects should degrade gracefully at low quality.
- **Style:** Match the existing code: `const`/`let`, arrow callbacks, 2-space indentation, and section comments. Keep changes focused and avoid unrelated refactors.

## Git

- Develop on the branch assigned for the session. Do not push to other branches without permission.
- Commit messages should describe the change in plain language.

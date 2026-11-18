# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a single-file, Raiden-style vertical shoot-'em-up that runs in the browser on a `<canvas>`. There is no build step, no package manager, and no test suite.

Repository contents:

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` wrapped in an IIFE with `'use strict'`.
- `README.md` — placeholder only.

To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Layout of `index.html`

| Region | Approx. lines | Contents |
| --- | --- | --- |
| `<style>` | 10–360 | All CSS, including HUD, overlay, touch controls, responsive layout, and `body.reduced-motion` overrides |
| Markup | 360–420 | `#gameCanvas`, HUD stats (`#score`, `#best`, `#mult`, `#weaponStat`, `#hearts`, `#bombs`), overlays, pause/mute/motion buttons, touch controls, live region `#srLive` |
| Script setup | 422–462 | DOM lookups, `STORAGE_*` keys |
| Config tables | 463–527 | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` |
| Audio | ~545–680 | Lazy Web Audio context (`ensureAudio`), `beep`, `noiseBurst`, `sfx` |
| HUD/FX helpers | ~680–750 | `renderHearts`, `renderBombs`, flashes, combo pop, stage banner, screen shake, mute/difficulty handlers |
| Input | ~753–870 | `resize` (base canvas 480×720), keyboard, virtual joystick, touch fire/bomb, mouse steering |
| Game state | ~864–912 | `resetGame()` and the shared mutable state (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`) |
| Entities | ~913–1150 | `drawPlane`, `drawBossShip`, spawning, `useBomb`, `fireWeapon` |
| Update | ~1148–1305 | `update(dt)` — movement, firing, collisions, stage progression |
| Boss patterns | ~1304–1455 | `applyPowerUp`, the `fire*` pattern helpers, `BOSS_PATTERNS`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause` |
| Render | ~1507–1640 | `drawClouds`, `draw()` |
| Loop | ~1640–1668 | `loop(now)` via `requestAnimationFrame`; start button handler |

Everything is inside one IIFE, so nothing is exposed on `window`. Keep new code inside that scope unless there is a reason to export it.

## Gameplay model

- **Controls:** mouse moves and clicks to fire; Arrows/WASD move; Space fires; X or Shift bombs; P or Esc pauses. Touch uses a left-side drag joystick plus FIRE and BOMB buttons.
- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn timing, enemy speed, enemy shoot chance, starting lives, starting bombs, and a boss HP multiplier.
- **Stages:** you advance after `KILLS_PER_STAGE` (18) kills. The stage then spawns a boss (after a 1.2 s delay). Stage themes cycle through `THEMES`; bosses cycle through `BOSSES` by stage.
- **Enemies** (`ENEMY_TYPES`): `grunt`, `interceptor` (weaving), and `tank` (4 HP, unlocked at stage 2).
- **Power-ups** drop as `life`, `bomb`, or `weapon`. The weapon ladder is shown in the HUD (`VULCAN` and its level).
- **Combo multiplier** (`#mult`) builds through `addScore`/`breakChain`.
- **Persistence** uses `localStorage` with these keys, all prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`, `reducedMotion`. Every access to `localStorage` should be treated as something that can fail (private mode, blocked storage), and the game should still run without it.
- **Accessibility:** `reducedMotion` defaults from `prefers-reduced-motion` and can be toggled. When it is on, screen shake, hit flashes, and heart-beat animations are dampened or skipped. Announcements go through `announce()` into the `#srLive` region.
- **Adaptive quality:** `trackPerf` samples frame times and steps a `quality` level (0–2) up or down based on average frame time. Keep expensive effects behind this level where practical.

## Conventions

- **No dependencies.** Do not add libraries, bundlers, or a `package.json` unless the user asks. Plain HTML/CSS/JS is the design.
- **Keep it a single file** unless the user explicitly asks to split it up.
- **Use the existing style:** `const` and `let`, arrow functions for callbacks, two-space indentation, single quotes in JS, and the section banner comments (`// ---------- Name ----------`).
- **Use `dt`-based updates.** Movement and timers are multiplied by the frame delta so behavior stays consistent across refresh rates.
- **Put tunable numbers in the config tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than burying them in logic.
- **Guard audio** with `if (muted || !audioCtx) return;` and create the `AudioContext` only from a user gesture via `ensureAudio()`.
- **Respect `reducedMotion`** for any new shake, flash, or flicker effect.
- **Mobile first:** the canvas is scaled from a 480×720 base; check that new HUD elements still fit at phone width and do not block the touch controls.

## Making changes

1. Open `index.html` in a browser (or serve the folder) and play through at least one stage and one boss after any gameplay change.
2. Check all three difficulties if you touch `DIFFICULTIES` or spawn logic.
3. Check the mobile layout (narrow viewport and touch emulation) if you touch CSS or HUD markup.
4. Check the console for errors. The game should run with `localStorage` disabled.
5. Keep commits focused. Describe gameplay-visible changes in the commit message.

## Git

- Development branch for this work: `claude/gifted-tesla-wylefd`.
- Do not force-push or rewrite shared history.
- Do not open a pull request unless asked.

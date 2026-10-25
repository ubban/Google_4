# CLAUDE.md

Guidance for working in this repository.

## What this is

**Sky Fighter** — a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is one self-contained file, `index.html` (HTML, CSS, and JavaScript inline). `README.md` is a placeholder.

There is no build step, no package manager, no bundler, no framework, and no test suite. Do not add one unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server. Verify gameplay changes by playing them in a browser; there are no automated tests to run.

## File layout (`index.html`)

- **Lines ~1–9:** `<head>` meta tags, title, and an inline SVG favicon.
- **Lines ~10–361:** all CSS. Layout is a 480×720 logical canvas scaled to fit the viewport (`BASE_W` / `BASE_H`), with touch controls and a mobile-friendly layout.
- **Lines ~362–421:** the DOM. Key IDs: `#gameCanvas`, `#hud` (`#score`, `#best`, `#mult`, `#hearts`, `#bombs`, `#weaponName`, `#weaponLvl`), `#bossBarWrap`, `#comboPop`, `#stageBanner`, `#overlay` (start, pause, and game-over card), `#touchControls`, and `#srLive` (screen-reader live region).
- **Lines ~422–1668:** one IIFE containing all game logic, wrapped in `'use strict'`. Nothing is exported or global.

Within the script, the main sections are, in order:

| Section | Purpose |
|---|---|
| Config tables | `DIFFICULTIES`, `THEMES` (stage backdrops), `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE` |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object (Web Audio, synthesized, no asset files) |
| HUD helpers | `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake` |
| Input and layout | `resize`, joystick and touch handling, `canvasPointFromEvent`, keyboard handlers |
| State and spawning | `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion` |
| Update | `update(dt)`, `applyPowerUp`, `useBomb`, `fireWeapon`, `addScore` / `breakChain` |
| Boss patterns | `BOSS_PATTERNS` plus the `fire*` helpers, driven by `updateBoss` |
| Lifecycle | `loseLife`, `killBoss`, `endGame`, `togglePause` |
| Rendering | `drawPlane`, `drawBossShip`, `drawClouds`, `draw()` |
| Main loop | `loop(now)` — clamps `dt` to 0.05 s, runs `trackPerf`, then `update` and `draw` while running and unpaused |

## Game model

- **Player:** starts with `vulcan` at level 1. Weapon power-ups raise the level up to 3, then toggle between `vulcan` and `laser`. Life and bomb power-ups add to their caps.
- **Difficulty** (`easy` / `normal` / `hard`) sets spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
- **Stages:** a stage ends after `KILLS_PER_STAGE` (18) kills and triggers a boss. Each stage picks a theme from `THEMES` and a boss from `BOSSES`, cycling by stage number.
- **Enemies:** `grunt` (stage 1+), `interceptor` (weaving, stage 1+), `tank` (4 HP, stage 2+).
- **Scoring:** every kill extends a combo chain. The multiplier is `1 + floor(chain / 5)`, capped at 9x. The chain resets after 2.2 s with no kills (`breakChain`).
- **Quality scaling:** `trackPerf` drops render quality (`quality` 0–2) when frame time runs long and restores it when frames are fast.

## Controls

- **Keyboard:** arrows or WASD to move, Space to fire, X or Shift to bomb, P or Esc to pause. The `keydown` handler is registered on `window`.
- **Mouse:** move to steer, click to fire.
- **Touch:** drag the left zone to fly, tap FIRE, tap BOMB.

## Persistence

`localStorage` keys (all prefixed `skyfighter.`):

- `bestScore` — personal best, written on game over when beaten
- `muted` — `'1'` when muted
- `difficulty` — last selected difficulty
- `reducedMotion` — explicit override; when unset, the `prefers-reduced-motion` media query is used

Reads and writes are not wrapped in try/catch. If you add more storage, guard it the same way or accept that private-mode browsers may throw.

## Conventions

- Match the existing style: 2-space indent, single quotes in JS, `const` by default, `let` where reassigned, and arrow functions for callbacks.
- Section headers use `// ---------- Name ----------`. Follow that pattern for new sections.
- Keep new code inside the existing IIFE. Do not introduce globals.
- Gameplay numbers are inline in the config tables. Tune them there rather than hardcoding them in logic.
- Accessibility: announce important state changes through `announce()` (the `#srLive` region), and respect `reducedMotion`. It currently scales down screen shake in `screenShake()`; any new motion effect should do the same.
- Keep the game working at phone width and on touch as well as on desktop.

## Working on this repo

- Development branch: `claude/gifted-tesla-xmhk6v`.
- There is no lint, format, or test command. Check changes by loading the page and playing through a few stages on at least one difficulty.
- When changing a sound effect, check that it still runs after the first user gesture (browsers block Web Audio before one).

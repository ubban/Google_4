# CLAUDE.md

Guidance for working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up built as a single-page HTML5 canvas app.

## Repository layout

```
.
├── index.html   # The entire game: markup, CSS, and JavaScript in one file
└── README.md    # Minimal project title
```

There is no build system, package manager, test suite, or linter configured.

## Running the game

No build step is needed. Open `index.html` directly in a browser, or serve the directory with any static file server, for example:

```sh
python3 -m http.server 8000
```

## Architecture of `index.html`

The file is organized in three blocks:

1. **`<style>`** (lines 10–359): all CSS, including the HUD pills, glassmorphic overlay card, and touch controls. Reduced motion is handled in JavaScript, not in a CSS media query.
2. **`<body>`** (lines 361–421): the DOM skeleton. Key IDs include `gameCanvas`, `hud`, `bossBarWrap`, `comboPop`, `stageBanner`, `touchControls`, `overlay`/`overlayCard`, and `diffRow`.
3. **`<script>`** (starts at line 422): all game logic, wrapped in a single `(function () { 'use strict'; ... })();`. Major sections, in order:
   - DOM element lookups and `STORAGE_*` keys
   - Performance tracking (`trackPerf`, `quality` 0–2) that lowers render cost on slow devices (fewer particles, no shadow glow)
   - Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`
   - Settings persistence (best score, mute, difficulty, reduced motion)
   - Audio: a lazily created `AudioContext` (`ensureAudio`), with `beep`, `noiseBurst`, and the `sfx` map
   - HUD and feedback helpers (`renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`)
   - Input: keyboard (`keys`), virtual joystick and buttons for touch, and mouse steering/firing on the canvas
   - Entity spawning and updates: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `applyPowerUp`
   - Boss attack patterns (`BOSS_PATTERNS`, with `fireRadial`, `fireAimedSpread`, `fireCross`, and so on), plus `updateBoss` and `killBoss`
   - Game flow: `loseLife`, `endGame`, `togglePause`
   - Rendering: `draw`, `drawClouds`, and the per-entity draw functions
   - Main loop: `loop(now)` with a delta-time step driven by `requestAnimationFrame`

### Persisted state

`localStorage` keys (all prefixed `skyfighter.`):

| Key | Value |
| --- | --- |
| `skyfighter.bestScore` | Number |
| `skyfighter.muted` | `'1'` or `'0'` |
| `skyfighter.difficulty` | `'easy'`, `'normal'`, or `'hard'` |
| `skyfighter.reducedMotion` | `'1'` or `'0'`; absent means follow the OS preference |

Changing a key's name or format will reset existing players' saved settings, so treat these as a public contract.

### Controls

- Keyboard: arrows/WASD to move, Space to fire, X or Shift to bomb, P or Esc to pause
- Mouse: move to steer, click to fire
- Touch: drag left zone to fly, FIRE and BOMB buttons

When you change a control, update the `#controlsHint` text in the markup as well.

### Game tuning

Balance lives in constant tables near the top of the script. Prefer editing these over changing logic:

- `DIFFICULTIES`: spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier
- `ENEMY_TYPES`: size, HP, speed, score value, and the first stage each type appears in
- `BOSSES`: name, colors, and HP multiplier per stage
- `THEMES`: background palette per stage (cycled)
- `KILLS_PER_STAGE`: kills needed to advance a stage

## Conventions

- **Vanilla JavaScript only.** Do not add frameworks, bundlers, or external CDN dependencies without a clear reason. The single-file, no-build layout is intentional.
- **Keep it in `index.html`** unless the file becomes unmanageable. If you split it, update this document in the same change.
- **Match the existing style:** 2-space indentation, `const` by default, single quotes in JS, short section comments in the form `// ---------- Name ----------`.
- **Guard storage access.** `localStorage` reads and writes should tolerate being unavailable (private browsing, blocked storage).
- **Respect reduced motion.** Any new screen shake, flash, or animation should check the `reducedMotion` flag.
- **Keep touch and keyboard parity.** New actions should be reachable on both input paths, or explicitly documented as desktop-only.
- **Mobile layout.** The canvas scales to fit the viewport (`resize()`, base size 480×720, max scale 1.6). Verify new HUD elements at phone widths.

## Verification

There are no automated tests. Before committing gameplay or UI changes:

1. Open `index.html` in a browser and play through at least one full stage, including a boss fight.
2. Check all three difficulties and confirm that best score persists after a reload.
3. Exercise pause (P/Esc), mute, and the reduced-motion toggle.
4. On a narrow viewport or with touch emulation, confirm the joystick, FIRE, and BOMB controls still work.
5. Check the browser console for errors.

## Git workflow

- Develop on the branch assigned for the task (currently `claude/gifted-tesla-3kip13`), not on `main`.
- Write descriptive commit messages that explain *why* a change was made.
- Do not open a pull request unless explicitly asked.

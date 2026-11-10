# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up played in the browser. The entire game is one self-contained file, `index.html` (HTML, CSS, and JavaScript inline, roughly 1,650 lines). `README.md` is a placeholder and has no project documentation.

There is no build step, no package manager, no dependencies, and no test suite. Do not add any of these unless the user asks.

## Running the game

Open `index.html` directly in a browser. Nothing needs to be installed or served. For a quick local server, run `python3 -m http.server` in the repo root and visit the page.

## File layout (`index.html`)

- **`<style>` (about lines 10–360):** all CSS, including the HUD pills, the glassmorphic overlay card, touch controls, and the reduced-motion and low-health vignette effects.
- **`<body>` markup (about lines 363–420):** the `<canvas id="gameCanvas">`, HUD elements, overlay with difficulty buttons and start button, touch joystick and FIRE/BOMB buttons, an `aria-live` region (`srLive`) for screen-reader announcements, and the controls hint text.
- **`<script>` (starts at about line 422):** a single IIFE with `'use strict'`. Sections, in order:
  1. DOM lookups and `STORAGE_*` keys
  2. Adaptive performance (`trackPerf`, which lowers render quality when frame times slip)
  3. Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`
  4. Audio (`ensureAudio`, `beep`, `noiseBurst`, and the `sfx` table)
  5. HUD helpers (hearts, bombs, combo pop, stage banner, screen shake)
  6. Input: keyboard, touch joystick, touch and mouse buttons, mouse steering on the canvas
  7. Game state: `resetGame`, `update(dt)`, spawning, power-ups, bombs, and weapons (`fireWeapon`)
  8. Bosses: `BOSS_PATTERNS` (bullet-pattern functions), `updateBoss`, `killBoss`
  9. Lifecycle: `loseLife`, `endGame`, `togglePause`
  10. Rendering: `draw()` and the `drawPlane` / `drawBossShip` / cloud helpers
  11. Main loop: `loop(now)`, followed by the start-button wiring

## Controls

- **Keyboard:** Arrow keys or WASD to move, Space to fire, X or Shift to use a bomb, P or Esc to pause.
- **Mouse:** move to steer, hold the button to fire.
- **Touch:** drag on the left to fly (virtual joystick), tap FIRE, tap BOMB.

Keyboard and joystick input take priority when active, so the two sources never fight each other.

## Game systems to know

- **Difficulty** (`DIFFICULTIES`): sets spawn rate, enemy speed, enemy shot chance, starting lives, starting bombs, and a boss HP multiplier. Chosen in the overlay and saved.
- **Stages:** every `KILLS_PER_STAGE` (18) kills advances the stage. Stages change the background theme (`THEMES`, cycled) and unlock enemy types by `minStage`. A boss fight follows the wave progression.
- **Enemies** (`ENEMY_TYPES`): `grunt`, `interceptor` (fast, weaving), and `tank` (4 HP, unlocked at stage 2).
- **Bosses** (`BOSSES`): six named bosses, each with a multi-phase bullet pattern from `BOSS_PATTERNS`, a health bar (`bossBarWrap`), and an HP multiplier.
- **Weapons:** Vulcan fires spread shots that widen with weapon level (1–3, raised by power-ups). A separate power-up toggles between Vulcan and Laser (`applyPowerUp`). Laser fires faster (0.1 s delay vs 0.15 s).
- **Bombs:** a stock set by difficulty (`bombs`). They clear the screen and damage bosses.
- **Scoring:** a kill-chain multiplier (`breakChain` resets it) and a combo popup.
- **Power-ups:** weapon level, bomb, and extra life (`applyPowerUp`, `maybeSpawnPowerUp`).
- **Adaptive quality:** `quality` (0 full, 1 medium, 2 low) is lowered automatically when the frame rate drops.

## Persistence

The game uses `localStorage` with these keys. Wrap reads and writes in `try/catch` when editing, because storage can be unavailable (private windows, blocked site data).

| Key | Meaning |
|---|---|
| `skyfighter.bestScore` | best score (written in `endGame`) |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | `'1'` or `'0'`. If unset, the game falls back to the OS preference. |

Renaming a key resets that player's saved value, so change keys only on purpose.

## Conventions

- Keep the game as a single file with no external dependencies, unless the user explicitly asks for a split.
- Keep new code inside the existing IIFE and use the existing style: `const` and `let`, arrow functions, and two-space indentation.
- Put new tuning values in the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding numbers in logic.
- Keep the game loop time-based (`dt`). Do not assume a fixed frame rate.
- Respect the accessibility features: keep announcements going through `announce()` and honor `reducedMotion` for new screen-shake or flash effects.
- Keep touch and keyboard behavior in parity when you add a control. Update the `controlsHint` text to match.

## Verification

There are no automated tests. After a change, load `index.html` in a browser and check the affected behavior by hand. Cover at least: starting a game, firing and bombing, taking a hit, a stage transition, a boss fight, pausing, game over with a best-score update, and the mobile layout (touch controls on a narrow viewport). If you cannot run a browser, say so in your report instead of claiming the change works.

## Git

- Develop on the branch the session names. Do not push to `main` unless told to.
- Write descriptive commit messages in the imperative mood, with a short summary line and a body that explains the why for non-trivial changes.

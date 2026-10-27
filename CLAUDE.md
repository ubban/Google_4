# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser as an HTML5 canvas game.

- `index.html` — the entire game: markup, CSS, and JavaScript in one self-contained file (~1,670 lines).
- `README.md` — placeholder title only.

There is no build step, package manifest, bundler, test suite, or linter. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server` from the repo root) and visit it on localhost. No install step is needed.

## Code layout inside `index.html`

- **Markup and CSS** (roughly lines 1–420): HUD pills, the start/pause overlay card, touch controls, boss health bar, vignette and flash effects. Colors and sizing are plain CSS with `clamp()` for responsive text.
- **Script** (inline `<script>` starting around line 422): one IIFE wrapped in `'use strict'`. Nothing is exported to `window`; keep new code inside the IIFE.
- Key sections of the script, in file order:
  - Config constants: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
  - Persistence keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`.
  - Audio: a Web Audio synth (`beep`, `noiseBurst`, `sfx.*`). Audio is created lazily via `ensureAudio()` on the first user gesture.
  - Input: keyboard (`keys[e.code]`), mouse, and touch (joystick plus FIRE/BOMB buttons).
  - Game state and loop: `resetGame`, `update(dt)`, `draw()`, `loop(now)`. Spawning, collisions, and scoring live in `update`.
  - Boss logic: `updateBoss`, `killBoss`, and the bullet-pattern helpers (`fireRadial`, `fireSpiralBurst`, and so on).
  - Adaptive quality: `trackPerf` lowers the effects level when the frame rate drops.

## Game rules (for reference when changing balance)

- **Difficulty** (easy / normal / hard): sets starting lives (4 / 3 / 3), starting bombs (3 / 2 / 2), spawn rate, enemy speed, enemy fire chance, and boss HP multiplier (0.8 / 1.0 / 1.3).
- **Stages**: each stage requires `KILLS_PER_STAGE` (18) kills, then a boss fight. Stages cycle through `THEMES` (sky palettes).
- **Enemies**: `grunt` (1 HP), `interceptor` (weaving, fast), and `tank` (4 HP, from stage 2).
- **Bosses**: six in `BOSSES`, each with its own HP multiplier and firing patterns.
- **Weapons**: Vulcan and Laser, each with 3 power levels. Weapon power-ups raise the level, then switch weapon type.
- **Power-ups**: `life`, `bomb`, `weapon`.
- **Score**: kill chain multiplier, reset by `breakChain()`.

## Controls

- Keyboard: arrows or WASD to move, Space to fire, X or Shift to bomb, P or Esc to pause.
- Mouse: move to steer, click to fire.
- Touch: drag left side to fly, tap FIRE, tap BOMB.
- Mute, reduced-motion, and pause buttons are icon buttons in the game UI (`#muteBtn`, `#motionBtn`, `#pauseBtn`).

## Conventions

- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let`, short section comments in the `// ---------- Name ----------` form.
- Keep everything in `index.html` unless the user asks for a split. Do not introduce a framework or a bundler.
- Any new persisted setting should get a `skyfighter.*` localStorage key and be wrapped the same way the existing ones are.
- Balance changes belong in the config constants at the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`), not scattered through the update loop.
- Accessibility matters here: keep `aria-live` announcements (`announce()`) working for state changes, and respect `reducedMotion`.
- Keep the game playable on phones: touch targets, `touch-action: none`, and the `clamp()`-based sizing should stay intact.

## Verification

With no test tooling in the repo, verify changes by running the game in a browser and playing through the affected path (start, fire, bomb, a stage clear, a boss fight, game over). Check the browser console for errors. If you add tooling later, document its commands here.

## Git

- Develop on the branch named in the session instructions. Do not push to other branches without explicit permission.
- Write clear commit messages that describe the gameplay or code change.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style
vertical shoot-'em-up. The entire game lives in a single file, `index.html`
(about 1,670 lines: inline `<style>`, markup, and one inline `<script>` wrapped in an
IIFE). `README.md` is a placeholder.

There is no build step, no package manager, no bundler, and no test suite. Nothing
to install.

## Running and checking changes

- Open `index.html` directly in a browser, or serve the directory with any static
  server (for example `python3 -m http.server`).
- Verify changes by playing them in a browser. Check at least: start, fire, bomb,
  pause/resume, a stage transition, a boss fight, game over, and the restart path.
- Also check touch controls and a narrow (phone-sized) viewport if the change touches
  layout, HUD, or input.
- For UI or gameplay changes, a headless Playwright run with a screenshot is a useful
  sanity check; the browser is pre-installed in cloud sessions.

## Architecture (inside `index.html`)

**Layout of the file**

- `<style>`: dark-theme styling. Colors and HUD sizing use `clamp()`/`vw`, and the
  canvas is scaled to fit the viewport.
- `<body>` markup: `#gameCanvas`, the HUD (`#hud`, `#hearts`, `#bombs`, weapon/multiplier
  stats), boss bar (`#bossBarWrap`), combo/stage banners, touch controls
  (`#touchControls`), and the start/pause overlay (`#overlay`, `#overlayCard`).
- Script: one IIFE with `'use strict'`. All state is module-local `let`/`const`.

**Key constants (search for these names)**

- `DIFFICULTIES`: `easy` / `normal` / `hard`. Each entry sets spawn timing, enemy speed,
  enemy fire chance, starting `lives`, starting `bombs`, and `bossHpMul`.
- `KILLS_PER_STAGE`: kills needed to advance a stage.
- `THEMES`: background palettes; the stage cycles through them.
- `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), and `tank` (multi-hit, unlocks at
  stage 2). Each has size, hp, speed multiplier, colors, and score value.
- `BOSSES`: one entry per boss (name, colors, HP multiplier).
- `BOSS_PATTERNS`: per-boss arrays of bullet-pattern functions (`fireRadial`,
  `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`,
  `fireHomingSeeded`, `fireDoubleRing`). `updateBoss` drives them.

**Main loop**

- `loop(now)` is a `requestAnimationFrame` loop. It clamps `dt` to 0.05 s, calls
  `trackPerf`, then `update(dt)` and `draw()` only while `running && !paused`.
- `update(dt)` advances the simulation (player, bullets, enemies, power-ups, collisions,
  boss, stage progression). `draw()` renders everything on the canvas.
- `trackPerf` adapts a `quality` level (0 full, 1 medium, 2 low) from average frame time.
  Keep per-frame work cheap, since this is the mechanism that protects low-end devices.

**Input**

- Keyboard: arrows/WASD move, Space fires, X or Shift bombs, P or Esc pauses.
- Mouse: move to steer, click to fire (canvas listeners).
- Touch: left-side joystick drag (`moveZone`), FIRE and BOMB buttons.
- Input state is read from the `keys` object and the `firing` flag inside `update`.

**Persistence**

- `localStorage` keys, all prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`,
  `reducedMotion` (constants `STORAGE_*`). Reads currently have no try/catch, so guard
  any new storage access (private mode and blocked storage can throw).

**Audio**

- Web Audio, created lazily by `ensureAudio()` (required because browsers only allow
  audio after a user gesture). Sound effects are built from `beep` and `noiseBurst`.
  Respect the mute flag.

**Accessibility**

- `announce()` writes to the `#srLive` live region.
- Reduced motion (user toggle, or the `prefers-reduced-motion` media query) disables or
  softens screen shake, flashes, and pulses. New visual effects that shake or flash
  should check `reducedMotion`.

## Conventions

- Keep the game self-contained: no external libraries, CDNs, or build tooling unless the
  user asks for them. Everything stays in `index.html`.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default,
  short descriptive section comments (`// ---------- Input ----------`).
- Prefer data-driven tuning (add an entry to `ENEMY_TYPES`, `BOSSES`, `THEMES`, or
  `DIFFICULTIES`) over new special-case branches.
- The hearts and bombs HUD is rendered with `innerHTML` (`renderHearts`, `renderBombs`)
  from numeric counts only. Do not feed user-controlled or free-text strings into
  `innerHTML`; use `textContent` for any text.
- Preserve `touch-action`, `user-select`, and viewport meta settings. They are needed
  for mobile play.

## Git

- Active development branch for this workspace: `claude/gifted-tesla-f1busi`.
- Commit messages: short imperative subject, optional body explaining the gameplay or
  technical reason for the change.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is one self-contained file, `index.html` (about 1,670 lines): HTML markup, an inline `<style>` block, and one inline `<script>` IIFE that holds all game logic, rendering, audio and input. `README.md` is a placeholder and has no project documentation.

There is no build step, no package manager, no dependencies, no tests and no CI configuration. Do not add any of these unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). Nothing needs to be installed.

## Layout of `index.html`

- **Head**: meta tags, theme colour, and an inline SVG favicon as a data URI.
- **`<style>`**: dark colour scheme, HUD pills (`#hud`), touch controls, overlay card, and keyframe animations (`heartBeat`, and related effects).
- **Body markup**: `#gameCanvas` (the only game surface), `#hud` (score, best, multiplier, weapon, hearts, bombs), `#bossBarWrap`, `#comboPop`, `#stageBanner`, `#topBar` (mute, reduced motion, pause), `#touchControls` (joystick, FIRE, BOMB), and `#overlay` (title, difficulty buttons, Start button, controls hint).
- **Script**: a single IIFE. Its sections, in file order, are:
  1. DOM lookups and `STORAGE_*` keys
  2. Adaptive performance (`trackPerf`, which lowers quality when frames are slow)
  3. Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`
  4. Audio (`ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object)
  5. HUD and effect helpers (hearts, bombs, flashes, combo pops, stage banner, screen shake)
  6. Resize and input (keyboard, mouse, joystick, touch buttons)
  7. Game state: `resetGame`, drawing helpers (`drawPlane`, `drawBossShip`, `drawClouds`), spawning, scoring, bombs, weapons
  8. `update(dt)`, the per-frame simulation; `applyPowerUp`; boss attack patterns (`BOSS_PATTERNS`) and `updateBoss`; `killBoss`; `loseLife`; `endGame`; `togglePause`
  9. `draw()`, the per-frame renderer
  10. `loop(now)`, the `requestAnimationFrame` driver, which clamps `dt` to 50 ms, followed by the start-button handler

## Game design reference

- **Canvas**: logical size `BASE_W × BASE_H = 480 × 720`. `resize()` scales it to fit the window.
- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default) and `hard`. Each sets spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
- **Stages**: reaching `KILLS_PER_STAGE` (18) stage kills triggers a boss warning (`bossWarned`). Killing the boss (`killBoss`) increments `stage`. Each stage picks a theme from `THEMES` (six, cycled). Bosses come from `BOSSES` (six, cycled).
- **Enemies**: `grunt`, `interceptor` (fast, weaving) and `tank` (4 HP, unlocked at stage 2).
- **Weapons**: `vulcan` and `laser`, each with three levels. A weapon power-up raises the level; once maxed, it switches weapon and resets to level 1.
- **Power-ups**: `life`, `bomb` and `weapon`.
- **Scoring**: base points per kill, multiplied by a combo chain that `breakChain()` resets.

## Persistence and accessibility

- `localStorage` keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Always read and write them defensively, because storage can be blocked or throw.
- Reduced motion turns off screen shake, flashes and pulses. The initial value comes from `prefers-reduced-motion` and is saved after the player changes it.
- Screen-reader announcements go through `announce()` and the `#srLive` live region.

## Conventions

- Keep the project a single file unless the user asks for a split. Keep it free of build tools and external dependencies.
- Preserve the IIFE wrapper. Do not add globals.
- Match the existing style: two-space indentation, `const` by default, short section comments using the `// ---------- Name ----------` form, and semantic class and ID names.
- Keep tuning numbers in the existing config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`), not scattered through logic.
- Keep the game working on both desktop (keyboard and mouse) and touch devices. Check any input change against both.
- Keep the layout usable on phones: the canvas fits the viewport, and `touch-action: none` and `user-select: none` stay on.

## Verifying changes

Nothing automated checks the game, so verify by hand:

1. Open `index.html` in a browser and check the console for errors.
2. Start a game on each difficulty, and check that the HUD, score, weapons, bombs, pause and game-over flow all work.
3. For visual or balance changes, check the change in at least one later stage and at a boss fight.
4. If you touch input, check it with the keyboard and mouse and in a touch-emulation mode.

A quick syntax check of the inline script is possible with Node: extract the `<script>` block into a temporary file in the scratchpad directory, then run `node --check` on it.

## Git

- Work on the branch you were given. Do not push to other branches unless the user explicitly asks.
- Write clear, present-tense commit messages that describe the change.

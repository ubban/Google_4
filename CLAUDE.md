# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A single-page browser game, **Sky Fighter** — a Raiden-style vertical shoot-'em-up. The whole game lives in one file:

- `index.html` (~1,670 lines): inline `<style>`, markup, and one inline `<script>` IIFE.
- `README.md`: placeholder only.

There is no build step, no package manager, no test suite, and no external scripts or stylesheets. Open `index.html` in a browser to run it.

## Running and verifying

- Run: open `index.html` directly, or serve the folder with any static server (e.g. `python3 -m http.server`) and browse to it.
- There are no automated tests or linters. Verify changes by playing the game in a browser, and check both keyboard and touch paths if you touch input code.
- Keep the file dependency-free. Don't add a bundler, framework, or CDN script unless the user asks for it.

## Layout of `index.html`

| Area | Where (approx. lines) | Notes |
|---|---|---|
| Styles | 10–359 | Overlay, HUD, touch controls, `.reduced-motion` class |
| Markup | 360–421 | `#gameCanvas`, HUD elements, start/pause overlay, touch controls, difficulty buttons |
| Script setup & DOM refs | 422–462 | All elements are grabbed by ID at the top |
| Constants | 463–528 | Storage keys, `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES` |
| Audio | 545–679 | WebAudio synth (`beep`, `noiseBurst`, `sfx.*`); no audio files |
| HUD & effects | 680–752 | Hearts, bombs, combo pop, stage banner, screen shake, difficulty |
| Canvas sizing | 753–767 | Fixed 480×720 logical resolution, scaled to fit the window |
| Input | 768–870 | Keyboard, virtual joystick + fire/bomb buttons (touch), mouse steering |
| Game state & lifecycle | 871–912 | `resetGame()` is the single place that initializes a run |
| Drawing helpers | 913–1021, 1507–1519 | `drawPlane`, `drawBossShip`, `drawClouds` |
| Spawning & combat | 1022–1147 | Enemies, bosses, power-ups, explosions, scoring, bombs, firing |
| Update loop | 1148–1303 | `update(dt)`: movement, collisions, spawn timer, boss trigger |
| Power-ups & boss patterns | 1304–1411 | `applyPowerUp`, `BOSS_PATTERNS` bullet-pattern functions |
| Boss, death, pause | 1411–1506 | `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause` |
| Render & main loop | 1520–1669 | `draw()`, `loop(now)` via `requestAnimationFrame`, start button |

Everything is inside one IIFE with `'use strict'`. Keep new code inside it; don't leak globals.

## Game rules (current behavior)

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- **Stages**: each stage ends after `KILLS_PER_STAGE` (18) kills, which triggers a boss ~1.2 s later. Defeating the boss advances `stage` and resets `stageKills`. The stage's theme is `THEMES[(stage - 1) % THEMES.length]`.
- **Enemies**: `grunt` (1 HP), `interceptor` (fast, weaving, 1 HP, from stage 1), `tank` (4 HP, from stage 2).
- **Bosses** (`BOSSES`): cycle in order with their own colors and HP multiplier. Each uses patterns from `BOSS_PATTERNS`.
- **Combo chain**: each kill increments `chain`; `multiplier = 1 + min(9, floor(chain / 5))`. The chain resets via `breakChain()` when 2.2 s pass without a kill, or when the player loses a life.
- **Power-ups** (`maybeSpawnPowerUp`): `life` (restores 1 heart, up to max), `bomb` (restores 1 bomb, up to max), `weapon` (upgrades weapon level 1→3, then toggles VULCAN↔LASER).
- **Damage**: `loseLife()` ignores hits while `player.invuln > 0` (1.5 s after a hit). Zero lives ends the run.
- **Persistence** (`localStorage`): best score, mute, difficulty, and reduced-motion preference. Keys are prefixed `skyfighter.`. Keep the prefix when adding keys.

## Conventions

- **Naming**: `camelCase` for functions and variables; `UPPER_SNAKE` for constants and storage keys; sections marked with `// ---------- Name ----------` comments. Match these.
- **Tuning**: gameplay numbers live in the constant tables near the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) or in the `player` object in `resetGame()`. Prefer changing those over hard-coding new magic numbers in logic.
- **Time**: `update(dt)` and `draw()` use seconds. `dt` is clamped to 0.05 s in the loop, so pausing or tab-switching won't teleport things.
- **Accessibility**: announce important state changes through `announce()` (the `#srLive` region). Respect `reducedMotion` by using `screenShake()` and related effects, which already dampen when it's on.
- **Audio**: call `ensureAudio()` from any user-gesture handler before playing sound; browsers block audio until a gesture.
- **Touch vs. keyboard**: `touchControls` is shown only on touch-capable devices. Any new control needs both a keyboard path and, where relevant, a touch path.
- **Canvas**: draw in the fixed 480×720 logical space; CSS scaling handles the rest. Don't read `window.innerWidth` inside game logic.

## Git workflow

- Develop on the branch the session specifies (currently `claude/gifted-tesla-uibz2j`). Don't push to `main` directly.
- Keep commits focused, with a short imperative subject line describing the gameplay or code change.
- Don't open pull requests unless the user asks for one.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 web application. The entire game logic, UI, styling, and audio synthesis lives in `index.html` as a ~1700-line self-contained module.

## Running & Testing the Game

**To play the game:**
- Open `index.html` in a web browser
- Difficulty can be changed from the main menu (Easy, Normal, Hard)
- Desktop controls: Mouse to steer, click to fire, Space to fire, X/Shift to bomb, P/Esc to pause
- Mobile: Drag left to move, tap FIRE, tap BOMB

**No build step required** — this is a standalone HTML file with no dependencies. Changes to `index.html` are live immediately upon refresh.

## Architecture & Structure

The JavaScript is organized as a single **IIFE (Immediately Invoked Function Expression)** starting at line 422, with logical sections marked by comments:

### Core Game Systems

1. **Adaptive Performance** (lines 473-483)
   - Monitors frame time and automatically reduces quality (particle count, shadow blur, glow effects) on slower devices
   - Three quality levels: 0 (full), 1 (medium), 2 (low)

2. **Input Handling** (lines 767-862)
   - Keyboard, mouse, and touch input unified into a single control system
   - Mouse-aiming takes precedence only when keyboard/joystick are inactive (no fighting)
   - Touch devices get virtual joystick + fire/bomb buttons

3. **Game State** (lines 863-869)
   - Central arrays: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
   - Flags: `running`, `paused`, `gameOver`
   - Counters: `score`, `lives`, `chain` (for combo multiplier), `stage`, `stageKills`

4. **Update Loop** (lines 1148-1302)
   - Called each frame; handles movement, collision detection, spawning
   - Collision logic: player/enemies, player/bullets (from enemies), bullets/enemies, bullets/boss, powerups/player
   - Chain timer decay triggers combo multiplier reset

5. **Drawing** (lines 1520-1637)
   - Canvas cleared, background gradient applied, dynamic cloud scrolling
   - Entities rendered: player, enemies, boss, bullets, explosions, powerups
   - Screen shake offset applied in transform

6. **Audio System** (lines 542-676)
   - WebAudio synthesizer (no external audio files)
   - `beep()` for tonal sounds, `noiseBurst()` for explosion/impact sounds
   - Individual SFX for shoot, laser, explosions (grunt/interceptor/tank), bomb, powerup, boss patterns, game-over
   - Mute state persisted in localStorage

### Game Content

**Difficulty Tiers** (line 485): Easy/Normal/Hard settings scale enemy spawn rate, speed, shoot chance, health, and bomb count.

**Enemy Types** (line 502):
- **Grunt**: 1 HP, slow, 10 points
- **Interceptor**: 1 HP, fast, can weave side-to-side, 15 points
- **Tank**: 4 HP, slow, 35 points (appears stage 2+)

**Boss Mechanics** (line 508):
- 6 distinct bosses, each with unique attack patterns and color scheme
- HP scales with stage and difficulty multiplier
- Boss enters from top with brief telegraph before first attack
- Telegraph is visual (center core glows) and audio (bossTelegraph SFX)
- Each boss has 3 attack patterns (accessed cyclically from `BOSS_PATTERNS` at line 1372)

**Weapon System** (line 872):
- **Vulcan** (default): spread shot, 3 bullets at level 3
- **Laser**: pierce-through, silent cooldown advantage, 3 bullets parallel at level 3
- Weapons switch after reaching level 3, reset to level 1
- Firing cooldown: 0.15s (vulcan), 0.1s (laser)

**Power-ups** (line 1058):
- Life (6% spawn chance): +1 health up to max
- Bomb (8% spawn chance): +1 bomb up to max
- Weapon (20% spawn chance): +1 weapon level or switch weapon

**Visual Themes** (line 493): 6 themes cycle with stages (daylight, sunset, night, storm, arctic, volcanic).

## Making Changes

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` (line 502): `{ w, h, hp, speedMul, color, dark, score, minStage, weave? }`
2. Update `spawnEnemy()` logic (line 1022) to include it in the pool
3. Add explosion SFX mapping in `EXPLODE_SFX` (line 678) if needed
4. Collision and death behavior is generic (hp countdown, explosion spawn, score add)

### Adding a Boss Attack Pattern

1. Utility functions for patterns exist: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`
2. Add a new pattern function or compose existing ones
3. Append to the array for the target boss in `BOSS_PATTERNS` (line 1372)
4. Boss cycles through patterns in order; each pattern is a function `(boss) => { enemyBullets.push(...) }`

### Adjusting Game Balance

- **Difficulty multipliers**: Edit `DIFFICULTIES` (line 485)
- **Spawn rates**: Adjust `spawnBase` and `spawnMin`, or the `difficultyRamp` scaling in `update()` (line 1201)
- **Enemy/boss stats**: Edit `ENEMY_TYPES` or `BOSSES` (lines 502, 508)
- **Weapon cooldowns**: Change `fireDelay` in `update()` (line 1177)
- **Enemy bullet speed/count**: Edit the pattern functions in `BOSS_PATTERNS` (line 1372)
- **Multiplier scaling**: `multiplier = 1 + Math.min(9, Math.floor(chain / 5))` at line 1090

### UI & Visual Changes

- HUD styling: CSS from lines 33–359 (stat boxes, heart/bomb icons, boss bar, etc.)
- Overlay screen (pause/game-over): CSS lines 200–357, HTML lines 401–419
- Drawing functions: `drawPlane()` (913), `drawBossShip()` (970), `drawClouds()` (1507)
- Screen shake: Modify `screenShake(mag, dur)` calls (e.g., line 1108)

### Accessibility

- **Reduced motion preference** is detected at startup (line 520) and can be toggled (line 535)
- Animations conditionally disabled (line 349)
- Screen reader announcements via `announce()` (line 468)
- All buttons have titles and active states

## Key Variables & Constants

| Name | Purpose |
|------|---------|
| `frame` | Frame counter, incremented each loop for animation |
| `chain` | Consecutive kill count (decays after ~2.2s no kills) |
| `multiplier` | Score multiplier derived from chain |
| `stageKills` | Kills in current stage (triggers boss spawn at 18) |
| `quality` | Performance level 0–2; affects particle/glow quality |
| `shakeTime`, `shakeMag` | Screen shake state |
| `player` | Object: `{ x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }` |

## localStorage Keys

- `skyfighter.bestScore`: Best score (integer)
- `skyfighter.muted`: Mute state ('0' or '1')
- `skyfighter.difficulty`: Last selected difficulty ('easy', 'normal', 'hard')
- `skyfighter.reducedMotion`: Motion preference ('0' or '1')

## Common Pitfalls

- **Collision detection** uses rectangle overlap with half-widths/heights: `Math.abs(a.x - b.x) < (a.w + b.w) / 2`
- **Radial enemy bullets** use `dx`/`dy` for velocity instead of `vx`/`vy`; ensure you're updating the right field
- **Boss entering state** skips all collision/damage checks; check `!boss.entering` before damage
- **Frame-based animation**: Rotate/bobbing uses `frame * <scalar>`. Ensure enough precision to avoid jumpiness
- **Touch devices**: The `isTouchDevice` check (line 778) enables virtual controls; test both touch and desktop

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**Sky Fighter** is a single-file HTML5 Canvas game — a Raiden-style vertical shoot-'em-up playable in any modern browser. The entire game (code, styling, markup) is contained in `index.html` (~1670 lines). No build process, no dependencies, no external assets.

## Running & Testing

- **Play locally**: Open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge)
- **Live test**: When making changes, reload the browser tab; changes are instant
- **Controls**:
  - Desktop: Mouse (steer) + click (fire), Arrows/WASD (move), Space/click (fire), X/Shift (bomb), P/Esc (pause)
  - Mobile: Drag left zone (move), tap FIRE, tap BOMB

## Architecture

### Single-File Structure

The code is organized in a top-down IIFE (Immediately Invoked Function Expression) with logical sections:

1. **DOM Setup & Constants** (lines 422–527)
   - Query all UI elements (canvas, HUD, overlay, buttons)
   - Define game parameters: `DIFFICULTIES` (easy/normal/hard), `THEMES` (6 visual themes), `ENEMY_TYPES` (grunt/interceptor/tank), `BOSSES` (6 boss definitions)
   - Initialize stored state (best score, mute, difficulty, reduced motion)

2. **WebAudio Synth** (lines 542–676)
   - `ensureAudio()`: Lazy-init Web Audio API context
   - `beep(freq, dur, type, vol, opts)`: Generate oscillator tones with envelopes & filters
   - `noiseBurst(dur, vol, opts)`: White-noise bursts with filters
   - `sfx` object: Predefined SFX (shoot, explode, bomb, powerup, boss, gameover, etc.) — all procedural, no external audio files

3. **UI Rendering** (lines 680–750)
   - `renderHearts()` / `renderBombs()`: Update HUD displays
   - `flashHit()` / `flashBomb()`: Screen flash effects (respects reduced-motion preference)
   - `showComboPop()` / `showStageBanner()`: Overlay animations
   - `screenShake()`: Camera shake (controlled by reduced-motion setting)

4. **Input & Controls** (lines 754–861)
   - `resize()`: Responsive canvas scaling (480×720 base, scales up to 1.6× on large screens)
   - Keyboard: `keys` object tracks key state; P/Esc/X/Shift trigger pause/bomb
   - Mouse: `mouseActive` + `mouseTarget` for analog steering (lower priority than keyboard)
   - Touch: Virtual joystick (move zone) + FIRE/BOMB buttons; joystick centered on first touch

5. **Game State** (lines 864–902)
   - `player`: position, velocity, weapon type/level, cooldown, invulnerability
   - `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`: Arrays for entity management
   - `score`, `lives`, `stage`, `chain` (multiplier counter), `multiplier`: Core metrics
   - `resetGame()`: Reinitialize all state and start a new session

6. **Rendering** (lines 913–1520)
   - `drawPlane(x, y, w, h, color, darkColor, flame)`: Draws player/enemy ships (triangles with flame detail)
   - `drawBossShip(b)`: Multi-part boss visuals (hull, wings, core, health bar)
   - `drawClouds(speed, fillColor, rx, ry)`: Parallax background clouds
   - `draw()`: Main render loop — clears canvas, draws sky gradient, clouds, entities (player, enemies, bullets, particles), HUD, overlays

7. **Game Logic** (lines 1022–1410)
   - **Spawning**: `spawnEnemy()` (picks random type), `spawnBoss()`, `spawnExplosion()` (particle burst), `maybeSpawnPowerUp()` (random weapon/bomb drop)
   - **Collision**: `rectsOverlap()` checks AABB collisions between bullets/ships
   - **Score**: `addScore()` updates score & chain counter; combo multiplies base score (capped at 8×)
   - **Bomb**: `useBomb()` triggers 0.5s invulnerability + screen clear + SFX; limited supply
   - **Weapons**: Player cycles between VULCAN (default rapid-fire) and LASER (charged). Boss has phase-based attack patterns (radial, aimed spread, columns, spirals, homing, dual-ring)
   - **Boss Logic**: `updateBoss()` handles movement, telegraphs, attack phases. `killBoss()` rewards points and advances stage
   - **Health**: `loseLife()` subtracts 1 from `lives`; `endGame()` triggers game-over screen with final score
   - **Power-ups**: `applyPowerUp()` boosts weapon level or grants bomb

8. **Main Loop** (lines 1640–1668)
   - `loop(now)`: Called by `requestAnimationFrame()`
   - Computes delta time
   - Calls `update(dt)` for physics & game state
   - Calls `draw()` for rendering
   - Adaptive quality: `trackPerf()` reduces particle/effect quality if avg frame time > 25ms

### Data Structures

- **Player**: `{ x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }`
- **Enemy**: `{ type, x, y, vx, vy, w, h, hp, shootCooldown }`
- **Bullet**: `{ x, y, vx, vy, w, h, friendly, damage }`
- **Particle**: `{ x, y, vx, vy, life, color }` — short-lived debris
- **PowerUp**: `{ x, y, vx, vy, type ('bomb' | 'weapon'), ttl }`
- **Boss**: `{ x, y, w, h, hp, phase, phaseTimer, dir, patternIndex, attackCooldown, ...appearance }`

## Key Design Decisions

### No Build System
- Plain HTML/CSS/JS; open-and-play simplicity
- All assets (audio, graphics) are procedural or data-URIs
- Single file keeps deployment trivial

### Procedural Audio
- WebAudio API synthesizes all SFX in real-time
- `beep()` creates oscillator tones with frequency sweeps and filters
- `noiseBurst()` creates white-noise effects (bomb, tank explosions)
- No audio file I/O; works offline; audio context suspends if muted

### Responsive Canvas
- Base resolution 480×720 (portrait-friendly)
- Scales up to fit larger screens (max 1.6×)
- CSS `max-width/max-height` ensures it fits in viewport
- Touch controls adapt to screen size

### Difficulty Tiers
- **Easy**: Slower enemies, lower spawn rate, easier boss, 4 lives
- **Normal**: Balanced baseline, 3 lives
- **Hard**: Fast enemies, dense spawns, 30% tougher boss, 3 lives (fewer supplies)
- Settings persist in localStorage

### Performance Tuning
- Adaptive quality tracking (particle reduction on frame-time spike)
- Collision checks use simple AABB (axis-aligned bounding box)
- Reduced-motion preference (CSS `@media (prefers-reduced-motion)`) disables screen shake, flash, and animations

### Visual Themes
Six parallax sky gradients (Daylight, Sunset, Night, Storm, Arctic, Volcanic) auto-rotate per stage.

## Common Dev Tasks

### Tweaking Game Balance
- **Enemy difficulty**: Edit `DIFFICULTIES` object — `spawnBase/spawnMin` (spawn rate), `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`
- **Enemy types**: Adjust `ENEMY_TYPES` — `hp`, `speedMul`, `score`, `minStage` (when to first appear)
- **Boss definition**: Edit `BOSSES` array — `name`, colors (`hullTop/hullBottom/wing/core`), `hpMul`
- **Score multiplier cap**: Search `multiplier = Math.min(8, ...)` and adjust the `8`
- **Weapon stats**: `fireWeapon()` controls fire rate (search `player.cooldown`) and fire patterns (laser vs. vulcan)

### Adding/Tweaking SFX
- Every SFX is in the `sfx` object (line 624+)
- Each function calls `beep()` or `noiseBurst()` with frequency, duration, volume, optional filters (lowpass, highpass)
- To silence a sound: change function body to empty or add early return
- To adjust volume: tweak the `vol` parameter in each `beep`/`noiseBurst` call
- Test by toggling mute button (🔊) during play

### Changing Visuals
- **Ship colors**: `drawPlane()` uses `color` and `darkColor` params; enemies get their colors from `ENEMY_TYPES`
- **Boss appearance**: `drawBossShip()` reads from boss object (`hullTop`, `hullBottom`, `wing`, `core`)
- **Sky themes**: `THEMES` array holds gradient stops and cloud/sun colors; add a new object to the array
- **HUD colors**: Search for `#val`, `#label`, or specific colors like `#ffd54f` (gold) in the CSS section
- **Particle colors**: `spawnExplosion()` at line 1073 takes a `color` param

### Balancing Weapon Fire Patterns
Boss has 8 attack patterns (see `updateBoss()` line 1411+):
1. `fireRadial()`: Bullets spread outward in a circle
2. `fireAimedSpread()`: Spread toward player
3. `fireColumns()`: Vertical columns
4. `fireSpiralBurst()`: Spiraling outward
5. `fireCross()`: Cross pattern
6. `fireHomingSeeded()`: Homing bullets
7. `fireDoubleRing()`: Two concentric rings at different speeds
8. `fireAimedSpread()` (again, different parameters)

To adjust difficulty: change bullet counts, speeds, or pattern sequence in `updateBoss()`.

## UI State Management

- **Overlay**: Game-over/start screen; toggles visibility
- **HUD**: Live updates during play (score, lives, weapon, bombs)
- **Boss bar**: Appears when boss spawns; disappears when killed
- **LocalStorage keys**:
  - `skyfighter.bestScore` — high score
  - `skyfighter.muted` — audio mute toggle
  - `skyfighter.difficulty` — player's last chosen difficulty
  - `skyfighter.reducedMotion` — accessibility setting

## Stage Progression

- Each stage requires 18 kills (`KILLS_PER_STAGE`)
- After 18 kills, a boss spawns
- Boss defeat increments `stage`, resets `stageKills` to 0
- Stages cycle through 6 themes (sky gradients)
- Higher stages spawn harder enemy types (tank appears at stage 2+)

## Accessibility

- Keyboard controls (arrows, WASD, Space, X, P)
- Touch controls for mobile
- ARIA live region (`#srLive`) announces game state changes
- Reduced motion preference respected (disables animations/shake/flash)
- Color-blind friendly palette (reds, oranges, blues, yellows with high contrast)

## Performance Considerations

- Canvas is 480×720 internal; CSS scales it up
- Particles and collision checks run every frame; limit particle count if performance degrades
- WebAudio context auto-suspends when muted to save CPU
- Adaptive quality reduces particles on slower devices
- No physics engine; all collision is axis-aligned bounding box (fast, sufficient for game type)

## Testing Checklist

When making changes:
1. **Gameplay**: Start game, move around, fire, dodge, reach a boss, trigger bomb, pause/resume, gameover → restart
2. **Difficulty**: Test each difficulty mode (Easy, Normal, Hard); verify spawn rate and enemy speed differ
3. **Audio**: Mute toggle works; all SFX play correctly; no audio errors in console
4. **Mobile**: Test on a phone/tablet; joystick and buttons responsive; canvas fits screen
5. **High score**: Finish a game, verify score saves to localStorage; reload page and check best score persists
6. **Accessibility**: Test keyboard-only play; toggle reduced motion and verify animations disable/enable
7. **UI**: All HUD elements update (hearts, bombs, score, weapon level, multiplier)

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML application with no external dependencies. The entire game (HTML, CSS, and JavaScript) is contained in `index.html` (~1,669 lines).

### Key Features
- **Three difficulty levels** (Easy/Normal/Hard) with configurable difficulty parameters
- **Weapon system** with two weapon types (Vulcan/Laser) that upgrade through 5 levels
- **Boss encounters** with 6 unique boss designs and telegraphed attack patterns
- **Score multiplier system** with chaining mechanics
- **WebAudio synthesis** for all sound effects (no audio files required)
- **Adaptive performance scaling** (3 quality levels based on frame time)
- **Full mobile support** with touch joystick and on-screen buttons
- **Accessibility features** including reduced-motion support and screen reader announcements
- **Persistent state** via localStorage (best score, settings, preferences)

## Architecture Overview

### Single-File Organization
All code is organized in a single IIFE (Immediately Invoked Function Expression) with logical sections marked by comments like `// ---------- Section Name ----------`. The main sections are:

1. **DOM References** (lines ~426-461): Canvas, UI elements, HUD components
2. **Storage/Constants** (lines ~463-515): LocalStorage keys, difficulty configs, enemy/boss definitions
3. **Audio System** (lines ~542-676): WebAudio context, synthesized sound effects
4. **UI/Rendering Utilities** (lines ~680-750): HUD updates, animations, UI state management
5. **Input Handling** (lines ~752-862): Keyboard, mouse, and touch input with joystick emulation
6. **Game State** (lines ~864-912): Core game variables (player, entities, score, etc.)
7. **Drawing Functions** (lines ~913-1022): Canvas rendering for planes, bosses, effects
8. **Game Loop** (lines ~1148-onwards): Main update() and render() logic with animation frame

### Core Game Entities

**Player Object**
```javascript
{ x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }
```
- Position and dimensions
- Cooldown timer for weapon fire
- `rapid` counter for continuous fire (space bar)
- Weapon type ('vulcan' or 'laser') with upgrade level (1-5)
- Invulnerability frames after being hit

**Enemy Object**
```javascript
{ type, x, y, vx, vy, w, h, hp, color, dark, shootTimer, score, weapon }
```
- `type`: 'grunt', 'interceptor', or 'tank' (each has different stats in ENEMY_TYPES)
- Position, velocity, dimensions, health
- Weapon system (fires bullets when shootTimer triggers)

**Boss Object**
```javascript
{ idx, x, y, w, h, hp, maxHp, pattern, patternTimer, fireTimer, colors, ... }
```
- Boss index determines appearance and HP multiplier (from BOSSES array)
- Pattern-based movement and attack telegraphing system
- Boss fires in distinct patterns: radial, aimed spread, columns, spiral burst

**Bullet/Projectile Objects**
```javascript
// Player bullets: { type, x, y, vx, vy, w, h }
// Enemy bullets: { x, y, vx, vy, w, h, color }
// Boss bullets: { x, y, vx, vy, w, h, color }
```

**Power-up Objects**
```javascript
{ type, x, y, w, h, vx, vy }
```
- Types determined by random selection or predefined array
- Floats down the screen and disappears if player doesn't collect

### Key Game Systems

**Difficulty System** (`DIFFICULTIES` object)
- Controls spawn rates, enemy speed, shoot frequency, lives, bombs, boss HP multiplier
- Can be changed at menu (if not currently running/paused)
- Stored in localStorage

**Weapon Progression**
- Starts at level 1 (lowest fire rate)
- Levels 2-5 increase rapid-fire rate via `player.rapid` counter
- Weapon pickup powerups cycle between weapon types and upgrade level
- Each weapon has distinct visual and audio feedback

**Scoring & Multiplier**
- Base score: enemy kill value × multiplier
- Multiplier increases per consecutive kill (chain)
- Chain resets if 1+ seconds pass without a kill (chainTimer)
- Score multiplier displayed in HUD

**Boss Pattern System**
- Boss enters at specific kill thresholds per stage (KILLS_PER_STAGE)
- Each boss has 6 different attack patterns (radial, aimed spread, columns, spiral, double columns, aimed+spiral)
- Patterns are telegraphed: warning sound + visual telegraph before firing
- Boss bars show health and pause on defeat (stage transition)

**Theme System** (`THEMES` array)
- 6 visual themes with different sky gradients, cloud colors, sun colors
- Theme cycles per stage (theme = stage % 6)
- Affects background rendering and visual atmosphere

**Particle System**
- Explosions spawn particles that fade and rise
- Particles are simple circles with gravity and alpha fade
- Used for enemy deaths, bomb effects, visual feedback

**Screen Shake & Visual Effects**
- `screenShake(magnitude, duration)` applies camera offset per frame
- `flashHit()` and `flashBomb()` trigger CSS animations
- Vignette effect pulses when player is at 1 HP
- All effects respect reducedMotion preference

## Common Development Tasks

### Running the Game
Simply open `index.html` in a web browser. No build step or server required. The game starts at the title screen with difficulty selection.

### Modifying Difficulty Parameters
Edit the `DIFFICULTIES` object (line ~485):
```javascript
easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, ... },
normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, ... },
hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, ... }
```
- `spawnBase`: Initial spawn timer value
- `spawnMin`: Minimum spawn timer (hard cap on spawn rate)
- `enemySpeed`: Multiplier on all enemy velocities
- `enemyShootChance`: Probability enemy fires each frame
- `lives`, `bombs`: Starting player resources
- `bossHpMul`: Multiplier on boss max HP

### Adding New Enemy Types
1. Add entry to `ENEMY_TYPES` object (line ~502) with stats
2. Update `KILLS_PER_STAGE` threshold if needed
3. Add spawn conditions in `spawnEnemy()` (line ~1022) if stage-gated
4. Create or reuse an explosion effect in `EXPLODE_SFX` (line ~678)

### Adding New Boss Patterns
Bosses use predefined attack pattern functions. To add a pattern:
1. Create a new `fireRadial/AimedSpread/Columns/SpiralBurst` variant (lines ~1324-1351)
2. Add to boss's pattern array in `spawnBoss()` (line ~1043)
3. Pattern function receives boss object and should call `enemyBullets.push(...)`

### Adding Sound Effects
SFX are synthesized via WebAudio. Add to `sfx` object (line ~624):
```javascript
effectName: () => {
  beep(freq, duration, 'square'|'sine'|'sawtooth'|'triangle', volume, opts);
  // opts: { sweepTo, filterType, filterFreq, pan, delay }
  // or use noiseBurst(duration, volume, opts) for noise-based effects
}
```
Call with `sfx.effectName()` wherever needed. Automatically muted if `muted` flag is true.

### Adjusting Visual Themes
Edit `THEMES` array (line ~493). Each theme has:
- `sky`: RGB gradient array [top, middle, bottom]
- `cloud`, `sun`: RGB values (as comma-separated string for canvas fillStyle)
- `name`: Display name

### Adding Power-ups
Power-up types are predefined. To add new type:
1. Add handling in `applyPowerUp(type)` (line ~1304)
2. Update `maybeSpawnPowerUp()` (line ~1058) to include new type in random selection or specific conditions
3. Add corresponding visual rendering if not already generic

### Adjusting Input Sensitivity
- **Keyboard/Joystick**: `player.speed` (line ~877) controls movement speed
- **Mouse steering**: `mouseTarget` tracking is in `canvas.addEventListener('mousemove')` (line ~847)
- **Touch joystick**: Max distance capped at 50px in `joystickMove()` (line ~799)

## Code Patterns & Conventions

### Entity Updates
Entities are stored in arrays (bullets, enemies, particles, etc.) and updated each frame in `update(dt)`:
1. Iterate array backwards when removing (to avoid index shift issues)
2. Check collision via `rectsOverlap(a, b)` (line ~1069)
3. Spawn particles/effects immediately on entity death
4. Remove dead entities after collision/health check

### Screen Coordinates
- Canvas is 480×720 (BASE_W, BASE_H)
- Canvas scales with CSS but game logic uses native 480×720 coordinates
- Y increases downward; enemies move down (positive vx), player at bottom

### Performance Optimization
- Adaptive quality system (trackPerf) reduces particle/effect density on slow frames
- Frame time tracking via `frameTimes` array (windowed average of 45 frames)
- Quality levels: 0 (full), 1 (medium), 2 (low)
- Quality affects particle spawn count in explosions

### Audio Timing
- WebAudio uses `audioCtx.currentTime` for precise scheduling
- All beeps/noise bursts use envelope (attack, sustain, release)
- Gain curves use `linearRampToValueAtTime()` or `exponentialRampToValueAtTime()`
- Pan values calculate from bullet position for spatial audio

## Game Balance Notes

### Progression
- Enemies scale in difficulty: grunt → interceptor → tank → bosses
- Tanks introduced at stage 2; tank count increases per stage
- Boss HP scales with stage and difficulty (bossHpMul)
- Weapon upgrades provide meaningful progression (level 5 = much faster fire)

### Difficulty Tuning
- **Easy**: 4 lives, 3 bombs, slower enemies, lower shoot chance
- **Normal**: 3 lives, 2 bombs, baseline difficulty
- **Hard**: 3 lives, 2 bombs, fast enemies, high shoot chance, tanky bosses

### Score Scaling
- Base points per enemy: 10 (grunt), 15 (interceptor), 35 (tank)
- Multiplier scales per-frame (chain system encourages rapid kill streaks)
- Boss kills do not grant direct points (stage completion reward comes from stage clear bonus)

## Storage Keys
All persistent data uses localStorage with `skyfighter.` prefix:
- `skyfighter.bestScore`: Best score (number)
- `skyfighter.muted`: Audio mute state ('0' or '1')
- `skyfighter.difficulty`: Selected difficulty ('easy', 'normal', 'hard')
- `skyfighter.reducedMotion`: Reduced motion toggle ('0' or '1')

## Browser Compatibility
- Requires Canvas 2D context (all modern browsers)
- WebAudio API (all modern browsers; gracefully disables audio if unavailable)
- Touch events for mobile; fallback to keyboard/mouse for desktop
- LocalStorage for persistence (optional; game playable without it)
- Works on mobile, tablet, and desktop with responsive scaling

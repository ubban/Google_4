# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up browser game. It's a single-file HTML application with inline JavaScript and CSS, featuring no external dependencies. The game is fully self-contained in `index.html` with embedded:
- Canvas-based 2D graphics rendering
- WebAudio API synthesized sound effects (procedurally generated, no audio files)
- Complete game logic in vanilla JavaScript
- Responsive CSS with mobile touch controls

## Architecture

The game code is structured as a single IIFE (Immediately Invoked Function Expression) containing:

### Core Systems

**Input Handling** (lines 768-861)
- Keyboard input (WASD, arrows, space, shift/X for bomb, P/Esc for pause)
- Mouse steering with visual cursor indicator
- Touch controls with on-screen joystick (left side) and action buttons (right side)
- Touch and mouse activate audio context automatically

**Game State** (lines 864-911)
- Player object: position, speed, weapon level, invulnerability timer
- Arrays for bullets, enemies, enemy bullets, particles, power-ups, boss
- Score, lives, multiplier/chain system
- Difficulty settings and stage progression
- localStorage persistence for: best score, mute state, difficulty, reduced motion preference

**Audio System** (lines 542-676)
- WebAudio API synthesis—no external audio files
- Procedurally generated sound effects for all game events
- Master gain with compression for dynamic range
- Positional audio (panning based on screen position)
- Sound effects: shoot, laser, explosions (grunt/interceptor/tank), hit, bomb, powerup, boss telegraph, boss defeat, stage clear, UI click, game over
- Muted state toggleable via UI or localStorage

**Rendering Pipeline** (lines 1520-1638)
- Canvas clear and draw (with optional screen shake translation)
- Sky gradient background with theme (6 themes based on stage)
- Cloud parallax layers
- Player plane with weapon-specific visual (laser has glow circle)
- Bullets with glow effects
- Enemy planes and tanks (with health bars for tanks)
- Boss ship with animated core
- Power-ups (rotating icons)
- Particles (explosions fade out)
- All rendering respects `quality` setting for performance

**Update Loop** (lines 1148-1302, 1411-1439)
- Input-driven player movement (keyboard, joystick, or mouse steering)
- Weapon firing with cooldown (spread patterns for Vulcan, pierce for Laser)
- Enemy spawning with dynamic difficulty ramp
- Enemy behavior (weaving, targeted/spread shooting)
- Collision detection: player-enemy, bullet-enemy, bullet-boss, enemy bullet-player
- Chain/combo multiplier logic (resets on gap > 2.2 seconds)
- Boss spawn at KILLS_PER_STAGE (18)
- Boss attack patterns with telegraph phases
- Power-up application
- Particle physics

### Difficulty System

Three presets (lines 485-489) control:
- Spawn rate (base and minimum delay)
- Enemy speed and shoot chance
- Starting lives and bombs
- Boss HP multiplier

### Weapon System

**Vulcan** (default):
- Straight line fire with spread at higher levels (1: center, 2: dual, 3: triple)
- 0.15s cooldown
- Can be picked up to increment level (1→2→3) or swap to Laser

**Laser**:
- Focused beam fire, pierce enemies (1→2→3 pierces)
- 0.1s cooldown (faster)
- Can be picked up to increment level or swap back to Vulcan
- Visual: glow circle around player

### Boss Patterns

Six boss designs, one per stage cycling (lines 508-515). Each has unique attack pattern array (lines 1372-1409):
- **VANGUARD**: Radial burst, aimed fan, V-fan double streams
- **BEHEMOTH**: Heavy columns, slow homing spread, dense radial wall
- **SPECTER**: Teleport + radial, tight spiral, double aimed shots
- **DREADNOUGHT**: 8-way cross, homing volley, radial wall
- **WRAITH**: Teleport barrage, double ring, aimed cross-spread
- **TITAN**: Columns, full ring, homing volley

Boss telegraph (22 frames) before firing, giving player reaction time.

### Performance Adaptation

Quality levels (lines 473-483):
- `quality = 0`: Full detail (shadow blur for glows, full particle count)
- `quality = 1`: Medium (shadow blur reduced, 60% particles)
- `quality = 2`: Low (no shadow blur, 35% particles)

Adaptive based on frame time over rolling 45-frame window. Automatically downgrades under load, upgrades when CPU headroom returns.

### Accessibility

- Screen reader support (live region updates for game events, announcements)
- Reduced motion preference detection (prefers-reduced-motion media query)
- Reduced motion can be toggled in-game (affects animations, screen shake, flash)
- Color contrasts for UI and HUD
- Controls hint displayed on start screen

### Visual Themes

Six stage-based sky themes (lines 493-500) with:
- Three-color sky gradient (top, mid, bottom)
- Cloud color and sun color (for radial glow)
- Theme-specific names

Theme cycles every 6 stages.

## Common Development Tasks

### Testing the Game
Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge). No build step required. LocalStorage is used to persist best score and settings.

### Tweaking Game Balance

**Spawn Rate**: Lines 485-489, `DIFFICULTIES` object
- Decrease `spawnBase` and `spawnMin` for more enemies
- Increase `enemySpeed` for faster enemies
- Adjust `enemyShootChance` to make enemies more aggressive

**Weapon Balance**: 
- Vulcan cooldown: line 1177 (`fireDelay = 0.15`)
- Laser cooldown: line 1177 (`fireDelay = 0.1`)
- Spread patterns: lines 1127-1133 (Vulcan)
- Pierce damage: lines 1259, 1141 (Laser)

**Boss HP**:
- Line 1046 `baseHp = 70 + stage * 35`
- Multiply by `d.bossHpMul` (difficulty) and `design.hpMul` (boss-specific)

**Player Damage**:
- Vulcan does 1 HP per hit (line 1239 implicit)
- Laser does 3 HP per hit (line 1259)

### Adding New Enemy Types

1. Define in `ENEMY_TYPES` (lines 502-506):
   ```javascript
   newType: { w: 40, h: 40, hp: 2, speedMul: 1.2, color: '#...', dark: '#...', score: 25, minStage: 3, weave: false }
   ```
2. Add to `spawnEnemy()` selection logic (lines 1028-1030)
3. Add explosion sound mapping (line 678) if custom explosion needed
4. Draw logic will auto-call `drawPlane()` (line 1604)

### Adding New Boss Attack Patterns

1. Create function in `BOSS_PATTERNS` section (starting line 1372)
   - Use helper functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`
2. Add pattern array to `BOSS_PATTERNS` at appropriate index
3. Boss selection is automatic based on `BOSSES` array index (line 1045)

### Adding a New Theme

Add to `THEMES` array (lines 493-500):
```javascript
{ 
  sky: ['#topColor', '#midColor', '#bottomColor'], 
  cloud: 'r,g,b', 
  sun: 'r,g,b', 
  name: 'Theme Name' 
}
```
Themes are indexed by `(stage - 1) % THEMES.length`.

### Adding New Sound Effects

Add to `sfx` object (lines 624-676):
- Use `beep()` for tonal sounds (frequency, duration, type, volume, options)
- Use `noiseBurst()` for noise-based sounds (duration, volume, options)
- Each plays through the synthesizer when not muted
- Call with `sfx.effectName()`

### Modifying UI Elements

HUD elements are HTML with CSS classes updated by JavaScript:
- Score, best, multiplier, weapon: text content updates in `update()` and `resetGame()`
- Hearts/bombs: regenerated as HTML in `renderHearts()` and `renderBombs()`
- Boss health bar: width updated in line 1300
- Overlay (start screen, pause, game over): shown/hidden and content updated in event listeners and `endGame()`

## Key Functions Reference

**Utility:**
- `rectsOverlap(a, b)`: AABB collision detection (line 1069)
- `spawnExplosion(x, y, color, count)`: Create particles (line 1073)
- `addScore(base)`: Increment score with multiplier, update chain (line 1087)
- `breakChain()`: Reset combo multiplier (line 1096)

**Player Actions:**
- `fireWeapon()`: Add bullets based on current weapon/level (line 1124)
- `useBomb()`: Clear enemy bullets, damage enemies/boss, deduct bomb (line 1103)
- `applyPowerUp(type)`: Handle life/bomb/weapon pickups (line 1304)

**Game Flow:**
- `resetGame()`: Initialize all state for new game (line 875)
- `endGame()`: Show game over screen, update best score (line 1469)
- `togglePause()`: Pause/resume with overlay (line 1491)
- `spawnBoss()`: Create boss at stage (line 1043)
- `killBoss()`: Advance stage, restart spawning (line 1441)
- `loseLife()`: Decrement lives, check game over (line 1456)

**Rendering:**
- `drawPlane(x, y, w, h, color, darkColor, flame)`: Draw player/enemy plane (line 913)
- `drawBossShip(b)`: Draw boss with animated core (line 970)
- `draw()`: Main render function (line 1520)

**Animation:**
- `showComboPop(text)`: Animate combo popup (line 712)
- `showStageBanner(text)`: Animate stage banner (line 719)
- `flashHit()`: Screen flash on damage (line 698)
- `flashBomb()`: Screen flash on bomb (line 704)
- `screenShake(mag, dur)`: Apply shake (line 727)

**Input:**
- `joystickStart/Move/End()`: Touch control handling (lines 785-812)
- `canvasPointFromEvent(e)`: Convert screen coords to canvas coords (line 840)

## LocalStorage Keys

- `skyfighter.bestScore`: Best score achieved
- `skyfighter.muted`: Audio mute state (1 or 0)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Reduced motion toggle (1 or 0)

## Performance Considerations

- Adaptive quality prevents frame drops on lower-end devices
- Canvas drawing uses direct context calls (no external rendering libraries)
- Collision detection is simple AABB, O(n²) for small enemy counts
- Particles capped indirectly by quality setting
- Removed particles immediately on death
- Enemy/bullet cleanup happens each frame (dead entities filtered out)
- Audio context created on first user input (lazy initialization)
- Sound effects are synthesized in real-time, no latency from loading

## Browser Compatibility

- Requires: Canvas 2D, Web Audio API, localStorage, ES6 (arrow functions, const/let, template literals)
- Touch support optional (gracefully degrades to keyboard/mouse)
- Tested on: Chrome, Firefox, Safari, Edge (modern versions)

## Responsiveness

- Canvas maintains 480×720 base resolution, scaled to fit window (max 1.6x)
- UI elements use `clamp()` for responsive font sizes
- Safe area insets respected for notched devices (mobile)
- Orientation change handled automatically

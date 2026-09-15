# Sky Fighter - Codebase Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML/Canvas application. Players control a fighter plane, manage weapon power-ups, deploy smart bombs, and battle stage bosses across multiple difficulty levels.

**Tech Stack:**
- HTML5 Canvas for rendering
- Vanilla JavaScript (ES6+)
- Web Audio API for sound effects and music
- localStorage for persistent game state
- Touch support for mobile devices

## File Structure

- **index.html** (1669 lines) - Single monolithic file containing all HTML, CSS, and JavaScript

## Game Architecture

### Core Game Loop
The main game loop (`update()` and `draw()` functions) runs continuously:
1. **Update Phase** - Calculate physics, handle input, manage enemies/projectiles
2. **Draw Phase** - Render all game elements on canvas

### Game State Management
- **Difficulty System** - Three presets: easy, normal, hard (controls spawn rates, enemy AI, lives, bombs)
- **Player State** - Position, health, ammo count, current weapon, bomb count, score, multiplier
- **Enemy Management** - Array-based spawning, movement, shooting, destruction
- **Boss Battles** - Stage bosses with custom attack patterns and health tracking
- **Performance Tracking** - Adaptive quality system monitors frame times and adjusts rendering

### Key Constants & Thresholds
- `KILLS_PER_STAGE = 18` - Enemies to defeat before boss appears
- `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` - localStorage keys for game state

## Major Functional Components

### Game Control & Setup
- `resetGame()` - Initialize game state for new game (line 875)
- `setDifficulty(name)` - Apply difficulty preset and update UI (line 741)
- `togglePause()` - Pause/resume game (line 1491)
- `endGame()` - Handle game over sequence (line 1469)

### Player Aircraft
- `drawPlane(x, y, w, h, color, darkColor, flame)` - Render player ship with optional flame (line 913)
- `fireWeapon()` - Fire currently-equipped weapon (line 1124)
- `useBomb()` - Deploy smart bomb (line 1103)
- `loseLife()` - Handle player damage (line 1456)

### Enemy & Boss Management
- `spawnEnemy()` - Create enemy with random position and behavior (line 1022)
- `spawnBoss()` - Instantiate stage boss (line 1043)
- `drawBossShip(b)` - Render boss sprite (line 970)
- `updateBoss(dt)` - Update boss AI, movement, attacks (line 1411)
- `killBoss()` - Handle boss defeat and stage transition (line 1441)

### Boss Firing Patterns
Custom attack patterns for variety and challenge:
- `fireRadial(b, count, speed, offset)` - Circular spread pattern (line 1324)
- `fireAimedSpread(b, count, speed, spreadStep)` - Aimed shots at player (line 1330)
- `fireColumns(b, speed)` - Vertical bullet columns (line 1342)
- `fireSpiralBurst(b, speed)` - Rotating spiral attack (line 1347)
- `fireCross(b, speed)` - Cross/plus pattern (line 1354)
- `fireHomingSeeded(b, count, speed)` - Homing projectiles (line 1359)
- `fireDoubleRing(b, speedA, speedB)` - Concentric ring attack (line 1367)

### Power-Up System
- `maybeSpawnPowerUp(x, y)` - Random drop on enemy kill (line 1058)
- `applyPowerUp(type)` - Apply weapon or bomb upgrade (line 1304)

### Score & Multiplier
- `addScore(base)` - Add score with multiplier, handle chain logic (line 1087)
- `breakChain()` - Reset combo multiplier (line 1096)
- Multiplier increases on consecutive kills, resets on pause/miss

### Particle & Visual Effects
- `spawnExplosion(x, y, color, count)` - Create explosion particles (line 1073)
- `flashHit()` - White flash on player damage (line 698)
- `flashBomb()` - Flash effect for bomb detonation (line 704)
- `showComboPop(text)` - Display combo/multiplier popup (line 712)
- `showStageBanner(text)` - Show stage name on transition (line 719)
- `screenShake(mag, dur)` - Camera shake effect (line 727)

### Audio System
- `ensureAudio()` - Initialize Web Audio API (line 545)
- `beep(freq, dur, type, vol, opts)` - Generate tone for UI/effects (line 566)
- `noiseBurst(dur, vol, opts)` - Generate noise burst (line 597)
- `panFor(x)` - Stereo panning helper (line 562)
- Mute state persisted to localStorage

### Rendering & Animation
- `drawClouds(speed, fillColor, rx, ry)` - Render parallax clouds (line 1507)
- `draw()` - Main render function (line 1520)
- Theme system with 6 different sky/environment color schemes

### Input Handling
**Keyboard:**
- Arrow keys / WASD - Move player
- Space - Fire
- X or Shift - Bomb
- P or Esc - Pause

**Mouse:**
- Move to position player
- Click to fire

**Mobile/Touch:**
- Drag on left zone for movement
- FIRE button for weapon
- BOMB button for smart bomb
- Joystick-style input with visual feedback

- `canvasPointFromEvent(e)` - Convert event coords to canvas space (line 840)
- `joystickStart/Move/End()` - Mobile joystick control (lines 785-807)

### UI & Display
- `renderHearts(count, max)` - Display health indicator (line 680)
- `renderBombs(count, max)` - Display bomb counter (line 690)
- `weaponLabel()` - Get display name for current weapon (line 871)
- `updateMotionBtn()` - Reduced motion toggle (line 529)
- `updateMuteBtn()` - Mute/audio toggle (line 732)
- Accessibility: screen reader live region for announcements

### Collision & Geometry
- `rectsOverlap(a, b)` - AABB collision test (line 1069)

## Weapon System

Six weapon types available as power-ups:
1. **Spread** - Wide fan of projectiles
2. **Heavy** - Large, slow, powerful shots
3. **Laser** - Continuous beam effect
4. **Homing** - Self-guided missiles
5. **Railgun** - High-velocity single shot
6. **Burst** - Rapid-fire machine gun

Each weapon has a level (0-2), tracking damage and behavior.

## Difficulty Settings

```javascript
DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
}
```

- **spawnBase/Min** - Enemy spawn timing thresholds
- **enemySpeed** - Multiplier for enemy velocity
- **enemyShootChance** - Probability of enemies firing per frame
- **lives** - Starting health
- **bombs** - Initial bomb count
- **bossHpMul** - Boss health multiplier

## Performance Optimization

- **Adaptive Quality** - Monitors frame time over 45 frames, adjusts rendering:
  - Quality 0: Full rendering
  - Quality 1: Medium (fewer particles, simplified effects)
  - Quality 2: Low (minimal particles, basic rendering)
- **Framerate Target** - Targets 60 FPS, degrades gracefully on slower devices
- **Canvas Sizing** - Responsive canvas that scales to window dimensions

## Browser Storage

- `skyfighter.bestScore` - Personal high score
- `skyfighter.muted` - Audio mute state
- `skyfighter.difficulty` - Last selected difficulty
- `skyfighter.reducedMotion` - Accessibility preference

## Common Tasks

### Adding a New Enemy Type
1. Modify `spawnEnemy()` to include new sprite/behavior
2. Update collision and damage logic in `update()`
3. Add to `maybeSpawnPowerUp()` drop chances

### Creating a New Boss
1. Add boss object properties (position, hp, phase, attackTimer)
2. Implement attack pattern function (e.g., `fireNewPattern()`)
3. Add boss case to `updateBoss()` for phase transitions
4. Update `drawBossShip()` if using custom sprite

### Balancing Difficulty
- Adjust `DIFFICULTIES` presets
- Modify `KILLS_PER_STAGE` for boss timing
- Tweak weapon damage in `fireWeapon()` and power-up effects
- Adjust boss health with `bossHpMul`

### Adding Visual Effects
- Use `showComboPop()` for text overlays
- Use `spawnExplosion()` for particle effects
- Implement new keyframe animations in CSS (search `@keyframes`)
- Use `screenShake()` for impact feedback

## Code Organization Notes

- Single global IIFE closure to avoid namespace pollution
- Extensive use of canvas rendering for custom graphics
- Web Audio API for procedural sound (no audio files)
- Touch/mouse input normalized through shared functions
- Responsive design with viewport-relative sizing and safe area insets
- CSS Grid/Flexbox for UI overlays

## Development Workflow

1. **Local Testing** - Open `index.html` in browser
2. **Cross-Browser** - Test on Chrome, Firefox, Safari
3. **Mobile Testing** - Use browser DevTools device emulation or physical device
4. **Performance** - Monitor frame rate in DevTools, test on low-end devices
5. **Accessibility** - Test keyboard navigation, screen reader announcements

## Known Limitations & Future Improvements

- Single-file architecture limits code organization (could split into modules)
- No persistent high score cloud sync (localStorage only)
- Sound generation procedural (no spatial audio)
- Boss attack patterns could expand further
- Visual polish opportunities (particle count, vfx variations)

---

*This guide documents Sky Fighter as of September 2026. Refer to function line numbers for implementation details.*

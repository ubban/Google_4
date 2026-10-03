# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game written in vanilla JavaScript (ES6+). The entire game is self-contained in a single `index.html` file (~58KB) combining:
- HTML structure for game canvas and UI overlays
- CSS for styling, animations (combo popups, stage banners, vignette effects), and responsive layout
- JavaScript game engine with physics, collision detection, sprite rendering, and WebAudio synthesis

The game runs entirely in the browser with no build process, external dependencies, or server requirements.

## Running the Game

**Development/Testing:**
```bash
# Open the game in a browser
open index.html
# Or serve via HTTP (e.g., with Python or your preferred server)
python3 -m http.server 8000
```

The game works on desktop (keyboard/mouse) and mobile (touch controls with virtual joystick). It automatically adapts rendering quality if frame rates drop below 40 fps (see "Adaptive Performance" below).

## Code Architecture

### Single IIFE Pattern
The game code is organized as an Immediately Invoked Function Expression (IIFE) wrapping all logic to avoid polluting the global scope. Key sections:

```javascript
(function() {
  // DOM element references
  // Constants (DIFFICULTIES, ENEMY_TYPES, BOSSES, THEMES)
  // WebAudio synthesis system
  // Input handling (keyboard, mouse, gamepad, touch)
  // Core game loop and rendering
})();
```

### Main Game State
Key state variables initialized at line ~865:
- `player`: player object with x, y, hp, currentWeapon, weaponLevel
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`: entity arrays
- `boss`: current boss object (null if no boss active)
- `score`, `lives`, `maxLives`, `running`, `paused`, `gameOver`: game flags

### Core Systems

#### 1. **Difficulty & Stage Progression**
- **DIFFICULTIES** (line 485): `easy`, `normal`, `hard` with spawn rates, enemy speed, lives, bomb count, boss HP multiplier
- **KILLS_PER_STAGE** (line 491): 18 enemy kills progress to next stage
- Each stage increments enemy spawning intensity and introduces new enemy types at specific stages (tank appears at stage 2+)

#### 2. **Enemy Types & Bosses**
- **ENEMY_TYPES** (line 502): `grunt`, `interceptor`, `tank` — each with width, height, HP, speed multiplier, color, score, and stage availability
- **BOSSES** (line 508): 6 unique bosses with distinct visual designs (color schemes) and HP multipliers
- Boss selection is deterministic; player fights one boss per 6 stages

#### 3. **Weapon System**
- Players start with basic "Vulcan" weapon and can upgrade through 5 levels via pickups
- Weapon levels affect fire rate and bullet count
- Players can switch between primary and alternate weapons on the fly

#### 4. **Collision & Damage**
- Simple axis-aligned bounding box (AABB) collision checking
- Player collision with enemy bullets or enemies causes 1 HP loss
- Bombs clear all on-screen enemy bullets and damage nearby enemies
- Vignette effect (red pulsing edge) signals low health (~1 HP)

#### 5. **Visual Themes**
- **THEMES** (line 493): 6 interchangeable sky/cloud/sun color schemes changing every 1–2 stages
- Each theme includes gradient sky backgrounds and cloud/sun rendering parameters

#### 6. **Particle & Effect System**
- Particles pool drawn at line ~1500+ for explosions, hits, and visual feedback
- Flash overlay (white) for hit feedback; red vignette for low health
- Stage banner and combo popup animations (CSS keyframes + JS triggering)

#### 7. **Adaptive Performance**
- Tracks frame times over 45-frame window (line 476)
- Adjusts `quality` level (0 = full, 1 = medium, 2 = low) based on average frame time
- High-quality mode skips certain particle effects or reduces draw detail

#### 8. **WebAudio Synthesis**
- No external audio files; all SFX synthesized in real-time (line 542+)
- Fires, explosions, pickups, and UI sounds generated using oscillators and envelope shaping
- Master gain with compressor for audio normalization

#### 9. **Input Handling**
- **Desktop**: WASD/arrow keys for movement, mouse for aiming, spacebar for shoot, F for fire bomb
- **Gamepad**: Dual-stick support (left stick move, right stick aim) or full analog
- **Touch**: Virtual joystick in bottom-left, fire and bomb buttons in bottom-right
- Supports DeviceMotion for tilt-based aiming on mobile

#### 10. **Accessibility**
- Screen reader live region (`srLive`) for announcements
- Reduced motion toggle respects `prefers-reduced-motion` media query
- High-contrast UI elements with text shadows for readability

### Key Functions

**Game Loop:**
- `update(dt)`: Updates player, enemies, bullets, particles; checks collisions; advances wave/boss logic
- `render()`: Draws sky, clouds, player, enemies, bullets, particles, HUD
- `gameLoop()`: requestAnimationFrame wrapper calling `update()` and `render()`

**Rendering:**
- `drawPlayer()`, `drawEnemy()`, `drawBullet()`: Entity sprite drawing
- `drawSky()`: Gradient + cloud rendering with theme colors
- `drawParticles()`: Batch particle rendering

**Gameplay:**
- `spawnEnemy()`: Creates random enemy type based on stage and spawn rules
- `fireBullet()`: Adds bullet to array with velocity
- `dropBomb()`: Clears enemy bullets, applies AOE damage
- `addScore()`: Updates score and checks for best score

**UI:**
- `startGame()`: Initializes new game state
- `endGame()`: Shows game-over screen with final stats
- `updateLives()`, `updateScore()`: Updates HUD display
- `setDifficulty()`: Changes game difficulty and stores preference

## Development Notes

### Adding New Features

1. **New Enemy Type**: Add entry to `ENEMY_TYPES` with sprite dimensions, HP, speed, colors, and score. Update `drawEnemy()` if custom sprite needed.

2. **New Boss**: Add entry to `BOSSES` array. Boss spawn logic is at line ~1100 (stage check). Customize `drawBoss()` if using unique shape.

3. **New Weapon Level**: Modify weapon fire rate or bullet count in firing logic (~line 1050). Add visual feedback if needed.

4. **New Theme**: Add entry to `THEMES` with sky gradient colors and cloud/sun RGB values.

### Testing Checklist

- **Difficulty progression**: Verify spawn rates increase; enemies get faster each stage
- **Boss fight**: Confirm boss spawns at stage 6, 12, etc.; check HP scales correctly
- **Weapons**: Test all 5 weapon levels; verify fire rate and pattern changes
- **Collision**: Confirm player takes damage only when hit; bombs clear bullets
- **Touch controls**: Test joystick responsiveness and button sensitivity on various devices
- **Accessibility**: Verify screen reader announces score changes and game state; test reduced-motion mode
- **Performance**: Monitor FPS; confirm quality downgrade under load; test on low-end devices

### Debugging Tips

- **Frame rate drops**: Check `quality` variable; particle count and draw calls scale with this
- **Collision issues**: Verify AABB logic in `checkCollisions()` matches entity dimensions
- **Audio not playing**: Confirm `ensureAudio()` successfully creates AudioContext; WebAudio may require user interaction first
- **Touch not responsive**: Check `joyVec` and `fireBtn`/`bombBtn` state; ensure `touchControls` visibility is correct
- **Theme mismatch**: Verify `THEMES` index matches current stage calculation

### Performance Optimization Opportunities

1. Spatial partitioning for collision detection (currently O(n²) enemy-bullet pairs)
2. Canvas off-screen rendering for static elements
3. WebGL for high-end devices instead of 2D canvas
4. Sprite atlasing if adding more visual variety
5. Audio pooling for frequent effects

## Local Storage

Game persists the following to localStorage:
- `skyfighter.bestScore`: Best score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Selected difficulty
- `skyfighter.reducedMotion`: Accessibility preference

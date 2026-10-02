# Sky Fighter Codebase Guide

## Overview
Sky Fighter is a Raiden-style vertical shoot-'em-up game built as a single-page HTML5 application. Players control an aircraft, defeat waves of enemies, face stage bosses, manage resources (lives, bombs), and achieve high scores.

## Project Structure
- `index.html` - Complete game implementation (~1669 lines) with embedded HTML, CSS, and JavaScript
- `README.md` - Project title and timestamp

## Architecture

### Canvas & Rendering
- Uses HTML5 Canvas API for 2D graphics
- Game loop driven by `requestAnimationFrame` with delta-time calculations
- Responsive layout that maintains aspect ratio and handles device rotation
- Safe area inset support for notched devices (iOS)

### Game State Management
Key game variables:
- `gameState` - Tracks current mode: "menu", "playing", "paused", "boss", "gameover", "levelup"
- `player` - Aircraft position, velocity, health, weapon state
- `difficulty` - Affects enemy spawn rates and behavior
- `stage` - Current level (0-based indexing)
- `lives` - Player health count
- `bombs` - Smart bomb count
- `score` - Points earned (scaled by difficulty)

### Core Game Systems

#### Player System
- **Controls**: Keyboard (WASD/Arrows + Space), Mouse (click/drag), Touch (joystick)
- **Movement**: Smooth acceleration/deceleration with max velocity
- **Weapons**: 
  - Cannon (rapid-fire bullets)
  - Spread (fan of bullets)
  - Laser (piercing beam)
  - Rockets (explosive, homing capability)
  - Cycles with weapon key
- **Health**: Visual feedback via rendered hearts
- **Bombs**: Smart bombs with screen-wide damage, recharge over kills

#### Enemy System
- **Spawning**: Wave-based with difficulty scaling
- **Types**: Standard fighters and boss ships
- **Behavior**: 
  - Basic: Linear movement with periodic shots
  - Boss: Multi-phase fights with distinct attack patterns
- **Projectiles**: Auto-aimed at player position
- **Collision**: Circle-based against player and projectiles

#### Power-Up System
- Weapons, health, bomb refills spawn from defeated enemies
- Visual indicators (particles, glow effects)
- Auto-collect when player nearby

#### Audio System
- Dynamic synth-based sound design (Web Audio API)
- Sound effects: Weapon fire, hits, explosions, combo notifications
- Spatial audio panning based on event location
- Mute toggle with persistent state (localStorage)
- Fallback for browsers without audio support

#### Difficulty & Progression
- **Levels**: 3 difficulties (Easy, Normal, Hard)
- **Scaling**: Score multiplier, enemy spawn frequency, boss health
- **Stage System**: Multiple waves per stage, boss encounters after waves
- **Combo System**: Track consecutive kills for score multiplier

### Visual Effects
- **Hit Flash**: Player/enemy damage feedback
- **Screen Shake**: Intensity-based camera jitter
- **Combo Popups**: Floating score notifications
- **Stage Banners**: Level transition displays
- **Particle Effects**: Explosions, hit markers, weapon trails

### User Interface
- **HUD**: Score, lives, bombs, weapon, FPS counter
- **Motion Button**: Toggle auto-aiming (accessibility)
- **Mute Button**: Audio control
- **Mobile Joystick**: Virtual stick for touch devices
- **Menus**: Start menu, pause, game over, level up screens

### Input Handling
- **Keyboard**: WASD/Arrows for movement, Space to shoot, P to pause, M for mute
- **Mouse**: Click to shoot, movement, optional auto-aim with motion button
- **Touch**: Virtual joystick (left side) and tap-to-shoot (right side)
- **Gamepad**: Support via keyboard mapping

### Performance Optimization
- Performance tracking (FPS counter)
- Efficient canvas clearing and redraw
- Object pooling patterns for bullets and enemies
- Collision detection optimization through spatial awareness
- Audio context reuse to avoid per-frame allocations

## Key Functions

### Game Loop
- `update(dt)` - Physics and state updates
- `draw()` - Render all game objects
- Main loop runs at 60 FPS with delta-time compensation

### Collision Detection
- Circle-based hitboxes for simplicity
- Player-bullet, enemy-bullet, pickup-player tests
- Efficient nested loops with distance calculations

### Drawing Primitives
- `drawPlane(x, y, w, h, color, darkColor, flame)` - Renders player aircraft
- `drawBossShip(b)` - Multi-part boss sprite rendering
- Custom shapes built from canvas paths

### Event System
- Touch events for mobile
- Mouse events for desktop
- Resize listener for responsive scaling
- Visibility change detection for pause on tab switch

## Data Persistence
- Uses `localStorage` for:
  - Mute state
  - High score tracking
  - Difficulty preference (potential)

## Responsive Design
- CSS media queries for mobile layout
- Touch-action properties to prevent browser gestures
- Viewport meta tags for proper scaling
- Safe area insets for notched displays

## Game Balance
- Enemy difficulty scales with stage and difficulty setting
- Boss health increases with progression
- Weapon variety prevents dominance of single strategy
- Bomb cooldown creates resource management gameplay
- Score multiplier for combos encourages aggressive play

## Browser Compatibility
- Requires modern browser (ES6+ JavaScript, Canvas, Web Audio API)
- Graceful degradation for audio unavailable
- Touch event support for mobile
- Fullscreen API support where available

## Extension Points
- Boss patterns can be added to `spawnBoss()` function
- New weapon types via weapon cycling system
- Additional stages by increasing stage progression logic
- New enemy types through spawn system
- Custom sound effects in audio synthesis functions

## Development Notes
- All code is inline for single-file deployment
- No external dependencies or build step required
- Performance-critical sections use requestAnimationFrame
- Audio synthesis uses typed arrays for efficiency
- Mobile-first approach with touch-optimized controls

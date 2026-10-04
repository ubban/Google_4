# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5, CSS, and JavaScript. The entire game is contained in a single `index.html` file (~1,669 lines). Players control an aircraft, switch between weapon types, drop smart bombs, and defeat stage bosses to advance through levels.

**Target Platform**: Web browsers (desktop and mobile)  
**Technology**: Vanilla JavaScript, HTML5 Canvas, Web Audio API  
**Repository**: `ubban/google_4`

## File Structure

```
Google_4/
├── index.html          # Complete game implementation (HTML + CSS + JavaScript)
├── README.md           # Basic project info
└── CLAUDE.md          # This file - codebase documentation
```

## Architecture Overview

The game uses a monolithic single-file architecture with a self-executing IIFE (Immediately Invoked Function Expression). The game loop runs via `requestAnimationFrame`, processing input, updating game state, and rendering each frame.

### Main Game Systems

1. **Game State Management**: Tracks current game phase (menu, playing, paused, gameover), difficulty, score, and wave data
2. **Input Handling**: Keyboard, mouse, touch, and motion sensor (accelerometer) support
3. **Rendering Pipeline**: Canvas-based 2D graphics with layered drawing (background, entities, UI)
4. **Entity System**: Player, enemies, bullets, explosions, and pickup particles
5. **Wave/Stage System**: Predefined enemy formations that spawn in waves
6. **Boss System**: Boss enemies with health bars, attack patterns, and special mechanics
7. **Audio System**: Web Audio API for sound effects and dynamic mixing
8. **Storage**: LocalStorage for best scores, settings (mute, difficulty)

## Key Game Mechanics

### Player Controls
- **Movement**: Arrow keys (desktop) or touch/joystick (mobile)
- **Fire**: Spacebar or touch fire button
- **Bomb**: Z key or touch bomb button
- **Pause**: P key or pause button

### Weapon System
Players cycle through weapon types (Cannon, Pulse, Laser). Each weapon has:
- Bullet pattern and speed
- Upgrade levels (affects damage and bullet count)
- Energy cost per shot

### Difficulty Levels
- **EASY**: Lower enemy health, fewer bullets
- **NORMAL**: Standard difficulty
- **HARD**: Increased enemy health and attack frequency

### Boss Mechanics
- Each stage ends with a boss encounter
- Bosses have health bars, multiple attack patterns, and item drops
- Defeating bosses unlocks the next stage

## Core JavaScript Structure

### Key Global Variables
- `gameState`: Current game phase (menu, playing, paused, gameover)
- `player`: Player aircraft object with position, health, weapon, bombs
- `waves`: Array of predefined enemy waves
- `enemies`: Active enemy entities
- `bullets`: Active projectiles
- `effects`: Particle effects and explosions
- `score`, `bestScore`: Gameplay scoring
- `combo`: Consecutive enemy hits for multiplier
- `currentWave`, `waveTimer`: Wave spawning control

### Main Game Loop
```javascript
function gameLoop() {
  // 1. Update game state
  // 2. Handle input
  // 3. Update entities (player, enemies, bullets, effects)
  // 4. Check collisions
  // 5. Draw frame
  requestAnimationFrame(gameLoop);
}
```

### Key Functions

#### Initialization & Setup
- `resetGame()`: Initialize game state
- `startGame()`: Begin gameplay from menu
- `changeWave(waveIndex)`: Load and start a new wave
- `initAudio()`: Set up Web Audio API

#### Game Logic
- `updatePlayer()`: Handle player movement and fire rate
- `updateEnemies()`: Move enemies, handle movement patterns
- `updateBullets()`: Move bullets, check for off-canvas
- `updateEffects()`: Animate particles and explosions
- `checkCollisions()`: Detect bullet-enemy and enemy-player hits
- `advanceWave()`: Move to next stage
- `endGame(won)`: Handle game over state

#### Rendering
- `draw()`: Main draw function orchestrating all rendering
- `drawPlayer()`: Render player aircraft with rotation
- `drawEnemies()`: Render all enemy types
- `drawBullets()`: Render projectiles
- `drawEffects()`: Render explosions and particles
- `drawHUD()`: Render score, weapons, health UI

#### Input Handling
- `handleKeyDown()`, `handleKeyUp()`: Keyboard input
- `handleMouseMove()`, `handleMouseClick()`: Mouse input
- `handleTouchStart()`, `handleTouchMove()`, `handleTouchEnd()`: Touch input
- `handleDeviceMotion()`: Accelerometer for mobile tilt control

#### Audio
- `playSound(name)`: Play a sound effect
- `updateAudioLevel()`: Adjust audio based on game activity

### Entity Objects

#### Player Object
```javascript
player = {
  x, y,                    // Position
  velX, velY,              // Velocity
  width, height,           // Dimensions
  health,                  // Current health
  maxHealth,               // Maximum health
  angle,                   // Rotation angle
  weapon,                  // Current weapon type
  weaponLevel,             // Weapon upgrade level
  bombs,                   // Number of bombs remaining
  fireRate,                // Cooldown between shots
  fireTimer,               // Current fire cooldown
  shieldTimer,             // Invulnerability timer after damage
}
```

#### Enemy Objects
```javascript
enemy = {
  x, y,                    // Position
  velX, velY,              // Velocity
  width, height,           // Dimensions
  health,                  // Health points
  maxHealth,               // Maximum health
  type,                    // Enemy type (fighter, bomber, etc.)
  fireRate,                // How often to fire
  fireTimer,               // Fire cooldown
  score,                   // Points awarded on defeat
  bulletDamage,            // Damage dealt to player
}
```

#### Boss Objects
Special enemy type with:
- Larger health pool
- Health bar display
- Multiple attack phases
- Item drops on defeat
- Complex movement patterns

## Wave & Level System

Waves are predefined in the `waves` array with enemy configurations:
- Enemy types and positions
- Spawn timing and patterns
- Difficulty scaling per difficulty level

Each level culminates in a boss fight, then progresses to the next stage.

## Rendering Pipeline

1. **Clear Canvas**: Fill with sky gradient
2. **Draw Stars**: Parallax starfield background
3. **Draw Enemies**: With health bars (if boss)
4. **Draw Bullets**: Player and enemy projectiles
5. **Draw Player**: With rotation and shield effects
6. **Draw Effects**: Explosions and damage particles
7. **Draw HUD**: Score, weapons, health, bomb count
8. **Draw Overlays**: Game over screen, pause menu

## Storage & Persistence

LocalStorage keys:
- `skyfighter.bestScore`: Best score achieved
- `skyfighter.muted`: Audio mute state (boolean)
- `skyfighter.difficulty`: Last selected difficulty

## Mobile/Touch Optimizations

- Touch-based joystick for movement
- On-screen fire and bomb buttons
- Device motion (accelerometer) support
- Responsive canvas sizing
- Safe area insets for notched devices

## Audio

Uses Web Audio API for:
- Sound effects (shoot, hit, explosion, etc.)
- Dynamic audio scheduling to prevent overlap
- Mute toggle for audio control

## Common Patterns for Modification

### Add a New Enemy Type
1. Define enemy object in wave configuration
2. Add drawing code in `drawEnemies()`
3. Add update logic in `updateEnemies()`
4. Define firing pattern if needed

### Add a New Weapon
1. Define weapon in weapon list with stats (fireRate, damagePerBullet, etc.)
2. Create bullet pattern in firing code
3. Add weapon drawing in HUD

### Add New Boss
1. Create boss enemy object with unique stats and attack pattern
2. Implement attack pattern function
3. Add boss arena/background if needed
4. Define reward items on defeat

### Adjust Difficulty
Modify difficulty scaling multipliers in the relevant update functions or enemy definitions.

## Performance Considerations

- **Canvas Size**: Responsive with `max-width` and `max-height` to fit viewport
- **Entity Pooling**: Reuse bullet and effect objects when possible
- **Draw Order**: Optimize layering to minimize overdraw
- **Game Loop**: Uses `requestAnimationFrame` for 60 FPS target
- **Memory**: Single IIFE scope keeps globals contained

## Debugging Tips

1. **Open DevTools Console**: All game variables are in window scope
2. **Pause Game**: Press P to pause and inspect state
3. **Check Collision**: Look for collision detection in `checkCollisions()`
4. **Audio Issues**: Verify Web Audio API context is started
5. **Touch Issues**: Check `handleTouchMove()` and joystick calculation

## Browser Compatibility

- Chrome/Edge: Full support
- Firefox: Full support
- Safari: Full support (requires user gesture for audio)
- Mobile browsers: Full support with touch optimizations

## Future Enhancement Ideas

- Additional weapon types or upgrade system
- Power-up items (shield, rapid fire, etc.)
- Story/narrative elements
- Leaderboards (requires backend)
- Different stage themes
- 2-player co-op
- Procedurally generated levels

## Development Workflow

1. **Edit index.html**: Make your changes to the game code
2. **Test locally**: Open in a web browser
3. **Check git status**: Review changes
4. **Commit changes**: Use clear commit messages describing the modification
5. **Push to branch**: Push to `claude/inspiring-planck-tku0qa`

## Key Sections in index.html

- **Lines 1-50**: HTML structure and meta tags
- **Lines 51-300**: CSS styling (canvas, HUD, buttons, overlays)
- **Lines 301-400**: HTML markup (canvas, buttons, overlays)
- **Lines 401-end**: JavaScript game implementation
  - Game initialization
  - Wave/enemy data
  - Main game loop
  - Input handlers
  - Rendering functions
  - Audio system
  - Collision detection

---

*Last updated: 2026-10-04*  
Generated by Claude Code Agent

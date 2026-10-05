# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 canvas application. The game features a player-controlled aircraft battling waves of enemies, multiple weapon types, smart bombs, boss encounters, and a dynamic difficulty system.

- **Type**: Web-based game
- **Technology**: HTML5, Canvas, Web Audio API, JavaScript (ES6+)
- **Entry Point**: `index.html`
- **Repository**: `ubban/Google_4`

## Architecture

The entire game is implemented in a single HTML file (`index.html` - 1,669 lines) using a monolithic architecture with modular JavaScript sections:

### Core Game Loop
- **Animation Frame Loop**: Uses `requestAnimationFrame()` for consistent 60 FPS rendering
- **Update/Draw Separation**: Distinct `update(dt)` and `draw()` functions for logic and rendering
- **Delta Time**: Frame-independent physics using time deltas

### Major Systems

#### 1. **Game State Management**
- Game states: `running`, `paused`, `gameOver`
- Wave system with progressive difficulty
- Score tracking and high score persistence (localStorage)
- Player stats: health, bombs, multiplier

#### 2. **Player System**
- Position tracking and boundary constraints
- Health/lives system (3 lives by default)
- Weapon switching (primary, laser, spread)
- Smart bomb management
- Invulnerability frames after damage

#### 3. **Combat System**
- **Weapons**:
  - Primary: Standard bullets
  - Laser: Piercing beam weapon
  - Spread: Wide-angle projectile spray
- **Bullets**: Tracked in `bullets` array with laser type distinction
- **Enemy Bullets**: Circular projectiles from enemies
- **Collision Detection**: AABB (Axis-Aligned Bounding Box) collision system

#### 4. **Enemy System**
- Multiple enemy types defined in `ENEMY_TYPES`:
  - Fighter: Standard enemies with basic projectiles
  - Tank: Heavy armored enemies with health bar
  - Boss: End-of-wave boss encounter with complex attack patterns
- Enemy spawning via wave controller
- Pathfinding with sin/cos trajectory patterns

#### 5. **Visual & Effects**
- Canvas rendering with layered drawing order
- Particle system for explosions and hit effects
- Screen shake effect for impact feedback
- Vignette effect for damage/critical state
- Flash effects for hits and bombs
- Glow effects for weapon visuals (quality-dependent)

#### 6. **Audio System**
- Web Audio API for sound effects
- Dynamic audio synthesis (no pre-recorded files)
- Sounds for: shooting, explosions, bomb, power-ups, game over
- Lazy initialization on first user interaction

#### 7. **UI & HUD**
- Responsive HUD with player stats (health, bombs, weapon, multiplier)
- Game overlay with start/pause controls
- Performance stats overlay (FPS, frame time, memory)
- Game over screen with high score tracking
- Touch/mobile responsive design

#### 8. **Input Handling**
- Mouse tracking for aiming
- Keyboard controls (WASD for movement, Space for shoot, E for bomb)
- Touch/mobile controls with crosshair feedback
- Multi-input support

## Key Files

### index.html (Single Application File)
The complete game implementation in a single HTML file with:
- **Head Section** (lines 1-150): Meta tags, styles, favicon
- **Canvas Container** (lines 150-200): Game viewport and HUD elements
- **Script Section** (lines 200+): Complete game logic

## Game Mechanics

### Weapon System
1. **Primary Bullets** (Default)
   - Rapid-fire standard projectiles
   - High rate of fire

2. **Laser Weapon**
   - Piercing beam that passes through enemies
   - Higher damage but slower fire rate

3. **Spread Weapon**
   - Wide spray of bullets
   - Good area coverage but dispersed damage

Weapons cycle through on pickup or can be selected with number keys.

### Enemy Progression
- Waves increase in difficulty with more enemies and stronger types
- Boss appears every 5-10 waves
- Boss has multiple attack patterns and significant health pool

### Power-ups
- **Life (Red)**: Restores one health
- **Bomb (Yellow)**: Adds smart bomb charge
- **Weapon (Green)**: Increases current weapon power

### Bomb System
- Limited supply (typically 3)
- Clears all on-screen bullets and damages all enemies
- High damage to bosses
- Visual feedback with flash effect and screen shake

### Difficulty Scaling
- Enemy count increases per wave
- Boss health pool increases with progression
- Multiplier increases score value over time
- Performance throttling: quality setting reduces particle count and glow effects on low FPS

## Development Notes

### Performance Optimization
- Quality setting adapts to frame rate
- Particle limits for lower-end devices
- Shadow effects and glow disabled when quality = 0 (low FPS)
- Input debouncing for touch events

### Code Organization
Within the single file:
- **Configuration**: `ENEMY_TYPES`, `ENEMY_PROJECTILE_SIZE`, game constants
- **Game Variables**: Arrays for entities (enemies, bullets, particles, etc.)
- **Initialization**: Canvas setup, event listeners
- **Functions**: Organized by concern (collision, drawing, audio, game logic)
- **Main Loop**: `requestAnimationFrame(loop)` entry point

### Entity Management
- Arrays for: `enemies`, `bullets`, `enemyBullets`, `powerUps`, `particles`
- No entity pooling; direct array push/filter operations
- Memory managed through periodic cleanup (filter dead entities)

### State Persistence
- High score stored in localStorage under key: game-high-score
- Survives page reloads

## How to Modify

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with properties: `speed`, `hp`, `fireRate`, `projectileDamage`, `scoreValue`
2. Update spawning logic in wave controller
3. Add rendering logic in `drawPlane()` if visual customization needed

### Adding a New Weapon
1. Add weapon type to weapon cycle logic
2. Create bullet generation code in fire() function
3. Update HUD weapon display logic
4. Adjust collision/rendering for new projectile type

### Adjusting Difficulty
- Modify wave controller spawn rates
- Adjust `ENEMY_TYPES` health/damage values
- Change multiplier growth rate
- Modify enemy bullet firing frequency

### Performance Tuning
- Adjust `MAX_PARTICLES` limit
- Modify quality threshold for FPS optimization
- Optimize collision detection with spatial partitioning (future)
- Consider object pooling for frequently created entities

## Testing Recommendations

1. **Game Loop**: Verify smooth 60 FPS performance
2. **Collision**: Test bullet hit detection accuracy
3. **Weapons**: Confirm each weapon's fire rate and damage
4. **Enemies**: Verify spawning, movement, and boss behavior
5. **Audio**: Check all sound effects trigger correctly
6. **Mobile**: Test touch controls and responsive HUD sizing
7. **Performance**: Monitor FPS and memory usage over extended play

## Future Enhancement Ideas

- Difficulty settings (Easy/Normal/Hard)
- Additional enemy types or mini-boss encounters
- Weapon upgrade system
- Leaderboard integration
- Sound effect volume controls
- Visual settings (quality, screen size)
- Progressive unlock system for weapons
- Combo/chain mechanics for score multipliers

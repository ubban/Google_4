# Sky Fighter Codebase

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built entirely in HTML5 Canvas. It's a single-file web game featuring classic arcade action with weapon switching, smart bombs, enemy waves, and boss battles.

- **Technology**: Vanilla JavaScript, HTML5 Canvas
- **Type**: Browser-based game
- **Platform**: Cross-platform (any modern browser supporting Canvas)
- **File**: `index.html` (single self-contained file)

## Game Features

### Core Gameplay
- **Vertical scrolling shooter**: Player controls a fighter at the bottom of the screen, enemies descend from above
- **Player controls**: Arrow keys or touch input to move, spacebar or tap to shoot
- **Health system**: Player starts with 3 health points (hearts)
- **Smart bombs**: Limited-use special attack that clears the screen
- **Score system**: Points earned by defeating enemies
- **Multiplier**: Score multiplier increases with consecutive hits

### Weapon System
- **Multiple weapons**: Different weapon types with varying firing patterns
- **Weapon switching**: Cycle through available weapons (Q/E keys or buttons)
- **Weapon types**:
  - Standard: Straight shots
  - Spread: Multiple angle shots
  - Homing: Seeks nearest target
  - Burst: Fast-fire pattern

### Enemy Types
- **Basic enemies**: Standard small fighters
- **Harder enemies**: More durable, faster
- **Boss battles**: Major encounters at stage end with extended health bars
- **Wave progression**: Difficulty increases with each wave

### Visual Polish
- **Gradient sky background**: Dynamic blue sky gradient
- **Particle effects**: Explosions and visual feedback
- **Vignette effect**: Red screen edges when taking damage (critical state)
- **Flash effects**: Screen flash on hit and bomb activation
- **HUD**: Real-time display of score, health, bombs, weapon type, multiplier
- **Mobile-friendly**: Responsive layout with touch-action support

## File Structure

### `index.html` - Main Game File
Single file containing:
- **HTML structure**: Basic container for canvas and UI elements
- **CSS styling**: Game styling, animations, responsive design
- **JavaScript**: Complete game logic and rendering

### Key Sections in JavaScript

#### Game State Object
Maintains current game state:
- `canvasWidth`, `canvasHeight`: Game dimensions
- `score`, `multiplier`: Scoring system
- `health`, `bombs`: Player status
- `currentWeapon`: Active weapon type
- `gameRunning`, `gamePaused`: State flags
- `waves`: Wave progression
- `bossHealth`, `bossActive`: Boss battle state

#### Game Objects
- **Player**: Position, health, movement, shooting
- **Enemies**: Position, health, velocity, shooting patterns
- **Projectiles**: Position, velocity, owner (player vs enemy)
- **Effects**: Explosions, visual feedback, particles
- **Bosses**: Extended health, special attack patterns, stage clear triggers

#### Core Game Loop
- `init()`: Initialize game state and canvas
- `update()`: Update game state (movement, collisions, spawning)
- `render()`: Draw all game objects and HUD
- `gameLoop()`: Main animation frame loop

#### Input Handling
- **Keyboard**: Arrow keys (movement), Space (shoot), Q/E (weapon switch), P (pause)
- **Touch/Mouse**: Click to move player position, hold to shoot
- **Mobile support**: Responsive to device rotation and safe areas

#### Collision Detection
- **Bullet vs Enemy**: Remove both on hit, add score
- **Enemy Bullet vs Player**: Reduce health, apply vignette effect
- **Bomb vs Enemies**: Clear all enemies on screen
- **Enemy vs Player**: Reduce health on contact

#### Weapon System
Each weapon has:
- Fire rate (cooldown between shots)
- Spread pattern (single, spread, homing, burst)
- Damage per bullet
- Visual appearance

#### Enemy AI
- Basic movement: Descend while moving horizontally
- Firing: Occasionally shoot toward player position
- Waves: Enemy composition changes per wave
- Difficulty scaling: Speed and fire rate increase over time

#### Boss Encounters
- Appear every N waves
- Extended health bar (multiple hits to defeat)
- Special movement patterns
- Increased fire rate and projectile count
- Wave progression continues after boss defeat

## Development Guidelines

### Adding New Features
1. **New Enemy Types**: Add to enemy spawn logic in wave progression
2. **New Weapons**: Define weapon properties, add to weapon switching loop
3. **Visual Effects**: Add to render() or create particle system
4. **Game Balance**: Adjust difficulty settings, enemy health, fire rates

### Performance Considerations
- Canvas rendering is optimized for 60 FPS
- Particle effects are limited to prevent slowdown
- Enemy count scales with wave progression
- Collision detection uses bounding box (AABB)

### Mobile Optimization
- Touch-friendly UI with large hit targets
- Responsive canvas sizing with `max-width` and `max-height`
- Safe area insets for notched devices (`env(safe-area-inset-top)`)
- Landscape orientation optimized
- No horizontal scroll

### Browser Compatibility
- Requires HTML5 Canvas support
- ES6+ JavaScript (arrow functions, template literals, destructuring)
- Modern CSS (Grid, Flexbox, Backdrop filters)
- Touch events for mobile

## Testing & Gameplay Tips

### Testing Checklist
- [ ] Player movement and boundaries work correctly
- [ ] All weapon types fire and damage enemies
- [ ] Weapon switching cycles through types
- [ ] Enemy spawning follows wave pattern
- [ ] Collision detection for all object types
- [ ] Score and multiplier increment correctly
- [ ] Health loss and bomb usage work
- [ ] Boss battles trigger and complete
- [ ] Vignette and flash effects display properly
- [ ] Game pause/resume functions
- [ ] Mobile touch controls responsive
- [ ] No memory leaks during extended play

### Game Balance Parameters
- **Player health**: 3 starting health points
- **Bombs**: 2 starting bombs
- **Enemy difficulty**: Increases by 10% per wave
- **Score multiplier**: Resets on player hit, increases with consecutive hits
- **Boss health**: Scales with wave progression

## Future Enhancement Ideas

1. **Sound effects and music**: Add audio for shooting, explosions, background music
2. **Difficulty levels**: Easy, Normal, Hard modes
3. **Weapon upgrades**: Pickups during gameplay
4. **Shield/protective items**: Temporary invulnerability
5. **Leaderboard**: High score persistence
6. **More visual effects**: Screen shake, particle trails
7. **Story/campaign mode**: Themed levels and bosses
8. **Special enemies**: Rare spawns with unique behaviors
9. **Power-ups**: Speed boost, rapid fire, shield
10. **Level variety**: Different backgrounds, terrain obstacles

## Author Notes

This is a complete, playable game suitable for:
- Educational purposes (learning Canvas and game development)
- Portfolio showcase
- Browser-based game distribution
- Mobile-friendly arcade game
- Classic arcade enthusiasts

The game uses only vanilla JavaScript with no external libraries, making it lightweight and fast-loading.

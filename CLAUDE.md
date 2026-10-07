# Sky Fighter - CLAUDE.md

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 canvas application. It features player-controlled aircraft, weapon switching, smart bombs, stage bosses, and wave-based enemy encounters.

### Key Characteristics
- **Single-file implementation**: All game logic, rendering, and assets are contained in `index.html`
- **Technology**: HTML5, Canvas API, pure JavaScript (no external libraries)
- **Responsive design**: Adapts to different screen sizes and supports touch controls
- **Dark theme**: Space-themed UI with blue and cyan color palette

## Architecture

### Game Loop
The game runs on a continuous animation loop that:
1. Updates game state (player position, enemy positions, projectiles)
2. Detects collisions between player, enemies, projectiles, and obstacles
3. Renders all game entities to the canvas
4. Manages wave progression and level difficulty

### Core Game Systems

#### Player System
- Player starts at bottom center of screen
- Movement controlled by mouse/touch position
- Three weapon types with switching capability
- Smart bomb power-up for clearing screen
- Health/lives system

#### Enemy System
- Wave-based spawning with increasing difficulty
- Multiple enemy types with different behaviors
- Boss encounters at end of stages
- Procedural behavior patterns (sine wave movement, targeted shots, etc.)

#### Weapon System
- **Rapid Fire**: Standard continuous shooting
- **Spread Shot**: Multiple projectiles in fan pattern
- **Laser**: Concentrated beam weapon
- Weapons cycle through on player input

#### Canvas Rendering
- Direct canvas API calls for all rendering (no external graphics library)
- Procedurally generated sprites/shapes (circles, triangles, rectangles)
- Text-based HUD for score, lives, weapon status
- Performance optimized for mobile devices

### Game State Variables
The game maintains state for:
- Player position, velocity, health, current weapon
- Array of active enemies with position, type, health, behavior state
- Array of active projectiles (player and enemy)
- Score, level, wave count, power-ups collected
- Game pause/game-over states

## Development Guidelines

### Working on the Game

1. **Single File Architecture**: All changes go into `index.html`. Structure your edits by:
   - `<style>` section for CSS styling
   - `<script>` section for JavaScript game logic
   - HTML elements for container and HUD

2. **Canvas Coordinates**: 
   - Origin (0,0) is top-left
   - X increases rightward, Y increases downward
   - Canvas size is responsive but maintains aspect ratio

3. **Adding New Features**:
   - New enemy types: Add to the enemy initialization logic and update behavior functions
   - New weapons: Add weapon type enum, firing logic, and rendering code
   - New visual effects: Use canvas methods (drawCircle, fillRect, etc.) or path drawing

### Performance Considerations

- The game is optimized for 60 FPS on modern devices
- Canvas rendering is the primary bottleneck
- Minimize object allocations in the game loop
- Use efficient collision detection (rectangular bounds checking before pixel-perfect)
- Mobile considerations: Touch events replace mouse events on touch devices

### Testing the Game

1. **Open in Browser**: Simply open `index.html` in a web browser
2. **Desktop Controls**: 
   - Mouse movement to aim/move player
   - Click/key press to fire weapon
   - Spacebar or UI button to switch weapons
   - Spacebar or UI button to use smart bomb

3. **Mobile Testing**:
   - Touch to move player
   - Touch-and-hold to fire continuously
   - On-screen buttons for weapon switch and smart bomb

### Visual Hierarchy

- **HUD**: Top of screen - score, lives, weapon type, bomb count
- **Game Area**: Main canvas showing player, enemies, projectiles, effects
- **UI Messages**: Game-over, level complete, stage boss announcements

## Code Sections (in `index.html`)

### HTML Structure
- `<div id="container">`: Main game container
- `<canvas id="gameCanvas">`: Primary rendering surface
- `<div id="hud">`: Heads-up display for game info
- `<div id="uiOverlay">`: Overlay for menus and messages

### CSS Styling
- `:root` variables for dark theme
- Responsive sizing using CSS clamp() and viewport units
- Mobile-first design approach
- Safe area insets for notch/status bar devices

### JavaScript Game Logic
Organized by functionality:
- **Input handling**: Mouse/touch/keyboard event listeners
- **Game state initialization**: Game object with player, enemies, projectiles arrays
- **Update functions**: Movement, collisions, state transitions
- **Render functions**: Canvas drawing for all entities
- **Utility functions**: Distance calculations, random number generation, collision detection

## Common Tasks

### Adjusting Game Difficulty
- Enemy spawn rates and types are in the wave progression logic
- Adjust `enemy.health` for different durability
- Modify projectile counts and firing patterns

### Adding Visual Effects
- Particles for explosions: Create small circles/shapes at enemy position with velocity
- Screen flash on damage: Draw semi-transparent rectangle over canvas
- Weapon trails: Render line segments from previous projectile positions

### Tweaking Player Mechanics
- Movement speed: Adjust mouse/touch coordinate scaling
- Weapon firing rate: Modify `fireRate` values
- Bomb recharge rate: Adjust `bombRechargeRate`

## Deployment Notes

- Game is self-contained in single HTML file
- No build process required
- No external dependencies
- Can be hosted on any static web server
- Works on all modern browsers (desktop and mobile)

## Future Enhancements

Potential additions without changing architecture:
- Sound effects and background music (using Web Audio API)
- More enemy varieties and boss types
- Power-up system (shields, extra weapons, etc.)
- High score persistence (localStorage)
- Leaderboard (if backend integration desired)
- Mobile app wrapper (using Electron or Cordova)

---

**Last Updated**: 2026-10-07
**Project Type**: Single-file HTML5 Game
**Target Browsers**: Modern browsers (Chrome 90+, Firefox 88+, Safari 14+)

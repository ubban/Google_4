# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-page HTML5 web application. The game features:

- **Player Aircraft**: Controllable with keyboard (arrow keys) and mouse/touch input
- **Weapon System**: Multiple weapon types that can be switched using number keys (1-3)
- **Smart Bombs**: Limited-use area-of-effect attacks triggered by spacebar
- **Enemy Waves**: Progressive difficulty with varied enemy patterns
- **Stage Bosses**: Challenging boss encounters at the end of each stage
- **Score System**: Points-based progression with combo multipliers

## File Structure

```
Google_4/
├── README.md           # Basic project readme
├── CLAUDE.md           # This file - codebase documentation
└── index.html          # Complete game implementation (~1670 lines)
```

## Architecture

The entire game is implemented in a single `index.html` file with:

- **HTML Structure**: Canvas-based rendering with UI elements for HUD (heads-up display)
- **CSS Styling**: Game styling with animations, visual effects, and responsive layout
- **JavaScript Logic**: Pure vanilla JavaScript game engine with no external dependencies

## Key Game Components

### Game State
- Player position, health (3 hearts max), and inventory (bombs)
- Current weapon selection (3 types)
- Score and combo multiplier
- Enemy wave counter and stage progression
- Boss health states

### Rendering System
- Canvas-based 2D graphics rendering
- Sprite system for player, enemies, projectiles, and explosions
- Effects layer: damage vignette, flash hits, and bomb explosions
- HUD overlay with stats display

### Input Handling
- **Keyboard**: Arrow keys (movement), number keys (weapon select), spacebar (bomb)
- **Mouse**: Crosshair tracking for aiming
- **Touch**: Mobile-optimized input with analog stick simulation

### Game Loop
- Main animation frame loop driving all updates
- Collision detection between player bullets and enemies
- Enemy AI and movement patterns
- Score and multiplier calculations

## Code Organization

The code is organized as a single self-contained module with:

1. **Initialization**: Canvas setup and game state variables
2. **Game Objects**: Player, Enemy, Projectile, Explosion classes
3. **Game Logic**: 
   - Wave generation and enemy spawning
   - Collision detection
   - Scoring system
   - Status updates (HUD)
4. **Input System**: Event listeners for keyboard, mouse, and touch
5. **Rendering**: Canvas drawing functions and visual effects
6. **Main Loop**: Animation frame scheduling and update/render cycle

## Development Guidelines

### Adding Features
- Keep additions self-contained and avoid external dependencies
- Test on both desktop (keyboard/mouse) and mobile (touch) platforms
- Update HUD display for any new player stats
- Ensure collision detection works with new entity types

### Modifying Weapons
- Weapon types are defined and switch via number keys
- Each weapon has: fire rate, damage, and visual properties
- Projectile pooling prevents object creation overhead

### Adding Enemy Types
- New enemies should inherit the Enemy pattern
- Define movement, health, and scoring properties
- Integrate with wave spawning system

### Performance Considerations
- Canvas rendering is the primary bottleneck
- Use requestAnimationFrame for smooth 60 FPS gameplay
- Minimize DOM updates; mostly draw to canvas
- Pooled objects reduce garbage collection pauses

## How to Run

1. Open `index.html` in a web browser
2. Game starts automatically when page loads
3. Use arrow keys or mouse to control the aircraft
4. Switch weapons with 1, 2, 3 keys
5. Press spacebar to use smart bombs (limited supply)

### Testing
- Test on various screen sizes (desktop, tablet, mobile)
- Verify touch input works on mobile devices
- Check that weapon switching is responsive
- Ensure boss encounters are challenging but fair

## Notable Features

- **Responsive Design**: Game scales to fill the viewport while maintaining aspect ratio
- **Mobile Support**: Full touch control with visual feedback
- **Screen Shake**: Visual feedback on hits and explosions
- **Sound Effects Ready**: Structure supports audio integration (currently visual-only)
- **Progressive Difficulty**: Enemies and bosses get stronger with each stage

## Future Enhancement Ideas

- Add audio/music system
- Implement multiple difficulty levels
- Create level editor for custom waves
- Add power-ups (health, bombs, weapon upgrades)
- Leaderboard/high score persistence
- Particle effects system for more visual polish
- Networking/multiplayer support

## Dependencies

**None** - This is a pure vanilla JavaScript game with no external libraries or frameworks.

## Last Updated

Created: 2026-10-03  
Initial project documentation for Sky Fighter game.

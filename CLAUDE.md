# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a browser-based Raiden-style vertical shoot-'em-up game built with vanilla HTML5 Canvas and JavaScript. Players navigate through stages, switch between weapons, deploy smart bombs, and battle stage bosses.

- **Last Updated**: 2026-02-23
- **Main Entry Point**: `index.html` (1669 lines)
- **Technology Stack**: HTML5 Canvas, Vanilla JavaScript (no frameworks)
- **Target Platform**: Web browser (desktop and mobile)

## Repository Structure

```
Google_4/
├── index.html          # Main game application (single-file implementation)
├── README.md           # Basic project metadata
├── .git/               # Git version control
└── CLAUDE.md          # This documentation file
```

## Architecture Overview

The entire game is implemented as a single-page application in `index.html`. The architecture consists of:

### Frontend Layer
- **Canvas Rendering**: Uses HTML5 Canvas API for all game graphics
- **Responsive Design**: CSS media queries and viewport settings for desktop and mobile
- **Touch & Mouse Input**: Supports both mouse and touch controls for cross-platform play

### Game Engine
The game implements a classic game loop with:
- **Update Phase**: Game logic, collision detection, entity updates
- **Render Phase**: Canvas drawing and scene rendering
- **Input Handling**: Real-time keyboard/mouse/touch input processing

### Core Game Systems

#### Player System
- Single player character with position tracking
- Health/lives system
- Movement constraints within game bounds
- Visual representation as a triangle-shaped aircraft

#### Weapon System
- Multiple weapon types available
- Weapon switching mechanics
- Ammunition tracking
- Different fire patterns and behaviors

#### Enemy System
- Various enemy types with different behaviors
- Enemy spawning and despawning
- Health and damage systems
- Boss encounters at stage endpoints

#### Visual Elements
- HUD (Heads-Up Display) for stats
- Score tracking
- Stage progression display
- Health/bomb indicators

#### Smart Bomb Feature
- Limited use power-up
- Area-of-effect damage
- Visual feedback and animations

## Key Features

1. **Stage-Based Progression**: Multiple stages with increasing difficulty
2. **Weapon System**: Switch between different weapon types during gameplay
3. **Boss Battles**: Stage bosses with unique attack patterns
4. **Smart Bombs**: Limited-use area attacks for clearing dangerous situations
5. **Score System**: Points awarded for defeating enemies
6. **Responsive UI**: HUD adapts to screen size
7. **Cross-Platform**: Works on desktop and mobile devices

## Development Notes

### Canvas Rendering
- The game uses a single canvas element (`#gameCanvas`) for all rendering
- Dynamic sizing to fit the container while maintaining aspect ratio
- CSS gradient background for sky effect

### Input System
- Supports mouse movement for aiming
- Keyboard controls for game actions
- Touch support for mobile play
- Cross-browser compatibility considerations

### Performance Considerations
- Single canvas context reduces rendering overhead
- Efficient entity lifecycle management
- Touch action optimization with CSS `touch-action: none`
- User select disabled to prevent text selection during gameplay

### Mobile Optimization
- Viewport meta tags for proper mobile rendering
- Safe area insets for notch-aware displays
- Touch highlight color disabled
- Responsive font sizing with `clamp()`
- Flexible HUD layout with flexbox

## Browser Compatibility

- Modern browsers with HTML5 Canvas support
- ECMAScript 2015+ features used
- CSS Grid and Flexbox support required
- Mobile browsers: iOS Safari, Chrome, Firefox
- Desktop browsers: Chrome, Firefox, Safari, Edge

## To Run the Game

1. Open `index.html` directly in a web browser
2. Or serve via a simple HTTP server for best performance
3. Use mouse to aim/move player
4. Click to shoot
5. Press keys for additional actions (weapon switch, bombs, etc.)

## Future Enhancement Opportunities

- Audio system (background music, sound effects)
- Save/load functionality for high scores
- Additional enemy types and behaviors
- More boss variations
- Power-ups and collectibles
- Difficulty settings
- Mobile app deployment

## Code Organization

The entire game logic is contained within `index.html`:
- HTML structure: Canvas, HUD container, container div
- CSS styling: Layout, animations, responsive design, theme colors
- JavaScript: Game engine, entity systems, input handling, rendering

All game state is maintained in JavaScript variables and objects. There are no external dependencies or build processes required.

## Contributing Guidelines

When modifying the game:
1. Maintain single-file architecture (append to `index.html`)
2. Keep game loop logic clear and performant
3. Add comments for complex algorithms
4. Test on both desktop and mobile viewports
5. Verify touch and mouse input functionality
6. Update this documentation for significant changes

## Performance Profile

- Canvas size: Responsive (max 100vw × 100vh)
- Frame rate: Target 60 FPS via requestAnimationFrame
- Memory: Single global game state object
- No external HTTP requests during gameplay

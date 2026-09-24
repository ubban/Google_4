# Sky Fighter - Claude Development Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5, Canvas, and JavaScript. The entire game runs as a single HTML file (`index.html`) without external dependencies, making it portable and easy to deploy.

### Game Mechanics
- **Player Ship**: Move vertically/horizontally, avoid enemy fire, defeat bosses
- **Weapons**: Switch between different weapon types during gameplay
- **Power-ups**: Collect drops from defeated enemies
- **Smart Bombs**: Limited-use screen-clearing attacks
- **Stages**: Progressive difficulty with boss encounters
- **Scoring**: Multiplicative score system based on performance

## Tech Stack

- **Runtime**: Vanilla JavaScript (ES6+)
- **Graphics**: HTML5 Canvas 2D API
- **Styling**: Inline CSS (responsive design with CSS clamp for scaling)
- **Audio**: Web Audio API (where implemented)
- **Input**: Touch events + keyboard input for cross-platform support
- **Deployment**: Single HTML file (no build step required)

## File Structure

```
Google_4/
├── index.html         # Single-file game implementation (~1670 lines)
│   ├── <head>         # Metadata, favicon, CSS styles
│   ├── <body>         # Game container and HUD elements
│   └── <script>       # Complete game logic in JavaScript
├── README.md          # Project metadata
└── CLAUDE.md          # This file
```

## Architecture

The game is implemented as a single-file monolithic application. Key architectural patterns:

### Game Loop Structure
- RequestAnimationFrame-based rendering loop
- Delta time tracking for frame-rate independent movement
- Clear-render-update cycle

### Main Game Classes/Objects
1. **Game State Manager**: Tracks game phases (menu, playing, paused, game over)
2. **Player Object**: Ship position, health, weapons, bombs
3. **Enemy System**: Enemy spawn patterns, movement, projectiles
4. **Collision System**: Hit detection for player, projectiles, enemies
5. **Particle System**: Visual effects (explosions, impacts)
6. **UI/HUD System**: Score display, health hearts, bomb counter, weapon indicator
7. **Canvas Renderer**: All drawing operations to the game canvas

### Key Concepts

**Canvas Coordinate System**
- Origin (0,0) at top-left corner
- X increases rightward, Y increases downward
- Game dimensions scale based on viewport

**Entity System**
- Player ship with position (x, y), velocity, rotation
- Enemy entities with AI patterns and projectiles
- Projectile entities with collision detection
- Particle effects with lifetime tracking

**Game Phases**
- Menu/Start phase
- Active gameplay (update loop running)
- Pause state
- Game Over state
- Boss stages with unique patterns

**Score Multiplier**
- Increases with consecutive hits
- Resets on player damage
- Affects point values from defeated enemies

## Development Guidelines

### Code Organization
The code is organized in the `<script>` section with clear logical sections:
1. Initialization and canvas setup
2. Game state variables
3. Entity definitions (Player, Enemy, Projectile classes)
4. Rendering functions (drawBackground, drawPlayer, drawEnemies, etc.)
5. Update functions (updatePlayer, updateEnemies, updateProjectiles, etc.)
6. Collision detection functions
7. Event handlers (keyboard, touch, resize)
8. Main game loop (gameLoop function)
9. Utility functions (createParticles, playSound, etc.)

### Style Conventions
- CSS variables in `:root` for theming
- Responsive sizes using `clamp()` for scaling
- Dark theme optimized for arcade aesthetics
- Accessibility: high contrast, readable fonts

### Performance Considerations
- Canvas-based rendering for efficiency
- Projectile pooling/recycling to reduce allocations
- Viewport-based culling for off-screen entities
- Minimal DOM interaction (mostly canvas drawing)

## Running and Testing

### Local Development
1. Clone the repository
2. Open `index.html` in a web browser (supports Chrome, Firefox, Safari, Edge)
3. Game should run immediately - no build step or dependencies needed

### Testing
- **Manual Testing**: Load in browser, test gameplay mechanics
- **Input Testing**: Test both keyboard and touch controls
- **Cross-browser**: Verify in multiple browsers (desktop and mobile)
- **Responsive**: Test at different viewport sizes

### Deployment
Simply serve `index.html` on any HTTP server. Single file = easy deployment.

## Common Development Tasks

### Adding a New Enemy Type
1. Define enemy properties in the enemy spawning system
2. Implement movement pattern in `updateEnemies()`
3. Add rendering code in `drawEnemies()`
4. Define projectile pattern and timing
5. Test collision detection

### Adding a New Weapon
1. Add weapon type constant and properties
2. Implement projectile creation in player fire function
3. Update projectile rendering with weapon-specific visuals
4. Adjust damage values
5. Update HUD weapon display

### Adding a New Stage/Boss
1. Create stage definition with enemy spawn patterns
2. Implement boss movement and attack patterns
3. Add boss health tracking and phase transitions
4. Design unique projectile patterns
5. Test progression difficulty

### Visual Effects
- Explosions: Use particle system with varying lifetimes
- Screen flash: Use flash overlay with animation keyframes
- Screen shake: Adjust camera position in render functions
- Vignette: Use CSS filters or canvas alpha blending

## Code Patterns

### Entity Update Pattern
```javascript
// For each frame:
1. Update position based on velocity
2. Apply acceleration/deceleration
3. Check boundaries and clamp
4. Test collisions with other entities
5. Render to canvas
```

### Collision Detection
- Circle-to-circle collision (distance-based)
- Rectangle-to-rectangle (AABB)
- Implemented inline in update functions

### Particle Effects
- Create particles on impact events
- Track lifetime and fade opacity
- Remove when lifetime expires
- Renders independently each frame

## Important Files and Sections

- **Canvas Setup**: Early in script tag - initializes canvas and context
- **Game Loop**: `gameLoop()` function - main update/render cycle
- **Input Handlers**: `addEventListener('keydown')`, touch event handlers
- **Rendering Functions**: `draw*()` functions for each game element
- **Update Functions**: `update*()` functions for game logic
- **State Variables**: Global game state (score, health, level, etc.)

## Known Limitations and Future Improvements

### Current Limitations
- Single-file architecture makes code large (~1670 lines)
- No persistent high score storage (no LocalStorage)
- Audio not fully implemented in current version
- Mobile optimization possible (touch controls work but could be refined)

### Potential Enhancements
- Split into modules (if breaking single-file becomes necessary)
- Add sound effects and music
- Implement highscore leaderboard
- Add more stages and boss patterns
- Particle system optimization
- Mobile-specific UI improvements

## Debugging Tips

### Console Logging
- Game state logged via `console.log()` statements
- Check browser dev tools console for errors
- Use breakpoints in DevTools for step debugging

### Visual Debugging
- Temporarily draw collision boxes: modify canvas draw calls
- Draw entity paths: overlay movement traces
- Highlight active entities: change rendering colors

### Performance Profiling
- Use Chrome DevTools Performance tab
- Profile fps with `requestAnimationFrame` timestamps
- Monitor memory usage in DevTools

## Contributing Guidelines

When making changes:
1. Test functionality in multiple browsers
2. Verify responsive behavior on different screen sizes
3. Check that no console errors appear
4. Ensure game remains playable after changes
5. Commit changes with clear, descriptive messages

## Resources and References

- [MDN Canvas API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)
- [MDN Game Development](https://developer.mozilla.org/en-US/docs/Games)
- [Raiden Game Series](https://en.wikipedia.org/wiki/Raiden_(video_game_series)) - Inspiration
- HTML5 requestAnimationFrame for game loops
- Web Audio API for potential sound implementation

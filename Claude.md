# Claude.md - Sky Fighter Development Guide

## Project Overview

Sky Fighter is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5 Canvas and JavaScript. The game features:
- Player aircraft with weapon switching capabilities
- Smart bomb mechanics
- Stage bosses with dynamic patterns
- Responsive design for desktop and mobile devices
- Dark theme UI with immersive visuals

## Project Structure

```
/
├── index.html          # Main game file (includes all HTML, CSS, and JavaScript)
├── README.md           # Basic project info
└── .git/               # Version control
```

## Development Guidelines

### Single-File Architecture

The entire game is contained in `index.html`. This includes:
- **HTML Structure**: Game canvas container and HUD elements
- **CSS Styling**: Responsive design, canvas styling, HUD layout, and animations
- **JavaScript Logic**: Game engine, game states, entity management, collision detection, and input handling

### Code Organization Within index.html

When making changes:
1. **Game Constants**: Top-level configuration (canvas size, colors, speeds)
2. **Asset/Image Data**: Inline data URIs or string literals for graphics
3. **Game State Management**: Game state variables and initialization
4. **Entity Classes**: Player, enemies, bullets, effects
5. **Core Game Engine**: Update loops, render functions, event handlers
6. **Event Listeners**: Input handling and initialization code at the bottom

### Key Concepts

- **Canvas-based rendering**: All visuals are drawn to a 2D canvas context
- **Coordinate system**: Origin (0,0) at top-left, Y increases downward
- **Game loop**: RequestAnimationFrame handles 60fps updates and rendering
- **Collision detection**: Rectangle-based AABB (Axis-Aligned Bounding Box) checking
- **Responsive design**: Uses viewport units and CSS clamp() for scaling

## Common Development Tasks

### Adding Features

1. **New Weapon Type**
   - Define weapon properties (damage, fire rate, spread, sound)
   - Add weapon class or update weapon system
   - Integrate into player weapon selection logic
   - Test collision and effects

2. **New Enemy Type**
   - Create enemy class with movement and attack patterns
   - Define sprite/graphics representation
   - Add to spawn system and wave definitions
   - Implement collision and death effects

3. **New Game State**
   - Add state constant (e.g., `GAME_STATE.PAUSED`)
   - Implement state-specific logic in update/render loops
   - Add transition handlers

### Bug Fixes

- Collision detection issues: Check AABB math and entity bounds
- Movement anomalies: Verify velocity/position update logic
- Rendering issues: Inspect canvas transform state and draw order
- Mobile responsiveness: Test with viewport units and touch event handling

### Performance Optimization

- Use object pooling for frequently created/destroyed entities (bullets, effects)
- Minimize reflows by batching DOM/canvas operations
- Profile with browser DevTools Performance tab
- Reduce draw calls by consolidating canvas operations

## Testing

### Manual Testing Checklist

- [ ] Player movement (keyboard and mouse/touch)
- [ ] Weapon switching and firing
- [ ] Smart bomb activation and effects
- [ ] Enemy spawning and wave progression
- [ ] Boss behavior and patterns
- [ ] Collision detection (bullets, enemies, player)
- [ ] Score and combo systems
- [ ] Mobile responsiveness (landscape/portrait, various screen sizes)
- [ ] Sound effects and audio cues (if implemented)

### Browser Compatibility

- Test on Chrome, Firefox, Safari, and Edge
- Verify touch controls on mobile devices
- Check responsive layout at breakpoints (320px, 768px, 1024px+)

## Build and Deployment

The game is self-contained in `index.html` and requires no build step.

### Development

Open `index.html` directly in a browser or serve with a local HTTP server:
```bash
# Using Python
python3 -m http.server 8000

# Using Node.js (if http-server is installed)
npx http-server

# Using a simple server or IDE preview
```

### Deployment

Upload `index.html` to any web hosting service. The file is completely standalone with no external dependencies.

## Code Style Conventions

- **Variable naming**: camelCase for variables and functions, UPPER_SNAKE_CASE for constants
- **Comments**: Minimal comments; focus on non-obvious logic and intent
- **Function length**: Keep functions focused and under 50 lines when practical
- **Closures**: Use for encapsulation of game subsystems (e.g., player controller, weapon system)
- **No external dependencies**: Keep the game dependency-free for maximum portability

## Performance Notes

- Canvas size: Scales responsively; be mindful of rendering overhead on lower-end devices
- Entity count: Monitor active bullets/enemies to prevent frame rate drops
- Animation frames: Use requestAnimationFrame for smooth 60fps rendering
- Memory: Avoid memory leaks by properly cleaning up entity references

## Collaboration Notes

All changes should:
1. Maintain the single-file structure (no splitting into separate files)
2. Preserve game balance and playability
3. Include inline comments for complex algorithms
4. Be tested in multiple browsers and on mobile
5. Follow the existing code style and organization

## Git Workflow

- Commit messages should be descriptive and reference what was changed (e.g., "Add homing missile weapon", "Fix collision detection for rotating enemies")
- Test locally before committing
- Push to the designated feature branch: `claude/inspiring-planck-6r88rp`

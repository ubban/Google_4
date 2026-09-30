# Sky Fighter — Claude Code Guidelines

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in vanilla HTML, CSS, and JavaScript. The game features:
- Player-controlled aircraft with directional movement
- Multiple weapon types (bullets, missiles, spread shots)
- Smart bomb mechanic for clearing enemies
- Progressive enemy waves and stage bosses
- Responsive design for desktop and mobile platforms

## Architecture

### Single File Structure
- **index.html** (58KB): All game logic, assets, and UI contained in one self-contained HTML file
  - Embedded CSS for styling and layout
  - Embedded JavaScript for game loop, entity management, collision detection, and rendering
  - Inline SVG assets for sprites and effects
  - Base64-encoded or procedural graphics for game elements

### Game Loop Components
1. **Input Handling**: Keyboard and mouse input (with touch fallbacks)
2. **Update Phase**: Entity position updates, collision checks, AI logic
3. **Render Phase**: Canvas-based drawing with sprite batching
4. **State Management**: Game state, wave progression, score tracking

## Key Implementation Patterns

### Game Entities
- **Player**: Single controllable entity with weapon system
- **Enemies**: Spawned in waves with AI pathfinding
- **Projectiles**: Bullets, missiles, and effects
- **Particles**: Explosions and visual feedback
- **Bosses**: Special multi-phase enemies with unique attack patterns

### Rendering
- Single canvas element (`#gameCanvas`) for all visuals
- Pre-calculated sprite positions and frame data
- Camera-relative coordinate system
- Z-ordering via draw order (no explicit z-index needed)

### Collision Detection
- Bounding box collision for most entities
- Circular collision for projectiles vs enemies
- Quadtree or grid-based spatial partitioning (if implemented)

## Development Conventions

### Code Style
- Vanilla JavaScript (ES6+)
- Functional composition for entity systems
- Avoid external dependencies (all-in-one file principle)
- Use consistent naming: camelCase for functions/variables, snake_case for constants

### Game Constants
- Define all magic numbers at the top of the script section
- Use named constants for speeds, damage values, spawn rates
- Maintain aspect ratio and safe viewport boundaries

### Performance Considerations
- Minimize DOM manipulation (canvas-only rendering)
- Pool/reuse entity objects to reduce GC pressure
- Batch similar operations (rendering, collision checks)
- Limit particle count; use pooling for effects
- Profile on mobile devices; optimize for 60 FPS target

### Mobile Optimization
- Touch event handling with preventDefault() where needed
- Responsive viewport scaling via CSS
- Safe area insets for notched/folded devices
- Disable zoom and text selection during gameplay

## Critical Files & Sections

| Section | Purpose | Typical Size |
|---------|---------|--------------|
| Head Meta Tags | Viewport, theme color, description | ~10 lines |
| CSS | Game canvas sizing, HUD layout, responsive design | ~200 lines |
| Game Script | All logic, entity definitions, game loop | ~2000+ lines |

## Modification Guidelines

### When Adding Features
1. **Keep it contained**: Changes should not require external files
2. **Follow the pattern**: Match existing entity/update/render structure
3. **Use constants**: Define new values as module-level constants
4. **Test on mobile**: Verify touch input and scaling still work
5. **Profile**: Check frame time and memory usage

### When Fixing Bugs
- Reproduce in both desktop and mobile views
- Check canvas coordinate transforms if position-related
- Verify collision detection geometry
- Use console logging sparingly; trace game state instead

### When Optimizing
- Profile before optimizing; identify actual bottlenecks
- Prioritize collision detection and rendering
- Consider entity pooling for frequently created objects
- Test frame rates on lower-end devices

## Development Workflow

1. **Create a branch**: `git checkout -b feature/description`
2. **Edit index.html**: Make changes to CSS or JavaScript
3. **Test locally**: Open index.html in a browser (dev tools for debugging)
4. **Commit with context**: Include what changed and why
5. **Push to branch**: `git push -u origin feature/description`

## Important Constraints

- **Single file only**: Do not split into multiple files
- **No external libraries**: Use only vanilla HTML/CSS/JavaScript
- **No build step**: File must work immediately when opened
- **No server required**: Game runs entirely in the browser
- **No external assets**: All graphics/sounds must be embedded or procedural
- **Responsive design**: Must work on mobile, tablet, and desktop

## Performance Targets

- **Frame rate**: Consistent 60 FPS on modern devices
- **Mobile FPS**: Minimum 30 FPS on mid-range devices
- **Load time**: < 2 seconds to playable state
- **Memory usage**: < 50MB during gameplay

## Testing Checklist

When making changes, verify:
- [ ] Game starts without errors
- [ ] Player movement works (keyboard and mouse)
- [ ] Enemies spawn and move correctly
- [ ] Collision detection triggers properly
- [ ] Game over condition works
- [ ] Score tracking updates
- [ ] Touch input works on mobile
- [ ] Responsive scaling works on different screen sizes
- [ ] No console errors or warnings

## References & Resources

- **Game Loop**: Standard fixed timestep pattern with update/render phases
- **Collision**: AABB collision with swept circle-rect tests for projectiles
- **Wave System**: Procedural enemy spawning with difficulty scaling
- **Boss Patterns**: Predefined attack sequences with state machine logic

## Attribution & Commit Standards

All commits should include:
```
<Brief description of changes>

<Detailed explanation if needed>

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
Claude-Session: <session-url>
```

## Questions or Unclear Patterns?

When in doubt:
1. Check existing entity implementation for patterns
2. Look at similar features already in the game
3. Prioritize simplicity and maintainability over cleverness
4. Test across mobile and desktop before considering complete

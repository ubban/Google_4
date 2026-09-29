# Claude.md - Sky Fighter Codebase Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5 Canvas and JavaScript. The game features player-controlled aircraft, enemy waves, weapon switching, smart bombs, and boss encounters.

## Repository Structure

```
Google_4/
├── index.html          # Single-file game implementation
├── README.md           # Project description
└── CLAUDE.md           # This file - AI guidance
```

## Implementation Details

### Architecture

The game is implemented as a single-file HTML5 application (`index.html`) containing:
- HTML structure with a canvas element and HUD display
- CSS styling for game container, canvas, and UI elements
- JavaScript game engine with core game loop

### Key Game Components

- **Player**: Controllable aircraft at bottom of screen
- **Enemies**: Spawned in waves with various behaviors
- **Weapons**: Multiple weapon types with switching capability
- **Smart Bombs**: Screen-clearing power-up
- **Bosses**: Stage bosses with health and attack patterns
- **HUD**: Real-time display of score, health, weapons, bombs

### Game Systems

1. **Rendering**: Canvas-based 2D graphics
2. **Input**: Keyboard and touch controls
3. **Collision Detection**: Rectangle-based AABB
4. **Entity Management**: Player, enemies, projectiles, particles
5. **Game State**: Menu, playing, pause, game-over, boss encounters

## Development Guidelines

### Code Style

- **Language**: Vanilla JavaScript (ES6+)
- **No external dependencies** - self-contained implementation
- **Comments**: Use for non-obvious logic, complex algorithms, game design choices
- **Naming**: Clear, descriptive names for game entities and methods
- **Constants**: Define game balance values at top of script for easy tuning

### Game Balance Variables

Look for configuration constants in the JavaScript section:
- Player movement speed and fire rate
- Enemy spawn rates and damage
- Weapon damage and firing patterns
- Boss health and attack intervals
- Score multipliers and difficulty scaling

### Performance Considerations

- Canvas rendering at 60 FPS
- Entity pooling for projectiles/explosions
- Collision checks optimized to relevant entity types only
- Touch event throttling for mobile performance

## How Claude Can Help

Claude can assist with:

1. **Bug Fixes**: Collision detection issues, input handling problems, state management bugs
2. **Feature Addition**: New weapons, enemy types, boss patterns, visual effects
3. **Optimization**: Performance improvements, code refactoring, memory management
4. **Mobile Support**: Touch controls, viewport handling, responsive design
5. **Game Balance**: Tuning difficulty, enemy patterns, scoring system
6. **Testing**: Creating test scenarios for specific game mechanics

## Testing Recommendations

- Test on multiple screen sizes (desktop, tablet, mobile)
- Verify touch controls on actual mobile devices
- Check framerate stability under heavy entity counts
- Test edge cases: screen boundaries, rapid weapon switching, boss patterns
- Validate scoring and difficulty progression

## Common Tasks

### Adding a New Weapon Type
1. Define weapon properties (damage, fire rate, projectile speed)
2. Implement fire method to create projectiles
3. Add to weapon switching UI
4. Test collision detection with enemies

### Adding a New Enemy Type
1. Define spawn behavior and movement pattern
2. Implement AI logic for attacking
3. Set health and damage values
4. Add to enemy spawn system

### Adjusting Game Difficulty
1. Modify spawn rates in enemy spawning logic
2. Adjust enemy health and damage values
3. Tune boss attack patterns and health
4. Test progression across different difficulty levels

## Performance Metrics

Monitor these for optimization opportunities:
- Canvas rendering time (target: <16ms per frame)
- Entity count during heavy combat
- Memory usage during long play sessions
- Touch event response time

## Browser Compatibility

- Modern browsers with HTML5 Canvas support
- Chrome, Firefox, Safari, Edge (latest versions)
- Mobile browsers: iOS Safari, Chrome Mobile
- Touch event support for mobile devices

## Attribution

When making commits for this project, include:
```
Co-Authored-By: Claude <noreply@anthropic.com>
```

## Notes for Future Development

- Consider WebGL migration if performance optimization is needed
- Modularization of game systems could improve maintainability
- Sound effects and music would enhance gameplay experience
- Progressive difficulty scaling based on player performance
- Leaderboard/score persistence for engagement

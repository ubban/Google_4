# Claude.md - Sky Fighter Game Development Guide

## Project Overview

**Sky Fighter** is a vertical shoot-em-up game inspired by Raiden, developed as an HTML5/JavaScript web game. The game features a player-controlled aircraft that shoots enemies while navigating obstacles and collecting power-ups.

## Architecture & Key Files

- **index.html** - Main game entry point with embedded HTML, CSS, and JavaScript
- **README.md** - Basic project metadata

### Code Organization (within index.html)

The game code is organized into logical sections:

1. **HTML Structure** - Canvas element and basic page layout
2. **CSS Styling** - Game styling, animations, and responsive design
3. **JavaScript Game Engine**:
   - Game state management
   - Entity classes (Player, Enemy, Bullet, PowerUp, etc.)
   - Collision detection system
   - Rendering loop
   - Input handling
   - Game physics

## Development Guidelines

### Code Style

- Use consistent naming conventions:
  - Classes: PascalCase (e.g., `PlayerShip`, `EnemyBullet`)
  - Functions/methods: camelCase (e.g., `updatePosition`, `checkCollision`)
  - Constants: UPPER_SNAKE_CASE (e.g., `CANVAS_WIDTH`, `PLAYER_SPEED`)

- Keep methods focused and single-purpose
- Use JSDoc comments for complex game logic
- Maintain proper separation of concerns between game systems

### Game Mechanics

- Physics: Movement, velocity, acceleration
- Combat: Shooting, collision detection, damage
- Progression: Waves of enemies, difficulty scaling
- Feedback: Score tracking, lives/health system

### Testing

- Test new game mechanics in isolation before integration
- Verify collision detection accuracy with edge cases
- Check frame rate performance with many entities on screen
- Test input responsiveness and edge case player controls

## Common Tasks

### Adding a New Enemy Type

1. Create a new class extending the base Enemy class
2. Define unique properties (speed, size, weapon type, sprite)
3. Implement movement pattern in `update()` method
4. Add spawning logic to the wave manager
5. Test collision and damage interactions

### Adding Power-ups

1. Create a PowerUp class with specific effects
2. Define spawn conditions and probability
3. Implement collision and activation logic
4. Add visual feedback when collected

### Adjusting Game Difficulty

1. Modify enemy spawn rate in wave configuration
2. Adjust enemy health/damage values
3. Change bullet speed and fire rate
4. Tune enemy movement patterns

### Performance Optimization

- Monitor entity count and consider object pooling for bullets/particles
- Profile rendering performance with browser DevTools
- Use requestAnimationFrame for smooth 60 FPS gameplay
- Consider spatial partitioning for collision detection if performance degrades

## Git Workflow

- Commit frequently with clear, descriptive messages
- Use feature branch: `claude/inspiring-planck-8rfcwi` (already configured)
- Test changes thoroughly before committing
- Write commit messages describing game mechanic or feature changes

## Deployment & Running

- Open `index.html` in a modern web browser
- Ensure JavaScript is enabled
- Test on different screen sizes for responsive behavior
- Verify game runs at consistent frame rate across browsers

## Known Limitations & Future Improvements

- Single canvas rendering (no layering system)
- Basic collision detection (could be optimized with spatial hashing)
- Limited audio support (can be enhanced)
- Mobile touch controls could be improved
- Save/load game state functionality can be added

## Code Review Priorities

When reviewing changes:
1. **Correctness** - Game logic behaves as intended
2. **Performance** - No unnecessary entity creation or calculations per frame
3. **Collision Detection** - Accurate and responsive
4. **User Experience** - Controls feel responsive, feedback is clear
5. **Code Quality** - Readable, maintainable, follows established patterns

## Contributors

- Claude Haiku 4.5 - AI Development Assistant

---

*Last Updated: 2026-09-26*

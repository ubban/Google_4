# Claude.md: Sky Fighter Development Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built with HTML5 Canvas. It features:
- Player aircraft with switchable weapons
- Wave-based enemy spawning system
- Boss encounters at stage end
- Smart bomb mechanic for emergency clearing
- Score multiplier system
- Health management with critical state effects

The entire game is self-contained in a single `index.html` file with embedded JavaScript and CSS.

## Project Structure

- **index.html**: Complete game implementation (HTML structure, CSS styling, and JavaScript logic)
  - Canvas rendering engine
  - Game state management
  - Input handling (keyboard, mouse, touch)
  - Particle effects and animations

## Development Approach

### Code Organization Principles

1. **Single-file simplicity**: The game is intentionally kept as a single HTML file for easy deployment and portability. Maintain this approach for new features.

2. **Modular logic sections**: Within the file, code is organized into logical sections:
   - Configuration constants (canvas size, physics, spawn rates)
   - Game state variables
   - Utility functions (collision, physics, rendering)
   - Game class and entity classes
   - Event listeners and main loop

3. **Canvas-based rendering**: All graphics are drawn directly to canvas. Use efficient rendering techniques:
   - Clear canvas each frame
   - Batch draws when possible
   - Use requestAnimationFrame for consistent timing

### Coding Standards

- Use modern ES6+ syntax (classes, arrow functions, template literals)
- Maintain clear variable naming (playerHealth, enemyBullets, etc.)
- Keep functions focused and under 50 lines when possible
- Add comments only for non-obvious logic or game mechanic explanations
- Use constants for magic numbers (speeds, sizes, spawn rates)

### Game Mechanics

- **Weapon System**: Player can cycle through weapons (standard, rapid, spread)
- **Scoring**: Base points per enemy type, multiplied by current multiplier
- **Health**: Player has 3 hearts; taking damage reduces health
- **Smart Bombs**: Limited use emergency clearing mechanic
- **Difficulty**: Increases with each stage/wave

### Testing & Validation

1. **Manual testing** (primary method):
   - Load the game in browser
   - Test all weapon types
   - Verify enemy patterns and boss behavior
   - Check collision detection accuracy
   - Validate score calculation and multiplier mechanics
   - Test mobile touch controls

2. **Before pushing**:
   - Verify game loads without console errors
   - Test on both desktop and mobile
   - Check that gameplay feels responsive and fair
   - Validate visual effects aren't too CPU-intensive

### Common Tasks

#### Adding a new enemy type
1. Add enemy configuration to the enemy spawn patterns
2. Define its properties (health, points, movement pattern)
3. Update collision damage values if needed
4. Test spawn rates and wave integration

#### Modifying weapon behavior
1. Update weapon configuration constants
2. Modify bullet spawning in weapon fire logic
3. Test firing pattern and damage application
4. Verify ammo count updates correctly

#### Tuning game difficulty
1. Adjust `SPAWN_RATE_INCREASE` and enemy health scaling
2. Modify wave progression timing
3. Tweak boss behavior patterns
4. Test difficulty curve feels natural

## Git Conventions

- Commit messages should be descriptive and action-oriented
- Keep commits focused on single features or fixes
- Include game mechanic details in commit messages when relevant
- Example: "Add smart bomb visual feedback and reduce cooldown"

## Known Constraints

- Single HTML file format limits code splitting (intentional)
- Canvas performance on very old devices may be limited
- Mobile controls work via touch; mouse fallback available
- Full-screen API may require user gesture on some browsers

## Performance Considerations

- Limit simultaneous entities to maintain 60 FPS
- Use efficient collision detection (grid or spatial partitioning if needed)
- Minimize canvas state changes
- Avoid frequent DOM manipulation outside canvas

## Future Enhancement Areas

- Local high score persistence (localStorage)
- Sound effects and music (Web Audio API)
- Additional enemy types and boss patterns
- Difficulty settings/modes
- Mobile app wrapper for native deployment

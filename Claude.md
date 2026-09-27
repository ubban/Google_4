# Claude.md - Sky Fighter Project Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The game features:
- Player aircraft with multiple weapon types
- Enemy waves and stage bosses
- Smart bomb mechanic
- Responsive controls (keyboard and touch)
- Score and combo system

## Architecture

This is a **single-file HTML application** (`index.html`) with:
- **HTML structure** (minimal): Canvas element + HUD container
- **CSS** (embedded): Game styling, responsive layout, HUD styling
- **JavaScript** (embedded): Complete game engine with:
  - Game state management
  - Entity system (player, enemies, bullets, effects)
  - Collision detection
  - Rendering loop
  - Input handling (keyboard + touch)
  - Game logic (waves, bosses, scoring)

## Key Development Guidelines

### Code Organization
- Keep all game code within the single HTML file
- Organize JS into logical sections:
  - Constants and configuration at the top
  - Utility functions
  - Entity classes (Player, Enemy, Bullet, etc.)
  - Game state and world management
  - Input handling
  - Rendering and animation
  - Main game loop

### Performance Considerations
- Canvas rendering is performance-critical
- Minimize DOM manipulation during gameplay
- Reuse object pools for bullets and effects when possible
- Keep collision detection efficient
- Test on mobile devices (primary target platform based on viewport settings)

### Browser Compatibility
- Target modern browsers (Chrome, Firefox, Safari, Edge)
- Use modern JS features (ES6+) as appropriate
- Test touch controls on mobile browsers
- Ensure responsive canvas sizing for different devices

### Game Mechanics

#### Player
- Single player aircraft
- Multiple weapon types (can be switched)
- Smart bomb with limited uses
- Health/shields system

#### Enemies
- Wave-based spawning
- Simple movement patterns
- Ranged attacks
- Boss enemies with special patterns

#### Scoring
- Points for destroying enemies
- Combo system for consecutive hits
- Power-ups and bonuses
- High score tracking (localStorage)

## Git Workflow

- Work on the feature branch: `claude/inspiring-planck-33vanm`
- Commit changes with clear, descriptive messages
- Push to the remote branch when complete
- Keep commits focused and logical

## Testing Guidelines

Before committing game changes:
1. Test gameplay mechanics in a modern browser
2. Verify touch controls work on mobile
3. Check canvas responsiveness at different viewport sizes
4. Ensure no JavaScript console errors
5. Test enemy waves and boss battles
6. Verify scoring and combo systems

## Performance Tips

- Use `requestAnimationFrame` for smooth 60 FPS gameplay
- Profile with browser DevTools if performance degrades
- Minimize function calls in the game loop
- Use efficient collision detection (spatial hashing if needed for many entities)
- Optimize canvas drawing order

## UI/UX Considerations

- HUD should be clearly readable on mobile and desktop
- Game controls must be intuitive for touch and keyboard
- Provide clear feedback for player actions (hits, score, etc.)
- Pause/resume functionality should be prominent
- Game over/level complete screens should be clear

## File Maintenance

- Keep the single `index.html` well-organized with clear section comments
- Maintain proper indentation and formatting
- Include inline comments for complex game logic
- Version information should be updated in the HTML structure if needed

## Common Tasks

### Adding a New Enemy Type
1. Create an enemy class in the JavaScript section
2. Implement `update()` and `draw()` methods
3. Add spawn logic to the wave management
4. Test collision detection with player and bullets

### Implementing a New Weapon
1. Define weapon stats in the constants
2. Modify bullet spawning logic
3. Update player weapon switching
4. Test firing patterns and damage

### Adjusting Game Balance
1. Modify constants (damage, enemy health, spawn rates, etc.)
2. Test different difficulty levels
3. Verify scoring feels appropriate

## Debugging

- Use browser DevTools Console for logs
- Check the Network tab for any missing resources
- Use Chrome DevTools Performance tab for FPS/timing issues
- Test in multiple browsers for compatibility

## Notes for Claude Code

- This is a single-file application, so all changes go to `index.html`
- Changes are self-contained and easy to version
- Game logic can be tested by running the HTML in a browser
- Be mindful of the 60 FPS game loop when making changes
- Always consider both desktop and mobile user experiences

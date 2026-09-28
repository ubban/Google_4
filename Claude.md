# Claude.md - Sky Fighter Codebase Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla JavaScript and HTML5 Canvas. The game features weapon switching, smart bombs, and stage bosses.

### Project Files

- **index.html** - Single-file game implementation containing HTML, CSS, and JavaScript
  - Game canvas rendering
  - Game state management
  - Input handling (keyboard, mouse, touch)
  - Entity management (player, enemies, projectiles, bosses)
  - HUD display and UI
  - Audio system

- **README.md** - Project metadata
- **.git/** - Version control

## Development Guidelines

### Architecture

The entire game is implemented in a single HTML file with inline CSS and JavaScript. This approach:
- Simplifies deployment (single file)
- Makes it easy to test locally
- Reduces build complexity
- Works well for game prototyping

### Game Loop

The game uses `requestAnimationFrame` for the main loop. Key game states:
- **Menu/Intro** - Start screen
- **Playing** - Active gameplay
- **Boss Battle** - Special boss encounter state
- **Game Over** - End screen
- **Victory** - Level completion screen

### Code Style

- Use vanilla JavaScript (no external libraries)
- Keep game logic self-contained within the single file
- Use clear variable names for game entities and states
- Comment complex game mechanics and collision logic

### Performance Considerations

- Canvas is optimized for 2D rendering - avoid layout thrashing
- Batch DOM updates when modifying HUD elements
- Use appropriate frame-rate independent movement calculations
- Monitor performance on lower-end devices (mobile especially)

### Testing Changes

1. Open `index.html` directly in a browser
2. Test core gameplay:
   - Player movement and controls (keyboard, mouse, touch)
   - Weapon firing and switching
   - Enemy spawning and behavior
   - Collision detection
   - Score/life tracking
3. Test on multiple devices:
   - Desktop (different screen sizes)
   - Mobile (touch controls)
   - Various browsers (Chrome, Firefox, Safari, Edge)

### Common Tasks

#### Adding a New Enemy Type

1. Define enemy properties in the initialization section
2. Add spawn logic to the enemy spawner
3. Implement behavior in the game loop update
4. Add collision handling if needed
5. Test spawning at various difficulties

#### Modifying Game Balance

1. Look for numeric constants (enemy health, damage, spawn rates)
2. Make incremental changes and test
3. Ensure difficulty progression feels smooth
4. Consider impact on different device performance

#### UI/HUD Changes

1. Modify CSS in the `<style>` section
2. Update HUD elements via JavaScript DOM manipulation
3. Test responsive behavior across screen sizes
4. Ensure readability on various displays

### Debugging Tips

- Open browser DevTools (F12) to access the console
- Use `console.log()` for debugging game state
- The performance tab can help identify frame rate issues
- Breakpoints can be set on specific game logic
- Check mobile DevTools for touch input issues

### Commit Messages

Use clear, descriptive commit messages:
- `feat: add new boss type` for new features
- `fix: correct collision detection` for bug fixes
- `improve: optimize particle system` for improvements
- `refactor: reorganize game state management` for refactoring

## Contributing

When working on this codebase:

1. Test changes thoroughly on the intended platform (desktop/mobile)
2. Maintain the single-file structure for simplicity
3. Keep code readable and well-organized within the file
4. Document significant changes or complex logic
5. Ensure game balance remains fun and challenging
6. Consider accessibility and input method support

## Resources

- [MDN Canvas API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)
- [requestAnimationFrame Documentation](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame)
- [Touch Events API](https://developer.mozilla.org/en-US/docs/Web/API/Touch_events)

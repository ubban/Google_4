# Claude Development Guidelines for Google_4

## Project Overview

**Google_4** is an interactive web-based game project featuring "Sky Fighter," a Raiden-style vertical shoot-em-up game built with HTML5 Canvas and JavaScript.

## Codebase Structure

- **index.html**: Main entry point containing the game engine, game logic, and UI
- **README.md**: Project metadata and description

## Development Principles

### Code Organization
- The primary game code is contained in `index.html` with embedded HTML, CSS, and JavaScript
- Keep the code modular within the single-file structure where possible
- Use descriptive variable and function names

### Game Architecture
- Game state is managed through object properties
- Canvas rendering drives the visual updates
- Input handling (keyboard, touch) drives game interactions
- Collision detection uses simple rectangular bounds checking

### Testing & Validation
- Manual testing in the browser is the primary verification method
- Test core gameplay loops: player movement, shooting, enemy spawning, collision detection
- Verify touch/keyboard controls work properly
- Test on different screen resolutions

## Development Workflow

### Before Making Changes
1. Review the existing game code structure and understand affected systems
2. Run the game in a browser to understand current behavior
3. Plan changes to minimize impact on working features

### Making Changes
1. Keep changes focused and minimal
2. Preserve existing functionality unless specifically changing it
3. Test gameplay after any modifications
4. Ensure visual rendering works correctly

### Committing Changes
- Write clear, descriptive commit messages
- Reference what was added, fixed, or changed
- Keep commits focused on single features or bug fixes
- Include attribution footer as specified in the session

## Key Game Systems

### Rendering
- Canvas 2D context used for all rendering
- Game loop handles continuous updates and draws
- Screen clearing and redrawing each frame

### Input Handling
- Keyboard controls for player movement and actions
- Touch/mouse support for mobile compatibility

### Game Objects
- Player entity with position, health, and weapon state
- Enemies with movement patterns and health
- Bullets for both player and enemies
- Collision detection between all entities

### Game State
- Score tracking
- Lives/health management
- Wave/level progression
- Game over and pause states

## Performance Considerations
- Keep the game loop efficient
- Minimize redraws of unchanged elements where possible
- Use requestAnimationFrame for smooth animations
- Monitor performance on lower-end devices

## Future Enhancement Areas
- Sound effects and background music
- Particle effects for explosions
- Additional enemy types and patterns
- Power-ups and special weapons
- Persistent high score tracking
- Mobile optimization

## Notes for Claude Code Agent
- This is an entertainment/game project - focus on gameplay experience
- Visual feedback is critical to the user experience
- Test changes in a real browser environment before committing
- Keep the codebase maintainable despite being in a single HTML file
- Consider accessibility where appropriate (keyboard controls, screen reader hints if applicable)

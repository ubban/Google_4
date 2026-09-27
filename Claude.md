# Claude.md - Sky Fighter Development Guide

## Project Overview

Sky Fighter is a Raiden-style vertical shoot-'em-up game built with HTML5 Canvas and vanilla JavaScript. The game features a player aircraft that can switch weapons, drop smart bombs, and fight through stage bosses.

## Project Structure

- `index.html` - Single-file game implementation containing all HTML, CSS, and JavaScript
  - Canvas rendering for game graphics
  - Game loop and update/render cycle
  - Input handling (keyboard, mouse, touch)
  - Game state management
  - Enemy and weapon systems
  - UI/HUD rendering

## How to Run

1. Open `index.html` directly in a modern web browser (Chrome, Firefox, Safari, Edge)
2. The game runs in fullscreen canvas mode with responsive sizing
3. No build process or server required - it's a standalone HTML file

## Development Guidelines

### Code Organization

The entire game logic is contained in a single `index.html` file with three main sections:
1. **HTML Structure** - Minimal DOM (canvas container and HUD)
2. **CSS Styling** - Game canvas styling, responsive design, dark theme
3. **JavaScript** - Core game engine, game loop, entity management

### Game Mechanics to Understand

- **Player Movement**: Arrow keys or mouse/touch control
- **Weapons**: Multiple weapon types that can be switched
- **Bombs**: Smart bomb mechanic with special effects
- **Enemies**: Various enemy types and formations
- **Bosses**: Stage bosses with unique attack patterns
- **Score/Lives**: HUD displays score, lives, and bomb count

### When Making Changes

1. **Test in Browser**: Always test changes by opening index.html in a web browser
2. **Responsive Design**: Maintain compatibility with mobile (touch), tablet, and desktop screens
3. **Performance**: Canvas rendering is performance-critical; avoid expensive operations in the game loop
4. **Game Balance**: Adjust difficulty through enemy spawn rates, health values, and damage amounts
5. **Visual Polish**: Use canvas drawing operations for all rendering to maintain consistent look

### Important Conventions

- Use `requestAnimationFrame` for the game loop (already implemented)
- Maintain game state in a single global object or closure
- Use canvas context methods for all rendering (no DOM manipulation during gameplay)
- Keep frame rate consistent and smooth for gameplay experience
- Test on both desktop and mobile devices when possible

## Common Tasks

### Adding a New Enemy Type
1. Define enemy properties (health, speed, attack pattern)
2. Add spawn logic in the enemy spawn system
3. Define drawing/rendering code in the canvas render function
4. Test collision and damage interactions

### Adjusting Game Difficulty
1. Modify enemy spawn rates and quantities
2. Adjust enemy health values
3. Change weapon damage or fire rates
4. Modify boss attack patterns and health

### Improving Visual Effects
1. Add particle effects or animations
2. Enhance explosion or impact visuals
3. Improve HUD display formatting
4. Add visual feedback for player actions

## Notes for Claude Code

- This is a single-file game - all changes go in `index.html`
- No external dependencies - pure HTML5/JavaScript
- Game loop runs at 60 FPS (or browser refresh rate)
- Canvas API is used for all rendering
- Mobile-responsive design with touch support
- For bug fixes: always test in a browser to verify the fix works
- For features: ensure the gameplay remains balanced and fun
- Commit messages should describe game changes clearly (e.g., "Add laser weapon", "Fix enemy spawn rate")

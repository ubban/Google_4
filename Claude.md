# Claude Code Guide for Sky Fighter

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5, CSS3, and JavaScript. The game runs as a single `index.html` file with embedded styles and scripts.

### Key Features
- Vertical scrolling shoot-'em-up gameplay
- Player ship with directional movement
- Weapon system with switching mechanics
- Smart bombs for crowd control
- Stage bosses with unique patterns
- Mobile and desktop support with responsive controls
- Touch and keyboard input handling

## Running the Game

### Local Development
```bash
# Simply open the index.html file in a web browser
# Option 1: Direct file open
open index.html

# Option 2: Using a local HTTP server (recommended for consistency)
python3 -m http.server 8000
# Then visit http://localhost:8000
```

### Browser Requirements
- Modern browsers with Canvas API support
- Hardware acceleration enabled for optimal performance
- Touch support for mobile devices
- Viewport meta tags configured for responsive display

## Project Structure

The entire game is contained in a single `index.html` file with three main sections:

1. **HTML Structure**
   - Container div for game canvas and HUD
   - Fixed viewport with safe area support for mobile
   - No external dependencies

2. **CSS Styling**
   - Dark theme with blue gradient background
   - Responsive sizing using viewport units and clamp()
   - Safe area inset support for notched devices
   - Canvas shadow effects and styling

3. **JavaScript Game Engine**
   - Main game loop with requestAnimationFrame
   - Player class for ship control and state
   - Weapon system with multiple types
   - Bullet/projectile management
   - Enemy spawn system and AI patterns
   - Boss encounters
   - Collision detection
   - Score and UI updates

## Development Guidelines

### When Making Changes

1. **Structure**: Keep all code within the single HTML file. Do not split into separate files unless fundamentally refactoring the architecture.

2. **Performance**: 
   - Monitor frame rate (target 60 FPS)
   - Minimize redraws and DOM manipulation
   - Canvas operations should be efficient
   - Avoid creating objects in the render loop

3. **Compatibility**:
   - Test on mobile devices and different screen sizes
   - Ensure touch controls work alongside keyboard input
   - Test on Chrome, Firefox, Safari, and Edge

4. **Code Style**:
   - Use clear variable names that indicate purpose
   - Comment complex game logic and algorithms
   - Keep functions focused and single-purpose
   - Avoid deep nesting in game loops

5. **Testing**:
   - Test on actual devices when possible
   - Verify both touch and keyboard input paths
   - Check canvas rendering at different resolutions
   - Ensure smooth performance on lower-end devices

### Key Game Systems to Understand

- **Game Loop**: Uses `requestAnimationFrame` for smooth 60 FPS updates
- **Input Handling**: Both keyboard (`keydown`/`keyup`) and touch events (`touchstart`/`touchmove`/`touchend`)
- **Canvas Rendering**: All graphics drawn to a 2D canvas context
- **Collision Detection**: Box collision or point-in-rect checks
- **State Management**: Player state, enemies, projectiles, score tracked during game loop

## Files in This Repository

```
Google_4/
├── index.html        # Complete game with HTML, CSS, and JavaScript
├── README.md         # Project description
└── Claude.md         # This file - Claude Code guidance
```

## Tips for Claude Code

1. **No External Dependencies**: The game doesn't use npm or build tools. It's pure vanilla JavaScript.

2. **Testing Changes**: Use the `/run` skill or open the file in a browser to test changes immediately.

3. **Debugging**:
   - Use browser DevTools console for debugging
   - Check the browser's Performance tab for frame rate
   - Mobile debugging via remote device inspection

4. **Game Balance**:
   - When adjusting difficulty, test multiple playthroughs
   - Enemy spawn rates and patterns significantly impact gameplay
   - Weapon balance affects player experience

5. **Mobile Optimization**:
   - Test on various screen sizes and orientations
   - Ensure touch controls feel responsive
   - Check safe area inset support

## Common Tasks

### Adding a New Weapon Type
1. Add weapon data to the weapon system
2. Update the weapon switching logic
3. Add rendering code for the new weapon bullets
4. Test fire rate and damage balance

### Adding a New Enemy Type
1. Create enemy object with position, health, and sprite data
2. Implement movement and attack patterns in the update loop
3. Add collision and rendering code
4. Integrate into enemy spawn system

### Adjusting Difficulty
1. Modify enemy spawn rates and patterns
2. Adjust enemy health/damage
3. Change bullet patterns for bosses
4. Test progression across levels

### UI/HUD Updates
1. Modify the `#hud` element styling in CSS
2. Update HUD text rendering in the game loop
3. Test at different viewport sizes

## Resources

- Canvas API: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API
- Touch Events: https://developer.mozilla.org/en-US/docs/Web/API/Touch_events
- Game Programming Patterns: https://gameprogrammingpatterns.com/

## Session Attribution

Work on this project is tracked in Claude Code sessions. Session info and commits include attribution to Claude models and session URLs.

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>

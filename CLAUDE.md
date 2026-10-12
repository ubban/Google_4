# Google_4 - Sky Fighter

## Project Overview

Sky Fighter is a **Raiden-style vertical shoot-'em-up game** implemented as a single-file HTML5 Canvas game. Players control an aircraft to switch weapons, drop smart bombs, and defeat stage bosses.

### Technology Stack
- **Frontend**: Vanilla HTML5 Canvas + JavaScript
- **Architecture**: Single-file application (index.html contains all code)
- **Target Platform**: Web browser (desktop and mobile with touch support)
- **Responsive Design**: Mobile-optimized with viewport scaling and safe-area support

## Project Structure

```
Google_4/
├── index.html          # Main game implementation (complete)
├── README.md           # Project description
└── CLAUDE.md          # This file - development documentation
```

## Running the Project

### Local Development
1. Open `index.html` directly in a web browser
2. The game should load with the Sky Fighter title screen
3. Use keyboard or mouse controls:
   - **Mouse/Touch**: Crosshair aiming and movement
   - **Keyboard**: Arrow keys or WASD for movement
   - **Space**: Fire weapon
   - **Other**: Bomb/special actions as configured

### Browser Compatibility
- Modern browsers with Canvas support (Chrome, Firefox, Safari, Edge)
- Mobile browsers with touch support enabled
- Dark color scheme optimized for the game aesthetic

## Game Features

### Visual Elements
- Dark blue gradient background (#05122b to #7fc1ff)
- Player aircraft sprite
- Enemy formations and bosses
- Weapon effects and explosions
- HUD display for score, lives, bombs

### Game Mechanics
- Multiple weapon types with switching capability
- Smart bomb system
- Boss encounters
- Score tracking
- Lives/health system

## Development Guidelines

### Code Style
- Single-file HTML/CSS/JavaScript structure
- Canvas API for rendering
- Event-driven input handling
- Mobile-first responsive design

### Key Considerations
- **Performance**: Optimize Canvas rendering and collision detection
- **Mobile Compatibility**: Maintain touch controls and viewport settings
- **Accessibility**: Ensure crosshair cursor and color contrast
- **Testing**: Manual testing in multiple browsers (desktop and mobile)

## Testing Checklist

### Functionality
- [ ] Game starts and displays main screen
- [ ] Player aircraft moves correctly with controls
- [ ] Weapons fire and function as expected
- [ ] Bombs detonate and clear enemies
- [ ] Collision detection works properly
- [ ] Bosses spawn and behave correctly
- [ ] Score increments on enemy defeat
- [ ] Game over screen displays

### Browser Testing
- [ ] Chrome/Edge (desktop)
- [ ] Firefox (desktop)
- [ ] Safari (desktop)
- [ ] Chrome/Safari (mobile)
- [ ] Touch controls work smoothly

### Mobile Testing
- [ ] Viewport scaling works correctly
- [ ] Safe area insets respected (notch devices)
- [ ] Touch input doesn't scroll page
- [ ] HUD text remains readable
- [ ] Canvas adapts to device dimensions

## Known Limitations
- Single HTML file (no build process required)
- No persistent storage/save system currently
- No online multiplayer

## Future Enhancement Ideas
- Level progression system
- Power-up system
- Sound effects and music
- Leaderboard/high scores
- Additional weapon types
- Enemy AI improvements
- Visual effects (particles, animations)

## Dependencies
- None - pure vanilla HTML5/CSS/JavaScript

## Attribution
- Project: Sky Fighter (Google_4)
- Game Type: Vertical Shoot-'em-up
- Last Updated: 2026-10-12

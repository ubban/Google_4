# Sky Fighter - Claude.md

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5 Canvas and JavaScript. It features weapon switching, smart bombs, stage bosses, and responsive controls for both desktop and mobile devices.

**Repository**: `ubban/google_4`
**Language**: JavaScript (vanilla, no frameworks)
**Build**: Single HTML file (`index.html`)

## Architecture

### Core Game Structure

The game is implemented as a single-file HTML5 application with embedded JavaScript and CSS. The architecture consists of:

- **Game Loop**: Canvas-based rendering using requestAnimationFrame
- **Entity System**: Player, enemies, projectiles, and visual effects
- **Input Handling**: Keyboard (desktop) and touch/gyro (mobile)
- **Collision Detection**: Simple AABB collision system
- **UI Layer**: HUD overlay for stats, stage info, and mobile controls

### Key Components

1. **Player Ship**: 
   - Multiple weapon types (bullet spray, wave, laser)
   - Smart bomb mechanic
   - Health and damage system
   - Responsive movement with boundaries

2. **Enemy System**:
   - Wave-based spawning
   - Stage bosses with unique patterns
   - Projectile attacks
   - Destruction visual effects

3. **Weapons & Items**:
   - Weapon switching via E-key or mobile buttons
   - Bomb pickups for smart bomb refills
   - Weapon upgrades and special effects

4. **Mobile Support**:
   - Touch controls with on-screen buttons
   - Gyroscope-based aiming (if available)
   - Responsive layout with safe-area insets
   - Portrait and landscape orientation support

5. **Visual & Audio**:
   - Canvas-based particle effects
   - Gradient backgrounds
   - Text-based UI with text shadows
   - Game state transitions (menu, playing, game over)

## File Structure

```
Google_4/
├── index.html          # Single-file game application
├── README.md           # Project readme
└── CLAUDE.md           # This file - Claude development guide
```

## Development Workflow

### Running the Game

Open `index.html` in a web browser or use Claude Code's built-in server:
```bash
# The game runs standalone in any modern browser
# For development testing: use a local HTTP server if needed
```

### Making Changes

1. **Edit Code**: All game logic is embedded in `index.html` within `<script>` tags
2. **Test Changes**: Reload the browser to test your changes immediately
3. **Mobile Testing**: Test on actual devices or use browser dev tools for mobile emulation

### Code Organization within index.html

The embedded JavaScript is organized into sections:

- **Initialization**: Canvas setup, event listeners
- **Game State**: Current game variables and constants
- **Entity Classes/Functions**: Player, enemies, projectiles
- **Rendering**: Canvas drawing functions
- **Input Handling**: Keyboard and touch event handlers
- **Game Loop**: Main update and render cycle
- **Collision Detection**: Entity interaction logic
- **UI Rendering**: HUD and overlay rendering

## Common Development Tasks

### Adding a New Enemy Type

1. Create a new enemy class/constructor function
2. Define its `update()` and `draw()` methods
3. Add spawn logic in the wave management system
4. Define projectile patterns in the update method

### Adding a New Weapon

1. Define weapon properties (fire rate, damage, spread pattern)
2. Implement projectile spawn logic
3. Add weapon cycling to input handlers
4. Update HUD display to show new weapon

### Adjusting Game Balance

- **Enemy Health**: Modify `health` properties in enemy constructors
- **Damage Values**: Adjust projectile damage constants
- **Fire Rates**: Modify `fireRate` and `fireCounter` logic
- **Spawn Patterns**: Adjust wave definitions and spawn intervals
- **Player Movement Speed**: Tweak velocity constants

### Visual Tweaks

- **Colors**: Search for hex color codes (e.g., `#1976d2`, `#ff6b00`)
- **Particle Effects**: Modify particle count and lifespan in effect functions
- **Canvas Dimensions**: Check viewport meta tag and canvas sizing logic
- **UI Positioning**: Adjust HUD element positioning via CSS flexbox properties

## Testing Checklist

Before pushing changes:

- [ ] Game initializes without errors
- [ ] Player movement works smoothly
- [ ] Weapon switching functions correctly
- [ ] Enemies spawn and behave as intended
- [ ] Collision detection works properly
- [ ] Game over and win conditions trigger
- [ ] Mobile controls responsive on touch devices
- [ ] No console errors
- [ ] Performance acceptable (60 FPS target)

## Performance Considerations

- Canvas rendering is the main performance bottleneck
- Limit active entities to maintain 60 FPS
- Use object pooling for frequently created/destroyed entities if needed
- Minimize DOM manipulation (mostly not an issue since we use Canvas)
- Profile with browser DevTools if performance degrades

## Browser Compatibility

- Target: Modern browsers (Chrome, Firefox, Safari, Edge)
- Requires: HTML5 Canvas, ES6+ JavaScript support
- Mobile: iOS Safari, Chrome Mobile, Firefox Mobile

## Git Workflow

When working on features:

1. Create feature branch: `git checkout -b feature/description`
2. Make changes to `index.html`
3. Test thoroughly
4. Commit with clear message: `git commit -m "feat: add new feature"`
5. Push to branch: `git push -u origin feature/description`

## Notes for Claude Code

- The entire game logic is in one file, so changes are straightforward
- Testing requires opening the HTML in a browser
- No build step or compilation needed
- Changes are immediately visible on reload
- Keep the file well-commented for future maintainability

# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with HTML5 Canvas. It's a fast-paced action game where players control a fighter aircraft, switch weapons, drop smart bombs, and battle stage bosses.

- **Repository**: https://github.com/ubban/Google_4
- **Type**: Single-file HTML5 game (self-contained)
- **Platform**: Web browser (desktop and mobile)

## Project Structure

```
.
├── index.html          # Complete game implementation (HTML, CSS, JavaScript)
├── README.md           # Project metadata
└── CLAUDE.md          # This file
```

The entire game is implemented in a single `index.html` file with embedded CSS and JavaScript.

## Key Features

- **Vertical shoot-'em-up gameplay** - Classic Raiden-style combat
- **Multiple weapons** - Switch between different attack types
- **Smart bombs** - Special devastating attacks
- **Stage bosses** - Boss battles as level climaxes
- **Mobile responsive** - Full touch and mouse support
- **Dark theme** - Modern UI with dark blue aesthetic
- **Canvas rendering** - Smooth performance with HTML5 Canvas API

## Technical Stack

- **HTML5** - Document structure and Canvas API
- **CSS3** - Styling with responsive units (vmin, clamp)
- **Vanilla JavaScript** - No dependencies; pure game logic
- **Canvas API** - 2D graphics rendering
- **Touch & Mouse Events** - Input handling for multiple devices

## Code Organization (in index.html)

The single file contains:

1. **HTML Structure** (lines 1-100+)
   - Container div for canvas
   - HUD (Heads-Up Display) for game stats
   - Game canvas element
   - Modal for menus/dialogs

2. **CSS Styling** (lines 10-200+)
   - Root color scheme and dark theme
   - Layout with fixed positioning
   - HUD styling with flex layout
   - Canvas styling with shadows and responsive sizing
   - Modal and button styles
   - Responsive font sizing with clamp()

3. **JavaScript Game Engine** (lines 200+)
   - Game state management
   - Player character logic
   - Enemy and projectile systems
   - Collision detection
   - Weapon system
   - Level/stage management
   - Input handling (keyboard, mouse, touch)
   - Rendering loop

## Development Guidelines

### Code Style
- Use vanilla JavaScript (no frameworks)
- Keep game logic separate from rendering
- Use descriptive variable and function names
- Prefer const/let over var

### Testing
- Test gameplay on multiple browsers (Chrome, Firefox, Safari, Edge)
- Test on mobile devices (iOS Safari, Chrome)
- Verify touch controls work smoothly
- Check performance on lower-end devices

### Performance Considerations
- Canvas rendering is the performance bottleneck
- Avoid excessive DOM manipulation during gameplay
- Use requestAnimationFrame for smooth 60 FPS
- Optimize collision detection algorithms
- Cache CSS calculations (use clamp() instead of JS-based sizing)

### Mobile Considerations
- Touch-action: none for preventing browser gestures
- Viewport fit: cover for notched devices
- Safe-area-inset support for notches and safe areas
- Crosshair cursor for pointer feedback
- Max-width/max-height to fit various screen sizes

## Gameplay Mechanics

### Core Loop
1. Handle input (keyboard, mouse, touch)
2. Update game state (position, velocity, timers)
3. Check collisions (bullets vs enemies, enemies vs player)
4. Render (clear canvas, draw sprites, update HUD)
5. Repeat at 60 FPS

### Player
- Centered at bottom of screen
- Controlled by WASD, arrow keys, or touch drag
- Has health/lives system
- Can switch weapons (typically E key or on-screen button)
- Can fire bombs (typically Space or on-screen button)

### Enemies
- Spawn in waves or patterns
- Have AI for movement and attack
- Drop rewards when defeated
- Stage bosses are special enemy types

### Weapons
- Multiple weapon types available
- Switch between them during gameplay
- Each has different fire rate, spread, and damage
- Ammo or energy system may apply

### Smart Bombs
- Powerful screen-clear attack
- Limited uses per level
- Activated manually

## Building and Running

1. **No build process needed** - Direct browser loading
2. Open `index.html` in a web browser
3. Use a local web server for best results:
   ```bash
   python3 -m http.server 8000
   # Then visit http://localhost:8000
   ```

## Git Workflow

- **Main branch**: Stable game version
- **Feature branches**: Named `claude/<descriptive-name>`
- **Commit messages**: Describe gameplay/feature changes clearly
- **Testing**: Always test gameplay before committing

## Common Tasks for Claude Code

### Adding New Features
1. Identify where logic fits (player, enemy, weapon, collision systems)
2. Update game state/variables
3. Modify update loop if needed
4. Update rendering code for visual changes
5. Test thoroughly on multiple devices
6. Commit with clear message

### Fixing Bugs
1. Identify in-game symptom and reproduction steps
2. Trace through game logic and collision detection
3. Fix in the relevant system
4. Test fix across different scenarios
5. Commit with "Fix:" prefix

### Performance Optimization
1. Profile game performance (browser DevTools)
2. Identify bottleneck (usually rendering or collision)
3. Optimize algorithm or reduce draw calls
4. Verify FPS remains at 60
5. Commit with explanation of optimization

### UI/UX Changes
1. Modify CSS for styling changes
2. Update HUD rendering code for stat display changes
3. Adjust canvas sizing with responsive values
4. Test on multiple screen sizes
5. Commit with "UI:" prefix

## Browser Compatibility

- **Target**: All modern browsers (Chrome, Firefox, Safari, Edge)
- **Minimum**: Canvas API support + ES6 JavaScript
- **Mobile**: iOS Safari 12+, Chrome 60+
- **Performance**: 60 FPS on mid-range devices

## Resources & References

- [Canvas API MDN](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)
- [Touch Events](https://developer.mozilla.org/en-US/docs/Web/API/Touch_events)
- [Raiden (arcade game)](https://en.wikipedia.org/wiki/Raiden_(arcade_game))
- [Game Programming Patterns](https://gameprogrammingpatterns.com/)

## Contact & Attribution

- **Repository Owner**: ubban
- **Repository**: https://github.com/ubban/Google_4
- **Email**: bobwu@irobotx.top (project contact)

---

Last updated: 2026-09-26

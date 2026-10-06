# Claude.md — Sky Fighter Game Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application using Canvas for rendering and vanilla JavaScript for game logic.

### Key Characteristics
- **Language**: HTML5 + CSS + JavaScript (vanilla, no frameworks)
- **Single File Architecture**: All code contained in `index.html` (1669 lines)
- **Game Type**: Vertical bullet-hell shooter
- **Target Platform**: Web browsers (desktop and mobile)
- **Rendering**: HTML5 Canvas 2D context
- **Responsive Design**: Scales to viewport using CSS and viewport meta tags

## Project Structure

```
Google_4/
├── index.html      # Main game file (all HTML, CSS, and JS)
├── README.md       # Minimal project readme
└── .git/           # Git repository
```

## Running the Project

### Development/Testing
1. Open `index.html` in a modern web browser (Chrome, Firefox, Safari, Edge)
2. The game runs client-side with no dependencies or build process required
3. Use browser DevTools (F12) to debug and inspect code

### Production
- Simply host `index.html` on any web server
- Works offline as a standalone HTML file
- Mobile-friendly with touch controls support

## Game Architecture

### Main Components (Classes)
The codebase uses a class-based object-oriented design:

- **Game Controller**: Main game loop and state management
- **Player**: Player ship with movement, weapons, and collision handling
- **Enemy**: Enemy patterns and AI behavior
- **Projectile**: Bullet and bomb rendering and movement
- **Effect**: Visual effects (explosions, impact)
- **Particle**: Particle system for effects

### Game Loop
- Runs at 60 FPS using `requestAnimationFrame`
- Input handling (keyboard/touch)
- Update logic (positions, collisions, state)
- Rendering to canvas

### Control Scheme
- **Desktop**: Arrow keys or WASD for movement, Space for primary fire, Shift for secondary fire
- **Mobile**: Touch controls for movement (if implemented)

## Development Guidelines

### Code Style
- **JavaScript**: ES6+ classes and modern syntax
- **Naming**: camelCase for variables/functions, PascalCase for classes
- **Comments**: Minimal but present for complex game logic
- **Indentation**: 2 spaces

### Key Patterns
1. **Classes**: Each major entity is a class (Player, Enemy, Projectile, etc.)
2. **Canvas Rendering**: Direct 2D context drawing
3. **Input Handling**: Event listeners for keyboard and touch
4. **Collision Detection**: Likely bounding box or circle-based
5. **Game State**: Central controller manages overall game state

### Common Tasks

#### Adjusting Game Parameters
Game constants (speeds, sizes, colors) are typically defined near the top of their respective classes or in the Game controller. Look for:
- Velocity/speed values
- Sprite dimensions
- Color constants (hex values)
- Spawn rates

#### Adding New Enemy Types
1. Locate the Enemy class definition
2. Create subclass or add variant in constructor
3. Implement custom behavior in update/render methods
4. Add spawning logic to game controller

#### Modifying Weapons
1. Find Player class and weapon properties
2. Update projectile spawn logic
3. Adjust damage, rate of fire, or projectile speed
4. Add new weapon types as needed

#### Optimizing Performance
- Reduce particle count or effects quality
- Optimize collision detection (use spatial partitioning if needed)
- Profile with Chrome DevTools Performance tab
- Check for memory leaks in long play sessions

## File Editing Workflow

### Viewing Code
- Use browser DevTools Elements tab to inspect HTML structure
- Use DevTools Sources tab to view and debug JavaScript
- Or edit directly in your editor (VS Code recommended)

### Testing Changes
1. Edit `index.html`
2. Save the file
3. Refresh browser tab (F5 or Cmd+R)
4. Test gameplay to verify changes

### Debugging
- `console.log()` statements visible in DevTools Console
- Set breakpoints in DevTools Sources tab
- Check Network tab for any asset loading issues (though none expected)

## Browser Compatibility

**Supported**: Modern browsers with ES6+ and Canvas support
- Chrome/Chromium 60+
- Firefox 55+
- Safari 11+
- Edge 15+

**Mobile Support**: iOS Safari, Android Chrome with touch event handling

## Git Workflow

- Work on designated branch: `claude/inspiring-planck-d19swx`
- Commit changes with descriptive messages (game mechanic changes, feature additions)
- Push to remote after testing

## Common Issues & Solutions

### Canvas Not Rendering
- Check browser console for errors
- Verify DevicePixelRatio handling for high-DPI displays
- Ensure canvas element exists in DOM

### Touch Controls Not Working
- Verify touch event listeners are attached to canvas
- Check touch event prevention (preventDefault) if needed

### Performance Issues
- Profile with DevTools Performance tab
- Check for infinite loops or excessive re-renders
- Consider entity pooling for frequently created/destroyed objects

### Memory Leaks
- Ensure event listeners are removed when entities are destroyed
- Check for circular references in class instances
- Use DevTools Memory profiler for heap snapshots

## Future Enhancements
Consider potential improvements:
- Persistent high score storage (localStorage)
- Sound/music support (Web Audio API)
- Progressive Web App (PWA) features
- Mobile app wrapper (Cordova/React Native)
- Multiplayer support (WebSockets)
- Level editor or data-driven level system

## Contact & Attribution
- Repository: `ubban/google_4`
- Last Updated: 2026-10-06

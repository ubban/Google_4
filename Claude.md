# Sky Fighter - Development Guide for Claude Code

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in HTML5 with vanilla JavaScript. The game features:
- Player-controlled aircraft with weapon selection mechanics
- Enemy waves with varied AI behavior
- Boss fights with progressive difficulty
- Smart bomb mechanics
- Score system and HUD display

## Project Structure

```
Google_4/
├── index.html          # Main HTML file (self-contained game)
├── README.md           # Basic project information
└── Claude.md           # This file - development guidelines
```

## Architecture

The entire game is contained in a single `index.html` file with embedded CSS and JavaScript. This is an intentional design choice for simplicity and self-contained deployment.

### Key Components

1. **Canvas Rendering**: Game uses HTML5 Canvas API for all graphics rendering
2. **Game Loop**: Main update/render cycle driven by `requestAnimationFrame`
3. **Input Handling**: Keyboard controls for player movement and actions
4. **HUD System**: On-screen display of score, lives, ammo, and game status
5. **Entity Management**: Game objects (player, enemies, bullets, effects) with update/render cycles
6. **Collision Detection**: Bounding box collision for all game entities
7. **Audio System**: Sound effects for weapons, explosions, and gameplay events

## Development Guidelines

### Code Style

- Use vanilla JavaScript (no frameworks or libraries)
- Keep functions focused and reasonably sized
- Use descriptive variable and function names
- Add comments only for non-obvious logic
- Maintain consistent indentation (2 spaces)

### Adding Features

When adding new features:

1. **Weapons System**: New weapons should follow the existing weapon switching pattern
2. **Enemies**: New enemy types should implement the basic update/render interface
3. **Visual Effects**: Particle effects and animations should be efficient for mobile
4. **HUD Updates**: New stats should be added to the HUD display with appropriate formatting

### Performance Considerations

- The game must run smoothly on mobile devices (target 60 FPS on mid-range phones)
- Minimize DOM manipulation - use Canvas rendering instead
- Batch collision checks where possible
- Clean up particle effects and bullets efficiently
- Monitor draw call count for performance-critical paths

### Mobile Optimization

- Touch controls must respond immediately to user input
- Canvas should respect viewport dimensions on all devices
- Safe area insets handled via CSS env() variables
- No horizontal scrolling - maintain vertical orientation focus

## Common Development Tasks

### Running the Game

Simply open `index.html` in a modern web browser. No build process or server required.

### Testing Changes

1. Open index.html in a browser
2. Test on desktop with keyboard controls
3. Test on mobile device with touch controls
4. Verify HUD displays correctly at various viewport sizes
5. Check frame rate using browser DevTools

### Debugging

- Use browser DevTools console for `console.log()` output
- Check the Canvas rendering by inspecting element bounds
- Use DevTools performance profiler to check frame rates
- Mobile debugging can be done via remote debugging (Chrome DevTools)

### Adding New Content

- **New Weapons**: Modify weapon firing logic and visual representation
- **New Enemies**: Define sprite size, movement pattern, and fire pattern
- **New Stages**: Adjust enemy wave sequences and boss stats
- **UI Changes**: Update HUD styling and layout via CSS

## Game Features Reference

### Player Mechanics
- WASD or Arrow keys for movement
- Mouse click or touch for firing
- Number keys or weapon switch button for weapon selection
- Space for smart bomb (limited ammo)

### Enemy System
- Enemies spawn in waves with increasing difficulty
- Different enemy types with unique behavior patterns
- Boss fights with multi-phase design

### Scoring System
- Points awarded for destroying enemies
- Multiplier system for consecutive hits
- Bonus points for perfect waves (destroying all enemies without taking damage)

## File Editing Best Practices

Since all code is in a single HTML file:

1. Use a quality text editor with HTML/JavaScript support
2. When modifying, be careful with the structure:
   - CSS goes in the `<style>` tag
   - JavaScript goes in the final `<script>` tag
   - HTML structure is minimal - just container and canvas
3. Keep the file well-organized with comment sections for major components
4. Test after each significant change

## Version Control

- Commit messages should be descriptive
- Include what changed and why
- Keep commits focused on a single feature or fix
- Use branch `claude/inspiring-planck-85p9w1` for development

## Dependencies

**None** - This is a zero-dependency project using only:
- HTML5 Canvas API
- Vanilla JavaScript (ES6+)
- CSS3 (no preprocessors)
- Modern browser APIs (requestAnimationFrame, touch events)

## Browser Support

- Chrome/Edge 90+
- Firefox 88+
- Safari 14+
- Mobile browsers with modern JavaScript support

## Performance Targets

- **Desktop**: 60 FPS minimum
- **Mobile**: 30-60 FPS (graceful degradation acceptable)
- **Memory**: Keep particle count and entity count reasonable
- **Draw Calls**: Minimize Canvas operations per frame

## Future Considerations

When expanding the project:
- Consider breaking into multiple files if it grows beyond ~5000 lines
- Evaluate module bundlers only if significant dependencies are added
- Maintain mobile-first approach
- Keep touch control responsive as primary input method

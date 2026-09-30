# Claude.md - Sky Fighter Development Guide

## Project Overview
Sky Fighter is a Raiden-style vertical shoot-em-up browser game built with HTML5 Canvas and vanilla JavaScript. The game features aircraft combat with weapon switching, smart bombs, and boss battles.

## Development Guidelines

### Code Structure
- **Single file codebase**: All game logic is contained in `index.html`
- **Canvas-based rendering**: Uses HTML5 Canvas API for graphics
- **Game loop**: Runs at 60 FPS with requestAnimationFrame
- **Modular design**: Game is organized into distinct components (player, enemies, bullets, etc.)

### Before Starting Work
1. Always read the full `index.html` file to understand the current implementation
2. Test changes in a browser before committing
3. Maintain mobile responsiveness - the game targets mobile devices with viewport-fit=cover

### Code Style
- Use vanilla JavaScript (no frameworks)
- Maintain consistent indentation (2 spaces)
- Use descriptive variable and function names
- Add comments for complex game logic or non-obvious calculations
- Keep the file under 60KB where practical

### Testing
1. **Local browser testing**: Open `index.html` in a browser (Chrome/Firefox/Safari)
2. **Mobile testing**: Test on actual mobile devices or use browser dev tools device emulation
3. **Game flow**: Test menu navigation, gameplay, pause/resume, level progression, boss fights
4. **Graphics**: Verify visual effects render correctly (explosions, weapon effects, etc.)
5. **Performance**: Monitor frame rate stability (target 60 FPS)

### Game Mechanics to Preserve
- Player ship controls (arrow keys or touch)
- Weapon system with switching
- Smart bomb mechanics
- Enemy spawning and AI patterns
- Boss battles with stage progression
- Score and health systems
- Pause/resume functionality

## Commit Guidelines
Follow the standard format with attribution:
```
<Clear, descriptive message describing the change>

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
Claude-Session: <session URL>
```

### Commit Types
- `feat:` New game feature or mechanic
- `fix:` Bug fix in game logic or rendering
- `perf:` Performance improvements
- `refactor:` Code reorganization without behavior change
- `style:` CSS or visual improvements

## Known Constraints
- Single HTML file deployment - keep total file size reasonable
- No external libraries - maintain vanilla JS approach
- Mobile-first design philosophy
- Touch and keyboard input support required

## When Adding Features
1. Update game mechanics thoughtfully to maintain balance
2. Test extensively on various devices and screen sizes
3. Preserve existing game flow and user experience
4. Document any new game mechanics clearly
5. Keep visual style consistent with existing design

## Performance Considerations
- Avoid creating excessive objects per frame
- Use object pooling for bullets and enemy projectiles
- Limit particle effects based on device capabilities
- Monitor canvas draw calls and optimize rendering

## Debugging Tips
- Use browser DevTools console to log game state
- Check Canvas context methods for typos
- Verify timing/collision calculations
- Test across different browsers for compatibility

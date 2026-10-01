# Claude.md - Google_4 Project Guidelines

## Project Overview

**Google_4** is a web-based vertical shoot-em-up game inspired by Raiden-style arcade games. The project contains:
- **index.html** - Main game file with embedded HTML, CSS, and JavaScript
- **README.md** - Project metadata

## Development Guidelines

### Code Organization

The project uses a single-file HTML structure with embedded styles and scripts. All game logic, rendering, and event handling are contained within `index.html`.

### When Making Changes

1. **Game Mechanics**: Ensure changes maintain the core Raiden-style gameplay (vertical scrolling, player ship control, bullet patterns, enemy waves)
2. **Testing**: Always test changes in a browser before committing:
   - Game initialization and startup
   - Player controls (movement, shooting)
   - Enemy spawning and behavior
   - Collision detection
   - Score and lives system
3. **Performance**: Monitor for any jank or frame rate drops when adding visual effects or new features

### Version Control Practices

- Commit messages should clearly describe what was changed (e.g., "Add new enemy type", "Fix collision detection", "Optimize rendering")
- Include the standard Claude Code attribution in commit messages
- Each commit should represent a logical unit of work

### Testing Approach

1. **Manual Testing**: Since this is a game, browser-based manual testing is essential
2. **Test Checklist**:
   - Start the game and verify it loads without errors
   - Test player movement (arrow keys or mouse)
   - Test shooting mechanics
   - Verify enemy spawning and behavior
   - Check collision detection accuracy
   - Confirm UI elements display correctly
   - Test game over and restart functionality

### Browser Requirements

- Modern browser with HTML5 Canvas support
- JavaScript ES6+ support
- No external dependencies required

### How to Run

```bash
# Open index.html in a web browser
# Can be served through any local web server or opened directly as a file
```

## Common Tasks

### Adding a New Feature
1. Identify the relevant section in the JavaScript code
2. Add the feature implementation
3. Test thoroughly in browser
4. Commit with clear message describing the addition

### Bug Fixes
1. Reproduce the bug in browser
2. Identify root cause in code
3. Implement fix
4. Verify fix resolves issue
5. Check for regressions in other features

### Performance Optimization
1. Profile the game to identify bottlenecks
2. Optimize code or rendering
3. Verify improvements with browser DevTools
4. Ensure no visual quality degradation

## Project Rules

- **No external libraries**: Keep dependencies minimal - the game should work with vanilla HTML/CSS/JavaScript
- **Browser compatibility**: Target modern browsers (Chrome, Firefox, Safari, Edge)
- **Readable code**: Maintain clear variable and function names for future maintenance
- **Game balance**: When adjusting difficulty, player speed, or scoring, ensure the game remains playable and fun

## Contact & Attribution

Developed with Claude Code by Anthropic.

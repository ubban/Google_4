# Claude.md — Sky Fighter Game Development Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single-file HTML5 Canvas application. The entire game logic, rendering, and UI are embedded in `index.html` as inline JavaScript and CSS.

## Project Structure

```
Google_4/
├── index.html          # Main game file (complete game implementation)
├── README.md           # Project readme with timestamp
└── Claude.md           # This file — Claude Code guidelines
```

## Key Technical Details

- **Technology**: HTML5 Canvas, vanilla JavaScript, CSS3
- **Single-file architecture**: All game logic, graphics, and styling in one file
- **Target format**: Open in any modern web browser

## Development Guidelines

### Code Organization in index.html

The index.html file is structured as follows:

1. **HTML structure** (top): Basic document setup with single canvas element
2. **CSS styles** (embedded): Game styling, HUD, responsive layout, mobile support
3. **Game logic** (inline script): Canvas rendering, game state, player/enemy/bullet management

### Working with the Game Code

When modifying the game:

1. **Canvas rendering** is done via the main game loop (requestAnimationFrame)
2. **Game state** includes: player position/health, enemies, bullets, score, stage
3. **Input handling** supports: mouse/touch controls, keyboard input for desktop
4. **Collision detection** handles player-bullet, player-enemy, bullet-enemy interactions
5. **Stage system** manages level progression with bosses

### Making Changes

- Keep all changes within the single index.html file
- Maintain responsive design for mobile and desktop
- Preserve the dark theme (#05122b background, light text)
- Update inline styles if UI changes are needed
- Test changes in a modern browser (Chrome, Firefox, Safari, Edge)

## Testing & Running

1. **Open in browser**: Simply open `index.html` in any modern web browser
2. **No build step needed**: This is a static HTML file, no compilation required
3. **Mobile testing**: The game is mobile-responsive; test on various screen sizes
4. **Console check**: Open browser DevTools console to check for any JavaScript errors

## Git Conventions

When committing changes:

- Use clear, descriptive commit messages that describe the game feature or fix
- Reference the specific game mechanic or system affected (e.g., "Fix weapon switching", "Add new enemy type", "Improve collision detection")
- Group related changes in a single commit
- Example: `Fix boss hit detection and improve bullet particle effects`

## Common Tasks

### Adding a New Feature

1. Identify which part of the game logic needs modification (rendering, input, collision, etc.)
2. Find the relevant section in the JavaScript code within index.html
3. Implement the feature, maintaining code style consistency
4. Test thoroughly in the browser before committing

### Fixing Bugs

1. Open browser DevTools to identify the exact issue
2. Locate the problematic code section
3. Apply minimal, targeted fix
4. Test across different browsers and devices
5. Commit with a clear reference to what was fixed

### Performance Optimization

- Monitor FPS and rendering performance
- Profile with browser DevTools if performance issues arise
- Keep the game loop efficient (requestAnimationFrame callback)
- Consider mobile device limitations when adding features

## Browser Compatibility

Target modern browsers with full ES6+ support:
- Chrome/Chromium 90+
- Firefox 88+
- Safari 14+
- Edge 90+

## Additional Notes

- The game uses no external libraries or dependencies — pure HTML5/CSS3/JavaScript
- Audio features (if implemented) should be handled via Web Audio API
- Save/load functionality would require localStorage or similar mechanisms
- Mobile controls use touch events; desktop uses mouse/keyboard events

---

**Last Updated**: 2026-09-27
**Branch**: claude/inspiring-planck-gfv1im

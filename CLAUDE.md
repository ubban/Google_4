# Sky Fighter - Claude Code Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5 and JavaScript. The game runs entirely in a single HTML file with embedded CSS and JavaScript, making it instantly playable in any modern web browser.

### Repository
- **Name**: Google_4
- **URL**: https://github.com/ubban/Google_4
- **Language**: JavaScript (HTML5 Canvas)
- **License**: Not specified (add if known)

## Project Structure

```
/
├── index.html          # Main game file (HTML, CSS, JS combined)
├── README.md           # Project readme
└── CLAUDE.md           # This file - Claude Code documentation
```

### Architecture

**index.html** contains:
- HTML structure with canvas element for rendering
- CSS styling (dark theme, responsive layout, animations)
- JavaScript game engine with full game logic
- HUD display system for stats and game state

## Key Features

- **Gameplay**: Player-controlled ship, vertical scrolling stage
- **Weapons**: Multiple weapon types with switching capability
- **Smart Bombs**: Special power-up weapon that clears screen
- **Boss Fights**: Stage-based progression with boss enemies
- **Mobile Responsive**: Touch and mouse controls, responsive design
- **Visual Effects**: Animations, flashes, vignette effects
- **HUD**: Real-time display of health, bombs, weapon type, multiplier

## Development Guidelines

### Code Organization

The JavaScript game engine is organized around:
- **Game state management**: Player health, weapons, bombs, score
- **Canvas rendering**: All graphics drawn to HTML5 canvas
- **Event handling**: Mouse, touch, and keyboard input
- **Game loop**: Continuous update/render cycle

### Style Conventions

- Dark theme with blue/cyan color scheme
- Responsive font sizing using `clamp()` for mobile
- Smooth animations using CSS keyframes and canvas effects
- Accessible contrast ratios for readability

### Adding Features

When adding new features:
1. Keep all code within index.html unless significant refactoring is planned
2. Maintain the existing dark theme and visual style
3. Ensure mobile responsiveness with viewport meta tags and touch-action CSS
4. Test on both desktop and mobile viewports
5. Preserve existing game balance and difficulty progression

### Performance Considerations

- Canvas is the primary rendering target (no DOM manipulation)
- Avoid layout thrashing or expensive DOM queries
- Use `requestAnimationFrame` for smooth animation
- Profile with browser DevTools to maintain 60 FPS

## Git Workflow

### Branch Naming

Follow these conventions for branch names:
- `main` - Production/stable branch
- `feature/[description]` - New features (e.g., `feature/boss-attack-patterns`)
- `fix/[description]` - Bug fixes (e.g., `fix/weapon-balance`)
- `claude/[adjective]-[noun]` - Claude-driven work (e.g., `claude/inspiring-planck-lmkf99`)

### Commit Messages

Write clear, descriptive commit messages:
- Use imperative mood ("Add feature" not "Added feature")
- Reference features or fixes in the message
- Example: "Add new boss attack pattern with phase transitions"
- Keep commits atomic - one logical change per commit

### Pull Requests

When creating a pull request:
1. Describe what changed and why
2. Reference any related issues
3. Include testing notes (browser versions tested, mobile testing done)
4. Ensure game is playable after changes
5. Verify visual effects render correctly

## Testing

### Manual Testing Checklist

Before pushing changes:
- [ ] Game loads without console errors
- [ ] Player ship renders and responds to input
- [ ] Weapons fire and damage enemies
- [ ] Enemy spawning and movement working
- [ ] HUD displays correct stats
- [ ] Mobile controls (touch) work on touch devices
- [ ] Canvas responsive on different screen sizes
- [ ] No FPS drops or stuttering on target devices

### Browser Compatibility

Target modern browsers:
- Chrome/Edge (latest 2 versions)
- Firefox (latest 2 versions)
- Safari (latest 2 versions)
- Mobile browsers (iOS Safari, Chrome Android)

## Deployment

The game is deployed as a static HTML file:
1. Push changes to `main` branch
2. GitHub serves the file directly (static hosting)
3. Game is immediately available at deployed URL
4. No build step required

## Development Tips

### Local Testing
- Open `index.html` directly in browser (file:// protocol)
- Or use a simple HTTP server: `python -m http.server 8000`
- Use browser DevTools Console to check for errors
- Use Performance tab to monitor frame rate

### Debugging
- Canvas drawing visible in DevTools Elements inspector
- Add `console.log()` statements to game loop for state inspection
- Use breakpoints in DevTools Sources tab
- Reduce game speed during debugging if needed

### Asset Management
- Currently all assets are embedded (colors, fonts, shapes drawn programmatically)
- No external images or resources required
- Reduces complexity and file transfer

## Code Review Focus Areas

When reviewing changes to this project:
1. **Correctness**: Does game logic work as intended?
2. **Performance**: Does it maintain 60 FPS on target devices?
3. **Compatibility**: Works on mobile and desktop?
4. **Style**: Consistent with existing visual theme?
5. **Mobile**: Touch controls work as expected?
6. **Balance**: Game difficulty reasonable?

## Contact & Attribution

- **Repository Owner**: ubban
- **Development**: Claude Code AI assistant
- **Last Updated**: 2026-09-24

---

*Generated with [Claude Code](https://claude.ai/code)*

# Sky Fighter - Codebase Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-page web application. Players control an aircraft to defeat enemies, switch weapons, deploy smart bombs, and battle stage bosses across multiple levels.

**Repository**: ubban/Google_4  
**Tech Stack**: HTML5, Canvas API, Vanilla JavaScript  
**Platform**: Web browser (Desktop & Mobile)  

## Project Structure

```
Google_4/
├── index.html          # Main game application
├── README.md           # Basic project info
└── CLAUDE.md          # This file
```

### Core Files

- **index.html** (58KB): Complete game implementation
  - Contains all HTML, CSS (embedded), and JavaScript
  - Self-contained single-page application
  - Includes UI/HUD rendering and game logic

## Architecture & Key Systems

### Game Loop
- Canvas-based 2D graphics rendering
- Real-time input handling (mouse/touch)
- Collision detection system
- Sprite management and animation

### Game Features
- **Player Aircraft**: Controllable with mouse position or touch
- **Weapons System**: Multiple weapon types with switching capability
- **Smart Bombs**: Special destructive items for clearing the board
- **Enemy AI**: Various enemy types with attack patterns
- **Stage Bosses**: Boss encounters with unique mechanics
- **HUD**: Real-time display of score, health, lives, current weapon

### UI/UX
- Dark theme optimized for gameplay
- Responsive design (scaling based on viewport)
- Touch-friendly mobile controls
- Safe area support for notched devices
- No horizontal scrolling

## Development Workflow

### Branch Strategy
- Main development branch: `claude/inspiring-planck-xjbdgp`
- Production branch: `main`
- Always commit and push changes to the feature branch

### Making Changes
1. Modify code in `index.html` (game logic, styling, or UI)
2. Test in browser to verify gameplay works
3. Commit with clear message explaining the change
4. Push to `claude/inspiring-planck-xjbdgp`

### Testing
- Manual testing in web browser (Chrome, Firefox, Safari, Edge)
- Test on both desktop and mobile viewports
- Verify game controls (mouse and touch)
- Check Canvas rendering performance
- Test collision detection and enemy behaviors

## Common Tasks

### Adding a New Feature
1. Locate the relevant system in the JavaScript code within `index.html`
2. Implement the feature (e.g., new weapon, enemy type, UI element)
3. Add appropriate CSS styles inline
4. Test thoroughly in browser
5. Commit and push

### Bug Fixes
- Identify the issue in browser dev tools
- Locate the buggy code in `index.html` JavaScript section
- Fix and test immediately in browser
- Commit fix with issue context

### Performance Optimization
- Profile using browser DevTools (Performance tab)
- Target the bottleneck (rendering, collision, animation)
- Implement optimization in JavaScript
- Verify performance improvement
- Commit with performance metrics

## Code Style Guidelines

- Use vanilla JavaScript (no frameworks or libraries)
- Maintain readable function names and variable naming
- Add comments for complex game logic
- Keep CSS organized and self-documenting
- Use semantic HTML5 elements

## Browser Compatibility

- Modern browsers with HTML5 Canvas support
- Chrome/Chromium
- Firefox
- Safari (iOS 13+)
- Edge

## Performance Considerations

- Canvas drawing is the main performance constraint
- Minimize DOM manipulations during game loop
- Use requestAnimationFrame for smooth rendering
- Batch collision detection checks
- Consider object pooling for frequently created/destroyed entities

## Debugging Tips

- Use browser DevTools JavaScript console for debugging
- Canvas rendering can be visualized with DevTools Elements inspector
- Performance profiling: DevTools Performance tab
- Network tab not needed (single-page application)
- Mobile debugging: Chrome DevTools remote debugging

## Future Enhancements

Potential areas for improvement:
- Additional weapon types
- More enemy varieties
- Additional stage bosses
- Score persistence (localStorage)
- Mobile app wrapper (PWA)
- Sound/music system
- Difficulty levels
- Leaderboard system

## Attribution

This codebase was initialized and is maintained through Claude Code automated sessions.

---
*Last updated: 2026-10-05*

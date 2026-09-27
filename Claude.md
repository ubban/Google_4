# Claude.md - Sky Fighter Development Guide

## Project Overview

**Sky Fighter** is a browser-based Raiden-style vertical shoot-'em-up game built with HTML5 Canvas and vanilla JavaScript. The game features:
- Player aircraft with weapon switching mechanics
- Smart bomb drops
- Stage bosses and progressive difficulty
- Canvas-based rendering with particle effects
- Touch and keyboard controls

## Repository Structure

```
Google_4/
├── index.html          # Single-file game implementation
├── README.md           # Project README
└── Claude.md           # This file - development guide
```

## Code Style & Standards

### JavaScript
- **No build system**: The game runs directly in the browser as a single HTML file
- **Vanilla JavaScript**: No frameworks or external libraries (except inline HTML)
- **Naming conventions**:
  - Classes: PascalCase (e.g., `Player`, `Enemy`, `Projectile`)
  - Functions/variables: camelCase (e.g., `updateGame`, `collisionDetected`)
  - Constants: UPPER_SNAKE_CASE (e.g., `CANVAS_WIDTH`, `GRAVITY`)
- **Comments**: Only when the "why" is non-obvious. Avoid over-commenting.
- **Performance**: Canvas rendering optimization is critical—minimize DOM operations, batch draw calls

### HTML & CSS
- **Single-file design**: Keep all HTML, CSS, and JavaScript in `index.html`
- **Dark theme**: Match the existing #05122b background and blue/cyan palette
- **Responsive**: Use `clamp()` and viewport units for mobile/desktop scaling
- **Accessibility**: Ensure color contrast and support both keyboard and touch input

## Development Guidelines

### When Making Changes
1. **Test in browser**: Always test gameplay before committing changes
   - Test on desktop (keyboard controls)
   - Test on mobile (touch controls)
   - Verify no console errors
2. **Keep scope focused**: Bug fixes don't need refactoring; new features shouldn't include cleanup
3. **Avoid premature abstractions**: Three similar lines is better than an overly generic helper
4. **Performance first**: Monitor frame rate; avoid memory leaks or excessive garbage collection

### Git Commits
- Write clear, concise commit messages describing the "why"
- Example: `"Improve collision detection accuracy for tight boss patterns"` not `"fix collision"`
- Each commit should be a logically complete unit

### Testing
- No automated tests required for canvas games, but:
  - Play through at least one full round after changes
  - Test edge cases (screen edges, rapid input, multiple enemies)
  - Verify no regressions in existing features

## Game Architecture Notes

### Suggested Structure (not mandatory, but helps)
- **Game State**: Track player, enemies, projectiles, score, level
- **Update Loop**: Process input, update positions, handle collisions
- **Render Loop**: Draw game objects using `canvas.fillRect()`, `drawImage()`, etc.
- **Input Handling**: Support both `keydown` events and touch/mouse events
- **Collision System**: Use simple bounding-box collision detection for performance

### Common Tasks

**Adding a new weapon type:**
- Define weapon properties (damage, fire rate, spread)
- Add draw logic in the render function
- Handle weapon switching logic in input processing
- Test collision with enemies

**Adjusting difficulty:**
- Modify enemy spawn rates in the update loop
- Adjust enemy movement patterns
- Scale boss health/attack patterns by level
- Increase projectile speeds at higher stages

**Fixing performance issues:**
- Profile with DevTools Performance tab
- Reduce enemy/projectile count if needed
- Optimize draw calls (batch operations, use offscreen canvas if appropriate)
- Cache frequently-accessed values

## Deployment

The game is ready to deploy as-is:
- No build process needed
- Open `index.html` in any modern browser
- Works on desktop and mobile devices
- No external dependencies to manage

## Contact & Attribution

Game development and maintenance guided by Claude Code agent (Haiku 4.5)

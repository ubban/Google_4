# Claude.md - Sky Fighter Game Development Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla JavaScript and HTML5 Canvas. Players switch weapons, drop smart bombs, and take down stage bosses in an engaging arcade-style experience.

### Key Technologies
- HTML5 Canvas for rendering
- Vanilla JavaScript (ES6+)
- Mobile-optimized with touch support
- Responsive design with viewport handling

## Code Organization

The `index.html` file contains:
- **Structure**: Single HTML file with embedded styles and JavaScript
- **Canvas**: Game rendering via `#gameCanvas` element
- **HUD**: Game status display via `#hud` div with stats

## Development Guidelines

### When Adding Features
1. Keep all game logic in vanilla JavaScript - no external frameworks required
2. Maintain canvas-based rendering for performance
3. Ensure touch events work alongside mouse/keyboard input
4. Test on both desktop and mobile viewports
5. Update the HUD display when adding new stats or information

### Code Style
- Use modern ES6+ syntax
- Keep functions focused and single-purpose
- Use descriptive variable names for game entities (player, bullets, enemies, bosses)
- Comment non-obvious game mechanics and collision detection logic

### Game Mechanics to Preserve
- **Player controls**: WASD or arrow keys for movement, spacebar to shoot
- **Weapon system**: Ability to switch between different weapon types
- **Smart bombs**: Special attack that clears the screen
- **Boss encounters**: Stage-based progression with boss battles
- **Scoring system**: Points for destroying enemies and bosses

### Mobile Considerations
- Touch-action: none (prevent default touch behaviors)
- Viewport-fit: cover (use full screen on notched devices)
- Safe area insets: Account for mobile notches and status bars
- Crosshair cursor for intuitive aiming
- Responsive font sizing with clamp()

## Testing Checklist

Before pushing changes:
1. ✓ Game renders correctly on canvas
2. ✓ Player movement works with keyboard and touch
3. ✓ Shooting mechanics function properly
4. ✓ Weapon switching works
5. ✓ Bomb mechanic functions
6. ✓ Collision detection accurate
7. ✓ Mobile viewport scaling correct
8. ✓ No console errors or warnings
9. ✓ Performance remains smooth at 60fps
10. ✓ HUD displays current game state

## Performance Notes

- Use requestAnimationFrame for game loop
- Minimize DOM manipulation - rely on canvas
- Batch canvas operations when possible
- Profile with DevTools to maintain 60fps target
- Mobile devices may have constraints; test on lower-end devices

## File Structure

```
Google_4/
├── index.html          (Main game file with all HTML, CSS, JS)
├── README.md           (Project description)
├── Claude.md           (This file - development guidelines)
└── .git/               (Version control)
```

## Git Commit Guidelines

- Use clear, descriptive commit messages
- Reference what changed (e.g., "Add enemy wave system", "Fix collision detection")
- Keep commits focused on single features or fixes
- Include Co-Authored-By and Claude-Session attribution as configured

## Important: Preserving Game State

When modifying the game:
- Preserve existing game balance and difficulty progression
- Don't alter core mechanics without explicit request
- Maintain backward compatibility with save/load systems if they exist
- Test new features don't break existing boss encounters

## Resources & References

- Canvas API documentation for rendering optimizations
- Mobile viewport best practices for full-screen games
- requestAnimationFrame for consistent game loops
- Touch Events API for mobile input handling

---

**Last Updated**: 2026-09-27
**Maintained By**: Claude Code

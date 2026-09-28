# Sky Fighter Codebase Guide

## Project Overview

Sky Fighter is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5 Canvas and JavaScript. The game features switching weapons, smart bombs, and stage bosses with progressive difficulty.

**Key Details:**
- **Type:** Browser-based game
- **Technology:** HTML5, Canvas API, vanilla JavaScript
- **Main File:** `index.html` (single-file implementation)
- **Status:** Active development

## Architecture

The game is implemented as a single HTML file (`index.html`) that contains:
- HTML structure with canvas element
- CSS styling (dark theme, game UI, animations)
- JavaScript game logic

### Key Game Systems

1. **Player System**
   - Health/lives system with visual hearts indicator
   - Multiple weapon types (starting weapon, rapid-fire, etc.)
   - Weapon switching mechanic
   - Smart bomb mechanic with limited charges

2. **Enemy System**
   - Multiple enemy types
   - Boss encounters at stage completion
   - Progressive difficulty scaling

3. **Rendering**
   - Canvas-based 2D rendering
   - Dynamic HUD with stats display (health, score, weapon, multiplier)
   - Visual effects (flashing, vignette for damage)
   - Touch-action support for mobile

4. **Input Handling**
   - Mouse/touch positioning for aiming
   - Touch-action: none to prevent browser default behavior
   - Crosshair cursor for better targeting feedback

## File Structure

```
Google_4/
├── index.html          # Main game file (all code in one file)
├── README.md           # Project readme
└── CLAUDE.md           # This file
```

## Development Guidelines

### When Making Changes

1. **Single-File Considerations**
   - All changes are made to `index.html`
   - CSS is in `<style>` tags (lines 10-100+)
   - JavaScript game logic follows after the HTML structure
   - Keep game logic organized in logical sections

2. **Git Practices**
   - Create descriptive commit messages
   - Reference game systems when making changes (e.g., "Add new weapon type", "Fix boss AI")
   - Test gameplay thoroughly on both desktop and mobile
   - Use branches for experimental features

3. **Testing**
   - Open `index.html` directly in a browser
   - Test on multiple screen sizes (desktop, tablet, mobile)
   - Verify touch controls work on mobile devices
   - Check performance on slower devices

### Code Organization Tips

- **CSS**: Located in `<style>` block early in the file
  - Root theme variables at top
  - Component styles organized by element ID
  - Animations defined at the end

- **JavaScript**: Follows HTML structure
  - Game initialization
  - Core game loop
  - Input handling
  - Rendering logic
  - Asset/data definitions

### Performance Considerations

- Canvas rendering is optimized for real-time updates
- Touch-action: none prevents browser scrolling on mobile
- Viewport-fit: cover ensures full-screen on mobile
- Use requestAnimationFrame implicitly through game loop

## Game Mechanics

### Weapons
- Different weapon types affect gameplay
- Players can switch weapons mid-game
- Each weapon has distinct fire patterns and damage

### Bombs
- Limited smart bomb charges
- Visual indicator shows available bombs
- Strategic resource for tough situations

### Difficulty Progression
- Score-based progression
- Boss encounters increase in difficulty
- Enemy patterns scale with progression

## Browser Compatibility

- Modern browsers with Canvas support
- Mobile browsers with touch support
- Requires JavaScript enabled
- Works in fullscreen mode

## Common Tasks

### To Add a New Weapon Type
1. Define weapon properties in the weapon data section
2. Add rendering logic for weapon appearance
3. Implement fire pattern in the weapon firing logic
4. Update weapon cycling logic

### To Adjust Game Difficulty
1. Modify enemy spawn rates
2. Adjust boss health/damage
3. Tune progression speed
4. Balance resource availability (bombs, pickups)

### To Add New Visual Effects
1. Create CSS animations in `<style>` block
2. Add effect triggers in JavaScript game loop
3. Use z-index carefully for layering

## Attribution

When committing changes:
```
Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
Claude-Session: https://claude.ai/code/session_01U5iic1pBcfFpC9V1FVwkco
```

## Resources

- Canvas API: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API
- Game Development Best Practices: Consider frame rate, collision detection, state management
- Mobile Optimization: Test viewport settings, touch input, device rotation

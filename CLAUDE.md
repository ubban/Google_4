# Claude.md - Sky Fighter Game Codebase Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with HTML5 Canvas and JavaScript. Players control a fighter jet, switch weapons, drop smart bombs, and battle stage bosses across multiple levels.

### Key Characteristics
- **Type**: Single-file HTML5 game application
- **Main File**: `index.html` (contains HTML, CSS, and JavaScript)
- **Target**: Web browsers (desktop and mobile)
- **Architecture**: Game loop with Canvas-based rendering
- **Styling**: Embedded CSS with dark theme, responsive layout

## Codebase Structure

### index.html (Main Application File)
The entire game is contained within `index.html`:
- **HTML Section**: Canvas container (#gameCanvas) and HUD elements (#hud)
- **CSS Section**: Embedded styles for:
  - Game canvas styling with sky gradient background
  - Heads-up display (HUD) for stats and UI
  - Responsive layout with viewport-fit cover for mobile
  - Theme colors: Dark blue (#05122b, #0b1d3a) with blue accents (#1976d2)
- **JavaScript Section**: Game engine with Canvas API
  - Game loop using `requestAnimationFrame`
  - Input handling (keyboard, mouse, touch)
  - Physics and collision detection
  - Enemy AI and projectile management
  - Score and lives tracking
  - Level progression and boss encounters

### README.md
Contains project timestamp and basic description.

## Game Mechanics

### Player Aircraft
- **Firing System**: Primary weapon with ammunition management
- **Smart Bombs**: Limited-use special weapons
- **Health**: Lives system tracking player damage
- **Movement**: Responsive controls (WASD keys, mouse/touch input)

### Enemy Types
- Standard enemies with predictable patterns
- Boss encounters at stage completion
- Projectiles and collision mechanics

### Level Structure
- Progressive difficulty through multiple stages
- Boss encounters between levels
- Score-based progression system

## Technical Guidelines

### Canvas Rendering
- Use `requestAnimationFrame` for consistent 60 FPS gameplay
- Optimize drawing calls by batching and clearing canvas appropriately
- Consider WebGL acceleration for performance-critical operations

### Input Handling
- Support multiple input methods: keyboard (WASD/arrows), mouse, touch
- Ensure responsive feel with minimal input latency
- Handle window blur/focus events for pause functionality

### Mobile Considerations
- Responsive canvas sizing using max-width/max-height constraints
- Safe area insets for notched devices (`env(safe-area-inset-top)`)
- Touch-action CSS to prevent browser gestures interfering with gameplay
- Viewport configuration prevents user scaling

### Performance Optimization
- Use object pooling for bullets and enemies to reduce GC pressure
- Cache DOM queries and computed styles
- Minimize layout reflows during game loop
- Consider sprite batching for rendering optimization

## Working with This Codebase

### For Bug Fixes
1. Locate the relevant section in the JavaScript game logic (controls, rendering, physics)
2. Test in browser using F12 developer tools
3. Verify collision detection and physics calculations
4. Commit with clear messages about mechanics affected

### For Feature Additions
1. Identify where in the game loop the feature should integrate
2. Consider performance impact (enemy count, projectile limit, etc.)
3. Update HUD display if adding new stats or UI elements
4. Test across desktop and mobile browsers
5. Verify touch input handling if applicable

### For Performance Improvements
1. Profile using Chrome DevTools Performance tab
2. Check for unnecessary repaints or layout thrashing
3. Profile memory usage and GC behavior
4. Maintain 60 FPS target on target devices

## Browser Compatibility
- Modern browsers with Canvas API support
- Requires ES6+ JavaScript support
- Mobile browsers (iOS Safari, Chrome Mobile)
- Touch event support for mobile gameplay

## Build and Deployment
- No build process required (single static HTML file)
- Direct browser loading of index.html
- Can be served via any static file server
- Mobile-friendly viewport configuration enables responsive behavior

## Code Style Notes
- Use descriptive variable/function names for game systems
- Comments for complex physics or collision logic
- Consistent indentation (2 or 4 spaces)
- Modularize game logic into clear functional sections (rendering, input, physics, AI)

## Testing Guidelines
1. **Gameplay**: Test full game loop from start to boss encounters
2. **Input**: Verify keyboard, mouse, and touch controls work correctly
3. **Visuals**: Confirm rendering on different screen sizes and devices
4. **Performance**: Monitor FPS and memory usage during extended play
5. **Mobile**: Test on various device sizes and orientations

## Known Systems

### Core Game Systems
- **Game State Manager**: Tracks game status (playing, paused, game over)
- **Renderer**: Canvas-based drawing system
- **Input Handler**: Keyboard, mouse, and touch input processing
- **Physics Engine**: Collision detection and entity movement
- **Enemy Manager**: Enemy spawning and AI behaviors
- **Projectile System**: Bullet and special weapon management
- **UI/HUD System**: Score, lives, weapon display

## Documentation
- All documentation integrated into this file
- Inline comments in JavaScript for complex logic
- CSS classes descriptively named for UI components

---

*Last Updated: 2026-09-27*
*Project: Sky Fighter Game*

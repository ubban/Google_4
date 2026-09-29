# Claude.md — Sky Fighter Game Codebase

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built with vanilla HTML5 Canvas and JavaScript. Players control an aircraft to defeat enemies, switch weapons, deploy smart bombs, and defeat stage bosses.

## Architecture

- **Single File**: `index.html` contains all game logic, rendering, and styling
- **Technology Stack**: 
  - HTML5 Canvas for rendering
  - Vanilla JavaScript (no frameworks)
  - CSS3 for animations and effects
  - Responsive design with viewport meta tags

## Game Mechanics

- **Player Ship**: Controllable aircraft with multiple weapon types
- **Weapons**: Switchable weapon systems with different fire patterns
- **Smart Bombs**: Limited-use area-of-effect attacks
- **Enemies**: Various enemy types with distinct behaviors
- **Bosses**: Stage bosses with health systems
- **Health System**: Player takes damage from enemies; visual feedback via vignette effect
- **Score/Multiplier**: Gameplay progression tracking

## Key Features

- Touch-friendly controls (mobile + desktop)
- HUD display with stats (health, bombs, weapon type, score multiplier)
- Flash and vignette visual effects for feedback
- Responsive canvas scaling to viewport
- Dark theme with accent colors

## Code Organization

The game is monolithic with these main sections:

1. **HTML Structure**: Game container and canvas
2. **CSS Styling**: 
   - Root variables for colors and theme
   - Layout and positioning
   - Animation keyframes (heartBeat, vignettePulse, flashHit, flashBomb)
3. **JavaScript Logic** (in `<script>` tag):
   - Game state management
   - Entity systems (player, enemies, projectiles)
   - Input handling (mouse, touch, keyboard)
   - Rendering and animation loop
   - Collision detection
   - Game flow (loading, playing, game over)

## Development Guidelines

### Making Changes

- **Visual Polish**: Update CSS animations or add new keyframes
- **Game Balance**: Adjust speed, damage, health values in JavaScript
- **New Features**: Add new enemy types, weapons, or mechanics in the JavaScript game loop
- **Mobile Testing**: Always test with touch controls; use `touch-action: none` for canvas

### Performance Considerations

- Canvas rendering is the bottleneck; minimize DOM reflows
- Use `requestAnimationFrame` for smooth animation (not explicitly in current code — may need optimization)
- Monitor object pooling for bullets and enemies to reduce garbage collection

### Common Patterns

- **Canvas Rendering**: 2D context for drawing shapes and sprites
- **Game Loop**: Typically `draw()` + `update()` pattern per frame
- **Event Handling**: 
  - Keyboard: `keydown`/`keyup`
  - Touch: `touchstart`/`touchmove`/`touchend`
  - Mouse: `mousemove`/`click`

## Testing & Running

1. **Local**: Open `index.html` in a modern web browser
2. **Live**: Deploy to a web server (static hosting works fine)
3. **Mobile**: Test on actual devices for touch responsiveness

## Notes for Future Development

- Consider extracting game logic into modular files (separate entity classes, input handler, renderer) for large feature additions
- Sound/music system could be added without major refactoring
- Difficulty progression could be data-driven (waves defined in config)
- Persistent state (high scores) could use localStorage

## Attribution

Last updated: 2026-09-29

When making improvements, maintain the single-file architecture unless adding significant complexity warrants modularization.

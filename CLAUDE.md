# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-page HTML application. The game features:
- Player-controlled aircraft with multiple weapon types
- Enemy formations and boss encounters
- Power-up systems (weapon upgrades, smart bombs)
- Multi-stage level progression
- Score and lives tracking
- Responsive canvas-based rendering

## Architecture

### Single File Structure
- **index.html** (~1700 lines): Complete game implementation with embedded HTML, CSS, and JavaScript

The monolithic architecture keeps all game logic, rendering, and assets in one file for easy deployment and offline play.

## Key Game Systems

### Game Loop
- **Core update cycle**: Handles input, physics, collision detection, and rendering at ~60 FPS
- **Frame-based timing**: Uses requestAnimationFrame for smooth animation
- **Delta time handling**: Adjustable quality settings affect frame rate and visual effects

### Player System
- **Movement**: WASD keys or mouse drag
- **Weapons**: 
  - Default bullets (yellow rectangles)
  - Laser weapon (cyan beams with glow)
  - Smart bombs (area damage, limited uses)
- **Health tracking**: Lives counter, collision detection with enemy bullets
- **Power-ups**: Weapon switches, bomb replenishment

### Enemy System
- **Multiple enemy types**: Basic fighters, tanks (with health bars), boss ships
- **Behaviors**:
  - Formation-based movement
  - Shooting patterns (spread, directed)
  - Stage-based difficulty progression
  - Boss fights with unique patterns
- **Drop systems**: Power-ups drop on enemy destruction

### Rendering Pipeline
- **Canvas drawing**: All graphics rendered via 2D canvas context
- **Layering**: HUD overlay for score/stats, main game layer
- **Quality settings**: Glow effects toggle for performance (quality 0 = glow enabled)
- **Mobile optimization**: Responsive sizing with safe area insets

## Important Code Sections

### Game State Management
- **game object**: Central state container (score, lives, stage, etc.)
- **Canvas reference**: `#gameCanvas` - primary rendering surface
- **Collision arrays**: `bullets`, `enemyBullets`, `enemies`, `powerUps`, `boss`

### Coordinate System
- **Canvas dimensions**: Dynamically sized to window (1:1 aspect ratio)
- **Center origin**: Player spawns center-screen
- **Bounds checking**: Game objects wrap or reset at canvas edges

### Input Handling
- **Keyboard**: WASD for movement
- **Mouse**: Drag to move, hover position for aiming
- **Touch**: Full touch support for mobile

## Development Guidelines

### When Adding Features
1. **Keep it contained**: Avoid adding new files - extend the single HTML structure
2. **Maintain game loop**: New logic must integrate into update/render cycle
3. **Test collision**: Verify hitboxes for new enemies/weapons
4. **Mobile first**: Ensure touch/responsive behavior works

### Code Organization Pattern
- **Configuration objects**: Enemy types, weapon definitions at top of script
- **Utility functions**: Math helpers (distance, angle), drawing primitives
- **Game loop functions**: `update()`, `render()`, input handlers
- **State arrays**: Maintain clear separation of game entities

### Performance Considerations
- **Canvas clearing**: Explicitly clear each frame (no retain)
- **Batch drawing**: Group similar entities to minimize state changes
- **Quality toggle**: Disable glow effects (`quality = 1`) for lower-end devices
- **Collision optimization**: Early exit in nested loops when possible

### Common Tasks

**Adding a new enemy type:**
1. Define in `ENEMY_TYPES` object with hp, speed, damage, drop chance
2. Add spawn logic in stage generation
3. Implement drawing in the enemy render section
4. Add collision/behavior in update loop

**Adding a new weapon:**
1. Define weapon properties (damage, speed, rate of fire)
2. Add to weapon selection/cycling system
3. Implement bullet spawn in input handler
4. Add rendering in bullet section

**Adjusting difficulty:**
1. Modify enemy spawn rates in stage arrays
2. Adjust `ENEMY_TYPES` health/damage values
3. Alter bullet speeds or fire rates
4. Increase boss pattern complexity

## Testing

No automated test suite - game is tested through:
- **Manual play-testing**: Run in browser, progress through stages
- **Console verification**: Check frame rate and entity counts
- **Mobile testing**: Test on various screen sizes and touch devices
- **Performance monitoring**: Use browser DevTools to track frame timing

## Deployment

Simply serve `index.html` as a static file. No build process, no dependencies.

## Browser Compatibility

- **Modern browsers**: Chrome, Firefox, Safari, Edge
- **Canvas support**: Requires 2D canvas API
- **Mobile**: iOS Safari, Android Chrome (with touch support)

## Notes for Future Development

- Consider extracting game logic to separate modules if complexity grows beyond ~2000 lines
- Asset handling (sprites, sounds) currently uses procedural drawing - no external image/audio files needed
- Quality settings provide a simple performance knob for resource-constrained environments
- Mobile touches currently treated as mouse input - could optimize with touch-specific event handling

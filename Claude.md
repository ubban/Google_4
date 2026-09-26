# Claude.md — Sky Fighter Development Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5/JavaScript/Canvas. Players control a fighter jet, switch between weapons, drop smart bombs, and defeat stage bosses. The entire game runs in a single `index.html` file.

## Architecture

### Single-File Structure
- **`index.html`**: Contains all HTML, CSS, and JavaScript for the game
- **Game Engine**: Canvas-based 2D rendering with collision detection
- **State Management**: Game loop with entity management (player, enemies, bullets, effects)

### Key Systems
1. **Rendering Pipeline**: Canvas context for sprites, effects, HUD
2. **Physics & Collision**: Bounding box collision, projectile trajectory
3. **Entity System**: Player, Enemy, Bullet, Effect classes
4. **Input Handling**: Keyboard/mouse/touch controls
5. **Audio**: Sound effects and background music (if present)
6. **HUD**: Real-time display of health, weapons, bombs, score

## Development Guidelines

### Code Changes
- **All changes go into `index.html`** — there are no separate files
- Locate code by searching for function/class names or comments
- Preserve the existing structure: HTML head, CSS styles, JavaScript
- Test changes immediately in a browser to verify game mechanics

### Common Tasks

#### Adding/Modifying Game Features
1. Find the entity class (Player, Enemy, Bullet, etc.) in the `<script>` section
2. Modify properties, methods, or create new subclasses
3. Update the game loop if adding new entity types
4. Test in browser with keyboard (arrow keys, spacebar, Z for bomb)

#### Adjusting Game Balance
- Look for constants like health values, weapon damage, spawn rates
- Typically found at the top of the JavaScript section or in entity constructors
- Changes take effect immediately on page reload

#### Modifying Visuals
- Update CSS in the `<style>` section for UI/layout
- Modify Canvas drawing code in render functions for sprites/effects
- Test responsive scaling on mobile viewports

#### Bug Fixes
- Use browser DevTools console to identify errors
- Add `console.log()` statements to trace game state
- Reload page after editing to test fixes

### Testing Locally
1. Open `index.html` directly in a browser or serve with `python -m http.server`
2. Test game flow: start, weapon switching, boss encounters
3. Verify responsive behavior on mobile viewport sizes
4. Check console for JavaScript errors

## File Organization Within index.html

### Sections (in order)
1. **DOCTYPE & Meta Tags**: Viewport, theme color, icons
2. **CSS Styles**: :root variables, component styles, animations
3. **HTML Structure**: Container, canvas, HUD elements
4. **JavaScript**:
   - Utility functions and constants
   - Entity classes (Player, Enemy, Bullet, etc.)
   - Game state and manager
   - Game loop and input handlers

## Code Patterns

### Entity-Based System
```javascript
class Entity {
  constructor(x, y) { this.x = x; this.y = y; }
  update(dt) { /* Move, update state */ }
  draw(ctx) { /* Render to canvas */ }
  isAlive() { return this.health > 0; }
}
```

### Game Loop Pattern
```javascript
function gameLoop() {
  update(dt);      // Update all entities
  checkCollisions();
  draw();          // Render to canvas
  requestAnimationFrame(gameLoop);
}
```

### Collision Detection
- Typically bounding-box based (AABB)
- Check player vs enemies, bullets vs enemies, bosses vs player

## Important Notes for Claude Agents

- **No transpilation/build step**: Changes are live; just edit and reload
- **Mobile support**: Game uses viewport-fit:cover and touch events; test viewport scaling
- **Performance**: Canvas rendering is synchronous; avoid heavy computations in render path
- **State isolation**: Game state should be self-contained; avoid global dependencies beyond canvas context
- **Backwards compatibility**: Existing game saves/scores are not persisted (stateless)

## Commit Message Conventions

Follow this format for commit messages:
```
[feature|bugfix|refactor|docs] Brief description of change

Longer explanation if needed. Reference specific features or fixes.
```

Examples:
- `[feature] Add triple-shot weapon variant`
- `[bugfix] Fix player collision bounds on mobile`
- `[refactor] Consolidate enemy spawn logic`

## Quick Reference

### Key Controls
- **Arrow Keys / WASD**: Move player
- **Spacebar / Mouse Click**: Fire weapon
- **Z / Mobile tap**: Drop smart bomb
- **Enter**: Start game / Pause (if implemented)

### Canvas ID
- Element: `<canvas id="gameCanvas">`
- Context: `ctx = canvas.getContext('2d')`

### HUD Elements
- `#hud`: Main HUD container
- `#hearts`: Player health display
- `#bombs`: Smart bomb count
- `#weaponStat`: Current weapon display
- `#multStat`: Score multiplier

## Resources for Enhancement

- **Canvas API Docs**: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API
- **Game Loop Reference**: Common pattern is `requestAnimationFrame` for ~60 FPS
- **Collision Detection**: AABB or circle-based approaches work well for 2D shmups
- **Mobile Optimization**: Touch events and viewport scaling are already implemented

---

**Last Updated**: 2026-09-26  
**Maintained by**: Claude Code Agent

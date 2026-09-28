# Claude.md - Google_4 Project Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-page HTML5 application. The game features:
- Vertical scrolling shooter mechanics
- Weapon switching system
- Smart bomb drops
- Stage bosses
- HUD for displaying score, health, bombs, and weapon status

## Project Structure

- `index.html` - Single-file game implementation containing HTML, CSS, and JavaScript
- `README.md` - Project metadata

## Development Guidelines

### Code Organization
The `index.html` file is a complete, self-contained game implementation. When making changes:

1. **CSS Styling** - Located in `<style>` tags in the document head
   - Uses modern CSS with flexbox, grid, and animations
   - Implements responsive design with `clamp()` for fluid typography
   - Includes dark theme with color scheme preference support

2. **JavaScript Game Logic** - Located in `<script>` tags at the end of `<body>`
   - Core game loop and state management
   - Player control handling (keyboard and touch)
   - Enemy and projectile management
   - Collision detection
   - UI/HUD updates

3. **Game Constants** - Defined at the beginning of the script section
   - Game dimensions
   - Player stats (health, speed, firing rate)
   - Enemy/boss configurations
   - Visual effects parameters

### Implementation Standards

#### Naming Conventions
- Variables: `camelCase` for variables and functions
- Classes: `PascalCase` for game entity classes
- CSS Classes: `kebab-case` for HTML class attributes
- Private properties: Prefix with underscore when using class fields

#### Code Style
- Use modern ES6+ features (arrow functions, destructuring, template literals)
- Keep functions focused and under 50 lines when practical
- Add comments for non-obvious logic only
- No comments stating what code does - let code be self-documenting

#### Performance Considerations
- Minimize DOM manipulation in game loop
- Use canvas rendering for game entities (preferred over DOM)
- Cache DOM selectors at initialization
- Avoid excessive object creation in the game loop
- Use requestAnimationFrame for smooth rendering

### Game Mechanics Guidelines

#### Player Movement & Controls
- Mouse/pointer: Track cursor position for aiming
- Keyboard: Arrow keys or WASD for movement
- Touch: Direct touch position control on mobile
- All input should update game state, not DOM directly

#### Combat System
- Weapon types should be clearly defined with distinct properties
- Projectiles and enemies need velocity vectors
- Collision detection must be efficient (spatial partitioning recommended for many objects)
- Visual feedback (explosions, flashes) should be immediate

#### Enemy AI
- Enemies should follow simple movement patterns
- Boss patterns should be deterministic and learnable
- Difficulty should scale appropriately

#### HUD Display
- Update stats using DOM manipulation (safe outside game loop)
- Use `#hud` container for all UI elements
- Maintain visibility on all screen sizes using responsive units

## Testing & Validation

### Before Pushing Changes

1. **Functionality Test**
   - Launch the game in a browser
   - Test player movement and controls
   - Verify weapon switching
   - Test bomb mechanics
   - Confirm collision detection works
   - Check HUD updates reflect game state

2. **Browser Compatibility**
   - Test on Chrome/Edge (Chromium)
   - Verify touch controls work on mobile
   - Check responsive layout at various viewport sizes

3. **Visual Quality**
   - Verify animations run smoothly (60 FPS target)
   - Check that vignette and flash effects trigger correctly
   - Confirm HUD elements are readable at all sizes

4. **Code Quality**
   - Verify no console errors
   - Check that canvas context is properly managed
   - Ensure no memory leaks from event listeners

### Git Workflow

- Work on the designated feature branch: `claude/inspiring-planck-u0ghpg`
- Commit changes with clear, descriptive messages
- Always test locally before pushing
- Push to the feature branch with: `git push -u origin claude/inspiring-planck-u0ghpg`

## Key Implementation Patterns

### Game Loop Pattern
```javascript
function gameLoop() {
  update();      // Update game state
  render();      // Draw game objects
  updateHUD();   // Update UI display
  requestAnimationFrame(gameLoop);
}
```

### Entity Management
- Keep arrays for enemies, projectiles, and explosions
- Remove dead/off-screen entities regularly
- Use Array.filter() for cleanup when practical

### State Management
- Store game state in a single object or class
- Player health, position, weapons, score
- Current level/stage information
- Enemy list and positions

### Event Handling
- Use delegated event listeners when possible
- Remove listeners on cleanup
- Prefer canvas-based input handling for performance

## Common Tasks

### Adding a New Weapon Type
1. Define weapon properties (damage, fire rate, projectile speed)
2. Add sprite/visual representation
3. Implement firing logic in the appropriate event handler
4. Update weapon cycling mechanism
5. Add visual feedback in HUD

### Adding Enemy Patterns
1. Create enemy class with position and movement vector
2. Define animation/movement pattern
3. Implement collision checking
4. Create visual representation
5. Add to enemy spawn logic

### Modifying Game Difficulty
1. Adjust enemy spawn rates
2. Modify enemy health/damage values
3. Tune player movement or fire rate
4. Adjust boss patterns
5. Test progression feels fair and challenging

## Notes for Claude AI Agents

- Always test changes in a browser before committing
- Be mindful of performance - this is a game with real-time constraints
- Maintain visual polish and responsive design
- Document any new game mechanics clearly
- Keep the single-file structure; use semantic HTML sections and organized CSS/JS blocks
- Prioritize player experience and game feel over micro-optimizations

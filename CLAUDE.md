# Sky Fighter - Claude Code Guidelines

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built with vanilla HTML5 Canvas and JavaScript. The game features:

- Player-controlled aircraft with weapon switching mechanics
- Multiple enemy types with varied behavior patterns
- Boss encounters with stage progression
- Smart bomb mechanics for clearing the screen
- Score and health management systems
- Responsive controls for both keyboard and touch input

## Development Branch

All feature development and bug fixes should be committed to: **`claude/inspiring-planck-plazob`**

## Code Style & Conventions

### General Principles

- **Vanilla JavaScript**: Use only vanilla JS, no external frameworks except for game logic
- **HTML5 Canvas**: All rendering is done through the Canvas 2D context
- **No transpilation**: Write ES6+ that runs directly in modern browsers
- **Comments**: Use sparingly; only explain the *why* when non-obvious
- **Naming**: Use clear, descriptive names for variables, functions, and game entities

### Structure

The game code is organized in a single `index.html` file with embedded `<script>` tag containing:

1. **Configuration**: Game constants and settings
2. **Game State**: Player, enemies, projectiles, particles
3. **Game Loop**: Update and render cycle
4. **Input Handling**: Keyboard and touch controls
5. **Entity Systems**: Player, Enemy, Projectile, Particle, Boss
6. **Rendering**: Canvas drawing routines
7. **Audio**: Sound effect and music management
8. **UI**: HUD and menu rendering

### Naming Conventions

- **Functions**: camelCase (e.g., `updatePlayerPosition`, `drawHealthBar`)
- **Classes/Constructors**: PascalCase (e.g., `Player`, `Enemy`, `Projectile`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `CANVAS_WIDTH`, `PLAYER_SPEED`)
- **Private methods**: Prefix with `_` (e.g., `_updateCollisions`)

### Game Constants

All magic numbers should be defined as constants at the top of the game logic:

```javascript
// Example constants
const CANVAS_WIDTH = 400;
const CANVAS_HEIGHT = 600;
const PLAYER_SPEED = 5;
const ENEMY_SPAWN_RATE = 0.02;
```

## Architecture Notes

### Game Loop

The game uses a standard game loop pattern:

1. **Update phase**: Update entity positions, health, state
2. **Render phase**: Clear canvas and draw all entities
3. **RequestAnimationFrame**: Loop continues at display refresh rate

### Entity System

All game objects (player, enemies, projectiles) follow a similar pattern:

- **Constructor**: Initialize entity properties
- **update()**: Update position, state, behavior
- **draw()**: Render entity to canvas
- **isActive()**: Check if entity should remain in game

### Weapon System

Weapons are defined with:
- Projectile speed and damage
- Fire rate and spread pattern
- Visual appearance and sound effects
- Weapon switching via number keys (1-3)

### Boss Encounters

Boss patterns are stage-specific and include:
- Scripted movement phases
- Attack patterns with cooldowns
- Health tracking and defeat condition
- Stage progression on boss defeat

## Testing Guidelines

### Manual Testing Checklist

Before committing changes:

- [ ] Game starts and displays correctly
- [ ] Player can move in all directions
- [ ] Weapons fire and deal damage
- [ ] Enemies spawn and behave correctly
- [ ] Collisions are detected accurately
- [ ] Score and health display correctly
- [ ] Touch controls work on mobile
- [ ] Game doesn't crash on specific edge cases
- [ ] Audio plays without errors
- [ ] Game resets properly on game over

### Performance Considerations

- Monitor frame rate (target 60 FPS)
- Keep particle count reasonable (~200 max)
- Avoid excessive canvas redraws
- Profile memory usage in DevTools
- Test on lower-end devices if possible

## Git Workflow

### Branch Strategy

- **main**: Stable, playable version
- **claude/inspiring-planck-plazob**: Development branch for Claude Code
- Feature work should go directly to the designated branch

### Commit Messages

Use clear, descriptive commit messages:

```
Add smart bomb visual effects with screen flash

- Implement white flash when bomb detonates
- Add particle burst animation for visual feedback
- Increase particle count for more dramatic effect
- Update enemy clearing logic for bomb range
```

Keep commits focused on single features or bug fixes. Each commit should leave the game in a playable state.

### Pull Requests

When creating a PR:

1. Ensure game is fully playable with all changes
2. Test on both desktop and mobile
3. Include description of changes and why they were made
4. Reference any related issues or stage progression changes

## Specific Tasks & Features

### Adding New Enemy Types

When adding a new enemy:

1. Create enemy constructor with properties (x, y, health, speed)
2. Implement update() method for movement and behavior
3. Implement draw() method using Canvas 2D API
4. Add spawn logic to stage definition
5. Test collision detection and damage

### Modifying Weapons

When adjusting weapons:

1. Update weapon constants (speed, damage, rate)
2. Test fire patterns and visual appearance
3. Verify ammunition doesn't exceed reasonable counts
4. Ensure audio cues play correctly
5. Balance difficulty appropriately

### Stage Design

When designing stages:

1. Define enemy spawn patterns as arrays of timing/type
2. Create boss encounter with health and attack phases
3. Scale difficulty progressively through the stage
4. Test completion time and pacing
5. Verify boss is defeatable but challenging

## Known Limitations & Constraints

- **Single-file structure**: Keeping code in one HTML file for easy deployment
- **No persistent save**: Game state resets on page reload
- **Canvas only**: No WebGL or external rendering libraries
- **Vanilla JS**: No build tools, bundlers, or transpilation
- **Mobile-first responsive**: Must work on various screen sizes

## Debugging Tips

- Use `console.log()` for debugging game state
- Check DevTools Performance tab for frame rate drops
- Use Canvas context methods (strokeRect, fillRect) to visualize hitboxes
- Monitor memory usage in DevTools Memory tab
- Test touch controls with Chrome DevTools mobile emulation

## Future Enhancement Areas

Potential features for future development:

- Multiple game stages/levels
- High score persistence (localStorage)
- Difficulty settings
- Sound volume controls
- Alternative control schemes
- Power-up systems
- Combo multipliers
- Leaderboard system

## Resources & References

- HTML5 Canvas API: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API
- Game Development Best Practices: Focus on game feel, player feedback, and responsive controls
- Performance: Test frequently on target devices

## Questions for Claude Code

When working on this codebase, use these guidelines to:

1. Maintain code consistency with existing patterns
2. Keep performance in mind when adding features
3. Test game functionality thoroughly before committing
4. Prefer small, focused commits over large monolithic changes
5. Maintain the vanilla JavaScript approach for simplicity and portability

---

**Last Updated**: 2026-09-26  
**Project**: Sky Fighter - Raiden-style Vertical Shoot-em-up  
**Status**: Active Development

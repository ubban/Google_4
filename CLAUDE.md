# Claude Development Guidelines for Sky Fighter

## Project Overview

Sky Fighter is a Raiden-style vertical shoot-'em-up game built with vanilla JavaScript and HTML5 Canvas. The game is implemented in a single `index.html` file containing embedded CSS and JavaScript.

## Architecture

- **Single-file architecture**: All code (HTML, CSS, JS) is in `index.html`
- **Canvas-based rendering**: Uses HTML5 Canvas API for all graphics
- **Game loop**: Implements a continuous animation frame-based game loop
- **Entity system**: Players, enemies, projectiles managed as arrays
- **Weapon system**: Multiple weapon types with switching capability
- **Boss battles**: Stage-based gameplay with boss encounters

## Development Guidelines

### Code Style

1. **Vanilla JavaScript**: No external dependencies. Keep it pure ES6+ where reasonable.
2. **Comments**: Only add comments for non-obvious logic, hidden constraints, or workarounds.
3. **Variable naming**: Use clear, descriptive names (e.g., `playerBullets`, `bossHealth`, not `pb` or `h`).
4. **Function organization**: Group related functions together logically.
5. **Spacing**: Maintain consistent indentation and blank lines between logical sections.

### When Making Changes

1. **Test in browser**: Always test gameplay changes by running the HTML file in a browser before considering work complete.
2. **Golden path testing**: Verify basic gameplay works (movement, shooting, enemy spawning, collisions).
3. **Edge cases**: Test boundary conditions (screen edges, extreme values, rapid interactions).
4. **Performance**: Be mindful of array operations in the game loop; avoid unnecessary object creation per frame.
5. **Visual regression**: Check that visual elements (HUD, canvas rendering, animations) still display correctly.

### Game Mechanics to Preserve

- Player movement with arrow keys or WASD
- Weapon switching (number keys or click to cycle)
- Smart bomb functionality
- Enemy spawning and wave progression
- Boss encounters with stage progression
- Health/lives system with visual feedback (hearts)
- Score and multiplier mechanics
- Collision detection (bullets/enemies, enemy/player)

### No Breaking Changes

- Do NOT remove weapon types or alter core game mechanics without explicit request
- Do NOT change control scheme without clear justification
- Do NOT modify boss encounter design in ways that make bosses trivial or impossible
- Do NOT alter the single-file structure without discussion

### Refactoring Limits

- Extract functions for clarity, but keep the single-file structure
- Do NOT introduce build steps, bundlers, or external dependencies
- Do NOT create separate files unless explicitly requested
- Do NOT add transpilation or module systems

## Commit Message Format

Follow conventional commits style:
- `feat: [description]` - New features (e.g., "feat: add triple-shot weapon")
- `fix: [description]` - Bug fixes (e.g., "fix: correct boss projectile collision")
- `improve: [description]` - Improvements to existing features (e.g., "improve: smooth player movement")
- `refactor: [description]` - Code reorganization without behavior change (e.g., "refactor: consolidate collision functions")
- `style: [description]` - Visual/styling changes (e.g., "style: enhance HUD glow effect")

Example:
```
feat: add enemy wave progression system

Implement escalating difficulty with enemy type variations per wave.
Adds wave counter to HUD and dynamic spawn rate increase.
```

## Testing Checklist Before Pushing

- [ ] Game starts without console errors
- [ ] Player can move in all directions
- [ ] Primary weapon fires and behaves correctly
- [ ] All weapon switches work (if changing weapons)
- [ ] Enemies spawn and move predictably
- [ ] Collisions work correctly (bullets hit enemies, enemies hit player)
- [ ] Smart bombs function (if working on them)
- [ ] HUD displays correctly (health, score, weapon, multiplier)
- [ ] Game completes a full stage
- [ ] No performance degradation (smooth 60fps)

## Performance Considerations

- Avoid creating objects in the main game loop
- Reuse arrays when possible instead of creating new ones
- Monitor canvas rendering performance
- Keep collision detection efficient (spatial optimization if needed)
- Consider pooling for frequently created/destroyed entities (bullets, small effects)

## Known Limitations & Technical Debt

Document any known issues or areas for future improvement here as they arise.

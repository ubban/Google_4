# Claude Code Guidelines for Sky Fighter

## Project Overview
**Sky Fighter** is a Raiden-style vertical shoot-em-up game built with vanilla JavaScript, HTML5 Canvas, and CSS. The game features:
- Player aircraft with weapon switching mechanics
- Multiple enemy types with varying behaviors
- Stage bosses with dynamic attack patterns
- Smart bomb mechanics
- Score tracking and visual HUD
- Responsive design for desktop and mobile

## Architecture & Code Organization

### Main Structure
- **index.html**: Single-file application containing HTML, CSS, and JavaScript
- **Game Loop**: Uses `requestAnimationFrame` for 60fps gameplay
- **Entities**: Player, enemies, projectiles, and effects managed via object pools where appropriate

### Key Systems
1. **Input Handling**: Keyboard (arrow keys, space) and mouse/touch targeting
2. **Collision Detection**: Simple AABB (Axis-Aligned Bounding Box) based
3. **Particle Effects**: Explosions and visual feedback
4. **Audio**: Game sounds and background music (if implemented)

## Development Guidelines

### Code Style
- Use **vanilla JavaScript** — no build tools or frameworks unless absolutely necessary
- Variable names: `camelCase` for variables/functions, `UPPER_SNAKE_CASE` for constants
- Keep functions small and focused — aim for <50 lines per function
- Use object literals for configuration (weapon stats, enemy definitions, stage layouts)
- Comments should explain **why**, not what; code is self-documenting

### Game-Specific Conventions
- **Entity objects**: Store `x`, `y`, `vx`, `vy`, `width`, `height`, `health` consistently
- **Update pattern**: Call `update(dt)` before rendering; `dt` is delta time in seconds
- **Rendering**: Draw in a consistent order (background, entities, effects, UI)
- **Pooling**: Reuse projectile and particle objects to reduce garbage collection pauses

### Adding Features
1. **New Weapons**: Define in a `weapons` object with properties like `fireRate`, `damage`, `speed`
2. **New Enemy Types**: Add to `enemyTypes` object; implement `update()` and `render()` methods
3. **Boss Patterns**: Encapsulate boss AI in a state machine or pattern generator function
4. **Visual Feedback**: Use particle effects and screen shake rather than static animations

## Testing & Validation

### Manual Testing Checklist
- [ ] Player movement smooth on desktop and mobile
- [ ] All weapons fire correctly with proper damage
- [ ] Enemies spawn and behave as expected
- [ ] Collisions detected accurately (no "ghost" hits or missed hits)
- [ ] Score updates reflect enemy kills
- [ ] Boss stages load and patterns execute
- [ ] Smart bombs clear enemies as intended
- [ ] Game over triggers correctly
- [ ] Responsive on various screen sizes (phone, tablet, desktop)

### Performance Considerations
- Monitor frame rate using DevTools (target: 60 FPS)
- Use object pooling for frequently created/destroyed objects
- Minimize reflows/repaints in render loop
- Cache DOM references if accessing them frequently

## Git & Commit Conventions

### Branch Strategy
- Work on feature branches: `claude/` prefix for Claude-authored changes
- Feature branches merge to `main` via pull request
- Commit format: Clear, imperative messages ("Add smart bomb mechanic" not "Fixed stuff")

### Commit Messages
- First line: Short (50 chars max), imperative mood
- Body (optional): Explain why the change was made, not what changed
- Example:
  ```
  Add multi-shot weapon type
  
  Implements a fan-shaped spread shot that hits multiple targets.
  Balances against single-shot weapons by reducing individual damage.
  ```

## Common Patterns & Utilities

### Entity Update Pattern
```javascript
function update(dt) {
  this.x += this.vx * dt;
  this.y += this.vy * dt;
  // Boundary checks, collision checks, etc.
}
```

### Collision Detection
```javascript
function checkCollision(a, b) {
  return a.x < b.x + b.width &&
         a.x + a.width > b.x &&
         a.y < b.y + b.height &&
         a.y + a.height > b.y;
}
```

### Frame-Rate Independent Timing
Always multiply movement/animation by `dt` (delta time) to ensure smooth gameplay regardless of frame rate.

## Known Limitations & Future Considerations
- Single HTML file may grow large with additional features; consider refactoring to modules if needed
- Mobile performance optimizations may be required for lower-end devices
- Audio implementation follows platform audio policies (autoplay restrictions)

## For Code Review

### What to Check
- Performance: Are pooling patterns used for frequent allocations?
- Correctness: Are collision and damage calculations accurate?
- Responsiveness: Does UI/input feel responsive on all devices?
- Readability: Are variable names clear and functions properly scoped?

### What Not to Change Without Discussion
- Game balance (weapon damage, enemy spawn rates, difficulty curves)
- Asset loading strategy (inline vs. external)
- Core game loop timing mechanisms

## Resources & References
- [Canvas API Docs](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)
- [requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame)
- [Touch Events](https://developer.mozilla.org/en-US/docs/Web/API/Touch_events)

---

**Last Updated**: 2026-09-26
**Maintained By**: Claude Code Agent

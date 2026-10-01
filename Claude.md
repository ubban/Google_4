# Claude.md — Sky Fighter Development Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla JavaScript and HTML5 Canvas. Players control a fighter aircraft, switch between weapons, deploy smart bombs, and defeat stage bosses.

**Key Technologies:**
- Vanilla JavaScript (ES6+)
- HTML5 Canvas for rendering
- No external dependencies

## Repository Structure

- `index.html` — Single-file HTML/CSS/JavaScript application containing the complete game
- `README.md` — Project metadata

## Architecture & Game Systems

The game is implemented as a monolithic HTML file with the following major systems:

### Rendering & Canvas
- Central game loop using `requestAnimationFrame`
- Canvas-based 2D rendering with gradients and effects
- Responsive viewport with viewport-fit support for mobile notches

### Game State Management
- Player object: position, health, weapons, bombs
- Enemy spawning and wave management
- Projectile and explosion systems
- Collision detection

### Input Handling
- Mouse/touch input for aiming
- Keyboard controls for movement and weapon switching
- Mobile touch support with event handling

### Visual Effects
- HUD (heads-up display) with stats overlay
- Damage vignette effect with red pulsing animation
- Screen flash effects for hits and bombs
- Heart beat animation for critical health

### Audio & Feedback
- Text-based visual feedback (no audio assets required)

## Development Workflow

### Before Making Changes
1. Review the relevant section of `index.html` — the file is single-file, so context is important
2. Test changes in a browser by opening `index.html` locally
3. Check both desktop and mobile viewports

### Common Tasks

#### Adding/Modifying Game Features
- Weapon balance: Search for weapon-related constants and damage values
- Enemy patterns: Look for wave definitions and enemy spawn logic
- Visual effects: Modify CSS keyframes or Canvas rendering calls
- HUD updates: Edit the display stat elements

#### Debugging
- Use browser DevTools console to inspect game state
- Canvas rendering issues often relate to coordinate systems or z-order
- Performance: Profile with DevTools Performance tab for frame rate issues

#### Testing
- Test gameplay loop: spawn enemies, fire weapons, take damage
- Test mobile: verify touch input works with viewport scaling
- Test edge cases: screen boundaries, rapid input, many projectiles
- Test bosses: verify boss patterns and difficulty scaling

## Code Patterns & Guidelines

### Naming
- Use camelCase for functions and variables
- Use UPPERCASE for constants
- Descriptive names for game entities (player, enemies, projectiles)

### Performance
- Minimize DOM manipulation — game logic is primarily Canvas-based
- Use object pooling for frequently created/destroyed entities (projectiles, explosions)
- Avoid layout thrashing; batch DOM updates

### Responsive Design
- Use `clamp()` CSS for scalable font sizes: `clamp(12px, 2.8vw, 16px)`
- Respect safe areas with `env(safe-area-inset-*)`
- Test canvas scaling on different aspect ratios

### Browser Compatibility
- Target modern browsers (Chrome, Firefox, Safari, Edge)
- Use standard Web APIs (Canvas, Touch Events, RequestAnimationFrame)
- Graceful degradation for older viewports

## Interaction with Claude Code

### Using Claude for Tasks
- Ask Claude to review game mechanics for bugs or balance issues
- Request visual effect improvements or new animations
- Ask for refactoring to improve maintainability (while keeping single-file structure)
- Discuss game design and feature ideas

### What Claude Should Not Do
- Do not split the single HTML file into multiple files without explicit approval
- Do not add external dependencies (libraries, frameworks) without discussion
- Do not remove features without understanding gameplay impact

## Testing & Verification

When making changes:
1. **Smoke test**: Open `index.html` in a browser, verify the game starts
2. **Functional test**: Test the specific feature that was changed
3. **Regression test**: Verify unrelated features still work (movement, combat, HUD)
4. **Mobile test**: If touch-related, test on mobile or use browser DevTools device emulation

## Documentation & Attribution

All commits should include clear descriptions of changes. When using Claude Code:
- Include commit attribution footer as configured
- Reference specific line numbers when discussing code
- Keep commit messages focused and descriptive

## Future Considerations

- Consider Canvas rendering optimizations if performance becomes an issue
- Mobile input refinements based on user feedback
- Potential asset management system if adding graphics/audio
- Score persistence with localStorage if desired

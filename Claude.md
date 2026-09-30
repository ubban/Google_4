# Claude.md - Sky Fighter Game Development Guidelines

## Project Overview

This repository contains **Sky Fighter**, a Raiden-style vertical shoot-em-up game built with vanilla JavaScript, HTML5 Canvas, and CSS. The game features player-controlled aircraft, enemies, projectiles, and score tracking.

## Development Approach

### When Working on This Codebase

1. **Minimize Refactoring**: Only refactor code when explicitly requested or when a task requires it. Avoid preemptive abstractions or "improvements" that aren't part of the current request.

2. **Preserve Existing Functionality**: When making changes, thoroughly test to ensure no regressions in game mechanics (player movement, shooting, collision detection, scoring, etc.).

3. **Keep It Simple**: Prefer straightforward implementations over complex patterns. A clear, working solution is better than an elegant but unclear one.

4. **Comments**: Only add comments when the "why" is non-obvious or when documenting workarounds for specific issues. Well-named variables and functions should explain themselves.

## Code Style and Conventions

- **JavaScript**: Use modern ES6+ syntax (arrow functions, const/let, template literals)
- **HTML**: Semantic HTML5 elements, id-based element references for game canvas
- **CSS**: Use CSS Grid/Flexbox for layouts, avoid excessive nesting
- **Naming**: Use clear, descriptive names:
  - PascalCase for classes (e.g., `Player`, `Enemy`)
  - camelCase for functions and variables
  - UPPER_CASE for constants

## Game Architecture

The codebase uses an entity-based architecture:
- **Player**: Handles player aircraft movement and firing
- **Enemy**: Manages enemy behavior, spawning, and movement patterns
- **Projectile**: Tracks bullets/missiles and collision detection
- **Game Loop**: Canvas-based render loop with update and draw phases

Key game mechanics:
- Player controls (arrow keys or WASD)
- Automatic or manual shooting
- Collision detection between player, enemies, and projectiles
- Score tracking and level progression

## Testing and Validation

1. **Manual Testing**: Before submitting changes, test the game in a browser:
   - Player movement and boundaries
   - Firing and projectile behavior
   - Enemy spawning and movement
   - Collision detection (hit feedback)
   - Score updates and game state

2. **Edge Cases**: Pay attention to:
   - Canvas boundary collisions
   - Rapid input handling
   - Large numbers of entities on screen
   - Game state transitions (start, paused, game over)

## Commit Messages

When committing changes, use descriptive messages that explain the "why":
- Good: `Add power-up system for rapid-fire capability`
- Avoid: `Fix stuff` or `Update game`

Include the attribution footer for all commits:
```
Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
Claude-Session: <session-url>
```

## Working with HTML Canvas

The game uses the HTML5 Canvas API. Key considerations:
- Canvas coordinates: (0,0) is top-left, X increases right, Y increases down
- All drawing must happen within each frame's update
- Transform matrices should be reset or managed carefully
- Collision detection uses bounding boxes or distance calculations

## Performance Considerations

- Limit the number of active entities to maintain 60 FPS
- Use object pooling or recycling for frequently created objects
- Avoid creating new objects in tight loops
- Profile with browser DevTools when frame rate drops

## No Feature Creep

Unless explicitly requested:
- Don't add new game modes
- Don't implement sound effects or music
- Don't add UI screens beyond what's needed
- Don't refactor for "future extensibility"

Focus on completing the requested feature or fix cleanly.

## Permissions and Git Workflow

- Always develop on the designated feature branch (`claude/inspiring-planck-8jom7z`)
- Use `git push -u origin <branch-name>` when pushing
- Create commits as you complete logical units of work
- Do NOT push to main or master without explicit permission
- Do NOT create pull requests without being asked

---

Last updated: 2026-09-30

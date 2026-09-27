# Claude.md — Sky Fighter Codebase Guidelines

This document provides guidelines for Claude Code and AI agents working on the Sky Fighter project.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-page HTML5 application with vanilla JavaScript. The game features:
- Vertical scrolling gameplay with player ship control
- Weapon switching mechanics
- Smart bomb functionality
- Stage bosses and enemy waves
- HUD displaying score, health, and game state

## Project Structure

```
Google_4/
├── index.html          # Main game file (HTML + CSS + JavaScript)
├── README.md           # Project metadata
└── CLAUDE.md          # This file
```

The entire game logic, rendering, and UI are contained in `index.html`.

## Development Guidelines

### Code Style & Conventions

1. **Single File Architecture**: All game code is in `index.html`. When making changes:
   - Maintain clear separation between HTML structure, CSS styles, and JavaScript logic
   - HTML structure should be minimal and semantic
   - CSS should be scoped in the `<style>` tag
   - JavaScript should be in the `<script>` tag at the end

2. **JavaScript Conventions**:
   - Use camelCase for variables and functions
   - Use UPPER_SNAKE_CASE for constants
   - Keep functions focused and reasonably small
   - Add comments only for non-obvious logic (why, not what)

3. **Game Loop & Performance**:
   - The game uses `requestAnimationFrame` for the main loop
   - Minimize DOM manipulation; use canvas rendering where possible
   - Be mindful of memory allocation in the game loop
   - Profile performance changes, especially in tight loops

### Testing & Validation

1. **Manual Testing**: 
   - Always test gameplay changes in a browser before committing
   - Verify core mechanics: movement, shooting, weapon switching, bombs
   - Test on multiple screen sizes (desktop, tablet, mobile)
   - Check touch controls on mobile/tablet devices

2. **Regression Testing**:
   - Ensure existing game mechanics still work after changes
   - Test both keyboard and mouse/touch input methods
   - Verify HUD updates correctly (score, health, state)

### Git Conventions

1. **Commit Messages**:
   - Use clear, descriptive messages
   - Start with a verb: "Add", "Fix", "Refactor", "Update"
   - Reference game features: e.g., "Fix weapon switching lag", "Add new boss pattern"
   - Keep commits atomic and focused

2. **Branch Naming**:
   - Feature branches: `feature/descriptive-name`
   - Bug fix branches: `fix/issue-description`
   - Example: `feature/new-weapon-type`, `fix/collision-detection`

3. **Pull Requests**:
   - Describe what was changed and why
   - Include test results and any performance implications
   - Reference any related issues

## Game Architecture Notes

### Core Components

- **Game State**: Tracked via global variables (score, health, level, game phase)
- **Rendering**: Canvas-based 2D rendering in the game loop
- **Input Handling**: Keyboard, mouse, and touch event listeners
- **Entity System**: Player, enemies, projectiles, effects managed in update loops

### Key Files to Understand

All game code is in `index.html`. Key sections to understand:

1. **HTML Structure** (top of file):
   - Container div for game canvas
   - HUD element for displaying game information
   - Canvas element for rendering

2. **CSS Styles** (`<style>` tag):
   - Responsive layout and mobile support
   - Safe area insets for notched devices
   - Canvas styling and positioning

3. **JavaScript** (`<script>` tag):
   - Configuration constants
   - Game state variables
   - Update functions (per-frame logic)
   - Rendering functions (canvas drawing)
   - Input handlers (keyboard, mouse, touch)
   - Game initialization

## Common Tasks

### Adding a New Feature

1. Identify which section of the JavaScript handles the feature area
2. Add necessary state variables
3. Implement update logic in the game loop
4. Add rendering code
5. Handle input if needed
6. Test thoroughly in a browser

### Fixing a Bug

1. Reproduce the issue in a browser
2. Use browser DevTools to inspect state and log values
3. Locate the relevant code section
4. Implement the fix
5. Test the fix and verify no regressions
6. Commit with a clear message

### Performance Optimization

1. Profile using Chrome DevTools Performance tab
2. Identify bottlenecks in rendering or update loops
3. Consider: reducing draw calls, batch updates, object pooling
4. Measure before and after
5. Commit with performance notes

## Important Constraints

1. **Single File Limitation**: The entire game must remain in `index.html` (do not split into multiple files)
2. **No External Dependencies**: The game uses only vanilla JavaScript (no libraries)
3. **Mobile Compatibility**: Must work on touch devices with safe area support
4. **Performance**: Game must maintain smooth 60 FPS on target devices

## When to Ask for Help

Reach out to the project maintainer when:
- Uncertain about the game design direction
- Major refactoring that changes core architecture
- Performance issues that seem architectural
- Questions about game balance or mechanics

## Tools & Resources

- Browser DevTools for debugging and performance profiling
- Canvas API documentation for rendering questions
- RequestAnimationFrame for game loop timing

---

Last updated: 2026-09-27

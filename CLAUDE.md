# Claude Code Guidelines for Sky Fighter

This document provides guidance for Claude Code agents working on the Sky Fighter project.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented in HTML/JavaScript. Features include:
- Vertical scrolling gameplay
- Weapon switching mechanics
- Smart bombs
- Stage bosses
- Mobile-friendly controls with touch support

## Repository Structure

- `index.html` - Main game file containing HTML structure, CSS styles, and embedded JavaScript
- `README.md` - Project documentation
- `.git/` - Git repository metadata

## Development Guidelines

### Code Organization
- All game logic is currently embedded in `index.html` within `<script>` tags
- CSS styling is in `<style>` tags within the HTML head
- Keep related game mechanics together (physics, rendering, input handling)

### JavaScript Patterns
- Use vanilla JavaScript (no external dependencies required for core game)
- Maintain clear separation between game logic, rendering, and input handling
- Use descriptive variable and function names
- Comments should explain non-obvious game mechanics or calculations

### Styling & UI
- Use CSS custom properties (CSS variables) for theming
- Ensure mobile responsiveness with appropriate viewport settings
- Use safe area insets for notched devices
- Test canvas rendering on various screen sizes

### Game-Specific Conventions
- Canvas-based rendering for performance
- Touch events alongside keyboard input for mobile compatibility
- Game state management should be clear and modular
- Physics calculations should use consistent units and gravity models

## Testing

No automated test suite currently exists. Manual testing should include:
- Gameplay on desktop browsers (Chrome, Firefox, Safari, Edge)
- Touch controls on mobile/tablet devices
- Boss encounters and difficulty progression
- Weapon switching and bomb mechanics
- Performance on lower-end devices

## Building & Running

The game runs directly as an HTML file - no build process required. Simply open `index.html` in a modern web browser.

## Git Workflow

All development should occur on the designated feature branch:
- **Feature Branch:** `claude/inspiring-planck-usph1g`
- Write clear, descriptive commit messages
- Include `Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>` footer in commits
- Push changes with: `git push -u origin <branch-name>`

## Common Tasks

### Adding Game Features
1. Identify the appropriate section in the JavaScript code
2. Implement the feature with clear variable/function naming
3. Test thoroughly on both desktop and mobile
4. Commit with descriptive message

### Bug Fixes
1. Reproduce the issue
2. Locate the problematic code
3. Fix with minimal changes
4. Test the fix doesn't break related functionality
5. Commit with clear explanation of what was fixed

### Performance Optimization
- Profile with browser DevTools
- Focus on rendering efficiency in the game loop
- Avoid unnecessary canvas redraws
- Optimize collision detection algorithms

## Code Style

- Use clear, readable code over clever one-liners
- Maintain consistent indentation (2 or 4 spaces)
- Comment game mechanics that aren't immediately obvious
- Use meaningful variable names (avoid single letters except for loops)

## Browser Compatibility

- Target modern browsers (Chrome, Firefox, Safari 12+, Edge)
- Use ES6 features as appropriate
- Test on mobile browsers (iOS Safari, Chrome Mobile)
- Ensure canvas rendering works smoothly across platforms

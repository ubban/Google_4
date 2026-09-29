# Claude.md - Sky Fighter Project Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla JavaScript and HTML5 Canvas. Players control an aircraft, switch weapons, drop smart bombs, and defeat stage bosses.

- **Tech Stack**: HTML5, CSS3, JavaScript (Vanilla)
- **Architecture**: Single HTML file with embedded JavaScript and CSS
- **Target**: Browser-based game (desktop and mobile support)

## Project Structure

- `index.html` - Main game file (HTML, CSS, and JavaScript combined)
- `README.md` - Basic project information
- `.git/` - Git repository

## Key Features

- Aircraft control with weapon switching
- Smart bomb mechanics
- Stage-based boss encounters
- Mobile-responsive controls (touch support)
- Dark theme UI with HUD display

## Development Guidelines

### Code Style & Standards

- Use vanilla JavaScript (no external frameworks or libraries)
- Maintain single HTML file structure
- Keep inline CSS organized with clear sections
- Use semantic HTML5 elements
- Implement touch and keyboard controls for broad compatibility

### Game Mechanics

- Player movement: Arrow keys or WASD (keyboard), touch/drag (mobile)
- Shooting: Spacebar or tap
- Weapon switching: Z/X keys or swipe gestures
- Smart bombs: Limited resource, strategic use

### Visual Design

- Dark blue color scheme (#05122b, #0b1d3a, #0d2f5e)
- Gradient sky background
- Glass-morphism style UI elements where appropriate
- Responsive design using CSS clamp() for fluid scaling

## How Claude Should Approach Tasks

### When Adding Features
1. Maintain the single-file structure unless specifically requested otherwise
2. Ensure mobile compatibility (touch controls, responsive sizing)
3. Test in both desktop and mobile viewports
4. Keep CSS organized and use CSS custom properties for reusable values
5. Document game state changes in comments

### When Fixing Bugs
1. Identify the root cause in the game loop or event handlers
2. Check for edge cases (mobile vs desktop, portrait vs landscape)
3. Verify collision detection and timing logic
4. Test with various input methods

### When Optimizing
1. Profile the game loop and rendering performance
2. Optimize canvas operations (batch draws, reduce repaints)
3. Minimize DOM manipulations
4. Consider performance on lower-end devices

## Testing & Running

- Open `index.html` in a modern web browser to run the game
- Test on mobile devices for touch controls
- Verify keyboard controls (arrow keys, WASD, spacebar, Z, X)
- Check responsive behavior at different viewport sizes

## Important Constraints

- No external dependencies allowed (keep it vanilla)
- Maintain browser compatibility (modern browsers with Canvas support)
- Preserve existing game balance and mechanics
- Mobile optimization is mandatory for any UI/UX changes

## Attribution

- Claude Code agent edits to this codebase should be properly attributed in commits
- Session information should be included in commit messages when applicable

---

Last updated: 2026-09-29

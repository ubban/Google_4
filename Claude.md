# Claude Code Guidelines for Google_4

This document provides guidance for Claude Code agents working on the Google_4 project.

## Project Overview

**Google_4** is a Raiden-style vertical shoot-em-up game built with HTML5 and JavaScript. The project is a browser-based game featuring sprite-based graphics and real-time gameplay mechanics.

## Repository Structure

- `index.html` - Main game file containing HTML, CSS, and JavaScript
- `README.md` - Project metadata

## Development Guidelines

### Game Architecture

The game is implemented as a single-file HTML5 application with:
- Canvas-based rendering for sprites and game objects
- Game loop with update and render cycles
- Input handling for player controls
- Collision detection system
- Enemy spawning and AI patterns
- Score/UI management

### Key Considerations for Changes

1. **Canvas & Rendering**
   - Game uses HTML5 Canvas for 2D graphics
   - Sprite positioning and animation handled through coordinate updates
   - Render pipeline: clear → draw background → draw sprites → draw UI

2. **Game Loop**
   - Runs continuously with requestAnimationFrame
   - Update phase: process input, move objects, check collisions
   - Render phase: draw all sprites and UI elements

3. **Player Input**
   - Keyboard controls for movement (arrow keys, WASD, etc.)
   - Shooting mechanics tied to input or auto-fire
   - Input state management during active gameplay

4. **Gameplay Mechanics**
   - Player ship with health/lives system
   - Enemy sprites with movement patterns and shooting
   - Projectile system (player bullets, enemy bullets)
   - Collision detection between player, enemies, and projectiles
   - Score system with point rewards

5. **Performance**
   - Minimize DOM manipulation within game loop
   - Use canvas transforms efficiently
   - Keep collision detection optimized for many objects

### Testing Guidelines

1. **Visual Testing**
   - Open `index.html` in a modern browser to verify gameplay
   - Test player movement and shooting mechanics
   - Verify enemy spawning and behavior patterns
   - Check collision detection accuracy
   - Confirm score updates and UI rendering

2. **Gameplay Testing**
   - Test difficulty progression if implemented
   - Verify enemy wave patterns
   - Test game over and restart mechanics
   - Check for performance issues with many sprites

3. **Cross-browser Compatibility**
   - Test in Chrome/Chromium, Firefox, Safari
   - Verify canvas API support
   - Test keyboard input handling

## Running the Game

Simply open `index.html` in a web browser. The game should start automatically or begin when a start button is clicked (depending on implementation).

## Making Changes

When modifying the game:

1. **Small fixes** (bug fixes, balance tweaks): Make changes directly
2. **Gameplay features** (new enemies, weapons, levels): Ensure collision detection and game loop integration
3. **UI improvements** (score display, menus): Test rendering in canvas
4. **Performance optimizations**: Profile before and after changes

## Code Style

- Use consistent indentation (2 or 4 spaces)
- Follow JavaScript best practices
- Keep game loop code efficient
- Add comments for complex algorithms (especially collision/AI logic)

## Common Tasks

### Adding a New Enemy Type
- Define new sprite graphics/coordinates
- Create enemy class or object with movement logic
- Add to spawning system
- Test collision detection with player bullets

### Adjusting Difficulty
- Modify spawn rates in enemy spawning code
- Adjust enemy health/damage values
- Change bullet speed or fire rates
- Update score multipliers

### Fixing Collision Issues
- Review collision detection math
- Verify hitbox coordinates for sprites
- Check if objects are being properly tracked
- Consider object lifecycle (creation/removal)

## Git Workflow

- Branch: `claude/inspiring-planck-pyars0`
- Commit changes with descriptive messages
- Push to the specified branch
- Don't force-push unless necessary

## Contact

For questions about the project structure or implementation details, review the game code in `index.html` directly.

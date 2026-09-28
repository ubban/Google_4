# Sky Fighter - Codebase Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file web application. The game is implemented entirely in HTML with embedded CSS and JavaScript, making it easily playable in any modern web browser.

**Repository**: ubban/Google_4
**Primary File**: index.html (single-file game implementation)

## Game Description

A classic-style vertical shoot-'em-up where players control an aircraft, switch between different weapon types, and use smart bombs to defeat enemies and stage bosses. The game features:

- **Player Aircraft**: Controlled via mouse/touch, can switch between multiple weapons
- **Weapon System**: Different weapon types with varying damage and patterns
- **Smart Bombs**: Limited-use special attacks for clearing large enemy groups
- **Enemy Stages**: Multiple stages with bosses
- **UI/HUD**: Real-time display of health, ammunition, bombs, weapon status, and multiplier
- **Visual Effects**: Explosions, screen vignette on damage, flash effects on hits
- **Responsive Design**: Adapts to different screen sizes and devices (desktop, mobile)

## Technical Architecture

### Single-File Structure
The entire game is implemented in one `index.html` file with three integrated sections:

1. **HTML Markup** (lines 1-200+)
   - Canvas element for rendering
   - HUD display elements (health, bombs, weapon info, score)
   - Flash and vignette effect elements

2. **CSS Styling** (embedded in `<style>` tags)
   - Dark theme with space/sky gradient backgrounds
   - Responsive sizing with `clamp()` for scaling across devices
   - Animation keyframes for heart beat, vignette pulse, flashes
   - Safe area insets for mobile notches
   - Touchscreen optimizations (no tap highlight, crosshair cursor)

3. **JavaScript Game Logic** (embedded in `<script>` tags)
   - Canvas rendering context
   - Game state management
   - Entity management (player, enemies, bullets, explosions)
   - Input handling (mouse, touch, keyboard)
   - Collision detection
   - Animation and timing loops
   - HUD updates

### Key Game Entities

- **Player**: Controllable aircraft with health, position, current weapon, bomb count
- **Enemies**: Various enemy types with health, spawning patterns, movement patterns
- **Bullets**: Player projectiles and enemy projectiles with damage values
- **Explosions/Effects**: Visual feedback for hits and bomb detonations
- **Bosses**: Stage bosses with unique behavior and higher health

## Development Guidelines

### When Modifying the Game

1. **Test in Browser**: Always test changes by opening index.html in a web browser before committing
2. **Mobile Compatibility**: Verify that changes work on touch devices (use browser dev tools to simulate)
3. **Performance**: The game runs on a 60 FPS canvas loop; avoid expensive operations
4. **Responsive Design**: Keep scaling and responsive behavior in mind with `clamp()` and viewport units
5. **Safe Areas**: Maintain safe area insets for devices with notches/rounded corners

### Common Tasks

- **Adding New Enemy Types**: Modify the enemy spawning logic in the JavaScript section
- **Adjusting Weapon Behavior**: Update weapon handling code and bullet creation
- **Tuning Difficulty**: Modify enemy health, spawn rates, and boss patterns
- **Visual Changes**: Update CSS for colors, shadows, animations
- **Bug Fixes**: Locate affected logic in the JavaScript section and test thoroughly

### Code Organization Within the File

The JavaScript code is organized conceptually into sections (though all in one `<script>` tag):
- Initialization and setup
- Game state variables
- Canvas and rendering setup
- Event listeners (mouse, touch, keyboard)
- Game loop (update and draw)
- Entity management functions
- Collision detection
- Rendering functions
- Enemy/boss patterns

## Git Workflow

When making changes:
1. Make edits to index.html
2. Test the game locally in a browser
3. Commit with a clear message describing the change
4. Push to the designated feature branch: `claude/inspiring-planck-mjkqq7`

Example commit: `git commit -m "Adjust boss enemy health and spawn patterns for increased difficulty"`

## File Metadata

- **Created**: Initial commit
- **Last Major Update**: Sky Fighter game implementation added
- **Branch**: claude/inspiring-planck-mjkqq7
- **Format**: Single-file HTML5 game with embedded CSS and JavaScript

## Performance Considerations

- Canvas rendering is the primary performance factor
- The game runs at approximately 60 FPS on modern browsers
- Mobile devices may have lower frame rates; test on actual devices when possible
- Avoid creating excessive DOM elements or complex animations outside the canvas

## Browsers and Compatibility

- Target modern browsers (Chrome, Firefox, Safari, Edge)
- Mobile browsers fully supported with touch controls
- Uses standard Web APIs: Canvas 2D, requestAnimationFrame, touch/mouse events
- No external dependencies or libraries required

## Future Enhancement Areas

- Sound effects and music
- Particle effects for weapons
- Additional weapon types or power-ups
- Persistent high score tracking (localStorage)
- Multiple difficulty levels
- Leaderboard system
- Mobile app deployment

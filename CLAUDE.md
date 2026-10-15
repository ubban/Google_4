# CLAUDE.md — Sky Fighter Project

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single HTML5 file with Canvas rendering and JavaScript game logic. The game runs entirely in the browser with no external dependencies.

## Key Files

- **index.html** — Single-file implementation containing all HTML, CSS, and JavaScript
  - Game canvas rendering and event handling
  - Game loop with update/render cycle
  - Player ship controls and weapon systems
  - Enemy spawning and AI
  - Boss battles
  - HUD and UI elements

- **README.md** — Basic project information

## Project Structure & Architecture

### Game Components

1. **Player Ship**
   - Position-based movement (arrow keys or WASD)
   - Multiple weapon types (Primary, Secondary, Smart Bombs)
   - Health system with damage detection
   - Invulnerability frames after being hit

2. **Weapon Systems**
   - Primary weapon: Fast-firing bullets
   - Secondary weapon: Spread shots or beam
   - Smart bombs: Area-of-effect damage
   - Ammunition/cooldown management

3. **Enemies**
   - Various enemy types with different behaviors
   - Projectile attacks
   - Health points
   - Collision detection with player projectiles

4. **Boss Battles**
   - Stage bosses with phases
   - Health tracking and visual feedback
   - Special attack patterns

5. **Game State**
   - Stages/levels
   - Score tracking
   - Lives/health system
   - Game over and win conditions

## Code Patterns & Conventions

### Canvas Rendering
- Single canvas element (`#gameCanvas`) for all graphics
- Game loop using `requestAnimationFrame` for smooth 60 FPS
- 2D context used for drawing sprites, text, and effects

### Input Handling
- Keyboard events (KeyDown/KeyUp) for player movement and actions
- Touch events for mobile compatibility
- Arrow keys (↑↓←→) or WASD for movement
- Space bar or click for weapons/bombs

### Game Loop
- **Update phase**: Process input, update positions, check collisions
- **Render phase**: Clear canvas, draw sprites, render HUD

### Entity Management
- Arrays for enemies, projectiles, effects
- Collision detection via bounding box or distance checks
- Removal of destroyed entities from arrays

### Visual Effects
- Particle effects for explosions and impacts
- Screen shake on major events
- Text notifications for power-ups or events

## Running the Project

The game runs directly in any modern web browser:
1. Open `index.html` in a web browser
2. Click to start the game
3. Use arrow keys (↑↓←→) or WASD to move
4. Left-click or spacebar to fire primary weapon
5. Right-click for secondary weapon/smart bombs

No build step or server required — pure HTML5.

## Development Guidelines

### When Modifying the Game

1. **Maintain Single-File Structure** — Keep all code in index.html to preserve the simple deployment model
2. **Preserve Game Balance** — Any weapon or difficulty changes should maintain playability
3. **Test on Multiple Browsers** — Canvas rendering may vary slightly between browsers
4. **Mobile Compatibility** — Consider touch inputs and viewport scaling
5. **Performance** — Monitor canvas rendering performance; limit particle effects if needed

### Making Changes

- CSS is in the `<style>` tag
- JavaScript is in a `<script>` tag at the end of the HTML
- Game logic is organized into sections: initialization, game loop, input handling, rendering

### Constants & Configuration

Look for configuration values near the top of the JavaScript section:
- Canvas dimensions
- Player speed and weapon stats
- Enemy spawn rates and patterns
- Boss health and attack patterns
- Score multipliers

## Testing

Test changes by:
1. Opening index.html in a browser
2. Playing through at least one full stage
3. Testing all weapons and game mechanics
4. Verifying collision detection works
5. Checking mobile/touch controls if applicable

## Attribution

This project is maintained by the Google_4 team. Claude Code agents may assist with:
- Bug fixes and gameplay improvements
- Adding new features (enemies, weapons, stages)
- Performance optimizations
- Mobile/accessibility enhancements
- Code organization and refactoring (while maintaining single-file structure)

---

Last updated: 2026-10-15

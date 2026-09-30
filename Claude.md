# Sky Fighter - Codebase Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla JavaScript and HTML5 Canvas. Players pilot a fighter jet, switch weapons, drop smart bombs, and engage stage bosses in arcade-style combat.

**Tech Stack:**
- HTML5 Canvas for rendering
- Vanilla JavaScript (no frameworks)
- Single-file architecture (index.html)
- Responsive design with touch and keyboard support

## Quick Start

1. **Run locally:**
   ```bash
   # Open index.html directly in a modern browser
   # Or serve via HTTP server:
   python3 -m http.server 8000
   # Then visit http://localhost:8000
   ```

2. **Play:**
   - **Move:** Arrow keys or WASD (desktop) / Touch drag (mobile)
   - **Shoot:** Spacebar or left-click (desktop) / Touch screen (mobile)
   - **Switch Weapon:** Q/E keys or weapon buttons
   - **Smart Bomb:** B key

## Architecture

### Game Structure

The game is implemented as a single HTML file with embedded CSS and JavaScript. Key sections include:

- **Game Loop**: Canvas animation frame loop handling input, updates, rendering
- **Player Object**: Movement, weapon state, health, collision detection
- **Enemies**: Various enemy types with AI patterns, health, firing behavior
- **Bullets & Effects**: Projectiles, explosions, visual effects
- **Weapons System**: Multiple weapon types (laser, missile, etc.) with ammo/cooldown
- **HUD**: Health, score, weapon status, stage info display
- **Audio**: Sound effects and background music (if applicable)

### Key Variables & Objects

- `canvas`, `ctx`: Canvas element and 2D context
- `gameState`: Current game state (playing, gameover, etc.)
- `player`: Player aircraft object with position, health, weapons
- `enemies`: Array of active enemies
- `bullets`: Array of active projectiles
- `particles`: Visual effects (explosions, hits)

### Game States

- **Menu/Loading**: Initial state
- **Playing**: Active gameplay
- **Paused**: Game paused
- **GameOver**: Level or game complete
- **Victory**: Boss defeated

## Development Notes

### Styling
- Dark blue theme (#05122b, #0d2f5e)
- Responsive via viewport units (vw, vh, clamp)
- Mobile-optimized with safe-area-inset
- Crosshair cursor during gameplay

### Mobile Support
- Touch events for movement and firing
- Full-screen capable (viewport-fit: cover)
- No horizontal scroll on mobile
- Tap highlights disabled for smooth interaction

### Performance Considerations
- Canvas rendering optimized for 60fps
- Efficient collision detection (spatial partitioning if many objects)
- Object pooling for bullets/effects (if implemented)
- Minimal DOM manipulation outside canvas

## Testing

### Manual Testing Checklist

- [ ] Player movement responds smoothly (desktop: keyboard/mouse, mobile: touch)
- [ ] Weapon firing and switching works
- [ ] Enemies spawn and behave correctly
- [ ] Collision detection (player-bullet, bullet-enemy, player-enemy)
- [ ] Score and health display correctly
- [ ] Game over/victory conditions trigger
- [ ] Smart bomb effect works
- [ ] Game restarts cleanly
- [ ] Responsive on different screen sizes

### Browser Compatibility

Target: Modern browsers supporting:
- ES6+ JavaScript
- Canvas API
- Touch Events API
- requestAnimationFrame

## Common Tasks

### Adding a New Enemy Type

1. Define enemy object structure (position, health, AI)
2. Create spawn logic in enemy manager
3. Add collision/damage handling
4. Define firing pattern if applicable
5. Test spawn rate and difficulty scaling

### Adding a New Weapon

1. Define weapon properties (damage, fire rate, spread, etc.)
2. Add to weapon selection menu/UI
3. Implement firing logic in player update
4. Add visual effect (bullet sprite/animation)
5. Balance cost/ammo if applicable

### Adjusting Difficulty

- Enemy spawn rates and health
- Enemy projectile speed and accuracy
- Boss attack patterns
- Player hit points
- Weapon power/cooldown times

## Notes for Claude Code

- This codebase is self-contained in a single HTML file
- Changes should maintain responsive design across mobile/desktop
- Keep performance in mind for canvas rendering
- Test on real devices when changing touch/input handling
- Consider accessibility (color contrast, readable fonts)

## Future Enhancements

Potential features to add:
- Multiple stages with increasing difficulty
- Power-ups and upgrades
- Leaderboard/score persistence
- Sound effects and music
- Visual polish (particle effects, screen shake)
- Mobile app packaging (Cordova, Electron)

---

**Last Updated:** 2026-09-30
**Repository:** ubban/Google_4

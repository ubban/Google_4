# Sky Fighter - Game Development Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up (shmup) game built with vanilla HTML5 Canvas and JavaScript. It's a single-file web application that runs directly in a browser.

## Game Mechanics

- **Player Control**: Move with mouse/touch, auto-fire with left click
- **Weapon System**: Three weapon types that cycle with spacebar or button click
- **Smart Bombs**: Area-of-effect damage dealt by pressing B or dedicated button
- **Enemy Types**: Various enemy behaviors and difficulty levels
- **Boss Fights**: Stage bosses with unique attack patterns
- **Scoring System**: Points for destroyed enemies, multiplier bonuses
- **Lives System**: Player starts with 3 lives, ends game on loss

## File Structure

```
Google_4/
├── README.md          # Basic project info with timestamp
├── index.html         # Complete game implementation (all-in-one file)
│   ├── HTML structure (game canvas, HUD, mobile controls)
│   ├── CSS styling (dark blue space theme, responsive layout)
│   └── JavaScript game engine (inline, ~3000+ lines)
└── .git/              # Git repository
```

## Architecture Notes

The game is implemented as a single HTML file with:
- **Canvas rendering**: 2D canvas for game graphics
- **Game loop**: requestAnimationFrame-based update/render cycle
- **Object-oriented design**: Separate classes for Player, Enemy, Projectile, Boss, Particle effects
- **Touch & Mouse input**: Dual input system for mobile and desktop
- **Responsive layout**: Scales to device viewport with safe area insets

## Key Game Systems

### Game State Management
- Game modes: Menu, Playing, Paused, Game Over
- Wave/stage progression system
- Score and combo tracking

### Rendering Pipeline
1. Background gradient rendering
2. Entity updates (position, velocity, life)
3. Collision detection
4. Particle effect rendering
5. HUD overlay rendering

### Entity System
- **Player**: Single aircraft with weapons and health
- **Enemies**: Multiple types with different patterns (straight, diagonal, circular movement)
- **Bosses**: Multi-phase bosses with special attack patterns
- **Projectiles**: Bullets from player and enemies
- **Particles**: Explosion effects and visual feedback

### Weapon System
- Weapon 1: Standard rapid-fire bullets
- Weapon 2: Spread shot (multiple angles)
- Weapon 3: Heavy cannons with wider spread
- Smart bombs: Area damage affecting all visible enemies

## Development Guidelines

### When Adding Features
- Keep the game loop performant (target 60 FPS)
- Add new enemy types as classes extending the Enemy base
- New bosses should have clearly defined phases
- Weapon balance: test fire rates and damage to avoid breaking difficulty curve

### Testing Changes
- Test gameplay at different difficulty levels
- Verify mobile touch controls work properly
- Check performance on lower-end devices (canvas operations can be heavy)
- Ensure particle effects don't cause frame drops

### Code Organization
- Game initialization and configuration at top
- Entity classes in middle section
- Game state and loop logic at bottom
- HUD rendering and UI near the end

## Browser Compatibility
- Requires HTML5 Canvas support
- Touch events for mobile (iOS, Android)
- Mouse events for desktop
- Modern browser (Chrome, Firefox, Safari, Edge)

## Performance Considerations
- Canvas rendering is synchronous and single-threaded
- Particle pooling to avoid garbage collection during gameplay
- Consider limiting particle counts if performance degrades
- Enemy AI uses simple calculations to stay performant

## Debugging Tips
- Open browser console (F12) for any error messages
- Check network tab if assets load (all assets are inline, no external files)
- Use requestAnimationFrame timing to verify FPS
- Test weapon fire rates with console timing
- Verify collision detection by enabling debug mode (if implemented)

## Future Enhancement Ideas
- Difficulty scaling (easier early waves, harder late)
- Power-ups system (shield, rapid fire, bigger bombs)
- Multiple player characters with different stats
- Online high-score leaderboard
- Sound effects and music (requires audio integration)
- Campaign progression with story elements

## Contributing
When working on this project:
1. Make targeted changes that don't break existing gameplay
2. Test gameplay before committing
3. Keep performance in mind (60 FPS target)
4. Document significant new features or systems
5. Maintain consistency with existing code style (camelCase, class-based entities)

---

**Last Updated**: 2026-09-24  
**Repository**: ubban/google_4  
**Main Branch**: main  
**Development Branch**: claude/inspiring-planck-dzczyl

# Sky Fighter – Codebase Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up arcade game built in vanilla HTML5/JavaScript/Canvas. The game features a player-controlled fighter, weapon switching mechanics, smart bombs, enemy spawning waves, boss battles, and dynamic visual feedback.

- **Repository**: ubban/google_4
- **Main File**: `index.html` (single-file HTML5 application)
- **Type**: HTML5 Game (Canvas-based)
- **Target**: Modern web browsers (desktop and mobile)

## Project Structure

```
Google_4/
├── index.html         # Complete game (single file)
├── README.md          # Project readme
└── .git/              # Git repository
```

## Key Features

### Core Gameplay
- **Player Ship**: Controllable via arrow keys (desktop) or touch/tilt (mobile)
- **Weapon System**: 
  - Vulcan (rapid-fire primary weapon)
  - Homing Missile (seeks enemies)
  - Spreadshot (wide arc of projectiles)
- **Smart Bombs**: Limited-use area-effect attacks
- **Enemy Waves**: Progressive difficulty with various enemy types
- **Boss Battles**: Stage bosses with unique behavior patterns
- **Scoring System**: Points multiplier for consecutive hits without getting damaged

### Visual Polish
- Gradient sky background
- Particle effects for explosions
- Hit flash effects (white screen vignette)
- Damage vignette (red pulsing edge)
- HUD with stats (lives, bombs, weapon, multiplier)
- Responsive canvas scaling for different screen sizes

### Mobile Support
- Touch controls (movement)
- Device orientation support
- Safe area insets (notch/cutout compatibility)
- Viewport configuration for mobile devices

## Game Mechanics

### Controls
- **Keyboard**: Arrow keys to move, Space for primary weapon, E for homing, R for spreadshot, Q for bomb
- **Mobile**: Touch to move, UI buttons or key mappings

### Enemy Types
- Standard fighters (weak, gives 100 points)
- Stronger variants (tougher)
- Boss patterns with phase changes

### Progression
- 5 stages with increasing difficulty
- Boss encounter at end of each stage
- Continuous scoring with hit multiplier system

## Technical Details

### Architecture
- Single-file HTML application with embedded styles and JavaScript
- Canvas-based 2D rendering
- Event-driven input handling
- Game loop with fixed timestep updates
- Collision detection via bounding boxes

### Canvas Setup
- Adaptive sizing: attempts 512x768px default, scales with window
- Uses `requestAnimationFrame` for smooth animation
- Touch-action disabled to prevent browser defaults
- Crosshair cursor over game canvas

### Key Game Objects
- **Player**: Position, velocity, health, weapon state, bomb count
- **Projectiles**: Position, velocity, type (vulcan, homing, spread)
- **Enemies**: Position, velocity, health, pattern (standard or boss)
- **Particles**: Temporary visual effects (explosions, debris)

### State Management
- Game state (playing, paused, game over, boss intro)
- Wave/stage tracking
- Difficulty scaling based on stage
- Combo/multiplier tracking

## How to Work on This

### Running the Game
1. Open `index.html` in a modern web browser
2. Game starts immediately with Stage 1
3. Test on desktop (keyboard controls) and mobile (touch controls)

### Making Changes

**Before Editing:**
- Keep all code inside the single `index.html` file (for simplicity)
- Changes to game mechanics, visuals, or controls go in the embedded `<script>` section
- CSS changes go in the `<style>` section

**Common Tasks:**
1. **Adjust Difficulty**: Modify enemy spawn rates, health values, or boss patterns in the game loop
2. **Add/Change Weapons**: Update weapon definitions in the weapon system object
3. **Tweak Visuals**: Modify colors, particle effects, or animation timings in the render loop
4. **Balance Gameplay**: Adjust player speed, projectile damage, enemy damage values
5. **Add Sound** (future): Would need Web Audio API integration

### Testing Checklist
- [ ] Keyboard controls work (arrow keys, Space, E, R, Q)
- [ ] Mouse/touch movement works on mobile
- [ ] All weapons switch and fire correctly
- [ ] Smart bombs clear enemies and damage bosses
- [ ] Boss patterns trigger at stage end
- [ ] Score multiplier increases on hits, resets on damage
- [ ] Game over triggers at 0 health
- [ ] Canvas scaling works on different screen sizes
- [ ] HUD displays correctly (lives, bombs, weapon, multiplier)

### Git Workflow

- **Main Branch**: `main` – stable release
- **Development Branch**: `claude/inspiring-planck-xxb4ma` – active development
- **Commits**: Use clear messages describing changes (e.g., "Balance boss health", "Add spreadshot weapon")
- **Attribution**: End commits with `Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>` and session URL

### Performance Notes

- **Canvas Size**: Larger canvas (>800px) may impact mobile performance; monitor FPS
- **Particle Count**: Limit simultaneous particles to ~200 for smooth 60fps
- **Collision Checks**: O(n²) against enemies; consider spatial partitioning if >100 enemies
- **Drawing**: Use clear() before redraw; batch draw calls where possible

## Important Game Constants

These can be adjusted to tune gameplay:

- **Player Speed**: Movement velocity (pixels/frame)
- **Weapon Damage**: Points per projectile hit
- **Enemy Health**: Durability per enemy type
- **Boss Health**: Total health pools for stage bosses
- **Spawn Rate**: Enemies spawned per wave
- **Bomb Count**: Starting bombs (typically 3-4)
- **Difficulty Scaling**: Stage-based multiplier for enemy toughness

## Future Enhancement Ideas

- Sound effects and music
- Particle effect improvements
- Leaderboard / high score persistence (localStorage)
- Additional weapon types or power-ups
- More boss patterns and variety
- Screen shake effects on hits
- Combo/chain bonuses
- Tutorial or story mode
- Accessibility options (colorblind mode, larger UI)

## Debugging

### Common Issues
1. **Game not starting**: Check browser console for errors (F12)
2. **Controls not responding**: Verify event listeners are attached to canvas
3. **Performance drops**: Profile with DevTools; check particle/enemy counts
4. **Canvas sizing wrong**: Verify viewport meta tag and window size calculation

### Debug Tips
- Open browser DevTools (F12) to see console output
- Use `console.log()` for debugging game state
- Test on multiple screen sizes and browsers
- Mobile testing: Use device preview in DevTools or actual device

## Dependencies

- **None** – Pure HTML5/JavaScript/Canvas
- No external libraries or frameworks
- Works offline once loaded

## Browser Compatibility

Tested on:
- Chrome/Edge (latest)
- Firefox (latest)
- Safari (latest, iOS included)
- Mobile browsers (Chrome Android, Safari iOS)

Requires:
- ES6+ JavaScript support
- Canvas 2D API
- Touch Events API (for mobile)

---

Last Updated: 2026-10-16

# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with HTML5 Canvas. The game is a single-file implementation (index.html) featuring:
- Player-controlled aircraft that can move around the screen
- Multiple weapon types that can be switched during gameplay
- Smart bombs for crowd control
- Stage bosses with unique attack patterns
- Score tracking with multiplier system
- Health/lives system
- Responsive touch and keyboard controls

## Architecture

The entire game is implemented in a single `index.html` file (~59KB) with:
- Embedded HTML structure defining game canvas and HUD elements
- Inline CSS styling for responsive design and animations
- Inline JavaScript containing all game logic

### Key Sections in index.html

1. **HTML Structure (lines 1-150+)**
   - Game container with canvas element for rendering
   - HUD elements: health display (#hud), weapon stats, score, multiplier
   - Flash and vignette effects for visual feedback
   - Fully responsive design with safe-area insets for mobile

2. **CSS Styling (lines 10-120+)**
   - Dark theme with space/sci-fi aesthetic (#05122b background)
   - Canvas styling with gradient background (sky effect)
   - HUD styling with glass-morphism effects (blur backdrop)
   - Animation keyframes for visual effects:
     - `heartBeat`: Critical health indication
     - `vignettePulse`: Damage vignette animation
     - `flashHit`: Player hit flash effect
     - `flashBomb`: Bomb detonation flash

3. **JavaScript Game Engine (rest of file)**
   - Game state management (player, enemies, projectiles, etc.)
   - Rendering loop using requestAnimationFrame
   - Input handling (keyboard and touch)
   - Collision detection
   - Enemy/boss AI
   - Particle effects and animations

## Game Mechanics

### Player
- Positioned in the lower portion of the screen
- Can move left/right and up/down
- Multiple weapon types with distinct fire patterns and damage
- Health/lives system (displayed as hearts ❤️)
- Smart bombs (displayed with special icon) - area-of-effect attack
- Weapon and score multiplier system (color-coded: green for weapon, orange for multiplier)

### Enemies
- Spawned in waves across the stage
- Various types with different movement patterns and behavior
- Collision with player projectiles causes damage/death
- Collision with player causes health loss

### Bosses
- Appear at end of stages
- More health and complex attack patterns
- Defeat triggers stage completion

### HUD Elements
- **Hearts (#hearts)**: Display current health (filled/empty states)
- **Bombs (#bombs)**: Display available smart bombs
- **Weapon Stat (#weaponStat)**: Current weapon type/level
- **Multiplier Stat (#multStat)**: Score multiplier
- **Vignette (#vignette)**: Red damage indicator with pulse animation
- **Flash (#flash)**: Visual feedback for hits and bombs

### Color Scheme
- Weapon stats: #7CFC00 (bright green)
- Multiplier stats: #ff8a65 (orange)
- Health (hearts): Default white with drop-shadow
- Backgrounds: Gradient from #0d2f5e (dark blue) to #7fc1ff (light blue)

## Development Guidelines

### For Bug Fixes
1. Issues typically involve collision detection, rendering, or input handling
2. Check the game loop (requestAnimationFrame cycle) for timing issues
3. Verify HUD updates are synchronized with game state changes
4. Test on both desktop and mobile/touch devices

### For New Features
1. Keep the single-file architecture unless the codebase grows significantly
2. Maintain responsive design - use `clamp()` CSS for flexible sizing
3. Ensure animations are smooth and performance-conscious
4. Update HUD styling if adding new stats or indicators
5. Test mobile gesture handling if modifying input system

### Important Considerations
- The game uses CSS grid/flex and `clamp()` for responsive sizing
- Touch events use `touch-action: none` to prevent browser defaults
- The vignette effect and flash feedback provide critical visual feedback
- HUD uses `max()` with `env(safe-area-inset-top)` for notch safety on mobile
- Performance is important - use `backdrop-filter: blur()` sparingly
- Text uses `text-shadow` for legibility over the sky gradient background

## Testing Checklist

When making changes:
- [ ] Game runs without console errors
- [ ] Canvas renders properly on desktop and mobile
- [ ] HUD displays all stats correctly
- [ ] Player movement is responsive
- [ ] Weapons fire and hit detection works
- [ ] Bomb effects are visible and effective
- [ ] Health/lives system responds to damage
- [ ] Score and multiplier update correctly
- [ ] Responsive design works at various viewport sizes
- [ ] Touch controls work on mobile devices

## Related Files
- `README.md` - Project metadata (minimal)
- `.git/` - Version control directory

---

*Last updated: 2026-10-08*
*Documentation created for AI-assisted development and maintenance*

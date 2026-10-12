# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The game features dynamic weapon switching, smart bombs, stage progression, and boss encounters with responsive design for both desktop and mobile platforms.

**Repository**: ubban/google_4
**Main File**: `index.html` (1669 lines)
**Technology Stack**: HTML5 Canvas, Vanilla JavaScript, CSS3

## Architecture

### Single-File Design
The entire game is contained in `index.html` with three integrated layers:
- **HTML**: DOM structure (canvas, HUD elements, UI components)
- **CSS**: Styling, animations, responsive layout
- **JavaScript**: Game engine, game loop, entity logic, input handling

### Core Game Systems

#### 1. Game Canvas & Rendering
- Uses HTML5 Canvas API for real-time 2D rendering
- Canvas is centered in viewport with responsive sizing (max-width/max-height)
- Gradient background: dark blue to light blue sky effect
- Uses requestAnimationFrame for 60fps game loop

#### 2. Entity System
Game objects are managed as entity arrays:
- `player`: Player aircraft (single instance)
- `playerBullets`: Array of bullets fired by player
- `playerBombs`: Smart bombs with area damage
- `enemies`: Regular enemy aircraft
- `enemyBullets`: Bullets fired by enemies
- `bosses`: Boss entities with large health pools
- `explosions`: Particle effects and impact visuals
- `powerUps`: Weapon upgrades and bomb drops

#### 3. Weapon System
Multiple weapon types with switching mechanics:
- **Bullet Shot** (Primary): Standard rapid-fire bullets
- **Laser** (Green): Higher damage beam weapon
- **Missile** (Purple): Seeking projectiles with explosion radius
- **Cannon** (Orange): Large shells with area damage
- **Power-Up Flash** (Rainbow): Temporary enhanced bullets

Each weapon has distinct visual appearance and damage characteristics.

#### 4. Player Controls
**Desktop/Mouse**: Move with mouse, click to fire
**Touch/Mobile**: 
  - Click/tap to fire
  - Touch+drag for movement
  - Gyroscope aiming (if supported)
  - Virtual buttons for bomb control

#### 5. Stage & Difficulty Progression
- Game progresses through multiple stages
- Each stage has wave patterns of enemies
- Boss encounter at stage end
- Difficulty scales with progression
- Health restoration between stages

#### 6. Boss System
- Boss entities with multi-phase patterns
- Distinct visual designs and attack patterns
- Boss health bar displayed during fights
- Defeat triggers next stage transition

#### 7. HUD & Display
- Score counter
- Health hearts (visual representation)
- Bomb counter (smart bomb availability)
- Current weapon indicator
- Combo multiplier display
- Stage banner animations
- Boss health bar
- Screen flashes for hits and explosions
- Red vignette effect during critical health

#### 8. Visual Effects
- **Hit Flash**: White screen flash on player damage
- **Bomb Flash**: Brighter flash for bomb explosions
- **Combo Pop**: Floating "+X COMBO" text
- **Stage Banner**: Large animated stage transition text
- **Vignette**: Red edge effect at critical health
- **Explosions**: Particle-based destruction effects

## Key Game Constants & Parameters

### Canvas Configuration
- Canvas maintains aspect ratio with responsive sizing
- Touch-action disabled for custom input handling
- Cursor set to crosshair for aiming feedback

### Player Configuration
- Starting health: 3 hearts
- Smart bomb capacity: 3 bombs
- Weapon switching: 1-5 keys or click weapon UI
- Movement speed adjustable
- Fire rate depends on weapon type

### Enemy & Boss Patterns
- Wave-based spawning system
- Positional patterns, timing delays, projectile counts
- Boss patterns vary by stage and difficulty

### Scoring System
- Base points per enemy: 100
- Bonus multiplier for combo streaks
- Scaling with difficulty/stage progression

## Development Guidelines

### Modification Workflow
1. Edit the HTML file in a text editor
2. Open in browser to test changes
3. Use browser DevTools console for debugging
4. Test on mobile devices for touch responsiveness

### Key Sections to Modify

#### Adding New Weapons
- Define weapon properties in weapon system
- Add rendering code in canvas draw loop
- Add firing logic in input handlers
- Add HUD display for weapon name/icon

#### Adding New Enemy Patterns
- Define new enemy wave patterns
- Add to stage progression sequence
- Include spawn positions and timing
- Test difficulty scaling

#### Adding New Boss Encounters
- Create boss entity with phase patterns
- Define attack patterns and health
- Add visual design to rendering loop
- Update stage progression with new boss

#### Tweaking Gameplay Balance
- Adjust player speed, fire rate, health
- Modify enemy health and damage
- Change weapon damage values
- Tweak difficulty progression curve

### Responsive Design Considerations
- All dimensions use `clamp()` for responsive sizing
- Test on various screen sizes: mobile, tablet, desktop
- Ensure touch targets are minimum 36×36px
- Verify HUD doesn't overlap critical areas on narrow screens
- Safe area insets (`env(safe-area-inset-*)`) used for notched devices

### Performance Optimization
- Canvas rendering is hardware-accelerated
- Avoid creating/destroying entities excessively
- Reuse bullet arrays instead of reallocating
- Limit explosion particles to maintain 60fps
- Monitor DevTools Performance tab for bottlenecks

## Testing Checklist

When modifying the game:
- [ ] Game starts and initializes without errors
- [ ] Player can move and fire weapons
- [ ] All weapon types work and switch correctly
- [ ] Enemies spawn and move in expected patterns
- [ ] Collision detection works (bullets, player, bombs)
- [ ] Damage system functions (enemies die, player takes damage)
- [ ] Scoring updates correctly
- [ ] Stage progression triggers at stage end
- [ ] Bosses spawn and follow attack patterns
- [ ] Mobile touch controls respond correctly
- [ ] Responsive layout works on multiple screen sizes
- [ ] No console errors in DevTools
- [ ] 60fps maintained during normal gameplay
- [ ] Visual effects display properly

## Common Issues & Solutions

### Game Freezes During Rendering
- Check for infinite loops in game loop
- Verify entity arrays aren't growing unbounded
- Check for blocking operations in frame logic

### Touch Controls Not Responsive
- Verify `touch-action: none` on canvas
- Check touch event listeners are attached
- Test with multiple touch points

### Visual Glitches
- Verify canvas is cleared before each frame
- Check z-index layering in CSS
- Ensure animations are being removed when complete

### Performance Issues
- Reduce explosion particle count
- Limit enemy bullet density
- Profile with DevTools Performance tab

## Future Enhancement Ideas

- Sound effects and background music
- Particle system improvements
- More elaborate boss patterns
- Special weapon combinations/synergies
- Leaderboard/high score persistence
- Difficulty modes (easy, normal, hard)
- Multiple game modes (endless, timed challenge)
- Controller gamepad support
- Advanced visual effects (bloom, distortion)

## Browser Compatibility

- **Desktop**: Chrome, Firefox, Safari, Edge (all modern versions)
- **Mobile**: iOS Safari, Chrome Mobile, Samsung Internet
- **Requirements**: HTML5 Canvas, ES6 JavaScript support
- **No external dependencies**: Pure vanilla JavaScript

## File Size & Performance

- Single HTML file: ~58KB (code + embedded assets)
- No external script or stylesheet dependencies
- Minimal HTTP requests (single file load)
- Suitable for deployment on static hosting

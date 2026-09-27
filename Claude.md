# Claude.md - Sky Fighter Codebase Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely with vanilla HTML5, CSS3, and JavaScript. The entire game is contained in a single `index.html` file (~1,700 lines).

### Core Features
- Vertical scrolling shoot-'em-up gameplay
- Multiple weapons system with switching capability
- Smart bomb mechanics
- Boss encounters with health bars
- Score multiplier system
- Touch and keyboard controls
- Responsive design for mobile and desktop
- Combo system for consecutive hits
- Difficulty levels (Easy, Normal, Hard)
- Best score tracking

## Project Structure

### Single-File Architecture
- `index.html` - Complete game implementation including:
  - HTML structure and semantic markup
  - CSS styling (responsive, mobile-first)
  - JavaScript game engine and logic
  - Canvas-based rendering system

### Key HTML Elements
- `#gameCanvas` - Main rendering surface
- `#hud` - Heads-up display (health, score, weapons, multiplier)
- `#overlay` - Menu and game state screens
- `#bossBarWrap` - Boss health bar display
- `#touchControls` - Touch input zones for mobile
- `#vignette` - Damage indicator effect
- `#flash` - Hit flash effects
- `#comboPop` - Combo notification

## Game Architecture

### Game Loop
The game runs on a requestAnimationFrame loop that:
1. Updates game state (player position, enemies, projectiles)
2. Detects collisions
3. Renders all game objects to canvas
4. Updates UI elements

### Key Game Systems

#### Player System
- Position-based movement
- Health system (3 lives by default)
- Multiple weapon types with switching
- Smart bomb mechanic (limited uses)
- Score and multiplier tracking

#### Enemy System
- Spawned in waves based on stage progression
- Various enemy types with different behaviors
- Projectile patterns
- Destruction with score rewards

#### Collision Detection
- Player-to-enemy collision
- Player projectiles-to-enemies
- Enemy projectiles-to-player
- Screen boundary detection

#### Boss System
- Boss encounters at stage completion
- Health bar display
- Unique attack patterns
- Defeat triggers stage progression

### Control Systems

#### Keyboard Controls
- Arrow keys or WASD for movement
- Space bar for firing
- 'B' key for bomb activation
- 'P' key for pause

#### Touch Controls
- Left 50% of screen: movement
- Right 50% of screen: fire and bomb buttons
- Responsive sizing based on viewport

#### Input Handling
- Polling-based input system for smooth movement
- Touch event listeners with preventDefault for mobile
- Mouse position tracking for aiming

## Coding Conventions

### CSS
- Mobile-first responsive design
- Uses `clamp()` for fluid typography
- Safe area insets for notched devices
- Dark theme with gradient backgrounds
- Layered z-index system for UI stacking

### JavaScript
- Immediately Invoked Function Expression (IIFE) wrapping
- Canvas context stored in variables for performance
- Object-based state management
- Collision detection using bounding boxes
- RequestAnimationFrame for smooth 60fps rendering

### Game State Object
```javascript
game = {
  stage: number,
  score: number,
  health: number,
  bombs: number,
  weapon: number,
  multiplier: number,
  combo: number,
  isPaused: boolean,
  isRunning: boolean
}
```

### Data Storage
- Uses `localStorage` for best score persistence
- Keys: `skyFighter_bestScore`, `skyFighter_bestScoreStage`

## Development Guidelines

### Adding New Features

#### New Enemy Type
1. Define enemy properties (position, health, speed, projectile pattern)
2. Add spawn logic in stage progression
3. Define enemy-specific update and render methods
4. Update collision detection if needed

#### New Weapon Type
1. Add weapon definition with damage, fire rate, projectile speed
2. Update weapon switching logic
3. Define projectile rendering (different colors/shapes)
4. Update weapon stat display in HUD

#### New Stage/Boss
1. Define stage number, enemy waves, progression timing
2. Create boss object with health, attack patterns, sprite
3. Add boss encounter trigger logic
4. Update stage banner display

### Debugging Tips
- Use `console.log()` for state tracking
- Canvas coordinate system: (0,0) is top-left
- Use `drawRect()` helpers for collision debugging
- Mobile debugging: use browser DevTools with responsive mode

### Performance Considerations
- Canvas rendering is the main performance bottleneck
- Keep collision detection efficient (early exit on non-overlapping areas)
- Use object pooling for frequently created/destroyed objects (projectiles)
- Avoid DOM manipulation inside game loop
- Profile with DevTools Performance tab

## Testing & Validation

### Manual Testing Checklist
- [ ] Game starts cleanly on load
- [ ] Keyboard controls respond immediately
- [ ] Touch controls work on mobile devices
- [ ] Score increments correctly on enemy destruction
- [ ] Weapon switching works (cycle through weapon types)
- [ ] Bombs fire and clear projectiles
- [ ] Boss encounters trigger at stage end
- [ ] Game over screen shows correct stats
- [ ] Score persistence works after page reload
- [ ] Difficulty selection changes gameplay speed/enemy count
- [ ] Pause/unpause functions correctly
- [ ] Audio (if added) doesn't autoplay

### Responsive Testing
- Test on various screen sizes: mobile, tablet, desktop
- Test in both portrait and landscape orientations
- Verify safe area insets work on notched devices
- Test with different pixel ratios (DPI scaling)

## Common Modifications

### Adjusting Game Difficulty
- `ENEMY_SPEED_BASE` - Base speed for all enemies
- `BOSS_HEALTH` - Starting boss health
- `SPAWN_RATE` - Frequency of enemy spawns
- Wave patterns in stage progression logic

### Tweaking Player Balance
- `PLAYER_HEALTH` - Starting health value
- `BOMB_COUNT` - Starting bombs available
- Weapon damage values
- Movement speed `playerSpeed`

### UI/Visual Tweaks
- Color values in CSS `:root` variables
- Canvas gradient colors in initialization
- Font sizes using `clamp()` for scaling

## Browser Compatibility

### Supported Features
- HTML5 Canvas (required)
- RequestAnimationFrame
- LocalStorage
- Touch Events
- CSS Variables
- CSS Gradients
- Flexbox

### Target Browsers
- Chrome/Edge 60+
- Firefox 55+
- Safari 12+
- Mobile Chrome/Safari (iOS 12+)

### Known Limitations
- Requires JavaScript enabled
- No progressive enhancement fallback
- Mobile performance varies by device

## Build & Deployment

### Current Setup
- Single file deployment
- No build process required
- Can be hosted on any static file server

### Deployment Checklist
- [ ] Test all features locally
- [ ] Verify on target mobile devices
- [ ] Check score persistence works
- [ ] Verify touch controls on actual devices
- [ ] Test on slow networks
- [ ] Validate responsive layout

## Future Enhancement Opportunities

- Sound effects and music
- Particle effects system
- Power-up items
- Multiple difficulty levels tuning
- Leaderboard system
- Different game modes (time attack, survival)
- Animated sprite characters
- More complex boss patterns
- Weapon balance adjustments
- Visual effects enhancements (screen shake, bloom)

## References & Resources

### Game Mechanics References
- Raiden game series (primary inspiration)
- Classic vertical shmups for design patterns

### Technical Documentation
- HTML5 Canvas API
- LocalStorage API
- RequestAnimationFrame API
- Touch Events specification

### Performance Optimization
- Canvas rendering best practices
- JavaScript animation optimization
- Mobile performance optimization

## Notes

- Game state is entirely client-side
- No server interaction required
- Best score persists across sessions in localStorage
- Combo system resets on miss/damage
- Weapon switching is instant with no cooldown
- Bombs are limited resource - must manage carefully
- Boss encounters serve as stage gates

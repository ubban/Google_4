# Sky Fighter - Codebase Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 Canvas application. The game features aircraft combat with weapon switching, smart bomb mechanics, stage bosses, and progressive difficulty scaling.

**Tech Stack:**
- Pure HTML5/CSS3/JavaScript (no frameworks)
- Canvas API for rendering
- Single `index.html` file (~60KB)

## Getting Started

### Running the Game

1. Open `index.html` in any modern web browser
2. The game launches immediately in fullscreen mode on mobile or maximized on desktop
3. Press any key or tap the screen to start after game over

### Controls

- **Desktop:** Mouse movement for aiming, WASD or Arrow Keys to move, SPACE to shoot, Q/E to switch weapons, SHIFT for bombs
- **Mobile:** Touch to aim and move, tap to shoot, swipe gestures for weapon switching and bombs

## Game Architecture

### Canvas Setup & Rendering

```
index.html (single file)
├── HTML Structure
│   ├── Canvas element (#gameCanvas)
│   ├── HUD containers (#hud, #hearts, #bombs)
│   └── Visual effect overlays (#vignette, #flash)
├── CSS Styling (inline)
│   ├── Game container sizing and layout
│   ├── HUD visual design (stats, hearts, bombs)
│   └── Animation keyframes
└── JavaScript (inline <script>)
    └── Complete game logic
```

### Game State Management

The game maintains state through global variables and object properties:

- **Player**: Position, health, weapon state, bomb count, score multiplier
- **Enemies**: Array of active enemies with position, health, behavior patterns
- **Projectiles**: Player bullets and enemy fire
- **Bosses**: Stage-specific boss entities with multi-phase behavior
- **Game Flow**: Current stage, game-over flag, pause state

### Core Game Loop

The game runs on `requestAnimationFrame`:

1. **Update Phase**: Process input, update positions, check collisions
2. **Render Phase**: Clear canvas, draw all game objects, update HUD
3. **Collision Detection**: Check projectile-enemy hits, enemy-player contact
4. **Scoring**: Update score and multiplier based on kills and combos

## Key Gameplay Systems

### Weapon System

Three weapon types cycle with Q/E (or swipe gestures):

1. **Standard**: Single stream, fastest fire rate
2. **Spread**: 3-way spread pattern, medium fire rate
3. **Homing**: Seeks nearest enemy, slowest fire rate

### Bomb Mechanic

- Start with 3 bombs per stage
- Activated with SHIFT (or two-finger tap)
- Clears screen of projectiles and damages all visible enemies
- Invulnerability window after detonation

### Difficulty Scaling

- Enemy spawn rate increases by 10% per stage
- Enemy health increases per stage
- Boss patterns become more aggressive with each stage

### HUD Display

```
HUD Elements:
├── Health Hearts (❤️ with lost state ghosted)
├── Smart Bombs Display (💣)
├── Current Weapon Type (colored text)
├── Kill Multiplier (orange text, resets on player damage)
└── Score (accumulated from kills)
```

## Code Organization

### HTML Section
- Meta tags with viewport and game description
- Single canvas element with responsive sizing
- HUD stat elements for real-time game info
- Visual effect layers (vignette pulse for damage, white flash for hits)

### CSS Section
- Root dark color scheme (`#0b1d3a`, `#05122b`)
- Canvas styling with gradient sky background
- HUD layout using flexbox with glassmorphism (backdrop blur, transparency)
- Responsive text sizing with `clamp()` for mobile/desktop
- Keyframe animations for heart pulse (critical health) and vignette pulse
- Safe area insets for notched devices

### JavaScript Section

Key Objects:

- **game**: Global game state object
  - `stage`, `score`, `paused`, `gameOver`, `canRestart`
  
- **player**: Player aircraft state
  - `x`, `y`, `health`, `maxHealth`
  - `weapon`, `bombsRemaining`
  - `inputState` for current control inputs
  
- **enemies[]**: Array of active enemy ships
  - Position, `health`, `type`, `behavior` function
  - Some enemies drop power-ups on death
  
- **bosses[]**: Stage boss entities (spawned at stage 3+)
  - Multi-phase patterns with complex attack sequences
  
- **bullets[]**: Player projectiles
  - `type` (standard, spread, homing), position, velocity
  
- **enemyFire[]**: Enemy projectiles
  - Position, velocity, created timestamp

### Major Functions

- `initGame()`: Set up new game
- `update()`: Main game loop update logic
- `render()`: Canvas drawing
- `spawnWave()`: Generate enemy formations
- `spawnBoss()`: Create stage boss
- `checkCollisions()`: Detect all hit interactions
- `gameOver()`: End game state
- `useBomb()`: Trigger smart bomb effect
- `changeWeapon()`: Cycle weapon types
- `updateHUD()`: Refresh visual stats display

## Collision System

- **Spatial hashing**: Enemies divided into grid zones for efficient collision checks
- **Projectile resolution**: Bullets check against all enemies per frame
- **Damage model**: 
  - Standard: 1 damage per hit
  - Bosses: 5-20 HP depending on phase
  - Enemies: 1 HP (one-shot kills)
  - Player takes 1 damage per enemy collision or enemy bullet

## Mobile/Touch Optimization

- Full viewport coverage with `width: 100vw; height: 100vh`
- Touch-action: none to prevent default gestures
- Responsive text sizing with `clamp(min, preferred, max)`
- Safe area insets for notched devices
- Crosshair cursor on desktop
- Gesture controls: swipes for weapon/bomb, tap to shoot

## Performance Considerations

- Canvas 2D context is reused (not recreated per frame)
- Enemies removed from array when destroyed (array splice)
- Projectiles culled when off-screen
- requestAnimationFrame prevents excessive CPU usage
- No asset loading (all graphics drawn with canvas primitives)

## Known Limitations & Future Improvements

### Current Limitations

1. **No persistent storage**: High scores not saved between sessions
2. **Audio**: No sound effects or music
3. **Graphics**: Minimal geometric shapes (no sprite sheets or images)
4. **Accessibility**: Limited keyboard/screen reader support
5. **Difficulty**: No adjustable difficulty settings

### Potential Enhancements

1. **LocalStorage**: Save high scores and stats
2. **Web Audio API**: Add sound effects and background music
3. **Asset Pipeline**: Sprite sheets for more detailed visuals
4. **Leaderboards**: Cloud-based score tracking
5. **Customization**: Difficulty settings, color themes
6. **Cheat codes**: Debug modes, level select
7. **Tutorial**: First-time user guide

## Testing & Debugging

### Manual Testing Checklist

- [ ] Game initializes without errors
- [ ] Player can move and shoot
- [ ] Weapon cycling works (Q/E or swipes)
- [ ] Bombs clear projectiles and damage enemies
- [ ] Enemies spawn in waves with increasing frequency
- [ ] Boss appears at stage 3
- [ ] HUD updates correctly (health, bombs, score)
- [ ] Game over triggers at 0 health
- [ ] Restart works with any key press
- [ ] Mobile touch controls function
- [ ] Responsive layout on various screen sizes

### Browser Compatibility

Tested and working on:
- Chrome 120+
- Firefox 121+
- Safari 17+
- Edge 120+
- Mobile Safari (iOS 15+)
- Android Chrome/Firefox

Requires:
- ES6 JavaScript support
- Canvas 2D API
- Touch Events API (for mobile)

## Development Notes

### Code Style

- Single file keeps dependencies minimal
- Global game state with `game` namespace object
- Pure functions for update logic where possible
- Canvas context (`ctx`) is global for easy access

### Adding Features

When adding new features:

1. **Update state objects**: Add properties to `game`, `player`, `enemies[]`
2. **Add update logic**: Modify `update()` function
3. **Add rendering**: Add drawing code to `render()`
4. **Add input**: Handle new controls in keyboard/touch listeners
5. **Test collision**: Update `checkCollisions()` if needed

### Common Modifications

- **Change colors**: Search for hex codes in CSS and canvas `fillStyle`
- **Adjust difficulty**: Modify spawn rates in `spawnWave()`, enemy health values
- **Add weapon**: Create new object type, add to weapon cycling logic
- **Change controls**: Update event listeners at bottom of script

## File Structure

```
Google_4/
├── README.md           (Project timestamp)
├── CLAUDE.md          (This file - development guide)
└── index.html         (Single-file game - 60KB)
    ├── HTML head (meta, title, styles)
    ├── HTML body (canvas, HUD)
    └── JavaScript (complete game engine)
```

## Contributing Guidelines

When working on this project:

1. Keep the single-file structure (easier to deploy, no build step)
2. Maintain responsive design for mobile and desktop
3. Test on multiple browsers before committing
4. Comment complex game logic (collision detection, boss patterns)
5. Update this CLAUDE.md for significant changes
6. Commit messages should reference feature or bug fix

## Version History

- **v1.0** (Current): Initial release with vertical shooter gameplay, weapon switching, smart bombs, stage progression

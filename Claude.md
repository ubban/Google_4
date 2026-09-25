# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in HTML/JavaScript. The game features:
- Vertical scrolling gameplay with progressive difficulty
- Multiple weapon types with upgradeable levels
- Boss battles at the end of each stage
- Smart bomb mechanics
- Score multipliers and combo system
- Multiple difficulty levels (Easy, Normal, Hard)
- Responsive touch controls for mobile devices
- Accessibility features (screen reader support, reduced motion preferences)
- WebAudio-based sound synthesis (no external audio files)

## Project Structure

The codebase is **monolithic** — all functionality is contained in a single `index.html` file (~2500 lines):

- **HTML**: Game UI structure, buttons, HUD elements, overlay screens
- **CSS**: Styling for all game elements, animations, responsive design
- **JavaScript**: Core game engine, game loop, entity management, input handling

### Key Files

- `index.html` — Complete game implementation
- `README.md` — Basic project info
- `.git/` — Version control

## Game Architecture

### Game Loop & Core State

The game runs on a standard 60 FPS animation loop using `requestAnimationFrame`. Key state variables include:

- **Game State**: `state` (menu, playing, gameOver, paused)
- **Player**: Position, health, bombs, current weapon
- **Enemies**: Array of active enemies with position, health, type
- **Projectiles**: Player bullets and enemy fire
- **Score & Progression**: Score, multiplier, stage count, best score

### Main Game Systems

#### 1. **Entities System**
- `Player`: Handles movement, shooting, weapon selection, damage
- `Enemy`: Base class for different enemy types (grunt, interceptor, tank)
- `Boss`: Special boss entity at end of each stage
- `Projectile`: Bullets and bombs with collision detection
- `Particle`: Visual effects (explosions, impacts)

#### 2. **Weapon System**
```
- VULCAN: Rapid-fire, upgradeable (levels 1-3)
- LASER: High damage, piercing shots
- SPREAD: Wide spread pattern
- HOMING: Seeks nearest enemy
- PLASMA: Chain damage between enemies
```
Each weapon has:
- Base damage and fire rate
- Upgrade levels affecting damage/fire rate
- Pickup logic for level increases

#### 3. **Difficulty System**
Three difficulty presets (Easy/Normal/Hard) affect:
- Enemy spawn rates and minimum spawn intervals
- Enemy speed and shoot frequency
- Player starting lives and bombs
- Boss health multipliers

#### 4. **Audio System**
Uses WebAudio API with procedural sound synthesis:
- `beep()` — Oscillator-based tones with optional frequency sweep
- `noiseBurst()` — White noise effects for explosions
- Dynamic pan based on screen position for spatial audio
- Master gain control with compressor for audio leveling

#### 5. **Rendering Pipeline**
- Sky gradient background (theme-based)
- Parallax scrolling clouds
- Entity rendering (player, enemies, bosses)
- Particle effects and trails
- HUD overlay (score, health, weapon info, boss bar)
- Screen effects (vignette on damage, flash on hit)

#### 6. **Input Handling**
**Keyboard/Mouse:**
- Arrow keys or WASD for movement
- Mouse movement for aiming
- Space or click for firing
- X or Shift for bombs
- P or Esc to pause

**Touch:**
- Left side drag zone for movement
- Right side buttons for fire and bomb
- Visual joystick feedback

### Enemy Types

| Type | Health | Speed | Score | Notes |
|------|--------|-------|-------|-------|
| Grunt | 1 | 1.0x | 10 | Basic cannon fodder |
| Interceptor | 1 | 1.7x | 15 | Fast, weaving pattern |
| Tank | 4 | 0.55x | 35 | Heavy armor |

Enemies appear at stage 1 and increase in difficulty/variety as stages progress.

### Boss System

Six unique bosses with different color schemes and health multipliers:
1. **VANGUARD** (1.0x HP) — Purple hull
2. **BEHEMOTH** (1.35x HP) — Orange, high health
3. **SPECTER** (0.8x HP) — Teal, fast
4. **DREADNOUGHT** (1.55x HP) — Gray, tanky
5. **WRAITH** (0.95x HP) — Deep purple
6. **TITAN** (1.7x HP) — Golden, highest health

Bosses cycle based on stage number and rotate their appearance.

### Stage Progression

- 18 kills per stage triggers boss encounter
- Each stage changes the sky theme (6 unique themes)
- Score multiplier increases per stage
- Difficulty scales with adaptive enemy spawning

## Key Code Sections

### Game Loop
```javascript
function gameLoop(now) {
  // ...
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  // Render sky, clouds
  // Update and render all entities
  // Handle collision detection
  // Update HUD
  // Track performance
  requestAnimationFrame(gameLoop);
}
```

### Difficulty Constants
```javascript
const DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, ... },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, ... },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, ... }
};
```

### Collision Detection
Uses AABB (Axis-Aligned Bounding Box) collision:
- Player vs projectiles
- Projectiles vs enemies/bosses
- Enemy projectiles vs player
- Proximity checks for pickups

### Performance Optimization
- **Adaptive Quality**: Monitors frame time and adjusts rendering quality (0=full, 1=medium, 2=low)
- **Object Pooling**: Reuses entity objects when possible
- **Culling**: Removes off-screen entities
- **Particle Limits**: Caps active particles to maintain performance

## Data Persistence

Uses `localStorage` for:
- `skyfighter.bestScore` — High score
- `skyfighter.muted` — Audio mute state
- `skyfighter.difficulty` — Last selected difficulty
- `skyfighter.reducedMotion` — Motion preferences

## Accessibility Features

1. **Screen Reader Support**: Text announcements for important events
2. **Reduced Motion**: Respects `prefers-reduced-motion` media query
   - Disables animations, screen shake, flash effects when enabled
3. **High Contrast**: Dark theme with good contrast ratios
4. **Mobile Touch Support**: Full touch-based controls for mobile play
5. **Keyboard Navigation**: All controls accessible via keyboard

## Responsive Design

Uses CSS `clamp()` for fluid scaling:
- Text sizes scale with viewport
- Button sizes adapt to screen
- Layout responds to portrait/landscape
- Safe area insets for notched devices

## Common Tasks

### Adding a New Weapon Type
1. Define weapon config in the weapons object
2. Add rendering logic in the player rendering section
3. Add projectile generation in the shooting function
4. Add pickup detection logic

### Modifying Enemy Behavior
1. Edit enemy type definitions in `ENEMY_TYPES`
2. Modify movement logic in enemy update function
3. Adjust spawn rates via `DIFFICULTIES`

### Adding a New Stage Theme
1. Add entry to `THEMES` array with sky gradient and cloud colors
2. Themes automatically cycle based on stage number

### Adjusting Difficulty
1. Modify values in `DIFFICULTIES` object
2. Values control spawn rates, enemy speed, lives, etc.

## Performance Considerations

- **Canvas Rendering**: 60 FPS target, adaptive quality if needed
- **Audio Context**: Lazily initialized on first interaction
- **Entity Management**: Dead entities removed from arrays immediately
- **Frame Time Tracking**: Monitors last 45 frames for quality adjustment
- **WebAudio**: Uses efficient oscillators and filters

## Browser Compatibility

- Modern browsers with:
  - HTML5 Canvas (2D context)
  - Web Audio API (optional for sound)
  - localStorage
  - requestAnimationFrame
  - ES5+ JavaScript support

## Testing Recommendations

1. **Difficulty Balance**: Test all three difficulty levels for appropriate challenge
2. **Mobile**: Test on various screen sizes and devices (phones, tablets)
3. **Performance**: Monitor frame rate under high entity count
4. **Accessibility**: Test with screen readers and reduced motion enabled
5. **Audio**: Verify sound effects work across browsers
6. **Touch Controls**: Test on actual touch devices

## Future Enhancement Ideas

- Multi-player or co-op modes
- Additional weapon types or upgrades
- Power-up system (shields, invincibility)
- Leaderboard system
- More boss varieties
- Procedural stage generation
- Performance metrics dashboard

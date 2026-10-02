# Sky Fighter — Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built with vanilla HTML5, Canvas, and JavaScript. Players control a fighter jet to defeat waves of enemies, collect power-ups, and face stage bosses. The entire game is self-contained in a single `index.html` file (~1669 lines).

### Key Features
- **Vertical scrolling shmup** gameplay with continuous enemy waves
- **Weapon system**: Switch between rapid-fire bullets and lasers
- **Power-ups**: Health recovery, weapon upgrades, smart bombs
- **Boss battles** at the end of each stage
- **Adaptive difficulty**: Enemy health and fire rate scale with player performance
- **Responsive design**: Works on mobile and desktop with touch/keyboard controls
- **Graphics**: Procedurally drawn plane sprites using Canvas 2D
- **High-performance rendering**: Uses quality settings to maintain 60 FPS

## File Structure

```
Google_4/
├── index.html          # Single-file game implementation
├── README.md           # Project timestamp/metadata
└── .git/               # Git repository
```

## Game Architecture

### Core Game Loop
The game runs a `requestAnimationFrame` loop that:
1. Clears and redraws the canvas
2. Updates player position (mouse tracking)
3. Updates bullets and collisions
4. Updates enemies and spawning logic
5. Updates power-ups and their collection
6. Renders HUD (health, score, weapon status)
7. Handles stage progression and boss battles

### Key Game Objects

#### Player (`player`)
- **Position**: Center of screen by default, follows mouse
- **Health**: Starts at 100, restored by power-ups
- **Weapon State**: `weapon` (0=bullets, 1=lasers) and `bombCharge` for smart bombs
- **Lives**: 3 by default
- **Invincibility frames**: Temporary damage immunity after being hit

#### Bullets (`bullets[]`)
- **Types**: 
  - Regular bullets (yellow, rapid-fire)
  - Lasers (cyan, more powerful, slower rate)
- **Physics**: Straight-line movement upward
- **Collision**: Destroys on impact with enemies

#### Enemies (`enemies[]`)
- **Types** (in `ENEMY_TYPES`):
  - `fighter`: Small fast enemies, low health
  - `tank`: Armored enemies, high health, visible health bar
  - `bomber`: Medium enemies, medium health
- **Spawning**: Waves spawn from off-screen based on stage progression
- **AI**: Simple straight-line or angled movement patterns
- **Firing**: Return fire at player (slower, red bullets)

#### Boss (`boss`)
- **Boss Ship**: Appears at end of stage with high health
- **Behavior**: 
  - Multi-phase combat with sustained fire patterns
  - Horizontal movement across screen
  - Split into vulnerable segments visually
- **Health bar**: Displayed during battle
- **Defeat**: Advances to next stage

#### Power-ups (`powerUps[]`)
- **Drop chance**: Random chance from defeated enemies
- **Types**: 
  - Green: Weapon upgrade (toggle between bullets/lasers)
  - Red: Health recovery (+20-30 HP)
  - Purple: Smart bomb (one-time area damage)
- **Behavior**: Descend with gravity, collected on contact

### Game State Variables

```javascript
stage          // Current stage number (1-indexed)
wave           // Current wave within stage
player.health  // Current player health (0-100)
player.lives   // Remaining lives
player.weapon  // Current weapon: 0=bullets, 1=lasers
score          // Player score
combo          // Kill combo counter (increases with consecutive kills)
```

### Collision Detection

**Simple AABB (Axis-Aligned Bounding Box)** collision checking:
- Bullets vs enemies: Rectangle overlap
- Player vs enemies: Rectangle overlap
- Enemies vs player: Mutual collision
- Power-up collection: Loose collision for player convenience

## Controls

### Keyboard
- **Arrow Keys / WASD**: Move player
- **Space**: Fire weapon
- **Q**: Drop smart bomb (if charged)
- **1-2**: Quality settings (debug)

### Mouse
- **Movement**: Player follows mouse cursor
- **Click**: Fire weapon (optional, can also use Space)

### Mobile/Touch
- **Touch drag**: Move player
- **Tap screen**: Fire weapon

## Key Implementation Details

### Graphics Rendering

#### Player Ship
- Procedurally drawn using Canvas paths
- Triangle-based design (main fuselage + wings)
- Color varies based on weapon type (blue for bullets, cyan for lasers)

#### Enemy Ships
- Scaled versions of player ship design
- Different colors for different enemy types
- Armor plating visualization on tanks

#### Boss Ship
- Larger multi-segment design
- Animated segments for visual feedback during battle
- Health bar underneath

#### Particles & Effects
- Glow effects on bullets (shadow blur at high quality)
- Crosshair at mouse position
- Flash effects on damage

### Performance Optimization

**Quality Settings** (`quality` variable: 0-2):
- Quality 0: No glow effects, shadow blur disabled → Best performance
- Quality 1: Standard glow/shadows
- Quality 2: Enhanced visual effects (slower)

**Rendering optimizations**:
- Canvas clear using `fillRect` (faster than `clearRect`)
- Context state save/restore for effect batching
- Conditional rendering for off-screen objects

### Audio & Feedback
- **No audio** (game is silent)
- **Visual feedback**: Screen shake on major events (partially implemented)
- **Haptics**: No mobile haptics implemented

### Difficulty Progression

**Scaling factors**:
- Enemy health increases with stage
- Enemy fire rate increases
- Bomb spawn rates increase
- Wave size increases over time

**Dynamic adjustment**:
- Game detects player skill (score, survival time)
- Adjusts enemy spawning accordingly

## How to Run

### Local Development
1. Clone repository: `git clone <repo-url>`
2. Open `index.html` in a web browser
3. Game starts automatically
4. No build step required

### Hosting
1. Copy `index.html` to web server
2. Serve with any HTTP server
3. Access via browser

## Development Workflow

### Making Changes
1. Edit `index.html` directly
2. Refresh browser to see changes (no build needed)
3. Use browser DevTools for debugging
4. Test on multiple devices for compatibility

### Key Variables to Modify
- `ENEMY_TYPES`: Adjust enemy stats (health, speed, fire rate)
- `STAGE_CONFIGS`: Modify wave spawning patterns
- `BASE_DIFFICULTY`: Global difficulty multiplier
- `#gameCanvas` dimensions: Change game resolution
- Color scheme: Update gradient and color constants

### Adding Features
- New enemy types: Add to `ENEMY_TYPES` object, update spawn logic
- New weapons: Modify weapon switching and firing code
- New power-ups: Add to power-up generation and collection logic
- New stages: Extend `STAGE_CONFIGS` array

## Browser Compatibility

**Tested on**:
- Chrome/Chromium 90+
- Firefox 88+
- Safari 14+
- Mobile browsers (iOS Safari, Chrome Android)

**Requirements**:
- HTML5 Canvas support
- `requestAnimationFrame` support
- ES6 JavaScript support
- Touch Events API (optional, for mobile)

## Debugging Tips

### Common Issues
- **Game too fast/slow**: Check device performance, adjust `quality` setting
- **Bullets not firing**: Check weapon type and firing rate cooldown
- **Enemies not spawning**: Check `currentWaveEnemy` counter and stage config
- **Collision not working**: Verify AABB overlap calculation

### Debug Output
- Open browser console (F12)
- Check `player`, `enemies`, `bullets` arrays
- Monitor `stage`, `wave`, `score` variables
- Use `console.log()` to trace game state

## Future Enhancement Ideas

1. **Sound**: Add background music and SFX
2. **Leaderboards**: High score tracking with localStorage
3. **Additional stages**: Expand from current stage count
4. **New enemies**: Boss variants, special enemy types
5. **Ship customization**: Multiple playable ships
6. **Particle effects**: Explosion animations, trails
7. **Mobile polish**: Haptic feedback, native app wrapper
8. **Tutorial/Story**: Intro sequence, stage briefings

## Technical Notes

### Performance Considerations
- Game targets 60 FPS on average devices
- Main loop overhead: ~5-10ms per frame
- Heavy on: Enemy spawning waves, glow effects at high quality
- Optimize with: Reduce `quality`, limit simultaneous enemies

### Known Limitations
- Single HTML file (no module system)
- Global variable scope (could be refactored to class)
- Simple collision detection (AABB only)
- No mobile-specific optimizations
- Canvas size fixed (could be responsive)

### Code Style
- Vanilla JavaScript (no frameworks)
- Functional imperative approach
- Heavy use of object literals for state
- Direct Canvas 2D API usage
- Minimal comments (code is relatively self-documenting)

## Attribution & Inspiration

- **Genre**: Inspired by classic vertical shooters (Raiden, 1990 Arcade)
- **Art Style**: Procedurally generated vector graphics
- **Platform**: Web-based implementation for accessibility

---

**Last Updated**: 2026-10-02

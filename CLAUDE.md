# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 web application. The game features player aircraft combat, multiple weapon types, boss encounters, and progressive difficulty stages.

**Technology Stack:**
- HTML5 Canvas for rendering
- Vanilla JavaScript (ES6+) for game logic
- Web Audio API for sound effects
- Responsive design supporting desktop and mobile touch controls

## Project Structure

### File Organization
- `index.html` - Single-file application containing all HTML, CSS, and JavaScript (~1669 lines)
- `README.md` - Project metadata

## Architecture & Key Systems

### 1. Game State Management
The game maintains global state for:
- **Player state**: position, health, weapons, invulnerability frames, bomb count
- **Game running state**: pause status, current stage, game over conditions
- **Entity collections**: `enemies`, `bullets`, `enemyBullets`, `powerUps`, `particles`
- **Boss state**: active boss entity with unique behavior
- **Performance tracking**: FPS monitoring and quality adjustment

### 2. Rendering System (Canvas-based)
- **Main draw loop**: `draw()` function manages all visual rendering
- **Layered rendering**: Sky gradient background → clouds → entities → effects → UI
- **Visual effects**:
  - Screen shake (controlled by `shakeTime` and `shakeMag`)
  - Flash effects (on hit/bomb)
  - Vignette effect (red border flash on damage)
  - Particle systems for explosions
  - Glowing effects for weapons/bullets (adjustable based on quality setting)

**Key rendering functions:**
- `drawPlane()` - Renders player and enemy aircraft
- `drawBossShip()` - Renders boss sprite with visual variations
- `drawClouds()` - Procedurally draws parallax cloud layers
- `ctx.canvas` - 800x600 canvas (responsive scaling)

### 3. Game Loop & Update System
- **Frame-based animation**: `requestAnimationFrame(loop)` - runs at ~60 FPS
- **Time-stepping**: Delta time (dt) capped at 50ms to prevent physics jumps
- **Update sequence**: 
  1. Input processing (keyboard/mouse/touch)
  2. Entity updates (movement, collision detection)
  3. Spawning logic (enemies, power-ups based on score/stage)
  4. Collision resolution
  5. Render

### 4. Control Systems
- **Desktop**: WASD movement + mouse aiming + Space/click to shoot
- **Mobile/Touch**: Touch position for movement, tap to shoot
- **Pause**: P key or pause button UI element
- **Quality adjustment**: Q key toggles glow effects

### 5. Combat System

#### Player Combat
- **Weapons**: 
  - Default: Standard bullet spray (5 shots per trigger)
  - Laser: Continuous beam weapon with higher damage
  - Weapon switching: Cycle with E key
- **Special moves**:
  - Smart bombs: Limited (typically 1-3 per stage), damage all on-screen enemies
  - Bomb activation triggers screen flash and particle effects
- **Health/Lives**: Default 5 hearts (configurable), lose 1 per enemy hit
- **Invulnerability**: Temporary i-frames after taking damage (prevents rapid damage)

#### Enemy Types
**ENEMY_TYPES definitions** include:
- Basic fighters: Default enemy type with configurable speed/health/damage
- Tanks: Heavily armored units with health bars, spawn power-ups on defeat
- Boss variants: Stage-specific boss ships with unique patterns and health values

**Enemy behavior:**
- Spawning patterns controlled by stage/score thresholds
- Movement patterns: Horizontal weaving, kamikaze dives
- Firing patterns: Basic bullets toward player position or leading shots
- Collision detection: Enemy bullets and aircraft collide with player and bullets

### 6. Progression System
- **Stages**: 1-N stages with increasing difficulty
- **Stage transitions**: Triggered after reaching score thresholds
- **Boss encounters**: Spawn at specific stage points
- **Visual themes**: Each stage has distinct sky gradient colors, sun color, cloud styling
- **Enemy variety**: Different enemy types spawn as stages progress

### 7. Score & Rewards
- **Points**: Earned by destroying enemies (varies by type)
- **Power-ups**:
  - Life (♥): Restore 1 health
  - Bomb (💣): Add smart bomb
  - Weapon (⭐): Change to laser weapon
- **Scoring multiplier**: Displayed in HUD, affects score calculation

### 8. Audio System
- **Audio initialization**: `ensureAudio()` handles Web Audio API setup and user interaction requirement
- **Sound effects** (via `sfx` object):
  - `shoot()`: Player weapon fire
  - `hit()`: Player takes damage
  - `enemyHit()`: Enemy destroyed
  - `bomb()`: Smart bomb explosion
  - `uiClick()`: Menu button clicks
  - `gameOver()`: Game end sound
- **Web Audio synthesis**: Sound generated procedurally (no external audio files)

### 9. UI System
**HUD Display** (at top of canvas):
- Lives/Health hearts (♥)
- Smart bombs remaining (💣)
- Current weapon (Laser/Normal)
- Score multiplier
- FPS (debug, if visible)

**Menu States:**
- Start screen overlay with game title
- Pause overlay
- Game over screen with score, high score, best stats
- Settings: quality toggle (glow effects)

### 10. Input Handling
- **Keyboard events**: 
  - WASD or arrow keys for movement
  - Space/E for weapons
  - Q for quality toggle
  - P for pause
- **Mouse events**: 
  - Movement for targeting
  - Click to shoot
  - Touch events mapped to click handlers
- **Touch controls**: Touch position = player target position

## Key Constants & Configuration

### Global Configuration
```javascript
CANVAS_WIDTH = 800
CANVAS_HEIGHT = 600
PLAYER_SPEED = 250  // pixels/sec
PLAYER_SIZE = 20
ENEMY_BULLET_SPEED = 150
```

### Theme System (THEMES array)
Each stage has a theme object:
- `sky`: Array of 3 colors for gradient (top → middle → bottom)
- `sun`: RGB values for sun glow color
- `cloud`: RGB values for cloud color

### Enemy Type Definitions (ENEMY_TYPES object)
Configurable per enemy type:
- `hp`: Health points
- `dmg`: Damage dealt to player
- `speed`: Movement speed
- `w`, `h`: Sprite dimensions
- `color`: Display color
- `dark`: Darker variant for shading

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with stats
2. Add spawn logic in `update()` function's enemy spawning section
3. Define movement pattern if needed (in enemy update loop)
4. Add rendering variant if visual differentiation needed

### Creating a New Stage Theme
1. Add theme object to `THEMES` array with sky/sun/cloud colors
2. Adjust enemy spawn patterns in stage progression logic
3. Increase boss HP/spawn rate as difficulty increases

### Modifying Weapon Behavior
1. Locate weapon switch in `update()` where bullets fire based on `player.weapon`
2. Adjust bullet spread, speed, or firing rate
3. Test collision detection with existing enemy behavior

### Adjusting Game Balance
- `PLAYER_SPEED`: Player movement responsiveness
- `ENEMY_TYPES[type].speed`: Enemy aggression
- `player.maxLife`: Game difficulty (fewer hearts = harder)
- Score thresholds in stage progression logic

## Performance Considerations

### Quality Settings
- `quality = 0`: Glow effects enabled (shadow blur on bullets, enemies)
- `quality = 1`: Glow effects disabled for better performance on mobile
- Toggle with Q key during gameplay

### Optimization Techniques
- Frame skipping for FPS measurement (only measure every N frames)
- Canvas clear/redraw only visible portions when possible
- Entity pooling could improve performance with many particles
- Avoid O(n²) collision checks; current implementation uses spatial reasoning

### Known Limitations
- Single canvas element (no layering/z-buffer)
- No entity spatial indexing (full array iteration for collision detection)
- Procedural sound synthesis may have timing quirks in some browsers

## Testing & Debugging

### Debug Features
- FPS counter (visible if `debug` mode enabled)
- Screen shake testing (press space rapidly to test vignette effect)
- Quality toggle (Q key) to benchmark rendering performance

### Common Issues & Solutions
1. **Audio not playing**: Requires user interaction first; `ensureAudio()` must be called from click/touch handler
2. **Performance drops on old devices**: Lower `quality` setting automatically, disable glow effects
3. **Touch controls unresponsive**: Check `touchAction: none` CSS; verify touch event listeners are bound

## Browser Compatibility
- **Modern browsers**: Chrome, Firefox, Safari (ES6+, Web Audio API)
- **Mobile browsers**: iOS Safari 12+, Chrome Android
- **Fallback**: Graceful degradation for audio if Web Audio API unavailable

## Future Enhancement Ideas
- Add mobile-optimized touch UI overlays
- Implement leaderboard system (localStorage)
- Add more complex boss patterns
- Implement power-up buffing system
- Add visual/audio feedback for combo multipliers
- Optimize for lower-end mobile devices

## Notes for Claude Code
- Single monolithic HTML file - all changes go to `index.html`
- Test gameplay after logic changes to verify balance
- Keep visual rendering synchronized with game update logic
- Performance-critical code: collision detection, render loop
- Canvas coordinates: (0,0) at top-left, X increases right, Y increases down

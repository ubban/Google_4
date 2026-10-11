# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-page HTML application. The game features player movement, weapon switching, smart bombs, enemies with AI, environmental hazards, stage bosses, and a scoring system.

- **Repository**: ubban/google_4
- **Main File**: `index.html` (self-contained game)
- **Type**: Browser-based game (HTML5 Canvas)
- **Language**: JavaScript (embedded in HTML)

## Architecture

### Single File Structure
The entire game is implemented in `index.html` with three main sections:
1. **HTML**: Game container and HUD elements
2. **CSS**: Styling, animations, and responsive layout
3. **JavaScript**: Game engine, game logic, and event handlers

### Code Organization Within index.html

#### HUD Elements (HTML)
- `#container`: Main game container
- `#gameCanvas`: Canvas element (where the game is drawn)
- `#hud`: Head-up display showing stats
  - `#hearts`: Player health indicator (5 hearts max)
  - `#bombs`: Smart bomb counter
  - `#weaponStat`: Current weapon display
  - `#multStat`: Kill multiplier display

#### Styling (CSS)
- **Colors**: Dark blue theme (`#05122b`, `#0b1d3a`)
- **HUD Styling**: Semi-transparent glassmorphic panels with backdrop blur
- **Animations**: 
  - `heartBeat`: Pulsing animation when health is critical
  - `vignettePulse`: Red vignette effect when taking damage
  - `flashHit`/`flashBomb`: Flash effects on impact
- **Responsive**: Uses `clamp()` for viewport-relative sizing

#### JavaScript (Game Engine)

**Core Game Object**: `game`
- Manages canvas, rendering, and game state
- Methods: `render()`, `update()`, `movePlayer()`, `shoot()`, `dropBomb()`, etc.

**Game State Variables**:
- `player`: Current player state (health, bombs, position, etc.)
- `bullets`: Array of active projectiles
- `enemies`: Array of active enemies
- `hazards`: Array of environmental hazards
- `stage`: Current stage configuration
- `score`, `kills`, `multiplier`: Scoring metrics

**Key Classes/Objects**:
1. **Player**: Ship with 5 health, weapon system, bomb capacity
2. **Enemies**: Various types with AI pathfinding and attack patterns
3. **Bosses**: Stage bosses with health bars and special attacks
4. **Weapons**: Multiple weapon types (primary, secondary, spread, laser)
5. **Smart Bombs**: Special attacks that clear screen
6. **Hazards**: Environmental obstacles (ice, wind, etc.)

**Event Handlers**:
- Mouse/touch movement tracking
- Keyboard controls (WASD for movement, Space for shoot, B for bomb)
- Mobile touch support with safeguards

**Rendering Pipeline**:
- Clear canvas and draw background gradient
- Render game objects (player, enemies, bullets, effects)
- Update HUD display
- Handle animations and visual effects

## Game Mechanics

### Controls
- **Movement**: Mouse position or touch (crosshair cursor)
  - WASD keys also supported
- **Shoot**: Left click or space bar
- **Bombs**: Right click or 'B' key
  - Max 3 bombs per stage
  - Clears screen and damages bosses

### Player
- **Health**: 5 hearts (displayed in HUD)
- **Weapons**: Switch between different weapon types via kills
- **Kill Multiplier**: Increases with consecutive kills without taking damage

### Enemies
- **Spawning**: Configurable patterns per stage
- **AI**: Pathfinding toward player or random movement
- **Bullet Patterns**: Some enemies shoot at player
- **Drop Items**: Health pickups, weapon upgrades, bomb pickups

### Bosses
- **Stage Bosses**: Special enemies with large health pools
- **Attack Patterns**: Multi-bullet spreads, tracking, phase changes
- **Defeat**: Boss destruction advances to next stage

### Scoring
- **Kill Points**: Base points per enemy type
- **Multiplier**: Kill chain bonus (resets when hit)
- **Boss Bonus**: Large points for defeating bosses

## Development Guidelines

### Adding New Features

**New Enemy Type**:
1. Define in `stage.enemies` array with:
   - `type`: Enemy identifier
   - `x`, `y`: Starting position
   - `health`: Health points
   - `pattern`: Attack pattern (if applicable)
2. Add rendering logic in `game.render()` for the new type
3. Add collision and behavior logic in `game.update()`

**New Weapon**:
1. Add weapon definition to player weapon system
2. Implement firing pattern in `game.shoot()`
3. Add visual rendering for bullets
4. Configure drop conditions (kill counts, etc.)

**New Stage**:
1. Create stage object with:
   - `name`: Stage identifier
   - `enemies`: Enemy spawn configuration
   - `boss`: Boss configuration
   - `hazards`: Environmental hazards
2. Add to stage progression system

### Code Style
- Use descriptive variable names (avoid single letters except in loops)
- Add comments for complex game logic and algorithms
- Keep rendering separate from game logic where possible
- Use canvas context methods consistently for drawing

### Testing
1. Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge)
2. Use browser DevTools console for debugging
3. Test on different screen sizes (responsive design)
4. Test touch input on mobile devices

### Mobile Considerations
- Viewport meta tags for proper scaling
- Touch event handling with preventDefault for game area
- Safe area insets for notched devices
- Crosshair cursor works with touch input
- Fallback keyboard controls for accessibility

## File Changes and Maintenance

### When Modifying
- Keep all code in `index.html` (single-file structure)
- Maintain CSS organization by purpose (layout, animations, effects)
- Test in browser after each significant change
- Use git commits to track changes

### Performance Notes
- Canvas rendering is optimized for 60 FPS
- Enemy/bullet count may affect performance on lower-end devices
- Consider pooling objects instead of creating/destroying frequently

## Game States
- **Splash Screen**: Initial menu (if implemented)
- **Playing**: Active gameplay
- **Stage Complete**: Boss defeated, ready for next stage
- **Game Over**: Player lost all health
- **High Score**: Final score display

## Future Enhancement Ideas
- Audio system (background music, sound effects)
- Particle effects for explosions
- More complex boss patterns
- Power-ups system
- Leaderboard/score persistence
- Campaign progression across multiple stages

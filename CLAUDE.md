# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up browser game built with vanilla JavaScript and HTML5 Canvas. Players control an aircraft, switch weapons, drop smart bombs, and battle through waves of enemies and stage bosses.

- **Type**: Single-player web game
- **Tech Stack**: HTML5, CSS3, vanilla JavaScript (no frameworks)
- **Entry Point**: `index.html` (complete single-file application)
- **Deployment**: Static HTML file, can be served from any web server

## Architecture

### Single-File Application
The entire game is contained in `index.html` with:
- **HTML markup**: Container div and canvas element
- **CSS styling**: All game UI, animations, and responsive design
- **JavaScript**: Complete game engine (embedded inline)

### Game Engine Components

#### Core Game Loop
- **Canvas rendering**: 2D context drawing at 60 FPS
- **Input handling**: Keyboard (WASD/arrows) and mouse controls
- **Physics**: Basic collision detection, movement, and animations
- **State management**: Player, enemies, bullets, powerups, waves

#### Player Ship
- **Position**: Center-bottom of canvas
- **Health**: Multiple lives system with heart display
- **Weapons**: 
  - Standard bullets (yellow, rectangular)
  - Laser weapon (cyan/light blue, glowing)
  - Weapon switching with cooldown
- **Bombs**: Smart bombs that damage all visible enemies
- **Movement**: Smooth directional movement with screen boundary constraints

#### Enemy System
- **Enemy types**:
  - Standard fighters (waves)
  - Tanks (heavily armored, with health bars)
  - Bosses (stage-ending encounters)
- **Enemy data**: Defined in `ENEMY_TYPES` object with properties like speed, hp, damage
- **Spawning**: Wave-based spawning with configurable timing
- **AI**: Simple directional movement and shooting patterns

#### Bullet System
- **Player bullets**: Fast, rectangular, yellow or laser-colored
- **Enemy bullets**: Slower, circular, red
- **Laser bullets**: Special glow effect with higher damage

#### Visual Effects
- **Glow/shadow effects**: Dynamic based on quality setting (quality === 0 enables glow)
- **Flash effects**: Hit flash (white screen) and bomb flash effects
- **Vignette**: Red edge effect that pulses when player takes damage
- **Health bar rendering**: For tank enemies
- **Targeting reticle**: Mouse-following crosshair

#### HUD Display
- **Stats shown**:
  - Score
  - Current weapon
  - Multiplier (for combo system)
  - Health (heart icons)
  - Bombs remaining
- **Position**: Top-center with responsive sizing
- **Styling**: Semi-transparent dark background with rounded corners

## Key Variables and Structures

### Player Object
```javascript
{
  x, y,           // Position (typically 50%, near-bottom)
  w, h,           // Width/height of player sprite
  hp,             // Current health
  maxHp,          // Maximum health
  weapon,         // Current weapon type (0=standard, 1=laser)
  cooldown,       // Time until next shot allowed
  bombs,          // Number of smart bombs available
  score,          // Player's current score
  multiplier      // Score multiplier from combos
}
```

### Enemy Object
```javascript
{
  x, y,           // Position on canvas
  w, h,           // Width/height of sprite
  type,           // Enemy type (from ENEMY_TYPES keys)
  hp,             // Current health
  cooldown,       // Cooldown for shooting
  color, dark,    // Visual rendering properties
  movePattern     // Optional: AI movement pattern
}
```

### Bullet Object
```javascript
{
  x, y,           // Position
  w, h,           // Dimensions
  vx, vy,         // Velocity
  laser,          // boolean: true if laser weapon
  owner           // 'player' or 'enemy'
}
```

### PowerUp Object
```javascript
{
  x, y,           // Position (dropped from defeated enemies)
  type,           // Type of powerup
  collected       // boolean: if true, remove from array
}
```

## Development Guidelines

### Adding New Features

#### New Enemy Types
1. Add entry to `ENEMY_TYPES` object with properties:
   - `speed`: Movement speed
   - `hp`: Health points
   - `damage`: Damage dealt to player
   - `fireRate`: Shooting frequency
   - `score`: Points for defeating
2. Update enemy spawning logic if special behavior needed
3. Add visual rendering in the enemies loop if custom appearance required

#### New Weapons
1. Update weapon constants/types
2. Modify player firing logic to handle new weapon behavior
3. Add visual rendering for new bullet types
4. Update HUD weapon display

#### New Powerups
1. Add to powerup array during enemy defeat
2. Implement collision detection with player
3. Add visual representation in drawing loop
4. Implement powerup effect in player update logic

### Performance Considerations

- **Quality setting**: The `quality` variable controls glow effects (0 = glow enabled, reduces performance)
- **Canvas size**: Scales responsively; consider mobile performance
- **Object pooling**: Bullets and enemies are created/destroyed frequently
- **Collision detection**: Currently O(n²) approach; optimize if needed for many objects

### Input Handling

- **Keyboard**: WASD or arrow keys for movement
- **Mouse**: Position drives weapon targeting and crosshair display
- **Touch**: Full mobile support with touch-action handling
- **Events**: keydown/keyup for movement state, mousemove for aiming

### Browser Compatibility

- **Modern browsers**: Chrome, Firefox, Safari, Edge (2019+)
- **Mobile**: Full touch support with viewport-fit: cover for notches
- **Canvas API**: 2D context with basic transforms and effects
- **CSS**: Flexbox, clamp(), env(), backdrop-filter

## Common Modifications

### Adjust Game Difficulty
- Modify `ENEMY_TYPES` properties (speed, hp, damage)
- Change wave spawning patterns
- Adjust player health and bomb count

### Change Visual Styling
- **Colors**: Update `fillStyle` and `ctx.shadowColor` values in drawing code
- **Canvas background**: Linear gradient in CSS
- **UI styling**: Modify `.stat` and HUD-related CSS

### Tweak Game Balance
- **Weapon balance**: Adjust firing rates and damage
- **Powerup frequency**: Modify drop rates in enemy defeat
- **Score multiplier**: Adjust multiplier increment/decay
- **Boss mechanics**: Modify boss-specific behavior

## Testing

Since this is a game, manual testing in a browser is primary:
1. Load `index.html` in a web browser (or serve via HTTP)
2. Test basic movement and collision
3. Verify weapon switching and firing
4. Check bomb functionality
5. Test wave progression and boss encounters
6. Verify score and multiplier systems
7. Test on mobile devices for touch responsiveness

## File Structure

```
Google_4/
├── index.html          # Complete game (HTML + CSS + JS)
├── README.md           # Project metadata
└── CLAUDE.md           # This file - codebase documentation
```

## Contributing

When making changes to this codebase:

1. **Understand the game loop**: It runs via requestAnimationFrame at 60 FPS
2. **Test locally**: Open index.html in a browser before committing
3. **Keep it single-file**: Maintain index.html as complete application
4. **Mobile-first**: Test responsiveness on different screen sizes
5. **Performance**: Monitor frame rate, especially with many objects
6. **Comments**: Add explanatory comments for complex game logic

## Notes

- The game uses canvas transforms for rendering the player ship and visual elements
- Enemy bullet patterns are basic (straight lines); extend with curved paths if desired
- Collision detection uses simple bounding box approach; consider pixel-perfect if needed
- The boss system is modular; add new boss types by extending ENEMY_TYPES
- Score multiplier decays when player doesn't defeat enemies consecutively

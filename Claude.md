# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML, CSS, and JavaScript. The game features:
- Player-controlled aircraft with weapon switching
- Multiple enemy types and patterns
- Smart bombs for crowd control
- Stage-based progression with boss encounters
- Responsive UI with health and bomb indicators
- Touch and mouse support for cross-device play

## Quick Start

### Running the Game

1. Open `index.html` in a modern web browser
2. Use **mouse/arrow keys** to move the player aircraft
3. **Left-click or spacebar** to shoot
4. **Right-click or E** to drop bombs
5. Defeat all enemies and the stage boss to advance

### File Structure

```
Google_4/
├── index.html          # Complete game implementation (single file)
├── README.md           # Basic project info
├── Claude.md           # This documentation
└── .git/               # Git repository
```

## Game Architecture

### Core Components (all in index.html)

**1. Game State**
- Player position, health (lives), bomb count
- Score and wave progression
- Enemy registry and active game objects
- Weapon systems (primary, secondary)

**2. Canvas Rendering**
- Game canvas with gradient background (sky effect)
- HUD overlay for stats (health, bombs, weapon, multiplier)
- Visual effects: flash on hit, vignette on damage, health indicator animations

**3. Game Loop**
- Handles player input (mouse, keyboard, touch)
- Updates game state each frame
- Renders all entities
- Collision detection
- Enemy AI and movement patterns

**4. Entity System**
- **Player**: Moves in 2D, shoots, uses bombs, takes damage
- **Enemies**: Various types with different behaviors and patterns
- **Bullets**: Player projectiles with collision
- **Bombs**: Area-of-effect damage with visual effect
- **Pickups**: Power-ups and score bonuses

### Game Mechanics

**Weapons**
- Primary weapon: Rapid-fire bullets
- Secondary weapon: Toggles between patterns (spread, focused fire, etc.)
- Weapon multiplier affects damage output

**Combat System**
- Collision-based damage detection
- Enemy knockback on hit
- Player damage invulnerability frames
- Boss health tracking

**Wave & Boss System**
- Enemies spawn in organized waves
- Boss fights trigger at stage completion
- Increasing difficulty with each stage
- Wave counter and progression tracking

## UI Components

### HUD Display
- **Hearts Display** (❤️): Shows remaining lives, blinks when health critical
- **Bombs Display** (💣): Visual indicator of available bombs
- **Weapon Stat** (⚔️): Current weapon and multiplier
- **Score Display**: Current score
- **Multiplier Stat**: Score multiplier indicator

### Visual Feedback
- **Flash Effect**: White screen flash on player damage
- **Vignette**: Red edge vignette pulses when taking damage
- **Animation**: Heart beat animation when critical health
- **Crosshair Cursor**: Indicates aim direction

## Development Notes

### Browser Compatibility
- Requires modern browser with Canvas and CSS Grid support
- Touch event support for mobile devices
- Uses `requestAnimationFrame` for smooth 60 FPS gameplay

### Performance Considerations
- Efficient collision detection using rectangular bounding boxes
- Bullet pooling to reduce garbage collection
- Canvas drawing optimized for mobile
- Touch action set to 'none' to prevent browser interference

### Responsive Design
- Game scales to fit viewport (clamps between min/max sizes)
- Safe area insets respected for notched devices
- Touch-optimized controls for mobile
- Flexible HUD layout with viewport-relative sizing

### Key Constants (in code)
- **Canvas Size**: Typically 480x640px
- **Player Speed**: Adjustable movement speed
- **Weapon Fire Rate**: Controls bullet spawn frequency
- **Bomb Damage Radius**: Area of effect size
- **Stage Difficulty**: Enemy health and spawn rates scale with progression

## Building & Deployment

### No Build Step Required
This is a single-file HTML game - no compilation or bundling needed:
```bash
# Simply open in a browser
open index.html
# or serve with any static file server
python -m http.server 8000
```

### Deployment
- Copy `index.html` to any static hosting (GitHub Pages, Netlify, Vercel)
- No server-side processing required
- All assets embedded as data URIs and CSS

## Testing

### Manual Testing Checklist
- [ ] Game loads and starts correctly
- [ ] Mouse/arrow key movement works
- [ ] Left-click/spacebar fires weapons
- [ ] Right-click/E deploys bombs
- [ ] Enemies spawn and move correctly
- [ ] Collision detection works (enemy-to-player, bullet-to-enemy)
- [ ] Damage system functions (player takes damage, enemies die)
- [ ] UI updates correctly (score, health, bombs)
- [ ] Boss encounter triggers at stage end
- [ ] Game over when health reaches 0
- [ ] Touch controls work on mobile
- [ ] Responsive layout on different screen sizes

## Common Development Tasks

### Adding New Enemy Types
1. Define behavior pattern in the enemy spawn logic
2. Set health and movement speed
3. Define sprite/visual representation
4. Add to appropriate wave formation

### Adjusting Difficulty
- Modify enemy health values in spawn logic
- Adjust spawn rates and wave timing
- Change boss health thresholds
- Modify damage values for weapons and bombs

### Adding Weapon Variations
- Create new weapon toggle case in fire logic
- Define projectile pattern and spread
- Set fire rate and damage
- Add UI indicator

## Git Workflow

- **Branch**: `claude/inspiring-planck-f7m3wn`
- **Main changes**: Edit `index.html` directly
- **Commit message format**: Clear, descriptive messages documenting what changed and why
- **Push**: Always push to the designated feature branch

## Resources & References

- Canvas API: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API
- Collision Detection: Rectangular bounding box system used throughout
- Touch Events: https://developer.mozilla.org/en-US/docs/Web/API/Touch_events
- RequestAnimationFrame: Game loop timing mechanism

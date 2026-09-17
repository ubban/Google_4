# Claude.md — Sky Fighter Codebase

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 canvas application. Players control a fighter jet, switch between weapons, deploy smart bombs, and battle stage bosses.

- **Type**: Browser-based game (HTML5 Canvas)
- **Entry Point**: `index.html`
- **Language**: HTML, CSS (inline), JavaScript (inline)
- **No external dependencies** - fully self-contained

## Key Features

- **Player Aircraft**: Moves and shoots, with health tracking
- **Weapon System**: Multiple weapon types with switchable power-ups
- **Smart Bombs**: Limited-use screen-clearing ability
- **Enemy Waves**: Progressive difficulty with stage bosses
- **HUD**: Real-time display of health, bombs, weapon type, and multiplier
- **Responsive Design**: Works on desktop and mobile devices
- **Screen Effects**: Hit flash, damage vignette, screen shake

## File Structure

```
Google_4/
├── index.html          # Complete game implementation (inline CSS + JS)
├── README.md           # Minimal project info
└── Claude.md           # This file - codebase documentation
```

## Architecture

### HTML Structure
- **Canvas Container**: `#container` - Fixed viewport for the game
- **Game Canvas**: `#gameCanvas` - Where all game rendering happens
- **HUD Elements**: Status displays for player stats
  - `#hearts` - Health indicator
  - `#bombs` - Smart bomb counter
  - `#weaponStat` - Current weapon display
  - `#multStat` - Score multiplier
- **Effects Layer**: `#vignette` and `#flash` for visual feedback

### Game Loop
The game runs on a continuous canvas animation frame loop that:
1. Updates all entities (player, enemies, bullets, effects)
2. Handles collision detection
3. Manages stage progression and enemy spawning
4. Renders everything to the canvas
5. Updates HUD display

### Core Game Systems

#### Player System
- Single player aircraft at bottom of screen
- Movement via mouse, keyboard, or touch
- Health system (starts with 3 HP)
- Weapon switching and power-ups
- Smart bomb deployment (max 3)

#### Weapon System
- Multiple weapon types (different fire patterns)
- Damage output varies by weapon
- Power-up pickups to enhance weapons
- Weapon indicator in HUD

#### Enemy System
- Waves of enemies with programmed movements
- Enemy bullets and attack patterns
- Enemy destruction triggers score/drops
- Boss encounters at stage milestones

#### Scoring & Multiplier
- Score accumulation from enemy defeats
- Multiplier increases from consecutive hits without damage
- Multiplier resets on player damage
- Visible in HUD (`#multStat`)

## Code Organization (Within index.html)

### CSS Sections
- **Global Styles**: Dark theme, dark color scheme, body/html setup
- **Canvas & Container**: Game viewport styling, fit to screen
- **HUD Styling**: Status bar layout, stat boxes, responsive typography
- **Animations**: Heart beat (low health), vignette pulse (damage), flash effects
- **Responsive**: Uses clamp() for fluid scaling across screen sizes

### JavaScript Sections
Inline script contains:
- **Game State**: Player, enemies, bullets, effects tracking
- **Input Handling**: Mouse, keyboard, touch event listeners
- **Game Logic**: Collision detection, scoring, weapon mechanics
- **Rendering**: Canvas drawing functions for all game elements
- **Animation Loop**: RequestAnimationFrame-based game update

## Development Guidelines

### When Modifying the Game

1. **Canvas Rendering**: All draw calls use canvas context (ctx)
2. **Game Loop**: Updates happen in animation frame; avoid blocking operations
3. **Input**: Multi-input support (mouse + keyboard + touch)
4. **Mobile**: Ensure touch targets are adequate, test responsive layout
5. **Performance**: Keep game logic efficient for smooth 60fps gameplay

### Debugging Tips

- Open browser DevTools (F12) to see console output
- Canvas rendering is visible in real-time; check DevTools to inspect game state
- Test on multiple screen sizes (use responsive design mode)
- Use arrow keys or WASD for movement on desktop
- Click canvas or press space to interact

### Common Tasks

**Adding a new weapon type**:
- Define weapon properties (damage, fire rate, pattern)
- Add rendering code in canvas draw loop
- Create power-up drop that awards the weapon

**Adjusting difficulty**:
- Modify enemy spawn rates and patterns
- Adjust enemy health/damage values
- Tweak stage progression thresholds

**Visual effects**:
- Modify CSS animations (@keyframes sections)
- Adjust canvas drawing colors and shadows
- Update HUD styling with CSS custom properties

## Browser Compatibility

- Modern browsers with HTML5 Canvas support
- Requires ES6+ JavaScript support
- Touch events for mobile devices
- Viewport meta tag for responsive behavior

## Performance Considerations

- Single canvas target for all rendering (efficient)
- Object pooling recommended for high-count entities (bullets, particles)
- No external asset loading - all data/rendering procedural
- Mobile optimization through touch-action: none and responsive scaling

## Build & Deployment

No build process required - simply serve `index.html` as a static file:
```bash
# Local testing
open index.html  # macOS
# or
python -m http.server 8000  # Python 3
```

Then visit `http://localhost:8000` in a browser.

## Git Workflow

- Main branch: `main` (or `master`)
- Development branch: `claude/inspiring-planck-4kk2hi`
- Each feature/fix is committed with descriptive messages
- Latest game build: commit `c4dc8f6` ("Add Sky Fighter: Raiden-style vertical shoot-em-up game")

## Future Enhancement Ideas

- Sound effects and background music (using Web Audio API)
- Save/load game state to localStorage
- High score tracking
- Additional enemy types and boss variants
- Power-up variety (shields, speed boosts)
- Particle effects for explosions
- Mobile-optimized UI improvements
- Accessibility features (keyboard-only play, high contrast mode)

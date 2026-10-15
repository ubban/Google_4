# Sky Fighter - Game Development Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely with vanilla HTML5, CSS3, and JavaScript. The game is a single-file web application (`index.html`) that requires no build process or external dependencies.

### Game Features
- **Vertical scrolling gameplay** with wave-based enemy encounters
- **Weapon system** with dual firing modes (standard bullets and lasers)
- **Smart bomb power-up** for clearing screen and invulnerability
- **Boss battles** with health bars and advanced attack patterns
- **Combo multiplier system** rewarding aggressive play
- **Particle effects** and visual feedback for impacts and explosions
- **Audio support** with dynamic sound management
- **Performance optimization** with quality scaling for frame rate stability
- **Responsive design** supporting mobile and desktop platforms

## Repository Structure

```
Google_4/
├── index.html          # Complete game implementation (1669 lines)
├── README.md           # Project title
└── .git/               # Version control
```

### index.html Architecture

The single HTML file contains:

1. **HTML Structure** (~200 lines)
   - Game canvas container
   - HUD (heads-up display) elements for stats
   - Boss health bar
   - Overlay UI for menus and game states

2. **CSS Styling** (~400 lines)
   - Responsive layout with safe-area-inset support
   - Animations (heartbeat, vignette pulse, combo popups, stage banners)
   - Dark theme (dark blue background, neon accents)
   - Mobile-optimized UI components

3. **JavaScript Game Engine** (~1000 lines)
   - Game state management (running, paused, game over)
   - Player ship mechanics (movement, shooting, invulnerability)
   - Enemy AI and wave generation
   - Boss ship with complex attack patterns
   - Collision detection and hit registration
   - Particle system for visual effects
   - Score and combo tracking
   - Audio management with context resumption
   - Performance monitoring and quality scaling
   - Canvas rendering and animation loop

## Key Game Mechanics

### Player Ship
- Positioned at bottom of screen, controlled by mouse/touch
- Two weapon modes: **Standard bullets** and **Lasers** (toggle-able)
- **Smart bombs**: Limited-use screen-clearing weapons with temporary invulnerability
- **Health**: 3 hearts visible in HUD; game over at 0 health

### Enemy System
- **Wave-based spawning**: Enemies appear in coordinated patterns
- **Enemy types**: Small fighters and tanks with different properties
- **Tanks**: Larger enemies with visible health bars
- **Bullet patterns**: Enemies fire in geometric patterns

### Boss Battles
- **Triggered at stage progression**
- **Complex attack patterns** with phase transitions
- **Health bar display** showing remaining HP
- **Visual distinction** from normal enemies

### Power-ups
- **Life hearts** (red): Restore health
- **Smart bombs** (yellow): Deploy screen-clearing bombs
- **Weapon upgrade** (green): Enhance fire rate

### Scoring System
- **Base points** for defeating enemies
- **Combo multiplier**: Increases with consecutive hits without getting hurt
- **Combo pop animations** showing current multiplier
- **Personal best tracking** with visual notification

## Development Guidelines

### Running the Game

1. **Local Play**
   - Open `index.html` directly in a modern web browser
   - No server or build process required
   - Works on desktop and mobile devices

2. **Testing**
   - Test in Chrome/Firefox/Safari for full compatibility
   - Verify mobile responsiveness (landscape orientation recommended)
   - Check audio context behavior (browser may require user interaction)

### Code Organization

- **Single-file design**: All code is in `index.html` for simplicity and deployability
- **No external dependencies**: Pure vanilla JavaScript (ES6+)
- **Canvas-based rendering**: Direct 2D context drawing
- **requestAnimationFrame loop**: 60fps target with dynamic quality scaling

### Performance Considerations

1. **Quality Levels**
   - `quality = 0`: Glow effects (shadow blur)
   - `quality = 1`: Minimal effects (no shadows)
   - Adjusts automatically if frame rate drops below 30fps

2. **Optimization Techniques**
   - Particle pooling and reuse
   - Quadtree-based collision detection
   - Lazy enemy AI updates
   - Canvas shadow/glow toggling for performance

### Common Development Tasks

#### Adding New Enemy Waves
Locate the wave definition in the `update()` function and add patterns:
```javascript
// Example: Enemies spawn in a diagonal pattern
const newEnemy = { type: 'fighter', x: ..., y: ..., ... }
enemies.push(newEnemy);
```

#### Modifying Enemy Behavior
Enemy types are defined in `ENEMY_TYPES` object. Adjust:
- `hp`: Health points
- `speed`: Movement speed
- `fireRate`: Shooting frequency
- `bulletSpeed`: Enemy bullet velocity

#### Adding New Weapon Modes
Extend weapon toggle logic in player input handling and add bullet generation patterns.

#### Tweaking Game Balance
Key parameters at top of script:
- Player health, weapon damage, bomb power
- Enemy spawn rates and attack patterns
- Combo multiplier progression
- Power-up drop rates

## Browser Compatibility

- **Target**: Modern browsers with ES6 support
- **Canvas support**: Required (2D context)
- **Audio**: Web Audio API for sound effects
- **Responsive**: CSS Grid/Flexbox, Safe Area Inset support

## Audio Implementation

- **Dynamic audio context**: Starts paused, resumes on user interaction
- **Web Audio API**: Creates and plays sound effects
- **Graceful degradation**: Game continues without audio if unavailable

## Mobile Considerations

- **Touch controls**: Mouse/touch event unification
- **Safe area insets**: Respects notches and home indicators
- **Landscape orientation**: Recommended for optimal gameplay
- **Viewport scaling**: Responsive canvas sizing

## Future Enhancement Ideas

- Persistent high score storage (localStorage)
- Different difficulty levels
- Achievement system
- Leaderboard
- Additional boss types
- Power-up variety expansion
- Sound effects and music tracks
- Tutorial mode for new players

## Attribution & History

- **Project**: Sky Fighter
- **Type**: Educational vertical shoot-'em-up game
- **Last Updated**: 2026-10-15
- **Status**: Active development

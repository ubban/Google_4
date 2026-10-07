# Sky Fighter - Codebase Documentation

## Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built with HTML5 Canvas and vanilla JavaScript. Players control an aircraft to defeat enemies, collect power-ups, switch weapons, and defeat stage bosses.

**Technology Stack:**
- Frontend: HTML5 Canvas (rendering engine)
- Language: Vanilla JavaScript (no frameworks)
- Deployment: Single HTML file (`index.html`)

## Project Structure

```
Google_4/
├── index.html          # Single-file game application
├── README.md           # Project README
└── Claude.md           # This documentation
```

## Architecture & Key Components

### Game Core (`index.html`)

The entire game is implemented in a single HTML file containing:

1. **HTML Structure**
   - Viewport configuration for mobile support
   - Canvas element for game rendering
   - HUD (Heads-Up Display) elements for stats
   - Visual effects layers (vignette, flash)

2. **Styling**
   - Dark space-themed UI (#05122b, #0b1d3a)
   - Responsive design using CSS clamp for mobile/desktop
   - HUD stats with semi-transparent backdrop filters
   - Animations for health, weapon display, hit effects

3. **Game Logic (JavaScript)**
   - **Player Control System**: Keyboard/mouse input handling
   - **Weapon System**: Multiple weapon types with switching capability
   - **Enemy Spawning**: Wave-based enemy generation
   - **Collision Detection**: Hit detection for projectiles and enemies
   - **Physics**: Movement, acceleration, explosion effects
   - **Boss Encounters**: Stage boss mechanics
   - **Power-ups**: Health restoration, weapon upgrades, smart bombs
   - **Score & Progression**: Score tracking, level advancement

### Key Game Features

- **Aircraft Controls**: Arrow keys or mouse movement for positioning
- **Weapons**: Multiple selectable weapons with different firing patterns
- **Smart Bombs**: Limited-use screen-clearing attacks
- **Health System**: Heart-based player health with visual indicators
- **Boss Battles**: Stage bosses with unique attack patterns
- **Visual Feedback**: Screen flashes, vignette effects for damage
- **HUD Display**: Real-time stats for score, health, bombs, weapon, multiplier

## Development Notes

### Code Organization Pattern
- All logic is contained within the HTML file's `<script>` tag
- Self-contained implementation for easy deployment and portability
- Canvas rendering loop and game update cycle

### Performance Considerations
- Uses Canvas 2D context for rendering
- Optimized for both desktop and mobile platforms
- Touch-action handling for mobile compatibility

### Mobile Optimization
- Viewport configuration prevents zooming
- CSS clamp() for responsive font sizes
- Safe area insets for notch support (env(safe-area-inset-top))
- Touch-action: none for smooth gameplay

## Development Workflow

### Branch Strategy
- **Main Branch**: Stable release version
- **Development Branch**: `claude/inspiring-planck-yhsdwx` for feature development

### Git Commits
Recent commits:
- "Add Sky Fighter: Raiden-style vertical shoot-em-up game" - Main implementation
- "Initial commit" - Project setup

### Building & Running
1. Open `index.html` in a modern web browser
2. No build step required - runs directly as a single-file web application

## Game Mechanics

### Player Mechanics
- Health: 5 hearts (visible in HUD)
- Smart Bombs: Limited uses per level
- Weapon Selection: Switch between available weapons
- Score Multiplier: Increases with consecutive hits

### Enemy System
- Varied enemy types with different behaviors
- Wave-based spawning progression
- Health drops for player recovery
- Weapon/bomb pickups

### Victory Conditions
- Defeat all enemies and stage boss
- Progress to next stage

## Browser Compatibility

- Modern browsers with HTML5 Canvas support
- Chrome, Firefox, Safari, Edge
- Mobile browsers with touch support

## Future Enhancement Opportunities

- Additional weapon types
- More boss patterns
- Sound effects and music
- Particle effects system
- Difficulty modes
- High score persistence
- Multiplayer support

## Dependencies

**Zero external dependencies** - Game runs entirely in vanilla JavaScript with no libraries or frameworks required.

---

*Documentation generated: 2026-10-07*
*Last Updated: Codebase initialization*

# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built with vanilla HTML5, CSS3, and JavaScript. The game runs entirely in the browser with no external dependencies.

## Architecture

### Technology Stack
- **Language**: Vanilla JavaScript (ES6+)
- **Rendering**: HTML5 Canvas 2D API
- **Styling**: CSS3 with animations and filters
- **Deployment**: Single HTML file (`index.html`)

### File Structure
```
Google_4/
├── index.html      # Complete game implementation (HTML, CSS, JavaScript)
├── README.md       # Project metadata
└── CLAUDE.md       # This documentation
```

## Game Features

### Core Gameplay
- **Player Ship**: Controllable vertical-scrolling fighter aircraft
- **Weapons System**: Multiple weapon types that can be switched during gameplay
- **Smart Bombs**: Special offensive items that deal area damage
- **Boss Battles**: Stage bosses with unique attack patterns
- **Health System**: Lives-based gameplay with visual heart indicators

### HUD Elements
- **Hearts**: Visual representation of remaining lives
- **Weapon Display**: Current weapon indicator with color coding (lime green)
- **Multiplier Stat**: Score multiplier tracker (orange colored)
- **Bombs**: Available smart bombs counter with icon display
- **Vignette Effect**: Screen damage/danger visual indicator (red pulse animation)
- **Flash Effects**: Hit feedback (white flash) and bomb detonation effects

### Visual Design
- **Color Scheme**: Dark blue sky gradient background (#05122b, #0d2f5e to #7fc1ff)
- **UI Colors**:
  - Primary text: White with text shadow
  - Values: Gold (#ffd54f) for general stats
  - Weapons: Lime green (#7CFC00)
  - Multiplier: Light orange (#ff8a65)
- **Animations**: Heartbeat animation for critical health, vignette pulse for damage, flash effects for hits

### Mobile & Responsive
- Viewport-fit cover for notch support
- Safe area insets for notched devices
- Touch-action handling for mobile gameplay
- Responsive font sizing with clamp()
- Cursor changes to crosshair during gameplay

## Code Organization

### HTML Structure
- `<div id="container">` - Main game viewport container
- `<canvas id="gameCanvas">` - 2D rendering surface
- `<div id="hud">` - Heads-up display for game statistics
  - Health hearts (`#hearts`)
  - Current weapon (`#weaponStat`)
  - Score multiplier (`#multStat`)
  - Bomb count (`#bombs`)
- `<div id="vignette">` - Damage indicator overlay
- `<div id="flash">` - Hit/damage flash effect

### CSS Architecture
- Global resets and dark theme configuration
- Fixed positioning for game viewport and HUD
- Flexbox layout for HUD elements
- CSS animations for continuous effects (heartBeat, vignettePulse, flashHit, flashBomb)
- Backdrop filters for glassmorphism effect on HUD
- Responsive units using clamp() for font sizes and spacing

### JavaScript (Embedded in HTML)
- Single application script with complete game logic
- Canvas 2D rendering for game entities
- Event handling for keyboard and mouse input
- Game state management
- Entity update and collision detection loop
- Animation frame management

## Development Notes

### Performance Considerations
- Canvas-based rendering for efficient 2D graphics
- requestAnimationFrame for smooth 60 FPS gameplay
- Touch-action CSS property optimizes mobile input
- Backdrop filter blur creates visual polish on modern browsers

### Styling Principles
- Dark theme with high contrast for visibility
- Use of text shadows for readability on dynamic backgrounds
- CSS filters for drop shadows on game elements
- Glassmorphic HUD with semi-transparent backgrounds and blur effects

### Browser Compatibility
- Modern browser required (CSS Grid, Flexbox, CSS Animations, Canvas 2D)
- Safe area support for notched mobile devices
- Hardware acceleration via CSS transforms

## How to Play

1. **Open** `index.html` in a modern web browser
2. **Control** the ship with keyboard/mouse
3. **Destroy** enemies and bosses
4. **Manage** your weapons and bombs strategically
5. **Survive** stage encounters to progress

## Future Enhancement Ideas

- Multiple difficulty levels
- Progressive enemy waves
- Power-up system (weapon upgrades, shield, score bonuses)
- Sound effects and background music
- Score persistence and leaderboards
- Tutorial/help screen
- Settings menu for controls and visual preferences

## Commit History

- **Initial commit**: Project setup
- **Latest**: Sky Fighter game implementation with full gameplay features

## Notes for Claude Code

When working on this project:
1. The entire game is contained in a single `index.html` file
2. All CSS is inline in `<style>` tags
3. All JavaScript is inline in `<script>` tags
4. Changes require editing the monolithic HTML file
5. Test changes by opening the HTML file in a browser
6. The game is fully functional - focus on gameplay balance, new features, or visual polish

# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely with HTML5, Canvas, and vanilla JavaScript. It features a dynamic difficulty system, weapon switching mechanics, smart bomb power-ups, and multi-stage boss encounters. The game is fully responsive and supports both desktop keyboard and mobile touch controls.

**Repository**: ubban/Google_4  
**Branch for Development**: `claude/inspiring-planck-qx9dvg`

## Game Mechanics

### Core Gameplay
- **Genre**: Vertical shoot-'em-up (shmup) with Raiden influence
- **Controls**: 
  - Desktop: Arrow keys or mouse/cursor movement to aim and navigate
  - Mobile: Touch controls with virtual directional pad and action buttons
- **Objective**: Survive enemy waves, destroy bosses, and achieve high scores across multiple stages

### Player Systems
- **Health**: 3 lives by default (displayed as hearts in HUD)
- **Weapons**: Multiple weapon types with individual ammo counts
  - Weapon switching via specific inputs
  - Weapon stat tracking (displayed in HUD)
- **Smart Bombs**: Limited-use screen-clearing power-ups (displayed as bomb count in HUD)
- **Multiplier System**: Combo multiplier that increases score for consecutive hits/kills

### Enemy Systems
- **Enemy Types**: Various enemy patterns and behaviors
- **Boss Encounters**: Stage bosses with health bars and distinct attack patterns
- **Score System**: Enemy defeats contribute to score with multiplier application

### Visual Polish
- **HUD Elements**:
  - Score, Level, Lives (hearts), Bombs, Weapon type, Multiplier display
  - Boss health bar (shown only during boss battles)
  - Combo notifications (large text pop-ups on successful chains)
  - Stage banners
- **Visual Effects**:
  - Damage vignette (red pulsing overlay when player takes damage)
  - Screen flash effects (hit/bomb)
  - Combo pop-up animations
  - Gradient sky background

## Technical Architecture

### Single-File Structure
The entire game is contained in `index.html`:
- **HTML Structure**: Canvas element (#gameCanvas) for game rendering
- **Embedded CSS**: Full styling for responsive design
- **Embedded JavaScript**: Complete game logic inline

### Game Loop Components
The game uses a canvas-based rendering loop with:
- Input handling (keyboard and touch)
- Game state management
- Physics and collision detection
- Entity updates and animations
- Canvas rendering

### Responsive Design
- Viewport meta tags for mobile support
- CSS clamp() for fluid typography and spacing
- Safe area insets for notched devices (env(safe-area-inset-*))
- Touch-action controls with on-screen button zones
- Cross-browser compatibility

## File Structure

```
Google_4/
├── index.html        # Complete game application (HTML + CSS + JS)
├── README.md         # Project metadata
└── CLAUDE.md         # This file
```

## UI Components

### HUD Layout
- **Top Center**: Score, Level, Lives (hearts), Bombs count, Weapon info, Multiplier
- **Top Right**: Icon buttons (settings, etc.)
- **Boss Encounters**: Boss name and health bar (centered near top)
- **Pop-ups**: Stage banner, combo notifications

### Overlay System
Modal overlay for:
- Main menu/title screen
- Game over screen with statistics
- Difficulty selection
- Instructions and controls

### Touch Controls
- **Move Zone**: Left side of screen for directional input
- **Action Buttons**: Right side for firing, bombs, and weapon switching
- **Responsive Layout**: Adjusts for different screen sizes

## Game States

- **Menu**: Initial state showing title and difficulty selection
- **Playing**: Active gameplay
- **Boss**: Playing during boss encounter (health bar visible)
- **Game Over**: Results screen with score, bonus, and restart option

## Development Priorities for Claude

### Feature Additions
- New enemy types and patterns
- Additional weapon types
- New power-up mechanics
- Stage progression variety
- Audio/music system

### Quality Improvements
- Performance optimization for mobile devices
- Enhanced collision detection
- Improved AI patterns
- Better touch input responsiveness
- Accessibility features

### Content Expansion
- More stages and bosses
- Leaderboard integration
- Achievement system
- Difficulty adjustments
- Cosmetic customization

## Key CSS Classes & IDs

- `#gameCanvas`: Main rendering canvas
- `#hud`: Heads-up display container
- `#bossBarWrap`: Boss health bar container
- `#overlay`: Modal overlay system
- `#touchControls`: Mobile control interface
- `#comboPop`: Combo notification element
- `#vignette`: Damage effect visual

## JavaScript Conventions

- Canvas 2D context rendering
- Event listeners for input (keydown, keyup, touch events)
- RAF-based game loop
- Entity collections for players, enemies, projectiles
- State management for game phases

## Notes for Maintenance

- The game is responsive and works on desktop, tablet, and mobile
- All styling is viewport-aware with clamp() functions
- Touch controls are optional; keyboard input works on any device with a keyboard
- The game uses a gradient background (sky theme) with responsive canvas sizing
- No external dependencies - pure vanilla HTML5, CSS, and JavaScript

## Running the Game

Simply open `index.html` in a modern web browser. The game will auto-scale to fit the viewport and support both keyboard and touch controls based on device capabilities.

---

*Last Updated: 2026-10-10*  
*Documented for Claude Code agent workflow*

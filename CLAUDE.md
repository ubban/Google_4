# Sky Fighter Codebase Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented in HTML5 Canvas. Players control a fighter jet, switch between weapons, deploy smart bombs, and battle stage bosses. The game is fully self-contained in a single `index.html` file.

## Project Structure

```
Google_4/
├── index.html          # Complete game implementation (HTML + embedded CSS + JavaScript)
├── README.md           # Project metadata
└── .git/              # Git repository
```

## Technology Stack

- **Language**: JavaScript (ES6+)
- **Rendering**: HTML5 Canvas
- **Styling**: Inline CSS (dark theme, #05122b background)
- **Target**: Web browsers (responsive, mobile-friendly with touch support)

## Game Features

- Vertical scrolling shoot-'em-up gameplay
- Player-controlled fighter jet with crosshair cursor
- Weapon switching system
- Smart bomb deployment
- Multiple stage bosses
- Score tracking and HUD display
- Responsive canvas sizing with safe-area insets (mobile notch support)
- Touch action support for mobile devices

## Development Guidelines

### Code Organization

The game code is contained within `index.html` in a single `<script>` tag. The structure follows:
1. **CSS Styles**: Game styling, HUD layout, canvas styling
2. **JavaScript**: Game engine, entity management, rendering, input handling
3. **Game Loop**: Implements frame-based animation with canvas rendering

### Key Game Mechanics

- **Movement**: Player controls via mouse position (desktop) or touch (mobile)
- **Weapons**: Multiple weapon types with switching capability
- **Bombs**: Limited smart bombs for crowd control
- **Enemies & Bosses**: Entity-based rendering and collision detection
- **Scoring**: Dynamic score calculation based on enemy elimination

### Canvas & Rendering

- Canvas element: `#gameCanvas`
- Game container: `#gameCanvas` inside `#container`
- HUD display: `#hud` with stats (Score, Lives, Weapon, Bombs)
- Responsive sizing: `max-width: 100%`, `max-height: 100%`, scaling via CSS `clamp()`

### Mobile Support

- Touch events handled via `touch-action: none`
- Safe-area insets for notched devices
- Viewport settings optimized for mobile (no zoom, viewport-fit=cover)
- Tap highlight disabled for better UX

## Running the Project

### Browser

1. Open `index.html` directly in a web browser
2. For local development, use a simple HTTP server:
   ```bash
   python3 -m http.server 8000
   # or
   npx http-server
   ```
3. Navigate to `http://localhost:8000`

### Desktop

- Supports mouse-based aiming with keyboard/mouse controls
- Full-screen capable on most browsers

### Mobile

- Touch-based aiming
- Optimized for landscape orientation
- Respects device notches and safe areas

## Git Workflow

- **Branch**: `claude/inspiring-planck-vk0gdu` (primary development branch)
- **Commits**: Include descriptive messages and co-authorship attribution
- **Versioning**: Track game feature additions and bug fixes

### Recent Commits

- `c4dc8f6`: Add Sky Fighter - Raiden-style vertical shoot-em-up game
- `4efdad1`: Initial commit

## Future Development

Potential enhancement areas:
- Additional weapon types and power-ups
- More stage bosses with varied attack patterns
- Sound effects and background music
- Difficulty scaling
- Leaderboard system
- Accessibility improvements (colorblind mode, remappable controls)

## Notes

- Game state is contained in JavaScript runtime; no persistent storage currently
- Performance optimized for 60 FPS on modern browsers
- Supports both portrait and landscape orientations (optimized for landscape)

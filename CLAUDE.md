# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely with vanilla HTML5, CSS3, and JavaScript. No frameworks or build tools required — the entire game runs from a single `index.html` file.

### Game Features
- **Vertical scrolling gameplay** with player-controlled aircraft
- **Multiple weapon systems** with switching capabilities
- **Smart bomb mechanic** for crowd control
- **Progressive difficulty** across multiple stages with unique boss encounters
- **Responsive design** supporting both keyboard and touch/mobile input
- **Real-time HUD** displaying health, score, combo multiplier, and weapon status
- **Stage progression** with boss encounters and visual feedback
- **Smooth animations** including hit effects, flash effects, and vignette damage indicators

## Repository Structure

```
Google_4/
├── index.html          # Single-file game implementation
├── README.md           # Project metadata (timestamp only)
└── .git/               # Version control
```

### File Organization Within index.html

The `index.html` file is organized into distinct sections:

1. **HTML Markup** (top): Game container and UI elements
2. **CSS Styling** (embedded): Game visuals, animations, responsive design
3. **JavaScript Game Logic** (embedded): Core game engine, game loop, and mechanics

## Game Architecture

### Core Systems

#### Canvas & Rendering
- Single canvas element (`#gameCanvas`) for all game rendering
- Centered, responsive design with safe-area inset support
- Background gradient: sky blue to light blue (typical vertical shooter aesthetic)

#### Game State Management
- Stores player position, health, score, weapon selection, bombs
- Tracks enemy list, bullets, explosions, and visual effects
- Manages game phases: menu, playing, game over

#### Game Loop
- Handles input processing
- Updates all game objects (player, enemies, bullets, effects)
- Renders canvas and HUD
- Runs at requestAnimationFrame cadence

#### Collision Detection
- Bullet-enemy collisions (with damage application)
- Enemy-player collisions (with health loss)
- Manages hit animations and screen shake effects

### UI/HUD Elements

Positioned absolutely over the canvas:
- **Top Center**: Score, health (hearts), weapon name, combo multiplier
- **Top Right**: Control buttons (mute/unmute, menu)
- **Boss Bar** (when active): Enemy name and health bar
- **Overlays**: Title screen, game over, stage banners, combo popups
- **Visual Effects**: Flash overlays, vignette damage indicator, stage transitions

### Input Handling

**Keyboard Controls:**
- Arrow keys or WASD for movement
- Space to fire main weapon
- 1-3 to switch weapons
- B for smart bombs
- P to pause

**Touch/Mobile Controls:**
- Left side: Movement zone (virtual joystick area)
- Right side: Fire zone (tap to shoot)
- Special button areas for weapon switching and bombs

### Animation System

CSS animations for:
- Combo popup burst effect (`comboPulse`)
- Stage banner fade-in (`stagePulse`)
- Health heart beat on low HP (`heartBeat`)
- Vignette pulse on player hit (`vignettePulse`)
- Hit/bomb flash effects (`flashHit`, `flashBomb`)
- Card entrance animation (`cardIn`)
- Icon bobbing (`iconBob`)

## Key Game Mechanics

### Weapons
- **Gun**: Standard rapid-fire projectile
- **Laser**: Continuous beam weapon
- **Spread**: Multi-projectile shots
- Switching with number keys (1-3) or on-screen buttons

### Scoring System
- Base points from enemy destruction
- Combo multiplier that increases with consecutive hits
- Multiplier resets on player damage
- Score persistence (best score saved/displayed on game over)

### Difficulty Levels
- **Easy**: Reduced enemy health and fire rate
- **Normal**: Standard difficulty
- **Hard**: Increased enemy health, density, and aggression

### Boss Encounters
- Special boss enemies appear at stage transitions
- Boss health bar displayed at top center
- Unique behavior patterns (more aggressive, complex movement)

### Smart Bombs
- Limited quantity (display with bomb counter)
- Clear all on-screen enemies and bullets
- Visual feedback: white flash overlay

## Development Notes

### No Dependencies
- Pure vanilla JavaScript (no frameworks)
- No build process or transpilation needed
- Cross-browser compatible (modern browsers with Canvas support)

### Responsive Design
- Uses `clamp()` for scalable font sizes and spacing
- `viewport-fit: cover` for notch/safe-area support
- `max-width: 100%` and `max-height: 100%` for canvas scaling
- Works on desktop, tablet, and mobile devices

### Performance Considerations
- Canvas rendering (2D context)
- Object pooling could improve performance for large bullet counts
- requestAnimationFrame for 60 FPS target
- Efficient collision detection with early exit conditions

### Customization Points

Key values to modify for balancing:
- Player speed and fire rate
- Enemy health and damage values
- Enemy spawn rate and behavior patterns
- Stage difficulty progression
- Combo multiplier calculation
- Weapon damage and fire rates

## Running the Game

Simply open `index.html` in a modern web browser. No server or build tools required.

Tested on:
- Chrome/Chromium (latest)
- Firefox (latest)
- Safari (iOS and macOS)
- Edge (latest)

## Future Enhancement Ideas

- Persistent high score storage (localStorage)
- Sound effects and music
- Additional weapon types or special abilities
- Leaderboard integration
- Achievements/badge system
- Power-up items (damage boost, shield, rapid fire)
- Boss variation within stages
- Difficulty modifiers (time limit, restricted weapons)
- Tutorial/training stage
- Accessibility features (colorblind modes, screen reader support)

## Git History

- **Latest**: Added Sky Fighter: Raiden-style vertical shoot-em-up game
- **Initial**: Repository setup

## Contributing

When making changes to the game:
1. Test thoroughly on multiple browsers and devices
2. Verify responsive behavior at different viewport sizes
3. Ensure touch and keyboard controls work as expected
4. Check performance impact of new features
5. Update this documentation for significant changes

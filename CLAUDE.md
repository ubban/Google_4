# Sky Fighter: Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up browser game built with vanilla HTML5 Canvas and JavaScript. Players control an aircraft, switch between weapon types, deploy smart bombs, and battle stage bosses across increasing difficulty levels.

## Repository Structure

```
Google_4/
├── index.html          # Single-file game implementation
├── README.md           # Project readme
└── CLAUDE.md          # This file
```

## Technology Stack

- **Frontend:** HTML5, CSS3, Canvas API
- **Language:** JavaScript (ES6+)
- **Platform:** Browser-based, responsive design
- **Performance:** 60 FPS target with canvas rendering

## Game Architecture

### Core Systems

1. **Game State Management**
   - Main game loop with 60 FPS target
   - Game states: title screen, playing, paused, game over
   - Wave/stage progression system

2. **Player Entity**
   - Position and velocity tracking
   - Multiple weapon types (bullets, lasers, missiles)
   - Health system with visual heart indicators
   - Smart bomb mechanic with limited uses

3. **Combat System**
   - Enemy spawning with wave patterns
   - Boss encounters at stage milestones
   - Collision detection for bullets, enemies, and explosions
   - Damage calculation and knockback

4. **Scoring & Progression**
   - Score tracking and display
   - Combo/multiplier system
   - Difficulty scaling per stage
   - Power-up drops from defeated enemies

5. **Visual & Audio**
   - Canvas-based rendering
   - CSS animations for UI elements (heart beat, flash effects)
   - Vignette effect on critical health
   - Weapon indicator colors

### Key Game Mechanics

- **Weapon Switching:** Cycle between weapon types (default bullets, advanced laser, missiles)
- **Smart Bombs:** Limited-use screen-clearing ability
- **Health System:** 5-heart life system with visual representation
- **Multiplier:** Increased by consecutive hits, affects score
- **Enemy Waves:** Progressively harder enemy patterns
- **Boss Battles:** Special stage bosses with unique attack patterns

## HTML Structure

The entire game is contained in a single `index.html` file with three main sections:

1. **Head Section:**
   - Meta tags for viewport, description, theme color
   - Embedded CSS styles
   - Favicon (inline SVG)

2. **Body Section:**
   - Game canvas container
   - HUD (heads-up display) showing stats
   - Vignette effect overlay
   - Flash effect overlay

3. **Script Section:**
   - Embedded JavaScript game logic
   - Asset initialization (inline sprites/images)
   - Event listeners for input

## Development Notes

### Styling Approach
- Dark theme with blue gradient background
- Responsive design using `clamp()` for font sizing
- Safe area insets for mobile/notch support
- Overlay effects for game feedback (flash, vignette)

### Input Handling
- Mouse/touch position tracking for aiming
- Keyboard input for weapon cycling and bomb deployment
- Touch-action prevention for smooth gameplay

### Performance Considerations
- Canvas rendering for fast 2D graphics
- Efficient collision detection algorithms
- Enemy pooling/recycling to reduce garbage collection
- RequestAnimationFrame for smooth animations

## Git History

- **Latest:** Add Sky Fighter: Raiden-style vertical shoot-em-up game
- **Initial:** Initial commit

## Configuration & Build

- No build process required - single HTML file
- Can be opened directly in any modern browser
- No external dependencies
- Works offline once loaded

## Mobile Support

- Responsive canvas scaling
- Touch controls (aiming via touch position)
- Safe area insets for notched devices
- Maximum viewport zoom prevention
- Optimized hit areas for touch gameplay

## Future Enhancement Opportunities

1. Sound effects and background music
2. Particle effects for explosions
3. Additional weapon types and power-ups
4. More boss variety and attack patterns
5. Local high score persistence
6. Mobile app wrapper (PWA or Cordova)
7. Leaderboard integration
8. Additional game modes (survival, time attack)

## Testing & Debugging

To test the game:
1. Open `index.html` in a modern web browser
2. Click on canvas to start
3. Move mouse/touch to aim
4. Click/tap to shoot
5. Use keyboard for weapon cycling and bombs
6. Test across different screen sizes and devices

## Code Style Guidelines

- Vanilla JavaScript with no external libraries
- Modular function organization within single file
- Clear variable naming for game entities
- Canvas context management for rendering
- Input state management for responsive controls

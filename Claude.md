# Google_4: Sky Fighter Game

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with HTML5 Canvas and vanilla JavaScript. The game features:
- Player-controlled aircraft with weapon switching mechanics
- Smart bomb capability for crowd control
- Stage-based progression with boss encounters
- Responsive design supporting desktop and mobile devices
- Touch and keyboard controls

## Architecture

### Single-Page Application
- **Single HTML file** (`index.html`) containing all game logic, styling, and markup
- No external dependencies - fully self-contained
- Canvas-based rendering for game graphics
- Responsive viewport configuration with safe-area-inset support

### Key Components

#### Game Engine
- Canvas rendering system
- Game loop for updates and rendering
- Entity management (player, enemies, projectiles, effects)
- Collision detection
- Particle effects and visual feedback

#### Player Mechanics
- Weapon system with switching capabilities
- Health and shield system
- Smart bomb special ability
- Movement controls via keyboard and touch/mouse

#### Enemy System
- Stage-based spawning
- Boss encounters
- Projectile attacks
- Destruction animations and score rewards

#### HUD Elements
- Score display
- Health/Shield indicators
- Weapon status
- Stage/Wave information
- Game state messages

### Visual Design
- Dark blue gradient background (`#05122b` to `#1e6fd8` to `#7fc1ff`)
- Crosshair cursor for aiming
- Text-shadow styling for readability over dynamic backgrounds
- Responsive sizing using CSS clamp() and viewport units

## Technical Details

### Styling Features
- Dark color scheme (`color-scheme: dark`)
- Flexible HUD using flexbox
- Safe area inset support for notched devices
- Touch action optimization (`touch-action: none`)
- Canvas clipping and shadows for visual depth

### Game Input
- Keyboard controls (arrow keys, spacebar, weapon switching)
- Mouse/Touch controls for aim and movement
- Responsive input handling without page scrolling

### Performance Considerations
- Single canvas rendering for efficiency
- Requestanimationframe-based game loop
- Touch-action disabled to prevent browser interference
- Optimized particle effects and animations

## Development Notes

### Code Organization
All code is in `index.html`:
1. HTML structure (container, canvas, HUD elements)
2. CSS styling (layout, typography, animations)
3. JavaScript game logic (inline script tag)

### Key Game Variables and Systems
- Game state management (playing, paused, game over, victory)
- Player object with position, health, weapons
- Enemy arrays for waves and boss battles
- Particle systems for visual effects
- Score and combo tracking

### Extending the Game
When adding features:
- Update HUD elements in HTML if needed
- Add CSS for new visual components
- Extend game objects and systems in the JavaScript
- Test responsiveness across device sizes

## Running the Game

### Browser Launch
Simply open `index.html` in a web browser:
- Desktop: Works with any modern browser (Chrome, Firefox, Safari, Edge)
- Mobile: Responsive design adapts to different screen sizes
- Touch controls work on tablets and smartphones

### Testing Considerations
- Test on multiple viewport sizes
- Verify keyboard and touch input responsiveness
- Check performance on lower-end devices
- Validate audio cues and visual feedback

## Project State

**Status**: Active game implementation with core mechanics  
**Last Updated**: 2026-02-23  
**Branch**: `claude/inspiring-planck-w2sj68`  
**Repository**: ubban/google_4

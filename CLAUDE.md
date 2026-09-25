# Claude.md — Google_4 Codebase Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML/JavaScript web application. The game features:
- Vertical scrolling shooter gameplay
- Weapon switching mechanics
- Smart bomb mechanics
- Boss encounters
- Touch and mouse input support
- Responsive design with viewport-fit cover for mobile devices

## Project Structure

```
Google_4/
├── index.html          # Single-file web game (complete game implementation)
├── README.md           # Basic project metadata
└── CLAUDE.md           # This file - Claude Code documentation
```

## Key Files

### index.html (~59KB)
The entire game implementation in a single HTML file containing:
- **Structure**: Canvas-based rendering with HUD overlay
- **Styling**: Dark space theme with gradient backgrounds, responsive sizing using clamp()
- **JavaScript Game Engine**: 
  - Game state management
  - Canvas rendering loop
  - Input handling (mouse, touch, keyboard)
  - Entity management (player, enemies, bullets, effects)
  - Physics simulation and collision detection
  - Audio playback
  - UI elements and menus

## Architecture Notes

### Rendering
- Uses HTML5 Canvas for game graphics
- 2D context for drawing sprites, particles, and UI
- Adaptive sizing with max-width/max-height constraints
- Fixed viewport with flex centering in container

### Input Handling
- Mouse/pointer tracking for targeting and movement
- Touch input support with viewport fit for mobile
- Keyboard controls (likely for alternative input)
- Crosshair cursor for visual feedback

### Game Systems
- Stage-based level progression with boss encounters
- Weapon system with switching mechanics
- Power-up and bomb management
- Particle effects for visual feedback
- HUD display showing stats (score, health, weapons, bombs)

### Mobile Optimization
- viewport-fit=cover meta tag for notch/dynamic island support
- Touch-action: none prevents default browser touches
- Responsive font sizing with clamp()
- Safe area insets for UI positioning

## Development Guide

### To Modify the Game
1. The entire game logic is in `index.html`
2. Locate the relevant code section:
   - Styles: Look for `<style>` tags near the top
   - Game initialization: Search for canvas setup code
   - Game loop: Look for requestAnimationFrame or setInterval
   - Input handling: Search for addEventListener (mousemove, touchmove, etc.)
   - Rendering: Look for ctx.drawImage, ctx.fillRect, etc.
   - Game entities: Player, Enemy, Bullet classes or equivalent

### Testing Changes
- Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge)
- Test on desktop with mouse/keyboard
- Test on mobile devices (iOS Safari, Android Chrome) for touch input
- Verify viewport scaling and notch/safe area handling

### Browser Compatibility
- Requires HTML5 Canvas API
- Modern event handling (touch, pointer events)
- CSS Grid/Flexbox support
- ES6+ JavaScript (if using modern syntax)

## Common Tasks

### Adding/Modifying Game Content
- Game stages, enemy patterns, boss AI, weapon properties
- Look for data structures or constants defining these elements
- May be embedded in the game loop or as separate configuration sections

### Adjusting Gameplay Balance
- Player health, movement speed, firepower
- Enemy spawn rates, health, behavior patterns
- Bomb effectiveness and cooldown
- Look for numeric constants and adjustable parameters

### Visual/UI Changes
- CSS styling in the `<style>` section
- Canvas drawing calls for game entities
- HUD layout and formatting
- Color scheme and effects

### Audio Integration
- Audio file loading and playback
- Sound effect triggering during gameplay
- Background music loops
- Look for Audio() or AudioContext references

## Performance Considerations

- Single-file deployment (no separate resources except audio)
- Canvas rendering performance is primary bottleneck
- Entity culling/management for large numbers of bullets/enemies
- Touch/pointer event debouncing for mobile smoothness

## Attribution & Workflow

- Use branch: `claude/inspiring-planck-gw8ut0`
- Commit messages should be descriptive and end with:
  ```
  Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
  Claude-Session: https://claude.ai/code/session_014X2eCFnyWMpfSUAbjq2rPC
  ```
- Push to the designated branch for code review

## Notes for Future Sessions

- This is a single-file web game - changes are isolated within index.html
- Always test gameplay changes in an actual browser before committing
- Mobile testing is important due to touch input and viewport sizing
- The game likely uses requestAnimationFrame for smooth 60fps gameplay

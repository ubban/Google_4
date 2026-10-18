# Sky Fighter - Codebase Documentation

**Project**: Sky Fighter  
**Type**: Browser-based Game (HTML/Canvas)  
**Genre**: Vertical Shoot-'em-up (Raiden-style)  
**Entry Point**: `index.html`

## Overview

Sky Fighter is a vertical shoot-'em-up arcade game built with vanilla HTML5, Canvas API, and JavaScript. The game features:
- Player-controlled aircraft with weapon switching mechanics
- Smart bomb ability for area damage
- Boss encounters at stage completion
- Score multipliers and progression system
- Responsive controls (keyboard and touch/mouse)
- Dark themed UI with HUD displaying health, bombs, weapon type, and score multiplier

## Project Structure

```
Google_4/
├── index.html       # Main game file (game logic, canvas rendering, event handling)
├── README.md        # Project metadata
└── CLAUDE.md        # This file
```

## Technical Architecture

### Single-File Design
The entire game is contained within `index.html` as a monolithic SPA with:
- **HTML**: Canvas container, HUD elements (health, bombs, weapon, multiplier displays)
- **CSS**: Game styling, animations (heartbeat, vignette pulse, flash effects), responsive design
- **JavaScript**: Game engine, physics, collision detection, rendering loop

### Key Game Mechanics

#### Player
- Health system: 3 hit points (visualized as hearts in HUD)
- Smart bombs: Limited quantity per stage (3 default), area-of-effect damage
- Weapon system: Multiple weapon types with different firing patterns
  - Primary: Spread shot
  - Secondary: Laser beam
  - Score multiplier increases with consecutive hits

#### Enemies & Bosses
- Enemy waves with varying patterns and bullet types
- Boss encounters that feature multiple health bars and special attack patterns
- Collision detection between:
  - Player bullets ↔ Enemies
  - Player ↔ Enemy bullets
  - Player ↔ Enemies (body contact)

#### Controls
- **Keyboard**: Arrow keys or WASD for movement, Space to shoot, B for bomb
- **Touch/Mouse**: Click/tap to move towards cursor, auto-shooting enabled
- Responsive hitbox for easier mobile play

#### Visual Effects
- Canvas gradient background (day sky effect)
- Damage vignette (red pulse when taking hits)
- Flash screen (white flash on player hit, brighter flash on bomb)
- Sprite rotation for directional movement
- Particle/explosion effects on enemy destruction

### Canvas Rendering Pipeline
1. Clear canvas with gradient background
2. Render all entities (enemies, bullets, effects)
3. Apply visual overlays (vignette, flash effects)
4. Update HUD elements
5. Request next animation frame

### Game Loop
- Main update cycle runs at 60 FPS via `requestAnimationFrame`
- Input polling for continuous movement
- Collision checks every frame
- Score calculation based on multiplier chains

## Development Notes

### Browser Compatibility
- Requires HTML5 Canvas support
- Works on desktop (Chrome, Firefox, Safari, Edge) and mobile (iOS Safari, Chrome)
- Touch support for mobile devices with safe-area-inset considerations

### Performance Considerations
- Canvas-based rendering (no DOM overhead for game objects)
- Sprite pooling and efficient collision detection
- Viewport-fit cover for immersive full-screen experience

### Accessibility Features
- Color-coded HUD elements with text labels
- Keyboard and touch input support
- User-select disabled to prevent accidental text selection during gameplay
- Tap highlight disabled for better mobile UX

## Future Enhancement Opportunities

- **Difficulty Levels**: Add selectable difficulty with adjusted enemy patterns
- **Sound Effects**: Add audio for shooting, explosions, boss fights
- **Visual Polish**: Add screen shake on explosions, improved particle effects
- **Mobile Optimizations**: Add on-screen button controls for better mobile accessibility
- **Persistence**: Save high scores to localStorage
- **Additional Content**: More boss designs, enemy types, weapon variations
- **State Management**: Decouple game state from rendering for better maintainability

## Quick Start

1. Open `index.html` in any modern web browser
2. Use arrow keys/WASD to move, Space to shoot, B for bomb
3. Defeat waves of enemies and bosses to progress
4. Try to achieve the highest score multiplier possible

## File Dependencies

- **No external libraries** - Game uses only vanilla JavaScript and Canvas API
- **No build process** - Single HTML file can be deployed as-is
- **No network dependencies** - Entirely client-side

---

**Last Updated**: 2026-10-18  
**Maintainer**: Claude Code Agent

# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-page web application. The game features a player-controlled fighter aircraft that must navigate through waves of enemies, collect power-ups, and defeat stage bosses.

### Key Features
- **Vertical scrolling gameplay** with dynamic enemy waves
- **Weapon system** with multiple weapon types (default shots, spread weapon, laser, guided missiles)
- **Power-up mechanics** including score multiplier, health recovery, and smart bombs
- **Boss encounters** at the end of each stage
- **Responsive design** supporting both desktop and mobile/tablet touch controls
- **Visual effects** including hit flashes, vignette effects, particle explosions, and screen shake

## Project Structure

```
Google_4/
├── index.html          # Main game file (HTML + CSS + JavaScript)
├── README.md           # Project info
└── CLAUDE.md           # This file
```

## Technical Architecture

### Single-File Design
The entire game is contained in `index.html` as a self-contained HTML5 Canvas application with:
- **HTML**: DOM structure with canvas element and HUD overlays
- **CSS**: Styling for canvas, HUD stats display, and animations
- **JavaScript**: Complete game logic, engine, and physics

### Key Game Systems

#### Rendering Engine
- Uses HTML5 Canvas 2D context for all graphics
- Resolution: 480×640px (logical), scaled to fit viewport
- Dynamic frame rate adjustment based on canvas size
- Double-buffering for smooth animation

#### Game Objects
- **Player**: Controllable fighter with health, weapon system, and movement
- **Enemies**: Multiple types with different behaviors and attack patterns
- **Bosses**: Stage-final enemies with multi-phase health and complex attack patterns
- **Projectiles**: Player bullets, enemy bullets, and bomb effects
- **Particles**: Visual effects for explosions, hits, and power-ups

#### Input Handling
- **Desktop**: WASD keys for movement, mouse crosshair aiming
- **Mobile**: Touch controls with joystick simulation and drag-to-aim
- **Full-screen button** for immersive gameplay
- Touch-optimized HUD with no accidental tap highlights

#### Game States
1. **Title Screen**: Game start and instructions
2. **Stage Intro**: Wave/boss introduction screen
3. **Playing**: Active gameplay
4. **Boss Battle**: Special stage for boss encounters
5. **Level Complete**: Score/stats display between levels
6. **Game Over**: Final score and restart option

### Audio System
- Uses Web Audio API for sound effects and music
- Supports procedural sound generation
- Master volume and mute controls

### Physics & Collision
- Simple rectangular collision detection for gameplay
- Enemy waves with timing-based spawning
- Projectile despawning at screen edges
- Screen shake effect on player hit
- Vignette effect with pulsing animation during damage state

## Game Flow

### Per-Frame Loop
1. **Update phase**: Process input, update entities, check collisions
2. **Render phase**: Clear canvas, draw game objects, update HUD
3. **Animation frame**: Request next frame via requestAnimationFrame

### Gameplay Progression
- Multiple stages with increasing difficulty
- Enemy wave spawning based on game time
- Score-based multiplier system
- Health system with recovery power-ups
- Smart bomb limited-use special attack

## Key Variables & Constants

### Game Configuration
- **Canvas dimensions**: 480×640 logical pixels
- **Game speed**: Dynamic based on canvas resolution
- **Frame rate**: Locked to screen refresh rate (typically 60 FPS)
- **Player health**: 3 hit points (visual hearts)
- **Smart bombs**: 3 per game

### Player Controls
- **Movement keys**: W/A/S/D or arrow keys
- **Mouse**: Aim weapon (desktop)
- **Touch**: Drag to move, aiming automatic
- **Space**: Fire weapon
- **B**: Drop smart bomb

### Power-ups
- **Score Multiplier**: Increases points earned (2x–8x)
- **Health Recovery**: Restores player health
- **Weapon Changes**: Spread, Laser, or Guided Missiles
- **Smart Bomb Refill**: Adds bomb charge

## Development Guidelines

### Adding New Features
1. **New Enemy Types**: Define in enemy spawn pattern, add to update/render logic
2. **New Weapons**: Add weapon type enum and projectile rendering
3. **New Stages**: Add wave definitions and boss encounters to stage data
4. **Visual Effects**: Create particle generator or animation loop

### Performance Considerations
- Canvas size dynamically adjusts; larger = more CPU intensive
- Limit particle count and despawn off-screen projectiles
- Use requestAnimationFrame for smooth 60 FPS target
- Cache DOM references for HUD updates

### Browser Compatibility
- Requires HTML5 Canvas support
- Uses Web Audio API (graceful fallback to silent if unavailable)
- Touch event handling for mobile/tablet
- CSS custom properties (:root) for theme colors

## Code Style Notes

- **Naming**: camelCase for variables/functions, UPPER_CASE for constants
- **Comments**: Sparse but present for non-obvious logic
- **Modularity**: Logic is organized into clear update/render sections per entity
- **Error handling**: Graceful fallback if audio API unavailable

## Testing & Validation

### Manual Testing Checklist
- [ ] Game starts and responds to player input (desktop & mobile)
- [ ] Enemies spawn and behave correctly across waves
- [ ] Weapon switching and firing mechanics work
- [ ] Boss encounters load and follow attack patterns
- [ ] Score multiplier and power-up collection functional
- [ ] Smart bombs clear enemy projectiles and damage enemies
- [ ] Game over/restart flow works
- [ ] Full-screen mode functional
- [ ] Responsive on various device sizes

## Future Enhancement Ideas

1. **Persistence**: Save high scores to localStorage
2. **Difficulty Levels**: Easy/Normal/Hard modes with adjusted spawning
3. **More Bosses**: Additional stage bosses with unique patterns
4. **Cosmetic Themes**: Alternative color schemes or visual styles
5. **Sound Effects**: Implement Web Audio API for SFX/music
6. **Leaderboard**: Online score submission (requires backend)
7. **Mobile App**: Wrap as PWA or native app

## Troubleshooting

### Game doesn't respond to input
- Ensure canvas has focus (click it first)
- Check browser console for JavaScript errors
- Verify input event listeners are attached

### Audio not working
- Web Audio API may be blocked or unavailable
- Check browser permissions for audio
- Fallback is graceful (game runs silent)

### Performance issues
- Reduce canvas size or browser zoom
- Check for excessive particle counts
- Monitor CPU usage in browser DevTools
- Close other CPU-intensive tabs

## Resources & References

- [MDN Canvas API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)
- [Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)
- [Touch Events Specification](https://www.w3.org/TR/touch-events/)
- [Raiden series](https://en.wikipedia.org/wiki/Raiden_(video_game)) - Design inspiration

---

**Last Updated**: 2026-10-03  
**Game Version**: 1.0

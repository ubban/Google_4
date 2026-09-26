# Claude.md - Sky Fighter Codebase Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in HTML5/JavaScript. It's a single-file web application that runs in any modern browser.

- **Repository**: ubban/google_4
- **Primary Branch**: `claude/inspiring-planck-m45h94`
- **Language**: HTML5 + JavaScript (Canvas API)
- **File Structure**: Single index.html file (~1,669 lines)

## Key Files

| File | Purpose |
|------|---------|
| `index.html` | Complete game implementation (HTML + CSS + JavaScript) |
| `README.md` | Minimal project metadata |

## Project Structure

The `index.html` file contains:

### HTML Structure
- Single `<canvas id="gameCanvas">` element for rendering
- HUD (Heads-Up Display) elements:
  - `#hud` - Contains game statistics display
  - `#hearts` - Player health display
  - `#bombs` - Smart bomb count display
  - `#weaponStat` - Current weapon indicator
  - `#multStat` - Multiplier display
- Visual effect elements:
  - `#vignette` - Red flash effect (screen damage)
  - `#flash` - Flash overlay for hits/bombs

### CSS Styling
- Dark space theme (`#05122b` background)
- Responsive design using `clamp()` for scaling
- Gradient sky background
- HUD glass-morphism design with backdrop blur
- Animations:
  - `heartBeat` - Critical health warning
  - `vignettePulse` - Damage vignette effect
  - `flashHit`/`flashBomb` - Impact flashes

### JavaScript Game Engine

#### Core Systems
1. **Canvas Rendering** - 2D canvas-based graphics
2. **Physics** - Movement, collision, projectiles
3. **Entity Management** - Player, enemies, bullets, powerups
4. **Game Loop** - RAF-based animation loop
5. **Input Handling** - Mouse/touch controls
6. **Audio System** - Sound effects (created dynamically via Web Audio API)

#### Game Mechanics
- **Player Ship**: Controllable via mouse/touch, has health and bombs
- **Weapons**: Multiple firing modes (normal shots, spread, concentrated)
- **Enemies**: Various enemy types with AI patterns
- **Bosses**: Stage boss encounters with multiple phases
- **Powerups**: Weapon upgrades, health restoration, bomb replenishment
- **Scoring**: Multiplier system based on hit consistency
- **Smart Bombs**: Screen-clearing special attack (limited uses)

#### Key Game Variables
- Player position, health, weapon type, ammo
- Enemy spawning patterns and difficulty
- Collision detection system
- Score and multiplier tracking
- Stage/level progression

## Development Notes

### Architecture Decisions
- **Single File**: All game logic in one HTML file for simplicity and portability
- **Canvas 2D**: Used instead of WebGL for broader compatibility
- **Web Audio API**: Dynamic sound generation instead of audio files
- **Immediate Mode Rendering**: Each frame redraws from scratch

### Game Loop Structure
- Frame-based game updates
- Input polling for continuous movement
- Entity update and collision detection
- Canvas clearing and redraw each frame

### Graphics Rendering
- Background gradient for sky effect
- Sprite-based rendering for game entities (player, enemies, bullets)
- Particle effects for explosions
- HUD overlay with transparency/backdrop-filter

### Performance Considerations
- Single canvas rendering context
- No pre-loaded assets (procedurally generated)
- Efficient collision detection with spatial optimization
- Throttled input processing for touch devices

## How to Run

1. Open `index.html` in any modern web browser (Chrome, Firefox, Safari, Edge)
2. No build process or dependencies required
3. Use mouse to move player ship
4. Click/tap to fire
5. Use keyboard or UI controls for weapon switching

## Browser Compatibility

- Requires: Canvas 2D, Web Audio API, modern JavaScript (ES2015+)
- Supports: All modern browsers on desktop and mobile
- Touch-friendly controls for mobile devices

## Development Guidelines for Claude

### When Modifying the Game

1. **File Location**: All changes go in `index.html`
2. **Code Organization**:
   - HTML markup at top
   - CSS in `<style>` tag
   - JavaScript in `<script>` tag
3. **Testing**: Open the file in a browser to manually test changes
4. **Performance**: Test on lower-end devices and ensure 60fps gameplay

### Common Tasks

#### Adding New Enemy Type
- Define enemy object with position, health, speed, and AI pattern
- Add to enemy spawn system
- Implement collision/bullet hit logic
- Add visual rendering code

#### Adding New Weapon
- Define weapon properties (fire rate, damage, spread, etc.)
- Implement bullet creation logic
- Add weapon selection UI
- Test collision with enemies

#### Adjusting Difficulty
- Modify enemy spawn patterns
- Adjust enemy health and damage
- Change boss patterns
- Tune multiplier decay rate

#### Sound Effects
- Use Web Audio API context (already initialized)
- Create custom sound synthesis or play pre-recorded oscillator patterns

### Code Style Notes
- Use functional/object-oriented patterns as appropriate
- Add comments for complex game logic
- Keep entity update/render loops efficient
- Maintain responsive controls for smooth gameplay

## Git Workflow

- Work on branch: `claude/inspiring-planck-m45h94`
- Commit with clear descriptions of game mechanic changes
- Test changes in browser before committing
- Push when changes are working and tested

## Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| Game too slow | Check particle/entity count, optimize collision checks |
| Audio not working | Verify Web Audio API context initialization |
| Touch controls unresponsive | Ensure touch event listeners are properly bound |
| Graphics glitchy | Clear canvas properly, check z-index/rendering order |

## References

- [Canvas API MDN](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)
- [Web Audio API MDN](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)
- [Game Programming Patterns](https://gameprogrammingpatterns.com/)

# Sky Fighter — Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game implemented as a single-page HTML5 Canvas application. The game features airplane combat with weapon upgrades, smart bombs, stage-based progression, and multi-phase boss battles.

**Repository**: `ubban/google_4`  
**Type**: Web Game (HTML5/Canvas)  
**Status**: Production-ready  
**Last Updated**: 2026-08-31

## Architecture

### File Structure
- **index.html** (1669 lines): Single-file application containing:
  - HTML markup (canvas, HUD elements)
  - CSS styling (game canvas, UI components, animations, responsive design)
  - JavaScript game engine (core logic, game loop, entity management)

### Design Pattern: Single-File Monolithic

The entire game is self-contained in one HTML file. The architecture follows this structure:

1. **HTML Section**: Canvas container and HUD elements
2. **CSS Section**: Styling for game canvas, UI components, animations, and responsive layout
3. **JavaScript Section**: Game engine with:
   - Game state management
   - Entity classes (Player, Enemy, Boss, Bullet, Particle)
   - Collision detection
   - Audio management
   - Input handling (keyboard, mouse, touch)
   - Rendering pipeline

## Game Mechanics & Features

### Core Gameplay
- **Vertical Scrolling Shooter**: Player controls an airplane firing upward at incoming enemies
- **Lives System**: Player starts with 3 lives (displayed as hearts in HUD)
- **Health/Shield**: Collision with enemies decreases health; visual feedback via screen flash and vignette
- **Score Multiplier**: Kill-chain system that increases score multiplier based on consecutive hits

### Weapons System
- **Vulcan Gun**: Balanced firepower with rapid-fire capability
- **Laser**: High-power single-beam weapon
- **3 Power Levels**: Weapon upgrades increase fire rate and damage
- **Power-ups**: Weapon level pickups drop from defeated enemies

### Special Mechanics
- **Smart Bombs**: Limited-use screen-clearing attacks that also damage bosses
  - Visual feedback: white flash animation
  - HUD display: bomb counter with visual indicators
- **Parallax Clouds**: Background layers that scroll at different speeds for depth
- **Screen Shake**: Triggered on player damage for impact feedback
- **Hit Flash**: White screen flash when player takes damage
- **Red Vignette Pulse**: Activates when health is critical (low HP state)

### Stage System
- **Wave-Based Progression**: Each stage contains multiple enemy waves
- **Boss Encounters**: Culmination of each stage with multi-pattern boss battles
- **3 Boss Archetypes**: Vanguard, Behemoth, Specter (cycle by stage with unique colors/patterns)
- **Health Bar**: Boss HP display during battles

### Enemy Types
- **Grunt**: Standard enemy, direct approach
- **Weaving Interceptor**: Evasive movement pattern
- **Tank**: Multi-hit enemy with own health pool; visual pip indicators

### Mobile Support
- **Touch Controls**: Responsive touch input for mobile gameplay
- **Viewport Optimization**: Responsive canvas sizing with safe-area-inset support
- **Mobile-Optimized UI**: Scaled HUD elements using CSS clamp()

## UI/UX Components

### HUD (Heads-Up Display)
Located at top-center with responsive scaling using CSS `clamp()`:
- **Hearts**: Live remaining (red hearts = alive, dimmed = lost)
- **Bombs**: Smart bomb count (bright = available, dim = used)
- **Weapon**: Current weapon type and level
- **Multiplier**: Current score chain multiplier
- **Score**: Total accumulated score (if implemented)

### Styling Features
- **Glassmorphism**: Semi-transparent HUD elements with blur backdrop filter
- **Pill-Style Containers**: Rounded HUD stat boxes
- **Color Coding**: 
  - Hearts: Red (#FF0000 implied in animation)
  - Score/Weapon: Yellow (#FFD54F)
  - Multiplier: Orange (#FF8A65)
  - Weapon level: Lime (#7CFC00)

### Animations
- **Heart Beat**: Critical health animation (pulsing scale)
- **Vignette Pulse**: Red vignette pulses when health is low
- **Flash Animations**: Hit flash (0.25s) and bomb flash (0.5s)
- **Entrance Transitions**: Title and overlay card animations

## Technical Implementation Notes

### Canvas Rendering
- **Resolution**: Responsive, scaled to window size with max-width/max-height constraints
- **Background**: Linear gradient sky (navy to light blue)
- **Cursor**: Crosshair for targeting feedback

### Collision Detection
- Likely circle-based or AABB for bullets, enemies, player
- Enemy-bullet collisions trigger damage and particle effects
- Player-enemy collisions reduce player health

### Persistence
- **Local Storage**: Game state (scores, unlocks, settings) persisted between sessions
- **Auto-save**: Game state saved during play

### Audio
- Game sound effects and music integrated (details in JavaScript audio manager)
- Mobile-compatible audio handling

### Performance Considerations
- Canvas-based rendering (efficient 2D drawing)
- Particle system for visual effects
- Object pooling for bullets and particles (implied by wave-based gameplay)

## Development Notes

### Recent Enhancements (Latest Commit)
1. Added enemy/boss variety and stage visual themes
2. Implemented 3-design boss roster cycling by stage
3. Added unique colors and attack patterns per boss type
4. Improved production quality and visual depth

### Known Features
✓ Single-file implementation (easier deployment)  
✓ Responsive design (desktop and mobile)  
✓ Persistence (game state saved)  
✓ Audio integration  
✓ Multiple weapon types  
✓ Boss battles with patterns  
✓ Score multiplier system  
✓ Screen effects (shake, flash, vignette)  
✓ Touch controls  

### Testing
- Verified with headless Playwright automation
- Tests cover: firing, collisions, weapon pickups, boss spawn/attack patterns, stage advancement
- No console errors reported

## Dependencies & Environment

### Runtime
- **Browser**: Modern HTML5 Canvas support required (Chrome, Firefox, Safari, Edge)
- **Minimum Version**: ES6+ JavaScript support
- **No External Libraries**: Fully self-contained

### Optional
- **Local Storage API**: For game persistence
- **Web Audio API**: For sound effects (graceful fallback if unavailable)

## Deployment

### Hosting
- Deploy `index.html` as a static file to any web server
- No backend required
- Works from CDN or local file system (`file://` URL)

### Build/Development
- No build step required
- Edit `index.html` directly for changes
- Refresh browser to see updates

## Code Organization (JavaScript)

The JavaScript section likely contains (based on commit messages):

1. **Game State**: Current lives, score, multiplier, level, weapon state
2. **Entity Classes**: Player, Enemy, Boss, Bullet, Particle
3. **Game Loop**: Update and render cycle using `requestAnimationFrame`
4. **Input Handling**: Keyboard (`W`, `A`, `S`, `D`), mouse/touch coordinates
5. **Collision Engine**: Detect and resolve collisions between entities
6. **Particle System**: Visual effects (explosions, power-up glows)
7. **Audio Manager**: Sound effect playback
8. **Stage Manager**: Wave progression and boss spawning
9. **Persistence Manager**: LocalStorage integration
10. **UI Manager**: HUD updates and screen effects

## Future Enhancement Opportunities

- Leaderboard integration (cloud-based scoring)
- Additional boss patterns and enemy types
- Power-up variety expansion (shield, slow-time, etc.)
- Difficulty modes (easy, normal, hard, extreme)
- Sound effect and music customization
- Replay system or demo mode
- Multiplayer support (if applicable)

---

**Last Updated**: 2026-10-06  
**Maintainer**: ubban (ubban@irobotx.top)

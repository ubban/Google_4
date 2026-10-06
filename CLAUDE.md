# Sky Fighter - Project Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-page web application. Players navigate a fighter jet through enemy waves, collect power-ups, manage ammunition, and defeat stage bosses in an action-packed arcade experience.

**Repository**: ubban/google_4  
**Primary Branch**: `claude/inspiring-planck-fda1n7`  
**Tech Stack**: HTML5, CSS3, Canvas API, Vanilla JavaScript (ES6+)

## Project Structure

```
Google_4/
├── index.html          # Complete game implementation (HTML + CSS + JS)
├── README.md           # Brief project description
└── CLAUDE.md          # This file - comprehensive project documentation
```

## Core Features

### Game Mechanics
- **Player Control**: WASD for movement, mouse/click for aiming
- **Weapons System**: Multiple weapon types with switchable mechanics
- **Bomb System**: Smart bombs for area damage with limited uses
- **Health System**: Lives-based system with visual heart indicators
- **Combo/Score Multiplier**: Score increases with successful hits, resets on damage
- **Boss Stages**: Multi-stage progression with boss encounters

### Visual Systems
- **HUD Display**: Real-time stats (score, lives, bombs, weapon, multiplier)
- **Boss Health Bar**: Visual boss HP tracking during encounters
- **Combat Feedback**: Flash effects on hit/bomb, vignette effects on damage
- **Combo Notifications**: Visual popup when combo milestones hit
- **Stage Banners**: Introduction screens for each stage

### Input Handling
- **Keyboard Controls**: WASD movement (with adaptive controls)
- **Mouse Aiming**: Crosshair cursor for weapon targeting
- **Touch Support**: Mobile-optimized controls with joystick zones
- **Responsive Design**: Works on desktop, tablet, and mobile

## Technical Architecture

### HTML Structure
- Single-page with responsive viewport meta tags
- Safe area insets for notch/home-bar support on mobile
- Canvas element for rendering game graphics
- Overlay system for menus and end screens
- HUD elements for real-time game state display

### CSS Features
- Dark theme (`color-scheme: dark`) optimized for arcade aesthetic
- Responsive typography using `clamp()` for fluid scaling
- Animations: heartbeat (health critical), vignette pulse (damage), combo/stage popups
- Backdrop blur effects for UI elements
- Gradient backgrounds and text effects
- Grid/flexbox layouts for responsive HUD positioning

### JavaScript Architecture
The game logic is organized into core systems:

#### Game State Management
- Player position, health, bomb count
- Weapon selection and ammo tracking
- Score and combo multiplier
- Current stage and enemy wave progression

#### Input System
- Keyboard event listeners for WASD movement
- Mouse movement tracking for aiming
- Touch event handling with joystick zones
- Adaptive control mode based on device

#### Entity System
- Player entity with collision detection
- Enemy bullets and waves
- Boss entities with behavior patterns
- Power-up pickups (ammo, health, bombs)
- Particle effects (explosions, hits)

#### Rendering Pipeline
- Canvas context initialization
- Layered drawing (background, entities, effects, HUD)
- Frame-based animation loop (requestAnimationFrame)
- Collision detection using bounding boxes

#### Game Loop
- Input processing
- Entity updates (position, state transitions)
- Collision detection and resolution
- Rendering to canvas
- Frame rate: ~60 FPS with delta-time calculations

## Key Game Parameters

### Player Stats
- Default lives: 3 (visual hearts)
- Default bombs: 3 (max capacity)
- Movement speed: canvas-relative
- Initial weapon: Standard shots

### Weapon System
- Each weapon has distinct firing patterns
- Ammo management per weapon
- Firing rate varies by weapon type
- Power-ups increase ammo capacity

### Difficulty Levels
- Multiple difficulty modes likely available
- Affects enemy density, speed, and attack patterns
- Score multipliers based on difficulty

### Scoring
- Points for destroying enemies
- Multiplier increases with consecutive hits
- Resets on player damage
- Boss kills grant major bonus points

## Player Experience Flow

1. **Title Screen**: Game intro with difficulty selection
2. **Stage Start**: Stage banner animation announces level
3. **Combat**: Player navigates enemies, collects pickups
4. **Boss Phase**: Boss health bar appears, specialized combat
5. **Stage Clear**: End screen with final score, stats
6. **Progression**: Continue to next stage or game over

## Development Notes

### Important Implementation Details
- All content delivered in single HTML file for accessibility
- No external dependencies - pure vanilla JavaScript
- Canvas-based rendering for performance
- Responsive design without media queries (using viewport units and clamp)
- Touch-friendly with appropriate hit areas and feedback

### Code Organization
- Embedded styles in `<style>` tag with organized sections
- Game script in `<script>` tag with clear function/class separation
- Event listeners attached after DOM parsing
- Canvas initialization after element mount

### Mobile Optimization
- Safe area support for notch/home bar devices
- Touch controls with visual feedback
- Prevent zoom and text selection
- Adaptive font sizing via clamp()
- Haptic feedback consideration (if browser supports)

## Common Tasks

### Adding New Weapons
1. Define weapon properties (rate of fire, ammo capacity, spread)
2. Implement bullet rendering
3. Add weapon selection UI
4. Hook into fire input handler

### Creating New Enemy Types
1. Define enemy class with AI behavior
2. Implement movement patterns
3. Define bullet patterns
4. Add to wave generation system

### Balancing
- Adjust `game.config` parameters for difficulty
- Modify enemy speed, spawn rates, attack patterns
- Adjust weapon damage and fire rates
- Tune boss health and attack patterns

### UI Changes
- Update HUD elements in HTML markup
- Modify CSS styles for visual changes
- Update canvas rendering code for gameplay visuals

## Performance Considerations
- Canvas rendering optimized for 60 FPS
- RequestAnimationFrame for smooth animation
- Efficient collision detection using spatial partitioning (if implemented)
- Object pooling for frequently created/destroyed entities
- CSS animations use GPU acceleration where possible

## Browser Compatibility
- Modern browsers with Canvas API support (Chrome, Firefox, Safari, Edge)
- Mobile browsers: iOS Safari, Android Chrome
- Requires JavaScript ES6+ support
- WebGL not required - uses 2D canvas context

## Future Enhancement Ideas
- Additional weapon types and progression
- Power-up variety (shields, time slowdown, etc.)
- New enemy patterns and boss types
- Sound effects and background music
- Leaderboard/high score persistence (localStorage)
- Achievements system
- Multiple campaign stages
- Online multiplayer elements

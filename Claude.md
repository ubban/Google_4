# Sky Fighter - Codebase Guide

## Project Overview
Sky Fighter is a Raiden-style vertical shoot-'em-up game built in vanilla JavaScript, HTML5 Canvas, and CSS. Players control an aircraft to dodge enemies, switch weapons, drop smart bombs, and defeat stage bosses.

## Architecture

### Single-File Design
- **index.html**: Complete game implementation (HTML, CSS, and JavaScript)
  - All game logic, rendering, and styling in one file
  - ~60KB file size
  - Minimal dependencies (pure vanilla JavaScript)

### Game Systems

#### Core Game Loop
- Canvas-based 2D rendering at 60 FPS
- Input handling (mouse/touch cursor movement)
- Entity update and collision detection
- Rendering of game world and HUD

#### Player Mechanics
- **Movement**: Follows cursor position (mouse on desktop, touch on mobile)
- **Health System**: 3 hearts by default; visual feedback with heartbeat animation
- **Weapons**: Switchable primary weapons with different fire patterns
- **Smart Bombs**: Limited-use destructive capability; displayed as bomb indicators
- **Weapon Multiplier**: Tracks accumulated attack power

#### Enemy & Obstacle Systems
- **Stage-Based Progression**: Multiple difficulty stages
- **Enemy Types**: Varied AI behaviors and attack patterns
- **Boss Battles**: Stage-ending boss encounters with distinct mechanics
- **Visual Feedback**: Hit flashes, vignette damage indicator (red overlay during damage)

#### UI/HUD Elements
- **Top-Center Display**: Shows health (hearts), bombs, weapon type, and multiplier
- **Responsive Layout**: Scales for mobile and desktop via `clamp()` CSS units
- **Safe Area Support**: Respects mobile notches with `env(safe-area-inset-top)`
- **Animations**: Heart beat effect when critical, bomb glow states

### Visual Theme
- **Dark Blue Sky Gradient**: Background shifts from dark (#0d2f5e) to bright blue (#7fc1ff)
- **Color Scheme**: 
  - Weapon indicator: Lime green (#7CFC00)
  - Multiplier indicator: Deep orange (#ff8a65)
  - Score text: Gold (#ffd54f)
- **Effects**: 
  - Red vignette overlay (damage indicator)
  - Flash effect (impacts/explosions)
  - Drop shadow text (HUD readability)

### Mobile Optimization
- Touch-action controls (cursor-following gameplay)
- Viewport meta tags for mobile display
- CSS `clamp()` for responsive typography
- Crosshair cursor for targeting feedback
- No horizontal scrolling; fixed fullscreen layout

## Key Features
1. **Dynamic Difficulty**: Multiplier system rewards consecutive hits
2. **Smart Bombs**: Powerful area-clearing ability with limited uses
3. **Boss Encounters**: Stage progression with increasingly difficult bosses
4. **Visual Polish**: Smooth animations, responsive feedback, dark theme
5. **Cross-Platform**: Works on desktop (mouse) and mobile (touch)

## Development Notes

### File Organization
This is a single-file game—all CSS is in `<style>` tags, all JavaScript in `<script>` tags.

### Testing the Game
1. Open `index.html` in a web browser
2. Use mouse cursor on desktop (move = aim, click = fire)
3. Use touch on mobile (move = aim, tap screen = fire)
4. Game runs fullscreen on load

### Important Implementation Details
- **Canvas Sizing**: Dynamically sized to fit viewport; uses `max-width`/`max-height` to maintain aspect ratio
- **Touch Support**: `touch-action: none` prevents browser defaults; all touch events handled by game
- **Performance**: Single-threaded game loop; canvas rendering optimized for 60 FPS
- **Responsive Units**: All text/spacing uses `clamp()` for fluid scaling across devices

### No External Dependencies
- Vanilla JavaScript (ES6+)
- HTML5 Canvas API
- CSS3 (including backdrop-filter for HUD blur effect)
- No build process required—runs directly in browser

## How to Modify

### Game Balance
Look for numeric constants in the JavaScript section for:
- Player health/damage values
- Enemy spawn rates and AI
- Boss attack patterns
- Weapon properties (fire rate, damage, spread)

### Visual Changes
Update CSS variables or style blocks for:
- Color scheme (search for `#0b1d3a`, `#05122b`, gradient colors)
- HUD layout and positioning
- Animation speeds (e.g., `heartBeat 0.8s`)
- Font sizing and responsive breakpoints

### Gameplay Mechanics
Modify the main game loop, entity update functions, and collision detection logic in the `<script>` tag. The structure follows classic game development patterns: init → update → render → repeat.

## Performance Considerations
- Canvas rendering is GPU-accelerated on modern browsers
- Touch events debounced to prevent input lag
- DOM updates minimal (only HUD stat values change during gameplay)
- No memory leaks; entities cleaned up after destruction

## Browser Compatibility
- Modern browsers (Chrome, Firefox, Safari, Edge)
- Mobile browsers (iOS Safari, Chrome Android)
- Requires HTML5 Canvas and ES6 JavaScript support
- Backdrop filter may degrade gracefully on older browsers

---

**Last Updated**: 2026-09-28  
**Project Type**: HTML5 Canvas Game  
**Maintenance**: Minimal—self-contained single file

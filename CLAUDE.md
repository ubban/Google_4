# Sky Fighter — Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-page HTML5 application. Players control a fighter jet, switch between multiple weapon types, use smart bombs, and face stage bosses across escalating difficulty levels.

**Repository:** `ubban/google_4`  
**Primary File:** `index.html` (comprehensive game implementation)  
**Live Demo:** GitHub Pages deployment  

## Game Features

- **Core Gameplay:** Vertical scrolling shooter with progressive difficulty
- **Player Mechanics:**
  - Arrow key / WASD movement (desktop)
  - Touch / mouse controls (mobile)
  - Space / Click to fire
  - Q key for smart bombs
  
- **Weapon System:**
  - Vulcan (rapid-fire bullets)
  - Missiles (explosive projectiles)
  - Laser (beam weapon)
  - Weapon switching (E key)
  - Ammo display in HUD

- **Boss Encounters:** Multi-stage boss fights with unique attack patterns

- **Progression:** Health system, bomb supply, score multiplier

- **Visual Polish:**
  - Gradient skybox
  - Vignette effects on damage
  - Flash animations on hits/explosions
  - Responsive UI (mobile-optimized)

## Architecture

### Single-File Structure
The entire game is implemented in `index.html` (~1500+ lines):

1. **HTML Structure:**
   - `<canvas>` element for game rendering
   - HUD display (stats, hearts, bombs, weapon indicator)
   - Vignette and flash effect overlays

2. **CSS:**
   - Dark space theme (`#05122b` background)
   - Responsive HUD with safe-area-inset support
   - Animations (heartbeat, vignette pulse, flash effects)
   - Mobile-first design

3. **JavaScript Game Loop:**
   - Canvas 2D rendering
   - Input handling (keyboard, mouse, touch)
   - Entity update cycle (player, enemies, projectiles, particles)
   - Collision detection
   - Score and state management

### Key Game Systems

- **Player:** Positioned center-bottom, health/ammo tracking, weapon selection
- **Enemies:** Flying formations with behavior patterns
- **Bosses:** Multi-phase encounters with attack telegraphs
- **Projectiles:** Player bullets, enemy fire, explosions
- **Particles:** Visual effects (sparks, explosions)
- **Collision:** AABB (Axis-Aligned Bounding Box) detection
- **HUD:** Real-time stats display with visual feedback

## Development Guidelines

### How to Test
1. Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge)
2. Desktop: Use arrow keys or WASD to move, Space to fire, E to switch weapons, Q for bombs
3. Mobile: Tap and drag to move, tap to fire
4. Game auto-starts on load

### Making Changes
- All game logic is in the `<script>` section at the end of `index.html`
- Classes/objects: `Player`, `Enemy`, `Boss`, `Projectile`, `Particle`
- Main loop: `gameLoop()` function called via `requestAnimationFrame`
- HUD updates: `updateHUD()` function
- Collision checks: Scattered throughout entity update methods

### Performance Considerations
- Canvas scaling optimized for mobile (max-width/max-height constraints)
- Particle pooling to reduce garbage collection
- Enemy spawning throttled by wave system
- Entity cleanup when off-screen

### Mobile Optimization
- Viewport meta tag with `user-scalable=no` for full-screen gameplay
- Touch action disabled to prevent scrolling
- Safe-area insets for notched devices
- Responsive font sizing (clamp values)
- Crosshair cursor on canvas

## Common Tasks

### Add a New Weapon Type
1. Define weapon properties (damage, rate of fire, spread pattern)
2. Add case to weapon switch logic
3. Create projectile spawning logic in player update
4. Update HUD weapon display

### Adjust Difficulty
1. Modify enemy wave spawning rates (look for `waveTimer` and spawn conditions)
2. Adjust enemy health and boss patterns
3. Tweak enemy fire frequency and damage

### Fix Mobile Controls
1. Check touch event listeners (look for `touchmove`, `touchstart`)
2. Verify canvas coordinate translation
3. Test on actual mobile devices (not just responsive mode)

### Deploy Changes
1. Commit and push to `claude/inspiring-planck-da3yfv` branch
2. Changes auto-deploy to GitHub Pages from main branch

## Technical Stack

- **Languages:** HTML5, CSS3, JavaScript (ES6+)
- **APIs:** Canvas 2D Context, Web Audio (if used)
- **Build:** No build step required (single-file delivery)
- **Target Browsers:** Modern browsers (ES6+ support required)
- **Mobile:** Fully responsive, touch-enabled

## Notes for Claude Code Agent

- **No Dependencies:** Pure JavaScript, no frameworks or libraries
- **Self-Contained:** All code in one file for easy modification
- **Performance-First:** Direct Canvas rendering, minimal allocations
- **Platform Support:** Desktop and mobile through unified control system
- **Git Workflow:** Work on `claude/inspiring-planck-da3yfv`, commit with attribution

---
Last updated: 2026-10-04  
Agent: Claude Code with Haiku 4.5

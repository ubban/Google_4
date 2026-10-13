# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla JavaScript and HTML5 Canvas. It's a single-file game (index.html) featuring:

- **Player Control**: Mouse/touch aiming, keyboard/touch movement
- **Weapons System**: Multiple weapon types with switching mechanics
- **Power-ups**: Score multiplier pickups
- **Boss Encounters**: Stage-based progression with boss battles
- **Score Tracking**: Persistent high-score storage via localStorage
- **Responsive Design**: Desktop, tablet, and mobile support

## File Structure

```
Google_4/
├── index.html          # Complete game (1669 lines: HTML, CSS, JavaScript)
├── README.md           # Project metadata
└── .git/               # Git repository
```

The entire game is contained in a single `index.html` file with:
- **Inline CSS** (lines ~10-400): Styling for canvas, HUD, overlays, animations
- **Inline JavaScript** (lines ~450+): Game engine and mechanics

## Game Architecture

### Core Structure (Conceptual Organization)

The JavaScript portion is organized as follows:

1. **Constants & Configuration** (~top of script)
   - Canvas dimensions, physics parameters
   - Color palette, animation timings
   - Difficulty levels and game balance

2. **Game State Management**
   - Current game state (running, paused, gameOver)
   - Player data (position, health, weapons, score)
   - Entity arrays (enemies, bullets, explosions, pickups)
   - Wave/stage progression

3. **Game Loop**
   - `requestAnimationFrame` based update/render cycle
   - Input handling (mouse, keyboard, touch)
   - Collision detection
   - Entity lifecycle (spawn, update, despawn)

4. **Entity Systems**
   - **Player**: Movement, weapon fire, health management
   - **Enemies**: Wave spawning, AI patterns, collision
   - **Projectiles**: Player bullets, enemy bullets
   - **Explosions/Particles**: Visual feedback
   - **Pickups**: Power-ups and score multipliers
   - **Bosses**: Special enemy with larger health pool and attack patterns

5. **UI & HUD**
   - Overlay screens (start, game over, pause)
   - Real-time HUD (health, bombs, weapon, multiplier)
   - Score display and high score tracking
   - Stage/boss announcements
   - Visual effects (flash, vignette, combo popups)

6. **Input Handling**
   - Mouse movement tracking (aiming)
   - Keyboard input (WASD/arrows for movement, space for bomb)
   - Touch controls with visual zones (movement and fire)
   - Touch aim reticle

7. **Persistence**
   - localStorage for high score
   - Game state serialization for pause/resume

### Key Game Systems

#### Weapon System
- Multiple weapon types (likely bullets, spread shots, homing)
- Player can switch weapons (keyboard-triggered)
- Weapon stat displayed in HUD (likely fire rate, damage, or type)

#### Difficulty System
- Difficulty selector (Easy, Normal, Hard)
- Affects enemy patterns, health, spawn rates
- Likely scales damage dealt and taken

#### Boss System
- Boss health bar at top of screen (#bossBarWrap)
- Boss name display
- Special visual effects (enemy bullets, larger attacks)
- Triggers when stage progression reaches boss

#### Score Multiplier
- Picked up from enemies
- Displayed in HUD (#multStat)
- Increases score per hit during active window

## Code Style & Conventions

### Naming
- **camelCase** for variables, functions, methods: `playerX`, `fireWeapon()`
- **UPPER_SNAKE_CASE** for constants: `CANVAS_WIDTH`, `DIFFICULTY_EASY`
- **PascalCase** for conceptual objects/states if used as class-like objects

### HTML/CSS
- CSS uses BEM-inspired naming: `.stat`, `.label`, `.val`
- Element IDs use camelCase with semantic names: `#gameCanvas`, `#hud`, `#overlay`
- Responsive units: `clamp()`, `vw`/`vh`, `env(safe-area-inset-*)`

### JavaScript Practices
- Inline script tag at end of HTML body
- Functional approach with closures for state management
- requestAnimationFrame for render loop (no setInterval for critical game logic)
- Direct DOM manipulation with `getElementById()` and `querySelector()`
- Canvas 2D context used for all rendering

### Constants Location
Look for configuration constants at the top of the script section:
- Canvas dimensions (likely 600x700 or similar)
- Physics values (player speed, bullet speed, friction)
- Colors and animation durations
- Wave/enemy configurations

## Common Development Tasks

### Adding a New Enemy Type
1. Define enemy template/configuration (health, speed, sprite/drawing code)
2. Add spawning logic to wave definition
3. Add drawing code to the render loop
4. Ensure collision detection works (projectiles can hit it)
5. Add despawn condition (when off-screen or health <= 0)

### Tweaking Game Balance
- Look for constants controlling: enemy health, player health, bullet damage, enemy fire rate, movement speeds
- Physics feel changes: acceleration, friction, max speeds
- Adjust difficulty multipliers in difficulty level definitions

### Adding Visual Effects
1. Canvas drawing code patterns: `ctx.fillStyle`, `ctx.fillRect()`, `ctx.arc()`, `ctx.drawImage()`
2. Animation approach: track elapsed time, interpolate properties
3. Particle cleanup: remove from array when complete (check lifetime)

### Modifying UI Overlays
- Overlay content in `#overlayCard`
- Text elements in `#overlay`, `#overlay h1`, `#overlay p`
- Buttons like `#startBtn` with click listeners
- Update state and trigger transitions to show/hide

### Touch Input
- Touch controls in `#touchControls` section
- Movement zone (#moveZone) and fire zone (#fireZone) with positioning
- Track `touches` array for multi-touch if needed
- Calculate offset from zone center for input values

## Important Game Values

### HUD Elements (Track These)
- `#hearts`: Player health display (visual count)
- `#bombs`: Smart bomb count and availability
- `#weaponStat`: Current weapon info
- `#multStat`: Score multiplier value
- `#bossBarWrap`: Boss health bar visibility and fill level

### Canvas Layers (Z-order concept in code)
1. Background gradient
2. Enemies, explosions
3. Projectiles (player and enemy)
4. Pickups
5. Vignette/visual effects overlay (#vignette)
6. Flash effect (#flash)
7. HUD text and UI (#hud, #topBar)
8. Overlay screens (#overlay at z-index 10)

## Development Workflow

### Running the Game
1. Open `index.html` in a web browser
2. Game starts in overlay mode (select difficulty, click Start)
3. Control with mouse (aim) + WASD/arrows (move) + Space (bomb)
4. Or use touch controls on mobile

### Testing Changes
- Changes to CSS: Hard-refresh browser (Ctrl+Shift+R)
- Changes to JavaScript: Reload page, clear localStorage if testing score persistence
- Test on multiple devices if touching responsive design (use browser dev tools device emulation)

### Debugging
- Use browser DevTools console for errors
- `console.log()` can track game state during loop
- Canvas drawing issues: use inspector to check element dimensions
- Touch issues: emulate touch events in Chrome DevTools

### Common Pitfalls
- **Game loop timing**: Ensure all updates happen once per frame (no double-updates)
- **Collision detection**: Off-by-one errors in bounding boxes
- **Asset loading**: All assets are inline SVG/CSS, no external images
- **State cleanup**: Enemy/bullet arrays can grow unbounded if despawn logic fails
- **Touch event handling**: Remember `preventDefault()` to avoid default scrolling

## Repository Standards

- **Branch**: Work on `claude/inspiring-planck-0rzlbp`
- **Commits**: Describe what changed and why
- **Single file**: Keep game in `index.html` (all code, styles, markup together)
- **No external dependencies**: Vanilla HTML5, no libraries
- **Responsive design**: Maintain clamp() usage and safe-area support

## References

### Canvas API patterns used:
- `ctx.fillRect()`, `ctx.strokeRect()` - rectangles
- `ctx.arc()` - circles
- `ctx.fillStyle`, `ctx.strokeStyle` - colors
- `ctx.fillText()`, `ctx.font` - text rendering
- `ctx.beginPath()`, `ctx.stroke()`, `ctx.fill()` - paths

### DOM APIs:
- `element.addEventListener('mousemove', ...)` - input listeners
- `element.classList.add/remove/toggle()` - dynamic styling
- `localStorage.setItem/getItem()` - persistence
- `requestAnimationFrame(callback)` - game loop

### Browser APIs:
- `Touch` events: `touchstart`, `touchmove`, `touchend`
- `PointerEvents` may be used (check code for `pointerdown`, etc.)
- `Math.random()`, `Math.sin()`, `Math.cos()` - basic math

---

**Last Updated**: 2026-10-13  
**Created by**: Claude Code Agent  
Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>

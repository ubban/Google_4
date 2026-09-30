# Sky Fighter Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5/Canvas application. Players control an aircraft that scrolls vertically through stages, collect power-ups, switch weapons, use smart bombs, and defeat stage bosses.

### Key Characteristics
- **Single-file architecture**: All code (HTML, CSS, JavaScript) is contained in `index.html`
- **Canvas-based rendering**: Uses 2D Canvas API for all graphics
- **Responsive design**: Mobile and desktop support with touch controls
- **Difficulty levels**: Multiple selectable difficulty modes
- **Boss battles**: Each stage culminates in a boss encounter
- **Combo system**: Rewards continuous enemy elimination with combo multipliers

---

## Repository Structure

```
Google_4/
├── index.html          # Single monolithic file containing HTML, CSS, and JavaScript
├── README.md           # Basic project metadata
└── Claude.md           # This file - codebase documentation
```

---

## Architecture & Design Patterns

### HTML Structure
- **Container**: `#container` - Main viewport wrapper
- **Canvas**: `#gameCanvas` - Primary rendering surface (800x600 default)
- **UI Layers**:
  - `#hud` - Head-up display with stats (health, score, bombs, weapon)
  - `#bossBarWrap` - Boss health indicator (shown during boss fights)
  - `#overlay` - Menu screens (title, game over, boss intro)
  - `#vignette` - Screen shake/damage indicator
  - `#flash` - Screen flash for hits or bomb effects
  - `#comboPop` - Floating combo counter animation
  - `#stageBanner` - Stage introduction banner
  - `#touchControls` - Virtual joystick and buttons for mobile

### Game Loop Architecture

The JavaScript contains:
1. **Game state management** - Current game state (menu, playing, game-over, etc.)
2. **Entity systems**:
   - Player ship (aircraft)
   - Enemies (spawned waves)
   - Projectiles (bullets from player)
   - Enemy projectiles
   - Power-ups (health, weapons, bombs)
   - Explosions and effects
3. **Collision detection** - AABB (Axis-Aligned Bounding Box) collision system
4. **Rendering pipeline** - Canvas draw order management
5. **Input handling** - Keyboard and touch input processing
6. **Audio system** - Sound effects (muted by default, toggleable)

### Key Game Mechanics

- **Player Movement**: WASD or arrow keys; touch drag on mobile
- **Weapons**: 
  - Primary weapon (rapid-fire bullets)
  - Secondary weapon (collected power-ups: spreadshot, laser, etc.)
  - Weapon switching (W/E keys or touch buttons)
- **Smart Bombs**: Limited quantity, clears screen of enemies + projectiles
- **Health System**: Lives-based (starts with 3)
- **Scoring**: Points from enemy elimination; multiplied by combo counter
- **Difficulty**: Affects enemy spawn rate, health, and bullet patterns

### State Management

Game states:
- `START` - Title/menu screen
- `STAGE_INTRO` - Stage banner display
- `PLAYING` - Active gameplay
- `BOSS_FIGHT` - Boss encounter (special rules)
- `GAME_OVER` - Game ended
- `PAUSED` - Game paused (ESC key)

---

## Technical Implementation Details

### Canvas Rendering
- Base canvas size: 800×600 (scaled to fit viewport)
- Rendering order (back to front):
  1. Background gradient
  2. Enemy projectiles
  3. Player projectiles
  4. Explosions/effects
  5. Enemies
  6. Player ship
  7. UI/HUD overlays
  8. Vignette (damage indicator)

### Collision System
- Uses bounding boxes defined by `{ x, y, w, h }`
- Checks: player-bullet vs enemy, enemy-bullet vs player, pickup collection
- Early exit optimization when entities are far apart

### Mobile Support
- Touch event listeners on `#gameCanvas`
- Virtual joystick zone for movement (left 40% of screen)
- Button controls for weapons and bombs (right side)
- Respects safe area insets (notch support)

### Responsive Design
- CSS `clamp()` for fluid typography and spacing
- `max-width`/`max-height` with `aspect-ratio` for canvas
- Uses viewport units (`vw`, `vh`) for adaptive layouts
- Environment variables (`env(safe-area-inset-*)`) for notch support

### Performance Considerations
- Single draw call per frame (batch all canvas operations)
- Object pooling pattern for bullets/effects (reuse instead of create/destroy)
- Early collision checks (spatial optimization)
- Conditional rendering (only draw visible entities)

---

## Important Code Patterns & Conventions

### Entity Structure
Entities (player, enemies, projectiles) follow this pattern:
```javascript
{
  x, y,           // Position
  w, h,           // Width, height (collision bounds)
  vx, vy,         // Velocity
  hp,             // Health points
  // ... type-specific properties
}
```

### Update Cycle
Each frame:
1. Update entity positions (apply velocity)
2. Remove off-screen entities
3. Handle input
4. Collision detection
5. Spawn new enemies/effects
6. Render all entities
7. Update HUD

### Enemy Wave System
- Enemies spawn from a predefined sequence per stage
- Spawn patterns vary by difficulty
- Boss appears at end of stage sequence
- Enemy removal triggers potential power-up drops

### Power-Up System
- Collected on contact with player
- Types: health, weapon upgrades, bombs
- Visual indication on HUD
- Weapon switching with W/E keys

---

## Development Guidelines

### When Modifying Code

1. **Keep the single-file architecture** - Do not split into separate files
2. **Maintain the HTML structure** - All UI elements must remain in DOM
3. **Preserve game balance** - Changes to damage, health, or spawn rates affect difficulty
4. **Test on both keyboard and touch** - Mobile is fully supported
5. **CSS ordering matters** - z-index layering is critical for visual correctness
6. **Canvas coordinate system** - Origin (0,0) is top-left; Y increases downward

### Game Balance Considerations

- **Player speed**: Too fast breaks difficulty; too slow frustrates
- **Enemy damage**: Should force players to use dodge mechanics
- **Bomb duration**: Must clear screen effectively without trivializing bosses
- **Score multipliers**: Combo system incentivizes aggressive play
- **Boss health**: Difficulty-scaling should be proportional to stage progression

### CSS Customization Points

- **Color scheme**: Primary theme color `#0b1d3a`, accent `#1976d2`
- **Gradient backgrounds**: Used for canvas, buttons, text effects
- **Animation timing**: Combo pulse, stage banner, card animations
- **Safe area insets**: Built-in for notch/cutout support

---

## Common Tasks for Claude

### Adding a New Enemy Type
1. Define enemy object with `x`, `y`, `vx`, `vy`, `w`, `h`, `hp`
2. Add spawn logic to enemy wave sequence
3. Add draw code in canvas render section
4. Define collision behavior in collision detection code
5. Add power-up drop logic on enemy death

### Modifying Game Balance
1. Enemy spawn rates: Adjust in wave sequence definitions
2. Damage values: Found in collision detection code
3. Health pools: Player `hp` property, enemy `hp` in spawn logic
4. Score multipliers: Combo increase logic and point calculations
5. Difficulty scaling: Parameters in difficulty selection

### Adding Visual Effects
1. Create animation keyframes in CSS (e.g., `@keyframes`)
2. Add HTML element with ID in overlay or touchControls
3. Trigger animation by adding `.show` or similar class
4. Remove after animation completes (via `animationend` event)

### Implementing New Input Controls
1. Add event listener to `#gameCanvas` or `document`
2. Update player velocity based on input state
3. For touch: Modify touch zone logic in `#touchControls`
4. Test on both keyboard and mobile devices

---

## Performance and Browser Compatibility

- **Supported browsers**: Modern browsers with Canvas and requestAnimationFrame
- **Target screen size**: 800×600 (game canvas), scales responsively
- **Frame rate**: 60 FPS (requestAnimationFrame)
- **Memory**: Pooling pattern used to avoid garbage collection stalls
- **Touch delay**: 0ms (user-scalable disabled for immediate response)

---

## References & Context

- **Raiden-style gameplay**: Vertical scrolling shooter with waves and bosses
- **Canvas API**: MDN Web Docs on Canvas 2D Context
- **Touch Events**: MDN Web Docs on Touch Events
- **Game Loop**: Standard requestAnimationFrame pattern
- **Collision detection**: AABB (bounding box) approach

---

## Debugging Tips

- Open browser DevTools console to check for errors
- Use `console.log()` to trace game state
- Check canvas rendering by inspecting element in DevTools
- Test different difficulty levels to verify balance changes
- Validate touch controls on actual mobile device (emulator may differ)
- Monitor FPS in DevTools Performance tab

---

## Version History

- **Current**: Single-file Sky Fighter implementation
- **Date**: 2026-02-23
- **Status**: Active development and enhancement

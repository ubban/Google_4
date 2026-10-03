# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up browser game built with a single HTML file containing embedded CSS and JavaScript (1669 lines total). Players control an aircraft, switch between weapon types, drop smart bombs, and defeat stage bosses.

### Key Features
- **Three difficulty levels**: Easy, Normal, Hard with scaling enemy behavior
- **Six themed stages**: Each with unique visual themes (Daylight, Sunset, Night, Storm, Arctic, Volcanic)
- **Multiple weapons**: Vulcan, Laser, Missiles with leveling system
- **Boss battles**: Multi-stage encounters with phase transitions
- **Score system**: Real-time scoring with multipliers, local high score persistence
- **Bomb mechanic**: Limited-use smart bombs that clear the screen
- **Touch controls**: Mobile-optimized virtual joystick and buttons
- **Accessibility**: Screen reader announcements, reduced motion support, keyboard controls
- **Performance adaptive**: Dynamic quality adjustment based on frame timing (3 quality levels: full, medium, low)

## Architecture

The game is a single-file application with three main sections:

### 1. HTML & DOM Structure (lines 1-421)
- Canvas element for game rendering
- HUD elements for score, health, weapon, multiplier, bombs
- Boss health bar display
- Overlay system for menus (start, game over, stage transition)
- Touch control zones (joystick + buttons)
- Accessibility elements (sr-live for screen reader announcements)

### 2. Styling (lines 10-358)
- Dark space theme with semi-transparent UI elements
- Responsive design using `clamp()` for fluid scaling
- Animations: heartbeat (critical health), combo pulses, card transitions, button effects
- Safe area support for notched devices (iPhone)
- Dark mode as default with `prefers-color-scheme` media queries

### 3. Game Logic in JavaScript (lines 422-1669)
IIFE (Immediately Invoked Function Expression) wrapping all game code for encapsulation.

## Game Loop & State Management

### Main Update Loop
```
updateGame(dt) → updates all game entities → collision detection → score updates → HUD updates
```

### Key Game States
1. **READY** - Awaiting player start, showing menu overlay
2. **PLAYING** - Active gameplay
3. **PAUSED** - Player paused the game
4. **DEAD** - Player lost all lives, showing game over screen
5. **STAGE_CLEAR** - Defeated stage boss, showing stage transition

## Key Systems

### 1. Difficulty Configuration (Lines 485-489)
Three presets that scale:
- Enemy spawn rates (`spawnBase`, `spawnMin`)
- Enemy movement speed (`enemySpeed`)
- Enemy firing frequency (`enemyShootChance`)
- Starting lives and bombs
- Boss HP multiplier (`bossHpMul`)

```javascript
const DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
};
```

### 2. Visual Themes (Lines 493-498)
Six themed backgrounds with different sky gradients, cloud colors, and sun tones. New themes automatically applied per stage in a cycle.

### 3. Input Handling
- **Keyboard**: WASD/Arrow keys for movement, Space/Ctrl for fire/bomb, P for pause, M for mute
- **Mouse**: Move to player position (when not on mobile)
- **Touch**: Virtual joystick (left 55% of screen) and action buttons (Fire/Bomb on right 45%)

### 4. Weapon System
**VULCAN** (default), **LASER**, **MISSILES** - Each with:
- Level progression (1-5)
- Unique firing patterns and speeds
- Different damage characteristics
- Automatic leveling on powerup collection

### 5. Enemy AI
- **Spawning**: Throttled based on difficulty and stage progress
- **Movement**: Downward with occasional lateral drifting
- **Shooting**: Probabilistic based on difficulty, only when visible
- **Small enemy groups**: 1-3 HP, quick despawn
- **Big enemies**: 5 HP, award multiplier bonuses
- **Bosses**: 12-16 HP (scaled by difficulty), distinct visual appearance

### 6. Collision System
- **Bullet-enemy**: Damage application + despawn
- **Enemy bullet-player**: Health loss + vignette flash effect
- **Player-enemy**: Immediate death condition
- **Powerups**: Weapon level/bomb acquisition

### 7. Scoring & Multiplier
- Base points per enemy: ~10
- Multiplier increases on consecutive kills (1x → 8x max)
- Multiplier resets on player damage
- Boss defeats award large bonuses
- Local storage persists best score

### 8. Performance Adaptive System
- Tracks last 45 frame times
- If avg > 25ms (40fps), reduce quality
- If avg < 17.8ms (56fps), improve quality
- Three quality levels affect particle density and visual effects

## File Structure

Single file: `index.html`
```
index.html
├── Head section
│   ├── Meta tags (viewport, description, theme-color)
│   ├── Favicon (inline SVG)
│   └── Styles (all CSS inline)
├── Body
│   ├── Canvas container
│   ├── Game UI elements (HUD, overlay, boss bar)
│   ├── Touch controls
│   └── Top control buttons (mute, motion, pause)
└── Script
    └── Game logic (IIFE wrapper)
```

## Key Constants

```javascript
// Storage keys for persistence
const STORAGE_BEST = 'skyfighter.bestScore';
const STORAGE_MUTE = 'skyfighter.muted';
const STORAGE_DIFF = 'skyfighter.difficulty';
const STORAGE_REDUCED = 'skyfighter.reducedMotion';

// Stage progression
const KILLS_PER_STAGE = 18; // Enemies to defeat per stage

// Game dimensions
Canvas is auto-sized to fit viewport while maintaining aspect ratio
```

## Development Notes

### Canvas Rendering
- 2D canvas context used for all rendering
- Sprite-based rendering (no external assets - all drawn with primitives)
- Coordinates: origin (0,0) at top-left, Y increases downward
- Resolution scales with viewport for responsive gameplay

### Entity Management
- All game entities (player, enemies, bullets, powerups) tracked in arrays
- Entities removed when off-screen or defeated
- Frame-by-frame update pattern with delta time (`dt`) parameter

### State Persistence
- Best score stored in `localStorage`
- User preferences (mute, difficulty, reduced motion) persisted
- Data survives page refreshes

### Accessibility Features
- Screen reader live region (`#srLive`) announces game events
- Keyboard controls fully support gameplay
- Reduced motion support respects `prefers-reduced-motion` media query
- Semantic HTML with proper ARIA attributes where applicable
- Text contrast ratios meet WCAG AA standards

### Mobile Optimization
- Responsive design with fluid typography using `clamp()`
- Touch event handling with `touch-action: none` to prevent browser gestures
- Safe area inset support for notched devices
- Virtual joystick for movement, buttons for fire/bomb

## Extending the Game

### Adding a New Weapon
1. Define weapon in weapons object with: `fireRate`, `projectileSpeed`, `damage`, `firePattern` function
2. Update weapon cycling logic (currently 3 weapons)
3. Add weapon name to HUD display mapping
4. Define firing pattern in the game loop

### Adding a New Enemy Type
1. Create enemy object with: `x`, `y`, `hp`, `width`, `height`, `shootChance`
2. Add to `enemies` array in spawn logic
3. Implement collision response in collision detection
4. Define visual rendering in draw function

### Adding a New Stage Theme
1. Add new theme object to `THEMES` array with: `sky` (gradient stops), `cloud` (RGB), `sun` (RGB), `name` (string)
2. Theme automatically cycles per stage
3. Update background gradient in canvas draw phase

### Audio Support
Currently game has no audio. To add:
1. Use Web Audio API or HTML5 audio elements
2. Track mute state via existing `muteBtn` toggle
3. Load sound files (consider data URIs or web-hosted assets)

### Boss Pattern Design
Current boss uses simple phase transitions based on HP. To create new bosses:
1. Define HP threshold for phases
2. Implement phase-specific attack patterns
3. Vary bullet spread angles, speeds, and spawn positions
4. Add visual feedback (color changes, particle effects)

## Common Debugging Points

- **Frame rate drops**: Check quality variable, watch canvas draw operations
- **Collision issues**: Verify bounding box dimensions and offset calculations
- **Unresponsive controls**: Check input event listeners (keydown, mousemove, touchmove)
- **Storage not persisting**: Verify localStorage is enabled (not incognito/private mode)
- **Mobile controls not working**: Ensure touch event listeners are attached to correct elements

## Browser Compatibility

- Modern browsers with Canvas 2D context support
- LocalStorage API for persistence
- ES6+ JavaScript features used (const/let, arrow functions, template literals)
- CSS Grid and Flexbox for layout
- CSS custom properties (CSS Variables) for theming

## Performance Considerations

- Single canvas for all rendering (good for simple 2D games)
- Entities culled when off-screen to reduce draw calls
- Frame time tracking allows dynamic quality reduction
- No external libraries - pure vanilla JavaScript reduces dependencies
- Assets are procedurally drawn or embedded (no image loading delay)

---

**Last Updated**: 2026-10-03  
**Game Version**: Sky Fighter  
**Maintained by**: Claude Code Agent

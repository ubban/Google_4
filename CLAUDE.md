# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up browser game built entirely in HTML5 and vanilla JavaScript. Players switch between weapons, drop smart bombs, and defeat stage bosses across multiple difficulty levels.

- **Main file**: `index.html` (~1669 lines, single-file application)
- **Technology**: HTML5 Canvas, JavaScript (ES6+), CSS Grid/Flexbox
- **Platform**: Browser-based (desktop and mobile with touch controls)
- **Difficulty levels**: Easy, Normal, Hard
- **Game modes**: Story mode with 6 stages + boss fights

## Architecture

### Code Organization

The entire game is contained within an IIFE (Immediately Invoked Function Expression) to maintain scope isolation. The structure is organized as follows:

1. **DOM Elements & Constants** (lines ~40-80)
   - Canvas and UI element references
   - Storage keys for persistence
   - Game settings and configuration

2. **Game Systems** (organized by function)
   - **Performance Tracking**: Adaptive quality system (`trackPerf()`)
   - **Audio System**: Web Audio API with sound pool for effects
   - **Physics & Movement**: Collision detection, entity movement
   - **Rendering**: Canvas-based drawing system
   - **Game Logic**: Spawn system, scoring, state management

### Key Constants & Configuration

```javascript
DIFFICULTIES: {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
}

KILLS_PER_STAGE: 18  // Required kills to advance to next stage

THEMES: 6 visual themes with sky gradients, cloud colors, and sun colors
```

### Game State

The game maintains state through:
- **LocalStorage**: Best score, mute status, difficulty preference, motion reduction
- **Runtime variables**: Current score, lives, bombs, stage, entity lists, weapon state
- **Canvas rendering context**: 2D drawing operations

## Core Features

### 1. **Player Ship**
- Weapon switching system (multiple weapon types)
- Weapon upgrade levels (increases damage/behavior)
- Health/lives system
- Smart bomb mechanic (screen-clearing attack)
- Input handling: keyboard + touch joystick support

### 2. **Enemy System**
- Multiple enemy types (configurable per difficulty)
- Movement patterns (straight down, sine wave, etc.)
- Shooting patterns (single bullet, spread, burst)
- Health system
- Score rewards for kills
- Multiplier system (higher for consecutive kills)

### 3. **Boss Fights**
- 6 unique bosses (one per stage)
- Boss health bar UI
- Dynamic HP scaling based on difficulty
- Specialized attack patterns

### 4. **Scoring System**
- Base score: enemy kill value
- Combo multiplier: increases with consecutive kills within time window
- Difficulty multiplier
- New best score detection and display

### 5. **Input Systems**
- **Desktop**: Arrow keys (movement) + Z (fire) + X (bomb)
- **Mobile**: Virtual joystick + action buttons
- **UI**: Touch-friendly controls
- **Accessibility**: Screen reader announcements

### 6. **Audio System**
- Web Audio API with audio pool pattern
- Sound effects for: fire, enemy death, player hit, boss defeat, bomb
- Mute toggle (saved to storage)
- Motion reduction option (respects `prefers-reduced-motion`)

### 7. **Visual Polish**
- Multiple environment themes (6 stages with unique sky gradients)
- Hit flash effect with vignette on player damage
- Combo pop-up numbers
- Stage transition banners
- Adaptive pixel density for Retina displays

### 8. **Responsive Design**
- Mobile-first with safe-area-inset support
- Touch-action optimization (prevent browser defaults)
- Adaptive font sizing with `clamp()`
- Joystick deadzone handling

## Development Guidelines

### Adding a New Enemy Type

1. Define enemy in spawn logic with properties: `x, y, hp, speed, shootChance, shootType`
2. Add movement pattern in the game loop
3. Add rendering in the canvas draw function
4. Add sound effect reference if needed

### Adding a New Weapon

1. Create weapon object with: `name, level, speed, spread, damage, behavior`
2. Add weapon switching logic in input handler
3. Implement firing behavior in bullet spawn section
4. Update HUD display logic

### Adding a New Stage/Boss

1. Update `KILLS_PER_STAGE` logic if changing progression
2. Create boss entity with: `hp, shootPattern, movementPattern`
3. Add theme definition to `THEMES` array
4. Implement unique boss attack pattern

### Modifying Difficulty Scaling

Adjust in `DIFFICULTIES` object:
- `spawnBase`: Initial spawn cooldown (higher = fewer enemies)
- `spawnMin`: Minimum spawn cooldown as difficulty increases
- `enemySpeed`: Movement speed multiplier
- `enemyShootChance`: Probability of enemy firing
- `lives`: Player starting health
- `bombs`: Starting bomb count
- `bossHpMul`: Boss health multiplier

### Performance Optimization

The game includes adaptive quality system:
```javascript
quality = 0  // Full detail
quality = 1  // Medium (reduce particle effects)
quality = 2  // Low (further reduce effects)
```

- Tracks frame time average over 45 frames
- Adjusts quality based on target 60 FPS
- Affects particle rendering, effect intensity

## Code Patterns & Conventions

### Naming
- Classes/Constructors: PascalCase (if added in future)
- Functions: camelCase
- Constants: UPPER_SNAKE_CASE
- Private/internal: Prefixed with `_` or suffixed with `_internal`

### Storage Keys
All keys prefixed with `skyfighter.` for namespacing:
- `skyfighter.bestScore`: High score persistence
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Motion reduction preference

### Event Handling
- Keyboard events: `keydown`, `keyup` for continuous movement
- Touch events: Custom joystick implementation on moveZone
- Window events: `resize` for canvas scaling, `deviceorientation` for mobile

## Running & Testing

### Browser Compatibility
- Modern browsers with HTML5 Canvas support
- Tested on: Chrome, Firefox, Safari, Edge
- Mobile: iOS Safari, Chrome Android

### Local Testing
1. Open `index.html` directly in browser or serve via HTTP
2. Game initializes on page load with difficulty selection overlay
3. Select difficulty to start game

### Testing Checklist
- [ ] Player movement (arrow keys + touch joystick)
- [ ] Weapon switching (works correctly)
- [ ] Enemy spawning and movement patterns
- [ ] Hit detection (player-enemy, bullet-enemy)
- [ ] Boss battles complete successfully
- [ ] Score calculation and multiplier
- [ ] Audio on/off toggle
- [ ] Mobile touch controls responsive
- [ ] High score persistence across sessions
- [ ] Difficulty selection saves preference
- [ ] Screen reader announcements work

### Debug Tips
- Open browser DevTools console
- Check `localStorage` for game state
- Monitor frame time in performance profiler
- Use canvas drawing order to debug layer issues

## Common Tasks

### To adjust game difficulty progression:
1. Modify `KILLS_PER_STAGE` constant
2. Adjust `DIFFICULTIES` parameters
3. Modify boss HP scaling in boss creation logic

### To add visual theme:
1. Add new entry to `THEMES` array with sky gradient colors
2. Update stage-to-theme mapping logic
3. Test gradient looks good at various screen sizes

### To change controls:
1. Update keyboard key mapping in input handler
2. Adjust touch joystick sensitivity constants
3. Update HUD button positions/sizing if needed

### To optimize for low-end devices:
1. Lower `spawnBase` values to reduce entities
2. Enable `quality = 2` by default
3. Reduce particle/effect count
4. Profile with performance monitor

## Technical Notes

### Canvas Setup
- High DPI awareness: canvas.width/height set based on devicePixelRatio
- Maintains aspect ratio: 640x960 base (9:16 portrait)
- Responsive scaling while maintaining aspect ratio

### Collision Detection
- Simple bounding box (rectangle) collision
- Hit detection between: bullets-enemies, player-enemies, bombs-entities
- Adaptive based on entity size properties

### Audio Pool Pattern
- Pre-allocates audio context and buffer sources
- Reuses audio nodes instead of creating new ones each play
- Reduces GC pressure during gameplay

### Animation Timing
- RequestAnimationFrame for smooth 60 FPS target
- Delta time tracking for frame-rate independent movement
- Adaptive quality reduces animation/effect load

## Browser Storage

Game persists the following using localStorage:
- **Best Score**: Maximum score achieved
- **Mute State**: Audio on/off preference
- **Difficulty**: Last selected game difficulty
- **Reduced Motion**: Accessibility preference flag

## UI/UX Elements

### HUD Layout
- Top: Score, Best Score, Stage/Multiplier, Weapon Info
- Bottom: Lives (♥), Bombs (✦)
- Accessibility: Screen reader live region

### Overlays
- Start screen: Difficulty selection
- Game over: Final stats, best score comparison
- Stage transitions: Stage number and theme name

### Controls Indicator
- Desktop: Shows keyboard hints
- Mobile: Virtual joystick + buttons auto-show

## Future Enhancement Ideas

- [ ] Leaderboard system with cloud sync
- [ ] Additional weapon types (laser, homing missiles)
- [ ] Enemy variety (faster, armored, special drops)
- [ ] Power-ups system (health restore, temporary invincibility)
- [ ] Campaign progression unlocks
- [ ] Local multiplayer support
- [ ] Replay system
- [ ] Settings menu (graphics, audio settings)
- [ ] Achievements/badges

## Accessibility Features

- **Keyboard controls**: Full keyboard support
- **Screen reader**: Live region announcements for game events
- **Reduced motion**: Respects `prefers-reduced-motion` preference
- **Color contrast**: All UI elements meet WCAG standards
- **Mobile touch**: Full touch control support
- **High DPI**: Crisp rendering on Retina/high-DPI displays

## Performance Targets

- **FPS**: 60 FPS target on desktop
- **Load time**: < 1 second
- **Memory**: < 50MB typical
- **Adaptive quality**: Maintains 40+ FPS minimum on low-end devices

---

**Last Updated**: October 2026
**Game Engine**: Vanilla HTML5 Canvas / JavaScript ES6+
**Status**: Production-ready single-file game

# Sky Fighter - Codebase Documentation

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5, CSS3, and JavaScript. Players navigate through multiple stages, collect weapon power-ups, and battle bosses while managing limited lives and smart bombs.

## Project Structure

The project is a single-file application:
- **index.html** — Main game file containing all HTML markup, CSS styles, and JavaScript game logic

### Key Files
- `README.md` — Project metadata
- `CLAUDE.md` — This documentation file

## Game Architecture

### Core Game Loop
The game runs at ~60 FPS using `requestAnimationFrame()`. The main loop:
1. Processes input (keyboard, mouse, touch)
2. Updates game state (player position, enemies, projectiles, effects)
3. Detects collisions
4. Renders to canvas
5. Updates HUD

### Game States
- **Menu** — Initial overlay with difficulty selection
- **Playing** — Active gameplay
- **Paused** — Game paused via P/Esc or pause button
- **GameOver** — Player defeated, shows final score

## Core Game Mechanics

### Player Ship
- **Movement**: Constrained to game bounds, follows mouse/touch position or keyboard input
- **Health**: Displayed as hearts (❤️), configurable per difficulty
- **Weapons**: Multiple weapon types with progression levels
  - VULCAN: Basic rapid-fire (default)
  - SPREAD: Wide cone pattern
  - BURST: High-velocity projectiles
  - LASER: Continuous beam
  - MISSILE: Homing projectiles
- **Bombs**: Smart bombs that clear screen and damage bosses; limited count

### Enemy Types
- **Regular Enemies**: Spawn continuously, shoot at player
- **Boss Enemies**: Multi-phase encounters at stage milestones
- **Spawning**: Difficulty-based spawn rates (`spawnBase`, `spawnMin`)

### Scoring System
- Enemy kills grant points
- Multiplier increases with consecutive kills (displayed as "x" multiplier)
- Boss defeats trigger stage advancement and multiplier bonuses
- Score tracked: current (`score`) and best (`bestScore`)

### Difficulty Levels
```javascript
// Configuration per difficulty
DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
}
```

## Visual Design

### Canvas Rendering
- **Resolution**: Responsive, scales to fit viewport while maintaining aspect ratio
- **Background**: Gradient sky with parallax clouds for depth
- **Rendering Strategy**: Separate passes for sky, player, enemies, projectiles, effects

### Theme System
Six visual themes rotate per stage:
1. Daylight Front (blue sky)
2. Sunset Corridor (orange/purple)
3. Night Approach (dark blue)
4. Storm Front (green/gray)
5. Arctic Wastes (cyan)
6. Volcanic Ridge (red/orange)

Each theme defines sky gradient, cloud color, and sun appearance.

### Visual Effects
- **Vignette**: Red pulsing overlay when player takes damage (critical health)
- **Flash**: White flash on hit or bomb detonation
- **Combo Display**: Animated text showing multiplier achievements
- **Stage Banner**: Title animation when entering new stage
- **Boss Health Bar**: Top-center display during boss fights

### Responsive UI
- **HUD Stats**: Score, best score, multiplier, weapon level, lives, bombs
- **Touch Controls**: Mobile-specific buttons (FIRE, BOMB) and joystick
- **Control Hints**: Keyboard and mobile input instructions
- **Accessibility**: Screen reader announcements (`sr-only` element)

## Input Handling

### Desktop Input
- **Keyboard**: Arrow keys or WASD for movement, Space/X/Shift for actions
- **Mouse**: Move to aim, click to fire
- **Buttons**: P/Esc for pause, top-right UI buttons (mute, motion, pause)

### Mobile Input
- **Touch Drag**: Left side of screen to move (joystick)
- **Touch Buttons**: Right side buttons for fire and bomb
- **Gestures**: Tap to interact with menu buttons

## Storage & Persistence

LocalStorage keys (prefixed with `skyfighter.`):
- `bestScore` — High score
- `muted` — Audio mute state
- `difficulty` — Last selected difficulty
- `reducedMotion` — Accessibility setting for reduced animations

## Accessibility Features

- **Screen Reader Support**: Live region announcements for score, stage, results
- **Reduced Motion**: Respects `prefers-reduced-motion` media query
  - Disables animations: vignette, hearts pulse, combo pop, stage banner, flash, button transitions
- **Keyboard Navigation**: Fully playable via keyboard
- **Color Contrast**: High-contrast HUD with text shadows

## Audio System

Basic sound effects infrastructure:
- Mute toggle button (🔊)
- Audio state persisted to localStorage
- Individual sound triggers for events (shooting, hit, bomb, etc.)
- Background music on loop

## Performance Optimization

### Adaptive Quality
Monitors frame times and adjusts quality level:
- **Level 0**: Full quality (target 60 FPS)
- **Level 1**: Medium (drops some visual effects)
- **Level 2**: Low (minimal effects, optimized rendering)

Automatically degrades if average frame time exceeds 1/40 sec.

### Canvas Optimization
- Double-buffering via canvas context
- Minimal DOM manipulation during gameplay
- Efficient collision detection (spatial partitioning possible)

## Code Organization

The JavaScript is organized as a single IIFE with sections:
1. **Setup**: Element references, constants, storage
2. **Game State**: Variables tracking game progress
3. **Utilities**: Helper functions (random, clamp, angle conversion)
4. **Rendering**: Canvas drawing functions
5. **Input**: Keyboard, mouse, touch handlers
6. **Game Logic**: Update, collision, entity management
7. **Event Listeners**: Button clicks, window resize
8. **Main Loop**: Game tick and frame management

## Development Guidelines

### Adding New Features

**New Weapon Type**:
1. Define weapon properties in weapon configuration object
2. Add rendering logic to player weapon draw function
3. Update HUD weapon name display

**New Enemy Type**:
1. Create enemy class/factory function
2. Define AI behavior in update method
3. Add rendering logic
4. Register in enemy spawning logic

**New Visual Theme**:
1. Add theme to THEMES array with sky gradient, cloud color, sun color
2. Ensure sufficient contrast for visibility

**Performance**:
- Keep entities under 500 active (enemies + projectiles combined)
- Profile with DevTools Performance tab
- Test on low-end mobile devices

### Debugging

- **Console Logs**: Add debug logs with condition flag
- **Performance**: Chrome DevTools > Performance tab, record 30-60 sec gameplay
- **Mobile Testing**: Use Chrome Device Emulation or Android device
- **Accessibility**: Axe DevTools, keyboard-only navigation test

## Browser Compatibility

- Modern browsers with Canvas 2D API
- Touch Events API for mobile
- LocalStorage for persistence
- requestAnimationFrame for smooth animation

## Testing Recommendations

1. **Functionality**: Play through all difficulty levels to stage 3+
2. **Input**: Verify keyboard, mouse, touch all work correctly
3. **Mobile**: Test on various screen sizes (portrait/landscape)
4. **Accessibility**: Navigate with keyboard only, test with screen reader
5. **Performance**: Monitor FPS on low-end devices
6. **Storage**: Clear localStorage and verify defaults load correctly

## Future Enhancement Ideas

- **Leaderboard**: Online high score tracking
- **Progression**: Unlockable weapons or stages
- **Audio**: Music tracks and sound effects
- **Effects**: Particle systems for explosions
- **Mobile Optimizations**: Haptic feedback, gyro controls
- **Statistics**: Track playtime, total kills, favorite weapon

# Sky Fighter - Codebase Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. Features include:
- Dynamic weapon system (Vulcan and Laser with progressive levels)
- Three difficulty tiers with escalating enemy patterns
- Boss encounters with unique attack patterns (Vanguard, Dreadnought, Wraith)
- Score multiplier and chain combo system
- Smart bomb mechanics
- Adaptive performance scaling
- Full WebAudio synthesized sound effects (no external audio assets)
- Touch and keyboard controls with mobile joystick support
- Reduced motion accessibility option

## Architecture

### Single File Design
The entire game is contained in `index.html` (1,669 lines):
- **Lines 1-425**: HTML markup and CSS styling
- **Lines 426+**: JavaScript game engine

### HTML Structure
```html
<canvas id="gameCanvas">        <!-- Main render target -->
<div id="overlay">              <!-- Menu/game-over screens -->
<div id="hud">                  <!-- Score, lives, weapon info -->
<div id="touchControls">        <!-- Mobile joystick and buttons -->
<div id="flash">                <!-- Visual feedback (damage, bomb) -->
<div id="vignette">             <!-- Screen shake effect -->
```

## Game Systems

### 1. Core Game Loop
- **Canvas sizing**: Responsive 480x720 base resolution with adaptive scaling
- **Frame timing**: Delta-time driven with adaptive performance tracking
- **Update cycle**: Player → Bullets → Enemies → Collisions → Rendering

### 2. Audio System
Uses WebAudio API to synthesize all SFX:
- Player gunfire (Vulcan: rapid bursts, Laser: tone sweeps)
- Enemy hits and explosions
- Power-up collection
- UI interactions
- Boss patterns (unique tones per boss)

All generated procedurally; no external audio files needed.

### 3. Input Handling
Multiple input methods with priority (keyboard > joystick > mouse):
- **Keyboard**: Arrow keys (move), Space (fire), X/Shift (bomb), P/Esc (pause)
- **Touch**: Joystick in move zone, separate fire and bomb buttons
- **Mouse**: Movement toward crosshair (when canvas is hovered), click to fire
- Joystick implementation uses 2-point touch tracking with 50px deadzone radius

### 4. Player Mechanics
```javascript
player = {
  x, y,              // Position
  w: 40, h: 40,      // Collision box
  speed: 260,        // Pixels/second
  weapon: 'vulcan'|'laser',
  level: 1,          // Weapon level (1-3)
  cooldown: 0,       // Firing cooldown timer
  rapid: 0,          // Vulcan rapid-fire counter
  invuln: 0          // Invulnerability duration
}
```

### 5. Weapon Systems
- **Vulcan (Default)**: Single rapid-fire stream, highest DPS
  - Level 1: 1 bullet per cycle
  - Level 2: 2 bullets per cycle (spread)
  - Level 3: 3 bullets per cycle (wide spread)
  
- **Laser**: Long-range beam, slower fire rate
  - Level 1: Single beam
  - Level 2: Double beam (V-pattern)
  - Level 3: Triple beam (wide coverage)

- **Power-ups**: 
  - Weapon upgrades (drop during combat, auto-collect when passing)
  - Bomb refills (up to max)

### 6. Enemy Types
Three enemy archetypes with pattern variations:
- **Regular enemies**: Spawn in formations, fire simple spreads
- **Weighted enemies**: Higher health, tougher patterns
- **Powerup-dropping variants**: Indicate power-up contents visually

Enemies use configurable patterns:
- Aimed shots (track player position)
- Spread patterns (radial, fan, wall)
- Time-delayed volleys
- Homing bullets (specific patterns only)

### 7. Boss System
Three progressive bosses encountered every 18 enemy kills:
1. **Vanguard**: Radial burst → aimed fan → wide V-fan
2. **Dreadnought**: 8-way cross fire + homing missile volley + dense wall
3. **Wraith**: Teleport barrage + expanding double ring + aimed cross-spread

Boss warning triggers 2 seconds before spawn. Boss bar shows health/max.

### 8. Difficulty Settings
Three tiers stored in localStorage:
- **Easy**: 2 starting bombs, slower spawns, weaker patterns
- **Normal**: 2 starting bombs, standard difficulty (default)
- **Hard**: 1 starting bomb, aggressive patterns, tight timing

### 9. Scoring System
- **Base**: 10 points per enemy kill
- **Multiplier**: 1x baseline, increases per unbroken chain
- **Chain/Combo**: Counter visible when ≥3 kills without taking damage
- **Best score**: Persisted to localStorage

### 10. Performance Management
Adaptive quality scaling when frame times exceed target (~16.67ms @ 60fps):
- Quality 0 (full): All particles, full effects
- Quality 1 (medium): Reduced particle count
- Quality 2 (low): Minimal particles, basic effects

## Key Game Constants

```javascript
const BASE_W = 480, BASE_H = 720;           // Canvas dimensions
const KILLS_PER_STAGE = 18;                 // Boss spawn interval
const DIFFICULTIES = {                       // Per-difficulty config
  easy: { lives: 4, bombs: 2, ... },
  normal: { lives: 3, bombs: 2, ... },
  hard: { lives: 3, bombs: 1, ... }
}
```

## Collision & Physics

- **AABB collision**: All entities use rectangular bounding boxes
- **Particle system**: Temporary visual/damage effects
- **Screen wrapping**: Bullets/enemies don't wrap; they despawn at edges
- **Invulnerability**: Player has invuln frames after getting hit (visual flashing)

## Visual Effects

- **Screen shake**: `screenShake(magnitude, duration)` - reduced if motion preference set
- **Flash overlay**: `flashHit()` and `flashBomb()` - brief white overlay for impacts
- **Vignette**: Red pulsing overlay during boss encounters (respects reduced motion)
- **Combo pop**: Center-screen text announces chain milestones
- **Stage banner**: Top-of-screen notification when boss approaches

## State Management

### Persistent State (localStorage)
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Selected difficulty
- `skyfighter.reducedMotion`: Motion reduction override

### Runtime State
- `score`, `lives`, `bombCount`, `kills`, `stage`
- `chain`, `multiplier` (combo system)
- `running`, `paused`, `gameOver` (state flags)
- `frame`, `spawnTimer`, `chainTimer` (timers)

## Code Organization Patterns

### Entity Arrays
```javascript
let bullets = [];          // Player projectiles
let enemies = [];          // Active enemies
let enemyBullets = [];     // Enemy projectiles
let particles = [];        // Visual effects
let powerUps = [];         // Collectable power-ups
let boss = null;           // Current boss (null when not fighting)
```

### Update Pattern
Each entity type follows:
1. Move (by velocity)
2. Apply effects (cooldowns, timers)
3. Check collisions
4. Update sprite/animation
5. Remove if dead/off-screen

### Rendering Pattern
```javascript
ctx.clearRect(0, 0, canvas.width, canvas.height);  // Clear
// Draw bg/parallax (if any)
// Draw player
// Draw all bullets, enemies, particles
// Draw HUD
```

## Development Guidelines

### Adding New Features
1. **New enemy type**: Add to `ENEMY_TYPES` object, define in enemy spawn handler
2. **New weapon**: Extend player weapon property, add fire/cooldown logic
3. **New visual effect**: Add to particle system or create dedicated effect (like vignette)
4. **New boss**: Add to `BOSSES` array, implement attack pattern logic in boss update

### Common Tasks
- **Adjust difficulty**: Modify `DIFFICULTIES` object values
- **Balance damage/health**: Find constants in collision detection sections
- **Tweak spawn rates**: Modify spawn timer thresholds
- **Change visual timing**: Adjust animation durations and effect timings
- **Modify input**: Update `keys` handling in keydown/keyup listeners

### Performance Considerations
- Use particle system sparingly (respects quality setting)
- Avoid frequent DOM queries outside event setup
- Canvas drawing is primary bottleneck; quality scaling is fallback
- Audio synthesis is cached per sound to avoid real-time generation cost

## Accessibility Features
- **Reduced motion**: Respects prefers-reduced-motion; can override in settings
- **Screen reader support**: ARIA labels on buttons, live region for combo announces
- **Keyboard fully playable**: No mouse or touch required
- **Touch optimized**: Joystick and large button targets
- **High contrast**: Dark theme with bright UI elements

## Testing Notes
- Game saves state locally; clear localStorage between test runs if needed
- Boss difficulty can be tested by modifying `KILLS_PER_STAGE` temporarily
- Audio can be muted via UI button or `muted` variable
- Frame rate can be monitored via `frameTimes` array (performance tracking)

## File Dependencies
None. This is a single, self-contained HTML file with no external dependencies:
- No external CSS frameworks
- No external JS libraries
- No external image/audio assets
- Pure Canvas API for rendering
- WebAudio API for sound
- localStorage for persistence

Everything needed to run the game is in `index.html`.

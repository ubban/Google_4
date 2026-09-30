# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in HTML5 Canvas and JavaScript. It's a single-file, self-contained game with responsive design, mobile touch support, accessibility features, and adaptive performance optimization.

- **Type**: Single-file HTML5 game
- **Architecture**: Vanilla JavaScript with Canvas API
- **Platform**: Browser-based (desktop and mobile)
- **Main File**: `index.html` (~60KB, includes HTML, CSS, and JavaScript)

## Game Concept

Players pilot a fighter jet through vertical stages, defeating enemies and bosses while collecting weapon power-ups and smart bombs. The game features:
- 3 difficulty levels (Easy, Normal, Hard)
- Progressive stage system with boss encounters
- Multiple weapon types with upgrade levels
- Lives and bomb mechanics
- Combo multiplier system
- High score tracking

## Architecture

### Single-File Structure
The entire game is contained in `index.html`, organized into three sections:
1. **HTML** (lines 1-420): DOM structure, game canvas, UI elements, HUD
2. **CSS** (lines 10-358): Styling, animations, responsive layout, dark theme
3. **JavaScript** (line 422+): Core game logic, engine, entities, systems

### Key Game Systems

#### 1. Game Loop & Timing
- Uses `requestAnimationFrame` for 60 FPS target
- Delta-time based physics (frame-independent movement)
- Adaptive performance: auto-adjusts quality based on frame rate
  - Quality 0 = full rendering
  - Quality 1 = medium (reduces particle effects)
  - Quality 2 = low (minimal visual effects)

#### 2. Entity System
The game manages several entity types:
- **Player**: Controlled ship with health, weapons, bombs
- **Enemies**: Various types with AI, health, shooting patterns
- **Projectiles**: Player bullets and enemy fire
- **Particles**: Visual effects (explosions, impacts)
- **Collectibles**: Weapon power-ups, bomb drops

#### 3. Game State Machine
- **Menu**: Title screen with difficulty selection
- **Playing**: Active gameplay
- **Paused**: Game temporarily halted
- **GameOver**: End screen with results

#### 4. Difficulty System
Three preset configurations affect gameplay balance:

```javascript
DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
};
```

Parameters:
- `spawnBase`: Initial enemy spawn rate (frames between spawns)
- `spawnMin`: Minimum spawn rate at high stages
- `enemySpeed`: Movement speed multiplier
- `enemyShootChance`: Per-frame probability enemies fire
- `lives`: Starting player health
- `bombs`: Starting bomb count
- `bossHpMul`: Boss health multiplier

#### 5. Weapon System
Multiple weapons with level progression:
- **Vulcan**: Fast, low-damage bullets
- **Railgun**: Penetrating shots
- **Twin**: Double side shots
- **Cluster**: Spreading bullet pattern

Weapons upgrade through pickups, increasing fire rate and damage.

#### 6. Scoring & Progression
- **Combo Multiplier**: Increases with consecutive hits, decays over time
- **Kills per Stage**: 18 enemies required to advance (KILLS_PER_STAGE)
- **Stage Themes**: 6 different visual themes with unique color palettes
- **Best Score**: Persisted in localStorage

## Key Game Constants

```javascript
KILLS_PER_STAGE = 18;           // Enemies to defeat per stage

THEMES = [                       // Visual themes (6 total)
  { sky, cloud, sun, name }      // Sky gradient, particle colors, theme name
];
```

## UI Components (HUD)

### In-Game Display
- **Score**: Current game score
- **Best**: High score (persisted)
- **Multiplier**: Current combo multiplier (x indicator)
- **Weapon**: Current weapon name and level
- **Hearts**: Remaining lives visual indicator
- **Bombs**: Available smart bombs visual indicator
- **Boss Bar**: Health bar appears when boss is active

### Overlay Elements
- **Title Screen**: Game title with start button and difficulty selector
- **Game Over Screen**: Final score, stage reached, best score, restart button
- **Combo Popup**: Displays combo milestone (animated, centered)
- **Stage Banner**: Shows stage name when transitioning
- **Accessibility**: Screen reader announcements (aria-live region)

### Control Buttons
- **Mute (🔊)**: Toggle audio on/off
- **Motion (🎆)**: Toggle reduced motion preference
- **Pause (⏸)**: Pause/resume game

## Input & Controls

### Desktop Controls
- **Movement**: Mouse position or Arrow Keys/WASD
- **Fire**: Mouse click or Space
- **Bomb**: X or Shift key
- **Pause**: P or Escape

### Mobile Controls
- **Movement**: Drag left side of screen with joystick visual
- **Fire**: Tap FIRE button (bottom right)
- **Bomb**: Tap BOMB button (top right)
- **Controls**: Show joystick base and touch buttons only on mobile

### Touch Handling
- Touch-drag for movement zones (left 55% width)
- Dedicated fire/bomb buttons (right 45% width)
- Joystick feedback (visual base + stick indicator)

## Visual Features

### Animations
- **Combo Pulse**: Milestone announcements pulse and fade
- **Stage Banner**: Stage transitions slide in and fade (2s)
- **Heart Beat**: Critical health (≤1 heart) triggers pulse
- **Flash Effects**: 
  - Hit flash: Quick white screen on damage
  - Bomb flash: Slower flash on bomb detonation
- **Vignette**: Red edge vignette when hit (with pulse animation)

### Responsive Design
- `clamp()` CSS values for scalable UI on all screen sizes
- Safe area insets for notched devices (`env(safe-area-inset-*)`)
- Canvas scales to fit viewport while maintaining aspect ratio
- Mobile-first approach with touch-optimized buttons

### Color Themes
6 atmospheric themes affect sky gradient and particle colors:
1. **Daylight Front**: Blue sky, white clouds
2. **Sunset Corridor**: Purple-orange gradient
3. **Night Approach**: Deep blue, cool tones
4. **Storm Front**: Green storm aesthetic
5. **Arctic Wastes**: Light blue ice tones
6. **Volcanic Ridge**: Red-orange volcanic theme

## Storage & Persistence

Uses `localStorage` for player preferences:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Accessibility motion preference

## Performance Optimization

### Adaptive Quality System
```javascript
quality = 0;  // Full quality
quality = 1;  // Medium (reduced particles)
quality = 2;  // Low (minimal effects)
```

Adjusts based on average frame time over 45 frames:
- Target: ~60 FPS (16.7ms per frame)
- If avg > 25ms (40 FPS): Degrade quality
- If avg < 17.9ms (56 FPS): Improve quality

### Canvas Optimization
- Single canvas context for all rendering
- Particle pooling for recycled effect objects
- Entity culling (off-screen enemies removed)
- Batch rendering where possible

## Accessibility Features

### Screen Reader Support
- Live region (`srLive`) announces game events
- Semantic HTML buttons with `title` attributes
- Reduced motion CSS respects `prefers-reduced-motion`
- High contrast UI suitable for vision impairment

### Motion Sensitivity
Toggle via motion button disables:
- Vignette pulses
- Heart beat animations
- Combo popups
- Flash effects
- Screen shakes (if implemented)

## Key Functions & Systems

### Game Initialization
- Canvas setup and context acquisition
- Event listener registration (keyboard, mouse, touch, resize)
- localStorage restoration
- DOM element caching

### Game Loop
1. Poll input state
2. Update player position and firing
3. Update/spawn enemies
4. Update projectiles and collisions
5. Update particles
6. Render scene and HUD
7. Track frame time for performance

### Collision Detection
- Bounding box collisions for bullets/enemies
- Radius-based detection for bomb effects
- Player hit zones

### Particle System
- Explosion effects (multiple particles radiating)
- Debris particles
- Trail effects
- Color-based on theme

### Boss System
- Boss appears after KILLS_PER_STAGE defeats
- Boss health bar display
- Distinct visual design
- Enhanced attack patterns

## Development Notes

### Code Organization
The JavaScript is organized within an IIFE (Immediately Invoked Function Expression) to:
- Avoid global scope pollution
- Maintain private state
- Provide encapsulation

### Constants & Configuration
Game behavior is easily tunable via constants at the top:
- `DIFFICULTIES`: Adjust per-difficulty parameters
- `KILLS_PER_STAGE`: Progression pacing
- `THEMES`: Visual variety
- Speed/damage constants in entity definitions

### Future Enhancement Areas
1. Sound effects and music system (structure in place for mute toggle)
2. Leaderboard/cloud save (beyond localStorage)
3. Additional weapon types or special abilities
4. Boss variety beyond current patterns
5. Particle effects refinement
6. Mobile haptic feedback

## Browser Compatibility

- **Requires**: HTML5 Canvas, ES6 JavaScript
- **Tested**: Modern browsers (Chrome, Firefox, Safari, Edge)
- **Mobile**: iOS Safari, Android Chrome
- **Fallbacks**: Graceful degradation if Canvas unavailable

## File Structure

```
Google_4/
├── index.html       # Complete game (HTML + CSS + JS)
├── README.md        # Project metadata
└── Claude.md        # This documentation
```

## How to Play

1. **Start**: Click "Start Game" on title screen
2. **Select Difficulty**: Choose Easy, Normal, or Hard
3. **Fly**: Move mouse/drag (or use arrow keys)
4. **Fire**: Click/space bar continuously
5. **Bomb**: Press X/Shift when needed (smart detonation)
6. **Progress**: Defeat 18 enemies per stage to advance
7. **Bosses**: Appear after each stage completion
8. **High Score**: Try to beat your best score!

## Technical Specifications

- **Canvas Resolution**: Adaptive to window size
- **Target FPS**: 60 (with adaptive fallback)
- **Color Space**: RGB (8-bit per channel)
- **Input Latency**: Frame-synchronized (no buffer)
- **Data Persistence**: localStorage (max ~5-10MB per domain)

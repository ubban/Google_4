# Sky Fighter: Codebase Documentation

## Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 web application. Players control a fighter jet, defeat waves of enemies across multiple stages, and face challenging boss battles. The game features dynamic difficulty settings, weapon upgrades, power-ups, and detailed visual effects.

**File**: `index.html` (1669 lines)
**Technology**: Vanilla HTML5, CSS3, JavaScript (Canvas 2D API)
**Play URL**: Open `index.html` in a modern web browser

---

## Architecture Overview

The entire game is contained in a single `index.html` file with three main sections:

1. **HTML Structure** (lines 1-421): DOM elements for game canvas, HUD, menus, and controls
2. **CSS Styling** (lines 10-361): Game canvas styles, animations, HUD layouts, responsive design
3. **JavaScript Engine** (lines 422-1669): Complete game logic in an IIFE (Immediately Invoked Function Expression)

---

## Game Mechanics

### Core Systems

#### 1. **Game State & Configuration** (lines 463-519)
- **Storage Keys**: Best score, mute state, difficulty, reduced motion preference
- **Difficulties**: Three presets (Easy/Normal/Hard) with distinct spawn rates, enemy speed, and boss HP multipliers
- **Themes**: Six visual themes with different sky gradients and color schemes
- **Enemy Types**: Grunt, Interceptor, Tank (with varying HP and behavior)
- **Bosses**: Six unique boss types (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)

#### 2. **Player System** (lines 875-911)
- Position, movement speed (260 px/sec), weapon system
- Two weapons: Vulcan (rapid-fire) and Laser (powerful, slower)
- Weapon levels (1-5) that modify fire rate and damage
- Lives system (3 by default, 4 on Easy)
- Invulnerability timer after taking damage
- Cooldown tracking for weapon fire rate

#### 3. **Combat System**
- **Player Bullets** (lines 1160-1165): Tracked separately by weapon type
  - Vulcan: Lower damage, high fire rate, small projectiles
  - Laser: Higher damage, slower fire rate, larger projectiles
- **Enemy Bullets** (lines 1191-1192, 1219-1223): Tracked and updated each frame
  - Regular enemies spawn 3-bullet spreads
  - Interceptors fire homing-style bullets
  - Tank enemies fire tracking bullets
- **Collision Detection** (lines 1233-1256): Pixel-perfect collision for game entities

#### 4. **Enemy System** (lines 1022-1044)
- **Spawning**: Difficulty-scaled spawn timers that decrease as stage progresses
- **Wave Progression**: 18 kills per stage triggers stage transition and boss spawn
- **Enemy Behavior**:
  - Basic movement downward with difficulty-scaled speed
  - Weaving pattern for Interceptors
  - Random shooting based on difficulty-adjusted `enemyShootChance`
- **Enemy Destruction**: Triggers score award, particle effects, power-up drops

#### 5. **Boss System** (lines 1043-1054, 970-1022)
- **Boss Encounter**: Triggered after stage completion (18 kills)
- **Boss Design**: Visual representation using canvas paths (hull, wings, core)
- **Boss Behavior**:
  - Horizontal movement with boundaries
  - Projectile attack patterns with telegraphing
  - Telegraph phase (white flash) before heavy attacks
  - HP displayed in HUD bar
- **Boss Defeat**: Stage advancement, score bonus, next boss queued

#### 6. **Power-Up System** (lines 1254-1256)
- **Types**: Weapon upgrade, health restore, bomb restore
- **Mechanics**: Spawned when enemies destroyed, collected on collision
- **Effects**: Instant weapon level increase, life restore, or bomb restore

#### 7. **Bomb System** (lines 913-1119, 1260-1261)
- **Smart Bomb**: Area-effect attack clearing all enemies and bullets
- **Mechanics**: Activated via fire button while weapon is bomb, limited quantity
- **Visual**: Flash and vignette effects during detonation
- **Cooldown**: 0.5 second cooldown between uses

#### 8. **Scoring & Progression** (lines 1256-1258, 1264-1269)
- **Base Score**: 10-35 points per enemy depending on type
- **Multiplier Chain**: Consecutive hits increase multiplier (x1 to x5+)
- **Chain Decay**: 5-second timer resets chain on miss
- **Best Score**: Persisted to localStorage
- **Stage Progression**: 6 stages minimum, unlimited boss cycles

---

## Rendering System

### Canvas Rendering (lines 1275-1330)
- **Order of Operations**:
  1. Clear canvas and draw background gradient
  2. Draw particles, enemy bullets, power-ups
  3. Draw enemies (with animation frame)
  4. Draw player and player bullets
  5. Draw boss (if present)
  6. Apply effects (vignette, flash overlay)

### Drawing Functions
- **drawPlane()** (lines 913-968): Draws player/enemy fighters with wings, fuselage, windows, and flame effects
- **drawBossShip()** (lines 970-1022): Renders boss with hull gradient, wings, and animated core
- **Particle System**: Simple visual effects for explosions, impacts

### HUD Rendering (lines 678-727)
- **Score Display**: Real-time update of current score
- **Weapon Info**: Current weapon name and level
- **Multiplier**: x1 to x5+ chain multiplier display
- **Hearts**: Visual life indicator with critical state animation
- **Bombs**: Visual bomb count indicator
- **Boss Bar**: Enemy HP bar when boss present
- **Combo Pop**: Floating text showing multiplier milestones

---

## Input Handling

### Keyboard Controls (lines 780-820)
- **Arrow Keys**: Move player
- **Space**: Fire weapon
- **M**: Toggle mute
- **R**: Toggle reduced motion effects
- **P**: Pause game
- **Escape**: Return to menu

### Mouse/Touch Controls (lines 847-862, 827-846)
- **Mouse**: Free movement tracking, click-to-fire
- **Touch**: Virtual joystick + on-screen fire/bomb buttons
- **Joystick**: Left 50% of screen for movement, right 50% for aiming

---

## Audio System

### Sound Effects
- Background music during gameplay
- Fire sounds (weapon shooting)
- Hit feedback (enemy destroyed)
- Bomb detonation sound
- Boss defeat music

### Audio Control
- **Mute Button**: Toggle all audio (muted state persisted)
- **autoplay Policy**: Audio only starts on first user input (`ensureAudio()`)
- **Audio Elements**: Embedded as data URIs (base64 encoded)

---

## Performance Optimization

### Adaptive Quality System (lines 473-483)
- **Three Quality Levels**: Full (0), Medium (1), Low (2)
- **Frame Time Tracking**: Monitors last 45 frames for performance
- **Dynamic Adjustment**:
  - If avg frame time > 1/40s: reduce quality
  - If avg frame time < 1/56s: increase quality
- **Quality Impact**: Reduces particle count, effect intensity at lower quality levels

### Garbage Collection Optimization
- **Array Filtering**: Removes off-screen bullets and particles each frame
- **Object Reuse**: Particles created fresh but arrays managed efficiently
- **No Closures in Hot Path**: Critical functions avoid closure overhead

---

## Accessibility Features

### Screen Reader Support (lines 468-471)
- **ARIA Live Region**: `#srLive` announces game state changes
- **Announcements**: Stage transitions, boss encounters, game over states
- **Button Labels**: All control buttons have `title` attributes

### Visual Accessibility
- **Reduced Motion Mode**: Disables animations when `prefers-reduced-motion` or user preference set
- **Focus Management**: Keyboard navigation properly focused on interactive elements
- **Color Contrast**: HUD text uses drop shadows for legibility

### Touch Accessibility
- **Touch Controls**: Virtual joystick and on-screen buttons for mobile devices
- **Safe Area**: Uses CSS `env(safe-area-inset-*)` for notch-aware layout
- **Viewport Configuration**: Prevents zoom, disables tap highlight for better UX

---

## Mobile & Responsive Design

### Responsive Canvas (lines 100-130)
- **Scaling**: Canvas scales to fit viewport while maintaining aspect ratio
- **Touch Zone Definition** (lines 779-846):
  - Left 50%: Movement joystick area
  - Right 50%: Fire button
  - Joystick provides visual feedback with stick animation
- **Safe Area Insets**: Respects notches and system UI on mobile devices

### Device Detection (line 777)
- **Touch Device Check**: Disables mouse cursor and enables touch controls on touch-capable devices
- **Responsive Font Sizing**: Uses `clamp()` for HUD text that scales with viewport

---

## Storage & Persistence

### LocalStorage Keys (lines 463-466)
- `skyfighter.bestScore`: Best score achieved
- `skyfighter.muted`: Mute preference (0 or 1)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Motion preference (0 or 1)

---

## Game Loop

### Update Cycle (lines 1148-1270)
1. **Input Processing**: Check keyboard/mouse/touch for player movement and firing
2. **Player Update**: Movement with canvas boundary collision
3. **Bullet Update**: Move all bullets, check collisions
4. **Enemy Spawn**: Create new enemies on spawn timer
5. **Enemy Update**: Move enemies, handle AI shooting
6. **Boss Update**: Move boss, handle telegraph and attack patterns
7. **Particle Update**: Animate particles, clean up expired ones
8. **Collision Detection**: Resolve all collisions (player-bullets-enemies-bombs)
9. **Chain Update**: Decay multiplier chain if timer expires
10. **Render**: Draw entire scene to canvas

### Frame Timing (lines 1643-1668)
- **requestAnimationFrame**: Native browser timing (typically 60 FPS)
- **Delta Time (dt)**: Time since last frame for frame-rate independent movement
- **Performance Tracking**: Adaptive quality adjustment based on frame times

---

## Menu System (lines 391-420)
- **Start Screen**: Shows difficulty selection buttons
- **Game Over Screen**: Displays final stats (score, best, stage)
- **New Best Indicator**: Congratulates player on new high score
- **Difficulty Selection**: Easy (4 lives, slower enemies) → Normal → Hard (more enemies, faster)

---

## Key Game Constants

```javascript
KILLS_PER_STAGE = 18          // Enemies to defeat before boss
STORAGE_BEST = 'skyfighter.bestScore'
STORAGE_MUTE = 'skyfighter.muted'
STORAGE_DIFF = 'skyfighter.difficulty'
STORAGE_REDUCED = 'skyfighter.reducedMotion'
```

### Difficulty Parameters
- **Easy**: Spawn base 75, min 40, speed 1.0x, shoot chance 45%, 4 lives, 3 bombs, boss HP 0.8x
- **Normal**: Spawn base 58, min 26, speed 1.4x, shoot chance 70%, 3 lives, 2 bombs, boss HP 1.0x
- **Hard**: Spawn base 40, min 18, speed 1.9x, shoot chance 95%, 3 lives, 2 bombs, boss HP 1.3x

---

## How to Run

1. Open `index.html` in a modern web browser
2. Select difficulty (Easy/Normal/Hard)
3. Click "Start Game"
4. Use Arrow Keys to move, Space to fire
5. Defeat 18 enemies per stage to face boss
6. Progress through stages and defeat all bosses to beat the game

### Browser Requirements
- Canvas 2D API support
- ES6+ JavaScript support
- LocalStorage for persistence
- Web Audio API for sound (optional)

---

## Future Enhancement Ideas

- Multiple game modes (survival, time attack)
- More boss patterns and attack types
- Sound effect library expansion
- Leaderboard/cloud save support
- Pause state UI improvements
- Visual effect toggles
- Mobile-optimized touch controls
- Difficulty balancing refinements
- Sound effect loading fallback (currently silent on audio failure)

---

## Developer Notes

- **Single File Design**: Everything in one file for easy deployment as static site
- **No External Dependencies**: Pure JavaScript, no frameworks or libraries
- **Performance Focus**: Adaptive quality, efficient collision detection, minimal allocations in hot path
- **Accessibility Included**: Screen reader support, reduced motion mode, keyboard controls
- **Mobile First**: Touch controls, responsive scaling, safe area awareness
- **Persistent State**: LocalStorage for user preferences and best scores

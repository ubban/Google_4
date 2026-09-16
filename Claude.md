# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The game runs entirely in the browser with no external dependencies, featuring procedurally generated enemies, boss battles, multiple difficulty levels, and adaptive performance optimization.

### Key Features
- **Canvas-based 2D gameplay** with smooth animations
- **Procedural enemy spawning** with difficulty-based scaling
- **Boss encounters** with unique designs and attack patterns
- **Weapon progression system** with 4 weapon types (Vulcan, Pulse, Scatter, Laser)
- **Strategic bomb mechanic** for emergency situations
- **WebAudio synthesis** for all sound effects (no audio files)
- **Touch and keyboard controls** with responsive joystick
- **Difficulty levels**: Easy, Normal, Hard
- **Visual themes**: 6 different sky themes with unique color palettes
- **Adaptive quality** system that reduces visual effects on low-end devices
- **Accessibility features**: Screen reader support, reduced motion preferences

---

## Architecture Overview

### File Structure
The entire game is contained in a single `index.html` file (~1,700 lines) with:
- **HTML section** (lines 1-421): DOM structure, HUD elements, overlay screens
- **CSS section** (lines 10-420): Styling for canvas, HUD, animations, responsive layout
- **JavaScript section** (lines 422-end): Game logic implemented in IIFE scope

### Game Loop
```
RAF (requestAnimationFrame) → [time delta calculation]
  → update(dt)        [Update game state]
  → draw()            [Render canvas]
  → trackPerf(dt)     [Monitor performance, adjust quality]
```

### Core State Management
- **Game state** (running, paused, game over)
- **Player** object: position, health, weapons, bombs
- **Enemies** array: active enemy ships
- **Bullets** array: player and enemy projectiles
- **Boss** object: current boss encounter (null if none)
- **Score, multiplier, stage tracking**

---

## Key Systems

### 1. Player & Movement
- **Player object**: `{ x, y, w, h, hp, maxHp, bombs, weapons }`
- **Keyboard input**: Arrow keys (up/down/left/right) + Space (fire) + B (bomb)
- **Touch controls**: Joystick for movement, fire/bomb buttons on bottom
- **Movement speed**: Base 300px/s, affected by device quality settings
- **Bound to canvas**: Player stays within playable area

**Key functions:**
- `drawPlane(x, y, w, h, color, darkColor, flame)` - Renders player ship with gradient
- `joystickStart/Move/End()` - Touch joystick handling

### 2. Weapons System
Four weapon types with progression:

| Weapon | Behavior | Unlock |
|--------|----------|--------|
| **Vulcan** | Continuous stream, rapid fire | Default |
| **Pulse** | Explosive rounds with splash damage | After 50 kills |
| **Scatter** | Spread pattern, wider coverage | After 150 kills |
| **Laser** | Concentrated beam, high damage | After 400 kills |

Each weapon has:
- Base fire rate (affected by difficulty)
- Damage per bullet
- Spread pattern
- Visual appearance (different colors)

**Key variables:**
- `currentWeapon` (0-3): Active weapon index
- `weaponKills` (per weapon): Kills needed for next unlock
- `fireTimer`: Cooldown between shots

### 3. Enemy System

**Enemy Types:**
```javascript
ENEMY_TYPES = {
  grunt:       { w: 34, h: 34, hp: 1, speedMul: 1,    score: 10 },
  interceptor: { w: 26, h: 26, hp: 1, speedMul: 1.7,  score: 15, weave: true },
  tank:        { w: 46, h: 46, hp: 4, speedMul: 0.55, score: 35 }
}
```

**Spawning:**
- `spawnEnemy()`: Creates enemies based on difficulty settings
- Spawn rate decreases as player survives longer
- Enemy type selection based on current stage
- Enemies spawn at random X positions, enter from top

**Enemy AI:**
- `updateEnemies(dt)`: Movement and shooting logic
- Weaving movement (interceptors only)
- Random shooting with difficulty-scaled chance
- Collision detection with player and bullets

**Key functions:**
- `spawnEnemy()` - Create new enemy
- `updateEnemies(dt)` - Update enemy positions, shooting
- `drawEnemies()` - Render enemy ships

### 4. Boss System

**6 Boss Types** (progressively unlock with stages):
```javascript
BOSSES = [
  { name: 'VANGUARD',    hpMul: 1.0  },
  { name: 'BEHEMOTH',    hpMul: 1.35 },
  { name: 'SPECTER',     hpMul: 0.8  },
  { name: 'DREADNOUGHT', hpMul: 1.55 },
  { name: 'WRAITH',      hpMul: 0.95 },
  { name: 'TITAN',       hpMul: 1.7  }
]
```

**Boss Mechanics:**
- Boss spawns after 18 kills per stage (configurable)
- HP scales by difficulty multiplier
- Multi-phase attack patterns with telegraphing
- Colored hull with glowing core
- Health bar display at top of screen
- Boss destruction advances to next stage

**Key functions:**
- `spawnBoss()` - Initialize boss encounter
- `updateBoss(dt)` - Boss AI, bullet spawning, phase management
- `drawBossShip(b)` - Render boss with gradient and design

### 5. Bullet System
- **Player bullets**: Vary by weapon type, track velocity
- **Enemy bullets**: Slower, spawned from enemy positions
- **Collision detection**:
  - Player bullets vs enemies: Remove both on hit
  - Enemy bullets vs player: Damage player, remove bullet
  - Player hit detection: Circle-based collision
- **Bullets removed** when off-canvas or max bullets exceeded

**Key variables:**
- `playerBullets[]` - Player projectiles
- `enemyBullets[]` - Enemy projectiles
- `MAX_BULLETS` - Performance cap on bullets

### 6. Bomb Mechanic
- **2-3 bombs per game** (difficulty-dependent)
- **Effect**: Clear all bullets + damage all enemies
- **Trigger**: B key or bomb button
- **Visual feedback**: Screen flash, combo display
- **Cooldown**: 30 frame minimum between bombs
- **Audio**: Distinctive explosion sound

**Key functions:**
- `activateBomb()` - Trigger bomb effect
- `renderBombs(count, max)` - Display bomb count in HUD

### 7. Scoring System
- **Base score**: Enemy type value (10-35 points)
- **Multiplier**: 1x to 5x based on consecutive hits without taking damage
- **Multiplier break**: Taking damage resets to 1x
- **Combo display**: Shows multiplier value when achieved
- **Best score tracking**: LocalStorage persistence

**Key variables:**
- `score` - Current game score
- `multiplier` - Current score multiplier (1-5)
- `multiplierTimer` - Resets when hit
- `bestScore` - Saved in localStorage

### 8. Audio System
Entirely WebAudio API-based synthesis, no audio files:

**Sound Types:**
- **Player shoot**: Frequency sweep based on weapon (440-880 Hz)
- **Enemy shoot**: Lower frequency (240-360 Hz)
- **Hit/explosion**: Noise burst with panning
- **Power-up**: Rising tone sequence
- **Bomb**: Major chord + noise

**Key functions:**
- `beep(freq, dur, type, vol, opts)` - Tone synthesis
- `noiseBurst(dur, vol, opts)` - Noise-based sounds
- `panFor(x)` - Calculate stereo pan from X position

**Configuration:**
- Muted state saved to localStorage
- All synthesis respects muted flag
- Volume controlled by master gain (0.9)
- Compressor for dynamic range

### 9. Difficulty System

**Three Difficulty Levels:**
```javascript
DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2 }
}
```

**Affects:**
- Enemy spawn rate (lower = faster spawning)
- Enemy movement speed multiplier
- Enemy shooting probability
- Starting lives and bombs
- Boss HP multiplier
- Player fire rate scaling

---

## Visual Systems

### 1. Themes (Sky Backgrounds)
6 color themes affecting:
- Gradient sky (3 color stops)
- Cloud appearance
- Sun position

**Current theme cycles** with every stage/boss defeat

### 2. Drawing Order (Z-layers)
1. Sky gradient + clouds (background)
2. Enemy bullets (lower layer)
3. Player bullets (below player)
4. Player ship
5. Enemy ships
6. Boss ship
7. UI/HUD (fixed position)
8. Effects (vignette, flash, combo)

### 3. Visual Effects
- **Screen shake**: `screenShake(mag, dur)` - Camera offset
- **Flash**: `flashHit()`, `flashBomb()` - White screen overlay
- **Vignette**: Red pulsing edge when health critical
- **Combo pop**: Text animation showing multiplier
- **Stage banner**: Text animation at stage transitions

**Config:** Effects disabled if `reducedMotion` is true

### 4. Performance Optimization
Adaptive quality system:
```
quality = 0 (full) → 1 (medium) → 2 (low)
- Full: All effects enabled, high particle count
- Medium: Reduced screen shake intensity
- Low: No screen shake/vignette, simplified clouds
```

**Trigger:** If average frame time > 25ms for 45 frames

---

## Input System

### Keyboard
- **Arrow keys or WASD**: Move player
- **Space**: Fire weapon
- **B**: Activate bomb
- **P**: Pause game
- **M**: Mute audio
- **R** or **Enter**: Restart from game over

### Touch
- **Move zone** (left half): Virtual joystick for movement
- **Fire button** (bottom right): Shoot
- **Bomb button** (bottom left): Activate bomb
- **Pause, mute, motion buttons**: Top right

### Joystick
- Visual feedback: Stick moves within 50px radius base
- Normalized input: Clamped to [-1, 1] on each axis

---

## Game Stages & Progression

### Stage System
- **Kills per stage**: 18 (configurable `KILLS_PER_STAGE`)
- **After each stage**: 
  - Theme changes (visual feedback)
  - Enemy types unlock progressively
  - Difficulty increases slightly
- **Boss triggers**: Every stage after first kill
- **Weapon progression**: Unlocks by kill count (not stage)

### Stage-based Features
- **Stage 1**: Grunts only
- **Stage 2+**: Interceptors and tanks appear
- **Boss frequency**: After every 18 kills
- **Visual progression**: Theme cycles through all 6

---

## Game States

### States
1. **MENU** - Initial overlay, difficulty selection
2. **PLAYING** - Active game in progress
3. **PAUSED** - Game loop continues but no update
4. **GAME_OVER** - Final overlay with stats

### State Transitions
```
MENU → PLAYING (start game)
PLAYING → PAUSED (P key or pause button)
PAUSED → PLAYING (resume)
PLAYING → GAME_OVER (player HP = 0)
GAME_OVER → MENU (restart)
```

**State variables:**
- `gameActive` - Boolean, controls update/draw execution
- `overlay` display/hidden - Shows MENU or GAME_OVER screens

---

## Data Persistence

**LocalStorage Keys:**
- `skyfighter.bestScore` - High score
- `skyfighter.muted` - Audio mute state (0/1)
- `skyfighter.difficulty` - Last selected difficulty
- `skyfighter.reducedMotion` - Accessibility preference

---

## Development Guidelines

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` with properties: `w, h, hp, speedMul, color, dark, score, minStage`
2. Update `spawnEnemy()` to include new type based on stage/difficulty
3. If custom AI needed, add branch in `updateEnemies()` enemy.type check
4. Rendering uses `drawPlane()` automatically (or custom draw if needed)

### Adding a New Weapon
1. Add weapon data to weapon lists in `resetGame()` (damage, fire rate, spread)
2. Update `weaponLabel()` to display new weapon name
3. Add firing logic in `update()` weapon switch
4. Add audio synthesis in weapon firing section (beep call)
5. Update weapon unlock thresholds if needed

### Adding a New Boss
1. Add entry to `BOSSES` array with: `name, hullTop, hullBottom, wing, core, hpMul`
2. Boss selection in `spawnBoss()` is sequential by stage
3. Modify `updateBoss()` attack pattern section for custom AI
4. `drawBossShip()` rendering is automatic based on design colors

### Modifying Difficulty
1. Edit `DIFFICULTIES` object constants
2. `spawnEnemy()` uses difficulty settings for enemy spawning
3. Enemy speed applied in `updateEnemies()` loop
4. Boss HP multiplied at spawn time

### Adding Audio Feedback
1. Use `beep(freq, dur, type, vol, opts)` for tones
2. Use `noiseBurst(dur, vol, opts)` for impacts
3. Respect `muted` flag check (automatic in beep/noiseBurst)
4. Use `panFor(x)` to calculate stereo positioning
5. Ensure `ensureAudio()` called before first sound

### Performance Tuning
1. Monitor `quality` variable (0=full, 1=medium, 2=low)
2. Disable expensive effects when `quality > 0`
3. Example: `if (quality < 2) screenShake(mag, dur)`
4. Add performance-sensitive code to `trackPerf()` averaging window

---

## Key Constants & Configuration

### Gameplay Constants
- `BASE_W` / `BASE_H` = 480 x 720 (canvas base resolution)
- `KILLS_PER_STAGE` = 18
- `MAX_BULLETS` = 300 (global bullet pool limit)
- `BOMB_COOLDOWN` = 30 frames
- `MAX_MULTIPLIER` = 5
- `MULTIPLIER_TIMEOUT` = 180 frames (3 seconds)

### Weapon Stats (in resetGame function)
- **Vulcan**: 3 damage, 120 fire rate, single fire
- **Pulse**: 5 damage, 140 fire rate, slight spread
- **Scatter**: 4 damage, 100 fire rate, wide spread
- **Laser**: 8 damage, 180 fire rate, focused beam

### Boss HP Calculation
`baseHP = 80 + stage * 15` × `difficulty.bossHpMul` × `BOSSES[bossIndex].hpMul`

---

## Accessibility Features

### Screen Reader Support
- `srLive` element with `role="status"` for announcements
- `announce(text)` function for key events
- Game state changes announced (pause, resume, etc.)

### Reduced Motion
- Respects `prefers-reduced-motion` media query
- Toggle button in HUD
- Disables: Screen shake, vignette pulse, flash animations
- Saved to localStorage for persistence

### Touch Support
- Virtual joystick for movement
- Separate fire and bomb buttons
- Responsive sizing with viewport units
- Safe area insets for notched devices

---

## Responsive Design

### Resolution Scaling
- Base: 480×720 canvas
- Scales to fit screen maintaining aspect ratio
- Max scale: 1.6× to prevent blur on small screens
- Uses CSS transforms for crisp rendering

### Viewport Handling
- Fullscreen by default (fixed position)
- No scroll bars (overflow: hidden)
- Safe area insets for notched/rounded displays
- Touch-action: none for custom gestures

---

## Browser Compatibility

### Requirements
- ES2015+ (arrow functions, const/let, destructuring)
- Canvas 2D context
- WebAudio API (for sound)
- LocalStorage (for save data)
- RequestAnimationFrame

### Optional
- Touch events (keyboard-only fallback)
- Stereo panning (mono fallback)
- DynamicsCompressor (graceful skip if unavailable)

---

## Common Tasks

### Adjusting Game Difficulty Balance
Edit `DIFFICULTIES` object for spawn rates, speeds, and shooting chances.

### Changing Colors/Theme
Modify `THEMES` array colors (RGB values) and add/remove themes.

### Tuning Weapon Balance
Adjust fire rates, damage, and spread in weapon configuration sections.

### Adding New Sound Effects
Call `beep()` or `noiseBurst()` in appropriate update sections with desired parameters.

### Changing Canvas Size
Update `BASE_W`, `BASE_H`, and scale calculations; regenerate UI layout.

### Performance Profiling
Check `quality` variable and `frameTimes[]` array; adjust visual effects based on quality level.

---

## Future Enhancement Ideas

- **Persistent powerups**: Shields, rapid-fire boosts, score multipliers
- **Stage-specific mechanics**: Weather effects, destructible terrain
- **Co-op multiplayer**: Two-player simultaneous gameplay
- **Achievement system**: Challenges and unlockables
- **Mobile gestures**: Pinch-to-bomb, swipe controls
- **Procedural boss patterns**: Randomized AI per fight
- **Leaderboard**: Cloud-based high score tracking
- **Accessibility**: Voice control, eye-tracking support

---

## Code Statistics

- **Total lines**: ~1,700
- **Game loop**: ~10ms typical per frame at 60 FPS
- **DOM elements**: ~40 (mostly HUD and overlay)
- **Audio contexts**: Lazy-initialized on first interaction
- **Zero external dependencies**: Pure vanilla JavaScript + Canvas

---

## Debugging Tips

1. **Performance issues**: Check `quality` variable; toggle reduced motion
2. **Audio not playing**: Ensure `ensureAudio()` called and audio context resumed
3. **Enemy spawning too fast**: Check `DIFFICULTIES` and `spawnNext` timer
4. **Weapon not firing**: Verify `fireTimer` and `ammo` checks
5. **Touch not working**: Check `joystickId` tracking and `moveZone` bounds
6. **Game state stuck**: Check overlay display state and `gameActive` flag

---

## Attribution
This game was created as a web-based prototype with the following tech stack:
- Pure JavaScript (ES2015+)
- HTML5 Canvas 2D
- WebAudio API
- No external frameworks or libraries

Last updated: 2026-02-23

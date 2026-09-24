# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a **Raiden-style vertical shoot-'em-up** (shmup) game built entirely in a single HTML5 file. The game combines fast-paced arcade-style combat with strategic weapon management, offering multiple difficulty levels and an adaptive difficulty system.

### Core Features
- **Arcade Action**: Fast-paced vertical scrolling combat with smooth controls
- **Weapon System**: Multiple weapon types with power-up progression
- **Smart Bombs**: Strategic bomb mechanics with smart targeting
- **Boss Battles**: 6 unique boss types with distinct patterns and visual designs
- **Difficulty Modes**: Easy, Normal, and Hard with different spawn rates and mechanics
- **Adaptive Performance**: Real-time frame-time monitoring adjusts graphics quality automatically
- **Web Audio Synthesis**: All sound effects generated programmatically (no external audio assets)
- **Touch & Keyboard Support**: Full mobile support with touch controls and joystick simulation
- **Persistent Stats**: Best scores and settings saved to browser localStorage
- **Visual Themes**: 6 different sky themes with distinct color palettes

## Architecture

### Single-File Structure
The entire game is contained in `index.html` (~1670 lines):
- **HTML**: Game container and HUD elements (div-based UI)
- **CSS**: All styling, animations, responsive design, keyframes
- **JavaScript**: Complete game logic, physics, rendering, and audio

### Rendering System
- **Canvas 2D API**: All graphics rendered via canvas.getContext('2d')
- **Adaptive Quality**: 3-level quality system (0=full, 1=medium, 2=low)
- **Frame Time Tracking**: Average FPS monitored over 45-frame windows
- **Dynamic Adjustment**: Quality level adjusts up if FPS > 40, down if FPS > 56

### Audio System
- **WebAudio API**: All sound generated via `AudioContext` oscillators
- **Synthesized Sounds**: 
  - `beep()`: Variable-frequency tones with optional sweeps and filtering
  - `noiseBurst()`: Noise-based sounds for impacts and explosions
- **Spatial Audio**: Panning support based on screen position
- **Master Gain**: Compressor with dynamic range management
- **Mute Toggle**: Audio can be muted without page reload

### Game Loop
- Request Animation Frame-based main loop
- Frame-time capping for consistent gameplay
- Physics updates and collision detection each frame
- Canvas clearing and complete re-render each frame

## Game Systems

### Player (Aircraft)
- **Movement**: Keyboard (arrows/WASD) or mouse positioning
- **Lives System**: Starts with 3 (Easy: 4) lives per game
- **Health**: One hit = one life lost; health bar visual feedback
- **Weapons**: 5 different weapon types with upgrade levels
- **Bombs**: Limited smart bombs (2 per game, Easy: 3)

### Weapons
- **Laser**: Standard weapon (green, rapid fire)
- **Missiles**: Slower but powerful projectiles (blue)
- **Homing**: Seeks nearest enemy target (purple)
- **Spread**: Multiple bullets per shot (orange)
- **Charge**: High-damage penetrating shots (yellow)
- **Weapon Progression**: Levels 1-5 with increasing fire rate/damage
- **Weapon Switching**: Cycle through collected weapons via E key

### Enemy System
Three enemy types with difficulty scaling:

**Grunt** (minStage 1)
- HP: 1
- Speed: Normal
- Score: 10
- Color: Red (#e53935)
- Spawn: All difficulty levels from stage 1

**Interceptor** (minStage 1)
- HP: 1
- Speed: 1.7x (fast)
- Score: 15
- Color: Orange-red (#ff7043)
- Behavior: Weaving movement pattern
- Spawn: All difficulty levels from stage 1

**Tank** (minStage 2)
- HP: 4 (heavily armored)
- Speed: 0.55x (slow)
- Score: 35
- Color: Purple (#5e35b1)
- Spawn: Stage 2 and beyond

### Difficulty System
```
              Easy        Normal      Hard
spawnBase:    75          58          40    (frames between spawns)
spawnMin:     40          26          18    (minimum spawn interval)
enemySpeed:   1.0x        1.4x        1.9x
shootChance:  45%         70%         95%   (enemy fire probability)
Lives:        4           3           3
Bombs:        3           2           2
BossHpMul:    0.8x        1.0x        1.3x
```

### Boss System
6 unique boss types with distinct visual designs and health pools:

| Name | Hull Color | Core Color | HP Multiplier | Notes |
|------|-----------|-----------|---------------|-------|
| VANGUARD | Purple | Pink | 1.0x | Balanced |
| BEHEMOTH | Orange | Yellow | 1.35x | Heavy hitter |
| SPECTER | Teal | Cyan | 0.8x | Fast/weak |
| DREADNOUGHT | Gray | Light gray | 1.55x | Armored |
| WRAITH | Deep purple | Light purple | 0.95x | Elusive |
| TITAN | Gold | Light gold | 1.7x | Strongest |

### Scoring System
- **Base Score**: Enemy kill points (10-35 depending on type)
- **Combo Multiplier**: Increases for consecutive kills (displayed on-screen)
- **Best Score**: Persistent high score in localStorage
- **Stage Progression**: Tracks through multiple stages

### Stage Progression
- **Kills Per Stage**: 18 enemies defeated = stage clear
- **Stage Themes**: Each stage uses a different visual theme
- **Boss Spawning**: After stage clear, player faces stage boss
- **Unlimited Stages**: Game continues indefinitely with difficulty scaling

### Visual Themes
6 rotating sky themes applied per stage:

| Theme | Sky Gradient | Cloud Color | Sun Color |
|-------|-------------|------------|-----------|
| Daylight Front | Blue gradient | White | Warm yellow |
| Sunset Corridor | Purple/orange | Warm pink | Orange |
| Night Approach | Dark blue | Light blue | Cool blue |
| Storm Front | Green gradient | Cyan | Green |
| Arctic Wastes | Light blue | Pale blue | Pale |
| Volcanic Ridge | Red/orange | Warm | Fire orange |

## Controls

### Keyboard & Mouse
- **Move**: Mouse cursor position OR arrow keys / WASD
- **Fire**: Left mouse click OR spacebar
- **Bomb**: X key or Shift
- **Weapon Cycle**: E key
- **Pause**: P key or Esc

### Mobile/Touch
- **Move**: Drag left side of screen (joystick zone)
- **Fire**: Tap FIRE button (right side)
- **Bomb**: Tap BOMB button (right side)
- **Joystick**: Visual feedback in touch mode

### Settings
- **Mute Button**: Top-right audio toggle
- **Motion Toggle**: Reduce animations/shake for accessibility
- **Pause Button**: Pauses/resumes game

## UI Components

### HUD (Heads-Up Display)
- **Score Display**: Current game score
- **Best Score**: High score from localStorage
- **Weapon Name & Level**: Current weapon and upgrade level
- **Multiplier**: Current combo multiplier (displayed as orange)
- **Hearts**: Visual representation of remaining lives
- **Bombs**: Visual representation of available bombs

### Overlays
- **Start Screen**: Title card with difficulty selection and stats
- **Game Over Screen**: Final score and new personal best indicator
- **Boss Bar**: Health bar that appears when boss spawns
- **Combo Pop**: Large animated text showing combo counts
- **Stage Banner**: Announcement when stage starts
- **Pause Screen**: Overlay during pause

### Visual Effects
- **Flash on Hit**: White screen flash when player takes damage
- **Vignette**: Red pulsing border effect during low health
- **Particle Effects**: Explosions and visual feedback

## Performance & Optimization

### Adaptive Quality System
Monitors frame times every 45 frames:
- **Quality 0 (Full)**: Normal rendering with all effects
- **Quality 1 (Medium)**: Reduced particle effects
- **Quality 2 (Low)**: Minimal visual effects for low-end devices

### Quality Thresholds
- Upgrade quality if average FPS drops below 40
- Downgrade quality if consistently above 56 FPS
- Smooth degradation prevents jank on underpowered devices

### Canvas Optimization
- Single canvas context reused
- Offscreen rendering avoided (direct draw approach)
- Adaptive geometry detail based on quality level

## Data Persistence

### localStorage Keys
- `skyfighter.bestScore`: High score (number)
- `skyfighter.muted`: Mute state (0 or 1)
- `skyfighter.difficulty`: Last selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Motion preference (0 or 1)

## Development Patterns

### Game State Management
- Central game object manages all state
- Enemy and projectile arrays for entity management
- Player object maintains health, position, weapons

### Entity Rendering
- All entities drawn from individual update functions
- Canvas cleared completely each frame
- Z-order managed through drawing sequence

### Event Handling
- Keyboard event listeners for control input
- Mouse movement tracking for cursor positioning
- Touch event listeners for mobile control
- Window resize listener for responsive canvas scaling

### Animation & Effects
CSS keyframes handle:
- Combo pop animations
- Stage banner transitions
- Screen flash effects
- Vignette pulsing
- UI element transitions

## Browser Support

- **Desktop**: Chrome, Firefox, Safari, Edge (all modern versions)
- **Mobile**: iOS Safari 12+, Android Chrome
- **Requirements**: 
  - HTML5 Canvas API
  - Web Audio API (optional, falls back to silent)
  - ES6 JavaScript support
  - localStorage support (optional, for persistence)

## Accessibility Features

- **Screen Reader Support**: Live region element for announcements
- **Reduced Motion**: Respects prefers-reduced-motion media query
- **Color Contrast**: WCAG AA compliant HUD text colors
- **Keyboard Navigation**: Full game playable without mouse

## Known Limitations

- Single-file architecture (large file size)
- No external assets required (good for deployment)
- Performance dependent on device GPU capabilities
- Audio is WebAudio only (no mobile HTML5 audio fallback needed)

## Common Tasks

### Adjusting Difficulty Parameters
Edit the `DIFFICULTIES` object (around line 485):
- `spawnBase/Min`: Enemy spawn rate
- `enemySpeed`: Enemy movement multiplier
- `enemyShootChance`: Enemy fire probability
- `bossHpMul`: Boss health multiplier

### Adding New Enemy Types
1. Add entry to `ENEMY_TYPES` object with w, h, hp, speedMul, color, dark, score, minStage
2. Update spawn logic to include new type
3. Add rendering code in enemy draw function

### Adding New Boss Types
1. Add entry to `BOSSES` array with name, colors (hullTop, hullBottom, wing, core), hpMul
2. Boss drawing will automatically use new colors
3. Adjust hpMul for difficulty scaling

### Adding New Visual Themes
1. Add theme object to `THEMES` array with sky gradient, cloud color, sun color, name
2. Theme selection cycles through array per stage
3. Colors automatically applied to sky and background

### Modifying Audio
- `beep(freq, dur, type, vol, opts)`: Tone generation
  - `freq`: Frequency in Hz (e.g., 440)
  - `dur`: Duration in seconds
  - `type`: 'sine', 'square', 'sawtooth', 'triangle'
  - `opts.sweepTo`: Frequency sweep target
  - `opts.filterFreq/Type`: Optional filter
  - `opts.pan`: Stereo panning (-1 to 1)
- `noiseBurst(dur, vol, opts)`: Noise-based sounds

## Future Enhancement Ideas

- Boss unique attack patterns per type
- Power-up variety (shield, health restore, invincibility)
- Progressive difficulty curve
- Multiple lives/continues
- High score leaderboard (cloud sync)
- Game save/resume (checkpoint system)
- Sound effects library expansion
- Particle effect system
- Screen shake intensity customization
- Boss special abilities

## Git Branching

**Development branch**: `claude/inspiring-planck-wpinqk`

All changes should be committed to this branch with clear commit messages describing modifications.

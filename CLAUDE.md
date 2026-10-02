# Sky Fighter — Codebase Documentation

## Project Overview

**Sky Fighter** is a browser-based vertical shoot-'em-up game (Raiden-style) featuring:
- Player-controlled fighter jet in a vertical scrolling environment
- Dynamic weapon switching and power-ups
- Smart bomb mechanics for clearing enemies
- Multi-stage progression with boss encounters
- Combo scoring system with multipliers
- Responsive touch and keyboard controls
- Mobile-optimized UI with safe-area support

## Repository Structure

The project is contained in a **single HTML file** (`index.html`) with:
- Embedded CSS (styling and animations)
- Embedded JavaScript (game engine and mechanics)
- Inline SVG favicon

### File Listing
```
.
├── index.html          # Complete game (CSS + JS + HTML)
├── README.md           # Project timestamp and basic info
└── CLAUDE.md           # This documentation file
```

## Architecture & Core Components

### Game Engine
The game runs on a **single canvas-based game loop** with:
- **Game State Management**: Tracks player, enemies, projectiles, power-ups, stage progress
- **Input Handling**: Keyboard (WASD/Arrow keys) and touch controls (absolute and zone-based)
- **Physics**: Simple 2D movement with collision detection
- **Rendering**: Canvas-based sprite drawing and effects

### Key Game Classes/Objects

#### Player (`player` object)
- Position, velocity, health (lives), bombs
- Weapon system (multiple weapon types with different behaviors)
- Movement constraints (bounds checking, screen-relative)
- Hit detection and invulnerability frames

#### Enemies (`enemies` array)
- Spawned based on stage/wave progression
- Different enemy types with unique behaviors:
  - Basic fighters (linear movement)
  - Bombers (drop ordnance)
  - Formations (coordinated movement)
- Health points and explosion on death
- Collision with player and projectiles

#### Projectiles (`projectiles` array)
- Player bullets (multiple types based on weapon)
- Enemy fire (various patterns)
- Collision detection with player and enemies
- Despawn when off-screen or on impact

#### Power-ups (`pickups` array)
- Weapon upgrade drops from defeated enemies
- Life/health restoration
- Bomb ammunition
- Score multiplier bonuses

#### Boss System (`boss` object)
- Special high-HP enemies at stage end
- Unique attack patterns
- Boss health bar UI display
- Victory triggers stage progression

#### Scoring & Multiplier System
- Base score per enemy kill
- Combo counter (increases with consecutive kills)
- Multiplier bonus (up to 4x at high combo counts)
- Stage-based score scaling

### UI/HUD System

#### In-Game HUD (`#hud`)
- Player lives/health display (heart icons)
- Current bomb count
- Active weapon type and level
- Combo multiplier indicator

#### Overlays & Screens
- **Start Screen** (`#overlay`): Difficulty selection, instructions, high score display
- **Game Over Screen**: Final score, personal best indicator, restart prompt
- **Boss Bar** (`#bossBarWrap`): Boss name and health visualization
- **Stage Banner**: "STAGE X" announcement on level start
- **Combo Pop**: Large text showing combo multiplier when triggered

#### Visual Effects
- **Flash Effects**: White screen flashes on player hit or bomb activation
- **Vignette**: Red pulsing edge effect during low-health critical state
- **Animations**: Smooth transitions, key animations:
  - Heart beat (critical state)
  - Combo pulse (multiplier popup)
  - Stage banner wipe
  - Card-in animations (UI)

### Input System

#### Keyboard Controls
- `W` / `↑`: Move up
- `S` / `↓`: Move down
- `A` / `←`: Move left
- `D` / `→`: Move right
- `Space`: Fire weapon
- `E` / `B`: Drop bomb
- `R`: Select weapon mode/cycle weapons

#### Touch Controls
- **Move Zone** (left 40% of screen): Direct position mapping to finger location
- **Fire Zone** (right 40% of screen): Rapid-fire weapon when touched
- **Bomb Button**: Fixed button in UI for smart bomb activation

#### Difficulty Settings
- **Easy**: Reduced enemy count, slower fire rate, increased health
- **Normal**: Balanced difficulty (recommended)
- **Hard**: Increased enemy density, faster patterns, reduced lives

## Game Mechanics

### Weapon System
Multiple weapon types with progression:
- **Standard Gun**: Single-shot, rapid fire
- **Spread Gun**: Multi-directional fire
- **Laser**: High damage, narrow beam
- **Homing**: Auto-targeting projectiles
- Weapon upgrades drop from defeated enemies
- Cycling between equipped weapons (configurable key)

### Bomb Mechanics
- **Smart Bomb**: Clears screen of enemies and projectiles
- Limited ammo (3-5 per life depending on difficulty)
- Drops from defeated enemies or on-screen pickups
- Visual feedback: White flash + screen vignette

### Health & Lives
- Player starts with 3 lives (difficulty-dependent)
- One hit reduces health; collision triggers invulnerability frames
- Extra lives awarded at score milestones
- Game over when all lives exhausted

### Stage Progression
- 5 stages total (may expand)
- Progressive difficulty: enemy count, speed, complexity
- Boss encounter at end of each stage
- Difficulty carries across stages

### Combo & Multiplier System
- Combo count increments for each enemy killed
- Resets on player damage or inactivity (15-frame threshold)
- Multiplier bonus: 1x base → 4x at 20+ combo
- Displayed on-screen during active combo

## Controls & Settings

### Configuration (Hard-coded)
- Difficulty selection: Easy, Normal, Hard
- Control scheme auto-detection (touch vs. keyboard)
- Screen bounds: Dynamic based on canvas size
- Physics constants: Speeds, acceleration, collision radii

### Persistence
- **High Score**: Stored in browser `localStorage`
- **Volume/Mute**: Toggle button in top-right corner (audio muting)

## Visual Design

### Color Palette
- **Dark Theme**: Deep navy/black backgrounds (#05122b, #0b1d3a)
- **Accent Colors**: 
  - Gold/yellow (#ffd54f) for UI highlights
  - Cyan (#64b5f6) for primary UI elements
  - Orange (#ff8a65) for multiplier displays
  - Lime green (#7CFC00) for weapon indicators

### Responsive Design
- **Canvas Scaling**: Maintains aspect ratio, fills available viewport
- **Safe Area Support**: Accommodates notches/rounded corners (iOS)
- **Font Scaling**: Responsive font sizes using `clamp()` for readability
- **Touch-Friendly**: 36px minimum button size, 44px safe interaction zones

### Animations & Effects
- **Smooth Transitions**: 0.15-0.3s eases for UI changes
- **Particle Effects**: Explosions on enemy death (canvas-rendered)
- **Glow Effects**: Text shadows with blur for visual depth
- **Keyframe Animations**: 
  - Pulse effects (health critical, stage banner)
  - Bobbing (icon animations)
  - Flash sequences (hit/bomb feedback)

## Performance Considerations

### Optimization Techniques
- **Object Pooling**: Projectiles and explosions reused where possible
- **Dirty Rect Rendering**: Canvas clears and redraws only active elements
- **Collision Detection**: Spatial partitioning (grid-based pruning)
- **Animation Throttling**: RequestAnimationFrame for 60 FPS target

### Resource Usage
- **Single Canvas**: No off-screen rendering or intermediate buffers
- **Image-Free**: Vector-based rendering and CSS backgrounds
- **Minimal Plugins**: Pure vanilla JavaScript, no external libraries
- **Local Storage**: High score persistence only (minimal footprint)

## Development Notes

### Game Loop Structure
```
1. Input polling (keyboard + touch)
2. Update game state (movement, collision, AI)
3. Spawn logic (enemies, power-ups, effects)
4. Render frame (canvas draw)
5. Frame rate control (requestAnimationFrame)
```

### Adding New Content

#### New Enemy Types
1. Define spawn pattern and behavior in enemy creation logic
2. Add AI update function
3. Set health/damage/score values
4. Adjust stage wave definitions

#### New Weapon Types
1. Create projectile pattern function
2. Define fire rate and damage
3. Add visual representation
4. Integrate into weapon cycle system

#### New Stages
1. Define enemy wave sequence
2. Create boss encounter
3. Set difficulty scaling
4. Add stage banner text

### Known Limitations
- Single-player only (no multiplayer/network)
- Fixed 16:9 aspect ratio optimal (but scales responsively)
- Browser-based (requires JavaScript ES6+ support)
- Audio implemented as toggle only (no sound effects/music)

## Testing & Deployment

### Browser Compatibility
- Modern browsers with ES6 support (Chrome, Firefox, Safari, Edge)
- Touch support for mobile (iOS, Android)
- Safe-area support for notched devices

### Deployment
- Static file hosting (no server-side code required)
- Simply serve `index.html` over HTTP/HTTPS
- No build process or dependencies
- Works offline (all assets inline)

### Quality Assurance
- **Game Balance**: Difficulty progression, enemy wave pacing
- **Input Responsiveness**: Keyboard and touch latency
- **Visual Polish**: Animation smoothness, effect timing
- **Edge Cases**: Screen boundaries, score overflow, lives system

## Future Enhancement Ideas

- **Sound Design**: Add background music and sound effects
- **Additional Stages**: Expand beyond 5 stages with unique themes
- **Leaderboard**: Online high score tracking
- **Power-up Variety**: More upgrade types and effects
- **Enemy Variety**: Unique boss mechanics, special enemy patterns
- **Mobile UI**: Swipe gestures, tilt controls
- **Accessibility**: Colorblind modes, adjustable difficulty scaling
- **Replay System**: Record and playback game sessions

## Attribution & Credits

- **Project**: Sky Fighter (Vertical Shoot-'em-Up)
- **Technology**: Vanilla HTML5 Canvas + JavaScript
- **Genre Inspiration**: Raiden, Touhou, Strikers 1945
- **UI Framework**: CSS Grid/Flexbox with custom animations

## How to Play

1. **Open** `index.html` in a modern web browser
2. **Select Difficulty**: Choose Easy, Normal, or Hard
3. **Move**: Use arrow keys/WASD (or touch left zone)
4. **Fire**: Hold space (or touch right zone)
5. **Bomb**: Press E/B (or tap bomb button)
6. **Survive**: Avoid enemy fire and defeat stage bosses
7. **Progress**: Advance through 5 stages for the high score!

---

**Last Updated**: 2026-10-02  
**Game State**: Fully playable vertical shoot-'em-up with 5 stages, bosses, and scoring system

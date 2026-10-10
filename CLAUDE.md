# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game written as a single-file HTML5 application. The entire game (HTML, CSS, and JavaScript) is contained in `index.html` (~59KB).

### Key Characteristics
- **No build process required** — open `index.html` directly in any modern browser
- **No external dependencies** — uses only Canvas API and WebAudio API
- **Fully responsive** — adapts to desktop, tablet, and mobile screens
- **Self-contained** — all assets (graphics, sound synthesis) are procedurally generated

## Quick Start

To run the game:
1. Open `index.html` in a web browser
2. Select difficulty (Easy/Normal/Hard)
3. Click "Start Game"

Supported input methods:
- **Desktop**: Mouse (move to steer, click to fire), Keyboard (Arrows/WASD move, Space fire, X/Shift bomb, P/Esc pause)
- **Mobile**: Drag left zone to move, tap FIRE and BOMB buttons

No server or build step needed.

## Code Architecture

The entire game runs in a single `<script>` block using IIFE (Immediately Invoked Function Expression) for scope isolation.

### Main Game Loop
```
requestAnimationFrame(loop) → update(dt) → draw()
```
- `loop()`: Frame synchronization, calls update and draw
- `update(dt)`: Physics, collision detection, game state changes
- `draw()`: Canvas rendering

### Key Game Objects

**Player** (`player`)
- Position, size, movement speed
- Weapon (vulcan or laser), weapon level
- Lives, invulnerability timer
- Firing cooldown

**Enemies** (`enemies` array)
- Type-based (grunt, interceptor, tank)
- Health points, movement speed, shoot patterns
- Spread or aimed fire behavior
- Weaving movement (for interceptors)

**Bosses** (`boss` object)
- One boss per stage, multi-phase combat
- Health, entering animation, telegraph system
- Pattern-based attack generation
- Unique visual design per boss

**Bullets** (`bullets` array - player fire)
- Vulcan: spread shot with multiple trajectories
- Laser: piercing shots that go through enemies
- Collision detection with enemies and bosses

**Enemy Bullets** (`enemyBullets` array)
- Spread patterns (5-shot, radial bursts)
- Aimed patterns (homing toward player)
- Radial patterns (8-way cross, double rings, spirals)

**Power-ups** (`powerUps` array)
- Life: restore health
- Bomb: add smart bomb
- Weapon: upgrade weapon level or switch type

**Particles** (`particles` array)
- Explosion effects, simple fade-out
- Quality setting affects particle count

### Game State

Core variables:
- `stage`: Current stage (1-6)
- `score`, `multiplier`, `chain`: Combo system
- `lives`: Player health (0-3 or 0-4 on Easy)
- `bombCount`, `maxBombs`: Smart bomb resource
- `running`, `paused`, `gameOver`: Game state flags

Difficulty scaling:
- `DIFFICULTIES` object: spawn rates, enemy speed, health multipliers, starting lives/bombs
- Stage progression: 18 kills per stage triggers boss warning, then boss spawns

### Key Systems

**Difficulty System** (`DIFFICULTIES`)
- Easy: slower spawns, weaker enemies, 4 lives, 3 bombs
- Normal: moderate balance, 3 lives, 2 bombs
- Hard: fast spawns, strong enemies, 3 lives, 2 bombs

**Enemy Types** (`ENEMY_TYPES`)
- **Grunt**: basic slow enemy (score: 10)
- **Interceptor**: fast weaving enemy (score: 15, appears stage 1+)
- **Tank**: high-HP slow enemy (score: 35, appears stage 2+)

**Bosses** (`BOSSES`)
- 6 unique boss designs cycling through stages
- Each has unique color scheme, health multiplier, and 3 attack patterns
- Bosses defined in `BOSS_PATTERNS` array (pattern per boss type)

**Weapon System**
- **Vulcan**: spread increases with level (1→2→3 spread paths)
- **Laser**: lane count increases with level (1→2→2+center paths)
- Weapon switch and level reset on weapon type change

**Audio Synthesis** (`sfx` object, WebAudio API)
- No audio files needed — all sounds generated procedurally
- Frequency sweeps, noise bursts, oscillator types
- Spatial panning based on on-screen position
- Mute toggle with localStorage persistence

**Visual Effects**
- Screen shake: magnitude and duration parametrized
- Flash overlays: white flash on hit, larger on bomb
- Vignette: red pulsing frame on low health (1 life critical)
- Particles: explosion effects scale with quality setting
- Theme system: 6 sky/cloud/sun color schemes rotating per stage

**Quality Adaptation** (`quality` variable)
- Monitors frame time, adjusts quality (0=full, 1=medium, 2=low)
- Reduces particle count, shadow effects, glow at lower quality

**Responsive Canvas**
- Base resolution: 480×720 (portrait)
- Scales up to 1.6x if screen larger
- Safe area insets for notched devices

### Collision Detection
- Axis-aligned bounding box (AABB): `rectsOverlap(a, b)`
- Used for: bullet-enemy, bullet-boss, enemy-player, bullet-player, powerup-player

### Score System
- Base score per enemy type
- Score × multiplier (1-10x based on kill chain)
- Chain breaks if 2.2 seconds elapse without a kill
- Boss defeated = 500 points

## Important Constants and Configuration

```javascript
// Canvas
BASE_W = 480, BASE_H = 720

// Game balance
KILLS_PER_STAGE = 18  // enemy kills needed before boss spawns

// Difficulty parameters (spawn rates, speeds, lives/bombs)
DIFFICULTIES.easy/normal/hard

// Enemy types with HP, speed, colors, score values
ENEMY_TYPES.grunt/interceptor/tank

// Boss designs (6 bosses with color schemes and HP multipliers)
BOSSES[]

// Boss attack patterns (3 patterns per boss type)
BOSS_PATTERNS[][]

// Visual themes (6 sky color schemes)
THEMES[]

// Storage keys for persistence
STORAGE_BEST, STORAGE_MUTE, STORAGE_DIFF, STORAGE_REDUCED
```

## Common Modifications

### Change Enemy Difficulty
Modify `DIFFICULTIES` object — adjust `spawnBase`, `enemySpeed`, `enemyShootChance`, `bossHpMul`

### Add New Boss
1. Add entry to `BOSSES` array with color scheme and `hpMul`
2. Add 3 attack patterns to `BOSS_PATTERNS` array (functions taking boss object)
3. Use helper functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.

### Adjust Weapon Balance
- Vulcan spread angles: modify `spread` array in `fireWeapon()`
- Laser pierce count: modify `pierce` values
- Firing rate: adjust `fireDelay` constant

### Change Visual Theme
- Add new theme to `THEMES` array: `{ sky: [color1, color2, color3], cloud, sun, name }`
- Themes cycle per stage: `(stage - 1) % THEMES.length`

### Modify Particle Effects
- Count: controlled by `quality` setting and explosion `count` parameter
- Duration: `maxLife` property (default 0.5s)
- Dispersion: `vx`/`vy` random range (default ±200)

## File Structure

```
Google_4/
├── index.html          # Entire game (HTML + CSS + JS)
├── README.md           # Project metadata
└── CLAUDE.md           # This file
```

No subdirectories or build artifacts — the repository contains only the static HTML file and documentation.

## Testing and Debugging

Browser console can be used to access game state:
- Pausing: P key or ESC or pause button
- Muting: mute button in top-right
- Motion effects: toggle button in top-right
- Difficulty change: only on title screen

To test specific scenarios:
- Open browser DevTools (F12)
- Console commands access variables like `player`, `enemies`, `boss`, `score`
- Set `running = false` and `gameOver = true` to reach end screen
- Modify `lives`, `bombCount`, or `score` directly

No unit tests — game is single-file and tested manually in browser.

## Performance Notes

- Adaptive quality system monitors frame time and adjusts rendering
- Canvas size locked at 480×720 internally, CSS scale handles display size
- Particle count scales with quality setting for low-end devices
- Shadow effects and glow disabled on lower quality levels
- Enemy spawning decreases frame time at higher stages (more enemies on screen)

## Web API Dependencies

- **Canvas 2D Context**: all rendering
- **WebAudio API**: procedural sound synthesis, no audio files needed
- **LocalStorage**: persistence of best score and settings
- **RequestAnimationFrame**: 60 FPS game loop
- **Touch Events**: mobile input (touchstart, touchmove, touchend)

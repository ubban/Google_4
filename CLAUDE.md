# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML file with embedded CSS and JavaScript. The game features:
- Canvas-based 2D rendering with dynamic backgrounds
- Multiple difficulty levels (Easy, Normal, Hard)
- Progressive stage system with enemy variety and boss fights
- Weapon upgrade system with 6 distinct firing patterns
- Combo multiplier mechanics
- Touch and keyboard controls with joystick support
- Web Audio API for sound effects and music
- Adaptive performance scaling
- Progressive score tracking with local storage

## How to Run

The game requires no build process or external dependencies:

1. **Direct browser**: Open `index.html` in any modern web browser
2. **Local server** (recommended for reliable audio):
   ```bash
   # Using Python 3
   python -m http.server 8000
   # Using Python 2
   python -m SimpleHTTPServer 8000
   # Using Node.js http-server
   npx http-server
   ```
   Then navigate to `http://localhost:8000`

The game automatically detects:
- Screen size and adjusts canvas resolution
- Device capabilities (touch vs mouse, reduced motion preferences)
- Frame rate performance and scales rendering quality accordingly

## Game Architecture

### Core Systems

**1. Entity System**
- **Player**: Single aircraft (plane) with health, bombs, and active weapon
- **Enemies**: Spawned in waves with difficulty-scaled behavior
- **Bosses**: Stage-final encounters with multiple weapon patterns
- **Projectiles**: Player bullets and enemy shots with collision detection
- **Particles**: Visual effects (explosions, smoke) with short lifespans
- **Power-ups**: Drop from defeated enemies, upgrade weapons or grant bombs

**2. Weapon System** (6 upgradeable patterns)
- **Straight**: Single vertical stream
- **Spread**: 3-way horizontal fan
- **Columns**: Multiple vertical columns
- **Spiral**: Rotating burst pattern
- **Radial**: Full-circle explosion (boss weapon)
- **Aimed**: Homing shots targeting enemies

Each weapon upgrades from Level 1→3, increasing damage and fire rate.

**3. Difficulty System**
Affects spawn rates, enemy speed, shoot chance, health pools, and lives:
- **Easy**: Longer spawn intervals (75-40), slower enemies (1.0x), fewer shots, 4 lives
- **Normal**: Medium intensity (58-26), 1.4x speed, 3 lives
- **Hard**: Tight spawn (40-18), fast enemies (1.9x), frequent shots, 1.3x boss HP, 3 lives

**4. Stage Progression**
- Fixed kills per stage: 18 enemies complete a stage
- Boss spawns after stage completion
- 6 visual themes cycle through stages
- Score multiplier increases on successful combos

**5. Input Handling**
- **Keyboard**: Arrow keys or WASD for movement, Space to fire, X/Shift for bombs
- **Mouse**: Movement tracks to cursor, click fires
- **Touch**: Joystick on left side (drag to move), FIRE/BOMB buttons on right
- **Pause**: P or Esc key

**6. Audio System**
- Web Audio API with context creation on first user interaction
- Beeps, noise bursts, and volume normalization
- Mutable state persisted to local storage

**7. Performance System**
- Tracks frame times over 45-frame rolling window
- Adaptive quality scaling (0=full, 1=medium, 2=low) to maintain 40-60 fps
- Adjusts enemy count and particle density under load

### Data Flow

1. **Input** → Keyboard/mouse/touch handlers update player state
2. **Update** → Collision detection, entity movement, spawning logic
3. **Render** → Canvas 2D drawing with theme colors and effects
4. **Storage** → Score, settings persisted to `localStorage`

## Key Code Sections

### Game State Variables
```
- gameRunning, gamePaused: State flags
- player: { x, y, w, h, health, maxHealth, bombs, weapon, weaponLvl }
- enemies[], bosses[], bullets[], particles[]: Entity arrays
- score, multiplier, comboCount: Gameplay metrics
- currentTheme, currentStage, killCount: Progression state
```

### Main Game Loop
- `update(dt)`: Called via `requestAnimationFrame`, handles all entity movement and collisions
- `render()`: Canvas drawing after update completes
- Adaptive dt-based physics (frame-rate independent)

### Critical Functions
- `spawnEnemy()`: Creates random enemy with difficulty-scaled behavior
- `spawnBoss()`: Stage-end boss with health bar and pattern attacks
- `fireWeapon()`: Executes current weapon pattern
- `useBomb()`: Clears bullets, damages bosses, applies visual effects
- `applyPowerUp(type)`: Weapon upgrade or bomb grant
- `addScore(base)`: Score calculation with combo multiplier
- `rectsOverlap(a, b)`: Collision detection via AABB

### Visual Effects
- **Screen shake**: From bomb usage and hits
- **Flash overlay**: Flash element for visual feedback
- **Vignette**: Dark edges for immersion
- **Combo pop**: Yellow animated text on screen
- **Stage banner**: Theme name announcement
- **Boss health bar**: Shows during boss encounters

## Development Notes

### Storage Keys
- `skyfighter.bestScore`: High score persistence
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Accessibility preference

### Canvas Sizing
- Dynamically resizes on window resize
- Maximum dimensions: 360×720px (portrait) or 720×360px (landscape)
- Maintains aspect ratio with CSS `max-width/max-height: 100%`

### Theme System
Six hardcoded themes with sky gradient, cloud, and sun colors:
1. Daylight Front (blue sky)
2. Sunset Corridor (purple/orange)
3. Night Approach (dark blue)
4. Storm Front (greenish)
5. Arctic Wastes (icy blue)
6. Volcanic Ridge (red/orange)

Each theme cycles with stage progression.

### Enemy Spawn Algorithm
- Base spawn interval decreases as difficulty increases
- Minimum spawn interval prevents spawning too frequently
- Boss triggers after 18 kills per stage
- Spawn position randomized off-screen top

### Weapon Pattern Logic
Each weapon's `fire*` function (e.g., `fireSpread`, `fireColumns`) calculates bullet positions relative to player position, then adds to bullets array with velocity.

### Browser Compatibility
- Requires: Canvas 2D, Web Audio API, ES6+ (const/let, arrow functions)
- Touch support auto-detected via `touchstart` listener
- Fallback for systems without Web Audio (game works, sounds disabled)

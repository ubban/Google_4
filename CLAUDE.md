# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file Raiden-style vertical shoot-'em-up game built with HTML5 Canvas and WebAudio API. The entire game is contained in `index.html` (~1670 lines).

### Key Features
- Canvas-based 2D rendering
- Procedurally-generated enemy waves and boss encounters
- Multiple difficulty settings (Easy, Normal, Hard)
- Weapon system with multiple firing patterns
- Smart bomb mechanics
- Combo/multiplier scoring system
- Web Audio synth-based sound effects (no external audio files)
- Touch controls with virtual joystick for mobile
- Reduced motion accessibility option
- Local storage for best score and user preferences

## Development Commands

This is a static HTML file game with no build process. Simply open `index.html` in a browser to play.

### Running the Game
```bash
# Open in default browser
open index.html

# Or use a local HTTP server (recommended for better compatibility)
python -m http.server 8000
# Then visit http://localhost:8000/index.html
```

### No Build/Test/Lint Steps
Since this is a single HTML file with inline CSS and JavaScript, there are no separate build, lint, or test commands. However, you can validate with standard web tools:
- Browser DevTools console (F12) for runtime errors
- Web standards validation at https://validator.w3.org/

## Code Architecture

The entire game runs in a self-executing anonymous function (line 423) to avoid polluting the global scope.

### Core Game State

Defined around line 875-910 in `resetGame()`:
- **player**: Position, size, speed, weapon state, invulnerability timer
- **bullets**: Array of player projectiles (all bullets, various weapons)
- **enemies**: Array of active enemies with type-specific data
- **enemyBullets**: Array of incoming fire
- **particles**: Visual effects (explosions, impacts)
- **powerUps**: Collectibles on the field
- **boss**: Current boss encounter object (null when not active)
- **Game tracking**: score, lives, stage, frame count, chain multiplier

### Main Game Loop

**`loop(now)` (line 1640)**: RequestAnimationFrame callback
1. Calculates delta time from last frame
2. Calls `update(dt)` to advance game logic
3. Calls `draw()` to render current state
4. Requests next frame

**`update(dt)` (line 1148)**: Core game logic
- Player movement and firing
- Bullet/enemy collision detection
- Enemy AI and spawning logic
- Power-up collection
- Score and combo tracking
- Lives and game over conditions

**`draw()` (line 1520)**: Canvas rendering
- Clear and draw background (parallax clouds)
- Render all game objects (player, enemies, bullets, particles)
- Draw HUD (score, health, weapon info, boss bar)

### Key Components

**Player System**
- `drawPlane(x, y, w, h, color, darkColor, flame)`: Renders player ship with flame effect
- `fireWeapon()`: Shoots based on current weapon type
- Movement constrained to canvas bounds
- Invulnerability on spawn and after taking damage

**Enemy System**
- **ENEMY_TYPES** (line 502): Three enemy classes (grunt, interceptor, tank)
  - Each has unique stats: hp, speed, attack pattern
  - `spawnEnemy()`: Spawns one enemy with random placement
  - Enemies follow player with tracking AI
  - Different explosion effects per type

**Boss System**
- **BOSSES** array (line 508): Six named boss encounters (VANGUARD through TITAN)
  - `spawnBoss()`: Creates boss after 18 kills
  - `drawBossShip(b)`: Complex multi-part rendering
  - `updateBoss(dt)`: Boss attack patterns change per stage
  - Boss fires in patterns: radial, aimed spread, columns, spirals, etc.
  - `killBoss()`: Triggers stage advancement and next wave

**Weapon System**
- Currently four weapons: vulcan, laser, tesla, missiles
- Each has unique firing pattern and sprite
- Upgraded through weapon levels (via power-ups)
- Current weapon displayed in HUD

**Power-Up System**
- `maybeSpawnPowerUp(x, y)`: Random drop on enemy kill
- Types: weapon upgrades, health, bombs, shield (rapid-fire multiplier)
- Collision-based collection

**Scoring & Multipliers**
- Base enemy score varies by type (10-35 points)
- Kill chain: consecutive kills without taking damage = multiplier
- Score display updates in real-time
- Best score persisted to localStorage

**Audio System**
- No external audio files; all sounds are synthesized
- `beep()`: Oscillator-based tone generation with optional frequency sweep
- `noiseBurst()`: White noise with filter for impact sounds
- Extensive sound effects map: shoot, laser, explosions, hit, bomb, etc.
- Stereo panning based on screen position
- Master compressor for dynamic range control
- Mute toggle stored in localStorage

### Input Handling

**Mouse/Desktop**
- Move: Mouse position (constrained to canvas)
- Fire: Hold mouse button or continuous on canvas

**Touch/Mobile**
- Move: Virtual joystick (bottom-left, dynamic radius)
- Fire: Fire button (bottom-right)
- Joystick shows visual feedback (base and stick)

**UI Controls**
- Difficulty buttons: Set game difficulty pre-game
- Mute button: Toggle audio on/off
- Motion button: Reduce screen shake and flash effects
- Pause button: Pause/resume (in-game only)

### Collision & Rendering

**`rectsOverlap(a, b)`** (line 1069): AABB collision detection
- All collision checks use axis-aligned bounding boxes
- Used for bullets vs enemies, player vs enemies, player vs power-ups

**`drawClouds(speed, fillColor, rx, ry)`** (line 1507): Parallax background
- Procedural cloud rendering at two layers for depth effect

**Visual Effects**
- `spawnExplosion(x, y, color, count)`: Particle effects on impact
- `screenShake(mag, dur)`: Camera shake (disabled in reduced-motion mode)
- `flashHit()` / `flashBomb()`: Screen flash effects
- `showComboPop(text)`: Floating combo text

### Configuration Constants

**DIFFICULTIES** (line 485): Easy/Normal/Hard settings
- Affect enemy spawn rate, player starting lives, bomb count

**THEMES** (line 493): Color palettes for procedural enemy generation

**Game Balancing**
- KILLS_PER_STAGE: 18 (trigger boss spawn)
- Base health/bomb counts per difficulty
- Damage values for different enemies
- Score multipliers and power-up drop rates

### State Persistence

Uses `localStorage` for:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Selected difficulty
- `skyfighter.reducedMotion`: Accessibility preference

## Common Editing Patterns

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502) with stats
2. Create unique explosion sound in `sfx` object if needed (line 624+)
3. Update `spawnEnemy()` spawn logic if needed (line 1022)
4. Add rendering code in `draw()` if unique appearance is needed (line 1520+)

### Adding a New Boss
1. Add to `BOSSES` array with color palette and hp multiplier (line 508)
2. Add attack pattern function: `fireRadial()`, `fireAimedSpread()`, etc.
3. Update boss attack selection in `updateBoss()` (line 1411)

### Adding a New Weapon
1. Define weapon entry in `fireWeapon()` firing patterns (line 1124)
2. Add sound effect in `sfx` object
3. Update `weaponLabel()` for HUD display (line 871)
4. Add rendering logic in player update if needed

### Adjusting Difficulty
Edit `DIFFICULTIES` object (line 485):
- `lives`: Starting health
- `bombs`: Starting bombs
- `spawnRate`: Enemy spawn frequency
- Other multipliers affect enemy behavior

## Performance Notes

- 60 FPS target with adaptive quality adjustment (line 476-487)
- Tracks frame times to lower quality if needed (full → medium → low)
- Lower quality disables screen shake and reduces particle effects
- All calculations are lightweight Canvas 2D operations

## Accessibility Features

- Full keyboard support not currently implemented; mouse/touch only
- Reduced motion mode disables animations (shake, flash, particles)
- Screen reader announcements via `#srLive` (line 390)
- High contrast colors suitable for visibility
- Touch-friendly button sizing on mobile
- `announce()` function for screen reader feedback (line 468)

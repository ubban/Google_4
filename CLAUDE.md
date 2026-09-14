# Sky Fighter Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML application. Players control a fighter plane, switch between weapons, deploy smart bombs, and defeat stage bosses while building combo multipliers.

## Project Structure

- **index.html** (1669 lines) — Complete game application with embedded CSS and JavaScript
  - Styles: Game canvas, HUD display, responsive layout with safe-area insets
  - Game logic: Player controls, enemy AI, collision detection, scoring system
  - Audio system: Procedurally generated sound effects and background music
  - UI: Pause menu, game over screen, settings panel, difficulty selector

- **README.md** — Minimal project metadata
- **.git/** — Version control

## Game Architecture

### Core Game Loop
The main update loop (`update(dt)`) runs at ~60 FPS and handles:
- Player movement and input processing
- Player weapon fire
- Enemy spawning and behavior
- Boss AI and attacks
- Projectile movement and collision detection
- Power-up collection and application
- Score calculation with combo multipliers
- Game state transitions (playing → paused → game over)

### Player Mechanics
- **Movement**: Arrow keys or WASD for directional control, or joystick/touch input
- **Weapons**: Four weapon types (gun, spread, laser, homing) with individual ammo pools
- **Bombs**: Limited "smart bomb" charges that clear enemy projectiles and damage enemies
- **Health**: Heart system with visual feedback on HUD
- **Combo Multiplier**: Increases when destroying enemies in succession; resets on hit

### Enemy System
- **Spawning**: Randomized enemy selection based on current stage, with caps on active count
- **Types**: Multiple enemy variants with different movement patterns and projectile behaviors
- **Bosses**: Stage bosses with unique attack patterns and multi-phase health bars
- **Projectiles**: Different enemy bullet types (standard, spread, homing) with collision detection

### Power-Up System
- **Drop Mechanic**: Random chance when enemies are destroyed
- **Types**: Extra health, bomb charges, weapon ammo, temporary invincibility, weapon upgrades
- **Application**: `applyPowerUp(type)` handles power-up effects and HUD updates

### Difficulty System
- **Levels**: Easy, Normal, Hard with different enemy patterns and boss configurations
- **Scaling**: Enemy spawn frequency, projectile count/speed, and boss health adjusted per difficulty
- **UI**: Difficulty selector in pause menu with visual indicator

## Key Functions and Systems

### Rendering
- `drawPlane(x, y, w, h, color, darkColor, flame)` — Draws player ship with optional engine flame
- `drawBossShip(b)` — Renders boss with health bar and attack patterns
- `renderHearts(count, max)` — HUD health indicator
- `renderBombs(count, max)` — HUD bomb counter
- `spawnExplosion(x, y, color, count)` — Visual particle explosion effect

### Game State
- `resetGame()` — Reinitializes all game variables for new playthrough
- `setDifficulty(name)` — Updates game difficulty and enemy parameters
- `breakChain()` — Resets combo multiplier when hit
- `addScore(base)` — Applies combo multiplier and updates score display

### Input Handling
- `joystickStart(clientX, clientY, id)` — Initializes virtual joystick touch input
- `joystickMove(clientX, clientY)` — Updates joystick position during drag
- `joystickEnd()` — Clears joystick state
- `canvasPointFromEvent(e)` — Converts touch/mouse coordinates to canvas space

### Audio System
- `beep(freq, dur, type, vol, opts)` — Generates tone-based sound effects
- `noiseBurst(dur, vol, opts)` — Creates noise-based explosions and impacts
- `panFor(x)` — Stereo panning based on screen position
- `ensureAudio()` — Initializes Web Audio API context on first interaction

### Collision Detection
- `rectsOverlap(a, b)` — AABB collision check for projectiles and entities
- Hit detection loops within `update()` check player ↔ enemy bullets, bombs ↔ enemies, etc.

## Visual and Audio Design

### Colors and Theming
- **Background**: Dark blue gradient (`#05122b` → `#0d2f5e` → `#1e6fd8`)
- **Accents**: Bright cyan/blue (`#1976d2`, `#7fc1ff`) for UI and planes
- **Theme**: Dark mode with high contrast for readability

### Responsive Design
- CSS uses `clamp()` for fluid scaling across device sizes
- Safe-area insets for notched devices (`env(safe-area-inset-top)`, etc.)
- Touch-optimized with large tap targets and haptic feedback consideration
- Canvas scales to fit viewport while maintaining aspect ratio

### Audio Design
- Procedurally generated using Web Audio API
- Sound effects vary by game event (fire, hit, explosion, power-up)
- Mutable via in-game toggle with visual indicator

## Game Balance and Configuration

### ENEMY_TYPES Object
Defines each enemy variant with:
- `minStage` — Stage requirement to spawn
- `health` — Hitpoints
- `speed` — Movement speed
- `projectile_count`, `projectile_speed` — Weapon configuration
- `width`, `height`, `color` — Visual properties

### BOSS_TEMPLATES Object
Configures boss difficulty curves:
- Health scaling per stage
- Attack pattern intensity (projectile count, frequency)
- Unique movement and behavior per stage

### Difficulty Constants
- `SPAWN_INTERVAL`, `ENEMY_CAP` — Spawn frequency and max active enemies
- `PROJECTILE_SPEED_FACTOR` — Enemy bullet speed multiplier
- `BOSS_HEALTH_FACTOR` — Boss health scaling

## Development Guidelines

### Code Style
- Minimal comments; self-documenting function and variable names
- Single-file architecture: Keep related logic together, organized vertically
- Avoid abstractions until proven necessary; prioritize clarity over reusability

### Adding Features
- **New weapon type**: Add entry to `WEAPON_INFO` object, handle in `fireWeapon()` and weapon UI
- **New enemy**: Define in `ENEMY_TYPES` with sprite/color parameters, auto-included in spawn pool
- **New boss**: Add to `BOSS_TEMPLATES`, implement attack pattern in boss update section
- **New power-up**: Define type constant, add to `maybeSpawnPowerUp()`, handle in `applyPowerUp()`

### Testing
- Use browser DevTools console for breakpoints and state inspection
- Adjust `SPAWN_INTERVAL` and `ENEMY_CAP` for rapid difficulty iteration
- Test on mobile via phone preview or DevTools device emulation

### Performance Considerations
- Collision detection is O(n²) but capped by `ENEMY_CAP` and projectile limits
- Particle explosions are pooled to reduce GC pressure
- Audio context uses a single shared instance to avoid memory bloat

## Known Limitations and Future Improvements

- Single-file architecture limits modularity; consider splitting into separate files for very large expansions
- No persistent high score storage (data persists only during session)
- Boss patterns could be more varied and complex
- Mobile controls could benefit from haptic feedback API
- Sound design is procedural; could integrate pre-recorded audio for richer experience

## Deployment

- Game runs entirely client-side; no server required
- Serve via any HTTP server (live reload recommended for development)
- Responsive design works on desktop, tablet, and mobile
- Works offline once cached (no external dependencies)

---
_Updated by Claude Code on 2026-09-14_

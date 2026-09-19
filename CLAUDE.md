# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in HTML5 with no external dependencies. The entire game logic, rendering, and audio synthesis is contained in a single `index.html` file (~1669 lines).

### Game Features
- Vertical scrolling shooter mechanics
- Three difficulty levels (Easy, Normal, Hard)
- Weapon power-up system (Vulcan, Laser)
- Smart bomb mechanics
- Six unique boss encounters
- Combo/kill chain multiplier system
- Procedural audio (no asset files)
- Touch controls with joystick
- Mouse and keyboard input support
- Persistent score tracking
- Accessibility features (reduced motion, screen reader support)

## Architecture

The game uses a simple procedural architecture with the following core systems:

### Game Loop
- **`loop(now)`** - Main animation frame loop (requestAnimationFrame)
  - Calls `update(dt)` for game logic when running/not paused
  - Calls `draw()` for canvas rendering
  - Handles performance tracking and adaptive quality

### Game State
- **Player object** - `{ x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }`
- **Entity arrays** - `enemies`, `bullets`, `enemyBullets`, `particles`, `powerUps`, `boss`
- **Game variables** - `score`, `lives`, `stage`, `multiplier`, `chain`, `difficulty`, `frame`, `running`, `paused`, `gameOver`

### Core Functions

#### Update & Logic (`update(dt)`)
- Player movement (keyboard, mouse steering, joystick)
- Input handling (fire, bomb)
- Entity movement and lifecycle
- Collision detection (axis-aligned rectangles via `rectsOverlap()`)
- Enemy AI spawning and behavior
- Boss state management
- Power-up effects

#### Rendering (`draw()`)
- Canvas gradient sky background (theme-based)
- Player plane with flame animation
- Enemies and boss
- Projectiles (player bullets, enemy bullets)
- Particles (explosions)
- Power-ups (rotating)
- UI overlays (HUD, pause menu, game over)

#### Input
- **Keyboard** - Arrow keys, WASD for movement; Space for fire; X/Shift for bomb; P/Esc for pause
- **Mouse** - Move to aim/position; click to fire
- **Touch** - Left side joystick for movement; right side Fire/Bomb buttons
- All inputs unified through `keys` object and `joyVec` state

#### Audio
- **WebAudio API** - Procedural synthesis (no audio files)
- Sound effects synthesized with oscillators, filters, and noise buffers
- `beep(freq, dur, type, vol, opts)` - Tone synthesis
- `noiseBurst(dur, vol, opts)` - Noise effects
- `sfx` object with named effects (shoot, laser, explode, hit, bomb, powerup, bossHit, etc.)
- Mute state persisted to localStorage

### Entity Systems

#### Player
- Single player object with position, size, speed, weapon level, cooldown
- Invulnerability frames after taking damage
- Weapon system: Vulcan (spread) and Laser (piercing)
- Rapid fire bonus tracked separately

#### Enemies
- Three types: `grunt` (basic), `interceptor` (fast, weaving), `tank` (slow, durable)
- Each has position, size, HP, speed multiplier, color, and AI behavior
- Weaving enemies use sine wave offset
- Spawned dynamically with difficulty-based spawn rate
- Fire at player with aiming AI
- Despawn when off-screen

#### Boss
- Six unique boss types with distinct visual designs and HP multipliers
- Complex state: `entering` (animation phase), `telegraph` (attack warning), `attack` patterns
- Multiple attack patterns: radial spread, aimed spread, column fire, spiral burst
- Boss bar HUD shows HP
- Despawn triggers victory condition and stage advancement

#### Projectiles
- **Player bullets** - Array of bullets moving upward with optional horizontal velocity (vx)
  - Laser bullets pierce enemies (limited pierce count)
  - Regular bullets disappear on hit
- **Enemy bullets** - Array of projectiles from enemies/boss moving downward
  - Simple circle collision (4px radius)

#### Particles
- Explosion particles spawned on kill/hit
- Position, velocity, life timer, color
- Fade out as they expire

#### Power-ups
- Three types: `life` (extra heart), `bomb` (extra bomb), `weapon` (weapon upgrade)
- Spawn randomly on enemy kills
- Rotate and move downward
- Collected on player collision

### Game Difficulty System

Three difficulty presets (`DIFFICULTIES` object):
- **Easy** - Slower spawn rate, reduced enemy accuracy, more lives/bombs, reduced boss HP
- **Normal** - Baseline difficulty
- **Hard** - Fast spawn rate, high accuracy, boss bonus HP

Dynamic difficulty ramp: enemy spawn rate decreases every 5 seconds during stage.

### Visual Theming

Six visual themes (`THEMES` array):
- Daylight Front
- Sunset Corridor
- Night Approach
- Storm Front
- Arctic Wastes
- Volcanic Ridge

Each theme defines sky gradient colors, cloud color, and sun color.

### Performance & Adaptive Quality

Frame time tracking adjusts rendering quality in three levels:
- **Quality 0** - Full: shadow effects, glow effects on bullets/boss
- **Quality 1** - Medium: reduced shadows
- **Quality 2** - Low: minimal effects

Quality degrades if average frame time > 1/40s, improves if < 1/56s.

## Development Notes

### Code Organization
- **Lines 1-359** - HTML structure, CSS styling, theme definitions
- **Lines 360-422** - HTML body with canvas and UI elements
- **Lines 422-1147** - Initialization, constants, global state
- **Lines 1148-1297** - `update()` function (core game logic)
- **Lines 1298-1639** - Utility and rendering functions
- **Lines 1640-1669** - Main loop and startup

### Key Optimizations
- **Entity pooling**: No; entities created/destroyed dynamically
- **Canvas rendering**: 2D context used directly (no offscreen canvas)
- **Collision**: Simple AABB (axis-aligned bounding box) via `rectsOverlap()`
- **Procedural audio**: Reduces file size; ~3KB vs typical .mp3s

### localStorage Keys
- `skyfighter.bestScore` - High score
- `skyfighter.muted` - Mute state
- `skyfighter.difficulty` - Last selected difficulty
- `skyfighter.reducedMotion` - Reduced motion preference

### Testing the Game

No build step required—open `index.html` directly in a browser. Game starts on page load with the start screen overlay showing difficulty selection.

**To test new features:**
1. Open DevTools console
2. Modify game variables directly (e.g., `score = 1000`, `stage = 5`)
3. Call functions (e.g., `useBomb()`, `applyPowerUp('weapon')`)
4. Reset with `resetGame()`

## Common Modifications

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Update `spawnEnemy()` to choose the new type based on `stage` and `minStage`
3. Add explosion SFX in `EXPLODE_SFX` mapping (or reuse existing)
4. Add draw call in main rendering loop if unique visuals needed

### Adding a New Weapon
1. Add weapon data to player initialization in `resetGame()`
2. Implement firing pattern in `fireWeapon()` function
3. Update `weaponLabel()` to show weapon name in HUD
4. Add bullet properties (speed, spread, piercing, etc.)

### Adding a New Boss
1. Add boss definition to `BOSSES` array with: `name`, `hullTop`, `hullBottom`, `wing`, `core`, `hpMul`
2. Implement attack pattern functions in boss update section (pattern functions start around line 1324)
3. Call pattern functions from `updateBoss()` based on attack state

### Adjusting Game Balance
- Difficulty multipliers: Edit `DIFFICULTIES` constants (lines 485-489)
- Enemy spawn rate: Adjust `spawnBase` and `spawnMin` in difficulty presets
- Player speed: `player.speed` in `resetGame()` (line 877)
- Weapon fire rate: `fireDelay` variable in `update()` (line 1177)
- Enemy bullet damage: Currently instant kill; would need to add player HP system

## Accessibility

- Screen reader support via `#srLive` ARIA live region
- Reduced motion mode (`--reduced-motion` CSS class) disables animations
- Keyboard and touch controls available
- High contrast color scheme in dark mode
- Touch targets (buttons) are 78px for mobile

## Performance Notes

The game targets 60 FPS on mobile. Key considerations:
- Canvas is 480x720 (scaled to fit viewport)
- Particle count increases with explosions; consider a particle limit if adding more effects
- Boss attacks spawn many bullets; avoid excessive bullet counts
- Audio context initialization deferred to first user interaction for performance

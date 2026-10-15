# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with HTML5 Canvas and WebAudio. It's a single-file implementation (1669 lines) with no external dependencies, designed to run directly in any modern browser.

## Running the Game

- **Open in browser**: Simply open `index.html` directly in a web browser, or serve it via any HTTP server
- **No build process needed**: The game is pure HTML/CSS/JavaScript
- **Testing**: Open the game in a browser to see changes immediately; refresh to reload

## Core Architecture

The game uses a traditional game loop architecture with entity-based systems:

### Key Entity Types (all stored as objects/arrays in global scope)
- **Player**: Single instance managing position, weapon type/level, health, invulnerability, and firing state
- **Enemies**: Array of spawned enemies with types (grunt, interceptor, tank), HP, speed, and behavior patterns
- **Bullets**: Player projectiles with pierce mechanics (laser bullets pierce through enemies)
- **Enemy Bullets**: Fired by enemies and bosses in various attack patterns
- **Boss**: Single boss instance per stage with health, attack patterns, telegraph warnings, and phase management
- **Power-ups**: Drops from defeated enemies (weapons, lives, bombs)
- **Particles**: Temporary effect particles for explosions and impacts

### Game State Management
- Core state variables: `score`, `lives`, `stage`, `running`, `paused`, `gameOver`, `kills`, `chain`, `multiplier`
- Input state: `keys` object for keyboard, `joyVec` for joystick input, `firing`, `mouseActive`
- Difficulty settings stored in localStorage: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`
- Adaptive quality system based on frame time performance (adjusts particle counts and visual effects)

### Main Systems

**Update Loop** (`update(dt)` function):
- Handles player movement (keyboard, joystick, mouse steering)
- Weapon firing with cooldowns and rate-limiting
- Enemy spawning logic based on difficulty and stage progress
- Collision detection between player/bullets/enemies/boss
- Combo chain timing and multiplier calculation
- Power-up collection and weapon upgrades

**Render Loop** (`draw()` function):
- Canvas clearing and setup
- Sky gradient background with parallax clouds
- Entity rendering (player, enemies, bullets, particles, boss)
- HUD elements: score, lives, bombs, weapon status, combo multiplier
- Visual effects: screen shake, color vignette flashes, combo popup animations

**Boss System**:
- Boss enters from top of screen during stage transitions
- Telegraph warnings before each attack pattern (22 frames visual warning)
- Multiple attack patterns per boss type (defined in `BOSS_PATTERNS` array, one array per boss type)
- Boss moves horizontally and bounces off screen edges
- Damage from player bullets and smart bombs
- `killBoss()` triggers stage progression

### Difficulty Settings

Three difficulty presets control spawn rates, enemy behavior, starting lives, and bomb count:
- **Easy**: Slower enemies, lower HP bosses, 4 lives, 3 bombs
- **Normal**: Medium speed, standard HP, 3 lives, 2 bombs
- **Hard**: Fast enemies, high HP bosses, 3 lives, 2 bombs

### Input Handling

**Desktop**:
- Arrow keys or WASD: Move
- Space: Fire
- X or Shift: Use bomb
- P or Esc: Pause
- Mouse: Steering (glides toward pointer), left-click fires

**Touch/Mobile**:
- Virtual joystick in left zone for movement
- Fire and Bomb buttons in bottom corners
- Tap to activate (firing is continuous while held)

### Weapon System

Two weapon types with upgrade levels:
- **Vulcan** (default): Spread bullets, more projectiles at higher levels
  - Level 1: Single bullet
  - Level 2: Two bullets (angled)
  - Level 3: Three bullets (wide spread)
- **Laser**: Piercing bullets that pass through multiple enemies
  - Level 1: Single lane
  - Level 2: Two lanes
  - Level 3: Three lanes (center + sides)

Weapons cycle through `weapon` property and drop from defeated enemies via power-ups.

### Audio System

All sounds are procedurally generated using WebAudio API (no audio files). The `beep()` and `noiseBurst()` functions create:
- Shooting sounds
- Boss telegraph alerts
- Hit/death effects
- Bomb detonation
- UI clicks
- Game over stinger

### Collision Detection

Simple axis-aligned bounding box (AABB) collision using `rectsOverlap()` function:
- Checks if absolute distance between centers < sum of half-widths/heights
- Used for: player-enemy, player-bullet, bullet-enemy, player-power-up, enemy-boss boundary checks

### Visual Effects & Animation

- **Screen shake**: Applies random offset to canvas context transform based on magnitude and duration
- **Flash effects**: Full-screen white flash on hit (red vignette), color flash on bomb
- **Particle explosions**: Radial bursts with velocity and fade-out for impacts and deaths
- **Combo popup**: Animated text showing consecutive kills with scaling animation
- **Stage banner**: Announcement text that fades in/out with scale animation
- **Boss health bar**: Top-center bar that appears when boss is active

### Performance Adaptation

The `quality` variable (0 = full, 1 = medium, 2 = low) adjusts visual detail based on frame time:
- Particle counts scale down at lower quality levels
- Shadow blur effects disabled at low quality
- Stored in `frameTimes` array (last 45 frames) and adjusted automatically
- Intended to maintain 60 FPS on slower devices

### Responsive Design & Theming

**Canvas Sizing**:
- Base resolution: 480×720 px
- Scales up to 1.6x based on device size
- Responsive to window resize and orientation changes

**Themes** (cycle per stage):
- 6 different sky gradients, cloud colors, and names
- Themes change every stage and loop after stage 6

## Common Modifications

**Adding a new enemy type**:
1. Add entry to `ENEMY_TYPES` object with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Update `spawnEnemy()` function to include it in the selection logic

**Adding a new boss**:
1. Add boss design to `BOSSES` array with hull/wing/core colors and HP multiplier
2. Create corresponding attack pattern array in `BOSS_PATTERNS` (patterns are functions that fire enemy bullets)
3. Boss appears in stages 1, 2, 3... in order (loops after stage 6)

**Tweaking difficulty**:
Edit `DIFFICULTIES` object values: `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`

**Changing visuals**:
- Enemy colors: Edit `ENEMY_TYPES` color/dark properties
- Boss appearance: Edit `drawBoss()` function (uses fillStyle, shadow, and context transforms)
- Background: Edit `THEMES` array or the draw loop sky gradient logic

## Testing & Debugging

- Open browser DevTools (F12) to see console logs
- The `announce()` function logs to screen reader element for accessibility
- Combo multiplier and score updates logged to HUD in real-time
- Frame timing tracked in `frameTimes` array for performance debugging

## File Structure

```
index.html      Single-file game: HTML structure, CSS styling, JavaScript game code
README.md       Project metadata
CLAUDE.md       This file
```

## Key Performance Considerations

- Canvas rendering is 2D context only (no WebGL)
- Particles are simple colored dots with basic velocity
- Collision checks are O(n²) in worst case (could be optimized with spatial hashing for very large enemy counts)
- No animation frame skipping—runs at device's native refresh rate (60 FPS target)
- WebAudio context requires user interaction to initialize (ensured via click/touch events)

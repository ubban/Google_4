# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in HTML5. The entire game (HTML, CSS, and JavaScript) is contained in a single `index.html` file (1,669 lines). It features:
- Arcade-style gameplay with adaptive difficulty progression
- Multiple difficulty levels (easy, normal, hard)
- 6 unique boss encounters with different attack patterns
- WebAudio synthesis for all sound effects (no external audio assets)
- Touch controls and analog joystick support for mobile/tablet play
- Visual themes with parallax cloud effects
- Local storage for high scores and user preferences

## Code Architecture

The game uses a **single-file architecture** organized around these core systems:

### 1. **Game Loop** (line 1640)
- `loop(now)` - requestAnimationFrame-driven game tick
- Handles frame timing, delta time calculation, and performance tracking
- Calls `update(dt)` for game state, `draw()` for rendering
- Adaptive quality system adjusts performance based on frame times

### 2. **State Management** (lines 422-527)
- Core variables: `player`, `enemies`, `boss`, `bullets`, `enemyBullets`, `powerUps`, `particles`
- Game state flags: `running`, `gameOver`, `paused`, `firing`
- Player state: position, health, weapon type/level, invulnerability frames, cooldown
- Difficulty config: `DIFFICULTIES` object with spawn rates, enemy speeds, boss multipliers
- Stage progression: increments on boss kills, determines visual theme and enemy spawns
- Local storage keys for high score, mute state, difficulty, and motion preferences

### 3. **Update System** (line 1148)
Main `update(dt)` function handles:
- **Input Processing**: Keyboard (WASD/Arrows), mouse targeting, joystick input
- **Player Movement**: Constrained within canvas bounds, separate mouse-guided and keyboard-driven modes
- **Combat**:
  - Weapon firing with cooldown (laser fires faster than bullets)
  - Laser piercing mechanic (reduces pierce count per enemy hit)
  - Collision detection between player bullets and enemies/boss
  - Damage application and enemy death handling
- **Enemy Spawning**: Difficulty-based spawn timer with ramp scaling
- **Boss Transition**: Spawn boss after killing 18 enemies per stage (KILLS_PER_STAGE)
- **Collision Resolution**:
  - Enemy collision with player → life loss
  - Enemy bullet collision with player → life loss
  - Power-up collision with player → weapon/bomb gain
- **Particle Updates**: Position and lifetime decay

### 4. **Rendering System** (line 1520)
`draw()` function renders in order:
1. **Screen Shake**: Apply translation offset if shakeTime > 0
2. **Background**: Gradient sky based on current theme
3. **Sun Glow**: Radial gradient overlay
4. **Parallax Clouds**: Two layers at different speeds
5. **Player**: Weapon sprite depends on current weapon type
6. **Enemies**: Color and shading based on type
7. **Boss**: Drawn last to appear on top
8. **Bullets, Particles, UI Elements**: HUD and visual effects

### 5. **Audio System** (lines 542-676)
WebAudio synthesis (no external assets):
- `ensureAudio()` - Lazy-load and resume AudioContext
- `beep(freq, dur, type, vol, opts)` - Pure tone generation with optional frequency sweep, filter, pan, delay
- `noiseBurst(dur, vol, opts)` - White noise generation with filter
- `sfx` object - Named sound effects for game events (shoot, explode, bomb, etc.)
- Dynamic sound selection: `EXPLODE_SFX` maps enemy type to explosion sound
- Boss telegraph sound plays before attack patterns

### 6. **Entity Systems**

#### Enemy Management
- `ENEMY_TYPES` object defines: grunt, interceptor (fast/weaving), tank (armored)
- `spawnEnemy()` - Picks random type based on stage, randomizes position
- Enemy-specific behavior:
  - Grunts/Interceptors: Basic downward movement
  - Interceptors: Sinusoidal weaving pattern
  - Tanks: Slower, higher HP
  - All: Shoot at or near player with configurable spread patterns
- `stageKills` counter triggers boss spawn at threshold

#### Boss System
- 6 boss designs in `BOSSES` array: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Each boss has `BOSS_PATTERNS` (line 1372): 3 attack patterns rotated during fight
- Boss patterns available:
  - `fireRadial()` - Circular bullet spray
  - `fireColumns()` - Straight vertical streams
  - `fireAimedSpread()` - Spread attack aimed at player
  - `fireSpiralBurst()` - Rotating spiral
  - `fireHomingSeeded()` - Player-targeting bullets with noise
  - `fireCross()`, `fireDoubleRing()` - Geometric patterns
- Telegraph mechanic: 22-frame warning window before each attack
- Boss HP: Base 30, multiplied by `bossHpMul` from difficulty + `hpMul` from boss design
- `updateBoss(dt)` handles entering animation, horizontal bounds checking, pattern cycling

#### Weapon System
- Two weapon types: `bullet` (default, fast fire) and `laser` (slow fire, piercing)
- `fireWeapon()` - Generates bullets based on weapon type
- Laser piercing tracked by `b.pierce` (reduced on each hit, removed when <= 0)
- Weapon level affects fire rate (laser gains faster fire at level 2+)
- Power-ups increment weapon level, reset when switching weapons

#### Power-Up System
- Types: `weaponSwitch`, `weaponLvl`, `bomb`
- `maybeSpawnPowerUp()` - 20% chance per enemy kill
- `applyPowerUp()` - Implements effect (switch weapon, level up, add bomb)
- Visual representation: falling objects, collected on overlap

### 7. **Input Handling**

#### Keyboard & Mouse
- Arrow keys or WASD for movement
- Mouse click/hold for targeted movement
- Space bar for firing
- Pause button (⏸) for game pause

#### Touch & Joystick (mobile)
- Move zone joystick (left side) for directional input
- Fire button (right side) for shooting
- Bomb button for smart bomb activation
- `joyVec` tracks normalized joystick displacement

### 8. **Progression & Scoring**

- **Score**: Enemies yield points (`ENEMY_TYPES[type].score`), bosses give 500
- **Multiplier**: Increases per kill within time window (`chainTimer`)
- **Break Chain**: Reset multiplier and announce combo
- **Bomb System**: Limited uses per life, smart bomb clears screen
- **Stage**: Increments on boss defeat, changes visual theme and enemy spawns
- **Difficulty Ramp**: Spawn intervals decrease based on frame count

### 9. **Visual Effects**

- **Particles**: `spawnExplosion()` creates colored particles with physics
- **Screen Shake**: `screenShake(magnitude, duration)` for impact feedback
- **Flash**: `flashHit()` and `flashBomb()` add fullscreen color flash
- **Vignette**: Red pulse animation on critical health (1 life remaining)
- **Reduced Motion**: User preference respects `prefers-reduced-motion` media query

### 10. **UI & HUD** (built from HTML elements referenced in lines 425-461)

- **HUD Stats**: Score, best score, weapon type/level, multiplier
- **Health Display**: Heart emoji showing current/max lives
- **Bombs Display**: Bomb emoji showing available smart bombs
- **Stage Banner**: Large stage number announcement
- **Boss Health Bar**: Shows when boss spawns
- **Pause Overlay**: Modal overlay with resume/quit
- **Game Over Overlay**: Final stats and new best score notification
- **Screen Reader**: `srLive` element for accessibility announcements

## Development Notes

### Common Tasks

**Running the Game**
- Open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge)
- No build step required
- Local storage persists across sessions when opened from file:// or same origin

**Testing Changes**
- Game automatically starts in a paused state on page load
- Use browser DevTools Console to test specific functions:
  - Call `fireWeapon()` to test bullet spawning
  - Modify global variables like `lives`, `score`, `stage` to test progression
  - Trigger `endGame()` to test game-over screen
  - Call `spawnBoss()` to test boss encounters

**Debugging Performance**
- The adaptive quality system (`quality` variable) adjusts rendering quality
- Frame times tracked in `frameTimes` array (last 45 frames)
- Quality increases if average frame time > 1/40s, decreases if < 1/56s

### Key Constants

| Constant | Purpose |
|----------|---------|
| `KILLS_PER_STAGE` | Enemy kills before boss spawn (18) |
| `DIFFICULTIES` | Config per difficulty level |
| `THEMES` | 6 visual themes (sky gradients, colors) |
| `ENEMY_TYPES` | 3 enemy variants with stats |
| `BOSSES` | 6 boss designs with colors/HP multipliers |
| `BOSS_PATTERNS` | Attack patterns per boss (6 bosses × 3 patterns) |
| `EXPLODE_SFX` | Sound effect mapping by enemy type |

### Important Implementation Details

1. **Collision**: Simple AABB (axis-aligned bounding box) via `rectsOverlap()` function
2. **Weapon Switching**: Resets weapon level when player picks up opposite weapon type
3. **Bomb Mechanics**: Uses `spawnExplosion()` with larger radius to clear screen
4. **Canvas Sizing**: Responsive via CSS `max-width/max-height`; game logic uses fixed pixel coordinates
5. **Audio Context**: Lazily initialized on first user interaction to comply with browser autoplay policies
6. **Frame-Based Timing**: Despite using delta time (dt), some systems use frame count (`frame` variable) for deterministic timing
7. **Score Multiplier**: Chain system rewards consecutive kills within time window; breaks if no enemy killed within timeout

### Adding New Features

**New Enemy Type**: Add entry to `ENEMY_TYPES` object with stats, add rendering logic in `draw()`  
**New Boss Pattern**: Add function to `BOSS_PATTERNS` array (e.g., `fireHelix()`)  
**New Power-Up**: Add type case in `applyPowerUp()`, add spawn chance in `maybeSpawnPowerUp()`  
**New Visual Theme**: Add theme object to `THEMES` array with sky gradients and colors  

### Browser Compatibility

- Requires: ES6 (arrow functions, template literals, const/let)
- Uses: Canvas 2D Context, WebAudio API, requestAnimationFrame, localStorage, matchMedia
- Tested on: Chrome, Firefox, Safari 12+, modern mobile browsers
- Touch support: Joystick and button elements render on mobile layouts

### Performance Considerations

- Single canvas for all rendering (no off-screen canvases for sprites)
- Bullet/enemy arrays managed with filter operations (not object pool)
- WebAudio synth avoids external audio file loads (smaller footprint)
- Parallax effect uses modulo arithmetic to loop clouds efficiently
- Enemies capped naturally by screen bounds and removal on off-canvas

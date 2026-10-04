# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in HTML5/Canvas with no external dependencies. The game features:
- Player aircraft with dual weapon systems (Vulcan guns and laser cannons)
- Progressive enemy waves across multiple stages
- Boss battles with unique attack patterns
- Dynamically generated sound effects using Web Audio API
- Score multiplier system based on kill chains
- Difficulty settings (Easy, Normal, Hard)
- Full mobile touch controls support

**Canvas Resolution**: 480×720 (game logic), scaled responsively to screen

## Architecture Overview

The entire game is contained in a single `index.html` file (~1670 lines) with three main sections:

### 1. **HTML/CSS (lines 1–361)**
- Game canvas and HUD elements
- Touch control zones and buttons
- Overlay UI for start screen, pause, and game over
- Responsive styling with CSS animations
- Screen reader announcements for accessibility

### 2. **Audio System (lines 542–676)**
- **Web Audio API synthesizer** generating all SFX in real-time (no audio files)
- Waveform types: square, sawtooth, sine, triangle, noise
- Dynamic frequency sweeps, filtering, and stereo panning
- Sound effects mapped to game events (shoot, explode, bomb, power-up, boss warnings)
- Master gain with compression for audio balance

### 3. **Game Logic (lines 863+)**
- Player state and input handling
- Enemy spawning and AI
- Boss controllers with pattern-based attacks
- Collision detection (AABB rectangle overlap)
- Particle/explosion system
- Game state management (running, paused, game over)

---

## Key Game Systems

### Player & Input
- **Player Object**: `{ x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }`
- **Input Priority**: Keyboard/joystick > mouse (so no fighting between control types)
- **Weapons**: 
  - **Vulcan**: Spread fire pattern, increases spread at higher levels (level 1–3)
  - **Laser**: Higher speed, piercing bullets, dual lanes at level 2+
- **Fire Rate**: 150ms (Vulcan), 100ms (Laser)
- Mouse steering when hovering canvas; keyboard/joystick always take priority

### Enemies
Three enemy types defined in `ENEMY_TYPES`:
- **Grunt**: Fast, 1 HP, score 10, spawns from stage 1
- **Interceptor**: Very fast (1.7× multiplier), 1 HP, score 15, weaves side-to-side, spawns from stage 1
- **Tank**: Heavy (0.55× speed), 4 HP, score 35, spawns from stage 2

**Spawn Logic**:
- Random horizontal spawn at top (`-30px`)
- Difficulty affects spawn rate and enemy AI
- Enemies despawn when they leave bottom of canvas
- Spawn timer decreases as frames accumulate (difficulty ramp every 300 frames = ~5 seconds at 60fps)

**Enemy AI**:
- Weaving interceptors follow a sine wave offset by frame number
- Shoot patterns: either **spread** (3-way arc) or **aimed** (toward player)
- Shoot timer resets randomly after each shot (60–150 frames)

### Boss System
Six unique boss types cycle through stages (loops after stage 6):
- **VANGUARD**: Radial bursts, aimed fan, wide V-pattern
- **BEHEMOTH**: Heavy columns, slow homing, dense radial wall
- **SPECTER**: Teleport dashes, spiral bursts, double aimed shots
- **DREADNOUGHT**: 8-way cross fire, homing missiles, dense wall
- **WRAITH**: Teleport + cross fire, expanding double rings, aimed cross-spread
- **TITAN**: Heavy columns, full ring, relentless homing

**Boss Behavior**:
- Boss enters from top and settles at `y=120`
- Moves horizontally between bounds; bounces off edges at `vx = 55 pixels/sec`
- **Attack Pattern**: Telegraph (visual warning + beep) → 22-frame delay → Fire → 85-frame cooldown
- Each boss has 3 unique attack patterns (cycles through them)
- HP scales by stage, difficulty, and boss multiplier (`hpMul`)

**Boss Bullet Types** (in `fireRadial`, `fireAimedSpread`, etc.):
- **Radial**: Spreads outward in full circle
- **Aimed Spread**: Targets player with spread fan
- **Columns**: Vertical lanes from boss position
- **Spiral Burst**: Rotating ring pattern
- **Homing Seeded**: Aimed with randomness to prevent easy dodging
- **Double Ring**: Two concentric rings at different speeds

### Scoring & Multiplier
- **Chain**: Increments on every kill, resets after 2.2 seconds of no kills
- **Multiplier**: `1 + min(9, floor(chain / 5))` — caps at 10x at 45+ kills
- **Kill Milestone Pop-ups**: Shows "10 KILLS!", "20 KILLS!" etc.
- **Boss Defeat**: +500 base score (also multiplied)

### Difficulty Settings
```javascript
DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0,  enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4,  enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9,  enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
};
```
- **spawnBase/spawnMin**: Enemy spawn interval in frames (decreases per stage)
- **enemySpeed**: Multiplier for enemy downward velocity (60 pixel/frame base)
- **enemyShootChance**: Probability each enemy will shoot
- **lives/bombs**: Player starting resources
- **bossHpMul**: HP multiplier for boss (combined with boss.hpMul)

### Game Loop & Update
**Main Game Cycle**:
1. Handle input (keyboard, joystick, mouse)
2. Update player position (movement, fire rate, invulnerability)
3. Update player/enemy/boss/projectile positions
4. Update chain timer and break chain if expired
5. Collision detection (bullets vs enemies, bullets vs boss, enemies vs player, etc.)
6. Spawn explosions, particles, power-ups on kills
7. Render everything
8. Track frame timing for adaptive quality

**Frame-based Timing**: Uses delta time (`dt`) scaled calculations. Base canvas is 480×720; frame times tracked to adjust particle density if GPU-bound.

### Collision Detection
Simple AABB (axis-aligned bounding box) overlap test:
```javascript
rectsOverlap(a, b) = |a.x - b.x| < (a.w + b.w) / 2 AND |a.y - b.y| < (a.h + b.h) / 2
```
Used for:
- Bullets vs enemies
- Bullets vs boss
- Enemies vs player
- Enemy bullets vs player
- Boss vs player
- Power-ups vs player

### Power-Ups
Spawn randomly on enemy death (6% life, 8% bomb, 20% weapon; 66% spawn rate total):
- **Life**: Restores 1 HP (capped at `maxLives`)
- **Bomb**: Restores 1 bomb (capped at `maxBombs`)
- **Weapon**: Upgrade weapon level (Vulcan 1→3, then switch to Laser; Laser stays at level)

### Screen Effects
- **Screen Shake**: Triggered by bomb use, boss defeat, and player hits
- **Flash Animation**: White flash on hit, brighter on bomb
- **Vignette Pulse**: Red pulsing edge when player at critical health (1 HP)
- **Combo Pop**: Yellow text animation on 10-kill milestones
- **Stage Banner**: Large blue text when stage advances

### Rendering
- Canvas context drawn with canvas 2D API
- Background rendered with sky gradient
- Enemies/boss drawn as geometric shapes (circles, rotated rectangles)
- Bullets rendered as small rectangles
- Particles drawn as squares with decay
- HUD updated via DOM (not canvas)

---

## Constants & Tuning

### Canvas Sizing
- **Base dimensions**: 480×720 px
- **Scale factor**: `min(window.width / 480, window.height / 720, 1.6)`
- Maintains 4:3 aspect ratio on all screens

### Stage Progression
- **KILLS_PER_STAGE**: 18 regular enemy kills trigger boss warning
- Boss spawns 1.2 seconds after warning
- Defeat boss → stage increments, enemy spawn resets

### Themes (6 total, cycling)
Each theme defines sky gradient and cloud/sun colors:
- Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge

### Storage Keys (localStorage)
- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

### Performance Adaptation
Quality setting adjusts particle density:
- Quality 0: 100% particles
- Quality 1: 60% particles
- Quality 2: 35% particles

Tracking average frame time over 45 frames; quality up if FPS drops below 40, down if above 56 FPS.

---

## Key Functions & Entry Points

### Initialization
- `resetGame()`: Initializes all game state, called at start
- `ensureAudio()`: Lazy-initializes Web Audio context (browser requires user interaction first)

### Main Update Loop
- `update(dt)`: Called every frame with delta time; handles all game state updates
- `render()`: Canvas drawing (called after update)
- Main loop uses `requestAnimationFrame`

### Critical Helpers
- `spawnEnemy()`: Creates enemy with random type/pattern
- `spawnBoss()`: Creates boss for current stage
- `updateBoss(dt)`: Boss movement and attack patterns
- `fireWeapon()`: Player fire logic
- `useBomb()`: Clears enemy bullets, damages all enemies/boss
- `loseLife()`: Handles player hit, invulnerability frame, chain break
- `applyPowerUp(type)`: Upgrades weapon/health/bombs
- `addScore(base)`: Increments chain, applies multiplier
- `fireRadial/fireAimedSpread/fireColumns/etc.`: Boss attack patterns

### Rendering Helpers
- `renderHearts(count, max)`: HUD life display
- `renderBombs(count, max)`: HUD bomb display
- `spawnExplosion(x, y, color, count)`: Particle burst
- Various canvas drawing functions for enemies/bullets

---

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with properties: `w, h, hp, speedMul, color, dark, score, minStage`
2. (Optional) Add `weave: true` for side-to-side movement
3. Update spawn logic in `spawnEnemy()` to include new type in conditions
4. Add explosion SFX mapping in `EXPLODE_SFX` if unique sound needed

### Adding a Boss Attack Pattern
1. Add pattern function to `BOSS_PATTERNS` array (indexed by stage % BOSSES.length)
2. Pattern receives boss object `b` and must call `fireRadial`, `fireAimedSpread`, etc.
3. Boss automatically cycles through 3 patterns in order; telegraph delay is constant 22 frames

### Adjusting Difficulty
1. Modify `DIFFICULTIES` object entries (spawn rates, enemy speed, shoot chance, HP mul)
2. Changes apply on next game start
3. Player selection saved to localStorage, persists across sessions

### Adding Sound Effects
1. Create new function in `sfx` object that calls `beep()` and/or `noiseBurst()`
2. `beep(freq, duration, waveType, volume, options)` options: `{ sweepTo, filterType, filterFreq, pan, delay }`
3. `noiseBurst(duration, volume, options)` options: `{ filterType, filterFreq, pan, delay }`
4. Call from game event handlers (e.g., `sfx.shoot()` on fire)

### Tweaking Player Physics
- **Speed**: Modify `player.speed` (default 260 px/sec)
- **Fire rate**: Adjust `fireDelay` in update loop (default 150ms Vulcan, 100ms Laser)
- **Invulnerability duration**: Modify `player.invuln = 1.5` in `loseLife()`
- **Weapon spread pattern**: Adjust angles in `fireWeapon()` spread arrays

### Changing Visual Style
- Canvas background gradient is set inline in CSS (sky theme colors)
- Enemy colors defined in `ENEMY_TYPES`
- Boss colors in `BOSSES` (hullTop, hullBottom, wing, core)
- Screen shake magnitude and duration in calls to `screenShake(mag, dur)`
- Particle colors hardcoded in `spawnExplosion()` calls

---

## Testing & Debugging

### Browser DevTools
- Use browser console to access game state directly (e.g., `player`, `score`, `boss`)
- `localStorage` persists high score and settings; clear with DevTools Application tab if needed
- Mobile DevTools: Chrome remote debugging for touch control testing

### Quick Testing
- **Easy mode**: Load game, set difficulty Easy for rapid iteration
- **Spawn rate**: Modify `DIFFICULTIES[difficulty].spawnBase` to test enemy density
- **Boss HP**: Modify `boss.hp` in browser console mid-game to test patterns

### Accessibility
- Screen reader announcements via `announce(text)` → updates `#srLive` ARIA live region
- Reduced motion preference respected (particle/animation/flash/shake disabled if enabled)
- Color contrast meets WCAG AA for HUD text

---

## Git Workflow

When making changes:
1. Keep updates to `index.html` focused (game logic, tuning, bug fixes)
2. Test in browser before committing (especially collision/scoring/boss patterns)
3. Commit message should reference what changed (e.g., "Add new tank enemy variant", "Fix boss bullet spawn position")
4. No external dependencies to manage — single file keeps deployments simple

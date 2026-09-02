# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a self-contained HTML5 Raiden-style vertical shoot-'em-up game. The entire implementation lives in a single `index.html` file (1670 lines) combining HTML structure, CSS styling, and JavaScript game logic. No build tools, external assets, or dependencies required — everything is procedurally generated or synthesized.

## Quick Start

- **Run**: Open `index.html` in any modern web browser
- **Deploy**: Upload `index.html` and `README.md` to any static web host
- **No build step required** — this is pure HTML5 with Canvas and WebAudio

## Code Architecture

The game is organized into five major systems within a single IIFE (Immediately Invoked Function Expression) scope to avoid global pollution:

### 1. Rendering & State Management (Lines 426–462)
- DOM elements cached for efficient access
- Game state variables: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- HUD rendering (`scoreEl`, `multEl`, `heartsEl`, `bombsEl`, etc.)
- `canvas` context initialized once; canvas sizing responsive to viewport

### 2. Audio Subsystem (Lines 542–678)
- **No external audio files** — all sounds are procedurally synthesized via WebAudio API
- `ensureAudio()`: Lazy-initializes AudioContext (many browsers require user interaction first)
- `beep()`: Generates tones with frequency sweeps, optional filters, panning
- `noiseBurst()`: Synthesizes explosion/impact sounds with noise generation
- `sfx` object maps semantic sound names to synthesized audio (shoot, laser, explode, bomb, powerup, bossTelegraph, etc.)
- Master gain node with compressor for consistent audio levels
- Mute toggle persisted in localStorage

### 3. Input Handling (Lines 767–861)
- **Keyboard**: Arrow keys / WASD for movement, Space for fire, X/Shift for bomb, P/Esc to pause
- **Touch/Mobile**: Virtual joystick in bottom-left; FIRE/BOMB buttons in bottom-right; two-zone touch area
- **Mouse**: Pointer position steers player glide; click to fire
- Priority order: Keyboard/Joystick input → Mouse steering. When keyboard is active, mouse doesn't interfere

### 4. Game Loop & Update Cycle (Lines 1148–1301)
- `update(dt)`: Called per frame with delta-time; handles all game logic
  - Player movement & collision bounds
  - Weapon firing with cooldown timing
  - Enemy spawning with dynamic difficulty ramp
  - Collision detection (player vs. enemies/bullets, player bullets vs. enemies/boss, power-ups)
  - Enemy AI (weaving patterns, aiming)
  - Particle and projectile lifecycle management
  - Kill chain & score multiplier decay
- `draw()`: Canvas rendering with adaptive quality (glow effects only at quality level 0)
- `loop()`: RequestAnimationFrame callback; tracks FPS and adjusts quality level

### 5. Game State Controllers (Lines 875–1505)
- `resetGame()`: Initializes all state for a new game session
- `endGame()`: Handles game-over UI, score recording, new-best detection
- `togglePause()`: Pauses/unpauses gameplay and shows overlay
- `applyPowerUp(type)`: Handles life/bomb/weapon upgrades

### 6. Enemies & Boss Logic (Lines 1022–1433)
- **Enemy Spawning** (`spawnEnemy`): Three types (grunt, interceptor, tank) with stage-based unlocking
  - Enemy types have fixed HP, speed, color, and shoot patterns
  - Interceptors weave side-to-side; others move straight down
  - Shooting patterns: "spread" (radial burst) or "aimed" (toward player)
- **Boss Spawning** (`spawnBoss`): One boss per stage, chosen cyclically from BOSSES array
  - Entrance animation (slides down over 1.3s)
  - Boss movement: Horizontal pacing with wall bouncing
  - Telegraph phase (22 frames) before attack, then attack cooldown (85 frames)
- **Boss Attack Patterns** (BOSS_PATTERNS, lines 1372–1409): Each boss has 3 unique firing patterns
  - Patterns include radial bursts, aimed spreads, homing missiles, spirals, columns, teleporting barrage
  - Pattern rotates on each attack phase; boss cycles through 3 patterns continuously

## Configuration & Difficulty

### Difficulty Settings (Lines 485–489)
```javascript
const DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
};
```
- `spawnBase/spawnMin`: Enemy spawn rate (base frames between spawns; decreases over time)
- `enemySpeed`: Multiplier on enemy movement speed
- `enemyShootChance`: Probability each enemy will shoot
- `lives/bombs`: Starting resources
- `bossHpMul`: Boss health scaling

### Game Constants
- `KILLS_PER_STAGE = 18`: Kills required before boss appears
- `BASE_W = 480, BASE_H = 720`: Canvas resolution (16:9.6 aspect)
- `THEMES`: 6 visual themes with sky gradients, cloud colors, sun effects
- `ENEMY_TYPES`: Defines grunt, interceptor, tank (hp, speed, color, score, appearance stage)
- `BOSSES`: 6 named boss types with unique hull colors, core colors, HP multipliers

### Weapon System
Two weapons with level-based progression:
- **Vulcan**: Fast, narrow spread. Levels: 1 (straight), 2 (dual shot), 3 (triple spread)
- **Laser**: Slower, piercing. Levels: 1–2 (dual lanes, side only), 3+ (triple lanes with center)

### Scoring & Multiplier (Lines 1087–1101)
- Kill chain timer: 2.2 seconds between consecutive kills to maintain chain
- Multiplier formula: `1 + floor(chain / 5)` capped at 10x
- Chain breaks if player gets hit or timer expires
- Combo popup shows at every 10 kills milestone

## Collision & Physics

### Spatial Collision (Line 1069)
```javascript
function rectsOverlap(a, b) {
  return Math.abs(a.x - b.x) < (a.w + b.w) / 2 && Math.abs(a.y - b.y) < (a.h + b.h) / 2;
}
```
- Uses AABB (Axis-Aligned Bounding Box) collision
- Objects have `x, y, w, h` properties

### Entity Lifecycle
- **Bullets**: Deleted when off-screen or pierce reaches 0 (lasers only)
- **Enemies**: Deleted when off-screen bottom or HP ≤ 0
- **Enemy Bullets**: Deleted when off-screen bottom
- **Particles**: Deleted when `life <= 0`
- **Power-ups**: Deleted when off-screen or collected

## Visual & Audio Effects

### Screen Shake (Line 727)
- `screenShake(magnitude, duration)`: Applies random X/Y translation to canvas
- Reduced to 15% magnitude if reduced-motion is enabled
- Used for: boss death (18), bomb (14), player hit (10), boss hit (2)

### Flash Overlays (Lines 698–709)
- `flashHit()`: White flash (0.25s) on player damage
- `flashBomb()`: White flash (0.5s) on bomb usage
- Disabled if reduced-motion is enabled

### Vignette Effect (Lines 75–87)
- Red radial gradient vignette pulse when player has 1 life remaining
- Indicates critical condition with visual and audio heartbeat feedback

## Accessibility Features

1. **Reduced Motion Support** (Lines 529–540)
   - Respects `prefers-reduced-motion` media query
   - Toggle button (🎆/🚫) in top-right controls effects
   - Disables screen shake, flashes, animations when enabled
   - CSS rule at line 349 removes animations when `body.reduced-motion` is set

2. **Screen Reader Support** (Line 451, 470)
   - `srLive` element with `role="status"` aria-live
   - `announce()` function queues text updates for screen readers
   - Used for: pause notifications, life count, boss defeat, stage progression

3. **Touch Controls Display** (Lines 291–341)
   - Virtual controls only show on touch devices
   - Joystick base and stick visualize movement input
   - Buttons labeled FIRE and BOMB

## Performance Optimization

### Adaptive Quality (Lines 474–483)
- Tracks frame times (45-frame rolling average)
- Three quality levels:
  - Level 0 (full): Glow effects on bullets/bosses, particle count normal, shadow blur high
  - Level 1 (medium): Reduced glow/shadow, 60% particle count
  - Level 2 (low): No shadow effects, 35% particle count
- Quality degrades if average frame time > 25ms (40 FPS); recovers below 17.8ms (56 FPS)

### Rendering Optimization
- Particle count scales with quality level (line 1074)
- Shadow/glow effects conditionally applied (lines 1002, 1579–1591, 1594)
- Canvas cleared once per frame; batch draw operations

### Collision Culling
- Bullets/enemies automatically removed when off-screen (with 20px margin)
- No spatial partitioning (acceptable for ≈50–80 active entities)

## Extending the Game

### Adding a New Weapon Type
1. Update `fireWeapon()` (lines 1124–1146) with new weapon branch
2. Update `weaponLabel()` (line 871) to return display name
3. Modify weapon toggle logic in `applyPowerUp()` (line 1315)

### Adding a New Boss
1. Add entry to `BOSSES` array (lines 508–515) with hull colors and HP multiplier
2. Add corresponding pattern array entry to `BOSS_PATTERNS` (lines 1372–1409)
3. Update `drawBossShip()` (lines 970–1020) if visual design should differ (currently uses hull/wing/core colors from design object)

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` (lines 502–506) with w, h, hp, speedMul, color, score, minStage
2. Update `spawnEnemy()` (line 1026) logic to include new type in probability roll
3. Add explosion SFX entry in `EXPLODE_SFX` (line 678) if unique sound needed

### Adding a New Theme
1. Add entry to `THEMES` array (lines 493–500) with sky gradient stops, cloud color, sun color, name
2. Themes are selected by: `(stage - 1) % THEMES.length` in `draw()` (line 1529)

### Tweaking Difficulty
- Modify DIFFICULTIES object for spawn rate, enemy stats, resource counts
- Boss HP scaling: adjust base HP (line 1046) or individual boss `hpMul` values
- Adjust `KILLS_PER_STAGE` (line 491) for when boss appears

### Adjusting Audio
- All synthesis happens in `beep()` and `noiseBurst()` functions
- Modify frequency, duration, filter settings in individual `sfx.*()` definitions to change sound character
- Adjust `masterGain.gain.value` (line 556) to change overall volume

## Local Storage Keys

- `skyfighter.bestScore`: High score (numeric)
- `skyfighter.muted`: Mute state ("0" or "1")
- `skyfighter.difficulty`: Last difficulty selected ("easy", "normal", or "hard")
- `skyfighter.reducedMotion`: Accessibility toggle ("0" or "1")

## Canvas Sizing & Responsiveness

- Fixed internal resolution: 480×720
- Scales up to fit viewport, capped at 1.6x (line 757)
- Respects safe-area insets for notched devices (used in HUD positioning)
- Resize handler and orientationchange listener keep canvas responsive

## Known Patterns & Conventions

1. **Vector Operations**: Used for movement, rotation, collision. Objects typically have `x, y, vx, vy` or just `x, y` for position.
2. **Entity Objects**: All game entities (player, enemies, bullets, power-ups) stored as plain objects with positional and state properties. No classes.
3. **Timing**: All timing uses `dt` (delta-time in seconds). Frame-based counters for discrete events (telegraph timer, attack timer, etc.) stored as integer frame counts or float durations.
4. **Random Patterns**: Enemies use `patternRoll` for behavior selection; boss attacks rotate through patterns cyclically.
5. **Color Encoding**: Colors stored as hex strings or RGB arrays. Radial/linear gradients used for visual depth.
6. **Event Delegation**: Touch events use `e.changedTouches` to track multi-touch; single joystick supported via `joyId`.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. It features:
- Canvas-based rendering at 60 FPS
- Multiple difficulty levels (easy, normal, hard)
- Boss battles with unique attack patterns
- Weapon upgrades and power-up system
- Synthesized sound effects using Web Audio API
- Responsive design with touch controls for mobile
- Adaptive performance scaling
- LocalStorage-based save system

## Architecture

### Single-File Design
The entire game is contained in `index.html` (~1,670 lines):
- **HTML**: Canvas element, DOM elements for UI and HUD
- **CSS**: Responsive styling, animations, visual effects, touch controls
- **JavaScript**: Game logic in an IIFE (Immediately Invoked Function Expression) for scope isolation

### Core Game Loop
Located at line 1640-1652, the `loop(now)` function runs every frame:
1. **Track performance** (adaptive quality scaling)
2. **Handle input** (keyboard, touch, joystick)
3. **Update game state** (enemies, player, projectiles, effects)
4. **Draw everything** (background, objects, HUD, overlays)
5. **Schedule next frame** with `requestAnimationFrame`

The game uses delta-time (`dt`) for frame-independent updates.

### Key Game Systems

#### 1. **Rendering & Canvas** (lines 1520+)
- `draw()`: Main render function, calls all drawing routines
- `drawPlane(x, y, w, h, color, darkColor, flame)`: Renders player and enemy aircraft
- `drawBossShip(b)`: Boss-specific rendering with hull, wings, core
- `drawClouds(speed, fillColor, rx, ry)`: Parallax background clouds
- Canvas size adapts via `resize()` to window dimensions and device pixel ratio

#### 2. **Game State** (lines 424-528)
Global game state includes:
- `player`: Position, health, weapon level, bomb count
- `score`, `combo`, `multiplier`: Score tracking
- `enemies[]`: Active enemy list
- `boss`: Current boss (if any)
- `projectiles[]`: Player bullets and enemy fire
- `particles[]`: Visual effects (explosions, etc.)
- Difficulty settings, themes, input state

#### 3. **Enemy System** (lines 1022-1022)
- **Enemy Types** (line 502-506): `grunt`, `interceptor`, `tank` with distinct stats (HP, speed, AI, score)
- `spawnEnemy()`: Random enemy selection based on stage and difficulty
- `update(dt)` handles enemy movement, shooting, and collision detection
- Enemies use simple AI: move down, weave (interceptors), shoot when in range

#### 4. **Boss System** (lines 1043-1447)
- **Boss Types** (line 508-515): 6 boss variants with unique visuals and `hpMul` scaling
- `spawnBoss()`: Triggered after KILLS_PER_STAGE (18) enemy kills
- `updateBoss(dt)` runs boss attack patterns: telegraphed attacks with distinct firing modes
- Boss attack patterns include:
  - `fireRadial()`: Circular bullet spray
  - `fireAimedSpread()`: Aimed spread pattern
  - `fireColumns()`: Vertical columns
  - `fireSpiralBurst()`: Spiral waves
  - `fireHomingSeeded()`, `fireDoubleRing()`, `fireCross()`: Specialized attacks
- Boss health bar updated in HUD (`#bossBarWrap`)

#### 5. **Weapon & Power-Up System** (lines 1304-1370)
- Weapons are indexed (0-2) with level progression
- Power-ups spawn randomly from defeated enemies via `maybeSpawnPowerUp()`
- `applyPowerUp(type)`: Upgrades weapon or grants bombs
- Weapon name displayed in HUD (`#weaponName`, `#weaponLvl`)

#### 6. **Input Handling** (lines 785-870)
- **Keyboard**: Arrow keys for movement, `Z` to fire, `X` for bombs
- **Touch/Mouse**: Canvas click to move player to click position
- **Joystick** (mobile): Visual joystick with `moveZone`, `joystickBase`, `joystickStick` elements
- Input state tracked in `input` object (normalized x/y position)

#### 7. **Audio System** (lines 542-676)
- Web Audio API with synthesized sound effects (no external audio files)
- `beep()`: Tone synthesis with frequency sweep, filter, pan support
- `noiseBurst()`: Noise-based explosion sounds
- Sound effects for: shooting, explosions, powerups, boss attacks, game over
- Master gain and dynamics compressor for audio quality
- Respects mute toggle and user audio context state

#### 8. **Visual Effects & Performance** (lines 698-733)
- Flash effects: `flashHit()`, `flashBomb()` (respect reduced-motion preference)
- Screen shake: `screenShake(mag, dur)` for impact feedback
- Particle explosions: `spawnExplosion(x, y, color, count)`
- Adaptive quality scaling (line 474-483): Monitors frame time, reduces quality (particle count, effects) on slower devices

#### 9. **Score & Combo System** (lines 1087-1096)
- `addScore(base)`: Awards points with combo multiplier, triggers combo display
- Combo increments on enemy kills, breaks on player hit or timeout
- Score formatted with commas in HUD

#### 10. **Game State Management** (lines 875-1491)
- `resetGame()`: Initializes game for new play session
- `endGame()`: Triggers game over screen, saves best score
- `togglePause()`: Pauses/resumes game (line 1491)
- `loseLife()`: Handles hit detection, game over on lives = 0

## Common Development Tasks

### Running the Game
Simply open `index.html` in a modern browser:
```bash
# Locally (Linux/Mac)
open index.html

# Or serve via HTTP (Python)
python -m http.server 8000
# Then visit http://localhost:8000/
```

### Testing Gameplay
- **Quick test**: Open in browser, test basic controls (keyboard arrows, Z/X keys, mouse click)
- **Mobile test**: Use browser DevTools device emulation (Ctrl+Shift+M in Chrome) or test on actual device
- **Performance**: Open DevTools Performance tab (F12), record a 30-second play session, check FPS
- **Audio**: Test mute/sound toggle, verify effects play (shoot, explode, powerup, boss attacks)

### Editing Game Values
Core numeric constants are grouped at the top of the script:
- **Line 463-466**: Storage keys for persistence
- **Line 474-483**: Performance adaptation thresholds
- **Line 485-489**: Difficulty parameters (spawn rates, speeds, lives, bombs, boss HP scaling)
- **Line 491**: `KILLS_PER_STAGE = 18` — enemies to kill before boss spawns
- **Line 493-500**: `THEMES` array — visual themes (sky colors, cloud color)
- **Line 502-506**: `ENEMY_TYPES` — enemy stats (size, HP, speed, score)
- **Line 508-515**: `BOSSES` — boss visual colors and HP multiplier

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502-506) with stats: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Update `spawnEnemy()` (line 1022) to select the new type based on `stage` and `difficulty`
3. In `update(dt)` (line 1148), handle any unique behavior (special movement, shooting pattern)
4. Add corresponding entry to `EXPLODE_SFX` (line 678) if unique sound needed, or reuse existing

### Adding Boss Attack Pattern
1. Create a new attack function following the existing pattern (e.g., `fireRadial`, `fireColumns`)
2. Call it from `updateBoss(dt)` (line 1411) based on boss phase/timer
3. Attacks should create `{ x, y, vx, vy, damage, type }` bullets and push to `projectiles[]`

### Adjusting Difficulty
Modify the `DIFFICULTIES` object (line 485-489):
- `spawnBase`: Frame delay before first enemy spawn
- `spawnMin`: Minimum spawn interval as difficulty progresses
- `enemySpeed`: Multiplier for enemy velocity
- `enemyShootChance`: Probability enemy fires each frame
- `lives`: Starting player health
- `bombs`: Starting bomb count
- `bossHpMul`: Boss HP multiplier (scales off base HP)

### Responsive Design & Mobile
- Viewport meta tag (line 5) handles mobile layout
- Safe area insets respected via `env(safe-area-inset-*)`
- `resize()` (line 754) recalculates canvas size on window resize
- Touch controls added via `isTouchDevice` check (line 779) and joystick handlers (line 785+)
- Device pixel ratio applied for crisp rendering on high-DPI displays

### localStorage Keys
Game uses four persistent storage keys (line 463-466):
- `skyfighter.bestScore`: Best score achieved
- `skyfighter.muted`: Audio mute state (0/1)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Accessibility toggle (0/1)

## Code Patterns & Conventions

### Game Object Structure
Player, enemies, boss, projectiles follow this pattern:
```js
{
  x, y,           // Position
  vx, vy,         // Velocity
  w, h,           // Width, height
  hp, maxHp,      // Health
  // Type-specific fields (color, shape, behavior)
}
```

### Collision Detection
`rectsOverlap(a, b)` (line 1069): Checks AABB overlap between two rectangular objects.

### Drawing Optimization
- Culling: Objects outside canvas bounds are skipped during update
- Adaptive quality: Particle counts and effect intensity reduce on low-performance devices
- Batch rendering: Clouds, explosions drawn once per frame

### Animation & Timing
- Frame-based animations use timers (e.g., `bossAttackTimer`)
- CSS animations for UI (`@keyframes` at line 71+): heartBeat, flashHit, flashBomb, comboPulse, etc.
- Easing functions inline where needed

## Testing & Debugging

### Browser Console Tricks
```js
// In DevTools console:
player.hp = 10;  // Add lives (respects max health)
score = 100000;  // Set score
enemy.hp = 0;    // Kill enemy
boss.hp = 0;     // Kill boss
difficulty = 'hard'; // Change difficulty (next game)
```

### Visual Debugging
- Screen shake intensity reveals when player is hit
- Flash effect confirms hit detection
- Combo pop shows score awarded
- Boss health bar provides feedback on boss damage

### Performance Profiling
- Frame times logged every 45 frames (line 475)
- Quality auto-adjusts based on average FPS (line 481-482)
- Open DevTools Performance tab to analyze rendering bottlenecks

## Key Files & Line References

| Component | Lines | Notes |
|-----------|-------|-------|
| DOM elements & setup | 426-462 | All canvas and UI elements cached |
| Storage & preferences | 463-540 | Best score, mute, difficulty, motion settings |
| Audio system | 542-676 | Web Audio synth, SFX effects object |
| UI rendering helpers | 680-733 | Hearts, bombs, flashes, combo pop |
| Game state & reset | 875-908 | Player/enemy/boss initialization |
| Enemy system | 1022-1043 | Spawning, AI |
| Boss system | 1043-1447 | Spawning, attacks, health, death |
| Game update loop | 1148-1303 | Player, enemy, projectile, particle updates |
| Power-up system | 1304-1370 | Weapon upgrades, bomb drops |
| Boss attacks | 1324-1370 | Individual attack pattern functions |
| HUD & overlays | 1469-1519 | Game over screen, pause, stage banner |
| Rendering | 1520-1639 | Main draw function and sub-routines |
| Main game loop | 1640-1665 | `loop(now)` and frame scheduling |

## Browser Compatibility

Target modern browsers with:
- Canvas 2D API
- Web Audio API (graceful degradation if unavailable)
- LocalStorage
- RequestAnimationFrame
- Touch events (mobile)
- CSS Grid & Flexbox
- CSS custom properties (`:root`)
- CSS animations & gradients

Tested on: Chrome, Firefox, Safari, Edge (desktop and mobile versions).

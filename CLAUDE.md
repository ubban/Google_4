# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The game features procedurally-spawned enemies, multiple difficulty levels, boss encounters, and an adaptive difficulty system based on performance.

**Key Technical Stack:**
- Pure HTML5 Canvas 2D for rendering
- Web Audio API for procedural sound synthesis (no audio files)
- Vanilla JavaScript (single IIFE module, ~1,670 lines)
- No external dependencies
- Single file: `index.html`

## Running the Game

- Open `index.html` directly in any modern web browser
- No build process, no development server required
- Fully playable from the local filesystem
- Responsive design works on desktop, tablet, and mobile

## High-Level Architecture

The game is structured as a single JavaScript IIFE (Immediately Invoked Function Expression) that encapsulates all game logic and state. The architecture follows a classic game loop pattern:

```
Setup (constants, elements, state initialization)
  ↓
Game Loop (requestAnimationFrame)
  ├─ Input handling (keyboard, mouse, touch)
  ├─ Update phase (dt: time delta in seconds)
  │  ├─ Player movement & firing
  │  ├─ Entity updates (bullets, enemies, particles)
  │  ├─ Collision detection
  │  └─ Game state changes (lives, score, stage progression)
  └─ Render phase
     ├─ Background (gradient + parallax clouds)
     ├─ All entities (player, bullets, enemies, effects)
     └─ HUD (score, multiplier, weapon, health)
```

## Core Game Objects

### Player Object
```javascript
player = {
  x, y, w, h,           // Position and dimensions
  speed: 260,           // Pixels per second
  cooldown: 0,          // Weapon fire delay (seconds)
  rapid: 0,             // Rapid-fire timer (seconds)
  weapon: 'vulcan',     // 'vulcan' or 'laser'
  level: 1,             // Weapon upgrade level (1–5)
  invuln: 0             // Invulnerability timer (seconds)
}
```

### Bullet Objects
```javascript
{ x, y, w, h, vx, vy, laser: bool }  // laser = true for laser weapon
```

### Enemy Objects
```javascript
{
  x, y, w, h,
  type: 'grunt'|'interceptor'|'tank',
  color, dark,          // Main and shadow colors
  hp,                   // Current HP
  vx, vy,               // Velocity
  cooldown,             // Shoot timer
  weave: bool           // Some types strafe left/right
}
```

### Boss Object
```javascript
{
  x, y, w, h,
  hp, maxHp,
  phase: 0–2,           // Boss phases
  patternIndex: 0–6,    // Current attack pattern
  patternTimer,         // Time in current pattern
  cooldown,             // Pattern change timer
  // ... color scheme fields
}
```

### Game State Variables
```javascript
// Game control
running, paused, gameOver

// Player progression
lives, maxLives
score, bestScore
bombCount, maxBombs
stage, stageKills        // Stage 1–6; 18 kills per stage
kills                    // Total kills (triggers boss at 18)

// Combat/multiplier
chain, chainTimer        // Combo counter resets after 2.5 seconds
multiplier              // Score multiplier (1.0–5.0+)

// Rendering
frame                   // Frame counter (for animations)
quality                 // Adaptive: 0 (full) to 2 (low)

// Collections
bullets, enemies, enemyBullets, powerUps, particles, boss
```

## Key Systems

### Spawning & Progression

- **Enemy Spawning**: `spawnEnemy()` uses difficulty-based timing (`spawnBase`, `spawnMin`) with stage scaling
  - Enemy type pool expands as stage increases (`minStage` field on `ENEMY_TYPES`)
  - Spawn position randomized horizontally at top of screen
  
- **Stage Progression**: First 18 kills = Stage 1, next 18 = Stage 2, etc.
  - At stage 2+, tank enemies become available
  - Boss spawns at kill 18 of each stage (separate from enemy count)

- **Boss Progression**: Six bosses in order across six stages
  - Each boss has unique hull colors, wing colors, and core color
  - Boss HP multiplier and attack patterns vary by boss type

### Weapon System

Two weapon types with 5 upgrade levels:

1. **Vulcan (default)**
   - Rapid continuous fire
   - Fire rate increases with level

2. **Laser**
   - Continuous beam from player position
   - Damage scales with upgrades
   - Power-up toggle: `'L'` pickup switches weapon

### Difficulty Settings

Defined in `DIFFICULTIES` object with per-difficulty:
- `spawnBase`: Base spawn interval (frames)
- `spawnMin`: Minimum spawn interval (faster as game progresses)
- `enemySpeed`: Enemy speed multiplier (1.0–1.9)
- `enemyShootChance`: Probability enemy fires each frame
- `lives`: Starting lives
- `bombs`: Starting bombs
- `bossHpMul`: Boss health multiplier

### Collision Detection

- **`rectsOverlap(a, b)`**: Simple AABB (Axis-Aligned Bounding Box) collision
  - Used for bullet-enemy, enemy-player, bullet-player, powerup-player
  - Each entity has `.x, .y, .w, .h` for collision rect

### Visual Themes

Six themes defined in `THEMES` array. Each has:
- Sky gradient colors (3 stops)
- Cloud color (RGB)
- Sun color (RGB)
- Human-readable name

Themes rotate by stage; player can't select them directly (set in code).

### Sound System

Web Audio API procedural synthesis in `beep()` and `noiseBurst()`:
- **`beep(freq, dur, type, vol, opts)`**: Sine/sawtooth tone with optional pitch sweep
- **`noiseBurst(dur, vol, opts)`**: Noise burst with filter sweep
- Optional delay, filter, and stereo pan
- Master gain at 0.9; compressor limits peaks

All sounds muted if audio context unavailable or user toggles mute.

### Adaptive Performance

`trackPerf(dt)` maintains a rolling 45-frame average frame time:
- If avg > 1/40 (25 ms) and quality < 2: increase quality (reduce glow/shadow effects)
- If avg < 1/56 (~17.8 ms) and quality > 0: decrease quality (increase effects)

Quality levels affect:
- `glow = (quality === 0)`: Disables shadow/glow on high-quality mode
- Adaptive reduces visual effects to maintain smooth 60 FPS on low-end devices

## Common Development Tasks

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` object with fields: `w, h, hp, speedMul, color, dark, score, minStage, weave` (optional)
2. Adjust spawn pool thresholds in `spawnEnemy()` if stage-gated
3. Add visual rendering to `draw()` if needed (custom shape beyond `drawPlane()`)
4. Test collision and HP depletion; enemies drop power-ups on death

### Adding a New Boss

1. Add entry to `BOSSES` array with: `name, hullTop, hullBottom, wing, core, hpMul`
2. Implement attack pattern function (e.g., `fireRadial`, `fireAimedSpread`) in `updateBoss()`
3. Call pattern function inside `if (boss)` block, keyed by `boss.patternIndex`
4. Adjust phase transitions in `updateBoss()` based on HP thresholds
5. Test boss health calculation: `baseHp * difficultyMul * bossMul`

### Adding a New Weapon

1. Add weapon name to player's `weapon` field (string)
2. In `fireWeapon()`, add branch for new weapon type
3. Define bullet properties (speed, size, laser flag) and spawn bullets
4. Update `weaponLabel()` for HUD display
5. Consider level scaling (damage per level)

### Adding a New Power-Up Type

1. Add type to `maybeSpawnPowerUp()` spawning logic
2. Render in `draw()` loop within the powerUps forEach
3. Handle collection in `applyPowerUp(type)` switch statement
4. Update HUD display if applicable

### Adjusting Difficulty/Balance

- **Tuning enemy aggression**: Adjust `enemyShootChance` and `enemySpeed` in `DIFFICULTIES`
- **Tuning spawn rate**: Adjust `spawnBase` and `spawnMin`
- **Boss balance**: Modify `bossHpMul` or individual boss `hpMul`
- **Weapon damage**: Adjust bullet velocity or add HP checks in collision
- **Stage length**: Change `KILLS_PER_STAGE` (18) to progress faster/slower

### Adding Visual Effects

- **Screen shake**: Call `screenShake(magnitude, duration)` on hit
- **Flash vignette**: Add `vignetteEl.classList.add('on')` when damaged
- **Particle bursts**: Call `spawnExplosion(x, y, color, count)` on death
- **Popup text**: Call `showComboPop(text)` or `showStageBanner(text)`

## Important Code Locations

| Feature | Location |
|---------|----------|
| Game loop | Line ~1640 `function loop(now)` |
| Update phase | Line ~1148 `function update(dt)` |
| Render phase | Line ~1520 `function draw()` |
| Collision checks | Line ~1200–1290 (inside `update`) |
| Boss update logic | Line ~1411 `function updateBoss(dt)` |
| Boss patterns | Lines ~1324–1409 (fireRadial, fireAimedSpread, etc.) |
| Enemy AI | Line ~1022 `function spawnEnemy()` + update loop |
| Audio | Lines ~542–677 (ensureAudio, beep, noiseBurst) |
| Input handling | Lines ~785–869 (joystick, keyboard, mouse) |
| HUD updates | Scattered (renderHearts, renderBombs, etc.) |

## Performance Considerations

- **Frame capping**: Update clamped to max 50ms per frame to prevent spiral death
- **Particle limits**: Old particles culled when `life ≤ 0`
- **Enemy/bullet limits**: No hard cap, but spawning self-regulates by difficulty
- **Canvas size**: Responsive; set at resize event with `window.innerWidth/Height`
- **Mobile optimization**: Touch controls replace keyboard; reduced particle effects on low quality

## Accessibility Features

- **Reduced motion support**: Respects `prefers-reduced-motion` media query and has toggle button
- **Screen reader announcements**: `announce(text)` updates ARIA live region
- **Keyboard navigation**: Menu navigation via Tab/Space/Enter
- **Color contrast**: White text on dark backgrounds; gold accents for HUD elements

## Local Storage Keys

- `skyfighter.bestScore`: Best score (number)
- `skyfighter.muted`: Audio mute state (0 or 1)
- `skyfighter.difficulty`: Selected difficulty ('easy', 'normal', 'hard')
- `skyfighter.reducedMotion`: Reduced motion preference (0 or 1)

## Git History

- Initial empty repo, then single commit adding Sky Fighter
- Infrequent commits; treat as main development file

## Future Enhancement Ideas

- Multiple playable ships with different stats
- Weekly leaderboard (requires backend)
- New boss attack patterns
- Enemy formations and scripted waves
- Weapon combination mechanics (simultaneous weapons)
- Persistent progression system (level caps, prestige)
- Sound effect toggles per event type

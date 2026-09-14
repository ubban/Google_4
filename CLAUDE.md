# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file HTML5 vertical shoot-'em-up arcade game built entirely with procedural rendering and synthesis. It runs in a web browser with no external dependencies or assets.

- **Main file**: `index.html` (1669 lines) — contains all HTML, CSS, and JavaScript
- **Base resolution**: 480×720 (responsive scaling)
- **Browser support**: All modern browsers with Canvas 2D and Web Audio API support

## Running and Testing

Since this is a single HTML file, simply open it in a browser:
```bash
# Serve locally (pick one):
python -m http.server 8000
# or with npm:
npx http-server

# Then open: http://localhost:8000/
```

**Testing the game**:
- Press Space or click FIRE to shoot
- Move with mouse (aim and the plane follows), arrow keys, or WASD
- Press X or Shift to drop a bomb
- Press P or Escape to pause
- Mobile: drag joystick area to move, tap FIRE/BOMB buttons
- Difficulty selector on start screen (Easy/Normal/Hard)
- Mute button (🔊), motion effects toggle (🎆), pause button (⏸) in top-right

## Code Architecture

The codebase is organized sequentially in `index.html`:

1. **Lines 1–359**: CSS (styles for canvas, HUD, UI overlays, touch controls)
2. **Lines 361–421**: HTML markup (canvas, HUD elements, UI overlays, touch buttons)
3. **Lines 422–1669**: JavaScript game logic in a single IIFE (Immediately Invoked Function Expression)

### Key Sections in JavaScript

#### Constants and Configuration (Lines 463–515)
- `STORAGE_*`: LocalStorage keys for persistent settings (best score, mute, difficulty, reduced motion)
- `DIFFICULTIES`: Spawn rates, enemy speeds, damage, lives, bombs for Easy/Normal/Hard
- `THEMES`: 6 stage themes with sky gradients, cloud colors, sun colors
- `ENEMY_TYPES`: Grunt, Interceptor, Tank definitions (size, HP, speed, score, availability)
- `BOSSES`: 6 boss designs with hull/wing/core colors and HP multipliers

#### Game State (Lines 863–869)
```javascript
// Core arrays
player, bullets, enemies, enemyBullets, particles, powerUps, boss

// Game progress
score, lives, maxLives, stage, stageKills, kills, running, paused, gameOver

// Scoring and progression
chain, chainTimer, multiplier, bombCount, maxBombs, spawnTimer, frame
```

#### Audio System (Lines 542–676)
- **Procedural synthesis**: No external audio files — all sound effects are generated via Web Audio API
- `beep(freq, dur, type, vol, opts)`: Oscillator-based tones with optional frequency sweep, filter, panning
- `noiseBurst(dur, vol, opts)`: Noise-based effects (explosions, impacts)
- `sfx` object: Named effects for shoot, laser, explode (enemy-specific), hit, bomb, powerup, boss interactions, stage clear, UI clicks, game over

#### Input Handling (Lines 767–861)
- **Keyboard**: Arrow keys, WASD (movement), Space (fire), X/Shift (bomb), P/Esc (pause)
- **Mouse**: Movement aims the plane; click fires
- **Touch**: Left zone is joystick (movement), right zone has FIRE and BOMB buttons
- `firing`: Flag set by mouse/touch, checked in update loop
- `joyVec`: Joystick input vector {x, y}
- `mouseTarget`: Target position for plane to follow mouse

#### Rendering (Lines 913–1519)
- **Canvas**: 480×720, cleared and redrawn each frame
- **drawPlane(x, y, w, h, color, darkColor, flame)**: Player plane (procedural, includes thrust flame)
- **drawBossShip(b)**: Boss rendering with animation
- **draw()**: Main render function — background gradient, clouds, player, enemies, boss, bullets, effects, HUD updates
- **Theme system**: Background sky, clouds, and sun change per stage (6 themes cycle)
- **Glow effects**: Conditional shadow glow on bullets when `quality === 0` (good performance)

#### Update Loop (Lines 1600–1650)
- **Main loop**: `gameLoop(currentTime)` — uses `requestAnimationFrame`
- **Delta time**: Calculated from frame timestamps for smooth physics
- **Adaptive quality**: Tracks frame times; reduces visual effects (shadows, particle count) if FPS drops
- **Update sequence**:
  1. Player movement (keyboard/joystick input takes priority over mouse)
  2. Weapon firing (cooldown, bullet spawning)
  3. Enemy spawning (based on difficulty and stage progress)
  4. Boss spawning (when `stageKills >= KILLS_PER_STAGE`)
  5. Entity updates (position, collision, lifetime)
  6. Collision detection (bullets vs enemies/boss, enemy fire vs player)
  7. Enemy bullet firing (random chance or boss patterns)
  8. Particle updates (explosions fade out)
  9. Score/chain/multiplier updates
  10. Rendering

#### Boss Patterns (Lines 1310–1409)
- **BOSS_PATTERNS**: 2D array indexed by boss type, each boss has 3 attack patterns
- Patterns: `fireRadial()`, `fireColumns()`, `fireDoubleRing()`, `fireAimedSpread()`, `fireHomingSeeded()`
- Boss telegraph/warning: Brief pause before attack (telegraphed by white flash)
- Boss HP scales with difficulty and stage

#### Collision and Damage (Lines 1070–1150, approx)
- **Bullet-enemy collision**: Circle/rect overlap checks
- **Enemy-player collision**: Usually only via enemy bullets or contact
- **Chain/multiplier**: Kills within ~1 second of each other increase multiplier
- **Invulnerability frames**: Player has brief invuln after hit to prevent instant re-death

### Important Patterns

#### Entity Objects
All entities are plain JS objects with properties for position, velocity, size, health, etc. Updated and rendered in arrays.

```javascript
// Player
{ x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }

// Bullet
{ x, y, w, h, dx, dy, laser }

// Enemy
{ x, y, w, h, type, hp, vx, vy, shootChance, shootTimer }

// Boss
{ x, y, w, h, hp, maxHp, design, entering, vx, telegraph, attackTimer, patternIndex }
```

#### Procedural Rendering
No sprite sheets or image assets. All visuals drawn with Canvas 2D primitives:
- Planes: triangles, ellipses, paths
- Enemies: colored circles/rectangles
- Bosses: arcs, ellipses, gradients
- Effects: particles with opacity/scale animation
- UI: text, boxes, gradients

#### LocalStorage Persistence
Four keys store user preferences across sessions:
- `skyfighter.bestScore` — best score ever achieved
- `skyfighter.muted` — audio mute state
- `skyfighter.difficulty` — selected difficulty
- `skyfighter.reducedMotion` — animation/shake/flash preference (respects OS prefers-reduced-motion)

#### Performance Optimization
- **Adaptive quality**: `quality` variable (0 = full, 1 = medium, 2 = low) adjusts shadow/glow rendering
- **Particle cap**: Explosions don't spawn unlimited particles
- **Reduced motion mode**: Respects accessibility preferences; disables shakes, flashes, and animations

## Development Tips

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (define `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`)
2. Spawn it in the enemy spawn logic (~line 1195)
3. Add drawing code in the enemy render section (~line 1605)
4. Add death SFX if unique: add to `sfx` object and reference in `EXPLODE_SFX` map

### Adding a New Weapon
1. Add to weapon selection logic in player initialization and weapon switching
2. Modify firing logic (cooldown, bullet pattern, speed)
3. Update `weaponLabel()` and HUD weapon display
4. Create/adjust SFX for new weapon fire sound

### Adding a New Boss Attack Pattern
1. Create a pattern function like `fireRadial(b, bulletCount, speed)`
2. Add to `BOSS_PATTERNS[bossIndex]` array
3. Boss will cycle through patterns automatically

### Mobile Tweaks
- Touch controls appear automatically on touch devices (via `isTouchDevice` check)
- Joystick and buttons are UI-friendly sizes (36px buttons, 100px joystick base)
- `moveZone` (left 55% of bottom) and `actionZone` (right 45%) split the bottom half

### Debugging
- **Screen shake**: Currently tied to hits/bombs. Disabled in reduced-motion mode for accessibility.
- **Frame rate**: Check `quality` variable and `frameTimes` array to see FPS tracking
- **Spawn rates**: Adjust `DIFFICULTIES[difficulty].spawnBase` and `spawnMin` for enemy frequency
- **Audio**: Call `ensureAudio()` manually to wake up AudioContext (browser requirement)

## File Structure

```
Google_4/
├── index.html    (single file with all CSS, HTML, JS)
├── README.md     (minimal project description)
└── CLAUDE.md     (this file)
```

## No Build Required

This project has no build step, package manager, or external dependencies. Simply open the HTML file in a browser. All game assets (graphics, audio, data) are procedurally generated at runtime.

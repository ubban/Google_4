# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single-file HTML5 application. The entire game (HTML, CSS, and JavaScript) is contained in `index.html` (1669 lines).

## Key Architecture

### Game Structure
The codebase is organized into distinct sections within a single file:

1. **DOM Elements & Configuration** (lines 426-466): Canvas setup and UI element references
2. **Audio System** (lines 542-676): WebAudio synth for procedural sound effects (no external audio files)
3. **Game State Management** (lines 863-902): Player object, arrays for bullets/enemies/particles, score/lives tracking
4. **Input Handling** (lines 767-862): Keyboard (WASD/Arrows), mouse steering, touch controls with virtual joystick
5. **Game Logic** (lines 1100-1439): Main update loop, collision detection, boss patterns, difficulty scaling
6. **Rendering** (lines 1500+): Canvas drawing for player, enemies, bullets, particles, bosses
7. **Initialization** (lines 1650+): Event listeners, game start/pause/end handlers

### Core Game Loop
- **Update** (`updateGame(dt)`): Handles input, entity movement, collision detection, scoring
- **Render** (`renderGame()`): Draws all entities to canvas each frame
- **Animation Loop**: `requestAnimationFrame` with delta-time tracking for physics consistency

### Entity System
Entities (player, enemies, bullets, particles, boss) are stored in simple arrays:
- **Player**: Single object with position, weapon state, level, health, invulnerability timer
- **Enemies**: Array of objects; three types (grunt, interceptor, tank) with different stats
- **Bullets**: Player projectiles; two weapon types (vulcan, laser) with pierce mechanics
- **Enemy Bullets**: Stored separately; can be radial (boss patterns) or aimed
- **Particles**: Simple decay-over-time effects for explosions
- **Powerups**: Life, bomb, and weapon upgrades; spawn randomly from defeated enemies
- **Boss**: Single boss object during stage boss phase; visible only during boss phase

### Key Game Systems

#### Difficulty & Progression
- Three difficulty levels (Easy/Normal/Hard) stored in `DIFFICULTIES` object
- Stages progress after defeating 18 enemies (KILLS_PER_STAGE); boss spawns after that
- Six boss types (BOSSES array) cycle through stages; each has unique attack patterns
- Enemy spawn rate dynamically increases over time (harder as stage progresses)

#### Weapon System
Two weapon types with upgrade levels (1-3):
- **Vulcan** (default): Continuous fire, rapid rate
- **Laser**: Higher damage, pierce mechanic (can damage multiple enemies/boss)
- Power-ups can upgrade current weapon level or switch weapons (resets level to 1)

#### Scoring & Combo System
- Kills increment a counter; combo appears every 10 kills ("10 KILLS!", "20 KILLS!", etc.)
- Chain/combo system: multiplier starts at 1x and can increase (mechanism in applyPowerUp)
- Best score persisted to localStorage

#### Boss System
Each boss has:
- Three attack patterns cycled through (BOSS_PATTERNS array)
- Telegraph phase (22 frames of warning before attack)
- Attack cycle (85 frames of projectiles)
- Multiple projectile generation functions (radial, aimed, homing, spirals, columns, etc.)

#### Rendering Effects
- **Vignette**: Red edge effect when player has 1 life; pulsing animation
- **Screen Shake**: Camera shake on hits/explosions (reduced in reduced-motion mode)
- **Flash**: White/bright flash on hit and bomb detonation
- **Combo Pop**: Yellow text animation when hitting kill milestones
- **Stage Banner**: Announcement text when advancing to new stage
- **Boss Health Bar**: Appears when boss is active; updates in real-time

#### Input Handling
- **Keyboard**: Arrow keys or WASD for movement, Space/X/Shift for bomb
- **Mouse**: Movement steers player, click fires; joystick input takes priority
- **Touch**: Virtual joystick on left (movement zone), Fire/Bomb buttons on right
- Pause: P key or Escape

#### Accessibility Features
- **Reduced Motion**: Toggle via button; disables animations/shake/flash for vestibular sensitivity
- **Screen Reader Support**: `aria-live` region for announcements (lives lost, stage transitions, etc.)
- **Responsive Canvas**: Scales to fill viewport (max 1.6x base 480x720 size)

## Common Development Tasks

### Testing the Game
Since this is a single-file game, you can open `index.html` directly in a browser:
```bash
# Simply open in browser to test
# Or use any local server:
python3 -m http.server 8000
# Then navigate to http://localhost:8000/index.html
```

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Add to `spawnEnemy()` logic to determine when enemy appears (check `minStage`)
3. Add sound effect to `sfx` object for explosion (e.g., `sfx.explodeNewType`)
4. Add to `EXPLODE_SFX` mapping to play the correct sound when destroyed
5. In `updateGame()`, handle any special behavior (weave pattern, shooting pattern, etc.)

### Adding a New Boss
1. Add entry to `BOSSES` array with: `name`, `hullTop`, `hullBottom`, `wing`, `core` colors, `hpMul` multiplier
2. Add attack patterns to `BOSS_PATTERNS` array (array of functions that fire bullets)
3. Each pattern function receives boss object `b` and calls bullet-firing functions like `fireRadial()`, `fireAimedSpread()`, etc.
4. Pattern functions receive `b.patternIndex` to track which pattern is active
5. In `drawBoss()`, implement rendering using the boss's design colors

### Modifying Difficulty
Edit the `DIFFICULTIES` object to change spawn rates, enemy speed, lives, bombs, or boss health multiplier.

### Changing Visual Themes
The `THEMES` array controls sky gradient colors and cloud colors. Add new theme objects with `sky` (3-color gradient), `cloud`, and `sun` properties.

### Adding Sound Effects
Sounds use WebAudio synthesized via `beep()` (oscillators with sweeps) or `noiseBurst()` (noise with filtering). Add to `sfx` object:
```javascript
newEffect: () => {
  beep(freq, duration, type, volume, { options });
  noiseBurst(duration, volume, { options });
}
```

## Code Style & Patterns

- **Single IIFE**: Entire game runs in an immediately-invoked function expression for scope
- **Direct Canvas API**: No external game framework; uses 2D canvas context directly
- **Immediate Return**: Many early returns to skip logic when conditions aren't met
- **Object Pooling**: Not used; entities are created/destroyed directly
- **localStorage**: Used for persistence (best score, mute state, difficulty, reduced motion preference)
- **Delta Time**: Frame times tracked and averaged for adaptive quality; physics scaled by `dt`

## Important Constants

- `BASE_W = 480, BASE_H = 720`: Canvas resolution (internal); scales up to fill viewport
- `KILLS_PER_STAGE = 18`: Enemy kills needed to trigger boss spawn
- Storage keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
- Collision detection: Simple AABB via `rectsOverlap(a, b)` function
- Spawn timer: Base is difficulty-dependent; decreases as stage progresses for difficulty ramping

## Performance Considerations

- **Adaptive Quality**: Game monitors frame time; reduces rendering quality (`quality` variable) if FPS drops below 40
- **Object Cleanup**: Bullets, enemies, particles removed when off-screen or dead
- **Canvas Size**: Fixed internal resolution (480x720) scaled to fit viewport
- **Audio Context**: Created only on first interaction (lazy initialization via `ensureAudio()`)

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in vanilla HTML5/Canvas/JavaScript. It's a single-file application (`index.html`) with no external dependencies or build process.

### Running the Game

- Open `index.html` directly in a modern browser. That's it.
- Supports desktop (keyboard/mouse) and mobile (touch/joystick) controls.
- Game state (best score, difficulty, mute state, reduced motion preference) persists in `localStorage`.

## Architecture

The entire game logic lives in a single IIFE (Immediately Invoked Function Expression) in the `<script>` tag, roughly organized into these sections:

### 1. Initialization & DOM Elements
- Canvas setup (480×720 base resolution, scales responsively)
- UI element references (HUD, overlay, buttons, controls)
- Storage keys for persistence

### 2. Performance Tracking
- **Adaptive quality system**: Tracks frame times and auto-adjusts quality (0=full, 1=medium, 2=low)
- Reduces particle counts and shadow rendering at lower quality levels
- Code: `trackPerf(dt)`, `quality` variable

### 3. Game Configuration (Constants)
- **DIFFICULTIES**: Spawn rates, enemy speed, lives, bombs per difficulty
- **ENEMY_TYPES**: Stats for grunt, interceptor, tank (HP, speed, score, minStage)
- **BOSSES**: Visual designs, HP multipliers for 6 boss types
- **THEMES**: 6 visual themes with sky gradients, cloud colors, sun colors
- **BOSS_PATTERNS**: Attack patterns (radial, aimed spread, columns, spirals, etc.) per boss

These are the primary knobs for balancing the game.

### 4. Audio System
- **WebAudio synth** — no external audio files
- `beep(freq, dur, type, vol, opts)`: oscillator-based tones with frequency sweep, filter, pan
- `noiseBurst(dur, vol, opts)`: filtered noise
- `sfx` object: ~12 named effects (shoot, laser, explode, hit, bomb, powerup, bossHit, stageClear, gameover, etc.)
- Respects muted state and reduced-motion preference

### 5. Game State
- **Player**: position, size, speed, cooldown, weapon (vulcan/laser), level, invulnerability
- **Entities**: bullets (player), enemies, enemyBullets (enemy), particles, powerUps, boss
- **Session state**: score, lives, kills, stage, chain/multiplier, frame count
- Difficulty and settings stored in localStorage

### 6. Input Handling
- **Keyboard**: Arrow keys or WASD to move, Space/X/Shift for fire/bomb, P/Esc to pause
- **Mouse**: Hover over canvas to steer (smooth glide), click to fire
- **Touch**: Left half of screen is joystick for movement, right half has FIRE/BOMB buttons
- Input priority: Keyboard/joystick > mouse (so they don't fight)

### 7. Rendering Pipeline
- Sky gradient + animated clouds (parallax effect)
- Player plane (blue) with flame when moving
- Enemy planes (red/orange variants) with HP bars for tanks
- Player bullets (yellow/cyan) with glow
- Enemy bullets (red circles)
- Boss ship (colorful, animated core, telegraph state visualization)
- Power-ups (life/bomb/weapon, rotating for visibility)
- Particles (explosion effects, fade out)
- HUD overlay (score, best, multiplier, weapon, lives, bombs)
- Screen shake (applied via canvas transform)

### 8. Core Game Loop
```
loop(now) → trackPerf() → update(dt) → draw() → requestAnimationFrame(loop)
```

**Update** runs collision detection, enemy spawning, weapon fire, scoring, boss phases.
**Draw** renders all entities and HUD elements.

### 9. Boss System
- Boss spawns after 18 kills per stage (KILLS_PER_STAGE)
- 6 boss designs, cycling per stage
- HP scales by stage and difficulty
- **Attack phases**: telegraph telegraph (white flash) → fire pattern → cooldown
- **Patterns per boss**: Each boss has 3 unique attack patterns that cycle
  - VANGUARD: radial, aimed fan, V-fan
  - BEHEMOTH: columns, homing spread, dense radial
  - SPECTER: teleport+radial, spiral, double aimed
  - DREADNOUGHT: 8-way cross, homing missiles, radial wall
  - WRAITH: teleport+cross, double ring, cross-spread
  - TITAN: columns, dense radial, homing volley
- Damage: Player bullet = 1 HP (laser = 3), bomb = 24 HP fixed

### 10. Weapon System
- **VULCAN**: Spread increases per level (level 1→center, level 2→±12°, level 3→±24°)
- **LASER**: Lanes increase per level (level 1→center, level 2→±6 px, level 3→center+lanes), pierce enemies
- Power-ups cycle: weapon level 1→2→3→switch to other weapon at level 1
- Fire delay: Vulcan 0.15s, Laser 0.1s

### 11. Scoring & Multiplier
- Base scores: Grunt 10, Interceptor 15, Tank 35, Boss 500
- Chain timer: 2.2s without a kill to maintain multiplier
- Multiplier formula: 1 + floor(chain / 5), cap at 1..10x
- Bomb kills add 5 points each

### 12. Reduced Motion Support
- Checks `prefers-reduced-motion` on load
- When enabled: disables screen shake, flash, animations, vignette pulse
- User can toggle via 🎆 button

## Key Functions Reference

### Drawing
- `drawPlane(x, y, w, h, color, darkColor, flame)`: Player/enemy plane vector graphics
- `drawBossShip(b)`: Boss rendering with animated core and telegraph state
- `drawClouds(speed, fillColor, rx, ry)`: Parallax cloud layers
- `draw()`: Main render pass

### Game State
- `resetGame()`: Initialize all state for new game
- `endGame()`: Trigger game-over overlay and persist best score
- `togglePause()`: Pause/resume
- `setDifficulty(name)`: Set and persist difficulty

### Entities
- `spawnEnemy()`: Create random enemy with stage-appropriate type
- `spawnBoss()`: Trigger boss arrival after KILLS_PER_STAGE
- `spawnExplosion(x, y, color, count)`: Spawn particle cloud (quality-aware count)
- `applyPowerUp(type)`: Handle life/bomb/weapon pickup

### Damage & Scoring
- `addScore(base)`: Add to score with multiplier, update chain
- `breakChain()`: Reset multiplier (after taking damage)
- `loseLife()`: Decrement lives, invulnerability frames, check for game-over
- `killBoss()`: Handle boss defeat, advance stage, persist best score
- `useBomb()`: Clear enemy bullets, damage boss, damage all enemies

### Audio
- `ensureAudio()`: Lazy-init WebAudio context (browser requires user gesture first)
- `beep(freq, dur, type, vol, opts)`: Play tone with optional sweep/filter/pan
- `noiseBurst(dur, vol, opts)`: Play filtered noise burst

### Collision & Particles
- `rectsOverlap(a, b)`: AABB collision test (used for all hit detection)

### Boss AI (BOSS_PATTERNS)
- `fireRadial(b, count, speed, offset)`: N bullets in circle
- `fireAimedSpread(b, count, speed, spreadStep)`: Aimed toward player with spread
- `fireColumns(b, speed)`: 5 vertical columns
- `fireSpiralBurst(b, speed)`: 8-way radial with rotation offset
- `fireCross(b, speed)`: 8-way cardinal + diagonal
- `fireHomingSeeded(b, count, speed)`: Aimed with random scatter
- `fireDoubleRing(b, speedA, speedB)`: Two concentric radial rings

## How to Modify the Game

### Balance (Most Common Changes)

1. **Difficulty**: Edit `DIFFICULTIES` object
   - `spawnBase/spawnMin`: Frame count between enemy spawns
   - `enemySpeed`: Multiplier on enemy speed
   - `enemyShootChance`: Probability enemy will shoot (0–1)
   - `lives/bombs`: Starting inventory
   - `bossHpMul`: Boss health multiplier

2. **Enemy Stats**: Edit `ENEMY_TYPES`
   - `hp`: Hits to kill
   - `speedMul`: Speed relative to base difficulty speed
   - `score`: Points awarded
   - `minStage`: First stage to appear

3. **Boss Stats**: Edit `BOSSES`
   - `hpMul`: Health multiplier for that boss type
   - Adjust damage in `useBomb()` or bullet collision handler

4. **Scoring**: Adjust base scores in `ENEMY_TYPES` and `addScore()` call after kill

5. **Weapon Balance**: 
   - Adjust fire delay in `fireWeapon()` condition: `const fireDelay = ...`
   - Modify spread angles in `fireWeapon()` vulcan branch
   - Modify lanes in laser branch

### Visual Changes

1. **Colors**: Edit hex codes in CSS (`:root`, `#gameCanvas`, `.iconBtn`, etc.) or JS rendering
2. **Themes**: Edit `THEMES` array (sky gradients, cloud color, sun)
3. **Enemy/Boss appearance**: Modify `drawPlane()`, `drawBossShip()`
4. **Particles**: Change color in `spawnExplosion()`, `maybeSpawnPowerUp()`, `applyPowerUp()`

### Adding Content

1. **New boss**: Add to `BOSSES` array, add attack pattern(s) to `BOSS_PATTERNS` array in same order
2. **New enemy type**: Add to `ENEMY_TYPES`, filter by `minStage`, adjust spawn logic in `spawnEnemy()`
3. **New power-up**: Add type to `maybeSpawnPowerUp()` spawn chance, handle in `applyPowerUp()`, render in `draw()`
4. **New weapon**: Add to `fireWeapon()` branching, adjust in `applyPowerUp()` weapon switch logic

### Performance Tuning

- **Particle count**: `spawnExplosion()` uses `quality` to reduce count at lower FPS
- **Shadow rendering**: Glow effects applied only when `quality === 0`
- **Frame time tracking**: Adjust thresholds in `trackPerf()` if auto-scaling too aggressive

### Accessibility

- **Reduced motion**: Already respects system preference and user toggle; any animated properties should check this
- **Screen reader**: Minimal ARIA (status region) announces important events via `announce()` function
- **Touch controls**: Joystick appears automatically on touch devices; layout fills screen

## Code Style Notes

- IIFE prevents global namespace pollution
- Immediate numerical calculations rather than vectors (e.g., `b.x += b.vx * dt`)
- Collision is AABB (axis-aligned bounding box); no spatial partitioning
- No destructuring or ES6+ class syntax (vanilla JS, backwards-compatible)
- Comments are sparse; function names and logic are self-documenting

## Testing the Game

1. **Desktop**: Open in browser, use mouse or keyboard
2. **Mobile**: Open on phone/tablet, drag left zone to move, tap buttons
3. **Accessibility**: Enable reduced motion in OS settings or via 🎆 button
4. **Mute**: 🔊 button toggles audio
5. **Pause**: ⏸ button or P/Esc key
6. **Difficulty**: Choose at start screen (saved in localStorage)

## Browser Compatibility

- Requires Canvas 2D context
- WebAudio API (if unavailable, game runs silent)
- LocalStorage (fallback: settings lost on page reload)
- Touch Events (mobile controls degrade gracefully)

## Performance Characteristics

- 60 FPS target, degrades gracefully to 30 FPS
- ~50 particles per explosion at full quality
- No memory leaks; entities are culled off-screen
- Canvas is 480×720 logical, scaled to fit window

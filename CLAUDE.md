# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML/Canvas/JavaScript application. The game runs entirely in the browser with no external dependencies or build process.

- **File structure**: All code lives in `index.html` (~1670 lines)
- **Technologies**: HTML5 Canvas, WebAudio API, vanilla JavaScript
- **No build system**: Open `index.html` directly in a browser to play

## How to Run & Test

### Running the game
1. Open `index.html` in any modern web browser
2. Game loads immediately with the start menu
3. Use keyboard (Arrow keys/WASD + Space), mouse, or touch controls

### Testing changes
- **No build step required**: Any changes to `index.html` are live on browser refresh
- **Local development**: Use any local web server (e.g., `python -m http.server 8000`) to serve the file
- **Cross-device testing**: Use browser DevTools to test responsive design and touch controls
- **Performance testing**: Open DevTools Console to check for console errors; the game includes adaptive quality settings (check `quality` variable)

## Architecture & Key Systems

### Game State & Loop
- **Main loop**: `loop(now)` function at line 1640, runs via `requestAnimationFrame`
- **Update cycle**: `update(dt)` processes input, entity movement, collisions, and state changes
- **Render cycle**: `draw()` renders all game objects to canvas
- **Delta time management**: Capped at 50ms per frame (max 20 FPS floor) for consistency

### Core Game Objects
1. **Player** (`player` object): Aircraft at bottom of screen
   - Controlled by keyboard, mouse, or touch joystick
   - Two weapons: "vulcan" (spread bullets) and "laser" (piercing beams)
   - Invulnerability period after taking damage
   - Weapon levels 1-3 with progressive spread/pierce

2. **Enemies** (`enemies` array): Various aircraft types
   - `grunt`: Basic red enemy (1 HP)
   - `interceptor`: Fast orange enemy with weaving movement (1 HP)
   - `tank`: Slow purple heavily-armored enemy (4 HP)
   - Each type has unique color, speed, shoot patterns, and score values

3. **Boss** (`boss` object): Spawns after 18 enemy kills per stage
   - 6 unique boss designs (VANGUARD, BEHEMOTH, SPECTER, etc.) cycling per stage
   - Each boss has unique attack patterns (radial bursts, aimed spreads, homing, spirals, etc.)
   - Telegraph phase with visual/audio cues before attacks
   - HP scales with difficulty and stage number

4. **Bullets** (`bullets` array): Player projectiles
   - Vulcan: Simple straight bullets with optional spread
   - Laser: Piercing beams that hit multiple enemies

5. **Enemy Bullets** (`enemyBullets` array): Boss/enemy projectiles
   - Either homing or radial (fixed direction from boss)

6. **Power-ups** (`powerUps` array): Collectibles
   - `life`: Restore 1 health
   - `bomb`: Restore 1 bomb charge
   - `weapon`: Upgrade weapon level or switch weapon type

7. **Particles** (`particles` array): Explosion effects
   - Temporary visual feedback for destruction

### Key Constants & Configuration

**Difficulty Settings** (line 485-489)
```javascript
DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
};
```

**Visual Themes** (line 493-500): 6 sky themes with gradient colors cycling per stage

**Enemy Types** (line 502-506): Define stats like width, height, HP, speed multiplier, color, minimum stage appearance

**Bosses** (line 508-515): 6 boss designs with unique visual colors and HP multipliers

**Boss Attack Patterns** (line 1372-1409): Pattern arrays define boss attack behavior. Each pattern uses helper functions:
- `fireRadial()`: Circular bullet spray
- `fireAimedSpread()`: Spread aimed at player
- `fireColumns()`: Vertical streams
- `fireSpiralBurst()`: Rotating spiral
- `fireCross()`: 8-directional cross fire
- `fireHomingSeeded()`: Homing bullets with randomization
- `fireDoubleRing()`: Two concentric rings at different speeds

**Game Balance** (line 491)
- `KILLS_PER_STAGE = 18`: Enemies to defeat before boss spawns
- `BASE_W = 480, BASE_H = 720`: Canvas resolution (scales responsively)
- Spawn timers, bullet speeds, and difficulty curves control pacing

### UI & Visual Systems

**HUD Elements** (CSS at top of file)
- Score/Best/Multiplier display
- Weapon info (name + level)
- Hearts (❤️) for lives
- Bombs (💣) indicator
- Boss health bar (appears when boss spawns)
- Combo popup text
- Stage banner
- Top buttons: Mute, Motion effects toggle, Pause

**Visual Effects**
- Screen shake: `screenShake(magnitude, duration)` for impacts
- Flash effects: `flashHit()`, `flashBomb()` 
- Vignette: Red pulsing edge when at 1 HP (reduced motion respects `prefers-reduced-motion`)
- Combo popup: Shows at kill milestones (10, 20, 30 kills)

**Canvas Rendering**
- Sky gradient + sun glow + parallax clouds per theme
- Player plane with flame effect during thrust
- Enemy planes with damage indicators (tanks show HP bar)
- Boss ships with animated cores and wings
- Glowing bullets/particles with optional shadows (quality: 0 = full glow)

### Audio System (WebAudio Synthesis)

**No external audio files**: All sound effects are procedurally generated using WebAudio API.

**Sound Effect Functions** (line 624-676):
- `sfx.shoot()`, `sfx.laser()`: Weapon fire
- `sfx.explodeGrunt()`, `sfx.explodeInterceptor()`, `sfx.explodeTank()`: Enemy death
- `sfx.hit()`: Player damaged
- `sfx.bomb()`: Bomb detonation
- `sfx.powerup()`: Power-up collected
- `sfx.bossHit()`, `sfx.bossTelegraph()`, `sfx.bossDown()`: Boss interactions
- `sfx.stageClear()`: Stage complete
- `sfx.gameover()`: Game over

**Audio Details**
- Master gain: 0.9 with compressor (threshold -20dB, ratio 5)
- Spatial audio: `panFor()` uses panning based on screen X position
- Muted state persisted in localStorage
- Audio context created lazily on first interaction (iOS requirement)

### Input Handling

**Keyboard** (line 769-776)
- Arrow keys or WASD: Move player
- Space: Fire weapon
- X or Shift: Drop bomb
- P or Esc: Pause

**Mouse** (line 838-862)
- Move cursor over canvas to steer (smooth glide toward pointer)
- Click to fire

**Touch** (line 814-834)
- Left 55% of screen: Joystick for movement
- Right 45% of screen: FIRE and BOMB buttons
- Dynamic joystick UI appears while dragging

### Collision & Physics

**Collision detection**: `rectsOverlap(a, b)` at line 1069 uses AABB (axis-aligned bounding box)

**Physics**: Simple Euclidean movement
- Player: Keyboard/joystick direct movement OR mouse target pursuit
- Enemies: Linear vertical movement + optional horizontal weaving
- Bullets: Straight lines with optional velocity
- Boss: Bounces between screen edges at constant speed

### Scoring & Progression

**Combo System** (line 1087-1101)
- Chain: Incremented per enemy kill
- Multiplier: 1 + floor(chain / 5), capped at 10x
- Chain breaks if player doesn't kill for 2.2 seconds
- Score = baseScore × multiplier

**Enemy Scores**
- Grunt: 10 points
- Interceptor: 15 points
- Tank: 35 points
- Boss defeat: 500 points

**Stage Progression**
- 18 kills per stage → boss spawns
- 6 unique bosses cycle (stage 1→VANGUARD, stage 2→BEHEMOTH, etc.)
- Each stage increases enemy spawn rate and select boss HP
- Infinite stages possible

### Performance Optimization

**Adaptive Quality** (line 473-483)
- Tracks frame times over ~45 frames
- Quality levels: 0 (full, with glow), 1 (medium, half particles), 2 (low, 35% particles)
- Auto-degrades if avg frame time > 25ms; recovers if < 17.8ms

**Rendering Optimization**
- Particles/explosions spawned at quality-scaled counts
- Shadow blur and glow effects disabled at low quality
- Collision checks use simple AABB
- No spatial partitioning (acceptable for game scope)

### Persistence (localStorage)

Saved per browser/device:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Mute state
- `skyfighter.difficulty`: Selected difficulty
- `skyfighter.reducedMotion`: Motion effects preference

### Accessibility

**Screen Reader Support** (line 390, 468-471)
- `#srLive` with `aria-live="polite"` announces game state changes
- `announce()` function queues messages (30ms delay to ensure screen reader pickup)

**Reduced Motion Support** (line 349-358)
- Respects `prefers-reduced-motion: reduce` media query
- Toggle button (🎆) to manually control
- Animations disabled: shake, vignette pulse, combo pop, stage banner, flash

**Responsive Design**
- Canvas scales responsively but maintains 480×720 aspect ratio
- Safe area insets for notched devices (iPhone)
- Touch controls automatically enabled on touch devices

## Common Modifications

### Adding a new enemy type
1. Add to `ENEMY_TYPES` object (line 502-506): Define `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Update `spawnEnemy()` spawn probability logic (line 1028-1030) to include the new type
3. Optionally add explosion sound: Add entry to `EXPLODE_SFX` object (line 678) and define `sfx.explode[TypeName]()`

### Adding a new boss attack pattern
1. Create pattern function in the boss attack helper functions section (lines 1324-1370)
2. Add pattern array to `BOSS_PATTERNS` (line 1372+) at the index matching boss design order
3. Test by reaching that stage and observing the attack sequence

### Adjusting difficulty
1. Modify `DIFFICULTIES` object values (line 485-489)
2. Test by selecting difficulty and playing

### Adding visual theme
1. Add to `THEMES` array (line 493-500) with `sky` gradient colors, `cloud` color, `sun` color, and display `name`
2. Theme automatically cycles: stage N uses THEMES[(N-1) % THEMES.length]

### Adjusting weapon balance
1. Modify spread angles in `fireWeapon()` (line 1127, 1136)
2. Adjust fire delay at line 1177
3. Modify pierce/speed values in bullet objects

## Development Notes

- **No dependencies**: Game runs standalone with no npm/build
- **Single-threaded**: All code in one script block (IIFE at line 422)
- **Browser compatibility**: Works on modern browsers (IE11+ for WebAudio, or fallback gracefully)
- **Mobile-first**: Touch controls designed for phones; desktop controls secondary
- **Persistence**: All state (score, settings) saved to localStorage; survives page reload
- **Accessibility first**: Reduced motion, screen readers, safe areas respected
- **Code style**: Vanilla JS, camelCase, no frameworks or transpilation needed

## Known Limitations & Future Improvements

- No multiplayer support (single-player only)
- Boss patterns are fixed per difficulty (no dynamic adaptation)
- Limited to 6 bosses cycling (could expand with more designs)
- No leaderboard/cloud saving
- Touch controls on mobile are basic (no haptic feedback)
- Boss patterns use pseudo-random seeding for variety but could be more sophisticated

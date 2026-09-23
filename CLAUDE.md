# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up arcade game built as a single HTML file with embedded CSS and JavaScript. The game features:
- Progressive difficulty with 6+ stages
- Multiple bosses with complex attack patterns
- Weapon upgrade system (Vulcan/Laser with 3 levels)
- Smart bombs for panic moments
- Scoring system with combo multipliers
- Touch controls for mobile, keyboard/mouse for desktop
- Web Audio API for synthesized sound effects
- Particle effects, screen shake, and visual feedback

## Code Architecture

The entire game is contained in `index.html` (~1669 lines), structured as:

### Structure (order of appearance)
1. **HTML DOM** (lines 357–424): Canvas, UI overlays, HUD elements, touch controls
2. **CSS Styles** (lines 10–354): Game canvas, menus, HUD layout, animations (no external stylesheets)
3. **JavaScript** (lines 425–end): All game logic, no external scripts

### Key Systems

#### 1. Game Loop & State
- **`loop(now)`**: Main animation loop (requestAnimationFrame)
- **`update(dt)`**: Handles physics, collision, entity updates (called per frame)
- **`draw()`**: Canvas rendering (called per frame)
- **Game state**: `gameOver`, `paused`, `player`, `enemies`, `boss`, `powerUps`, `enemyBullets`, `particles`

#### 2. Player & Input
- **Player object**: `{x, y, w, h, hp, level, weapon, invuln}`
- **Input handling**:
  - Keyboard: `keys` object with WASD/Arrows for movement
  - Mouse: `mouseTarget` position driven by pointer events
  - Touch: Joystick (left side) + two action buttons (right side)
- **Movement**: Smooth follow toward target with bounded canvas clamping
- **Invulnerability**: 1.5s after taking damage (visual flashing)

#### 3. Weapons & Firing
- **Two weapon types**: `vulcan` (spread), `laser` (concentrated)
- **Three levels per weapon** (level 1–3), each with different fire rates and patterns
- **Firing patterns**:
  - **Vulcan**: Bullet spread originating from player
  - **Laser**: Narrow, high-speed beam (fires less frequently)
- **Power-ups restore**: Life, bomb count, or weapon/level progression
- **Bomb mechanics**: Limited count (max 3), full screen damage burst with vignette effect

#### 4. Enemy System
- **Types**: `grunt` (weak, slow), `interceptor` (fast, aimed), `tank` (tanky, radial fire)
- **Spawning**: 18 kills per stage → boss appears (KILLS_PER_STAGE = 18)
- **Life/score**: Each type has defined HP and point values
- **Enemy bullets**: Radial (omnidirectional) or straight (aimed at player)
- **Power-up drops**: Random on enemy death, 20% chance

#### 5. Boss System
- **6 boss designs**: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- **Attack patterns**: Each boss has 3 unique bullet patterns (stored in `BOSS_PATTERNS`)
  - Patterns include: radial bursts, aimed spreads, columns, spirals, homing missiles
- **Boss mechanics**:
  - Entry animation (top of screen → center)
  - Telegraph phase (22 frames) before attacking
  - Attack timer (85 frames) between patterns
  - Movement bounded horizontally with velocity reversal
  - HP bar UI tracking
- **Bullet firing functions**: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireHomingSeeded()`, etc.

#### 6. Collision & Scoring
- **Collision**: Simple AABB (axis-aligned bounding box) with `rectsOverlap(a, b)`
- **Player collision**: Loses life on enemy/bullet/boss contact
- **Projectile collision**: Player bullets destroy enemies/boss, enemy bullets damage player
- **Scoring**: Base points per kill + combo multiplier (increases per hit, breaks on miss)
- **Chain system**: Combo multiplier caps at ~150x, breaks on player damage

#### 7. Audio System
- **Web Audio synthesis** (no audio files; generated on-the-fly)
- **Functions**:
  - `beep(freq, dur, type, vol, opts)`: Sine/square/triangle tone
  - `noiseBurst(dur, vol, opts)`: White noise (for explosions)
- **SFX object**: Pre-defined effects (shoot, hit, bomb, boss telegraph, stage clear, etc.)
- **Audio context**: Created on first user interaction (`ensureAudio()`)
- **Panning**: `panFor(x)` stereo panning based on screen X position

#### 8. Visual Effects
- **Particles**: Dynamic spawn on explosions, moves with velocity decay
- **Screen shake**: `screenShake(mag, dur)` displaces canvas via transform
- **Flash effects**: Hit flash (#fff with fade) and bomb flash
- **Vignette**: Red radial gradient pulse during bomb
- **Stage banner**: Text overlay for stage transitions
- **Combo popup**: Floating text showing multiplier changes

#### 9. Difficulty & Settings
- **4 difficulty levels**: Easy, Normal, Hard, Lunatic
  - Affects: Enemy spawn rate, enemy count, difficulty (stat multiplier), boss HP
- **Persistent storage** (localStorage):
  - Best score, muted state, difficulty, motion preference
- **Reduced motion**: Disables animations if `prefers-reduced-motion` is set

#### 10. UI & Overlays
- **HUD** (in-game): Score, best, combo multiplier, weapon/level, lives (hearts), bombs
- **Boss bar**: Name + health bar (hidden until boss appears)
- **Main overlay**: Title screen, game over screen (shows score, stage, new best)
- **Difficulty selector**: Radio-button style buttons on title screen
- **Touch controls**: Two zones with visual joystick + action buttons
- **Pause overlay**: Simple pause menu

## Development Notes

### Key Variables & Constants
- **STORAGE_* keys**: localStorage prefixes (e.g., `skyfighter.bestScore`)
- **DIFFICULTIES**: Object mapping difficulty names to spawn/count/stat/boss HP multipliers
- **KILLS_PER_STAGE**: 18 enemy kills trigger boss spawn
- **THEMES**: Array of 6 stage themes with colors and names
- **ENEMY_TYPES**: Enemy definitions (type, hp, points, fire pattern)
- **BOSSES**: Boss definitions (width, height, max HP)
- **BOSS_PATTERNS**: Array of 3 attack pattern functions per boss design
- **BASE_W/BASE_H**: 480×720 game canvas size (responsive scaling)

### Common Edits
1. **Adjust difficulty**: Modify `DIFFICULTIES` object values
2. **Add boss pattern**: Create function in `BOSS_PATTERNS[bossIndex]` using firing helpers
3. **Tweak weapon fire rate/spread**: Edit `fireWeapon()` function (bullet spawn position/velocity)
4. **Change enemy stats**: Modify `ENEMY_TYPES` or spawn logic in `spawnEnemy()`
5. **Add sound effect**: Call `beep()` or `noiseBurst()` with desired parameters, or add to `sfx` object
6. **Adjust visual effects**: Modify particle spawn in `spawnExplosion()`, screen shake parameters, etc.

### Testing Notes
- Open `index.html` directly in a browser (no server required)
- Game saves best score to localStorage (persistent across sessions)
- Touch device detection: Sets joystick controls visible if touch events detected
- Audio context requires user gesture (click/tap to start game)
- Reduced motion support: Check browser DevTools → Rendering → Emulate CSS media feature

### Canvas Rendering
- Single 2D canvas context (`ctx`)
- Layers (bottom to top): Background clouds, enemies, player, bullets, particles, HUD, overlays
- No external graphics; all shapes drawn with canvas primitives (rect, arc, path)
- Responsive scaling: Canvas resizes on window resize via `resize()` function

## Performance Considerations
- Frame time tracking in `frameTimes` array for debugging
- Particle cleanup: Only live particles (`p.life > 0`) persist
- Enemy despawn: Bullets/enemies off-screen are removed
- Boss pattern culling: Only active boss fires bullets
- No major bottlenecks in current implementation; suitable for mobile browsers


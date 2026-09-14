# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The entire game (HTML, CSS, and JavaScript) is contained in `index.html` (~1670 lines). It runs in the browser with no external dependencies or build process.

### Key Features
- Canvas-based 2D rendering with procedural graphics (no image assets)
- WebAudio API synthesizer for sound (no audio files)
- Multiple enemy types with distinct behavior patterns
- Six boss encounters with unique attack patterns
- Difficulty levels (Easy, Normal, Hard) affecting enemy spawn rates and behavior
- Weapon system with two weapons (Vulcan spread-shot, Laser piercing shot) with three levels each
- Smart bomb mechanic that clears all enemies and deals boss damage
- Combo/kill-chain multiplier system
- Responsive touch and mouse controls
- LocalStorage for persistence (best score, sound mute state, difficulty, reduced motion preference)
- Progressive quality settings that adapt rendering based on frame rate

## Architecture & Game Flow

### Game Loop (lines 1640-1653)
The core game loop runs via `requestAnimationFrame` at ~60 FPS. Each frame:
1. Calculate delta time (capped at 50ms to handle frame drops)
2. Track performance for adaptive quality
3. Call `update(dt)` to advance game state
4. Call `draw()` to render the frame
5. Handle screen shake decay

### Core Game Systems

#### 1. Player System (lines 876-968)
- **Player object**: Position, size, speed (260 px/s), cooldown, weapon type/level, invulnerability frames
- **Input handling** (lines 768-861): Keyboard (WASD/Arrows), mouse pointer following, touch joystick
- **Firing mechanism** (lines 1124-1146):
  - Vulcan: Spread shot (1/2/3 projectiles depending on level) at 0.15s cooldown
  - Laser: Piercing shot (1/2/3 lanes at level 3) at 0.1s cooldown, can pierce through enemies
- **Movement**: Keyboard input has priority; mouse provides smooth steering when no keyboard input

#### 2. Enemy System (lines 502-506, 1022-1041, 1210-1228)
Three enemy types scale with stage progression:
- **Grunt** (red): 34x34, 1 HP, base speed, 10 points
- **Interceptor** (orange): 26x26, 1 HP, 1.7x speed, weaves, 15 points
- **Tank** (purple): 46x46, 4 HP, 0.55x speed, 35 points (appears stage 2+)

Enemies spawn with:
- Random spawn rate (affected by difficulty and frame count)
- One of two shoot patterns: spread (3-way fan) or aimed (homing toward player)
- Weave motion (interceptors only) using sine wave
- Collision detection triggers enemy death, explosion, score, and possible power-up

#### 3. Boss System (lines 508-515, 1043-1056, 1411-1439)
Six boss designs cycle through stages, each with:
- Unique color scheme and attack patterns
- HP scales by base (70 + stage×35) × difficulty × design multiplier
- Enter animation (moves from top to position)
- Attack phases: telegraph delay (22 frames), then pattern execution, 85-frame cooldown
- Pattern variety includes radial bursts, aimed spreads, columns, spirals, crosses, homing

**Boss patterns** (lines 1372-1409) are hardcoded per boss in `BOSS_PATTERNS`:
- Indexed by boss design (0-5 in BOSSES array)
- Each pattern is a function that fires bullets in different configurations
- Progression: enemies advance to boss after 18 kills per stage

#### 4. Collision System (lines 1069-1071, 1232-1295)
- AABB (axis-aligned bounding box) overlap detection
- Player-enemy: Takes damage, loses life
- Player-enemy bullet: Takes damage, loses life
- Player bullet-enemy: Enemy takes damage, dies at 0 HP
- Player bullet-boss: Boss takes 3 damage per hit
- Power-up pickup: Applies weapon/life/bomb effect
- Bomb: Clears all bullets and enemies, damages boss

#### 5. Particle & Effect System (lines 1073-1085, 1296-1297)
- Particles spawn on enemy death with velocity and lifetime
- Quality setting controls particle count (full/medium/low = 100%/60%/35%)
- Screen shake (managed separately, fades over time)
- Flash overlays (white flash for hits, bomb flash for explosions)
- Combo pop-up text (displayed at 10-kill intervals)

#### 6. Audio System (lines 542-676)
Pure WebAudio synthesis, no external files:
- **Oscillators** for tonal sounds (shoot, laser, boss hit, etc.)
- **Noise bursts** for percussive sounds (explosions, bomb)
- **Filtering** (lowpass/highpass) and panning (spatial audio)
- **Dynamic compression** on master gain to prevent clipping
- Each sound has frequency sweep, attack/release envelope
- Mutable (🔊/🔇), persists in localStorage

#### 7. Theme/Stage Progression (lines 493-500)
Six visual themes cycle through stages:
- Daylight Front → Sunset Corridor → Night Approach → Storm Front → Arctic Wastes → Volcanic Ridge
- Each theme defines sky gradient colors, cloud colors, sun glow
- Clouds drawn procedurally in two layers at different speeds
- Sun glow as radial gradient

#### 8. Score & Multiplier System (lines 1087-1101)
- Base score per enemy: grunt=10, interceptor=15, tank=35, boss=500
- Chain mechanic: Each kill increments chain counter, resets if >2.2s gap
- Multiplier = 1 + floor(chain/5), capped at 10x
- Score display updates in HUD
- Best score persisted in localStorage

### Game States
- **Overlay shown**: Start screen (before game), pause screen, game over
- **Running**: Active gameplay
- **Paused**: Game frozen, overlay visible, resume button shown
- **GameOver**: Final stats displayed, can start new game or change difficulty

## Development Notes

### Canvas Dimensions & Scaling
- **Base canvas**: 480×720 (portrait)
- **Dynamic scaling**: Limited to 1.6x max scale, fits within window
- **Responsive**: Resizes on window resize or orientation change

### Performance Optimization
- **Adaptive quality** (lines 474-483): Tracks average frame time over 45 frames
  - Quality 0 (full): Full shadows/glow effects
  - Quality 1 (medium): Reduced shadow blur, fewer particles (60%)
  - Quality 2 (low): Minimal effects, fewest particles (35%)
- **Object pooling**: Bullets and particles are managed directly (not pooled explicitly, but short-lived)
- **Filtering**: Off-screen entities culled (bullets, enemies, powerups)

### Difficulty Configuration (lines 485-489)
Each difficulty tier controls:
- Enemy spawn base delay and minimum delay
- Enemy movement speed multiplier
- Enemy shoot probability
- Player starting lives and bombs
- Boss HP multiplier

### Color Scheme
- Dark blue backgrounds for readability
- Bright accent colors for UI (yellow for score, red for health, green for weapon)
- High contrast for mobile
- Glow effects for power-ups and boss core (when quality allows)

## Common Modifications

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502-506) with w/h/hp/color/score
2. Add spawn condition in `spawnEnemy()` (line 1026-1030)
3. Add explosion SFX reference to `EXPLODE_SFX` (line 678) if custom sound needed

### Add a New Boss Attack Pattern
1. Add function to fire bullets (e.g., `fireCustom()` following pattern of lines 1324-1370)
2. Add pattern function to `BOSS_PATTERNS[bossIndex]` array (lines 1372-1409)
3. Pattern receives boss object and calls `enemyBullets.push()` with bullet objects

### Adjust Difficulty Settings
Edit `DIFFICULTIES` object (lines 485-489). All properties:
- `spawnBase`: Initial spawn delay (frames)
- `spawnMin`: Minimum spawn delay under difficulty ramp
- `enemySpeed`: Multiplier on enemy base speed
- `enemyShootChance`: Probability enemy shoots (0-1)
- `lives`: Starting player lives
- `bombs`: Starting bomb count
- `bossHpMul`: HP multiplier for bosses

### Modify Game Balance
- Player speed: `player.speed` (line 877)
- Fire cooldown: `fireDelay` (line 1177)
- Bullet speeds: Individual bullet `speed` properties
- Enemy spawn ramp: `difficultyRamp` calculation (line 1201)
- Multiplier formula: `multiplier = 1 + Math.min(9, Math.floor(chain / 5))` (line 1090)

### Tweak Visuals
- Canvas colors: Edit gradient definitions in `draw()` (lines 1529-1544)
- Particle decay: Adjust `p.life` decrement (line 1296) or particle initial count (line 1074)
- Animations: Frame-based; adjust animation constants (e.g., `frame * 0.05` for weave speed, line 1213)
- Shadow/glow: Controlled by `ctx.shadowBlur` and quality setting

## Testing & Debugging

### Debug via Browser Console
```javascript
// Advance to next stage
stage++;

// Spawn a boss immediately
spawnBoss();

// Kill current boss
if (boss) boss.hp = 0;

// Add score
score += 1000; scoreEl.textContent = score;

// Toggle pause
togglePause();
```

### Performance Issues
- Check `quality` variable (0=full, 1=medium, 2=low) — will degrade automatically
- Disable animations: Toggle reduced motion via UI button or:
  ```javascript
  reducedMotion = true;
  updateMotionBtn();
  ```
- Lower particle count: Edit quality thresholds in `spawnExplosion()` (line 1074)

### Sound Issues
- Game requires user gesture to initialize AudioContext (click/key pressed)
- Muted state persists in localStorage
- Check browser DevTools console for audio errors

## File Structure
- Single `index.html` file containing:
  - HTML structure (canvas, HUD elements, overlay UI)
  - CSS (responsive layout, animations, styling)
  - Vanilla JavaScript (game logic, rendering, audio synthesis)
  - Inline SVG favicon (data URI)

## No External Dependencies
- Pure Canvas API for rendering
- WebAudio API for synthesis
- LocalStorage for persistence
- No npm, no build tools, no external libraries
- Runs directly in any modern browser

---

**Last Updated**: 2026-09-14

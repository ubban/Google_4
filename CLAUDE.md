# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a vertical shoot-em-up game built in vanilla HTML5/Canvas/JavaScript. The entire game is contained in a single `index.html` file with no external dependencies — all styling and logic is inline. The game features multiple enemy types, boss encounters, difficulty levels, weapon upgrades, and adaptive performance tuning.

## Development & Deployment

### Running the Game

No build process is required. Simply open `index.html` in a web browser. The game works offline and stores persistent state (best score, settings) in `localStorage`.

- Desktop: Open directly in any modern browser
- Mobile: Works responsively on touch devices with gesture controls
- Local testing: `python -m http.server 8000` and navigate to `http://localhost:8000/index.html`

### File Structure

- `index.html` — Single-file game (1,670 lines)
- `README.md` — Minimal project info
- `.git/` — Version control

## Architecture & Key Systems

All code exists within an IIFE (Immediately Invoked Function Expression) at the bottom of `index.html` (lines 422–1666). The architecture is organized by functional systems:

### 1. State Management
- **Game State** (lines 864–869): `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- **Score/Lives** (lines 865–868): `score`, `lives`, `running`, `paused`, `gameOver`
- **Progression** (lines 867–868): `stage`, `stageKills`, `chain`, `multiplier`
- **Persistent Storage** (lines 517–523): Best score, mute state, difficulty, reduced-motion preference via localStorage

### 2. Input Handling
- **Keyboard** (lines 768–776): Arrow keys, WASD, Space, X/Shift (bomb), P/Escape (pause)
- **Mouse** (lines 836–861): Player follows mouse cursor when hovering canvas; click to fire
- **Touch** (lines 814–834): Joystick in left 55% of screen; FIRE and BOMB buttons in right 45%
- **Joystick Logic** (lines 785–812): Simulated analog stick with visual representation

### 3. Difficulty & Progression
- **DIFFICULTIES** (lines 485–489): Three presets (easy/normal/hard) with spawn rates, enemy speeds, HP, lives, bombs
- **THEMES** (lines 493–500): Six visual themes with sky gradients and colors, one per stage
- **BOSSES** (lines 508–515): Six named bosses with unique visual designs and stat multipliers
- **ENEMY_TYPES** (lines 502–506): Grunt, Interceptor, Tank with progressive unlocking per stage
- **Stage Progression**: Kill 18 enemies per stage → face boss → advance
- **Multiplier System** (lines 1087–1101): Combo chain increases score multiplier (max 10x every 5 kills)

### 4. Game Loop
- **Main Loop** (lines 1640–1653): `loop()` runs at 60 FPS via `requestAnimationFrame`
- **Frame Timing**: Delta-time capped at 50ms (line 1644) to prevent large jumps
- **Update Phase** (line 1648): `update(dt)` handles movement, spawning, collision, and state changes
- **Draw Phase** (line 1650): `draw()` renders the scene
- **Performance Tracking** (lines 476–483): Adaptive quality reduces particles/glow on low FPS

### 5. Rendering
- **Canvas Setup** (lines 752–765): Base 480×720, scaled to fit window with 1.6× max zoom
- **Background** (lines 1529–1544): Animated gradient sky + sun glow + layered moving clouds
- **Entities**: 
  - Player plane: `drawPlane()` (lines 913–968)
  - Enemies: Same plane function with different colors
  - Bosses: `drawBossShip()` (lines 970–1020) with unique designs per boss
- **Effects**: Screen shake, flash overlays, particle explosions, combo popups, stage banners

### 6. Audio (WebAudio Synthesis)
- **No External Assets**: All sounds are procedurally generated using Web Audio API
- **Master Gain & Compression** (lines 545–557): Compressor prevents clipping
- **Sound Functions** (lines 566–595):
  - `beep()`: Oscillator-based tones with optional frequency sweep, filter, panning
  - `noiseBurst()`: Filtered noise for impact sounds
- **SFX Library** (lines 624–676): 10+ sound effects for weapons, explosions, bosses, UI, game over
- **Panning** (lines 562–564): Bullets pan left/right based on X position for spatial audio

### 7. Collision & Damage
- **Rect Overlap** (lines 1069–1071): `rectsOverlap()` AABB collision detection
- **Bullet-Enemy**: Bullets pierce based on weapon type/level; tanks require multiple hits
- **Bullet-Boss**: 3 damage per bullet; bosses telegraph before attacking
- **Enemy Collision**: Enemies and boss damage player if not invulnerable
- **Invulnerability Window** (line 1459): 1.5 seconds after taking damage; player blinks

### 8. Weapons System
- **Vulcan** (lines 1126–1134): Spread increases with level (1 center → 3-wide at level 3)
- **Laser** (lines 1135–1145): More rounds per level; piercing and splitting into multiple lanes
- **Power-Up Drops** (lines 1058–1067): 6% life, 8% bomb, 20% weapon (from killed enemies)
- **Weapon Switching** (lines 1311–1320): Level 3 weapon swap triggers on next pickup

### 9. Boss System
- **Patterns** (lines 1324–1409): Each boss has 3 attack patterns (radial, aimed, special)
- **Entering Phase** (lines 1413–1417): Boss slides down from top before fighting
- **Telegraph** (lines 1421–1434): 22 frames of warning before boss fires (visual + audio cue)
- **Attack Timer** (lines 1429–1433): 85 frames between patterns
- **Bullet Types**: Radial (8-directional), aimed spreads, columns, spirals, crosses, homing
- **Stage Boss Selection** (line 1045): `BOSSES[(stage - 1) % 6]` cycles through 6 bosses

### 10. Particle System
- **Explosions** (lines 1073–1085): Burst of particles with random velocity; count scales with quality
- **Rendering** (lines 1628–1636): Particles fade out with life duration; size scales with remaining life
- **Performance**: Reduced particle count (60% at quality 1, 35% at quality 2) for low-end devices

### 11. UI & Accessibility
- **HUD** (lines 367–374): Score, best, multiplier, weapon level, hearts, bombs — spans top of screen
- **Boss Bar** (lines 376–379): Health bar appears only when boss is active
- **Overlays**: Title screen, pause screen, game-over screen with stats
- **Screen Reader** (line 451): `#srLive` announces important state changes (announce function, line 468)
- **Reduced Motion** (lines 520–540): Respects `prefers-reduced-motion` and disables shake/flash/animations

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502) with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Add explosion SFX to `EXPLODE_SFX` (line 678) if unique sound needed
3. Enemy appears in `spawnEnemy()` based on `minStage` check (line 1026)
4. Rendering handled by generic `drawPlane()` function

### Adding a New Boss
1. Add entry to `BOSSES` (line 508) with `name`, `hullTop`, `hullBottom`, `wing`, `core`, `hpMul`
2. Add 3 attack pattern functions (e.g., `fireRadial`, `fireAimedSpread`) to the appropriate index in `BOSS_PATTERNS` (line 1372)
3. Boss cycles through patterns in `updateBoss()` (line 1411); new boss will auto-select when stage number matches

### Adjusting Difficulty
Modify `DIFFICULTIES` (line 485):
- `spawnBase` / `spawnMin`: Enemy spawn intervals (lower = more frequent)
- `enemySpeed`: Multiplier for all enemy movement
- `enemyShootChance`: Probability enemies fire at player
- `lives` / `bombs`: Starting player resources
- `bossHpMul`: Boss health multiplier

### Tweaking Visual Appearance
- **Sky colors**: Modify `THEMES` RGB values (line 493)
- **Particle effects**: Adjust `spawnExplosion()` count or velocity (line 1073)
- **Screen shake**: Tune `screenShake(mag, dur)` calls throughout update and collision handlers
- **Quality settings**: Change glow thresholds or particle reduction in `quality` branches

### Adding a New Theme/Stage
1. Add new theme to `THEMES` array (line 493) with sky colors, cloud color, sun color, name
2. Themes cycle via `(stage - 1) % THEMES.length` — 6 themes loop indefinitely
3. Theme colors auto-apply in `draw()` (lines 1529–1544)

## Key Implementation Patterns

### Cooldown Tracking
```javascript
if (player.cooldown > 0) player.cooldown -= dt;
if (condition && player.cooldown <= 0) { action(); player.cooldown = delay; }
```
Used for fire rate, enemy shoot timers, invulnerability windows.

### Entity Pooling & Filtering
Bullets, enemies, particles are pushed/filtered rather than individually managed, reducing GC pressure.

### Lerp-based Movement
Player follows mouse cursor with velocity-based interpolation (line 1165–1169), allowing smooth catch-up without teleporting.

### Collision-Driven State
Collisions trigger state changes (lose life, spawn power-up) synchronously in the update loop rather than queued.

### LocalStorage Persistence
Settings saved immediately on change (e.g., line 537: `localStorage.setItem(STORAGE_REDUCED, reducedMotion ? '1' : '0')`)

## Performance Notes

- **Adaptive Quality**: `quality` variable (0=full, 1=medium, 2=low) auto-adjusts particle count and glow effects based on FPS
- **Canvas Resolution**: Fixed 480×720 logical size; CSS scaling handles viewport sizing
- **Draw Order**: Sky → clouds → player/enemies → bullets → particles → UI elements (no re-sorting needed)
- **Mobile Optimization**: Touch controls, reduced animations with `prefers-reduced-motion`, efficient collision checks

## Testing Checklist

When modifying game logic:
1. **Spawn behavior**: Verify enemy/boss spawn rates match difficulty
2. **Collision**: Check hit detection for close/overlapping objects
3. **Audio**: Confirm audio context initializes on first interaction (browsers require user gesture)
4. **Mobile**: Test touch controls on actual device (joystick, FIRE/BOMB buttons)
5. **Accessibility**: Test screen reader announcements for game over, stage clear, pause
6. **Responsive**: Verify canvas scales correctly across window sizes and orientations

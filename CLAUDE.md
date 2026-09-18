# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML, CSS, and JavaScript. The entire game runs in a single `index.html` file with no external dependencies. It's designed to be lightweight, performant, and playable in any modern web browser with support for canvas and Web Audio API.

## Quick Start

### Running the Game

The game runs directly as a static HTML file:

```bash
# Option 1: Open in a browser
open index.html

# Option 2: Start a local web server (Python 3)
python3 -m http.server 8000
# Then open http://localhost:8000

# Option 3: Using Node's http-server (if installed)
http-server
```

### Testing Changes

Since this is a single-file game:
1. Make changes to `index.html`
2. Save the file
3. Refresh the browser (Cmd+R or Ctrl+R) to see changes
4. Test the feature in-game

No build step, compilation, or package installation needed.

## Architecture Overview

The entire game logic is wrapped in an IIFE (Immediately Invoked Function Expression) starting at line 422. The architecture is organized into these major systems:

### 1. Game State Management
- **Core Variables** (lines 517-527): Best score, muted state, difficulty, reduced motion preference
- **Game State**: `running`, `paused`, `gameOver`, `frame` counter
- **Game Objects**: `player`, `enemies`, `boss`, `bullets`, `enemyBullets`, `powerUps`, `particles`

### 2. Difficulty System (lines 485-489)
- Three presets: `easy`, `normal`, `hard`
- Configures spawn rates, enemy speed, shoot chance, lives, bombs, and boss HP multiplier
- Difficulty can be changed at startup via UI buttons

### 3. Weapon System (lines 1124-1146)
- Two weapon types: **vulcan** (spread shot) and **laser** (piercing beams)
- Three level progression (1-3) that adds spread and lanes
- Weapons cycle when max level is reached with weapon powerup
- Sound effects use Web Audio beep synth with frequency sweeps

### 4. Enemy System (lines 502-506 and 1022-1041)
- **Three enemy types**:
  - `grunt` (red, 1 HP, basic) - always present
  - `interceptor` (orange, 1 HP, fast, weaves) - stage 1+
  - `tank` (purple, 4 HP, slow, heavy) - stage 2+
- Enemies spawn randomly within X bounds, move downward
- Two fire patterns: `spread` (fixed cone) and `aimed` (targets player)
- Spawn timer decreases based on difficulty and frame count (lines 1200-1203)
- **18 kills per stage** (KILLS_PER_STAGE) triggers boss spawn

### 5. Boss System (lines 508-515 and 1043-1056)
- **Six boss designs** (BOSSES array): VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Boss HP scales with stage, difficulty, and design multiplier (line 1046)
- Boss enters from top, paces horizontally, has invulnerability frames during entry
- Each boss has a pattern array with 3 distinct attack patterns that cycle
- Patterns include: radial bursts, aimed spreads, columns, spirals, homing projectiles, double rings (lines 1324-1409)
- **Telegraph system**: Boss glows/flashes 22 frames before attack (lines 1421-1434)
- Boss telegraph and attack animation tracked via `telegraph` and `patternIndex`

### 6. Collision & Physics
- **Collision Detection** (line 1069-1071): Simple AABB (Axis-Aligned Bounding Box) using center-distance
- **Bullet Behavior**:
  - `piercing` property determines how many enemies a bullet can pass through (laser-exclusive)
  - Radial bullets (boss projectiles) have explicit `dx`, `dy` velocity
- **Player Invulnerability** (line 1457): 1.5 second grace period after hit

### 7. Visual & Audio System
- **Canvas Drawing** (lines 1520-1669): Gradient sky, cloud layers, player/enemy rendering
- **Themes** (lines 493-499): Six theme sets with sky gradients and sun/cloud colors, cycling by stage
- **Web Audio** (lines 542-676): Procedural sound effects using oscillators, noise bursts, filters, envelope curves
- **Screen Shake** (line 713-725): Time-based shake with magnitude decay
- **Flash Effects** (lines 698-710): Hit flash (white), bomb flash (bright), vignette pulse (red)
- **Particle System** (lines 1073-1085): Explosion bursts with time-based life decay, quality-adjusted count
- **Adaptive Quality** (lines 474-483): Monitors frame time, adjusts quality from 0 (full) to 2 (low) to maintain 40 FPS

### 8. Game Loop & Timing
- **Main Loop** (lines 1670+): `requestAnimationFrame` based, calls `update()` then `draw()`
- **Delta Time** (dt): Calculated from RAF timing, passed to update functions
- **Frame Counter**: Incremented each frame for animation, phase, and timing purposes
- Time-sensitive systems use frame counter or explicit timers (chainTimer, spawnTimer, etc.)

### 9. UI & Accessibility
- **Screen Reader Support** (line 468-471): `#srLive` element for announcements
- **HUD Elements** (lines 435-461): Score, best score, multiplier, weapon info, hearts, bombs, boss bar
- **Overlay Screens**: Menu (start), pause, game over with difficulty selector
- **Touch Controls** (lines 299-341): Mobile joystick (left) and buttons (fire, bomb right)
- **Accessibility Features**: Reduced motion preference respects `prefers-reduced-motion`, mute toggle, screen reader announcements

### 10. Persistence
- LocalStorage keys (lines 463-466):
  - `skyfighter.bestScore`: High score
  - `skyfighter.muted`: Audio mute state
  - `skyfighter.difficulty`: Selected difficulty
  - `skyfighter.reducedMotion`: Accessibility preference

## Key Data Structures

### Player Object
```javascript
player = {
  x, y,              // Center position
  w, h,              // Dimensions
  speed,             // Movement speed in pixels/sec
  weapon,            // 'vulcan' or 'laser'
  level,             // 1-3
  invuln             // Invulnerability timer (seconds)
}
```

### Enemy Object
```javascript
{
  x, y, w, h,        // Position and size
  speed,             // Pixels per second downward
  type,              // 'grunt', 'interceptor', 'tank'
  hp,                // Current health
  canShoot,          // Boolean
  pattern,           // 'spread' or 'aimed'
  shootTimer,        // Countdown to next shot
  weave,             // Boolean - for interceptor horizontal movement
  baseX, weaveSeed   // Tracking for weave pattern
}
```

### Bullet Object
```javascript
{
  x, y, w, h,        // Position and size
  speed,             // Pixels per second
  vx,                // Horizontal velocity for enemy bullets
  pierce,            // Shots to survive (laser only)
  laser              // Boolean - indicates laser weapon
  // Radial bullets (boss) use: dx, dy, radial=true
}
```

### Boss Object
```javascript
boss = {
  x, y, w, h,        // Position
  hp, maxHp,         // Health
  entering,          // Boolean - entering animation state
  vx,                // Horizontal velocity for pacing
  design,            // Boss design object (BOSSES array)
  telegraph,         // Countdown before attack (22 max)
  attackTimer,       // Countdown to next telegraph
  patternIndex       // Which attack pattern to use
}
```

## Common Tasks & Code Locations

### Adding a New Enemy Type
1. Add to `ENEMY_TYPES` object (line 502-506)
2. Set `minStage` to control when it appears
3. Add spawn probability in `spawnEnemy()` (lines 1027-1030)
4. If it shoots differently, add pattern logic in enemy update section (lines 1217-1225)
5. Add explosion SFX mapping if needed (line 678)

### Adding a New Boss
1. Add to `BOSSES` array (lines 508-515) with colors and HP multiplier
2. Add attack pattern array to `BOSS_PATTERNS` (lines 1355-1409)
3. Patterns are functions that call `fireRadial()`, `fireAimedSpread()`, etc.
4. Boss selection cycles through array based on `(stage - 1) % BOSSES.length`

### Tweaking Difficulty
Modify the `DIFFICULTIES` object (lines 485-489):
- `spawnBase`: Base frames between enemy spawns
- `spawnMin`: Minimum spawn interval (floor)
- `enemySpeed`: Multiplier on enemy movement speed
- `enemyShootChance`: Probability enemy shoots when encountering player
- `lives`: Starting lives
- `bombs`: Starting bombs
- `bossHpMul`: Boss health multiplier

### Adjusting Game Balance
- **Enemy Spawn Rate**: Modify `spawnBase` and `spawnMin` in DIFFICULTIES
- **Boss HP**: Change base in `spawnBoss()` line 1046, or design's `hpMul`
- **Player Damage**: Change weapon pierce/levels or boss damage in `fireWeapon()` and boss collision (line 1259)
- **Weapon Progression**: Modify spread/lane logic in `fireWeapon()` (lines 1126-1145)

### Adding Visual Effects
- **Particle Bursts**: Call `spawnExplosion(x, y, color, count)` - count scales with quality
- **Screen Shake**: Call `screenShake(magnitude, duration)` 
- **Flash Effects**: Call `flashHit()` or `flashBomb()`
- **Vignette**: Toggles automatically on critical health

### Adding Sound Effects
1. Create a function in `sfx` object (lines 624-676)
2. Use `beep(freq, dur, type, vol, opts)` for tones
3. Use `noiseBurst(dur, vol, opts)` for percussion
4. Options: `sweepTo` (frequency ramp), `filterType`, `filterFreq`, `pan`, `delay`
5. All sounds check `muted` and `audioCtx` before playing

### Modifying HUD Display
- Score/multiplier: Update `scoreEl`, `multEl` (player updates elements when score changes)
- Weapon display: `weaponNameEl`, `weaponLvlEl` (updated in `applyPowerUp()`)
- Hearts/bombs: Call `renderHearts()` or `renderBombs()`
- Boss bar: Automatically shown/hidden by boss system

## Important Implementation Details

### Performance Optimization
The game adapts quality based on frame time (lines 474-483). Three quality levels:
- **Quality 0** (full): All particles, full glow effects, shadow blur on bosses
- **Quality 1** (medium): 60% of particles, reduced shadow blur
- **Quality 2** (low): 35% of particles, no shadow effects

Particle spawning uses quality multiplier (line 1074). This keeps 60 FPS on mobile.

### Audio Initialization
Web Audio context must be resumed after user interaction (line 560). The `ensureAudio()` function handles this. Audio is lazy-initialized on first input.

### Touch Control Detection
Touch controls are automatically enabled on touch devices. The joystick appears only when the user presses in the move zone (left 55% of screen). Fire and bomb buttons are always visible on mobile.

### Scoring System
- Kill chain tracked in `chain` variable, increments with every kill
- Multiplier = 1 + floor(chain / 5), caps at 10x
- Chain breaks on taking damage or timer expiring (chainTimer = 2.2s)
- Every 10 kills shows a combo popup (line 1248)

### State Transitions
- **Menu → Running**: `startBtn` click, sets `running = true`, clears variables
- **Running → Paused**: P key or pause button
- **Running → GameOver**: Lives reach 0, calls `endGame()`
- **Any → Menu**: Game over or pause screen, overlay becomes visible

## Debugging Tips

### Console Commands
Since everything is in an IIFE, direct variable access requires opening DevTools and attaching breakpoints. The following are useful:

- Check performance: `quality` variable shows current quality level
- Frame counter: `frame` increments each update
- Test sounds: Call `sfx.bossDown()` or any sfx function
- Monitor chain: Watch `chain` and `multiplier` variables

### Common Issues

**Lag/Frame Drops**: Check if quality is degrading. High particle count on low-end devices may need quality adjustment. Mobile use may benefit from setting quality = 2 by default.

**Audio Not Working**: 
1. Check if `muted` is true
2. Verify browser allows Web Audio (some older iOS versions don't)
3. Ensure user has interacted with page before audio plays

**Touch Controls Not Appearing**: Verify `touchControls` element is enabled and check `#moveZone` pointer-events is set to `auto`.

**Boss Not Spawning**: Check if `stageKills >= KILLS_PER_STAGE` is met (18 kills). Boss spawns ~1.2s after threshold via setTimeout.

## File Organization

Since this is a single HTML file, everything is organized by section with comments:

- **Lines 1-399**: HTML structure and CSS
- **Lines 401-420**: Game container and overlay
- **Lines 422-424**: IIFE wrapper and 'use strict'
- **Lines 463-676**: Constants, audio synth, settings
- **Lines 680-725**: UI rendering and visual effects
- **Lines 750-1020**: Drawing functions (player, enemies, bosses, particles)
- **Lines 1022-1350**: Game logic (spawning, collision, physics)
- **Lines 1355-1440**: Boss attack patterns and updates
- **Lines 1441-1670**: Game state (boss kill, lose life, game over)
- **Lines 1670+**: Rendering, event listeners, main game loop

## Testing Checklist

When making changes, verify:
- [ ] Game starts without console errors
- [ ] Player moves smoothly in all directions
- [ ] Weapons fire and damage enemies
- [ ] Enemies spawn with correct frequency for difficulty
- [ ] Bombs clear enemies and boss projectiles
- [ ] Boss spawns at 18 kills and attacks with patterns
- [ ] Screen shake and effects display correctly
- [ ] Sound effects play (if audio enabled)
- [ ] Score chain and multiplier work
- [ ] Game over triggers at 0 lives
- [ ] Touch controls work on mobile
- [ ] Pause/resume works
- [ ] High score persists in localStorage
- [ ] Difficulty selection affects gameplay
- [ ] Reduced motion toggle disables effects

## Performance Metrics

Target frame rate: 60 FPS (requestAnimationFrame)
Minimum acceptable: 40 FPS (before quality degrades)
Mobile baseline: Aim for stable 30 FPS with quality = 2

Particle budget: ~14 per explosion at quality 0, scales to ~5 at quality 2
Canvas size: Responsive, fits within viewport with aspect ratio maintained

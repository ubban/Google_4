# Sky Fighter - Codebase Guide

A Raiden-style vertical shoot-'em-up game built as a single-file HTML5 canvas application. All code is in `index.html`.

## Project Overview

**Sky Fighter** is a complete arcade-style game featuring:
- Player aircraft navigating through stages, collecting weapon upgrades
- 18 kills per stage to trigger boss encounters
- 6 unique boss types with distinct attack patterns
- Weapon system (Vulcan gun with spread fire, Laser with pierce)
- Smart bombs that clear all enemies and damage the boss
- Combo multiplier system for consecutive kills
- Multiple difficulty levels (Easy, Normal, Hard)
- Responsive design supporting mouse, keyboard, and touch controls
- Procedural audio synthesis (no external assets)
- Adaptive performance tuning based on frame time
- 6 different visual themes (sky colors/clouds)

## Architecture

### Single-File Structure
The entire game exists in `index.html` (~1670 lines):
- **Lines 10-359**: CSS (game canvas, HUD, overlays, animations)
- **Lines 422-1667**: JavaScript game logic in a self-contained IIFE

### Game Loop
- **`loop(now)`** (line 1640): Main animation frame callback
  - Calculates delta time (clamped to 50ms max)
  - Calls `update(dt)` for game logic
  - Calls `draw()` for rendering
  - Runs at ~60 FPS via `requestAnimationFrame`

### Key Game Systems

#### Input System (Lines 768-862)
- Keyboard: Arrow keys, WASD, Space (fire), X/Shift (bomb), P/Esc (pause)
- Mouse: Move to steer, click to fire (aim-assist toward pointer)
- Touch: Joystick on left side, Fire/Bomb buttons on right
- Each input type handled separately; keyboard/joystick take priority over mouse

#### Game State (Lines 863-869)
Primary state objects:
- `player`: Position, weapon, level, health, cooldown
- `enemies[]`: Array of active enemy objects
- `bullets[]`, `enemyBullets[]`: Player and enemy projectiles
- `boss`: Current boss object (null when not active)
- `powerUps[]`: Dropped weapon/life/bomb upgrades
- `particles[]`: Explosion and visual effect particles

#### Update Logic (Lines 1148-1302)
1. Player movement (keyboard/joystick/mouse steering)
2. Fire weapon (cooldown controlled, frame-rate independent)
3. Update bullets and remove off-screen projectiles
4. Spawn enemies based on difficulty timer
5. Update enemy AI and shooting patterns
6. Check boss spawn condition (18 kills per stage)
7. Collision detection: bullets ↔ enemies, enemies ↔ player, projectiles ↔ player
8. Power-up collection
9. Particle updates (fades over 0.5 seconds)
10. Boss HP display update

#### Rendering (Lines 1520-1638)
- Canvas size: 480×720 (logical), scaled to fit window
- 3-layer background: sky gradient + sun glow + clouds
- Draw order: sky, player, bullets, enemy bullets, enemies, boss, power-ups, particles
- Screen shake applied via `ctx.translate()` when damage taken/bomb used
- Visual quality degrades on slow devices (quality 0/1/2)

### Difficulty System (Lines 485-489)
Three difficulty presets control:
- `spawnBase` / `spawnMin`: Enemy spawn timing (frames)
- `enemySpeed`: Enemy movement multiplier
- `enemyShootChance`: Probability each enemy can shoot
- `lives` / `bombs`: Starting counts
- `bossHpMul`: Boss health multiplier

### Boss Patterns (Lines 1372-1409)
Each boss has 3 attack patterns, cycling continuously:
- **VANGUARD**: Radial burst, aimed fan, wide V-fan
- **BEHEMOTH**: Heavy columns, homing spread, dense radial
- **SPECTER**: Teleport dash, tight spiral, double aimed
- **DREADNOUGHT**: 8-way cross, homing missiles, radial wall
- **WRAITH**: Teleport barrage, expanding rings, cross-spread
- **TITAN**: Heavy columns, full ring, relentless homing

Boss patterns are functions that spawn enemy bullets; telegraph phase (22 frames) precedes each attack.

### Audio System (Lines 542-676)
- Uses Web Audio API synthesizer (no external audio files)
- Creates oscillators, filters, noise bursts, with frequency sweeps
- Individual effects for: shoot, laser, explosions (by type), hit, bomb, power-up, boss telegraph, boss death, stage clear, UI click, game over
- Spatial audio panning based on on-screen position
- Mutable via settings; survives page reload

### Performance Adaptation (Lines 474-483)
- Tracks last 45 frame times
- If average > 25ms (40 FPS): reduce quality (fewer particles, glow effects)
- If average < 17.9ms (56 FPS): improve quality
- Quality levels: 0 = full (glow, shadows), 1 = medium, 2 = low (particle reduction)

## Key Code Patterns

### Spawning Entities
```js
// Enemies spawn via spawnEnemy() (line 1022)
enemies.push({ x, y, w, h, speed, hp, type, pattern, ... })

// Bullets are instant: fireWeapon() (line 1124) pushes directly
bullets.push({ x, y, w, h, speed, vx, pierce, ... })

// Boss spawns at halfway through stage (KILLS_PER_STAGE = 18)
```

### Collision Detection
- Simple AABB: `rectsOverlap(a, b)` checks horizontal and vertical overlap
- Applied to: player↔enemies, bullets↔enemies, bullets↔boss, projectiles↔player
- Player invulnerability (1.5s after hit) blocks damage

### Scoring & Combo
- `addScore(base)` multiplies by `multiplier` (1–10×)
- Multiplier increases every 5 consecutive kills
- Chain breaks if 2.2s passes without a kill
- Bosses award flat 500 points when defeated

### Storage
Uses `localStorage` for persistence across sessions:
- `skyfighter.bestScore`: Highest score
- `skyfighter.muted`: Mute state (0 or 1)
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Accessibility setting

## Development Notes

### Modifying Difficulty
Edit `DIFFICULTIES` object (line 485-489). Each property scales game speed/challenge independently.

### Adding a Boss
1. Add entry to `BOSSES` array (line 508-515) with colors and hpMul
2. Add 3 attack patterns to `BOSS_PATTERNS` array (line 1372)
3. Patterns are functions receiving boss object, spawning bullets via fireRadial/fireAimedSpread/etc.

### Adding an Enemy Type
1. Add to `ENEMY_TYPES` (line 502-506): width, height, hp, speed multiplier, color, scoring
2. Update stage gate (`minStage`) if it should appear later
3. Spawn logic in `spawnEnemy()` (line 1022) automatically includes it

### Changing Visuals
- `THEMES` (line 493-500): Sky gradients, cloud/sun colors
- `drawPlane()` (line 913): Player and enemy aircraft shapes
- `drawBossShip()` (line 970): Boss visual assembly
- Adjust `canvas.width/height` and `BASE_W/BASE_H` (line 753) to change aspect ratio

### Performance Tuning
- Particle count scales with quality level (line 1074)
- Glow effects only at quality 0 (line 1579)
- Shadow blur adjusts with quality (line 1002)
- Disable shadows entirely: set `ctx.shadowBlur = 0`

## Testing Tips

- **Test Difficulty**: Edit `setDifficulty('hard')` call in resetGame or change default
- **Spawn Boss Immediately**: Set `KILLS_PER_STAGE = 0`
- **Infinite Lives**: Add `lives = 9999` to resetGame
- **Skip to Stage N**: Modify `stage` variable before spawnBoss
- **Mute All Audio**: Check localStorage or set `muted = true` at start
- **Reduced Motion**: Ensures animations are minimal for accessibility

## Known Patterns

### Frame-Rate Independence
All movement/timing uses delta-time (`dt`):
```js
object.y += object.speed * dt  // Always works at any FPS
```
Exception: Enemy spawn timer uses frame counts (line 1201), works fine since it's AI-facing.

### Canvas Sizing
Logical canvas (480×720) is separate from display size:
```js
canvas.width = 480  // Logical pixel space
canvas.style.width = scaled_px  // Display size (scaled to fit)
```
This allows high DPI scaling without redrawing logic.

### Input Priority
Order of precedence:
1. Keyboard/Joystick (always take control when active)
2. Mouse (only when no keyboard/joystick input)
3. Touch Fire button (direct toggle)

## Common Tasks

### Adjust Enemy Spawn Rate
Modify `d.spawnBase` and `d.spawnMin` in DIFFICULTIES.

### Change Boss Health
Adjust `baseHp` formula in `spawnBoss()` (line 1046) or difficulty's `bossHpMul`.

### Add Sound to Event
Call `sfx.<eventName>()`. If it doesn't exist, create a beep/noiseBurst function in the `sfx` object.

### Extend Stage Count
Bosses cycle via `(stage - 1) % BOSSES.length`. Add more BOSSES entries to extend variety.

### Modify Weapon Behavior
Edit `fireWeapon()` (line 1124) for Vulcan/Laser spread/pierce properties.

## Future Enhancement Ideas

- Leaderboard system (API integration)
- Score replay/recording
- Boss unlock system
- New weapon types (rapid-fire, ricochet, area damage)
- Enemy spawning patterns (formation flying)
- Power-up shop system
- Tutorial overlay on first play
- Mobile button layout customization
- Dual-stick controller support
- Speed run mode (time attack)

---

**Last Updated**: 2026-09-29  
**Game Complete**: Single HTML file, no dependencies, runs anywhere with a modern browser.

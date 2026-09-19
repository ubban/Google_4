# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical bullet-hell shoot-'em-up game built as a single-file HTML5 Canvas application. It features:
- Procedurally-spawned enemy waves with multiple enemy types
- Boss encounters at stage completions with unique attack patterns
- Dynamic weapon upgrade system (6 weapon types with progressive levels)
- Lives and smart bomb mechanics
- Difficulty selection (Easy, Normal, Hard)
- Touch and keyboard controls with adaptive joystick UI
- Procedural audio synthesis (no external audio assets)
- Adaptive performance scaling based on frame time tracking
- Full localStorage persistence for high scores and settings

## Architecture

### Single-File Structure (`index.html`)

All code is embedded in one 1669-line HTML file (lines 422+), organized into logical sections marked with `// ----------` comments:

#### **Core Systems**

1. **Adaptive Performance** (~lines 473–483)
   - Tracks average frame time over 45-frame window
   - Auto-scales rendering quality (0=full, 1=medium, 2=low) to maintain ~40–60 FPS
   - Quality is raised/lowered based on frame timing averages

2. **Audio System** (~lines 542–676)
   - WebAudio API synthesizer (no external files)
   - `beep()`: Oscillator with frequency sweep, filters, panning, gain envelope
   - `noiseBurst()`: Noise-based percussion and explosion effects
   - `sfx` object: Pre-tuned sound effects for all game events (shoot, laser, explode, hit, bomb, powerup, boss sounds, UI clicks, gameover)
   - Pan function maps canvas X position to stereo pan for spatial audio
   - Master gain with dynamics compressor for consistent loudness

3. **Input System** (~lines 767–861)
   - Keyboard support (Arrow keys + Space for fire/bomb)
   - Touch support with procedural joystick UI (shows on first touch)
   - `joystickStart()`, `joystickMove()`, `joystickEnd()`: Joystick state management
   - Converts mouse/touch events to normalized player input coordinates

4. **Game State** (~lines 863–1100+)
   - Player position, velocity, lives, bombs, score, combo multiplier
   - Active projectiles (player and enemy)
   - Active enemies and bosses
   - Current weapon type and level
   - Active explosions and visual effects
   - Stage/round counter (stage = 18 enemy kills)

5. **Rendering** (~lines 1507–1639)
   - `draw()`: Main render pipeline
     - Clear canvas with gradient sky
     - Render scrolling clouds
     - Draw all game entities (projectiles, enemies, player, boss)
     - Draw HUD (score, lives, bombs, weapon, multiplier)
     - Show special overlays (boss bar, combo pop, stage banner)
   - `drawPlane()`: Renders player/enemy ships with fill + shadow detail
   - `drawBossShip()`: Complex 6-color boss rendering with wings, hull, core
   - `drawClouds()`: Parallax background with perlin-like cloud pattern

6. **Game Loop** (~lines 1640–end)
   - `loop(now)`: RAF-driven main loop
   - Calculates delta time, calls `update()` then `draw()`
   - Tracks performance for adaptive quality

#### **Gameplay Systems**

1. **Enemy Spawning** (~lines 1022–1043)
   - Difficulty-based spawn intervals from `DIFFICULTIES` config
   - Three enemy types: `grunt` (1 HP, fast), `interceptor` (1 HP, very fast, weaving), `tank` (4 HP, slow, tanky)
   - Boss spawned after `KILLS_PER_STAGE` (18) enemy kills
   - Difficulty scales enemy count, speed, shoot chance, HP

2. **Weapons** (~lines 1124–1396)
   - 6 weapon types: Spray, Laser, Ring, Spiral, Cross, Homing
   - Each weapon has 5 progression levels (improves fire rate, spread, or homing strength)
   - `fireWeapon()`: Dispatches to weapon-specific fire function
   - Each fire function spawns projectiles with velocity/properties

3. **Boss System** (~lines 1411–1455)
   - 6 boss types: Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan
   - Each has unique:
     - Visual hull/wing/core colors
     - HP multiplier (affects difficulty scaling)
     - Attack pattern (telegraphed with sound, fires enemy projectiles in patterns)
   - Boss patterns rotate every ~1–2 seconds via time-based state machine
   - Render boss health bar at top of screen during fight
   - Boss defeat triggers stage clear SFX and score bonus

4. **Collision & Damage** (~lines 1069–1100+, within `update()`)
   - Rect-based AABB collision (`rectsOverlap()`)
   - Check player ↔ enemy, player ↔ boss, projectile ↔ enemy, projectile ↔ boss
   - Damage is 1 HP per hit; enemies die immediately; bosses accumulate
   - Visual feedback: screen shake, flash effect, hit SFX

5. **Power-ups** (~lines 1058–1068, 1304–1322)
   - Randomly spawn on enemy kills
   - Types: weapon upgrade, health restore, bomb restore, score multiplier boost
   - `applyPowerUp()`: Applies effect immediately

6. **UI & Settings**
   - Difficulty selection (Easy/Normal/Hard) with persistent localStorage
   - Mute toggle (persists on/off state)
   - Reduced motion toggle (checks `prefers-reduced-motion` media query, allows override)
   - Pause (P key, game logic frozen but rendering continues)
   - Screen shake & flash effects (skip if reduced motion enabled)

#### **Data Structures**

- `DIFFICULTIES`: Spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier per difficulty
- `THEMES`: 6 visual themes with sky gradient colors, cloud color, sun color
- `ENEMY_TYPES`: grunt, interceptor, tank properties (width, height, HP, speed multiplier, color, score value)
- `BOSSES`: 6 boss templates with color schemes and HP multipliers

### Performance Tuning

- **Adaptive Quality**: Rendered particle/effect count scales based on frame timing
- **Canvas Sizing**: Scales to fit screen while maintaining aspect ratio; uses CSS clamp() for responsive HUD
- **Audio**: Muting disables audio context entirely to save CPU
- **Collision**: Only checks active entities, no spatial partitioning (acceptable for ~50–100 projectiles + 20–30 enemies)

## Key Functions to Know

| Function | Purpose |
|----------|---------|
| `update(dt)` | Core game logic: move entities, handle input, check collisions, spawn enemies/bosses |
| `draw()` | Render everything from background to HUD |
| `loop(now)` | RAF callback, calls update + draw, handles performance tracking |
| `fireWeapon()` | Dispatch to active weapon's fire pattern |
| `updateBoss(dt)` | Boss state machine and attack pattern logic |
| `drawBossShip(b)` | Complex multi-part boss rendering |
| `spawnEnemy()` | Select random enemy type and place at top |
| `applyPowerUp(type)` | Apply collected power-up effect |
| `rectsOverlap(a, b)` | Simple AABB collision test |

## Common Tasks

### Testing/Running
- Open `index.html` directly in a modern browser (Chrome, Firefox, Safari, Edge)
- No build step, no dependencies, no server required
- Works offline and on mobile (with touch controls)

### Adding/Modifying Content

**Adding a new enemy type:**
1. Add entry to `ENEMY_TYPES` object (~line 502) with properties: `w, h, hp, speedMul, color, dark, score, minStage, weave` (optional)
2. Update `EXPLODE_SFX` mapping if custom explosion sound needed (~line 678)
3. Enemy spawning automatically picks from all types

**Adding a new weapon:**
1. Add a fire function (e.g., `fireNewWeapon(b)` ~line 1359+)
2. Add weapon name to weapon cycle logic (~line 1201 in `update()`)
3. Implement 5 progression levels in the fire function using `b.wpnLvl`

**Adding a new boss:**
1. Add to `BOSSES` array (~line 508) with color scheme and HP multiplier
2. Add attack pattern in `updateBoss()` (~line 1411) using time-based state machine

**Tuning Difficulty:**
- Edit `DIFFICULTIES` object (~line 485): adjust spawn rates, enemy speed, shoot chance
- Modify `KILLS_PER_STAGE` (~line 491) to change stage length
- Adjust `bossHpMul` to scale boss difficulty independently

### Debugging
- Check browser console (F12) for errors
- Game state is exposed globally; inspect `player`, `enemies`, `boss`, `score`, `chain` in console
- Use reduced motion toggle (🎆 button) to disable screen shake/flash if it obscures errors
- Performance: check frame times in console or use browser devtools profiler

### Visual Tweaks
- **Colors**: Theme gradients in `THEMES` (~line 493), enemy/boss colors in type definitions
- **Animations**: Keyframe timing in `<style>` section, particle counts in `draw()` and `update()`
- **HUD Layout**: Flex layout in CSS, clamp() for responsive sizing
- **Screen Shake**: `screenShake(mag, dur)` ~line 727; magnitude and duration parameters
- **Flash Effects**: `flashHit()`, `flashBomb()` ~lines 698–709 control flash opacity and timing

### Sound Design
- SFX tuned in `sfx` object (~line 624)
- Each effect uses `beep()` and/or `noiseBurst()` calls
- Customize by adjusting frequency ranges, sweep targets, filter frequencies, durations
- Panning uses `panFor()` to create stereo spatialization from projectile/enemy X position
- Mute state toggled via UI and persisted to localStorage

## Storage & Persistence

- **High Score**: `skyfighter.bestScore` (number)
- **Mute State**: `skyfighter.muted` ('0' or '1')
- **Difficulty**: `skyfighter.difficulty` (string: 'easy', 'normal', 'hard')
- **Reduced Motion**: `skyfighter.reducedMotion` ('0' or '1', defaults to system preference)

All values loaded on startup and synced on change.

## Browser Compatibility

- **Required**: Canvas 2D, WebAudio API, localStorage, requestAnimationFrame
- **Tested**: Chrome, Firefox, Safari, Edge (desktop and mobile)
- **Mobile**: Touch controls activate on first touch; supports gesture-free play
- **Accessibility**: Screen reader announcements via `srLive` element; reduced motion support

## Future Enhancement Ideas

- Multiple lives mode (continues) or endless scaling
- Leaderboard with server persistence
- Custom weapon creation UI
- Boss AI patterns procedurally generated from seed
- Replay system (record input, replay from seed)
- Mobile swipe gestures for weapon cycling

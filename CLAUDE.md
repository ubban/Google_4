# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 Canvas application. The entire game (HTML, CSS, and JavaScript) is contained in `index.html` (1669 lines).

## Development

### Running the Game
- Open `index.html` in a web browser (Chrome, Firefox, Safari, Edge)
- Game works on desktop (keyboard/mouse) and mobile (touch controls)
- No build process, server, or external assets required

### Key Technologies
- **Canvas API**: 2D rendering for all game graphics
- **Web Audio API**: Real-time synthesized sound effects (no audio files)
- **localStorage**: Persists best score, difficulty, mute state, and reduced motion preference
- **Touch/Pointer Events**: Mobile joystick and fire button controls
- **Adaptive Performance**: Auto-scales visual quality based on frame time

## Game Architecture

### Core Game Loop
Located around line 1580+, the game runs in a `requestAnimationFrame` loop (`gameLoop()`):
1. **Update phase**: Input handling, entity updates (player, enemies, bullets), collision detection
2. **Render phase**: Clear canvas, draw sky gradient, render all entities
3. **Performance tracking**: Monitor frame times and adjust quality level (0=full, 1=medium, 2=low)

### Entity System
All game entities are plain objects stored in arrays:
- **player**: Single player aircraft object with position, speed, weapon, health
- **enemies**: Array of enemy aircraft; three types defined in `ENEMY_TYPES`
- **bullets**: Player projectiles; includes velocity and lifespan
- **enemyBullets**: Enemy projectiles with basic tracking
- **particles**: Explosion/effect particles (visual only)
- **powerUps**: Dropped items (weapon upgrades, health, bombs)
- **boss**: Single boss entity (null when not active); spawned after 18 kills per stage

### Key Game Systems

#### Input Handling (lines 768-862)
- **Keyboard**: Arrow keys move, Space fires, X/Shift drops bombs, P/Escape toggles pause
- **Mouse**: Plane follows cursor on canvas, click to fire
- **Touch**: Virtual joystick (left) + fire button (right) on mobile
- Input system uses `keys` object and `firing`/`joyVec` flags; keyboard/joystick take priority over mouse steering

#### Audio System (lines 542-676)
- **Web Audio synth**: All sounds are synthesized, no external files
- Oscillators (sine, square, sawtooth, triangle) and noise bursts with filters
- Sound FX defined in `sfx` object: shoot, laser, explosions, hit, bomb, powerup, etc.
- **Pan positioning**: Sound effects use stereo panning based on screen position
- Muted state stored in localStorage

#### Weapon System
- Two weapons: **Vulcan** (rapid fire, line pattern) and **Laser** (slower, piercing)
- Each has 3 levels (upgraded by power-ups)
- Weapon level affects fire rate (`rapid`) and fire pattern
- Switch with W key (or cycle with collectibles)

#### Difficulty Levels (line 485-489)
Three presets (`DIFFICULTIES`) configurable per difficulty:
- `spawnBase`/`spawnMin`: Enemy spawn rate
- `enemySpeed`: Multiplier on enemy movement speed
- `enemyShootChance`: Probability each enemy fires per frame
- `lives`, `bombs`: Starting values
- `bossHpMul`: Boss health multiplier

#### Enemy Types (line 502-506)
- **grunt**: Basic enemy, slow, red
- **interceptor**: Fast, shoots, orange
- **tank**: Slow, tanky (4 HP), purple
- Each has color, HP, speed multiplier, score value, and minimum stage

#### Boss System (line 508-515)
- 6 unique boss types (Vanguard, Behemoth, Specter, etc.)
- Spawned after `KILLS_PER_STAGE` (18) kills
- Custom colors and HP multipliers
- Boss movement: follows player with warning telegraph
- Damaging boss updates health bar at top of screen

#### Scoring & Multiplier
- Score increases by kill value × difficulty multiplier (stages 2+)
- **Chain system**: Consecutive kills without pause build a multiplier (max 6×)
- Chain resets if gap between kills exceeds timeout
- Multiplier display on HUD, color changes when active

#### Rendering System
- **Canvas size**: Fixed 480×720 internal, scaled to fit window (max 1.6× via CSS)
- **Sky gradient**: Drawn from top 200px; changes per theme (6 themes defined)
- **Entity rendering**: Dedicated functions for player, enemies, bosses, bullets, particles
- **HUD overlays**: Canvas-relative fixed elements (score, health, weapon, boss bar)
- **Screen effects**: Flash on hit/bomb, vignette when critical health, screen shake on damage

### Key Variables & State
- **Game state**: `running`, `paused`, `gameOver`
- **Progression**: `stage`, `stageKills`, `kills`, `frame`
- **Player stats**: `score`, `lives`, `multiplier`, `chain`, `chainTimer`
- **Settings**: `difficulty`, `muted`, `reducedMotion`

## Important Implementation Details

### Collision Detection
- Enemies use AABB (axis-aligned bounding box) overlap
- Bullet-enemy: Check each bullet against all enemies
- Bullet-boss: Boss has custom polygon-based hit detection
- Player-enemy: Player box collision with auto-invulnerability frames

### Screen Shake & Vignette
- `screenShake(magnitude, duration)` called on damage
- Vignette (red pulse) activates when health = 1
- Both scale down if reduced motion is enabled

### Stage/Boss Progression
- Stages persist across boss defeats
- Boss appears after 18 normal kills
- Defeating boss: stage increases, kills reset, next boss randomly selected
- 6 bosses cycle through, so pattern can repeat across many stages

### Performance Optimization
- Adaptive quality system: Tracks average frame time over 45 frames
- If avg > 16.67ms (60 FPS target): Increase quality level
- If avg < 17.86ms (56 FPS target): Decrease quality level
- Quality affects particle count and effect density, not game logic

## Common Development Tasks

### Adding a New Enemy Type
1. Add type definition to `ENEMY_TYPES` object (around line 502)
2. Set `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
3. Add explosion sound mapping in `EXPLODE_SFX` object (line 678)
4. Add rendering case in `drawEnemy()` function

### Adding a New Boss
1. Add entry to `BOSSES` array (line 508)
2. Define colors (`hullTop`, `hullBottom`, `wing`, `core`) and `hpMul`
3. Boss rendering is handled by `drawBoss()`; adjust colors in that function if needed

### Adjusting Difficulty
1. Modify `DIFFICULTIES` object (line 485)
2. Tweak spawn rates, enemy speed, health multipliers as needed
3. Changes take effect immediately (no restart required if paused)

### Adding Sound Effects
1. Create function in `sfx` object (line 624+)
2. Use `beep(freq, duration, type, volume, opts)` or `noiseBurst(duration, volume, opts)`
3. Call via `sfx.effectName()`; mute check happens inside `beep`/`noiseBurst`

### Tweaking Visual Feedback
- **Flash effect**: `flashHit()` and `flashBomb()` toggle CSS class on `#flash` element
- **Screen shake**: Call `screenShake(magnitude, duration)` from update code
- **Combo/Banner text**: `showComboPop()` and `showStageBanner()` animate text overlays
- **Reduced motion**: `reducedMotion` flag scales duration and intensity of all effects

## File Structure
- `index.html`: Complete game (HTML + CSS + JavaScript)
- `README.md`: Minimal project info
- No external assets or dependencies

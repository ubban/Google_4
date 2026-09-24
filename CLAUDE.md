# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game written in vanilla HTML, CSS, and JavaScript. The entire game is contained in a single `index.html` file (~1700 lines) that runs in any modern web browser with no build process or dependencies.

## Running & Development

### Playing the Game
1. **Local development**: Open `index.html` directly in any modern web browser
2. **No build required** — the game runs as-is
3. **Mobile support**: Responsive design with touch controls (joystick on mobile, mouse on desktop)
4. **Controls**:
   - **Desktop**: Mouse to aim, `Z`/`Space` to shoot, `X` to drop bomb
   - **Mobile**: Touch joystick to move, tap near player to shoot, double-tap bombs

### Browser Compatibility
- Works in all modern browsers (Chrome, Firefox, Safari, Edge)
- Uses Web Audio API for sound (some browsers require user interaction to enable)
- Canvas 2D rendering
- Touch events for mobile

## Code Architecture

The entire game is in `index.html` with the following structure:

### Game Objects (Line 864)
- `player`: Player ship with position, health, weapons, bombs
- `bullets`: Array of projectiles fired by the player
- `enemies`: Array of active enemies
- `enemyBullets`: Projectiles from enemies
- `particles`: Visual effects (explosions, sparkles)
- `powerUps`: Health/bomb pickups that drop from enemies
- `boss`: Current boss object (null when not active)

### Core Game Loop
- **`loop(now)`** (Line 1640): RequestAnimationFrame loop that calls `update()` and `draw()` at ~60 FPS, tracks delta time
- **`update(dt)`** (Line 1148): Physics and logic — player movement, collision detection, weapon firing, enemy AI, score multiplier decay
- **`draw()`** (Line 1520): Rendering — background parallax, all game objects, UI/HUD, effects

### Key Game Systems

#### Player System
- **Position**: Center-bottom of screen, movable with mouse/touch
- **Health**: 3 lives by default (configurable by difficulty)
- **Weapons**: 4 weapon types (bullet, spread, laser, homing) that cycle with `Z`
- **Bombs**: Limited smart bombs that clear nearby bullets and damage enemies; regenerate one per stage
- **Damage feedback**: Screen flash, vignette effect, screen shake

#### Weapon System
- **fireWeapon()** (Line 1124): Creates bullets based on current weapon type
- **Weapon types** (DIFFICULTIES object, Line 485): Each difficulty sets weapon rate-of-fire and damage
- Weapons auto-cycle when all enemies cleared (momentum mechanic)

#### Enemy System
- **ENEMY_TYPES** (Line 502): 4 enemy types with different patterns and AI (grunt, interceptor, tank, fighter)
- **spawnEnemy()** (Line 1022): Picks random enemy, spawns at top with wave logic
- **updateBoss()** (Line 1411): Boss has phases, special attacks, beam patterns
- **Boss pattern**: Fires horizontal beams in patterns, moves around stage, spawns smaller bullets

#### Scoring & Multiplier
- **Base score**: Multiplied by current chain multiplier
- **Chain multiplier**: Increments with each hit, decays over time if player doesn't shoot
- **breakChain()** (Line 1096): Resets multiplier when conditions not met
- **addScore()** (Line 1087): Adds score and updates multiplier display

#### Collision Detection
- **rectsOverlap()** (Line 1069): Simple AABB collision check used for all hit detection
- Checked between: player↔enemy bullets, bullets↔enemies, player↔powerups, boss↔bullets

#### Audio System
- **Web Audio API**: Synthesized sound effects (no audio files)
- **beep()** (Line 566): Sine wave tones for weapon fire and UI
- **noiseBurst()** (Line 597): Noise bursts for explosions
- **panFor()** (Line 562): Stereo panning based on screen position
- Mute button in top-right corner persists to localStorage

#### Difficulty System
- **DIFFICULTIES** (Line 485): Defines enemy spawn rate, damage, frequency, player health
- 3 difficulty levels: Easy, Normal, Hard
- Selected at start screen, affects game balance

#### Visual Effects
- **Parallax clouds**: Background moves at different speeds
- **Particle system**: Explosions, debris, hit sparkles with rotation and fade
- **Screen shake**: Camera offset for impacts (controlled by `screenShake()`)
- **Vignette**: Red pulsing border during player damage (critical health indicator)
- **Flash effects**: White screen flash for hits, bomb explosions

### Drawing Functions
- **drawPlane()** (Line 913): Draws player/enemy ships with rotation, color, and flame effects
- **drawBossShip()** (Line 970): Draws multi-part boss with health bar
- **drawClouds()** (Line 1507): Parallax background clouds

## Important Implementation Details

### Game State Variables (Post-initialization)
- `running`: Whether game is active
- `paused`: Whether game is paused (pause button in top-right)
- `combo`: Current score multiplier (displayed in HUD)
- `score`/`stage`: Game progress tracking
- `difficulty`: Currently selected difficulty level
- `shakeTime`, `shakeX`, `shakeY`: Screen shake state

### Enemy Spawn Timing
- Enemies spawn in waves with increasing frequency as stage progresses
- Wave counter increments, triggers spawn rate increase every ~10 enemies
- Boss spawns after clearing enough enemies (stage-dependent)

### Weapon Cycling
- Weapons auto-cycle when all non-boss enemies cleared (auto-upgun mechanic)
- Player can manually cycle with `Z` key
- Each weapon has different fire rate (highest for spread, lowest for homing)

### Power-Up System
- Enemies drop either health (red square) or bomb (yellow square) on death
- Pickup triggers particle effect and audio feedback
- Health capped at max; bombs capped at 3

### Boss Mechanics
- Boss activates after defeating wave of enemies
- Has multiple attack patterns that cycle
- Fires horizontal beams in synchronized bursts
- Takes multiple hits to defeat (boss.hp tracked)
- Dropping boss ends stage and increments difficulty

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (Line 502) with: `pattern`, `fireRate`, `health`, `speed`, `color`, `size`
2. Add collision/AI logic in `update()` for the pattern
3. Add sound effect in `sfx` object (Line 624) if needed
4. Update `EXPLODE_SFX` (Line 678) if unique explosion sound

### Adding a New Weapon
1. Add to weapon list in player initialization (`resetGame()`, Line 875)
2. Add logic in `fireWeapon()` to generate bullets with the new weapon behavior
3. Adjust fire rate and damage in `DIFFICULTIES` (Line 485)

### Tuning Game Balance
- Enemy spawn rate: `DIFFICULTIES.Easy/Normal/Hard.spawnRate`
- Player damage: Adjust `bulletDamage` in `fireWeapon()` logic
- Enemy damage: Modify projectile speed and frequency
- Health/bomb counts: Set in `DIFFICULTIES`

### Audio Debugging
- Test without audio: Inspect mute button or check console for Web Audio errors
- Audio requires user interaction (Web Audio policy); `ensureAudio()` handles this
- Adjust volume parameters in `beep()` and `noiseBurst()` calls

### Performance Optimization
- `trackPerf()` (Line 476) logs FPS to console — watch for drops below 60
- Main bottleneck: Particle drawing in `draw()` (can be >1000 particles)
- Collision checks are O(n²) but manageable given typical entity counts
- Consider culling particles outside visible area if performance needed

## File Structure
```
Google_4/
├── index.html       (entire game: HTML structure + embedded CSS + JS)
├── README.md        (project timestamp/title)
└── .git/            (git repository)
```

## Notable Browser Features Used
- Canvas 2D context (drawing)
- RequestAnimationFrame (game loop)
- Web Audio API (procedural sound)
- Touch events (mobile controls)
- localStorage (mute state, high score persistence)
- window.devicePixelRatio (retina display handling)

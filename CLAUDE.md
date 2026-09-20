# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file web application. No build process or external dependencies required—open `index.html` in a browser to run.

## Architecture

The entire game runs in one HTML file (`index.html`) with embedded CSS and JavaScript (1670 lines). The architecture is monolithic and procedural rather than component-based, organized by functional sections.

### Core Systems

**Game State** — Defined by game objects and primitives:
- `player`: Position, velocity, weapon type/level, cooldown, invulnerability, health
- `enemies`, `enemyBullets`, `bullets`: Arrays of projectiles and enemy craft
- `boss`: Boss entity with health, movement, attack patterns, telegraph state
- `particles`, `powerUps`: Ephemeral objects for effects and collectibles
- `score`, `lives`, `kills`, `stage`, `chain`, `multiplier`: Game progression metrics
- `running`, `paused`, `gameOver`: State flags

**Rendering** — Canvas-based 2D graphics:
- `draw()`: Main render loop clearing canvas and drawing all entities
- `drawPlane()`: Aircraft sprite primitive (nose, wings, engine, canopy)
- `drawBossShip()`: Boss-specific visuals with design data (colors, core glow)
- `drawClouds()`: Parallax background elements
- Theme system: 6 sky gradients + color palettes swap per stage

**Input** — Three input modes work in priority order:
- Keyboard: Arrow keys or WASD for movement, Space/X/Shift for fire/bomb, P/Esc for pause
- Mouse: Pointer attracts player while hovering canvas; click to fire
- Touch: Virtual joystick on left (drag to move) + FIRE/BOMB buttons on right
- Gamepad-like behavior when keyboard/joystick is active (suppresses mouse steering)

**Audio** — WebAudio synth with no external sound files:
- `beep()`: Oscillator-based tones with frequency sweep and filters
- `noiseBurst()`: White-noise buffer with filter
- `sfx` object: Named sound effects (shoot, laser, explosions by type, powerup, boss sounds, UI click, game-over)
- Stereo panning based on projectile X position
- Mute toggle stored in localStorage

**Collision Detection** — Simple AABB:
- `rectsOverlap(a, b)`: Checks if two axis-aligned bounding boxes intersect
- Applied to: player ↔ enemies, player ↔ enemy bullets, player ↔ powerups, bullets ↔ enemies, bullets ↔ boss

**Difficulty System** — Three presets in `DIFFICULTIES` object:
- `easy`, `normal`, `hard`: Spawn rate, enemy speed, shoot chance, lives, bombs, boss health multiplier
- Applied during spawn and reset phases; setting persisted in localStorage

**Enemy & Boss System**:
- `ENEMY_TYPES`: grunt, interceptor, tank — each with HP, speed, color, score, stage gate
- Enemies spawn with random x, predetermined pattern (spread vs. aimed bullets)
- Weaving interceptors use sine-based movement
- `BOSSES`: 6 designs with unique colors and HP multipliers
- `BOSS_PATTERNS`: 6 arrays of attack functions (radial, aimed, columns, spirals, homing, etc.)
- Boss telegraph system: 22-frame wind-up before firing

**Power-ups**:
- Life (8% spawn chance), Bomb (8%), Weapon (20%)
- Weapon upgrades: Vulcan → Laser; within type, levels 1–3 → spread/double-fire
- Applied on collision; visual feedback and powerup sound

**Chain/Multiplier System**:
- Combo meter: Each kill increments chain; 2.2s decay timer
- Multiplier: 1 + floor(chain / 5), capped at 10x
- Used to boost base score of kills and bomb explosions

**Performance Adaptation** — Dynamic quality tuning:
- `trackPerf()`: Monitors frame time over 45 frames
- `quality` levels: 0 = full (glow shadows), 1 = medium (60% particles), 2 = low (35% particles)
- Adjusts shadow blur on boss core, particle spawn count, etc.

## Running the Game

1. **Play**: Open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge)
   - Tested on desktop and mobile (iOS Safari, Chrome)
2. **No build**: HTML, CSS, JS all embedded; localStorage used for save data
3. **Controls**:
   - Desktop: Arrow keys + Space/X/Shift, or mouse + click
   - Mobile: Virtual joystick + buttons

## Key Customization Points

**Game Balance**:
- `DIFFICULTIES`: Adjust spawn rates, enemy speeds, lives, bombs, boss health
- `KILLS_PER_STAGE`: Number of kills to trigger boss (default 18)
- Enemy type constants in `ENEMY_TYPES`: HP, speed multiplier, spawn weight
- Boss health in `spawnBoss()`: Base 70 + 35 × stage, then multiplied by difficulty/design

**Progression**:
- `THEMES`: 6 stage-cycling sky backgrounds (can add more)
- `BOSSES`: 6 boss designs; add to array to extend cycle beyond 6 stages
- Stage advance tied to boss kill; next stage increments stage number and resets kills counter

**Weapons**:
- Vulcan: Straight vertical bullets, spread increases with level
- Laser: Straight vertical, pierces, fires faster, spreads horizontally
- Toggle in `applyPowerUp()` when max level reached; weapon level resets to 1
- Fire cooldowns in `update()`: laser 0.1s, vulcan 0.15s

**Audio**:
- Mute button saves state; WebAudio synth calls can be disabled by setting `muted = true`
- Add new sounds by extending `sfx` object with beep/noiseBurst calls

**Accessibility**:
- `reducedMotion`: Disables screen shake, vignette pulse, flash animations, combo pop, etc.
- Respects `prefers-reduced-motion: reduce` media query on load
- Screen-reader live region (`#srLive`) for announcements

## Performance Notes

- Canvas size: 480×720 base, scales to window with max 1.6× multiplier
- Draw frequency: 60 FPS via `requestAnimationFrame`
- Particle count auto-reduces on slow frames
- Shadow blur disabled on boss at quality > 0
- Enemy bullet radial movement deferred to `updateBoss()`, not per-bullet physics

## Save Data (localStorage)

- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute flag (0/1)
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Reduced motion preference (0/1)

## Common Edits

**Add a stage-specific mechanic** (e.g., stage 3 spawns a new enemy type):
- Add enemy type to `ENEMY_TYPES`
- Set `minStage` gate; enemies spawn when `stage >= minStage`
- Optionally add explosion SFX to `EXPLODE_SFX` dict

**Adjust boss difficulty**:
- Scale `baseHp` in `spawnBoss()`
- Modify boss design's `hpMul` in `BOSSES` array
- Change attack pattern timing/count in `BOSS_PATTERNS`

**Add a new weapon type**:
- Add field to `player` state
- Handle in `weaponLabel()` and `fireWeapon()`
- Add power-up logic in `applyPowerUp()`

**Tweak visual polish**:
- Theme colors in `THEMES` (sky gradient, cloud, sun)
- Boss ship in `drawBossShip()`: stroke, fill, shadow, rotation
- Particle colors in `spawnExplosion()` calls
- Screen shake magnitude/duration in `screenShake()` and loseLife/bomb calls

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML application (index.html). The game features procedurally generated audio, difficulty tiers, weapon switching, smart bombs, score multipliers, and stage-based progression with boss encounters.

## Architecture Overview

The entire game runs in a single `(function() { ... })()` IIFE in the `<script>` tag of index.html (lines 423-1666). The game is organized around these core concepts:

### Game Loop
- **Main loop**: `loop(now)` function (line 1640) handles the game's heartbeat using `requestAnimationFrame`
- **Update cycle**: `update(dt)` (line 1148) processes game logic each frame
- **Render cycle**: `draw()` (line 1520) renders all game entities to the canvas
- **Frame timing**: Delta time (dt) is clamped to 0.05s to prevent spiral of death

### Game State Variables
- **Player**: `player` object containing position, dimensions, weapon, health, cooldown, invulnerability timer
- **Enemies**: `enemies` array of enemy ships spawned progressively through stages
- **Bullets**: `bullets` (player shots) and `enemyBullets` (enemy projectiles) arrays
- **Particles**: `particles` array for explosion/visual effects
- **Power-ups**: `powerUps` array for weapon upgrades and items
- **Boss**: `boss` object (spawned after KILLS_PER_STAGE enemies are destroyed)
- **Score/Lives**: `score`, `lives`, `maxLives` tracked for HUD and game over condition

### Key Entity Types

**Player** (line 877):
- Rectangular hitbox (40x40 by default)
- Two weapons: "vulcan" (rapid-fire) and "laser" (charged shots with aura)
- Fire cooldown and rapid-fire timer for weapon mechanics
- Invulnerability frames after taking damage
- Speed: 260 px/s, constrained to canvas bounds

**Enemies** (spawned by `spawnEnemy()`, line 1022):
- Various types: grunt, interceptor, tank (each with unique sprite drawing)
- Have spawn height, weaving patterns, and fire rates
- Drop power-ups on destruction
- Spawn in waves, increasing in difficulty per stage

**Bullets** (line 1180):
- Player bullets move upward at configurable speed
- Support laser and vulcan variants
- Have width/height for collision detection
- Laser bullets have visual aura effect

**Boss** (spawned by `spawnBoss()`, line 1043):
- Separate large ship with health bar and telegraph attack patterns
- Attacks: radial spread, aimed spread, column fire patterns
- Telegraphed attacks with warning effects
- Stage progression advances when boss is defeated

**Particles** (created by `spawnExplosion()`, line 1073):
- Simple velocity-based particles with life timers
- Used for explosion effects and visual feedback
- Auto-cleaned when life expires

### Audio System
Procedurally generated sound effects using Web Audio API (lines 545-676):
- **Base functions**: `beep()` (tone generation), `noiseBurst()` (noise burst)
- **Wave types**: sine, square, sawtooth, triangle
- **Effects**: frequency sweeps, filters, panning, envelope modulation
- **SFX catalog**: `sfx` object with named effects (shoot, laser, explode, powerup, bomb, etc.)
- **Master gain**: Controlled by mute button and `muted` state

### Input Handling
- **Keyboard**: Arrow keys for movement, Space for fire, Shift/X for bombs, P/Escape for pause (lines 769-776)
- **Mouse**: Click-to-move targeting with `mouseTarget` tracking (line 832)
- **Touch**: Virtual joystick for movement (left side) and fire/bomb buttons (right side) on touch devices (lines 778-827)

### Graphics & Rendering
- **Canvas size**: 480x720 (BASE_W/BASE_H), scaled with `resize()` to fit window (line 754)
- **Game entities**: Drawn with various canvas methods (fillRect, arc, custom path drawing)
- **Visual effects**: Screen shake, flash effects (white flash on hit), vignette (red pulse), combo/stage banners
- **Themes**: Dynamic sky gradients with clouds per stage (THEMES array, line 1529-1544)
- **Performance**: Three quality levels (full, medium, low) based on frame time tracking (line 468-521)

### Collision Detection
- **Rect overlap**: `rectsOverlap(a, b)` (line 1069) checks bounding box collisions
- **Player bullets vs enemies**: Checked in `update()` loop (line 1232-1256)
- **Player vs enemies**: Direct collision check (line 1268-1271)
- **Enemy bullets vs player**: Checked with invulnerability frames (line 1276-1288)
- **Power-ups vs player**: Collision detection and application (line 1289-1301)

### Game Progression
- **Stages**: Increment after defeating KILLS_PER_STAGE (18) enemies
- **Difficulty**: Four tiers (Easy/Normal/Hard/Brutal) controlled by `DIFFICULTIES` object
- **Boss fights**: Triggered after wave completion, requires defeating boss to advance stage
- **Score multiplier**: Increases with consecutive kills (combo system), resets on miss/damage
- **Weapon progression**: Limited to two weapons (vulcan/laser) with 3-level upgrade system

## Debugging and Development

### Key Game Constants
- `BASE_W = 480, BASE_H = 720`: Canvas dimensions
- `KILLS_PER_STAGE = 18`: Enemies to defeat before boss
- `DIFFICULTIES`: Object mapping difficulty names to spawn/fire rate parameters
- `THEMES`: Array of color schemes for each stage (sky colors, sun, clouds)

### Modifying Game Behavior
- **Enemy spawn rate**: Adjust `spawnTimer` threshold and `DIFFICULTIES` parameters
- **Player weapon speeds**: Change `player.speed` or `fireDelay` values
- **Enemy health**: Modify enemy spawn object properties
- **Boss patterns**: Edit `fireRadial()`, `fireAimedSpread()`, `fireColumns()` functions (lines 1324-1349)
- **Audio**: Modify `sfx` object or beep/noiseBurst parameters

### Testing a Change
1. Open index.html in a web browser
2. Make code changes directly in the HTML file
3. Refresh the browser to reload and test
4. Use browser DevTools console to monitor performance (frame times)
5. Check localStorage values (e.g., `localStorage.getItem('skyfighter.bestScore')`) for persistence

### Common Modifications
- **Adjust difficulty curves**: Modify `DIFFICULTIES` object with new spawn/fire parameters
- **Add new weapon types**: Create new firing pattern function and add case to weapon switch logic
- **New enemy types**: Add drawing function like `drawPlane()` and type case in `spawnEnemy()`
- **New visual effects**: Add particles via `spawnExplosion()` or create new animation in `draw()`
- **Audio tweaks**: Modify `sfx` properties or add new beep/noiseBurst combinations

## Performance Notes

- Game automatically adjusts quality level based on frame time tracking
- Particle/bullet cleanup happens each frame to prevent memory leaks
- Canvas is drawn fresh each frame with clearRect
- Web Audio API reuses buffer sources and gain nodes efficiently

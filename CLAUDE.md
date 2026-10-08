# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in a single HTML5 file with no external dependencies. The game features:
- Progressive difficulty stages with themed boss encounters
- Multiple weapon types that level up with power-ups
- Difficulty levels (Easy/Normal/Hard) with different enemy spawn rates and behaviors
- WebAudio-based procedural sound synthesis
- Touch-friendly controls with mouse/keyboard support
- Adaptive visual quality based on performance
- Accessibility features (reduced motion support, screen reader announcements)

## File Structure

The entire game is contained in **`index.html`** (~1669 lines):
- **Lines 10-358**: CSS styling (canvas, HUD, UI panels, animations)
- **Lines 361-421**: HTML structure (canvas, HUD elements, UI overlays)
- **Lines 422-1669**: JavaScript game logic (IIFE closure with all game code)

## Game Architecture

### High-Level Systems (Key Functions)

1. **Main Game Loop** (lines 1652-1665)
   - `requestAnimationFrame(loop)` drives the game at 60 FPS
   - `update(dt)` handles game logic, entity movement, collisions, spawning
   - `draw()` renders everything to canvas

2. **Entity Management** (arrays updated during loop)
   - `player` - player aircraft with position, weapon, health, invulnerability timer
   - `enemies` - array of active enemies (grunt, interceptor, tank types)
   - `bosses` - array of active bosses (6 unique boss types with themed patterns)
   - `bullets` - player projectiles with damage and laser tracking
   - `enemyBullets` - incoming fire from enemies
   - `explosions` - particle systems for visual feedback
   - `powerups` - collectibles that spawn after enemy kills

3. **Game State** (managed during reset/update)
   - `stage` - current level (affects enemy spawning and boss selection)
   - `lives`, `bombs` - player resources
   - `score`, `bestScore` - scoring and persistent high score (localStorage)
   - `weaponLvl`, `weapon` - current weapon type and upgrade level
   - `killCount` - progress toward stage completion (KILLS_PER_STAGE = 18)
   - `comboMult` - score multiplier for consecutive kills without getting hit

4. **Difficulty System** (DIFFICULTIES object, line 485-489)
   - Easy/Normal/Hard each define: spawn rates, enemy speed/accuracy, player lives/bombs, boss HP scaling
   - `resetGame()` applies selected difficulty on start

5. **Audio System** (lines 542-676)
   - `ensureAudio()` lazily initializes WebAudio context
   - `beep()` - oscillator-based tones with optional sweeps and filters
   - `noiseBurst()` - noise buffer source for explosion-like sounds
   - `sfx` object - named sound effect functions (shoot, explode, powerup, bossTelegraph, etc.)
   - All audio is procedurally generated; no external assets

6. **Input System** (lines 768-839)
   - Keyboard: arrows/WASD to move, Space to fire, X/Shift to bomb, P/Esc to pause
   - Mouse: move to steer, click to fire
   - Touch: left half is movement zone (virtual joystick), right half has FIRE/BOMB buttons
   - `mouseActive` flag gates mouse aiming when not in touchDevice mode

7. **Visual Effects & Rendering**
   - `draw()` (line 1520) - main render function with theme-based gradients and cloud parallax
   - `screenShake(mag, dur)` - camera shake that respects reduced motion preference
   - Flash overlays for hit/bomb effects
   - Vignette pulse when player health = 1
   - Sky themes cycle per stage from THEMES array (6 variants)

8. **Boss System** (BOSSES array, lines 508-515)
   - 6 boss types (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
   - Each has unique color scheme and HP multiplier
   - Boss attacks use varied fire patterns: radial, aimed spreads, columns, spirals, homing, etc.
   - `updateBoss(dt)` handles telegraph phase, attack execution, and damage

9. **Enemy Spawning & Progression**
   - `spawnEnemy()` (line 1022) picks random type based on minStage requirement
   - Spawn rate decreases as kills accumulate (spawn interval = `spawnBase - spawnMin * (killCount / KILLS_PER_STAGE)`)
   - At KILLS_PER_STAGE kills, boss spawns and stage increments

### Key Game Flow

1. **Game Start**: `resetGame()` initializes player, clears entities, sets difficulty params, shows stage banner
2. **Normal Play Loop**: `update()` moves entities, checks collisions, spawns enemies, applies physics
3. **Stage Completion**: Kill 18 enemies → boss spawns → defeat boss → stage++, repeat
4. **Game Over**: Player loses all lives → `endGame()` shows score card, allows restart

### Important Constants & Config

- **BASE_W = 480, BASE_H = 720** - game canvas size (scaled to fit window)
- **KILLS_PER_STAGE = 18** - enemies to defeat before boss appears
- **DIFFICULTIES** - easy/normal/hard parameters (spawn rates, speeds, health, lives)
- **THEMES** - 6 visual themes that cycle with stage progression
- **ENEMY_TYPES** - grunt, interceptor, tank (each with color, HP, speed, score value)
- **BOSSES** - 6 boss templates with color and HP multipliers

### Weapon System

Weapons level up independently:
- **Vulcan** (default) - fast, tight spread at player position
- **Laser** - narrow beam with blue aura, higher damage
- **Cannon** - slower, higher damage per shot
- Levels 1-5 increase fire rate/damage/behavior

Power-up spawning: 30% chance after enemy kill, grants one weapon level

### Performance Adaptation

`trackPerf(dt)` (line 476) monitors frame times and adjusts visual quality:
- `quality = 0` (full): glow effects, shadows on bullets
- `quality = 1` (medium): reduced glow
- `quality = 2` (low): minimal effects

Scales automatically if avg frame time exceeds 1/40 second.

## Development Notes

### Running the Game
- Open `index.html` in a modern browser (Firefox, Chrome, Safari, Edge)
- No build step, no dependencies, no server required
- Local storage used for best score, mute state, difficulty, reduced-motion preference

### Adding Features
- Keep the IIFE closure intact (lines 423-1669) to maintain variable scope
- Game state variables are declared early in the closure
- Add new entity types by following the pattern: define constants (ENEMY_TYPES, BOSSES, etc.), add update logic to `update()`, add rendering to `draw()`
- Audio: Add SFX to the `sfx` object using `beep()` or `noiseBurst()`
- Boss attacks: Create new fire pattern functions and call them from `updateBoss()`

### Testing
- Test all three difficulty levels (Easy/Normal/Hard)
- Verify touch controls on mobile/tablet (joystick + buttons)
- Check reduced-motion support doesn't break visuals
- Ensure performance doesn't degrade at high stage numbers (enemies accumulating)
- Audio edge cases: test on devices where WebAudio fails or is suspended

### Browser Compatibility
- Requires: HTML5 Canvas 2D, modern WebAudio API, localStorage, requestAnimationFrame
- Touch support detected via `'ontouchstart' in window` or `navigator.maxTouchPoints > 0`
- Reduced motion respects `prefers-reduced-motion: reduce` media query

### Debugging Tips
- `window.running` - true during active game, false in menu
- `window.paused` - pause state
- `stage`, `score`, `lives`, `player.weapon`, `killCount` - inspect game state in console
- Sprite collisions use `rectsOverlap()` (AABB)
- All entities are cleared on `resetGame()`, so memory should be stable

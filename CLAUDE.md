# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML/JavaScript application. The game features player ship combat, multiple weapon types, enemy waves, boss battles, scoring systems, and procedurally generated difficulty modulation.

## Architecture

The entire game runs in `/index.html` (1669 lines). It's organized as a single IIFE (Immediately Invoked Function Expression) containing:

### Core Game Loop
- `loop(now)`: RequestAnimationFrame-based main loop that calls update() then draw() each frame
- `update(dt)`: Game state updates (player movement, enemy AI, collision detection, spawning logic)
- `draw()`: Canvas rendering of all game entities

### Major Game Systems

**Player Ship**
- Position and velocity tracking
- Multiple weapon systems with leveling (vulcan, laser, spread, radiator, homing)
- Health/hearts system (0-3 lives)
- Bomb resource (0-3 max, cooldown-based)
- Weapon powerup collection and level progression

**Enemy System**
- Enemy types defined in `ENEMY_TYPES`: grunt, interceptor, tank
- Spawn logic in `spawnEnemy()` with stage-based difficulty progression
- AI: enemies move in patterns, fire bullets, and carry powerups on death
- Kill tracking per stage (18 kills triggers stage completion)

**Boss System**
- Boss types in `BOSSES` array (6 stages total)
- Boss spawning after stage completion
- Unique attack patterns per boss (radiating bullets, spreads, spirals, aimed patterns)
- Health bar UI rendering

**Projectile System**
- Player bullets: rectangles with laser/normal variants
- Enemy bullets: circles, spawned by enemy fire functions
- Homing bullets: track target enemies
- Collision detection using `rectsOverlap()` for all bullet/entity interactions

**Scoring & Progression**
- Base kill score + multiplier (multiplier resets on player death)
- Kill chain tracking (consecutive kills without death)
- Stage banners at progression milestones
- Best score persistence to localStorage

### UI & Effects
- HUD: score, best, multiplier, weapon level, health hearts, bomb count
- Status screen overlays (start, game over, pause)
- Combo pop animations
- Screen shake with magnitude/duration
- Red vignette pulse on critical health (1 heart)
- Flash effects on hit/bomb
- Reduced motion mode support (respects `prefers-reduced-motion`)

### Audio System
- Web Audio API synthesis-based SFX (no external audio files)
- Weapon fire beeps, explosions (noise bursts), impacts
- Dynamic panning based on bullet x position
- Mutable and persisted to localStorage
- Frequency/duration parameters for each sound type

### Input Handling
- Keyboard: arrow keys + spacebar (fire) + E (bomb)
- Mouse: target reticle, auto-fire on canvas
- Touch: on-screen joystick in lower-left + fire/bomb buttons
- Joystick state tracked in `joystickX/Y` variables

### Difficulty & Configuration
- Difficulty levels: easy, normal, hard (persisted to localStorage)
- `DIFFICULTIES` object controls enemy spawn rate, health scaling, fire rate
- Themes array for visual variety (colors cycle per stage)
- Reduced motion preference detection and toggle

## Key Constants & Configuration

```javascript
STORAGE_BEST: 'skyfighter.bestScore'
STORAGE_MUTE: 'skyfighter.muted'
STORAGE_DIFF: 'skyfighter.difficulty'
STORAGE_REDUCED: 'skyfighter.reducedMotion'

KILLS_PER_STAGE: 18  // Kills to complete stage and spawn boss
```

## Common Development Tasks

### Modifying Weapon Types
Edit the weapon firing functions (`fireWeapon()` calls specific fire pattern):
- `fireRadial()`: bullets in all directions
- `fireAimedSpread()`: bullets spread toward enemies
- `fireColumns()`: 2-3 vertical columns
- `fireSpiralBurst()`: rotating spiral pattern
- `fireCross()`: crosshair pattern
- `fireDoubleRing()`: inner/outer rings at different speeds
- `fireHomingSeeded()`: homing projectiles

### Adding Enemy Patterns
1. Add type to `ENEMY_TYPES` with hp, speed, color, fireRate
2. Modify `spawnEnemy()` logic to spawn on conditions
3. Add fire pattern calls in enemy fire logic within `update()`

### Adding Boss Patterns
1. Add boss definition to `BOSSES` array
2. Implement pattern in `updateBoss()` function (uses angle/phase for shooting)
3. Boss attack patterns use radial, aimed spread, or spiral firing

### Adjusting Difficulty
Edit `DIFFICULTIES` object:
- `spawnRate`: lower = more spawns per second
- `bossHpMult`: multiplier for boss health
- `enemyHpMult`: multiplier for enemy health
- `bulletFireRate`: enemy fire rate multiplier

### Canvas Rendering Customization
Enemy/boss drawing happens in `draw()`:
- `drawPlane()`: generic aircraft drawing (used for player and regular enemies)
- `drawBossShip()`: specialized boss renderer with animated turrets
- Shapes drawn with `ctx.fillRect()`, `ctx.arc()`, etc.

## Testing & Debug

No automated test suite. Manual testing approaches:
- Change difficulty during gameplay to test spawn/balance changes
- Toggle reduced motion (`motionBtn`) to verify animation behavior
- Mute audio and listen for console errors (audio initialization can fail on first click)
- Test touch controls on mobile viewport
- Check localStorage persistence by reloading after game

## Browser Features Required

- Canvas 2D context
- Web Audio API (synthesized audio generation)
- localStorage (best score, settings persistence)
- Touch events (mobile joystick)
- requestAnimationFrame
- matchMedia for reduced-motion detection

## Performance Considerations

- Canvas is resized on window resize via `resize()` function
- Frame time tracking in `frameTimes` array (used for FPS calculations)
- Particle/explosion count controlled in `spawnExplosion()` 
- Quality flag determines glow effects in rendering (can be toggled)
- Touch controls use `touch-action: none` to prevent browser defaults

## State Variables Overview

**Game State**
- `gameActive`, `paused`: game lifecycle flags
- `stage`, `stageKills`: progression tracking
- `score`, `scoreMult`, `scoreChain`: scoring
- `quality`: glow effects toggle

**Player State**
- `player`: {x, y, w, h, hp, maxHp, bombs, maxBombs, bombCooldown, weapon, weaponLevel}
- `mouseTarget`: {x, y} for reticle position

**Collections**
- `enemies[]`: active enemy entities
- `bullets[]`: player projectiles
- `enemyBullets[]`: enemy projectiles
- `powerUps[]`: dropped items
- `explosions[]`: particle effects
- `boss`: current boss or null

**Audio**
- `audioCtx`: Web Audio API context (initialized on first user interaction)
- `muted`: user preference
- `sfx`: object containing sound effect functions

## Future Enhancement Notes

- No module system or build process (single-file delivery)
- All styles inline in `<style>` tag
- HTML structure minimal (mostly just canvas)
- Touch UI elements added dynamically based on device detection
- Consider extracting weapon/enemy/boss definitions to separate config objects for easier iteration

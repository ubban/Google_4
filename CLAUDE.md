# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 canvas application. The entire game (~1670 lines) lives in `index.html` with embedded CSS and JavaScript. No external assets or dependencies—all graphics are drawn via canvas, and sound effects are synthesized with Web Audio API.

## Game Architecture

### Core Loop
The game runs via `requestAnimationFrame()` with a fixed update/render pattern:
1. **Update phase** (`update(dt)`): Moves entities, handles collisions, spawns enemies/bosses
2. **Render phase** (`draw()`): Clears canvas, draws sky gradient, all game objects, effects
3. **Performance tracking**: Adaptive quality scaling (0=full, 1=medium, 2=low) based on frame time averages

### Game State
Main entities managed in arrays:
- `player`: Single object tracking position, health, weapon, invulnerability
- `enemies[]`: Spawned dynamically; destroyed on kill or screen exit
- `bullets[]`: Player projectiles with pierce/laser properties
- `enemyBullets[]`: Tracked for collision with player
- `powerUps[]`: Spawned when enemies die (randomly: life, bomb, weapon)
- `particles[]`: Explosion effects, cleaned up per frame
- `boss`: Single active boss during stage progression (or null)

### Stage & Difficulty System
- **Stages**: Progress by defeating KILLS_PER_STAGE (18) regular enemies, then boss spawn
- **Difficulties**: Easy/Normal/Hard affect spawn rate, enemy speed, shoot chance, lives, boss HP
- **Bosses**: 6 unique designs with 3-pattern attack rotations each, cycled per stage
- **Themes**: 6 visual themes (sky gradients, clouds, sun colors) cycle per stage

### Weapons & Upgrades
- **Vulcan** (default): Spread shots that widen at level 2-3; fast fire rate
- **Laser**: Piercing high-speed shots that level up in count; toggles via power-up cycle
- Power-ups: Life restore, bomb refill, weapon level/toggle

### Input & Control
- **Keyboard**: Arrow keys / WASD to move, Space to fire, X/Shift for bomb, P/Esc to pause
- **Mouse**: Move to steer (glides toward pointer), click to fire
- **Touch**: Left zone for joystick movement, right buttons for FIRE/BOMB
- **Priority**: Keyboard/joystick input overrides mouse when active

### Collision & Physics
- Collision detection via `rectsOverlap()`: AABB check on all game objects
- Enemy weave pattern: Oscillates horizontally via sine wave during descent
- No gravity—all movement is velocity-based with explicit speed values

### Audio System
- **No external files**: All SFX synthesized via Web Audio API
- `beep()`: Oscillator-based tones with frequency sweeps, filters, panning
- `noiseBurst()`: White noise filtered for explosion/bomb effects
- Effects mapped in `sfx` object: shoot, laser, explode variants, hit, bomb, powerup, boss hits, etc.
- Muted state persisted in localStorage

### Persistence & Settings
- **localStorage keys**:
  - `skyfighter.bestScore`: High score
  - `skyfighter.muted`: Audio mute toggle
  - `skyfighter.difficulty`: Selected difficulty
  - `skyfighter.reducedMotion`: Motion effects toggle
- Reduced motion respects system `prefers-reduced-motion` media query on first load

## Development Tasks

### To Run / Test
No build step needed—open `index.html` directly in a browser. The game initializes and launches the start screen immediately.

### To Modify Game Balance
- **Difficulty parameters** (lines 485-489): `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`
- **Enemy spawn scaling** (line 1201): Difficulty ramp based on `frame / 300`
- **Enemy types** (lines 502-506): HP, speed, score, min stage requirement
- **Boss parameters** (lines 508-515): Color scheme, HP multiplier per design
- **Boss patterns** (lines 1372-1409): Attack functions per boss; patterns are called in sequence

### To Adjust Visuals
- **Canvas resolution** (line 753): `BASE_W = 480, BASE_H = 720` (scales to window up to 1.6x)
- **Plane drawing** (`drawPlane()`, line 913): Defines player/enemy ship geometry
- **Boss drawing** (`drawBossShip()`, line 970): Boss visual with animated elements
- **Themes** (lines 493-500): Sky gradients, cloud colors, sun colors per stage
- **UI colors**: HUD text, health bar, combo pop, stage banner all use inline hex colors

### To Add/Modify Sound Effects
Edit the `sfx` object (lines 624-676):
- Use `beep(freq, dur, type, vol, opts)` for pitched tones
- Use `noiseBurst(dur, vol, opts)` for noise-based effects
- Options: `sweepTo`, `filterType`, `filterFreq`, `pan`, `delay`
- Register new effects and call via `sfx.name()`

### To Change Weapon Mechanics
- **Vulcan spread** (lines 1126-1133): Adjust spread angles and bullet count per level
- **Laser lanes** (lines 1135-1145): Adjust lane offsets and pierce values per level
- **Fire rate** (line 1177): `fireDelay` is weapon-dependent; adjust values

### To Modify Boss Patterns
Boss patterns are functions in `BOSS_PATTERNS` array (lines 1372-1409):
- Each boss has an array of 3 attack functions
- Functions receive the boss object and call firing helpers:
  - `fireRadial()`: Bullets in circle
  - `fireAimedSpread()`: Spread toward player
  - `fireColumns()`: Vertical lanes
  - `fireSpiralBurst()`: 8-way spin
  - `fireCross()`: 8-direction cardinal+diagonal
  - `fireHomingSeeded()`: Scattered aimed shots
  - `fireDoubleRing()`: Two concentric rings
- Pattern selection rotates via `b.patternIndex % patterns.length` in `updateBoss()` (line 1426)

### To Add New Enemy Type
1. Add entry to `ENEMY_TYPES` object (lines 502-506) with properties: w, h, hp, speedMul, color, dark, score, minStage, (optional) weave
2. Add explosion SFX in `EXPLODE_SFX` (line 678) if using custom sound
3. Optional: Add to `spawnEnemy()` (lines 1028-1030) selection logic if not spawning from stage 1

### To Add Accessibility / Motion Preferences
- Reduced motion applied to animations via `body.reduced-motion` CSS class (lines 349-358)
- Check `reducedMotion` variable in code before applying shake/flash effects
- Screen reader announcements via `announce()` (lines 468-471) which updates `#srLive` element
- Use `.sr-only` class for hidden text (lines 343-347)

## Code Patterns & Conventions

### Drawing with Canvas Context
All drawing functions use the global `ctx` (canvas 2D context):
- `ctx.save()`/`ctx.restore()` to isolate state changes
- Transforms (translate, rotate) used for sprite rotation and positioning
- Gradients for backgrounds and visual variety
- `ctx.globalAlpha` for fade effects

### Animation & Timing
- **Frame counter**: `frame` increments every update, used for cyclic animations (frame % N)
- **Time-based** (`dt`): Updates use delta time for smooth motion independent of frame rate
- **CSS animations**: Define via `@keyframes` (e.g., `heartBeat`, `comboPulse`) for UI elements
- **Triggers**: Via class toggling (`.show`, `.on`, `.active`) or timer callbacks

### Event Handlers
- Keyboard: `keydown` / `keyup` toggle flags in `keys` object
- Mouse: `mousemove` updates `mouseTarget`, `mousedown` starts firing
- Touch: Multi-touch via `e.changedTouches`, each touch tracked by `identifier`
- Buttons: Click handlers for UI (start, difficulty, mute, pause)

### Game State Management
- `running`: Gameplay active (false during menu/game over/pause)
- `paused`: Mid-game pause state; update skipped but render continues for pause screen
- `gameOver`: Terminal state; no update/reset until "Play Again" clicked
- `overlay.hidden`: Toggle to show/hide menu, pause, game-over screens

## Common Development Workflows

### Testing a Game Balance Change
1. Modify difficulty values or spawn rates (see "Game Balance" above)
2. Refresh browser; change takes effect immediately
3. Test across difficulty levels using difficulty buttons on start screen
4. Check if changes apply retroactively (they do, via difficulty lookup each frame)

### Debugging Enemy/Boss Behavior
- Add console.log in `spawnEnemy()` to verify enemy properties
- Check `updateBoss()` for telegraph timing and pattern execution
- Inspect `enemyBullets` array in DevTools to see fired projectiles
- Frame-by-frame via debugger breakpoint in game loop

### Adjusting Visual Polish
- Screen shake: Modify `screenShake(mag, dur)` calls (e.g., line 1108, 1443)
- Flash effects: Controlled by `flashHit()` and `flashBomb()` (lines 698-709)
- Particle explosions: Adjust color and count in `spawnExplosion()` calls
- Vignette (red edge effect): Only shows when critical health; toggle via `vignetteEl.classList`

### Adding UI Elements
- HUD stats: Add div to `#hud` (line 34 in HTML), create element reference, update on game events
- Overlays: Modify `#overlay` structure (line 401); control visibility via `overlay.hidden`
- Accessibility: Add text to `#srLive` via `announce()` for screen reader feedback

## Performance Considerations

- **Quality levels**: Adaptive scaling reduces particle count, shadow effects at quality 1-2
- **Collision checks**: O(n²) for enemy bullets vs player; could optimize via spatial partitioning if needed
- **Drawing order**: Sky → clouds → player → bullets → enemies → boss → powerups → particles
- **Disabled features on low quality**: `glow` variable controls shadow/blur effects; reduces at quality > 0
- **Touch controls**: Hidden by default; only shown on touch devices via feature detect

## Persistence & State Recovery

The game does NOT auto-save mid-game state. Only high score and settings are persisted:
- On new game: `resetGame()` reinitializes all entities, score, lives
- On game over: Score and stage are shown but not saved (only if new best)
- Settings survive browser close via localStorage

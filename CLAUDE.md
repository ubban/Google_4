# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The game is fully self-contained with no external dependencies, using Canvas for graphics and WebAudio API for procedurally generated sound effects.

## Game Architecture

### Overall Structure

The entire game engine is contained in `index.html` (1669 lines total):
- **Lines 1-421**: HTML markup and CSS styling (dark theme, responsive layout, UI components)
- **Lines 422-1667**: JavaScript game engine (IIFE pattern for scope isolation)

### Core Game Systems

#### 1. Entity Management
The game maintains five main entity arrays:
- `player`: Single player aircraft with position, velocity, health, weapons, invulnerability timer
- `bullets`: Player projectiles (varies by weapon type)
- `enemies`: Spawned enemy aircraft (three types with different behaviors)
- `enemyBullets`: Enemy projectiles
- `particles`: Visual effects (explosions, hits, score popups)
- `powerUps`: Dropped items for weapon/health upgrades
- `boss`: Current boss entity (or null when not active)

#### 2. Game Loop (`loop()` at line 1640)
Implements a delta-time based update loop with requestAnimationFrame:
1. Calculate delta time (`dt`) capped at 50ms to prevent large jumps
2. Call `trackPerf(dt)` for adaptive quality adjustment
3. If running and not paused: call `update(dt)` then `draw()`
4. Schedule next frame with requestAnimationFrame

#### 3. Update Logic (`update()` at line 1148)
Handles all game logic per frame:
- Player movement (keyboard/mouse/touch) and collision detection
- Player weapon firing and cooldown management
- Enemy spawning based on difficulty and stage progression
- Enemy movement, rotation, and AI behavior
- Collision detection (player bullets vs enemies, enemy bullets vs player)
- Enemy death handling with score calculation and power-up drops
- Combo multiplier system (chain = consecutive kills within 1 second)
- Stage progression (18 kills to advance stage and spawn boss)
- Boss spawning and battle logic (`updateBoss()` at line 1411)
- Particle and power-up updates
- Game over detection

#### 4. Rendering (`draw()`)
Canvas rendering pipeline:
- Clear canvas and apply screen shake offset
- Draw background with gradient and theme colors
- Draw all entities: player, enemies, boss, bullets, particles, power-ups
- Draw HUD (score, lives, multiplier, weapon name/level, boss health bar)
- Apply visual effects (vignette on hit, white flash on damage)

### Configuration Objects

These dictionaries define game content and balance:

**`DIFFICULTIES`** (line 485): Three modes control enemy behavior and player resources
- `spawnBase`/`spawnMin`: Frame timers controlling enemy spawn frequency
- `enemySpeed`/`enemyShootChance`: Enemy AI difficulty
- `lives`/`bombs`: Player starting resources
- `bossHpMul`: Boss health multiplier relative to base value

**`THEMES`** (line 493): Six visual themes for progression
- Each theme provides: sky gradient colors, cloud color, sun color, display name
- Rotates automatically as player progresses through stages

**`ENEMY_TYPES`** (line 502): Three enemy variants
- `w`/`h`: Collision and render dimensions
- `hp`: Health points (tanks have 4, others have 1)
- `speedMul`: Movement speed multiplier
- `score`: Base score for killing (adjusted by multiplier and difficulty)
- `minStage`: First stage this type appears
- `weave`: Optional flag for side-to-side movement pattern (interceptors only)

**`BOSSES`** (line 508): Six unique boss encounters
- Each has `hullTop`, `hullBottom`, `wing`, `core` colors for rendering
- `name`: Display name for the boss health bar
- `hpMul`: Health multiplier for difficulty scaling (1.7x for TITAN, 0.8x for SPECTER)

### Gameplay Systems

#### Weapon System
Two weapon types with upgrade mechanics:
- **Vulcan**: Rapid-fire projectiles, fires at `1 / (cooldown + rapid)` frequency
- **Laser**: Single stream, higher damage per shot
- Each kill can drop a weapon power-up (type determined randomly)
- Reaching weapon level 4 grants "rapid" mode (faster fire rate)

#### Combo/Multiplier System (lines ~897-898)
- `chain`: Consecutive enemy kills
- `chainTimer`: Resets when 1 second passes without a kill
- `multiplier`: Increases by 0.1 for each combo (max 5.0), affects score calculation
- Visible combo pop (large text) shows when combo >= 3

#### Difficulty Progression
- Game starts at selected difficulty (easy/normal/hard)
- 18 kills per stage (`KILLS_PER_STAGE`)
- After stage 6, boss appears
- Each boss defeated progresses to next boss and returns to regular enemies
- Total 6 bosses, then cycles back or game is "won"

#### Control Schemes
1. **Keyboard**: Arrow keys to move, Spacebar to fire (detected but movement-based firing)
2. **Mouse**: Click anywhere on canvas to toggle firing, move mouse to position player
3. **Touch**: Virtual joystick on left (movement), fire/bomb buttons on right

#### Audio System (lines 543-730)
- WebAudio synth-based sound generation (no audio files)
- Functions: `beep()` (tonal sounds), `noiseBurst()` (explosion sounds)
- Panning based on X position (`panFor()`) for stereo field
- Master gain connected to dynamics compressor
- Mute toggle with localStorage persistence

#### Visual Effects
- **Screen shake**: `screenShake(magnitude, duration)` for hits and explosions
- **Flash effects**: White flash on damage, red vignette on critical health
- **Animations**: Combo popups, stage banners, boss health bar transitions
- Respects user's `prefers-reduced-motion` CSS media query

### State Management
Game state variables (lines 865-869):
- `running`: Whether game is active
- `paused`: Whether game is paused (fires still work)
- `gameOver`: Whether run has ended
- `frame`: Frame counter for timing logic
- `kills`: Total enemies killed this run
- `stage`: Current stage (1-6+)
- `stageKills`: Enemies killed in current stage (triggers boss at KILLS_PER_STAGE)
- `score`, `lives`, `bombCount`: Player resources

### localStorage Keys
- `skyfighter.bestScore`: Highest score achieved
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Accessibility motion preference

## Development Workflow

### Running the Game Locally
Simply open `index.html` in a modern web browser (Chrome, Firefox, Safari, Edge). Game runs in fullscreen-capable container and auto-resizes.

### Making Changes

#### Editing Game Balance
1. Modify `DIFFICULTIES` object (line 485) to adjust spawn rates, enemy behavior, or player resources
2. Modify `ENEMY_TYPES` (line 502) to change individual enemy stats
3. Modify `BOSSES` (line 508) to adjust boss appearance or health multipliers
4. Test across all three difficulty modes

#### Adding Visual Themes
1. Add new object to `THEMES` array (line 493)
2. Provide `sky` (3-color gradient), `cloud`, `sun` RGB values, and `name`
3. Sky colors should progress from dark (top) to bright (bottom) for depth
4. Test theme appears at appropriate stage

#### Adding/Modifying Enemy Types
1. Add new type to `ENEMY_TYPES` (line 502)
2. Set `minStage` to control first appearance
3. Implement spawn logic in `update()` if special behavior needed
4. Update `drawPlane()` (line 913) if custom rendering needed

#### Adjusting Audio
1. Modify `beep()` frequency, duration, type in attack/hit events
2. Adjust `noiseBurst()` duration and frequency range for explosions
3. Modify `panFor()` calculation if stereo field should be wider/narrower
4. Test with mute toggle to ensure UI still works when audio disabled

#### UI and Layout Changes
1. CSS is in `<style>` block (lines 10-420)
2. Responsive sizing uses `clamp()` and viewport-relative units (`vw`, `vh`)
3. Update `resize()` function (line 754) if canvas aspect ratio or layout changes
4. Test on mobile (touch controls) and desktop (keyboard/mouse)

### Common Code Patterns

**Creating a new entity:**
```javascript
enemies.push({
  x: canvas.width * 0.5,
  y: 40,
  w: 34, h: 34,
  vx: 0, vy: 120,
  hp: 1,
  type: 'grunt',
  angle: 0,
  shootCooldown: 0
});
```

**Spawning projectiles:**
```javascript
bullets.push({ x: player.x, y: player.y - 20, vx: 0, vy: -bulletSpeed, life: 4 });
```

**Playing a sound effect:**
```javascript
ensureAudio(); // Initialize audio context if needed
beep(440, 0.1, 'sine', 0.3, { pan: panFor(player.x) });
```

**Adding a particle:**
```javascript
particles.push({ x, y, vx, vy, life: 0.6, type: 'spark' });
```

**Checking collision (AABB):**
```javascript
if (e.x - e.w/2 < bullets[i].x && e.x + e.w/2 > bullets[i].x &&
    e.y - e.h/2 < bullets[i].y && e.y + e.h/2 > bullets[i].y) {
  // Collision detected
}
```

## Performance Considerations

### Adaptive Quality System (lines 474-483)
Automatically adjusts rendering quality based on frame time:
- Quality level 0: Full quality (target ~60fps)
- Quality level 1: Medium quality (when avg frame > 25ms)
- Quality level 2: Low quality (when avg frame < ~18ms)
- Currently tracks performance but quality adjustment logic is placeholder for future use

### Optimization Tips
1. **Entity pooling**: Particles and projectiles are created/destroyed frequently; could be pooled for better GC
2. **Collision checks**: Currently O(n²) for enemy-bullet collisions; could use spatial partitioning for high entity counts
3. **Canvas rendering**: Single global CTX; all entities draw directly without intermediate buffers
4. **Enemy AI**: Spawning is timed (not frame-based for consistency), shooting is probabilistic to reduce spikes

## Testing Changes

### Manual Testing Checklist
- [ ] Game starts and can be played to completion
- [ ] All three difficulty modes work and have correct balance
- [ ] All 6 themes display correctly during progression
- [ ] All 3 enemy types spawn at appropriate stages and behave correctly
- [ ] All 6 bosses spawn and have correct health/appearance
- [ ] Player can use both weapon types and upgrade levels work
- [ ] Keyboard, mouse, and touch controls all function
- [ ] Audio mute toggle works
- [ ] Reduced motion toggle works and hides animations
- [ ] Score persists in localStorage (high score and difficulty)
- [ ] Mobile viewport scaling and touch targets work properly

### Testing a Specific Feature
1. Use browser DevTools console to set game state: `difficulty = 'hard'; stage = 6;` etc.
2. Use DevTools to pause/resume execution and step through game logic
3. Use DevTools Network tab to verify no external assets load
4. Use Lighthouse or Chrome DevTools Performance tab to profile rendering

## Browser Compatibility

- Requires: Canvas API, requestAnimationFrame, localStorage, WebAudio API
- Works: Chrome/Edge 90+, Firefox 88+, Safari 14+
- Touch support: iOS Safari, Android Chrome/Firefox
- Accessibility: Screen reader announcements via `announce()`, reduced motion support

## File Format Notes

- Single HTML file intentional: easy to share, version control, deploy
- No transpilation needed: uses ES6+ features natively available in all modern browsers
- Self-contained assets: All colors, fonts, sounds generated in code (no external files)
- 1600+ lines is manageable for a complete game due to declarative data (configs) + helper functions

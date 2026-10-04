# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented entirely in a single HTML file (~1,700 lines). It features procedurally rendered graphics (no image assets), WebAudio synthesized sound effects, and full keyboard/touch/joystick controls.

## Architecture Overview

### Single-File Structure
All code resides in `index.html`:
- **Head**: CSS styling, viewport configuration, meta tags
- **Body**: Canvas and UI elements (HUD, overlay, touch controls)
- **Script**: Complete game implementation

### Core Game Loop
The game runs via `requestAnimationFrame`:
```
Main loop → update(dt) → render collisions/state → draw() → display
```

Key state objects:
- `player`: position, health, weapon type/level, invulnerability timer
- `enemies`: array of active enemies
- `boss`: current boss encounter (if active)
- `bullets`, `enemyBullets`: projectiles from player and enemies
- `powerUps`: health/bomb/weapon drop items
- `particles`: visual effects (explosions)

### Game State Constants

**Difficulty levels** (`DIFFICULTIES` object):
- `easy`, `normal`, `hard`: Enemy spawn rate, health multipliers, shoot frequency
- Applied via difficulty selector buttons; persisted to localStorage

**Enemies** (`ENEMY_TYPES` object):
- `grunt`: Basic red enemy, spawns stage 1+
- `interceptor`: Fast orange enemy, spawns stage 1+
- `tank`: Slow purple tank, spawns stage 2+, high HP
- Each type has different visual colors, speed multipliers, HP, and point values

**Bosses** (`BOSSES` array):
- 6 unique boss designs with distinct color schemes
- Each stage cycles through boss list
- HP scales with stage number and difficulty

**Environment** (`THEMES` array):
- Background gradient colors per stage
- Cycling theme every stage

**Boss Attack Patterns** (`BOSS_PATTERNS` array):
- 6 pattern sets, one per boss type
- Each pattern set contains 3 attack sequences (cycled during fight)
- Attack types: radial bursts, aimed spreads, columns, teleport barrages, homing missiles

### Core Game Systems

#### 1. Collision Detection
`rectsOverlap(a, b)`: Axis-aligned bounding box (AABB) collision test—returns true if two rectangles overlap within tolerance. Used for:
- Bullet-enemy hits
- Bullet-boss hits
- Enemy-player collision
- PowerUp pickup

#### 2. Player Movement & Firing
- **Movement**: Keyboard (arrows/WASD), joystick (touch), or mouse (point-to-click)
- **Weapons**: 
  - Vulcan: rapid-fire bullets with spread at level 3+
  - Laser: slower high-damage lasers, pierce through 1-2 enemies at level 3+
- **Firing**: Hold Space or touch fire button; cooldown between shots (0.1s laser, 0.15s vulcan)

#### 3. Enemy AI
- **Spawn**: Random x position, enter from top off-screen
- **Movement**: Steady downward speed; some types weave left/right
- **Firing**: Random chance per difficulty; shoot aimed (at player) or spread pattern

#### 4. Boss Encounters
- Triggered after killing `KILLS_PER_STAGE` enemies (default 15)
- **Phases**:
  1. **Entering**: Boss descends from top (~1.2s duration)
  2. **Active**: Boss bounces left/right, fires attack patterns
  3. **Defeated**: Explosion, score +500, advance stage
- **Attack System**:
  - Telegraph: Boss flashes white for ~0.37s before attack
  - Attack: Boss fires pattern from `BOSS_PATTERNS[bossIndex][patternIndex]`
  - Cooldown: ~1.4s between attacks

#### 5. Score & Combo System
- **Base score**: Fixed points per enemy type (grunt 10, interceptor 15, tank 35, boss 500)
- **Combo multiplier**: Increases by 1 for every 5 consecutive kills; resets if player takes damage
- **Chain timer**: Breaks combo if 2.2s pass without a kill

#### 6. Power-ups
Randomly spawn on enemy death (~6% life, ~8% bomb, ~20% weapon):
- **Life**: Restore 1 health (cap: `maxLives`)
- **Bomb**: Add 1 smart bomb (cap: `maxBombs`)
- **Weapon**: Upgrade weapon level or swap weapon type

#### 7. Smart Bomb
- Clears all enemy bullets and enemies on screen
- Damages boss (24 HP)
- Spawns explosion particles
- Causes screen shake

### Audio System (WebAudio Synth)
All sounds are procedurally generated using WebAudio; no external audio files:

**Key modules:**
- `ensureAudio()`: Initialize AudioContext (resume if suspended)
- `beep(freq, dur, type, vol, opts)`: Generate tone with envelope, sweep, filter, pan
- `noiseBurst(dur, vol, opts)`: Generate filtered white noise burst
- `sfx` object: Named effects (shoot, laser, explode, bomb, powerup, bossHit, gameover, etc.)

**Pan calculation:** `panFor(x)` maps canvas x-coordinate to stereo pan [-1, 1] for spatial audio.

### Rendering

**Canvas setup:**
- Base size: 480×720
- Scaled to fit viewport (max 1.6× magnification)
- Canvas resolution always 480×720 (pixel-perfect rendering)

**Key rendering functions:**
- `drawPlane(x, y, w, h, color, darkColor, flame)`: Procedural player/enemy ship
- `drawBossShip(b)`: Procedural boss with gradient hull, glowing core, rotating engines
- `drawClouds(speed, fillColor, rx, ry)`: Animated background parallax layers
- `draw()`: Main render loop—clears canvas, draws sky gradient, clouds, sprites, bullets, effects

**Visual effects:**
- **Particles**: Explosion burst with velocity, gravity decay, fade
- **Screen shake**: Random translation applied to canvas during hit/bomb
- **Flash**: Full-screen white/red overlay with CSS animation
- **Vignette**: Red radial vignette pulses when health critical (1 life remaining)

### Input Handling

**Keyboard:**
- Arrow keys / WASD: Movement
- Space: Fire
- P / Esc: Toggle pause
- X / Shift: Use bomb

**Touch:**
- Virtual joystick: Bottom-left corner (drag to control)
- Fire button: Bottom-right corner
- UI buttons: Top (pause, mute, motion, difficulty)

**Mouse:**
- If mouse moves during gameplay, player follows mouse cursor (point-to-click mode)
- Crosshair displayed at target

### Settings & Persistence

**LocalStorage keys:**
- `skyFighterBest`: High score
- `skyFighterMute`: Audio mute state (0/1)
- `skyFighterDiff`: Selected difficulty
- `skyFighterMotion`: Reduced motion preference (0/1)

**Reduced Motion:**
- Respects system preference (`prefers-reduced-motion`)
- Disables screen shake, particle explosions, flash
- Reduces animation duration/intensity

### Game Lifecycle

**States:**
- `running`: Game in active play
- `paused`: Game paused
- `gameOver`: Game ended (all lives lost)

**Flow:**
1. `startGame()`: Reset state, spawn first enemy wave
2. `update(dt)`: Process input, move objects, check collisions
3. `draw()`: Render current frame
4. Loop until `lives <= 0` → `endGame()`
5. `endGame()`: Show game-over overlay, check high score, enable restart

**Pause & Resume:**
- Pauses both update and draw
- Shows overlay; resumes on button click or P key

## Development Notes

### Adding New Features

**New Enemy Type:**
1. Add entry to `ENEMY_TYPES` with sprite colors, speed, HP, score
2. Update `spawnEnemy()` filter logic if stage-gated
3. Add explosion SFX to `EXPLODE_SFX` mapping (optional)

**New Boss:**
1. Add design to `BOSSES` array (color scheme)
2. Create attack pattern array in `BOSS_PATTERNS` with 3 attack functions
3. Boss type is auto-selected: `(stage - 1) % BOSSES.length`

**New Weapon:**
1. Add weapon name to player object
2. Modify `fireWeapon()` to handle new type
3. Add fire SFX to `sfx` object
4. Update HUD weapon display logic

**New Screen Effect:**
- Particle type: Add object to `particles` array with velocity, life
- Flash: Trigger via `flashHit()` or `flashBomb()` (CSS-driven)
- Vignette: Modify `vignetteEl` visibility/animation

### Performance Considerations

- **Quality setting** (not exposed in UI): Controls particle count and shadow blur
- **Frame skipping**: Not implemented; relies on 60 FPS target
- **Canvas size**: Always 480×720 for pixel-perfect scaling
- **Collision checks**: O(n²) enemy-bullet; optimize if > 50 entities

### Testing

No test framework. Manual testing:
1. Start game, verify spawn rate scales with difficulty
2. Kill enemies, confirm combo multiplier increases
3. Reach stage milestone, verify boss encounter
4. Use bomb, verify all bullets cleared and boss damaged
5. Pause/resume, check state persists
6. Reduce motion, verify screen shake/flash disabled

### Code Style

- ES6 syntax (const/let, arrow functions, template literals)
- No external dependencies
- Single global namespace (everything in one `<script>`)
- Canvas context cached in `ctx` variable
- Frame counter for animation/timing

## Common Commands

Since this is a static HTML game with no build step:

- **Open game locally:** Open `index.html` in a browser
- **Deploy:** Copy `index.html` to web host (or push to GitHub Pages)
- **Test on mobile:** Open via HTTP (not file://) on device; touch controls auto-enable
- **Debug:** Use browser DevTools console; breakpoint in `update()` or `draw()`

No npm/build scripts are needed.

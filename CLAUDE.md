# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 web application. The game features:
- Canvas-based rendering with procedural graphics (no image assets)
- Web Audio API synthesized sound effects (no audio files)
- Progressive difficulty with 6 themes, 3 difficulty levels, and 6 boss types
- Multiple enemy types with distinct behaviors
- Various weapon systems with different firing patterns
- Power-up system (health, bombs, weapon upgrades)
- Touch and mouse input support with joystick emulation
- Adaptive performance quality scaling based on frame rate

## Architecture & Core Systems

The entire game runs in `index.html` (~1670 lines). The code is organized functionally rather than by class, with clear separation of concerns:

### Game Loop
- **Entry point**: `requestAnimationFrame(loop)` at line 1652
- **Loop structure**: `update(dt)` → `draw()` → next frame
- **Update cycle** (line 1148): Handles player movement, bullet updates, collision detection, enemy spawning, boss AI
- **Draw cycle** (line 1520): Renders background, entities, HUD, effects

### Game State
State is maintained in module scope (closure). Key variable categories:
- **Player state**: Position, health, bombs, weapon level, firing state
- **Entity pools**: Arrays for bullets, enemies, explosions, power-ups
- **Game progress**: Score, combo counter, stage counter, kills
- **UI state**: Pause, game over, difficulty, muted, reduced motion

**Difficulty levels** (line 483-486) control spawn rates, enemy stats, player lives:
- `easy`: Slower spawning, weaker enemies, 4 lives, 3 bombs
- `normal`: Balanced difficulty (default)
- `hard`: Fast spawning, aggressive enemies, 3 lives, 2 bombs

### Key Systems

#### 1. **Rendering & Visuals**
- **Canvas setup**: 960x640 (line ~) with responsive scaling via CSS
- **Themes** (line 488-494): 6 procedurally generated sky themes with colors for gradient, clouds, and sun
- **Background**: Parallax scrolling clouds using `drawClouds()` (line 1507)
- **Procedural drawing functions**:
  - `drawPlane()`: Player and enemy ships rendered with triangular shapes + flame
  - `drawBossShip()`: Boss sprite with hull sections, wings, and animated core
  - `spawnExplosion()`: Particle explosions with physics
- **HUD**: Text and icon overlays (health hearts, bomb count, score, weapon level, combo)
- **Visual effects**: Screen shake, flash overlay (for hits), vignette (low health), combo popup

#### 2. **Audio System (Web Audio API)**
- **Initialization**: `ensureAudio()` creates AudioContext on first user interaction
- **Master setup**: Compressor chain for dynamic range control
- **Synthesis approach**: Procedural beeps and noise bursts, no external files
- **Sound effects** (line 655-683): Organized in `sfx` object with named methods:
  - `shoot()`, `laser()`: Player weapon sounds
  - `explode*()`: Enemy-specific explosion sounds (grunt, interceptor, tank)
  - `hit()`, `bomb()`: Player damage and bomb blast
  - `powerup()`, `bossHit()`, `bossTelegraph()`, `bossDown()`: Game event sounds
- **Helper functions**:
  - `beep(freq, dur, type, vol, opts)`: Oscillator-based tones with optional sweep, filter, panning
  - `noiseBurst(dur, vol, opts)`: White noise with envelope control
  - `panFor(x)`: Calculate stereo pan position based on screen x coordinate

#### 3. **Input Handling**
Supports keyboard, mouse, and touch (mobile joystick):
- **Keyboard** (line 769): Arrow keys / WASD for movement, Space to fire, Shift for bomb
- **Mouse** (line 847): Aim and fire with cursor; movement via zones
- **Touch** (line 814+): Left half of screen = movement zone, right half = fire zone
  - Virtual joystick drawn in DOM (2D joystick movement)
  - Bomb button via UI button
- **Input state**: `keys`, `firing`, `mouseActive`, joystick offset tracked in global scope

#### 4. **Collision Detection**
- **Simple AABB**: `rectsOverlap(a, b)` checks axis-aligned bounding boxes (line ~)
- **Used for**:
  - Player bullets hitting enemies
  - Enemy bullets hitting player
  - Player colliding with power-ups
  - Bomb blast radius vs enemies

#### 5. **Enemy & Boss System**
- **Enemy types** (line 497-501): grunt, interceptor, tank
  - Each type has: width, height, HP, speed multiplier, color, score value, min stage
  - `spawnEnemy()` (line ~): Random type based on stage, then random position + velocity
- **Enemy AI**:
  - Simple downward movement with occasional lateral weave
  - Firing chance based on difficulty (`enemyShootChance`)
  - Bullet patterns: single shot or occasional spread
- **Boss system** (line 502-508):
  - 6 boss types (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
  - Each boss has color scheme and HP multiplier
  - `spawnBoss()` (line ~): Triggered after KILLS_PER_STAGE (18) enemy kills
  - `updateBoss(dt)` (line 1411): Handles boss movement, AI, and boss-specific attack patterns
  - Boss fires 5+ distinct bullet patterns (radial, spiral, homing, columns, cross)
- **Destruction**: `killBoss()` awards massive bonus, plays victory sound, advances to next stage

#### 6. **Weapon & Power-up System**
- **Weapon levels**: Player starts at level 1; each power-up increments
  - Display: HUD shows current weapon level (green)
- **Firing patterns** by weapon level:
  - Level 1: Single center shot
  - Level 2+: Spread patterns increase with level
  - Each pattern function: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.
- **Power-up types** (line ~): Applied via `applyPowerUp(type)`:
  - `heal`: Restore 1 health (max 3 or 4)
  - `bomb`: Add 1 bomb
  - `weapon`: Increment weapon level
- **Spawn**: Random chance on enemy death via `maybeSpawnPowerUp(x, y)`

#### 7. **Score & Combo System**
- **Score tracking**: Base score per enemy type (10–35 pts)
- **Combo multiplier**: Tracks consecutive kills; resets on taking damage
  - Display: Combo value shown in HUD (orange)
  - Visual feedback: `showComboPop()` animates combo milestones
- **Best score**: Saved to localStorage, persists across sessions

#### 8. **Performance Adaptation**
- **Frame time tracking** (line 484–495): Last 45 frame times stored
- **Quality levels**: 0 (full), 1 (medium), 2 (low)
  - Scaling triggers at 40 FPS threshold (downgrades) or 56 FPS (upgrades)
  - Affects: particle count, effect intensity, motion reduction
- **Reduced motion preference**: Respects system `prefers-reduced-motion` media query

#### 9. **Game State Management**
- **Running state**: `running`, `paused`, `gameOver`
- **Life cycle**:
  - `resetGame()`: Initialize all entities, reset score/combo, start game
  - `update()`: Main game tick handling all logic
  - `loseLife()`: Decrement health, trigger effects, or end game
  - `endGame()`: Stop loop, save best score, show overlay with stats
  - `togglePause()`: Freeze/unfreeze without ending

## File Structure & Sections

The single `index.html` is logically organized:
1. **HTML** (~400 lines): Canvas container, HUD elements, UI buttons, overlay
2. **CSS** (~280 lines): Layout, animations, theme variables, responsive design
3. **JavaScript** (~1000 lines): Game logic in one IIFE closure

JavaScript execution order:
- Constants & difficulty defs (lines 483–522)
- DOM element caching (lines ~540–560)
- Audio system setup (lines 540–683)
- UI helpers (render hearts, bombs, flashes, popups; line 685–780)
- Difficulty & theme toggles (line 745–765)
- Input event listeners (line 769–860)
- Game entity arrays & state init (line 863–)
- Core functions: `update()`, `draw()`, boss AI, collision checks
- Game loop entry point (line 1650+)

## Commonly Used Operations

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` with properties (width, height, HP, speed multiplier, color, score, minStage)
2. Update `spawnEnemy()` to include new type in spawn pool based on stage
3. Add explosion sound effect in `sfx` if unique (e.g., `explodeNovotype`)
4. Update `EXPLODE_SFX` map if using custom explosion audio

### Adding a New Boss
1. Add boss object to `BOSSES` array (name, hull colors, wing color, core color, hpMul)
2. Add boss attack pattern function (e.g., `fireNovaBurst()`)
3. Update `updateBoss()` to call new pattern based on boss index or phase
4. Add boss-specific sound effects if needed

### Modifying Weapon Behavior
- Locate weapon fire function (e.g., `fireRadial()`)
- Adjust bullet spread angle, count, speed, or colors
- Test by picking up weapon power-ups or running at desired difficulty

### Adding a New Theme
1. Add object to `THEMES` array with sky colors (array of 3), cloud color string, sun color string, and name
2. Adjust `drawClouds()` if theme needs different cloud rendering
3. Colors must be in RGB format: `'R,G,B'` as strings

### Tuning Difficulty
Edit `DIFFICULTIES` object parameters:
- `spawnBase` / `spawnMin`: Control enemy spawn timing (lower = more frequent)
- `enemySpeed`, `enemyShootChance`: Adjust AI aggression
- `lives`, `bombs`: Player starting resources
- `bossHpMul`: Boss health multiplier

## Debugging & Testing Tips

- **Open DevTools** (F12) → Console to read announcements (screen reader output)
- **Performance**: Watch frame time histogram; if avg > 25ms, adaptive quality will degrade rendering
- **Audio issues**: Check `audioCtx.state` in console; may be 'suspended' until user interaction
- **Score/progress**: Use DevTools → Application → Local Storage to view/clear saved data (STORAGE_BEST, STORAGE_DIFF, STORAGE_MUTE, STORAGE_REDUCED)
- **Canvas size**: Responsive; check `canvas.width` and `canvas.height` after window resize
- **Touch testing**: Use DevTools device emulation or real mobile device; joystick zone is left half of screen

## localStorage Keys

- `sky-fighter-best-score`: Best score
- `sky-fighter-difficulty`: Last selected difficulty (easy/normal/hard)
- `sky-fighter-muted`: Mute state (0/1)
- `sky-fighter-reduced-motion`: Reduced motion preference (0/1)

## Known Patterns & Idioms

- **Entity pooling**: Bullets, enemies, explosions stored in reusable arrays; objects recycled rather than created/destroyed
- **Closure scope**: All game state in module closure; no global variables except `window` props (intentional for iOS compatibility)
- **Canvas transforms**: Screen shake applied via `ctx.translate()` in draw loop; shear not used
- **Responsive sizing**: `max-width` and `max-height` on canvas; CSS `clamp()` for font sizes
- **localStorage checks**: Null-coalesce with `||` for defaults; localStorage can throw (check before read in production)

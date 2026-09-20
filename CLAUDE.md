# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a vertical shoot-'em-up game (Raiden-style) built as a single-file HTML5 application. The game features:
- Canvas-based graphics rendering
- Procedurally-generated enemies with difficulty scaling
- Multi-stage progression with boss encounters
- Web Audio synthesized sound effects (no external audio files)
- Touch and keyboard/mouse controls
- Adaptive performance quality system
- Multiple difficulty levels and themed environments

## Architecture

The entire game logic is contained in a single `index.html` file (~1,669 lines) with inline CSS (lines 10–399) and JavaScript (lines 422–1,667). The structure is:

### Core Game State
- **Player**: Position, speed, weapon type/level, health (lives), invulnerability timer
- **Bullets**: Player projectiles (vulcan or laser types with pierce mechanics)
- **Enemies**: Dynamic list of enemies with type-specific behavior (grunt, interceptor, tank)
- **Enemy Bullets**: Projectiles spawned by enemies and bosses
- **Particles**: Transient visual effects (explosions, debris)
- **Power-ups**: Collectible items (life, bomb, weapon upgrade)
- **Boss**: Single active boss entity during stage progression

### Main Game Loop (`loop()` function, line 1640)
1. **Input**: Mouse/touch movement, keyboard/button firing, bomb triggers
2. **Update** (`update()`, line ~1100): Advances all game state (enemies, bullets, collisions)
3. **Draw** (`draw()`, line 1520): Renders current frame to canvas
4. **Performance Tracking**: Adapts rendering quality to maintain ~60fps

### Key Systems

#### 1. Difficulty System (DIFFICULTIES constant, line 485)
- **Easy**: Slower spawns, weaker enemies, 4 lives, 3 bombs, reduced boss HP
- **Normal**: Baseline difficulty (3 lives, 2 bombs)
- **Hard**: Aggressive spawns, fast enemies, boosted boss HP
- Difficulty modulates: spawn rate, enemy speed, AI shoot frequency, player lives/bombs, boss health

#### 2. Enemy Spawning (spawnEnemy, line 1022)
- **Type Selection**: Probability-weighted based on current stage (tanks appear at stage 2+)
  - Grunt (common): Basic enemy
  - Interceptor (medium): Fast, can weave
  - Tank (rare): High HP, tanky
- **Behaviors**:
  - Straight movement with optional weaving (sine wave horizontal pattern)
  - Two shoot patterns: spread shot (three bullets) or aimed at player
  - Difficulty scales spawn frequency over time (line 1201)

#### 3. Weapon System
- **Vulcan** (default): Rapid-fire, unlimited ammo, hitscan
  - Level 1–3: Increases fire rate
  - At level 4: Switches to laser weapon
- **Laser**: Slower fire rate, pierces through enemies
  - Can damage multiple enemies per shot
  - Level 1–3: Increases pierce count
  - At level 4: Switches back to vulcan
- Fire rate controlled by `player.cooldown` (update function decrements this)

#### 4. Boss System
- **Design**: Six boss types cycle (BOSSES constant, line 508), each with unique visuals and attack patterns
- **Spawning**: Triggered after 18 kills per stage (KILLS_PER_STAGE, line 491)
- **Entry Animation**: Boss slides in from top, invulnerable during entry
- **Attack Patterns** (BOSS_PATTERNS, line 1372): Each boss has 3 unique attack patterns
  - Patterns include: radial bursts, aimed spreads, columns, spirals, teleport-dash attacks
  - Boss telegraphs attacks with visual glow (22 frames warning)
  - Attack cycle duration ~85 frames
- **Difficulty Scaling**: Boss HP scales with difficulty multiplier and stage progression

#### 5. Weapon Mechanics (Player Firing)
- **Vulcan**: Fires upward from player center; no spread
- **Laser**: Fires upward; passes through multiple enemies with diminishing pierce
- Firing is input-driven (held button = continuous fire, respects cooldown)
- Sound effects differ by weapon type

#### 6. Collision System (rectsOverlap, line 1069)
- Simple AABB (axis-aligned bounding box) collision detection
- Checks: player-bullets vs enemies, player-bullets vs boss, player vs enemies/bullets, collectibles
- Multi-collision per frame: enemies can collide with multiple bullets in one update

#### 7. Combo and Scoring System
- **Chain Mechanic**: Each enemy kill increments a chain counter
- **Multiplier**: Increases every 5 consecutive kills (max 10x)
- **Timeout**: Chain breaks if no kill within 2.2 seconds
- Score formula: `base_score * multiplier`

#### 8. Visual Effects
- **Screen Shake**: Magnitude/duration controlled separately; applied to canvas transform
- **Flash**: White full-screen fade (hit or bomb detonation)
- **Particles**: Explosion geometry spawned on enemy destruction and impacts
- **Vignette**: Red pulsing edge during critical health
- **Themes**: Six visual themes (sky gradients, cloud styles, sun colors) cycling per stage

#### 9. Audio System
- **Web Audio API Synthesis**: No external files
- **Oscillator-based Sounds**: Beeps with frequency sweeps, envelopes
- **Noise Bursts**: Filtered white noise for impact sounds
- **Spatial Audio**: Pan stereo based on X position of event
- **Master Gain + Dynamics Compressor**: Volume normalization

#### 10. Control Schemes
- **Desktop**: Mouse movement (plane follows cursor), click to fire, Space/X to bomb
- **Keyboard**: Arrow keys / WASD for movement, Space to fire, X/Shift for bomb
- **Mobile**: Left side drag for movement joystick, FIRE/BOMB buttons (touch)
- **Priority**: Keyboard/joystick overrides mouse movement when active

#### 11. Adaptive Performance
- Tracks frame times over 45-frame window (line 475)
- Three quality modes: 0 (full), 1 (medium), 2 (low)
- Reduces particle spawning, shadow rendering based on quality
- Automatically scales down if frame time exceeds 25ms average

#### 12. Persistence
- **localStorage** stores: best score, mute state, difficulty, reduced motion preference
- Keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`

## Making Changes

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` constant (line 502):
   ```javascript
   newType: { w: 30, h: 30, hp: 2, speedMul: 1.2, color: '#ff00ff', dark: '#8b008b', score: 25, minStage: 3, weave: false }
   ```
2. Update `spawnEnemy()` probability logic (line 1025) if needed
3. Add to `EXPLODE_SFX` mapping if custom explosion sound
4. Enemy rendering (`draw()`) uses same `drawPlane()` for all types

### Adding a New Boss Pattern
1. Create a function in the BOSS_PATTERNS section (line 1372) following existing patterns
   - Functions receive boss object `b` and modify global `enemyBullets` array
   - Use helper functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.
2. Add pattern to the appropriate boss's array in `BOSS_PATTERNS`
3. Boss cycles through patterns sequentially; add up to 3 per boss

### Modifying Difficulty
Edit `DIFFICULTIES` constant (line 485) to adjust:
- `spawnBase`/`spawnMin`: Enemy spawn rate (in frames)
- `enemySpeed`: Multiplier on enemy movement speed
- `enemyShootChance`: Probability each enemy can fire
- `lives`/`bombs`: Starting player resources
- `bossHpMul`: Boss health multiplier

### Changing Visual Themes
Edit `THEMES` array (line 493):
- `sky`: Array of 3 gradient stops (top, middle, bottom)
- `cloud`: RGB color for cloud layers
- `sun`: RGB color for sun glow
- `name`: Display name shown on stage transition

### Adjusting Game Balance
Key tunable constants:
- `KILLS_PER_STAGE` (line 491): Enemies needed to spawn boss
- Player speed (line 877): `player.speed`
- Bullet speeds: Check firing functions (`beep()`, weapon fire logic)
- Enemy spawn difficulty ramp (line 1201): `difficultyRamp` calculation

## Code Organization Pattern

The code uses a single self-executing function scope to avoid global namespace pollution. Internal structure:
1. **DOM element cache**: Top-level assignments (lines 426–461)
2. **Constants**: Game config (DIFFICULTIES, THEMES, ENEMY_TYPES, BOSSES, etc.)
3. **Audio subsystem**: `ensureAudio()`, `beep()`, `noiseBurst()`, `sfx` object
4. **Input handlers**: Keyboard, mouse, touch event listeners
5. **Game state vars**: `player`, `enemies`, `score`, etc.
6. **Game logic functions**: `update()`, `draw()`, collision handlers
7. **Entity spawning**: `spawnEnemy()`, `spawnBoss()`, etc.
8. **Main loop**: `loop()` + `requestAnimationFrame` at line 1665

## Common Editing Tasks

### Increase Game Difficulty
Modify `DIFFICULTIES.hard` or create a new difficulty level and wire it to UI buttons.

### Add Visual Polish
- Create new particle effect by adding to `spawnExplosion()` calls
- Adjust canvas rendering in `draw()`: shadows, glow, blur filters
- Modify theme colors

### Change Player Starting Position
Edit player initialization in `resetGame()` (line 877): `player.y = canvas.height - 100`

### Adjust Weapon Fire Rate
Find weapon firing code (~line 1140) and modify `player.cooldown` reset values:
- Vulcan: Typically 4–6 frames between shots
- Laser: Typically 16–20 frames between shots

### Test a Single Boss
In `spawnBoss()` (line 1043), override boss design:
```javascript
boss.design = BOSSES[2]; // Force SPECTER boss
```

## Testing & Verification

The game runs directly in a browser; no build step required.
- **Load**: Open `index.html` in any modern browser
- **Performance**: Check adaptive quality at 60fps target
- **Audio**: Browser must allow Web Audio context activation (first user input)
- **Mobile**: Test touch controls on device or dev tools emulation (toggle mobile device mode, 375px width recommended)

## Notes

- The entire game is ~1,669 lines in a single file; modular extraction is possible but not required for current scope
- Canvas resolution matches window size via CSS (`width: 100%`) and pixel-perfect rendering (no DPI scaling in code)
- Sound synthesis parameters are tuned for the Web Audio API; audio context state must be resumed on first user input
- Reduced motion preference (prefers-reduced-motion media query) disables animations and shake effects for accessibility

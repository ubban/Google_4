# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 Canvas application. The entire game logic, UI, and styling are contained in `index.html` (~1670 lines).

## Architecture & Key Systems

The game follows a classic arcade game loop pattern with several interconnected systems:

### 1. Game Loop
- **Location**: Lines 1640-1653
- Runs via `requestAnimationFrame` with delta-time (dt) passed to update/draw
- Includes adaptive performance tracking that adjusts visual quality (line 476-483)

### 2. State Management
Core game state lives in module-scoped variables (line 864-869):
- **Player**: Position, weapon (vulcan/laser), level, lives, cooldown, invulnerability timer
- **Collections**: `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- **Game Flow**: `running`, `paused`, `gameOver`, `stage`, `stageKills`, `bossWarned`
- **Scoring**: `score`, `chain`, `chainTimer`, `multiplier` (hit chain mechanic)

### 3. Rendering & Visuals
- **Sky Themes**: 6 theme configurations (line 493-500) with gradient sky, sun, and cloud rendering
- **Entity Drawing**: 
  - Player ship: `drawPlane()` (lines 913-968, reused for enemies)
  - Boss ships: `drawBossShip()` (lines 970-1020) with unique designs per boss
  - Particles: Simple circles with fade-out
- **HUD**: HTML elements updated per frame (score, multiplier, weapon, lives, boss bar)
- **Visual Feedback**: Flash effects, screen shake, vignette on critical health, combo popups

### 4. Input System
Three input methods coexist (line 768-861):
- **Keyboard**: Arrow keys / WASD for movement, Space to fire, X/Shift for bomb
- **Mouse**: Move to target, click to fire (glides toward pointer when hovering canvas)
- **Touch**: Dual-zone interface — left side joystick for movement, right side buttons for fire/bomb
- Input priority: Keyboard/joystick > mouse (mouse doesn't override direct input)

### 5. Weapon System
- **Vulcan**: Rapid straight shots, spreads at level 2+ (line 1127)
- **Laser**: Slower pierce shots with homing-like concentrated fire, spreads at level 3 (line 1136)
- Weapon upgrade path: Collect 3 weapon power-ups to reach level 3, then cycle between weapons (line 1312-1316)

### 6. Enemy & Boss AI
- **Enemy Spawning**: Dynamic spawn rate scaling with difficulty and progression (line 1198-1203)
- **Enemy Types**: Grunt, Interceptor (weaving), Tank (multi-HP) with stage-gated availability (line 502-506)
- **Enemy Behavior**: Weaving movement pattern, shoot patterns (spread vs. aimed), varies by difficulty (line 1210-1227)
- **Boss System** (lines 1043-1056, 1372-1439):
  - 6 boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) cycling by stage
  - Each boss has 3 unique attack patterns (radial bursts, homing, spirals, etc.)
  - Telegraphing before attacks, entering animation, health bar feedback
  - Pattern selection: `BOSS_PATTERNS[designIndex][patternIndex % length]` (line 1425-1427)

### 7. Difficulty & Progression
- **Difficulties** (line 485-489): Easy, Normal, Hard with different spawn rates, enemy speed, shoot chance, HP multipliers
- **Progression**: Kill 18 enemies per stage to trigger boss warning, advance to next stage after defeating boss
- **Themes**: Cycle through 6 visual themes per stage (line 1529)

### 8. Audio (WebAudio Synthesis)
- **Location**: Lines 542-676
- All sounds synthesized via Web Audio API (no external files)
- Synthesized effects for: shoot, laser, explosions (grunt/interceptor/tank), hit, bomb, powerup, boss telegraph, stage clear, UI click, gameover
- `panFor()` calculates stereo panning based on entity x-position (line 562)
- Audio only plays if muted=false and audioCtx is initialized (line 545-561)

### 9. Scoring & Chain Mechanic
- **Location**: Lines 1087-1101
- Kill streak extends multiplier: `multiplier = 1 + Math.min(9, Math.floor(chain / 5))`
- Chain breaks if no kill for 2.2 seconds (chainTimer)
- Bomb and boss kills count as score actions

### 10. LocalStorage Persistence
- **Best Score**: `skyfighter.bestScore`
- **Mute State**: `skyfighter.muted`
- **Difficulty**: `skyfighter.difficulty`
- **Reduced Motion**: `skyfighter.reducedMotion` (system preference fallback)

## Working with the Codebase

### Adding a New Enemy Type
1. Add type to `ENEMY_TYPES` (line 502-506) with properties: w, h, hp, speedMul, color, dark, score, minStage
2. Update `spawnEnemy()` weights (line 1028-1030) to control spawn chance
3. Add explosion SFX mapping to `EXPLODE_SFX` (line 678) if custom sound needed
4. Enemy rendering reuses `drawPlane()`, so visual design is automatic

### Adding a New Boss
1. Add design object to `BOSSES` array (line 508-515) with colors and hpMul
2. Add attack pattern array to `BOSS_PATTERNS` (line 1372+) — pattern functions receive the boss object
3. Boss automatically cycles based on stage: `BOSSES[(stage - 1) % BOSSES.length]` (line 1045)

### Adding a New Attack Pattern
- Attack functions (line 1324-1370) take boss `b` and fire bullet spreads to `enemyBullets`
- Radial pattern functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`
- Pattern timing: Controlled by `b.telegraph` (wind-up) and `b.attackTimer` (cooldown between attacks)

### Modifying Difficulty Parameters
- Edit `DIFFICULTIES` object (line 485-489)
- Parameters: `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`
- Difficulty impacts how quickly enemies are spawned and how aggressive they are

### Adjusting Visual Themes
- Edit `THEMES` array (line 493-500)
- Properties: sky (3-stop gradient array), cloud color (rgb), sun color (rgb), name
- Themes rotate by stage: `THEMES[(stage - 1) % THEMES.length]`

### Tuning Weapon Balance
- **Cooldown**: `fireDelay` variable (line 1177) — lower = faster firing
- **Vulcan spread**: Modify angle arrays in `fireWeapon()` (line 1127-1131)
- **Laser pierce**: Adjust `pierce` property (line 1141)
- **Projectile speed**: Change `speed` and `vx` values in bullet objects

## Running & Testing

**No build step required** — simply open `index.html` in a modern browser.

**Testing focus areas**:
- Difficulty scaling: Verify enemy spawn rate and HP feel balanced at each difficulty
- Boss patterns: Ensure all 6 bosses have distinct, challenging attack sequences
- Input responsiveness: Test keyboard, mouse, and touch controls under lag
- Performance: Check frame rate adapts gracefully when many particles/enemies exist
- Mobile experience: Verify touch zones are responsive and readable on small screens

## Performance Considerations

- **Adaptive Quality** (line 474-483): Tracks frame times and adjusts visual quality (shadows, particle counts, blur effects)
- **Collision**: Simple AABB `rectsOverlap()` (line 1069) — O(n²) but acceptable for this game's entity count
- **Canvas Size**: Base 480×720, scales up to 1.6× display size depending on viewport
- **Particle Limits**: Adaptive based on quality level (line 1074)

## File Structure Note

Everything is in a single `index.html` file organized as:
1. **HTML Structure** (lines 362-420): Canvas, HUD elements, overlay, touch controls
2. **CSS Styling** (lines 10-358): Game canvas, HUD, responsive design, animations
3. **JavaScript** (lines 422-1666): Wrapped in IIFE, strict mode, all game logic

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built entirely as a single HTML5 file with embedded CSS and JavaScript. Players control a fighter plane, dodge enemy fire, collect power-ups, and defeat stage bosses across progressively harder waves.

## Project Structure

The entire game is contained in `index.html` (~1670 lines):
- **Lines 1-400**: HTML markup, CSS styling, and visual elements (canvas, HUD, UI overlay)
- **Lines 400-1670**: Single IIFE containing all JavaScript game logic

The game has no external dependencies—it uses native HTML5 Canvas 2D context and Web Audio API.

## Architecture & Key Systems

### 1. Game Loop & Rendering
- **Core loop** (line 1640): `loop(now)` uses `requestAnimationFrame` for ~60 FPS timing
- **Delta time clamped** to 0.05s to prevent frame skip jitter
- **Conditional updates**: Game logic only runs when `running && !paused`
- **Draw pipeline** (line 1520): Clear canvas → draw theme background → draw sky gradient, sun, clouds → draw entities (player, enemies, bullets, particles) → draw UI

### 2. Game State Management
Global state variables track:
- **Player**: `{ x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }`
- **Collections**: `bullets[]`, `enemies[]`, `enemyBullets[]`, `particles[]`, `powerUps[]`
- **Boss**: `boss` object or `null`
- **Progression**: `score`, `lives`, `stage`, `stageKills`, `kills`, `chain`, `multiplier`
- **UI**: `paused`, `gameOver`, `running`

**Initialization**: `resetGame()` (line 873) initializes state based on selected difficulty.

### 3. Input System
Three input methods co-exist without fighting:
- **Keyboard**: Arrow keys or WASD for movement, Space to fire
- **Mouse**: Move cursor over canvas to auto-aim player toward pointer; click to fire; `mouseActive` flag prioritizes keyboard input when active
- **Touch**: Virtual joystick on left side for movement (lines 778–829); tap fire/bomb buttons on right

Joystick normalized with `Math.hypot()` to prevent diagonal speed boost.

### 4. Weapon System
Two weapons with 3-level progression:

**Vulcan** (default):
- Level 1: Single shot straight up
- Level 2: 2-shot spread (±0.12 radians)
- Level 3: 3-shot spread (±0.24 radians)
- Fire delay: 0.15s between shots
- No piercing, bullets destroyed on enemy hit

**Laser**:
- Level 1: Single centered beam
- Level 2: Dual beams (±6px offset)
- Level 3: Dual beams + center shot (3 total)
- Fire delay: 0.1s (faster)
- Piercing: laser.pierce tracks remaining enemies hit before destruction
- Bullets destroyed after `pierce` hits reach 0

Power-up that reaches level 4+ swaps weapons and resets to level 1.

### 5. Difficulty System
`DIFFICULTIES` object (line 485) defines:
- `easy`, `normal`, `hard` modes with parameters:
  - `lives` (3, 3, 2)
  - `bombs` (3, 2, 1)
  - `enemySpeed`, `enemyShootChance`
  - `spawnBase`, `spawnMin` (spawn frequency)
  - `bossHpMul` (difficulty multiplier)

### 6. Enemy Spawning & Behavior
- **Types** (line 502): `grunt` (weak), `interceptor` (fast), `tank` (slow, high HP)
- **Spawn logic** (line 1022): Random type selection based on stage progression; weaker enemies early
- **Movement patterns**: 
  - **Straight**: Basic downward movement
  - **Weave**: Horizontal sine wave motion (lines 1212–1215)
- **Shooting patterns**:
  - **Spread**: 3-way bullet fan (±0.35 radians)
  - **Aimed**: Single homing bullet toward player
- **Stage progression**: 18 kills (`KILLS_PER_STAGE`) trigger boss warning; spawn timer decreases with frame count (line 1201) for difficulty ramp

### 7. Boss System
Bosses appear after 18 stage kills. **6 boss designs** cycle per stage (BOSSES array, line 508):
- **VANGUARD**: Radial burst + aimed fan + V-fan pattern
- **BEHEMOTH**: Heavy columns + homing + radial wall
- **SPECTER**: Teleport dash + tight spiral + dual aimed shots
- **DREADNOUGHT**: 8-way cross + homing volley + radial wall
- **WRAITH**: Teleport barrage + double ring + cross-spread
- **TITAN**: Heavy columns + full ring + relentless homing

**Boss patterns** (line 1372): Array of attack functions per boss; each defines 3 different phases.

**Boss state** (line 1047):
- `entering`: True while sliding into arena (invincible, no attack)
- `telegraph`: Pre-attack warning flash (22 frames); pattern fires when telegraph ≤ 0
- `attackTimer`: Cooldown between attacks (85 frames)
- `patternIndex`: Cycles through 3-phase patterns
- `hp`, `maxHp`: Scale with stage and difficulty

**Attack functions** (lines 1324–1370):
- `fireRadial(boss, count, speed, offset)`: Circular bullet spray
- `fireAimedSpread(boss, count, speed, spreadStep)`: Fan toward player
- `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`: Specialized patterns
- All spawn `enemyBullets[]` with `{ radial: true, dx, dy }` for velocity-based movement

### 8. Collision Detection
Simple AABB overlap (line 1069): `rectsOverlap(a, b)` checks distance between centers < half-widths.

**Collision processing order**:
1. **Enemy-bullet** (line 1232): Damage enemies, spawn particles, destroy bullets, award score
2. **Boss-bullet** (line 1255): Damage boss (3 HP per hit), destroy bullet
3. **Enemy-player** (line 1268): Trigger `loseLife()`, remove enemy
4. **Enemy bullet-player** (line 1276): Trigger `loseLife()`, remove bullet
5. **Boss-player** (line 1284): Trigger `loseLife()` (no invuln during boss entry phase)
6. **Power-up-player** (line 1289): Apply power-up type, remove power-up

### 9. Scoring & Chain Mechanics
- **Score formula** (line 1091): `score += base * multiplier`
- **Multiplier**: `1 + Math.min(9, Math.floor(chain / 5))`—max 10x at 50+ consecutive kills
- **Chain timer** (line 1089): 2.2 seconds; resets on each kill; expires → `breakChain()` resets to 1x
- **Boss defeats**: Award 500 points + explosion

### 10. Power-up System
Enemies drop power-ups (6%, 8%, 20% chance):
- **Life**: Restore 1 HP (up to max)
- **Bomb**: Restore 1 bomb (up to max)
- **Weapon**: Upgrade weapon level or swap weapon/reset level

`applyPowerUp(type)` (line 1304) handles application and sounds.

### 11. Particle & Visual Effects
- **Particles** (line 1073): Velocity-based mini projectiles with fade-out
- **Screen shake** (line 722): `screenShake(magnitude, duration)` translates canvas rendering
- **Flash effects** (line 699): Red vignette on low health; white flash on hit; screen-wide flash for bomb
- **Drawing**: Planes use canvas path drawing (`drawPlane`, line 913); bosses use `drawBossShip` (line 970)

### 12. Audio System
Centralized `sfx` object with lazy-loaded Web Audio context:
- `ensureAudio()` (line 541): Initializes audio on first user interaction
- Sound effects: shoot, laser, explode (3 types: grunt/interceptor/tank), powerup, bomb, hit, bossTelegraph, bossHit, bossDown, stageClear, gameover, uiClick, mute toggle
- No music loop—sound effects only

### 13. Rendering Themes & Visual Progression
`THEMES` array (line 493) cycles per stage with unique sky gradients, sun color, and cloud appearance:
- Gradient sky layers at 0%, 45%, 100%
- Parallax clouds at two speeds (0.5, 1.2) with semi-transparency
- Dynamic `drawClouds()` (line 1507) reuses ellipse patterns for efficiency

## Development Notes

### Adding New Enemy Types
1. Define in `ENEMY_TYPES` (line 502) with `{ w, h, speed, hp, color, dark, score, minStage }`
2. Spawn logic in `spawnEnemy()` (line 1022) auto-includes if `minStage <= stage`
3. Drawing uses generic `drawPlane()` with type colors

### Adding New Boss Patterns
1. Define attack functions in the pattern functions section (lines 1324–1370)
2. Add boss design to `BOSSES` (line 508) with `{ name, hpMul, wing, hullTop, hullBottom, core }`
3. Create 3-phase pattern array in `BOSS_PATTERNS` and push (order matches `BOSSES` index)

### Adding New Difficulty Tiers
1. Add key to `DIFFICULTIES` (line 485) with spawn, enemy speed, and boss HP parameters
2. Update difficulty button HTML (lines 412–414) to match key
3. Difficulty persists in localStorage under `skyfighter.difficulty`

### Tuning Performance
- **Quality setting** (line 513): 0 (high glow), 1 (medium), 2 (low)—disables shadow effects at quality > 0
- **Particle count** (line 1074): Scales with quality
- **Canvas resolution**: Fixed at 480×720 (BASE_W, BASE_H); rendered to fit screen

### Data Persistence
- `localStorage` keys:
  - `skyfighter.bestScore`: High score
  - `skyfighter.muted`: Audio mute state
  - `skyfighter.difficulty`: Last selected difficulty
  - `skyfighter.reducedMotion`: Accessible motion reduction toggle

## Common Development Tasks

**Testing a weapon upgrade path**: Modify power-up spawn chance in `maybeSpawnPowerUp()` (line 1059) from 0.34 to 1.0 to always spawn weapon pickups.

**Adjusting enemy difficulty**: Tweak `DIFFICULTIES` spawn timers (`spawnBase`, `spawnMin`) or enemy speed multipliers in `ENEMY_TYPES`.

**Fine-tuning boss attacks**: Modify `BOSS_PATTERNS` array indices or adjust bullet counts/speeds in attack functions (`fireRadial`, `fireAimedSpread`, etc.).

**Disabling visual effects for debugging**: Set `reducedMotion = true` in `resetGame()` to skip screen shake and particle effects.

## Performance Considerations
- Single-threaded, no web workers
- Canvas 2D only (no WebGL)
- ~1670 lines of code, all in global scope within IIFE
- Minimal garbage allocation during play (pre-sized arrays, object reuse where possible)
- Frame data: ~60 FPS target, clamped dt to 0.05s prevents frame-skip cascades

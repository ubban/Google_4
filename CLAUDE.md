# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML file with no external dependencies. The game runs directly in browsers with:
- Canvas-based 2D rendering
- Procedurally synthesized sound effects (Web Audio API)
- Responsive design for desktop, tablet, and mobile
- LocalStorage for persistence (best score, settings)

The entire game code (~1670 lines) is in `index.html`: HTML structure, embedded CSS, and pure JavaScript game logic.

## Architecture Overview

### High-Level Structure

The game follows a classic game loop pattern with these major phases:

1. **Initialization** (top of script): Set up DOM references, event listeners, and game state
2. **Game Loop** (`loop()` function, line 1640): Runs ~60 FPS via `requestAnimationFrame`
   - **Update phase**: Process input, move entities, detect collisions, spawn waves
   - **Draw phase**: Render sky, clouds, all sprites, particles, UI
3. **State Management**: Player stats, enemies, bullets, boss, particles managed in module scope

### Core Game Systems

#### Input Handling (lines 768-861)
- **Keyboard**: Arrow keys / WASD for movement, Space / X / Shift for firing / bombing
- **Mouse**: Player glides toward cursor when mouse is over canvas; click to fire
- **Touch**: Virtual joystick for movement (left zone) + Fire/Bomb buttons (right zone)
- **Priority**: Keyboard/joystick input takes priority over mouse to prevent fighting controls

#### Entity Management
- **Player** (`player` object, line 877): Position, weapon type/level, cooldown, invincibility
- **Enemies** (array, spawned each frame): 3 types (grunt, interceptor, tank) with different HP/speed/behavior
- **Bullets** (player and enemy): Tracked separately; player bullets have pierce/laser properties, enemy bullets are radial
- **Particles**: Spawn on hits/explosions; fade out over 0.5s
- **Power-ups**: Life, bomb, weapon upgrades; fall down screen at 90 px/s

#### Collision System (line 1069)
Simple rect-based: `rectsOverlap(a, b)` checks if axis-aligned boxes overlap. Used for:
- Bullet-to-enemy/boss
- Enemy/boss-to-player
- Power-up pickup

#### Difficulty Scaling (line 485)
Defined in `DIFFICULTIES` object with parameters:
- `spawnBase` / `spawnMin`: Enemy spawn rate thresholds (dynamically ramp difficulty over time)
- `enemySpeed`, `enemyShootChance`: Affect individual enemy behavior
- `lives`, `bombs`: Starting resources
- `bossHpMul`: Boss health multiplier

Additionally, adaptive quality reduction (line 474): If frame time > 1/40s, reduce particle count / glow effects to maintain 60 FPS.

#### Audio System (lines 543-676)
- **No audio files**: All sounds synthesized via Web Audio API (`AudioContext`)
- **SFX library** (`sfx` object): 14 different effects (shoot, laser, explode variants, hit, bomb, powerup, boss sounds, etc.)
- **Synthesis**: Combination of oscillators, noise bursts, and filters (lowpass/highpass)
- **Spatialization**: Pan effects based on where sound originates (canvas coordinates)
- **Muting**: Controlled by `muted` flag; stored in localStorage

#### Scoring & Multiplier System (lines 1087-1101)
- **Chain**: Incremented on each enemy kill; resets if >2.2s without a kill
- **Multiplier**: 1 + floor(chain/5), capped at 10x
- **Combo pop**: Visual feedback at 10, 20, 30... kills

#### Boss System (lines 507-1454)
- **6 boss types** (BOSSES array, line 508): Each has unique visuals (hull colors, core), health multiplier, and attack patterns
- **Cycle**: Bosses repeat with different stages (stage 1 = VANGUARD, stage 7 = VANGUARD again, etc.)
- **Attack patterns** (BOSS_PATTERNS array, line 1372): Each boss has 3 attack patterns (rotated cyclically)
  - Patterns use helper functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.
  - Attacks telegraph 22 frames before firing (visual effect + SFX warning)
  - Boss moves left/right; reverses at canvas edges
- **Entering animation**: Boss slides in from top; stays invulnerable during entering phase

#### Weapon System (lines 871-1146)
Two weapons, each with 3 levels:
- **Vulcan**: Rapid-fire spread that increases with level (1 shot → 2 → 3 lanes)
- **Laser**: Piercing beams; fires up to 3 lanes at level 3; slower fire rate but higher damage (3 HP per hit vs 1 for bullets)
- **Switching**: After level 3, next pickup switches weapon and resets level to 1

#### Visual Themes (line 493)
Six stage themes (sky gradient, cloud colors, sun color). Each theme cycles through stages:
- Theme changes every 6 stages
- Affects background rendering but not gameplay

#### Rendering Pipeline (lines 1520-1638)
Draws from back to front:
1. Sky gradient + sun glow
2. Animated clouds (2 layers at different speeds)
3. Player plane (with flame/laser aura)
4. Player bullets (with glow if quality = 0)
5. Enemy bullets (red circles)
6. Enemy planes + tank health bars
7. Boss plane (complex gradient + animated core)
8. Falling power-ups (rotating)
9. Particle effects (exploding dots)
10. Vignette/flash overlays (managed by ID's overlaid on canvas)

### Performance Considerations

**Adaptive Quality** (line 474): Reduces expensive effects when FPS drops:
- Quality 0: Full effects (16+ particles per explosion, shadowBlur, etc.)
- Quality 1: ~60% particles, lighter shadows
- Quality 2: ~35% particles, no shadows

**Canvas Scaling** (line 753): 
- Game logic runs at fixed 480x720 resolution
- CSS scales canvas up to fit window (max 1.6x) while maintaining aspect ratio
- Prevents GPU thrashing and keeps game logic frame-rate-independent

**Collision Optimization**:
- Bullets filtered out-of-bounds each frame (line 1189)
- Enemies culled when past bottom of canvas (line 1228)
- Particle cleanup based on lifetime (line 1297)

## How to Work with This Codebase

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` (line 502), e.g.:
   ```javascript
   drone: { w: 30, h: 30, hp: 2, speedMul: 1.3, color: '#00ff00', dark: '#00aa00', score: 20, minStage: 3 }
   ```
2. Update `spawnEnemy()` (line 1022) to include the new type in the spawn pool if needed
3. Add explosion SFX to `EXPLODE_SFX` (line 678) if unique sound desired
4. Add SFX function if needed (e.g., `sfx.explodeDrone()`)

### Adding a New Boss

1. Add entry to `BOSSES` array (line 508), e.g.:
   ```javascript
   { name: 'PHOENIX', hullTop: '#ff6b00', hullBottom: '#cc2200', wing: '#ff3300', core: '#ffff00', hpMul: 1.2 }
   ```
2. Add attack pattern array to `BOSS_PATTERNS` (line 1372) with 3 pattern functions
3. Boss spawning is automatic (line 1204-1207): After 18 kills per stage, boss spawns
4. Color scheme and hpMul control visuals/difficulty; patterns control attack behavior

### Adding a New Attack Pattern

Boss attack patterns go in `BOSS_PATTERNS`. Use helper functions:
- `fireRadial(boss, count, speed)`: Bullets in all directions from boss
- `fireAimedSpread(boss, count, speed, spreadStep)`: Aimed at player with spread
- `fireColumns(boss, speed)`: 5 vertical columns
- `fireSpiralBurst(boss, speed)`: 8-way radial with rotating offset per attack
- `fireCross(boss, speed)`: 8-way cardinal + diagonal
- `fireHomingSeeded(boss, count, speed)`: Homing-ish bullets with player-targeting noise
- `fireDoubleRing(boss, speedA, speedB)`: Concentric rings at two speeds

Custom patterns can also directly push to `enemyBullets` array (see line 1377 for example).

### Tweaking Game Balance

Key adjustable parameters:
- **Enemy spawning**: `DIFFICULTIES[difficulty].spawnBase` / `spawnMin` (line 485)
- **Boss HP**: Adjust `hpMul` in BOSSES or `bossHpMul` in DIFFICULTIES
- **Weapon damage**: Player bullets do 1 HP, lasers do 3 HP, bombs do 24 HP (see lines 1239, 1259, 1118)
- **Enemy behavior**: `enemySpeed`, `enemyShootChance` in DIFFICULTIES
- **Scoring**: Adjust `score` values in ENEMY_TYPES / powerup scoring
- **Resources**: `lives` and `bombs` in DIFFICULTIES

### Modifying Visuals

#### Rendering
- Player plane: `drawPlane()` (line 913) — simple geometry, modify points for different shape
- Boss plane: `drawBossShip()` (line 970) — uses boss design colors; easy to tweak without changing shape
- Enemy bullets: Hardcoded red circles (line 1598); change `ctx.fillStyle = '#ff5252'` to new color
- Particles: Color passed at spawn (line 1076), size/opacity calculated based on lifetime (line 1629)

#### Sky/Theme
Add new theme to `THEMES` array (line 493). Theme cycles every 6 stages automatically.

#### HUD
- DOM structure in HTML (lines 368-379, 384-388, 392-419)
- Styled in CSS (lines 10-358)
- Updated via `scoreEl.textContent = score`, etc. throughout game logic
- Hearts/bombs rendered via `renderHearts()` / `renderBombs()` (lines 680-696)

### Audio
- To add a new SFX: Create function in `sfx` object using `beep()` and `noiseBurst()` helpers
- `beep(freq, duration, type, volume, opts)`: Oscillator-based tone
  - `freq`: Starting frequency in Hz
  - `type`: 'sine', 'square', 'sawtooth', 'triangle'
  - `opts.sweepTo`: Frequency ramp target
  - `opts.filterFreq` / `filterType`: Lowpass/highpass filter
  - `opts.pan`: -1 (left) to 1 (right)
- `noiseBurst(duration, volume, opts)`: Filtered white noise
- `panFor(x)`: Converts canvas X coordinate to stereo pan value

To mute/unmute: Toggle `muted` flag (line 735)

## Important Notes

### No Build Step
This is a zero-dependency game. Just open `index.html` in a browser. No npm, bundler, or compilation.

### Mobile Responsiveness
- Touch controls automatically enabled if `ontouchstart` detected (line 779)
- Canvas scales to fit viewport (line 754-765)
- Safe area insets respected for notched devices (env safe-area-inset-*)

### Accessibility
- Reduced motion mode (line 521): Respects `prefers-reduced-motion` media query; toggleable via UI button
- Screen reader announcements via `announce()` function (line 468) → `#srLive` element
- Keyboard navigation: P/Esc to pause, X/Shift to bomb

### LocalStorage Keys
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio muted state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Motion preference override

### Common Edits Quick Reference
| What | File | Lines | Notes |
|------|------|-------|-------|
| Enemy types | ENEMY_TYPES | 502-506 | Add new types here |
| Bosses | BOSSES | 508-515 | Add with unique colors/hpMul |
| Boss patterns | BOSS_PATTERNS | 1372-1409 | 3 patterns per boss |
| Themes | THEMES | 493-500 | Sky gradients, cloud colors |
| Difficulty | DIFFICULTIES | 485-489 | Spawn rates, enemy stats, health |
| Weapon balance | fireWeapon() | 1124-1146 | Bullets, spread, damage |
| UI text/layout | HTML / CSS | 1-359, 361-420 | Overlay, HUD, buttons |
| Sound | sfx object | 624-676 | Audio synthesis functions |

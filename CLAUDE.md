# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML5 file with embedded CSS and JavaScript. No build tools or external dependencies are required—the game runs directly in any modern browser.

## Quick Start

### Running the Game

1. Open `index.html` directly in a web browser (desktop or mobile)
2. Use mouse movement to steer, click to fire, or keyboard controls (arrow keys/WASD, Space to fire, X/Shift for bombs)
3. On mobile: drag left to move, tap FIRE button, tap BOMB button

### Development

- **Edit**: Make changes directly to `index.html`
- **Test**: Save and refresh the browser
- **No build step required**—changes take effect immediately

There are no external dependencies, linters, or test suites to configure. This is a pure client-side game.

## Code Architecture

### Overview Structure
All code is contained in a single `index.html` file (1670 lines) organized into logical sections:

```
1. HTML Structure + Metadata (lines 1-361)
   - DOM elements for canvas, HUD, overlay, touch controls
   - Embedded CSS with responsive sizing and animations

2. JavaScript Game Engine (lines 422-1666)
   - Initialization & state management
   - Audio synthesis system (WebAudio)
   - Input handling (keyboard, mouse, touch)
   - Game state & entities (player, enemies, bullets, boss)
   - Update loop (entity movement, collision, spawn logic)
   - Rendering system (canvas drawing)
   - Main game loop with requestAnimationFrame
```

### Key Game Systems

#### 1. **Entity System**
- `player`: Player aircraft with weapon, level, health, invulnerability frames
- `enemies`: Array of enemy ships with HP, movement patterns, shooting behavior
- `bullets`: Array of player projectiles (Vulcan or Laser)
- `enemyBullets`: Array of incoming enemy fire
- `powerUps`: Weapon upgrades, health, bombs (randomly spawn on enemy death)
- `boss`: Single boss entity per stage with unique attack patterns
- `particles`: Temporary visual effects from explosions

#### 2. **Combat & Progression**
- **Weapons**: Two primary weapon types with upgrade levels:
  - **Vulcan**: Spread shot, faster fire rate (cooldown: 0.15s)
  - **Laser**: Piercing shots, slower fire rate (cooldown: 0.1s), can penetrate enemies
  - Levels 1–3 increase spread/lanes; switching weapons resets level to 1
- **Health System**: 
  - Lives vary by difficulty (3–4)
  - Invulnerability frames (1.5s) after hit
  - One heart container = one hit (no partial damage)
- **Bombs**: Smart bomb clears all enemies/bullets, deals 24 damage to boss
- **Scoring**:
  - Base points per enemy type (grunt: 10, interceptor: 15, tank: 35, boss: 500)
  - Multiplier system: +1x per 5 consecutive kills (max 10x), decays after 2.2s of no kills
  - Chain breaks on player hit or time expiration

#### 3. **Boss System**
- One boss per stage (cycles through 6 designs: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Boss enters from top, patrols side-to-side at y=120
- Each boss has 3 unique attack patterns (loops cyclically)
- Telegraph phase: 22 frames warning before attack (visual & audio cue)
- Attack patterns include: radial bursts, homing missiles, aimed spreads, columns, teleport barrages, expanding rings, cross fire
- Boss HP scales with stage and difficulty (base 70 + stage*35, multiplied by difficulty/design factors)
- Kill triggers stage clear, score bonus, stage banner, and next stage spawn

#### 4. **Difficulty System**
Three presets stored in `DIFFICULTIES` object (line 485–489):
- **Easy**: Slower spawns, slower enemies, 4 lives, 3 bombs, 0.8x boss HP
- **Normal**: Balanced (default), 3 lives, 2 bombs, 1.0x boss HP
- **Hard**: Fast spawns, fast enemies, 3 lives, 2 bombs, 1.3x boss HP

Spawning speeds ramp over time (every 300 frames = ~5s, spawn timer decreases by 4).

#### 5. **Audio System (WebAudio Synthesis)**
- `ensureAudio()`: Lazily initializes AudioContext on first user interaction
- `beep()`: Generates square/sawtooth/sine waves with frequency sweeps, filters, panning
- `noiseBurst()`: White noise with lowpass filtering for explosive sounds
- All sounds are procedurally generated; no external files
- Master gain compressor prevents clipping
- Pan effect based on horizontal position (stereo positioning for bullets/explosions)
- Can be muted via UI button (persists in localStorage)

#### 6. **Input System**
- **Keyboard**: Arrow keys / WASD (movement), Space (fire), X / Shift (bomb), P / Escape (pause)
- **Mouse**: Movement steers player toward pointer (when hovering canvas), click to fire
- **Touch**: 
  - Left 55% of screen: joystick for movement (constrained to 50px radius)
  - Right 45% of screen: FIRE and BOMB buttons
- Input priority: Keyboard/joystick > mouse steering (mouse acts as fallback)

#### 7. **Visual Rendering**
- **Canvas**: 480×720 base resolution, scales up to 1.6× max
- **Background**: Stage-dependent sky gradient with parallax clouds and sun glow
- **Planes**: Custom-drawn with filled paths (player, enemies, boss all use `drawPlane()` or `drawBossShip()`)
- **Effects**: 
  - Screen shake on hits/bombs (dampened if reduced-motion enabled)
  - Flash overlay on hit/bomb
  - Vignette pulse when critical health (1 life)
  - Combo pop-up animations at milestones (10, 20, 30… kills)
  - Stage banner on stage clear
- **Glow**: Conditional shadow/blur on bullets and powerups (disabled at lower quality)

#### 8. **Performance Adaptation**
- `quality` variable: 0 (full) → 1 (medium) → 2 (low)
- Tracked via rolling 45-frame average of delta-times
- Upgrades quality if avg FPS > 40, downgrades if avg < 56 FPS
- Lower quality reduces particle count in explosions and disables shadow effects
- Adjusts canvas rendering overhead

#### 9. **Accessibility & UX**
- Screen reader announcements via `announce()` (sr-only live region)
- Reduced motion support: Matches `prefers-reduced-motion: reduce` media query
  - Disables animations (vignette pulse, heartbeat, flash, screen shake)
  - Can be toggled manually via UI button
- Mute button for audio
- Difficulty selector on start screen
- Touch controls for mobile devices
- Pause functionality (P key or button)

#### 10. **Persistence**
LocalStorage keys (prefixed `skyfighter.`):
- `bestScore`: High score (number)
- `muted`: Audio mute state (1 or 0)
- `difficulty`: Last selected difficulty (easy/normal/hard)
- `reducedMotion`: Accessibility setting (1 or 0)

### Key Constants & Configuration

#### Difficulty Presets (`DIFFICULTIES` object, line 485)
```javascript
{ spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul }
```

#### Stage Mechanics (`KILLS_PER_STAGE`, line 491)
- 18 kills per stage triggers boss warning (after 1.2s delay)
- 6 visual themes cycle (Daylight Front → Sunset Corridor → Night Approach → Storm Front → Arctic Wastes → Volcanic Ridge)

#### Enemy Types (`ENEMY_TYPES` object, line 502)
```javascript
{ w, h, hp, speedMul, color, dark, score, minStage, weave? }
```
- **Grunt**: 34×34, 1 HP, red, stage 1+
- **Interceptor**: 26×26, 1 HP, faster, orange, stage 1+, weaves horizontally
- **Tank**: 46×46, 4 HP, purple, slower, stage 2+, shows HP bar

#### Boss Designs (`BOSSES` array, line 508)
```javascript
{ name, hullTop, hullBottom, wing, core, hpMul }
```
6 bosses with unique color schemes and HP multipliers (0.8–1.7×).

#### Weapon Fire Patterns
- **Vulcan** (`fireWeapon()`, line 1124): Spread based on level (1/2/3 bullets)
- **Laser** (`fireWeapon()`, line 1136): Piercing shots, 1–3 lanes depending on level
- **Boss patterns** (`BOSS_PATTERNS`, line 1372): 6 distinct pattern arrays (one per boss design)
  - Each pattern has 3 attack functions: radial, aimed spread, homing, columns, spirals, crosses, teleports, etc.

### Data Flow

1. **Input** → Key/touch/mouse handlers set state (keys[], joyVec, firing)
2. **Update** (line 1148):
   - Move player based on input
   - Fire weapon if cooldown allows
   - Move bullets & enemy bullets
   - Advance enemy AI (weaving, shooting)
   - Update boss behavior (telegraph, attack patterns)
   - Collision detection (bullets vs enemies/boss, enemies/bullets vs player)
   - Apply powerups
   - Spawn new enemies or boss
   - Update particles
3. **Draw** (line 1520): Render all entities to canvas with effects
4. **Loop** (line 1640): requestAnimationFrame calls update → draw every frame

## Development Notes

### Making Changes

**Adding a new enemy type:**
1. Add entry to `ENEMY_TYPES` object (line 502)
2. Update `minStage` to control when it spawns
3. Optionally add a new explode sound effect to `sfx` object

**Adding a new boss:**
1. Add entry to `BOSSES` array (line 508)
2. Create 3 attack pattern functions in a new array and append to `BOSS_PATTERNS`
3. Boss will automatically cycle through when stage count wraps

**Tweaking difficulty/balance:**
- Edit `DIFFICULTIES` object values
- Adjust `KILLS_PER_STAGE` (line 491) to change stage length
- Modify weapon cooldowns in `fireWeapon()` function
- Adjust enemy spawn timers or damage values

**Tuning visual effects:**
- Screen shake: `screenShake()` function (line 727), adjust magnitude & duration
- Colors: All hardcoded; update hex values directly
- Animation timings: Adjust `@keyframes` durations in CSS or particle life values

### Performance Considerations

- **Canvas rendering**: Uses 2D context, no WebGL (simpler, sufficient for 480×720)
- **Collision**: AABB (axis-aligned bounding box) via `rectsOverlap()` — O(n²) for enemies vs bullets, acceptable at <100 entities
- **Particles**: Capped implicitly by adaptive quality system
- **Audio**: WebAudio synthesis happens in real-time; minimal overhead
- **Mobile**: Touch input is pointer-events based (efficient); joystick constrained to 50px radius to prevent excessive calculations

### Testing Checklist

- [x] Start game at each difficulty level
- [x] Fire both weapon types and verify spread patterns
- [x] Collect all powerup types (life, bomb, weapon)
- [x] Trigger bomb (clear enemies, damage boss)
- [x] Reach boss fight, verify boss health bar and attack patterns
- [x] Test pause/resume
- [x] Test mute/unmute
- [x] Test reduced motion toggle
- [x] Verify high score persistence
- [x] Test on mobile (touch controls, responsive canvas size)
- [x] Verify keyboard controls (arrows, WASD, space, X, P, Escape)
- [x] Verify mouse steering and click-to-fire
- [x] Game Over screen shows correct stats
- [x] Difficulty selection persists across sessions

### Future Enhancement Ideas (Not Implemented)

- Combo screen popups at higher milestones
- Boss AI phase transitions (harder attacks at low HP)
- Weapon-specific power-ups
- Destructible terrain or cover
- Local 2-player split-screen
- Leaderboard (would require backend)
- Replays or death cam

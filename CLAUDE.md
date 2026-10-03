# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single self-contained HTML file with no external dependencies. Players navigate a fighter jet, switch between weapons, collect power-ups, and battle stage bosses across multiple difficulty levels.

### Key Characteristics
- **Single-file architecture**: All HTML, CSS, and JavaScript in `index.html`
- **No build process**: Open directly in any modern browser
- **No external assets**: All graphics rendered procedurally via Canvas 2D, all audio synthesized via WebAudio API
- **Platform support**: Desktop (mouse/keyboard), mobile (touch with joystick UI)
- **Responsive design**: Adapts to different screen sizes and device orientations

## Running the Game

1. Open `index.html` directly in a web browser
2. Click "Start Game" to begin playing
3. Use controls:
   - **Desktop**: Mouse to steer (crosshair), click to fire, arrow keys/WASD to move, Space to fire, X/Shift to bomb
   - **Mobile**: Drag left side to fly, tap FIRE button, tap BOMB button
   - **Pause**: Press P or Esc

The game persists player progress (best score, difficulty, mute state, motion preferences) to `localStorage`.

## Architecture & Game Systems

### Core Game Loop
- **`loop(now)`** (line 1640): Main requestAnimationFrame loop
  - Tracks performance (`trackPerf()`) and adjusts quality level
  - Calls `update(dt)` for game logic (when not paused)
  - Calls `draw()` to render the frame
  - Runs at ~60 FPS, capped at 50ms delta time per frame

### Game State
- **Player**: Position, velocity, weapon/level, cooldown, invulnerability timer
- **Entities**: Arrays of `enemies`, `bullets`, `enemyBullets`, `particles`, `powerUps`
- **Progression**: `stage`, `stageKills`, `kills` for stage completion; `chain`/`multiplier` for scoring
- **UI state**: `lives`, `bombCount`, `score`, `multiplier`

### Key Systems

#### 1. Input Handling (lines 768–861)
- **Keyboard**: `keys` object tracks held keys; P/Esc toggles pause
- **Touch**: Joystick emulation on left half of screen; FIRE/BOMB buttons on right
- **Mouse**: Free steering when hovering canvas; click to fire (priority over keyboard/joystick)
- **Audio activation**: Any input calls `ensureAudio()` to resume audio context

#### 2. Rendering & Visual Effects (lines 520–1638)
- **Canvas setup**: 480×720 base resolution, scaled to fit viewport (max 1.6× scale)
- **Theme system** (line 493): 6 rotating sky gradients with cloud parallax effects
- **Entity drawing**:
  - `drawPlane()` (line 913): Procedural fighter jet rendering
  - `drawBossShip()` (line 970): Boss with animated core glow
  - Bullets, particles, power-ups drawn with optional shadows (quality-dependent)
- **Visual feedback**:
  - `screenShake()` for impacts (respects reduced-motion preference)
  - `flashHit()` / `flashBomb()` for hit effects
  - `showComboPop()` for kill streaks, `showStageBanner()` for stage transitions
  - Vignette pulse when health critical

#### 3. Weapon System (lines 871, 1124–1146)
Two weapon types progress independently:
- **Vulcan** (default): Rapid-fire projectiles with spread at levels 2–3
- **Laser**: Single/dual/triple beam with pierce capability (goes through multiple enemies)
- **Power-ups** advance weapon level or switch between weapons; resets level to 1 on switch

#### 4. Enemy System (lines 1022–1041, 1210–1228)
- **Types**: `grunt`, `interceptor`, `tank` with different stats
  - HP, speed multiplier, shooting pattern, appearance, score value
  - New enemy types scale with `stage` (tank appears stage 2+)
- **Spawn control**: Difficulty affects spawn rate and min spawn intervals
- **AI**: Weaving patterns (interceptor), random shoot timers, two fire patterns
  - `pattern: 'spread'` fires 3-bullet spread
  - `pattern: 'aimed'` fires toward player

#### 5. Boss System (lines 508–515, 1043–1056, 1411–1454)
- **Boss data**: 6 designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) cycle per stage
- **Appearance**: Procedurally drawn with gradient hulls, glowing core, animated red engine ports
- **Attack patterns** (line 1372–1409): 3 patterns per boss (cycles through)
  - Radial/aimed spreads, homing barrages, teleporting dashes, dense bullet walls
- **Telegraph mechanic**: Boss pauses 22 frames with white glow before firing
- **Health scaling**: Base HP 70 + (stage × 35), modified by difficulty and boss multiplier

#### 6. Difficulty System (line 485–489)
- **Easy**: Slow spawn, weak enemies, 4 lives, weaker boss (0.8×)
- **Normal**: Balanced (default), 3 lives
- **Hard**: Fast spawn, aggressive enemies, 3 lives, tougher boss (1.3×)
- **Persistence**: Stored in localStorage; selected from menu before starting

#### 7. Audio System (lines 542–676)
- **WebAudio synth**: Dynamic oscillators, filters, gain envelopes (no external files)
- **Sound effects**: Shoot, laser, explosions (type-specific), hit, bomb, powerup, boss patterns, game over
- **Panalization**: Stereo panning based on event position on screen
- **Mute state**: Stored in localStorage

#### 8. Collision & Damage (lines 1069–1282)
- **Rectangle overlap**: `rectsOverlap(a, b)` for all collision checks
- **Hit resolution**:
  - Player bullets vs. enemies: Laser pierces based on level
  - Player bullets vs. boss: 3 damage per bullet
  - Enemy bullets vs. player: Loses 1 life, 1.5s invulnerability
  - Direct contact: Loses 1 life, break combo chain
- **Invulnerability**: Player blinks every 4 frames while invuln > 0

#### 9. Progression & Scoring (lines 1087–1101, 1246–1449)
- **Chain multiplier**: Increases by 1 per 5 consecutive kills (max 1 + 9×)
- **Chain timer**: Resets if 2.2s pass without killing
- **Stage progression**: Kill 18 enemies → boss warning → boss spawns after 1.2s
- **Boss defeat**: 500 base score + chain multiplier, advance stage, show theme banner
- **Best score tracking**: Persisted to localStorage, displayed on game over

#### 10. Performance Adaptation (lines 474–483)
- **Quality levels**: 0 (full effects), 1 (50% particles/shadows), 2 (minimal)
- **Dynamic adjustment**: Samples last 45 frames; lowers quality if avg frame time > 1/40s, raises if < 1/56s
- **Impact**: Affects particle count, shadow blur, glow effects

### Data Structures

```javascript
// Player
{ x, y, w: 40, h: 40, speed: 260, cooldown, rapid, weapon: 'vulcan'|'laser', 
  level: 1-3, invuln }

// Enemy
{ x, y, w, h, hp, speed, shootTimer, canShoot, pattern: 'spread'|'aimed', 
  type: 'grunt'|'interceptor'|'tank', color, dark, score, weave, weaveSeed, baseX }

// Boss
{ x, y, w: 90, h: 80, hp, maxHp, entering, phaseTimer, attackTimer, patternIndex, 
  vx, design, telegraph }

// Bullet (player)
{ x, y, w, h, speed, vx, pierce, laser }

// Bullet (enemy)
{ x, y, w, h, speed, vx } | { x, y, w, h, radial, dx, dy } // radial for boss attacks

// Particle
{ x, y, vx, vy, life, maxLife, color }

// Power-up
{ x, y, w: 20, h: 20, speed: 90, type: 'life'|'bomb'|'weapon' }
```

## Adding Features

### New Enemy Type
1. Add to `ENEMY_TYPES` (line 502) with stats: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Update `spawnEnemy()` (line 1024) spawn rate logic if needed
3. Add explosion sound to `EXPLODE_SFX` (line 678) if unique

### New Boss Pattern
1. Create pattern function in `BOSS_PATTERNS` (line 1372) that calls helper functions:
   - `fireRadial(b, count, speed, offset)`
   - `fireAimedSpread(b, count, speed, spreadStep)`
   - `fireColumns(b, speed)`
   - `fireSpiralBurst(b, speed)`
   - `fireHomingSeeded(b, count, speed)`
   - `fireCross(b, speed)`
   - `fireDoubleRing(b, speedA, speedB)`

### New Boss Design
1. Add to `BOSSES` (line 508) with colors: `hullTop`, `hullBottom`, `wing`, `core`, `hpMul`
2. Create 3 attack patterns in `BOSS_PATTERNS` for that boss index

### New Theme
1. Add to `THEMES` (line 493): `sky` array (3 colors), `cloud`, `sun` (RGB strings), `name`
2. Themes cycle automatically; themes are selected by `(stage - 1) % THEMES.length`

## Constraints & Notes

### Browser APIs Required
- Canvas 2D API (rendering)
- WebAudio API (sound synthesis)
- Touch Events (mobile)
- localStorage (persistence)
- requestAnimationFrame (animation loop)

### Performance Considerations
- Quality level dynamically adjusts particle count to maintain 60 FPS
- Shadow/glow effects disabled at quality level 2
- Max delta time capped at 50ms to prevent spiral lag on frame drops
- Enemy/boss bullet limits: No hard cap, but dense patterns can stress older devices

### Reduced Motion Compliance
- Respects `prefers-reduced-motion` media query on first load
- User toggle persisted to localStorage
- When enabled: Disables all CSS animations, screen shake, flash effects, pulse effects
- Logic: `if (reducedMotion) return;` before visual feedback

### Known Limitations
- No fullscreen API (intentional for simplicity)
- Single-threaded; no Web Workers
- Audio cannot be pre-cached; synthesized on-demand
- Mobile performance varies significantly by device

## Testing the Game

### Manual Test Checklist
- Start game, reach stage 2 (kill 18 enemies)
- Switch difficulty levels before starting
- Test all weapons (vulcan levels 1-3, laser levels 1-3, switch between)
- Test bomb mechanic (clear all enemies and bullets)
- Verify combo multiplier (11 consecutive kills = 2×)
- Test losing all lives and viewing game over screen
- Toggle mute and reduced motion in top-right buttons
- On mobile: Test joystick left, fire button right, bomb button
- Test pause/resume with P key or pause button

### Debug Tips
- Open DevTools console to check `quality`, `stage`, `multiplier`, `chain`
- Look at frame rate in DevTools Performance tab
- `localStorage.clear()` resets all saved progress
- Modify `DIFFICULTIES[difficulty]` in DevTools console to test balance

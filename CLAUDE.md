# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single-file HTML/CSS/JavaScript application. The game runs entirely in the browser with no external dependencies—all audio is synthesized procedurally using the Web Audio API, and all graphics are rendered via Canvas.

## Running & Testing

Since this is a single HTML file, you can run it directly:

- **Local testing**: Open `index.html` in any modern web browser (Chrome, Firefox, Safari, Edge)
- **Web server**: For development, serve via `python -m http.server 8000` (or `npx http-server`) then open `http://localhost:8000`
- **No build step**: Changes to `index.html` take effect immediately after refresh

## Architecture Overview

### Single-File Structure
All game code (HTML, CSS, JavaScript) is in `index.html`. The architecture is organized as:

1. **HTML Structure** (lines 362–420): Defines the canvas, HUD elements, overlays, and touch controls
2. **CSS Styling** (lines 10–359): Responsive design, animations, HUD styling, dark-mode themed
3. **JavaScript Engine** (lines 422–1666): Self-contained game loop using IIFE pattern

### Core Game Loop
The game uses `requestAnimationFrame` to drive a 60 FPS loop (`loop()` function at line 1640). Each frame:
1. **Update**: `update(dt)` processes input, entity movement, collisions, spawning
2. **Render**: `draw()` renders all game elements to canvas
3. **Performance Tracking**: Adaptive quality system (`trackPerf()`) adjusts particle counts based on frame times

### Key Game Systems

#### 1. Player & Input Handling (lines 768–862)
- **Keyboard**: Arrow keys or WASD for movement; Space for fire; X/Shift for bomb
- **Mouse**: Plane glides toward cursor when hovering canvas; click to fire
- **Touch**: Virtual joystick (left side) + FIRE/BOMB buttons (right side)
- Both keyboard and mouse input are prioritized over joystick to prevent fighting

#### 2. Weapon System (lines 871–1146)
Two weapons with 3-level progression each:
- **Vulcan**: Spreads across levels (single → dual → triple shot). Rapid fire (0.15s cooldown).
- **Laser**: Pierces enemies (up to 3 pierces at level 3). Slower fire (0.1s cooldown).
- Weapon power-ups cycle: Vulcan L1→L2→L3→Laser L1→L2→L3→back to Vulcan L1
- Player level stored in `player.level`; weapon stored in `player.weapon`

#### 3. Enemy System (lines 1022–1041, 1210–1227)
Three enemy types with difficulty-based spawn rates:
- **Grunt**: Basic enemy (red), spawns from stage 1
- **Interceptor**: Fast enemy (orange), spawns from stage 1, weaves side-to-side
- **Tank**: Heavy enemy (purple), spawns from stage 2, 4 HP, high score reward

Enemies spawn with configurable patterns:
- **"spread"**: Bullets fan out in fixed spread
- **"aimed"**: Bullets fire toward player position
- **"weave"** (Interceptors only): Enemies oscillate horizontally while falling

#### 4. Boss System (lines 1043–1056, 1411–1439)
Six rotating bosses (lines 508–515) with unique attack patterns (lines 1372–1409):
1. **VANGUARD**: Radial burst + aimed fan + V-fan attacks
2. **BEHEMOTH**: Heavy columns + homing spread + dense radial
3. **SPECTER**: Teleport dash + spiral + dual aimed shots
4. **DREADNOUGHT**: 8-way cross + homing missiles + radial wall
5. **WRAITH**: Teleport barrage + double ring + cross-spread
6. **TITAN**: Heavy columns + full ring + relentless homing

Boss appears after 18 kills per stage. Boss design cycles by stage (stage modulo 6). Each attack pattern has a telegraph phase (visual warning) before firing.

#### 5. Scoring & Multiplier System (lines 1087–1101)
- Base score per enemy kill determined by `ENEMY_TYPES[type].score`
- **Chain system**: Each kill extends chain by 2.2 seconds; breaks if no kills during window
- **Multiplier**: x1 → x2 at 5 kills → x3 at 10 kills → x10 at 50 kills
- Bomb kills and boss kills also add points (5 per bomb kill, 500 for boss)

#### 6. Difficulty System (lines 485–489)
Three presets adjust spawn rates, enemy speed, shoot chance, and boss HP multiplier:
- **Easy**: More time between spawns (75→40), slower enemies (1.0x), lenient boss (0.8x HP), 4 lives
- **Normal**: Balanced (58→26), standard enemy speed (1.4x), standard boss, 3 lives
- **Hard**: Aggressive (40→18), fast enemies (1.9x), tough boss (1.3x HP), 3 lives

Difficulty stored in localStorage; changeable only before game start or in pause menu.

#### 7. Audio System (lines 542–676)
**Web Audio synth** generates all SFX on-the-fly. No audio files needed. Key functions:
- `beep(freq, dur, type, vol, opts)`: Oscillator with optional sweep, filter, panning
- `noiseBurst(dur, vol, opts)`: Filtered white noise burst
- Individual SFX functions for: shoot, laser, explode variants, powerup, bomb, boss attacks, etc.
- Muted state stored in localStorage; controlled by mute button

#### 8. Visual Effects & Polish
- **Screen shake**: `screenShake(mag, dur)` for explosions, boss hit, bomb
- **Flash overlay**: White flash on player hit, larger flash on bomb
- **Combo pop**: Text animation showing kill streak
- **Stage banner**: "STAGE X" title animation on new stage
- **Vignette effect**: Red pulsing vignette when player has 1 life (critical state)
- **Particles**: Explosion particles fade out over 0.5s, count scales with quality
- All animations respect `prefers-reduced-motion` OS setting

#### 9. Adaptive Quality (lines 474–482)
Monitors average frame time across 45 frames:
- **Quality 0** (full): 60 FPS maintained, all particles rendered, shadow glows enabled
- **Quality 1** (medium): 40 FPS target, particles reduced to 60%, smaller glows
- **Quality 2** (low): 56 FPS target, particles reduced to 35%, minimal glows

Auto-adjusts based on observed frame time to keep game smooth on low-end devices.

#### 10. Responsive Canvas Sizing (lines 752–765)
- Logical canvas: 480×720 (portrait aspect ratio)
- Scales to fit screen while maintaining aspect, max 1.6x scale
- Handles orientation changes and window resize events

## Key Variables & State

### Game State (line 864–869)
```javascript
let player, bullets, enemies, enemyBullets, particles, powerUps, boss;
let score, lives, maxLives, running, paused, gameOver;
let bombCount, maxBombs, spawnTimer, frame, kills, stage, stageKills, bossWarned;
let chain, chainTimer, multiplier;
```

### Player Object Structure
```javascript
player = {
  x, y, w: 40, h: 40,
  speed: 260,
  cooldown,      // weapon fire cooldown
  weapon: 'vulcan' | 'laser',
  level: 1|2|3,
  invuln         // invulnerability timer after hit
}
```

### Entity Collision
All collisions use axis-aligned bounding box: `rectsOverlap(a, b)` at line 1069. Checks `|a.x - b.x| < (a.w + b.w)/2` and same for Y.

## How to Extend

### Add a New Boss
1. Add to `BOSSES` array (line 508–515) with color scheme and HP multiplier
2. Create attack patterns in `BOSS_PATTERNS` array (line 1372–1409)
   - Each pattern is a function taking boss object `b`
   - Use helper functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.
3. Boss automatically appears when stage `% BOSSES.length` rotates

### Add a New Enemy Type
1. Add to `ENEMY_TYPES` object (line 502–506) with width, height, HP, speed multiplier, score, etc.
2. Add `minStage` property to control when it spawns
3. Optionally add `weave: true` for side-to-side oscillation
4. Update `spawnEnemy()` (line 1022) spawn probability logic if needed
5. Add explosion SFX to `EXPLODE_SFX` (line 678) if custom sound desired

### Add a New Weapon
1. Create weapon type in `fireWeapon()` (line 1124)
2. Define spread patterns and bullet properties for 3 levels
3. Update `weaponLabel()` (line 871) to return display name
4. Update power-up cycling logic in `applyPowerUp()` (line 1311–1320)

### Add a Visual Theme
1. Add to `THEMES` array (line 493–500) with sky gradient, cloud color, sun color, and name
2. Theme automatically cycles based on `(stage - 1) % THEMES.length`
3. Theme is announced in screen reader via `announce()` when stage clears (line 1453)

### Adjust Game Balance
- **Difficulty curves**: Modify `DIFFICULTIES` object (line 485–489)
- **Spawn frequency**: Change `spawnBase` and `spawnMin` in difficulties
- **Enemy patterns**: Modify shooting rates and pattern types in `spawnEnemy()`
- **Boss HP scaling**: Adjust `baseHp` formula (line 1046) or `hpMul` in boss designs
- **Scoring**: Change `score` values in `ENEMY_TYPES` and `addScore()` calls

## Browser APIs Used

- **Canvas 2D**: All graphics rendering
- **Web Audio API**: Procedural audio synthesis (oscillators, noise, filters, gain, panning)
- **localStorage**: Persists high score, mute state, difficulty, reduced-motion preference
- **requestAnimationFrame**: Game loop timing
- **Touch Events**: Mobile touch input (touchstart, touchmove, touchend, touchcancel)
- **Mouse Events**: Desktop input (mousemove, mousedown, mouseup, mouseleave)
- **Window Events**: Input (keydown, keyup), resize, orientationchange
- **Media Queries**: Detects `prefers-reduced-motion` OS accessibility setting

## Accessibility

- **Screen reader support**: Status announcements via `#srLive` element with `aria-live="polite"`
- **Reduced motion**: Checkbox for users who prefer minimal animations; respects OS preference
- **Keyboard & Touch**: Full game playable without mouse
- **Readable HUD**: High-contrast text with text shadows for visibility

## Performance Notes

- **Particle culling**: Particle count scales with quality level to maintain 60 FPS
- **Object pooling**: Arrays (bullets, enemies, particles) reused each frame; objects pushed/spliced efficiently
- **Canvas context state**: Uses `ctx.save()/restore()` to minimize state changes
- **Collision checks**: Only runs rectangle overlap, no spatial partitioning (acceptable for ~30 entities)
- **Audio context**: Lazily initialized on first input to comply with browser autoplay policies

## Common Debugging

- **Game not responding**: Check if `running` is true and not `paused`
- **No audio**: Ensure audio context initialized via `ensureAudio()` (requires user interaction first)
- **Performance drop**: Check `quality` variable; if > 0, particles are reduced
- **Touch controls not working**: Verify `isTouchDevice` detected correctly (line 778)
- **High score not saving**: Check browser allows localStorage (not in private/incognito)

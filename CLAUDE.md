# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file Raiden-style vertical shoot-'em-up game built with HTML5 Canvas and Web Audio synthesis. No external dependencies—the game runs entirely as a static HTML page.

- **File**: `index.html` (1670 lines)
- **Technology**: Canvas 2D, Web Audio API, LocalStorage, RequestAnimationFrame
- **Platforms**: Desktop (keyboard/mouse), mobile (touch controls with virtual joystick)

## Quick Start

1. Open `index.html` in a web browser
2. Use mouse or keyboard to play (see controls hint in-game)
3. No build step required—this is pure client-side

## Architecture

### High-Level Structure

The entire game is an IIFE (Immediately Invoked Function Expression) within a `<script>` tag. Key systems are organized sequentially but interact through shared state:

1. **Game State** (lines 863–869): `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`, etc.
2. **Constants** (lines 485–515): Difficulty configs, enemy types, boss designs, themes, boss attack patterns
3. **Audio System** (lines 542–676): WebAudio synth functions for all SFX
4. **Input System** (lines 767–861): Keyboard, mouse, touch/joystick handlers
5. **Rendering** (lines 913–1638): Canvas drawing functions for all entities
6. **Game Logic** (lines 875–1505): Update loops, collision detection, enemy AI, boss behavior
7. **Main Loop** (lines 1640–1665): RequestAnimationFrame loop that calls update and draw

### Key Architectural Decisions

**Single Global Scope**: Everything lives in one namespace to avoid complexity. Easy to trace, but variables are tightly coupled.

**Immediate-Mode Rendering**: Each frame, the canvas is cleared and everything is redrawn. Particles, enemies, and bullets are simple objects with position/velocity; no sprite or animation classes.

**Difficulty-Driven Spawning**: Enemy spawn rate, speed, and attack frequency scale per difficulty. No procedural generation—enemies always spawn from the top, enemies weave or move straight, bosses follow hardcoded patterns.

**LocalStorage Persistence**: Best score, mute state, difficulty choice, and reduced-motion preference are saved per browser.

**Adaptive Quality**: Frame time is tracked to detect performance issues. If avg frame time exceeds 1/40s, quality level increases (fewer particles, less glow). If fps stays above 56, quality decreases.

### Entity Systems

**Bullets** (Player): Vulcan (spread shot) or Laser (pierce, multi-lane). Level 1–3 increases spread/lanes. Vulcan has fixed speed, Laser has pierce mechanic.

**Enemies**: Three types spawn based on stage:
- **Grunt**: Red, slow, 1 HP, basic enemy
- **Interceptor**: Orange, fast, weaves side-to-side, 1 HP
- **Tank**: Purple, slow, 4 HP, appears stage 2+

Each enemy has a `pattern` (spread-fire or aimed at player), `shootTimer`, and optional `weave` movement.

**Enemy Bullets**: Fired from enemies, travel in straight lines (some radial from bosses). No targeting AI—aimed-fire enemies compute a direction to the player once and fire.

**Bosses**: Six designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN), each with unique attack patterns. Bosses enter from the top, telegraph attacks with a visual pulse, then fire in a specific pattern. Each boss has a phase timer and pattern index to cycle through attacks.

**Power-ups**: Life, bomb, weapon. Weapon power-ups increase level (1–3) or swap weapon type, resetting level to 1.

**Particles**: Simple objects with position, velocity, and lifetime. Used for explosions and visual feedback.

### Score & Progression

- **Kills per Stage**: 18 kills → boss spawns
- **Chain Multiplier**: 5 kills = 1x multiplier, caps at 10x (50 kills). Breaks if you miss for 2.2 seconds.
- **Stage Progression**: Each stage increments the theme (cyclic), increases enemy spawn rate, and spawns the next boss in the rotation.

### Sound Design

All sounds are synthesized using the Web Audio API—no audio files. The `sfx` object contains functions for each sound:
- **shoot/laser**: Beeps with pitch sweeps for player fire
- **explodeGrunt/Interceptor/Tank**: Different explosion sounds based on enemy type
- **hit/bomb/bossTelegraph/bossDown**: Hit feedback, bomb detonation, boss warnings
- **powerup/stageClear/gameover**: UI feedback sounds

Each sound combines oscillators (sine, square, sawtooth), filters (lowpass, highpass), and noise bursts. Pan (stereo) is applied where relevant (gunshots pan to their on-screen location).

## Common Tasks

### Add a New Enemy Type

1. Add entry to `ENEMY_TYPES` (line 502–506) with sprite dimensions, HP, speed multiplier, color, and minimum stage:
   ```javascript
   newType: { w: 30, h: 30, hp: 2, speedMul: 1.2, color: '#00ff00', dark: '#009900', score: 20, minStage: 3, weave: true }
   ```
2. Update `spawnEnemy()` (lines 1022–1041) to include the new type in the `choices` filter and spawn logic
3. Add explosion SFX in `EXPLODE_SFX` map (line 678) if unique sound is needed
4. If unique behavior (e.g., bullet pattern), add logic in the enemy update loop (lines 1210–1227)

### Add a New Boss

1. Add to `BOSSES` array (lines 508–515) with hull colors, wing colors, core color, and HP multiplier
2. Add attack pattern array to `BOSS_PATTERNS` (lines 1372–1409) with 3 firing functions (called in sequence)
3. Each pattern function receives the boss object and fires enemy bullets using helper functions like `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.
4. Boss design is selected by `(stage - 1) % BOSSES.length`, so new bosses cycle automatically as stage increases

### Add a New Theme / Visual Style

Themes are stored in `THEMES` (lines 493–500). Each theme has:
- `sky`: Array of 3 colors for the gradient (top, middle, bottom)
- `cloud`: RGB values for cloud color
- `sun`: RGB values for sun glow
- `name`: Display name shown when entering the stage

Themes cycle by `(stage - 1) % THEMES.length`. Add a new object to the array to add a theme.

### Adjust Difficulty

Difficulty settings are in `DIFFICULTIES` (lines 485–489):
- `spawnBase` / `spawnMin`: Enemy spawn rate (lower = faster)
- `enemySpeed`: Multiplier applied to enemy movement
- `enemyShootChance`: 0–1, chance enemy will fire when spawned
- `lives` / `bombs`: Starting resources
- `bossHpMul`: Boss health multiplier

Modify these values to make stages harder or easier.

### Add a New Sound Effect

1. Create a function in the `sfx` object (lines 624–676) that calls `beep()` or `noiseBurst()` with frequency, duration, type, volume, and options:
   ```javascript
   newSound: () => beep(440, 0.1, 'square', 0.05, { sweepTo: 220 })
   ```
2. Call `ensureAudio()` before any audio (already done in input handlers)
3. Reference the function where needed (e.g., `sfx.newSound()`)

**Key Audio Functions**:
- `beep(freq, dur, type, vol, opts)`: Oscillator-based tone (square/sine/sawtooth/triangle)
  - `opts.sweepTo`: End frequency (pitch slide)
  - `opts.filterType/filterFreq`: Filter for tone shaping
  - `opts.pan`: Stereo pan (-1 to 1)
  - `opts.delay`: Delay before sound starts
- `noiseBurst(dur, vol, opts)`: White noise with envelope and optional filter

### Modify Weapon Behavior

Weapons are handled in `fireWeapon()` (lines 1124–1146) and weapon power-up logic (lines 1311–1320):
- Vulcan: Spread shot, uses `player.level` to determine spread angles
- Laser: Multi-lane piercing shot, uses level for lane count and pierce count

To adjust weapon balance, modify spread angles, speeds, or pierce counts in these functions.

### Debug Performance

The `quality` variable (line 474) tracks performance:
- `0`: Full quality (glow, shadows, full particles)
- `1`: Medium (reduced particles, some glow)
- `2`: Low (minimal particles, no glow)

To check frame time and force a specific quality level:
- Frame times are stored in `frameTimes` array (line 475)
- Add `console.log(quality, frameTimes[frameTimes.length - 1])` in the loop to monitor
- Force a quality level by setting `quality = 1` directly in the update loop (temporary)

### Test a Specific Stage or Boss

Add this to the `resetGame()` function (line 875) after `running = true`:
```javascript
stage = 5; // Jump to stage 5
kills = 18; // Trigger boss spawn immediately after a few kills
```

Then reload the page and start a game.

## Canvas & Coordinate System

- **Canvas Size**: 480×720 (BASE_W, BASE_H at lines 753–754)
- **Scaled**: Canvas is drawn at up to 1.6x its native size (line 757) to fit the window
- **Origin**: Top-left (0, 0) is upper-left corner. Y increases downward. X increases rightward.
- **Entities**: Stored as `{ x, y, w, h, ... }`. `x` and `y` are the center of the entity, `w` and `h` are full width/height.

## Input Handling

**Keyboard**: Arrow keys or WASD to move, Space to fire, X/Shift/P to bomb, P/Esc to pause.

**Mouse**: Position over canvas to steer. Click to fire. Steering takes priority over keyboard when mouse is active (see lines 1161–1170).

**Touch**: Left 55% of screen is movement zone with virtual joystick. Right 45% has FIRE and BOMB buttons. A joystick base appears when touching the move zone.

See `canvasPointFromEvent()` (lines 840–845) for event-to-canvas-coordinate conversion.

## localStorage Keys

- `skyfighter.bestScore`: Best score ever
- `skyfighter.muted`: Mute state (1 or 0)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Reduced motion preference (1 or 0)

## Performance Considerations

- **Particles**: Culled at the end of each update (line 1297). Quality level reduces particle count.
- **Collision Checks**: Nested loops over enemies and bullets (lines 1232–1253). No quadtree or spatial indexing—acceptable for ~30 entities.
- **Enemy AI**: Shoot timer countdown; no pathfinding. Weaving is a sine wave.
- **Glow/Shadow**: Conditionally applied based on quality level (line 1579) to avoid expensive canvas operations on low-end devices.

## Notes for Future Development

- **Code Organization**: Single IIFE is easy to read but hard to test. Consider extracting game logic into modules if complexity grows.
- **Asset Pipeline**: All graphics and sounds are generated, so there's no media pipeline. Keep it that way for minimal dependencies.
- **Mobile Optimization**: Touch controls work but could benefit from haptic feedback (if browser supports it).
- **Accessibility**: Screen reader announcements are in place via `announce()` (line 468) and the `#srLive` element. Test with a screen reader for edge cases.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a browser-based Raiden-style vertical shoot-'em-up game. The entire game is implemented in a single HTML file (`index.html`) containing embedded CSS and JavaScript.

**Key features:**
- Player-controlled aircraft with weapon and bomb mechanics
- Procedurally spawned enemies with different behaviors (grunt, interceptor, tank)
- 6 boss encounters with distinct attack patterns
- 3 difficulty levels (easy, normal, hard) with adaptive spawning
- 6 dynamic sky themes that change per stage
- Score multiplier based on kill combos
- Touch controls with joystick for mobile
- Web Audio API synthesized sound effects (no external audio files)
- LocalStorage persistence for best score, muted state, difficulty, and reduced-motion preference

## Architecture

The entire codebase is in `index.html` (1,670 lines). The JavaScript is wrapped in an IIFE (Immediately Invoked Function Expression) at line 423.

### Code Organization (within the single file):

1. **DOM References & Constants** (lines 423–525)
   - Canvas and UI element references
   - Difficulty presets: `DIFFICULTIES` object (spawn rates, enemy speed, lives, bomb count)
   - Enemy type definitions: `ENEMY_TYPES` (stats for grunt, interceptor, tank)
   - Boss definitions: `BOSSES` array (6 boss designs with names and visual parameters)
   - Theme definitions: `THEMES` array (6 sky gradients with cloud and sun colors)
   - LocalStorage keys for persisting game state

2. **Audio System** (lines 542–676)
   - Web Audio API context initialization (`ensureAudio()`)
   - Frequency-based synth beep generation (`beep()`)
   - Noise-based burst generation (`noiseBurst()`)
   - Sound effect library object `sfx` with 13+ effects (shoot, laser, explosions, hit, bomb, powerup, boss patterns, UI, gameover)

3. **Rendering & Animation Helpers** (lines 680–730)
   - `renderHearts()` / `renderBombs()`: Update HUD displays
   - `flashHit()` / `flashBomb()`: Trigger screen flash animations
   - `showComboPop()` / `showStageBanner()`: Show overlay text
   - `screenShake()`: Shake with magnitude and duration
   - Canvas drawing functions: `drawPlane()` (player & enemy), `drawBossShip()`, `drawClouds()`

4. **Input Handling** (lines 767–861)
   - Keyboard: Arrow keys / WASD for movement, Space to fire, X/Shift for bomb, P/Esc to pause
   - Mouse: Move to steer, click to fire; optional firing priority over keyboard when mouse is active
   - Touch: Joystick on left side for movement, FIRE and BOMB buttons on right
   - Joystick logic with constrained vector

5. **Game State & Core Loop** (lines 863–1302)
   - Main game state variables: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
   - `resetGame()`: Initialize/reset all state per difficulty
   - `update(dt)`: Tick game logic (movement, shooting, collisions, spawning)
   - `draw()`: Render game state to canvas
   - `loop()`: RequestAnimationFrame handler with delta-time tracking and performance monitoring

6. **Entity Updates & Collisions** (lines 1148–1439)
   - Enemy spawning with difficulty scaling and weave patterns
   - Boss spawning and behavior patterns
   - Weapon firing: separate logic for vulcan (spread) and laser (pierce)
   - Collision detection using `rectsOverlap()` (AABB check)
   - Power-up logic (life, bomb, weapon upgrades)
   - Boss attack patterns: 6 pattern arrays with 3 attacks each (radial, aimed spread, dense walls, etc.)

7. **Game Flow & UI** (lines 1456–1504)
   - `endGame()`: Persist best score, show results overlay
   - `loseLife()` / `killBoss()`: Stage progression
   - `togglePause()`: Pause overlay
   - Difficulty selection and UI button handlers

### CSS Organization

- **Global styles** (lines 10–18): Dark theme, touch handling, user-select disabled
- **Canvas & container** (lines 20–33): Centered, responsive scaling
- **HUD elements** (lines 34–74): Score, hearts, bombs, weapon, multiplier displays
- **Overlays** (lines 75–148): Vignette (critical health), flash effects (hit/bomb), combo popup, stage banner
- **UI buttons** (lines 181–200): Mute, motion toggle, pause buttons
- **Main menu overlay** (lines 200–290): Game over screen, difficulty picker, start button, animations
- **Touch controls** (lines 291–341): Joystick base/stick, FIRE/BOMB buttons (hidden on desktop)
- **Animations** (@keyframes): heartBeat, vignettePulse, flashHit, flashBomb, comboPulse, stagePulse, cardIn, iconBob
- **Reduced motion support** (lines 349–358): Disables animations when user prefers reduced motion

## Development & Running

### Browser Testing
Simply open `index.html` in a browser. No build step or dependencies.

### Responsive Behavior
- Canvas scaling: Base resolution 480×720, responsive up to 1.6× scale
- Mobile touch controls auto-enable on touch devices
- Safe area insets respected for notched devices

### Performance Adaptation
- `trackPerf()` (line 476): Monitors frame times and adjusts rendering quality (0=full, 1=medium, 2=low)
- Quality level reduces particle counts and glow effects when FPS dips
- See `quality` variable usage in explosion spawning and draw glow effects

### Game State Persistence
- Best score: `localStorage.skyfighter.bestScore`
- Muted: `localStorage.skyfighter.muted` (0/1)
- Difficulty: `localStorage.skyfighter.difficulty` (easy/normal/hard)
- Reduced motion: `localStorage.skyfighter.reducedMotion` (0/1, respects system preference)

## Game Mechanics

### Difficulty Settings
Each difficulty scales three core variables:
- **Spawn rate**: How fast enemies appear (base interval in frames)
- **Enemy speed**: Multiplier on enemy movement velocity
- **Shoot chance**: Likelihood enemy fires when eligible
- **Boss HP**: Damage multiplier on boss health

**Easy** (forgiving): Slower spawns, slower enemies, fewer shots, reduced boss HP
**Normal** (balanced): Moderate everything
**Hard** (challenging): Fast spawns, fast enemies, frequent shots, increased boss HP

### Scoring & Combo System
- Each enemy killed adds to a "chain" counter
- Multiplier increases with chain: `multiplier = 1 + floor(chain / 5)` (capped at 10×)
- Chain resets after 2.2 seconds of no kills (see `chainTimer`)
- Boss worth 500 fixed points (scales with multiplier)

### Weapons
- **Vulcan** (default): Straight shots, no pierce. Levels: 1 shot center → 2 shots wide → 3 shots (wide spread)
- **Laser**: Piercing shots through up to 3 enemies. Levels: 1 center → 2 sides → 2 sides + center
- Switch weapons via weapon power-ups; at max level (3), next power-up switches to other weapon

### Boss Behavior
- Boss enters from top, triggers on 18 kills per stage (`KILLS_PER_STAGE`)
- Boss has "entering" phase (simple downward movement), then combat
- Combat cycle: 1.3s attack delay → 0.37s telegraph → attack pattern fires
- Each boss has 3 patterns (radial burst, aimed spread, dense walls) cycled sequentially
- HP scales per stage and boss design (1.0× to 1.7× multiplier)

### Visual Themes
6 themes rotate per stage, each with unique sky gradient (3 color stops), cloud color, and sun glow. Theme name shown in stage clear announcement.

## Common Tasks

### Add/Modify Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502) with: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Update enemy spawn logic (line 1026) to filter by minStage
3. Add explosion sound mapping to `EXPLODE_SFX` (line 678) if needed

### Add/Modify Boss
1. Add entry to `BOSSES` array (line 508) with: `name`, `hullTop`, `hullBottom`, `wing`, `core`, `hpMul`
2. Add attack pattern array to `BOSS_PATTERNS` (line 1372) with 3 pattern functions
3. Pattern functions use helpers: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`

### Add Sound Effect
1. Add a function to the `sfx` object (line 624) using `beep()` and/or `noiseBurst()` helpers
2. Call via `sfx.newEffect()` wherever needed
3. Respects mute state automatically (checked inside `beep()` and `noiseBurst()`)

### Add UI Element
1. Add HTML in the container div (line 362)
2. Store reference at top (line 426–461)
3. Add CSS styling (lines 10–358)
4. Wire up event listeners (see pattern in lines 735–750)

## Design Notes

### Canvas Drawing
- Uses `ctx.save()` / `ctx.restore()` to isolate transform and style changes
- Enemies and player drawn with `drawPlane()`: separates main fuselage (filled), dark wings (filled), light cockpit (ellipse)
- Boss has gradient hull, animated cores, moving fins
- All shapes use canvas primitives (arc, ellipse, path fill); no image assets

### Performance Optimization
- Particles, explosions, and glow effects scale with detected FPS
- Collision uses simple AABB (rectangle overlap), not pixel-perfect
- Enemy bullets use radial properties (dx/dy velocity) instead of vx/vy for boss attacks
- Removed bullets/enemies/particles/powerUps filtered by boundary checks each frame

### Input Priority
Mouse aiming only applies when no keyboard input detected; prevents fighting between input methods. Firing and bomb always work regardless of input source.

### Accessibility
- Screen reader announcements via `#srLive` with `aria-live="polite"`
- Mute button toggles audio globally
- Reduced motion mode disables shakes, flashes, animations
- Respects system `prefers-reduced-motion` media query on first load

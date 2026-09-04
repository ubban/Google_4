# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in HTML5 Canvas with procedurally generated WebAudio synth. It's a single-file game (index.html) with no external dependencies, build tools, or framework requirements.

## Running the Game

Open `index.html` directly in a web browser. No server or build step is required. The game runs client-side with full feature support in modern browsers that support:
- HTML5 Canvas 2D
- Web Audio API
- LocalStorage
- ES6+ JavaScript

## Game Architecture

### High-Level Design

The entire game runs in a single IIFE (immediately invoked function expression) with these major systems:

**Game Loop**: `requestAnimationFrame` drives a tight loop that calls `update(dt)` for game logic, then `draw()` for rendering. The delta time is capped at 50ms to prevent large jumps.

**Performance Adaptation**: The `quality` variable (0=full, 1=medium, 2=low) tracks frame times and auto-scales effects based on performance. Visual effects like shadows, glows, and particle counts are reduced on lower quality settings.

**State Management**: Game state (player, enemies, bullets, boss, etc.) is stored in module-level variables within the IIFE. State transitions occur through discrete events like `resetGame()`, `endGame()`, and `killBoss()`.

### Key Systems

1. **Player Control & Movement**
   - Keyboard input (WASD/Arrows), mouse steering, and touch joystick
   - Diagonal movement normalized; mouse provides smooth gliding when active
   - Two weapons: Vulcan (spread shots) and Laser (piercing shots with splash)
   - Weapon levels 1–3, with level 3 unlocking weapon switching
   - Invulnerability after taking damage (1.5 seconds)

2. **Enemies**
   - Three types: grunt (basic), interceptor (fast, weaves), tank (slow, tanky)
   - Spawn difficulty ramps over time; tank type appears at stage 2
   - Two fire patterns: spread (bullets at fixed angles) and aimed (toward player)
   - Destroyed on collision with player bullets; drop random power-ups
   - Enemy bullets disappear on bomb or when player uses bomb

3. **Bosses**
   - One boss per stage (6 unique designs with distinct color schemes)
   - Six distinct attack patterns per boss (radial bursts, aimed spreads, telegraphed telegraphs, etc.)
   - HP scales with difficulty and boss design multiplier
   - Boss telegraphs incoming attack with a 22-frame warning
   - Visual telegraph via core color shift and SFX telegraph sound
   - Defeated boss triggers stage clear sound and advances to next stage

4. **Scoring & Multiplier**
   - Base score per enemy type (grunt=10, interceptor=15, tank=35, boss=500)
   - Kill-chain multiplier: increases by 1 for every 5 consecutive kills
   - Chain timer (2.2s) resets on any player damage
   - UI updates in real time; best score persists to localStorage

5. **Input Handling**
   - Keyboard: Arrow/WASD for movement, Space for fire, X/Shift for bomb, P/Esc for pause
   - Mouse: Move to steer (glides toward pointer), click to fire
   - Touch: Left zone = joystick movement (shows visual joystick base), right zone = Fire/Bomb buttons
   - Firing state managed via `firing` flag (true while Space/click/touch held)

6. **Audio**
   - WebAudio synth with no external files (all sounds generated procedurally)
   - `beep()`: oscillator with frequency sweep, optional filter, pan, and delay
   - `noiseBurst()`: noise source with filter (used for explosions)
   - Each SFX in the `sfx` object composes multiple beeps/bursts (e.g., `sfx.explodeTank()` chains 3 beeps + 2 noise bursts)
   - Master gain (0.9) with compressor for dynamic range control
   - Audio context lazy-initialized on first user input

7. **Visuals & Effects**
   - Screen shake: `screenShake(magnitude, duration)` applies random canvas translation
   - Flash effects: White fade on hit, yellow fade on bomb
   - Vignette pulse: Red radial gradient when health critical (1 heart remaining)
   - Combo popup: Animated text at screen center when hits % 10 === 0
   - Stage banner: Title card at stage entry/boss entry
   - Boss bar: Health track shows at top when boss is present
   - Particle effects: Spawned on explosions with gravity-free drift and alpha fade

8. **Data Persistence**
   - `localStorage` keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
   - Best score updates only on new high score
   - Difficulty and mute state persist across sessions
   - Reduced motion preference respects OS setting but can be overridden

9. **Accessibility**
   - Reduced motion mode: Disables animations (vignette pulse, heart beat, combo/stage popups, hit/bomb flash)
   - Screen reader support: `#srLive` (role="status", aria-live="polite") announces key events
   - `announce()` helper pushes text to screen reader queue
   - Keyboard-only gameplay fully supported

## Code Organization (Top to Bottom)

1. **DOM Setup** (lines 426–461): Query and cache all interactive elements
2. **Constants** (lines 463–515): Difficulties, enemy types, boss designs, themes, sound effects
3. **State** (lines 517–527): Initialization of bestScore, muted, difficulty, reducedMotion
4. **Audio** (lines 542–676): `ensureAudio()`, `beep()`, `noiseBurst()`, `sfx` object
5. **Rendering Helpers** (lines 680–729): HUD updates, particle effects, combo/stage banners
6. **Input Handling** (lines 768–861): Keyboard, touch joystick, mouse steering, firing state
7. **Game State** (lines 864–911): `resetGame()`, state initialization, HUD rendering
8. **Drawing Functions** (lines 913–1019): `drawPlane()`, `drawBossShip()`, `drawClouds()`, `draw()`
9. **Spawning** (lines 1022–1066): `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`
10. **Update Logic** (lines 1148–1302): `update(dt)` master function
11. **Boss Patterns** (lines 1324–1409): `fireRadial()`, `fireAimedSpread()`, etc., and `BOSS_PATTERNS` array
12. **Boss Update** (lines 1411–1439): `updateBoss(dt)` handles boss movement and attack telegraph
13. **Game Flow** (lines 1441–1505): `killBoss()`, `loseLife()`, `endGame()`, `togglePause()`, `applyPowerUp()`
14. **Main Loop** (lines 1640–1665): `loop()` and startup via `startBtn` click

## Key Implementation Details

**Collision Detection**: Axis-aligned bounding box check via `rectsOverlap()` (treats all objects as rectangles by half-width/height).

**Enemy Waves**: Spawn timer decays based on difficulty. Spawn base decreases over time as frame count increases (difficulty ramp-up). At 18 kills per stage, boss is queued after 1.2s delay.

**Boss Attack Telegraph**: Boss core flashes white and plays telegraph sound 22 frames (~367ms) before firing. This gives player reaction time.

**Adaptive Quality**: Frame times are tracked; if average FPS drops below ~40, quality is reduced. Glow effects (shadow blur), particle counts, and drawClouds calls scale with quality level.

**Power-up Drop Rates**: Random on enemy death: 6% life, 8% bomb, 20% weapon (totals 34% drop rate per enemy).

**Weapon Switching**: Laser mode requires level 3 Vulcan. Switching resets weapon level to 1, forcing player to choose between damage output (leveled Vulcan) or piercing (Laser).

## Common Tasks

**Adding a new enemy type**: Add entry to `ENEMY_TYPES` object with w, h, hp, speedMul, color, dark, score, minStage, and optional weave flag. Update enemy spawning logic in `spawnEnemy()` if new conditions needed.

**Adding a new boss**: Add entry to `BOSSES` array (name, hull colors, core color, hpMul). Add corresponding attack pattern array to `BOSS_PATTERNS` (will auto-index).

**Adding a new theme**: Add entry to `THEMES` array with sky gradient colors, cloud color, sun color, and name. Themes cycle by stage; update stage transition to use new theme.

**Tweaking difficulty**: Edit `DIFFICULTIES` object. Parameters: spawnBase (initial spawn timer), spawnMin (min spawn rate), enemySpeed (enemy speed multiplier), enemyShootChance (fire rate), lives, bombs, bossHpMul.

**New sound effect**: Add function to `sfx` object that composes `beep()` and/or `noiseBurst()` calls. Pass frequency, duration, volume, and opts (sweepTo, filterType, filterFreq, pan, delay).

## Testing in Browser

1. Open `index.html` in Firefox, Chrome, Safari, or Edge
2. Click "Start Game" to begin
3. Use Mouse (move + click), Keyboard (WASD + Space/X), or Touch (joystick + Fire/Bomb buttons)
4. Pause with P or click pause button
5. Mute sound or disable motion effects via top-right buttons
6. Switch difficulty before starting (affects enemy spawn rates, speed, HP, lives, bombs)

## Notes for Future Work

- The single-file design keeps the game portable and easy to share; modularization is not necessary unless the codebase grows significantly.
- WebAudio synth allows zero external asset dependencies; all audio is generated in real-time.
- Canvas 2D rendering is performant on mobile; no WebGL migration needed unless 3D is desired.
- Reduced motion mode is well-implemented and respects OS preference; ensure new effects check this flag.
- Collision detection uses axis-aligned bounding boxes; circular/pixelated collisions would require per-pixel testing (overkill for this game).

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in a single HTML file (1669 lines). It runs in any modern browser with no build step required—simply open `index.html` directly.

## Quick Start

1. **Play the game:** Open `index.html` in a web browser
2. **Develop:** Edit `index.html` and refresh the browser to see changes
3. **Deploy:** The single HTML file is the complete deliverable

## Code Structure

The entire game is in `index.html`, organized as follows within a single IIFE (Immediately Invoked Function Expression):

### HTML/CSS (lines 1–361)
- **Canvas container** (#container, #gameCanvas): The main 720×480 game viewport
- **HUD** (#hud): Score, best score, weapon level, lives, bombs
- **Overlay** (#overlay): Title screen, game over screen, difficulty selection
- **Boss bar** (#bossBarWrap): Boss health display when fighting bosses
- **Touch controls** (#touchControls): Mobile joystick and buttons (auto-hidden on desktop)
- **Visual effects**: Vignette (damage indicator), flash overlay, combo pop text, stage banner
- **Accessibility**: Screen reader live region (#srLive), reduced motion support

### Game Configuration (lines 463–515)
- **DIFFICULTIES**: Three difficulty levels (easy/normal/hard) with spawn rates, enemy speeds, lives, bombs
- **ENEMY_TYPES**: Three enemy classes (grunt, interceptor, tank) with HP, speed, score values
- **THEMES**: Six color themes for sky gradient and clouds
- **BOSSES**: Six boss variants with unique colors and HP multipliers
- **KILLS_PER_STAGE**: Enemy kills needed to trigger boss fight

### Audio System (lines 543–695)
- **WebAudio synthesis**: No external files—all sounds generated at runtime
- **beep()**: Oscillator-based tones with frequency sweep, filtering, panning
- **noiseBurst()**: Noise-based effects using offline audio context
- **sfx object**: Pre-defined sound parameters (pitch, duration) for 10+ effects (shoot, hit, bomb, explosion variants)
- **Mute state**: Persisted in localStorage; button in top-right corner

### Game State (lines 753–868)
- **Canvas dimensions**: 480×720 (BASE_W × BASE_H) with responsive scaling
- **Game objects**: player, bullets, enemies, enemyBullets, particles, powerUps, boss
- **Counters**: score, lives, stage, kills, chain, multiplier, bomb count
- **Input**: keys (keyboard), mouseTarget (mouse movement), joystick vectors (touch)
- **Performance tracking**: Adaptive quality level (0=full, 1=medium, 2=low) based on frame time

### Input Handling (lines 768–862)
- **Keyboard**: Arrow keys / WASD to move, Space / X / Shift for bombs, P / Esc to pause
- **Mouse**: Move to steer, click to fire
- **Touch**: 
  - Left 55% of screen: move zone (drag to fly)
  - Right 45%: action zone with FIRE and BOMB buttons
  - Virtual joystick base shown on first touch in move zone

### Game Logic (lines 871–1640)

#### Player & Movement
- **drawPlane()**: Renders player aircraft with rotation and flame animation
- **Weapon system**: Five weapon types (VULCAN, MACHINE, LASER, WAVE, SPIRAL) with levels 1–3
- **fireWeapon()**: Dispatch to weapon-specific fire patterns

#### Enemies
- **spawnEnemy()**: Selects enemy type based on stage and difficulty; spawns at random x position
- **Enemy AI**: Move downward, shoot at random intervals (difficulty-scaled), apply knockback from hits

#### Bosses
- **spawnBoss()**: Triggers after KILLS_PER_STAGE enemy kills
- **drawBossShip()**: Complex multi-part ship rendering with hull, wings, core
- **updateBoss()**: Boss cycles through attack patterns (fire radial, aimed spread, spiral, columns, etc.)
- **BOSS_PATTERNS**: Array of 6 patterns, each with timing and fire function reference

#### Collision & Damage
- **rectsOverlap()**: AABB overlap test for hit detection
- **Weapon collision**: Player bullets hit enemies; triggers knockback and damage
- **Enemy collision**: Player aircraft hit by enemy bullets or enemy bodies
- **Power-ups**: Spawn randomly from destroyed enemies; types: weapon upgrade, health, bomb, multiplier boost

#### Score & Multiplier
- **addScore()**: Accumulates score; increases chain/multiplier on consecutive kills
- **breakChain()**: Resets multiplier when a frame passes without kills (incentivizes rapid firing)
- **Multiplier visual**: Displayed in HUD; affects score per kill

#### Rendering
- **draw()**: Main render function; draws theme background, clouds, player, enemies, bullets, particles, HUD, boss
- **drawClouds()**: Parallax scrolling background; cloud shapes procedurally rendered
- **Visual feedback**: 
  - Damage vignette (red radial gradient pulse)
  - Flash overlay (white flash on hit/bomb)
  - Screen shake (camera jitter)
  - Particle explosions (on kill)

### Main Loop (line 1640–1669)
- **loop(now)**: RequestAnimationFrame-driven loop
- Computes delta time, calls update(dt), calls draw()
- Handles pause, game over state
- Tracks frame time for adaptive quality

### State Persistence (localStorage)
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Mute toggle
- `skyfighter.difficulty`: Selected difficulty (auto-selects next game)
- `skyfighter.reducedMotion`: Reduced motion preference

## Making Changes

### Adding a New Weapon Type
1. Add a new entry to the weapon data in `fireWeapon()` (around line 1124)
2. Define the firing pattern function (e.g., `fireRadial`, `fireAimedSpread`)
3. Update the player's weapon cycle to include it (modify weapon type array)
4. Test weapon progression through levels (lvl 1→2→3 should increase count/speed/spread)

### Adding an Enemy Type
1. Add entry to `ENEMY_TYPES` object with w, h, hp, speedMul, color, score, minStage
2. Update `spawnEnemy()` to include the new type in its selection logic
3. Tune difficulty scaling (enemy types can appear only at minStage+)

### Adding a Boss Pattern
1. Define a new firing function (e.g., `fireHomingSeeded`) following the same signature as existing patterns
2. Add an object to `BOSS_PATTERNS` with `name`, `fireFunc`, `interval`, `duration`, `count`
3. Optionally add a new BOSSES entry with unique colors

### Tweaking Game Feel
- **Difficulty**: Adjust spawn rates, enemy speed, shoot chance in `DIFFICULTIES`
- **Screen shake**: Modify `screenShake()` calls (mag, dur parameters)
- **Audio**: Adjust sfx parameters (pitch, duration) or add new beep/noiseBurst calls
- **Visuals**: Modify CSS animations (heartBeat, vignettePulse, comboPulse) or canvas colors

## Performance Considerations

The game adapts rendering quality based on frame time:
- **quality 0** (60 FPS target): Full effects, all particles, all enemies
- **quality 1** (40 FPS): Some particles skip, reduced visual effects
- **quality 2** (low-end devices): Minimal particles, simplified rendering

Check the `trackPerf()` function and its callers in `update()` to see where quality is used.

## Testing in Browser

1. Open the browser DevTools Console to check for errors
2. Verify all three difficulty modes work and have different enemy spawn rates
3. Test touch controls on mobile or use Chrome DevTools device emulation
4. Check reduced-motion mode (click the 🎆 button) to ensure animations respect user preference
5. Verify localStorage is working: play, set high score, reload—high score should persist
6. Test all weapon types (each enemy kill or crate pickup advances weapon)
7. Check boss encounters trigger after 18 enemy kills and display correctly

## Known Limitations

- Single-file HTML: No modular code splitting (good for a simple game, limits scalability)
- Canvas 2D: No 3D effects; performance depends on device GPU/CPU
- Audio synthesis: All sounds generated at runtime; no pre-recorded audio (keeps file size small but limits sound complexity)
- No unit tests: Game logic tightly coupled to rendering and input (typical for simple games)

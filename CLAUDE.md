# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built in vanilla HTML5/Canvas with no external dependencies. The entire game is contained in a single `index.html` file (~1670 lines).

### Key Technologies
- **Canvas API**: 2D rendering for all graphics (ships, bullets, effects, clouds, text)
- **Web Audio API**: Procedural sound synthesis (no audio files)
- **localStorage**: Persistent high scores, settings, difficulty, reduced-motion preference
- **Responsive design**: Scales to fit any screen; mobile touch controls included

## Game Architecture

The game follows a classic game loop pattern with clear separation of concerns:

### Core Systems

1. **Rendering (Canvas)**
   - `draw()`: Main render loop that draws sky gradient, clouds, player, enemies, bullets, particles
   - Theme system: 6 different sky/cloud/sun color schemes that cycle per stage
   - Glow effects adapt based on performance (quality setting: 0=full, 1=medium, 2=low)
   - Screen shake applied via canvas translation when triggered by hits/explosions

2. **Game Loop & Timing**
   - `loop(now)`: Called via `requestAnimationFrame`, maintains frame counter and deltaTime
   - Adaptive performance tracking: samples 45-frame window, adjusts quality setting if avg frame time drifts outside 40-56 FPS
   - Time-based updates (`dt` in seconds) ensure physics independence from framerate

3. **Input Handling**
   - **Keyboard**: Arrow keys, WASD for movement; Space to fire; X/Shift for bomb; P/Esc to pause
   - **Mouse**: Click to fire, move to steer (glides player toward pointer)
   - **Touch**: Joystick in lower-left (drag), Fire/Bomb buttons in lower-right
   - Input state stored in `keys` object for axis-aligned movement; keyboard/joystick take priority over mouse steering

4. **Physics & Collision**
   - `update(dt)`: Advances all entities (player, bullets, enemies, particles)
   - Axis-aligned bounding-box collision detection via `rectsOverlap()`
   - All speeds are in pixels/second (framerate-independent)
   - Player confined to canvas bounds

5. **Audio (Web Audio Synthesis)**
   - `ensureAudio()`: Initializes AudioContext on first user interaction (browser requirement)
   - Procedural sounds stored in `sfx` object (shoot, laser, explode variants, bomb, powerup, boss sounds, etc.)
   - `beep()` and `noiseBurst()` generate raw waveforms with envelope and filtering
   - Spatial audio: pan effects scale with X position for left/right separation
   - Master gain + compressor for consistent volume

6. **Difficulty & Game Settings**
   - `DIFFICULTIES` object: defines spawnRate, enemySpeed, shootChance, lives, bombs, bossHp multiplier per difficulty
   - Stored in localStorage and loaded on startup
   - UI buttons allow switching difficulty before game starts
   - Reduced-motion toggle respects `prefers-reduced-motion` media query; disables animations/screen-shake/flash when enabled

### Entity Types

- **Player**: Single ship with weapon cooldown, rapid-fire state, invunerability frames, level-based upgrades
- **Enemies**: Three types (grunt, interceptor, tank) with different HP, speed, score value, and min spawn stage
  - Grunt/Interceptor: simple 2D sprite
  - Tank: has HP bar above it
  - Weaving behavior for interceptors (sine wave pattern)
- **Bullets**: Player bullets travel upward; enemy bullets fall downward
  - Laser weapon: pierces enemies (damage multiple)
  - Vulcan: spread pattern at higher levels
- **Boss**: Large ship with phases, telegraph warning, bullet patterns
  - 6 unique boss designs that rotate every stage
  - Each boss has 3 unique attack patterns (radial bursts, aimed spreads, columns, spirals, etc.)
  - Telegraph phase gives 22 frames warning before attack
- **Power-ups**: Life, bomb, weapon (spawn from killed enemies)
- **Particles**: Explosion effects (brief lifetime, gravity)

### Game Flow

1. **Initialization**
   - `resetGame()` sets up fresh state (player position, empty arrays for bullets/enemies, difficulty params)
   - Overlay shown with difficulty selector and start button
2. **Stage Loop**
   - Spawn enemies until `KILLS_PER_STAGE` (18) reached
   - Boss spawns after brief delay once stage goal hit
   - Score multiplier chain resets if no kill for 2.2 seconds
3. **Boss Phase**
   - Boss enters from top (1.2s delay)
   - While boss present, no normal enemy spawning
   - Boss moves side-to-side, fires patterns, takes bullet damage (3 per bullet, +2.4 from bombs)
4. **Stage Clear**
   - Boss defeated → stage counter increments, theme changes, banner shown
5. **Game Over**
   - Lives reach 0 → final score/stage displayed, high score checked and saved

## Code Locations & Common Edits

| What | Where |
|------|-------|
| Game constants (stage kills, themes, enemy stats) | Lines 491–515 |
| Input handling | Lines 768–861 |
| Player ship rendering | `drawPlane()` at line 913 |
| Boss ship rendering | `drawBossShip()` at line 970 |
| Enemy spawning logic & balancing | `spawnEnemy()` at line 1022 |
| Weapon fire patterns | `fireWeapon()` at line 1124 |
| Collision detection & scoring | Lines 1232–1294 |
| Boss attack patterns | `BOSS_PATTERNS` at line 1372 |
| Canvas rendering | `draw()` at line 1520 |

## Extending the Game

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` with dimensions, HP, speed multiplier, color, score (line 502)
2. Update `spawnEnemy()` filters to include new type based on `minStage` (line 1026)
3. (Optional) Add explosion SFX to `EXPLODE_SFX` mapping (line 678)

### Adding a Boss Attack Pattern
1. Define new fire function (e.g., `firePatternName(b)`) that calls `enemyBullets.push()`
2. Add to existing boss's pattern array in `BOSS_PATTERNS` (line 1372)
3. Boss will cycle through all patterns in its array on repeat

### Tweaking Difficulty
- Modify `DIFFICULTIES` object entries (line 485): `spawnBase` affects enemy density; `enemySpeed`, `enemyShootChance`, `bossHpMul` scale difficulty
- Ramp applied in `spawnEnemy()` every 300 frames: `difficultyRamp = Math.floor(frame / 300) * 4`

### Changing Themes
- Add theme object to `THEMES` array (line 493): sky gradient colors, cloud color, sun color, display name
- Themes automatically cycle per stage

## Performance & Quality Settings

- Adaptive quality system monitors frame times and automatically degrades particle count, glow effects, and shadow blur
- At quality=0 (full): full particles, shadow glow on bullets/powerups, shadow blur on boss core
- At quality=2 (low): 35% particles, no glow
- Mute and reduced-motion toggles persist via localStorage

## Running the Game

Simply open `index.html` in a modern browser. No build step, no server, no dependencies.

- **Desktop**: Mouse to steer, click to fire, Space to fire continuously, X/Shift for bomb, P or Esc to pause
- **Mobile**: Drag left zone to move, tap FIRE, tap BOMB

## Notes for Future Development

- **Single file design**: Keeping everything in one file keeps the game dependency-free and instantly playable. If it grows beyond ~2000 lines, consider splitting into modules.
- **Canvas coordinates**: Origin (0, 0) is top-left. Player moves downward (positive Y) when pressing down arrow.
- **Audio context**: Web Audio must be initialized in response to user gesture. `ensureAudio()` handles this.
- **Collision**: Uses simple AABB (axis-aligned bounding box) with center-to-center distance checks. Sufficient for this game's scale.
- **Animation timings**: Screen shake, combo popups, stage banners use CSS animation where possible; game-loop-driven timers for boss telegraphs and particle life.
- **Accessibility**: Reduced-motion toggle hides shake/flash effects; keyboard controls provided alongside mouse/touch; screen-reader live region announces game events.

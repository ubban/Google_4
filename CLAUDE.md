# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in vanilla JavaScript with HTML5 Canvas and WebAudio API. It's a single-file application (`index.html`) with no external dependencies — all gameplay, graphics, audio synthesis, and UI are self-contained.

## Quick Start

- **Play the game**: Open `index.html` in a browser
- **No build step**: This is a static file; no compilation or bundling needed
- **No dependencies**: Pure HTML5/Canvas/WebAudio — nothing to install

## Architecture Overview

### Single-File Structure
The entire game logic lives in one `<script>` block within `index.html`. The codebase uses a functional, procedural style with a game loop (`requestAnimationFrame`).

### Core Systems

**Game Loop & State Management**
- `loop(now)`: Main game loop using `requestAnimationFrame`
- `update(dt)`: Update all game objects each frame (player, bullets, enemies, particles, etc.)
- `draw()`: Render game state to canvas
- Global state: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`, etc.

**Input Handling**
- Keyboard: Arrow keys, WASD, Space (fire), X/Shift (bomb), P/Esc (pause)
- Mouse: Move to steer, click to fire
- Touch: Left side for movement (joystick), right side buttons (Fire, Bomb)
- Input is read every frame; no event-based state storage (prevents input lag)

**Entity System**
- Entities are plain objects with position (`x`, `y`), dimensions (`w`, `h`), and type-specific properties
- **Player**: Position, speed, weapon (vulcan/laser), level, cooldown, invulnerability timer
- **Enemies**: Type (grunt/interceptor/tank), HP, color, movement pattern, shooting behavior
- **Bullets**: Player bullets have pierce/laser flags; enemy bullets can be radial (boss attacks)
- **Particles**: Explosions; simple velocity + lifetime physics
- **Power-ups**: Type (life/bomb/weapon); fall and are collected on collision

**Collision Detection**
- `rectsOverlap(a, b)`: Simple AABB collision check used for all entity interactions

**Rendering & Visual FX**
- Canvas context (`ctx`) is used for drawing planes, particles, text, UI
- **Procedural graphics**: All enemies and the player are drawn as vectors (no sprites)
- **Boss ships**: Each boss type has unique colors and design variables (`BOSSES` array)
- **Screen effects**: 
  - Screen shake on impact/bomb
  - Flash overlay on hit/bomb
  - Vignette pulsing when health is critical
  - Combo pop-ups and stage banners with animations
  - Adaptive particle count based on performance quality

**Audio System**
- WebAudio API synthesis (no external audio files)
- `beep()`: Generate tones with optional frequency sweep and filtering
- `noiseBurst()`: Generate noise for explosion effects
- Audio is panned based on source position for spatial awareness
- Master gain and dynamics compressor for polish
- Mute toggle persists to localStorage

**Performance Adaptation**
- `trackPerf(dt)`: Monitors frame times and adjusts `quality` level (0=full, 1=medium, 2=low)
- Lower quality reduces particle counts, disables glow effects, and reduces shadow blur
- Ensures 60 FPS on slower devices

**Difficulty & Progression**
- Three difficulty levels: Easy, Normal, Hard (differ in spawn rate, enemy speed, enemy accuracy, lives, bombs, boss HP multiplier)
- Stages unlock after defeating 18 enemies per stage
- Bosses spawn at stage transition; 6 unique boss designs cycle through stages
- Each boss has unique attack patterns (8 patterns per boss × 6 bosses = 48 unique attacks)

**Weapon & Multiplier System**
- **Vulcan**: Spread shot (1–3 projectiles), fast fire rate
- **Laser**: Piercing beam (1–3 lanes), slower fire rate, can pass through enemies up to pierce limit
- **Power-ups cycle**: Level 1→2→3 within weapon, then switch weapons and reset to level 1
- **Combo multiplier**: Each kill extends a timer; multiplier increases with kill count (1 + kills/5, capped at 10x)
- Breaking the combo chain resets multiplier to 1x

**Theming System**
- Six theme variants (Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge)
- Themes cycle per stage and define sky gradient, cloud colors, and sun glow
- Purely cosmetic; no gameplay impact

**Accessibility**
- Reduced motion preference: respects `prefers-reduced-motion` and allows toggle
- Animations are skipped or reduced when reduced motion is active
- Screen reader support: HUD updates announced via aria-live region
- High-contrast visuals with clear indicators for critical state

### Storage (localStorage)
- `skyfighter.bestScore`: Best score
- `skyfighter.muted`: Audio mute state (0 or 1)
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: User's motion preference override

### Constants & Balancing

**DIFFICULTIES**: Spawn timing, enemy speed/accuracy, lives, bombs, boss HP multiplier per difficulty
**KILLS_PER_STAGE**: 18 kills required to reach boss
**ENEMY_TYPES**: Grunt, Interceptor, Tank — each with HP, speed, score, min stage to appear
**BOSSES**: 6 boss designs, each with unique colors and HP multiplier
**THEMES**: 6 visual themes for stages
**BOSS_PATTERNS**: Attack patterns indexed by boss type (3 patterns per boss)

### Key Functions to Know

- `resetGame()`: Initialize/reset all state when starting a new game
- `update(dt)`: Core game logic; reads input, updates positions, checks collisions
- `draw()`: Render current frame state
- `fireWeapon()`: Spawn bullets based on player's current weapon and level
- `updateBoss(dt)`: Boss movement, attack pattern timing, and telegraph
- `spawnEnemy()`: Randomly pick enemy type and spawn off-screen
- `spawnBoss()`: Initialize boss encounter at stage transition
- `applyPowerUp(type)`: Handle power-up collection
- `addScore(base)`: Add to score with multiplier and extend combo timer
- `loseLife()`, `endGame()`: Handle death and game over state
- `togglePause()`: Pause/resume gameplay
- `setDifficulty(name)`: Change difficulty setting

## Common Development Tasks

### Tweaking Difficulty
Edit `DIFFICULTIES` object (lines 485–489):
- `spawnBase`/`spawnMin`: Control enemy spawn frequency (lower = faster)
- `enemySpeed`: Base enemy movement speed multiplier
- `enemyShootChance`: Probability enemy will fire (0–1)
- `lives`, `bombs`: Starting inventory
- `bossHpMul`: Multiplier for boss health pool

### Balancing Enemy Encounters
- `ENEMY_TYPES` (lines 502–506): Damage output (`hp`), speed, appearance frequency, when they unlock
- `KILLS_PER_STAGE` (line 491): How many kills needed before boss
- `spawnEnemy()` (lines 1022–1041): Spawn rate and weighting logic

### Tweaking Boss Battles
- `BOSSES` (lines 508–515): Boss colors, HP multiplier, visual design
- `BOSS_PATTERNS` (lines 1372–1409): Attack sequences; each pattern is a function that fires bullets
- Pattern functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`
- `updateBoss()` (lines 1411–1439): Boss movement and attack timing logic
- Attack pattern changes happen after telegraph (22-frame warning visual)

### Adjusting Weapon Balance
- `fireWeapon()` (lines 1124–1146): Bullet spawn logic for Vulcan vs Laser
  - **Vulcan**: Spread pattern increases with levels; lower damage, fast fire (0.15s cooldown)
  - **Laser**: Multiple lanes with pierce increases with levels; higher damage, slower fire (0.1s cooldown)
- Bullet speed, spread angle, pierce count all configurable here

### Adding/Modifying Sound Effects
The `sfx` object (lines 624–676) contains all sound synthesis functions. To modify:
- `beep(freq, duration, type, volume, options)`: Generate pitched tones
  - `options.sweepTo`: Frequency ramp to another pitch
  - `options.filterFreq`, `options.filterType`: Apply lowpass/highpass filter
  - `options.pan`: Stereo panning (-1 to 1)
- `noiseBurst(duration, volume, options)`: Generate filtered noise
- Add new sfx by calling `beep()` or `noiseBurst()` with appropriate parameters

### Modifying Visuals
- **Canvas size**: `BASE_W = 480, BASE_H = 720` (lines 753–754); aspect ratio is 2:3 (portrait)
- **Colors**: HEX codes sprinkled throughout; major palettes in `THEMES`, `ENEMY_TYPES`, `BOSSES`
- **Drawing functions**: `drawPlane()`, `drawBossShip()`, `drawClouds()` are procedural; modify shapes/colors there
- **Animations**: CSS keyframes in `<style>` for UI overlays; in-code animations (screen shake, fade, bob) use frame counter

### Performance Tuning
- `trackPerf()` and `quality` variable control adaptive quality
- High-quality (0): All particles, glow effects, shadow blur
- Low-quality (2): Fewer particles (35% count), no glow, minimal shadows
- Tweak thresholds in `trackPerf()` if targeting specific devices

### Game Balance Metrics
- **Spawn Difficulty Ramp**: Spawning gets harder over time (`difficultyRamp` in `update()`, line 1201)
- **Combo System**: Multiplier caps at 10x; extends 2.2 seconds per kill
- **Invulnerability**: 1.5 seconds after losing a life
- **Weapon Fire Rate**: Vulcan 0.15s, Laser 0.1s (cooldown)
- **Bomb Damage**: Fixed 24 HP to boss, kills all on-screen enemies with 5-point bonus each

## Testing & Debugging Tips

- **Browser DevTools**: Canvas debugging, performance profiling, localStorage inspection
- **Frame Counter**: `frame` variable increments every update; useful for timing-dependent debug
- **Difficulty Cheat**: Open Console and set `difficulty = 'easy'` to change on-the-fly (next game)
- **Mobile Testing**: Open `index.html` on phone to test touch controls; Firefox DevTools responsive mode works too
- **Audio Context**: AudioContext needs user gesture to initialize; click the screen first
- **Performance Profiler**: Monitor frame times; drop frames indicate quality adjustment is active

## Git Workflow

- Develop on branch: `claude/inspiring-planck-g94i0z`
- Commit frequently with clear messages describing changes
- Push changes to the designated branch

## File Size & Scope

The entire game is 1,669 lines of HTML/CSS/JS in one file. Avoid adding external dependencies; keep everything self-contained. If adding a new feature, ensure it fits within the existing procedural style and doesn't bloat the codebase unnecessarily.

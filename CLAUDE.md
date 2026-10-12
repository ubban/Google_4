# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 canvas application with procedural WebAudio synthesis for sound.

## Running the Game

1. **Open the game in a browser:** Simply open `index.html` in any modern web browser (tested on Chrome, Firefox, Safari, Edge).
2. **No build process required:** The entire game is in a single HTML file with embedded CSS and JavaScript.
3. **Development server (optional):** For local testing with live reload, run: `python3 -m http.server 8000` then navigate to `http://localhost:8000/index.html`

## Architecture

### Single-File Structure

All game logic, rendering, and audio synthesis are contained in `index.html` (~1669 lines). The file is organized into logical sections with comments:

- **HTML/CSS:** Game UI, HUD, controls, responsive canvas sizing
- **JavaScript Game Loop:** Main update and render cycle
- **Input System:** Keyboard, mouse, touch, and virtual joystick
- **Entity Management:** Player, enemies, bullets, particles, power-ups, bosses
- **Audio System:** WebAudio API synthesizer (no external assets)
- **Difficulty System:** Configurable parameters per difficulty level
- **Boss Patterns:** 6 unique boss types with stage-based progression

### Core Game Objects

**Player Object** (`player`)
- Position (x, y), dimensions (w, h)
- Speed: 260px/sec
- Cooldown timer for weapon fire
- Weapon type: 'vulcan' or 'laser' (with level progression)
- Invulnerability timer (`invuln`)
- Rapid-fire timer (`rapid`)

**Enemy Types** (defined in `ENEMY_TYPES`)
- **Grunt:** Basic enemy, fast, low HP
- **Tank:** Armored, slower, higher HP
- **Interceptor:** Fast, weaves, shoots spread patterns

**Boss System** (defined in `BOSSES` array)
- Stage-based boss progression (6 unique boss designs)
- Boss names: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Each boss has unique attack patterns, HP multipliers, and telegraphing behavior
- Boss HP scales with stage and difficulty multiplier

### Key Game Mechanics

**Weapon System**
- **Vulcan:** Rapid-fire straight shots, spread increases with level (0/±0.12/±0.24 radians)
- **Laser:** Piercing shots, higher damage, spread lanes increase with level
- Weapons switch via power-ups; weapon level increases with same power-up type

**Difficulty Levels** (`DIFFICULTIES` object)
- Easy: Lower enemy speed, higher shoot chance, 4 lives, 3 bombs
- Normal: Balanced gameplay (default)
- Hard: Faster enemies, more shooting, fewer lives/bombs
- Settings stored in `localStorage` for persistence

**Combo/Multiplier System**
- Destroying enemies increases chain counter
- Multiplier = 1 + floor(chain / 5), max 10x
- Chain resets if player doesn't destroy enemy for ~2.2 seconds
- Visual feedback: combo popup and stat display

**Difficulty Scaling**
- Enemy spawn rate, speed, shoot chance adjust per difficulty
- Boss HP has difficulty multiplier + unique per-boss multiplier
- Base boss HP: 70 + (stage * 35)

### Input Handling

**Keyboard/Joystick**
- Arrow keys or WASD to move
- Space to fire (auto-repeat while held)
- X, Shift, or Gamepad button for bomb

**Mouse**
- Move cursor over canvas to aim (player glides toward pointer)
- Click to fire
- Keyboard/joystick input takes priority when active

**Touch Controls**
- Left zone: Virtual joystick for movement
- Right zone: Fire button
- Bottom: Bomb button
- Layout adapts to screen size

### Rendering & Visual Effects

- **Canvas sizing:** Base 480×720px, scales up to 1.6x based on viewport (maintains aspect ratio)
- **Screen shake:** Magnitude and duration configurable, disabled if reduced motion enabled
- **Flash effects:** White flash on hit, colored flash for bombs
- **Vignette:** Red pulse effect when player takes damage
- **Particle system:** Explosions spawn particles with velocity, decay over time
- **Reduced motion:** Respects prefers-reduced-motion; disables shake, flash, vignette pulse

### Audio System

- **WebAudio API synth:** No external audio files; all sounds generated at runtime
- **Sound effects:** Shoot, laser, hit, bomb explosion, UI click, boss patterns
- **Procedural generation:** Sweep frequency, filter type, attack/decay envelopes
- **Mute toggle:** Stored in `localStorage`

### Game State Management

**Key State Variables**
- `running`, `paused`, `gameOver`: Game state flags
- `score`, `lives`, `maxLives`: Player progress
- `chain`, `chainTimer`, `multiplier`: Combo system
- `stage`, `stageKills`: Stage progression
- `frame`: Frame counter (used for timing and animations)
- `quality`: Particle/visual quality level (0=high, 1=medium, 2=low)

**Entity Arrays**
- `bullets`: Player projectiles
- `enemies`: Active enemies
- `enemyBullets`: Enemy projectiles
- `particles`: Visual effects
- `powerUps`: Dropped items (life, bomb, weapon)
- `boss`: Current boss (null if none active)

## Editing & Customization

### Difficulty Tuning

Edit the `DIFFICULTIES` object around line ~880:
- `lives`, `bombs`: Starting player resources
- `enemySpeed`, `enemyShootChance`: Enemy behavior scaling
- `spawnRate`, `bossKillsForNext`: Stage progression
- `bossHpMul`: Difficulty-based boss HP multiplier

### Adding/Modifying Enemies

The `ENEMY_TYPES` object defines enemy templates:
- `w`, `h`: Dimensions
- `speed`, `speedMul`: Base speed and difficulty multiplier
- `hp`, `score`: Health and points
- `color`, `dark`: Visual appearance
- `weave`, `shootPattern`: Behavior

Enemy drawing function: `drawEnemyShip()` (around line ~950)

### Boss Customization

The `BOSSES` array defines boss patterns. Each boss has:
- `name`: Display name
- `hpMul`: Health multiplier
- `patterns`: Array of attack pattern functions
- `attacks`: Object mapping pattern types (spread, aimed, spiral, etc.)

Boss update logic: `updateBoss()` function (line ~1411)
Boss drawing: `drawBossShip()` function (line ~970)

### Sound Tweaking

Sound functions in the `sfx` object (around line ~600):
- `sweep()`: FM synthesis for weapon sounds
- `noise()`: Noise-based effects for explosions
- Adjust frequency, filter, duration parameters

## Performance Optimization

### Quality Levels

The game has three quality levels (`quality` variable: 0=high, 1=medium, 2=low):
- Affects particle count in explosions
- Controls shadow rendering precision
- Reduces visual effects on lower-end devices

Adaptive quality: Game monitors frame time and reduces quality if FPS drops

### Frame Rate

- Target: 60 FPS
- Uses `requestAnimationFrame()` for smooth animation
- Delta time calculation: `dt = (currentTime - lastTime) / 1000`
- Independent of monitor refresh rate

## Browser Compatibility

- **Chrome/Edge:** Full support
- **Firefox:** Full support
- **Safari:** Full support (iOS and macOS)
- **Touch devices:** Virtual joystick controls
- **Mobile:** Responsive canvas scaling, touch-optimized UI

## Local Storage

Persistent user preferences:
- `STORAGE_BEST`: High score
- `STORAGE_MUTE`: Audio mute state
- `STORAGE_DIFF`: Difficulty selection
- `STORAGE_MOTION`: Reduced motion preference

## Game Loop

Main update cycle (line ~1148):
1. Input processing (keyboard, mouse, joystick, touch)
2. Player movement and weapon cooldown
3. Bullet updates and collision detection
4. Enemy spawning and behavior
5. Boss pattern execution and damage
6. Particle and effect updates
7. Score and UI updates
8. Canvas rendering

Update functions called per frame:
- `updatePlayer(dt)`: Movement, input, firing
- `updateBullets()`: Position, collision with enemies/boss
- `updateEnemies(dt)`: AI, spawning, shooting
- `updateBoss(dt)`: Pattern selection, attack timing
- `updateParticles(dt)`: Decay and removal
- `updatePowerUps(dt)`: Collection detection

## Tips for Development

- **Testing changes:** Edit values and reload in browser—no build needed
- **FPS monitoring:** Open DevTools → Performance tab, check frame timing
- **Boss testing:** Modify `spawnBoss()` to force a specific stage
- **Visual debugging:** Uncomment the collision rect drawing in render loop
- **Sound debugging:** Use browser DevTools → Audio tab to monitor synthesis
- **Touch testing:** Use Chrome DevTools device emulation with touch enabled

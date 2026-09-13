# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single, self-contained HTML file with no external dependencies. It's implemented entirely in vanilla JavaScript, with procedurally generated audio using the Web Audio API.

### Quick Start

- **Run the game**: Open `index.html` in a web browser (supports desktop and mobile/touch)
- **No build step required**: The game is ready to play immediately
- **No dependencies**: The game uses only browser APIs (Canvas, Web Audio, localStorage)

## Architecture Overview

The entire game exists in one file (`index.html`), organized as a single IIFE (Immediately Invoked Function Expression) containing:

1. **HTML Structure** (lines 1–421): Semantic HTML with CSS-in-head
2. **Game Logic** (lines 422–1669): JavaScript game engine in one IIFE

### High-Level Code Organization

The JavaScript code follows a **classic game loop pattern**:

```
1. Initialize (around line 864)
   ↓
2. Main loop (line 1640: function loop(now))
   - Measure delta time
   - Call update(dt) 
   - Call draw()
   - Request next frame
```

**Key Game Systems**:

| System | Purpose | Location |
|--------|---------|----------|
| **Update Loop** | Physics, collisions, spawning, state changes | `update(dt)` at line 1148 |
| **Rendering** | Canvas drawing, HUD, visual effects | `draw()` at line 1520 |
| **Entity Management** | Player, enemies, bullets, particles, boss, powerups | Global arrays declared at line 864 |
| **Audio Synthesis** | Beeps, noise bursts, and synth sounds | `beep()` and `noiseBurst()` functions, no external audio files |
| **Input Handling** | Keyboard, mouse, touch controls | Event listeners throughout (mouse/touch move, click, keydown/keyup) |
| **Game State** | Lives, score, difficulty, pause, game over | Global variables (line 865+) |
| **UI/HUD** | Score display, health, weapons, boss bar | DOM updates in `update()` and `draw()` |

## Code Structure Breakdown

### Configuration Constants (lines 463–516)

These define all game balancing, visuals, and progression:

- **`DIFFICULTIES`** (line 485): Easy/Normal/Hard modes with spawn rates, enemy stats, starting lives
- **`THEMES`** (line 493): 6 visual themes with sky gradients, cloud colors, sun colors
- **`ENEMY_TYPES`** (line 502): grunt, interceptor, tank—defines their size, HP, speed, scoring
- **`BOSSES`** (line 508): 6 boss types with color schemes and HP multipliers
- **`KILLS_PER_STAGE`** (line 491): Enemy kills needed to progress to the next stage

**Tip**: To add a new difficulty level, enemy type, or boss, just add entries to these constants. The rest of the game logic uses them generically.

### Rendering Functions

- **`drawPlane(x, y, w, h, color, darkColor, flame)`** (line 913): Draws player/boss ships with shading and optional flame
- **`drawBossShip(b)`** (line 970): Boss-specific rendering with wings and core
- **`drawClouds(speed, fillColor, rx, ry)`** (line 1507): Parallax scrolling cloud background

### Game Loop: `update(dt)` (line 1148)

Runs every frame and handles:
1. Input capture (arrow keys, WASD, mouse, touch)
2. Player movement and firing
3. Weapon upgrades and bomb detonation
4. Enemy spawning (spawn timers decrement until enemies spawn)
5. Bullet movement and cleanup
6. Particle system updates (explosions, debris)
7. Collision detection (bullets ↔ enemies, enemy bullets ↔ player)
8. Power-up collection
9. Enemy movement and AI (shooting at player)
10. Boss spawning and behavior (via `updateBoss(dt)`)
11. Stage progression and game over detection
12. HUD updates (score, lives, weapon display)

### Game Loop: `draw()` (line 1520)

Renders each frame in order:
1. Clear canvas with gradient sky
2. Draw parallax clouds
3. Draw all particles
4. Draw power-ups
5. Draw enemy bullets
6. Draw enemies
7. Draw player
8. Draw bullets
9. Draw vignette effect (reduced motion respects prefers-reduced-motion)
10. Draw flash/hit effects

### Entity Representation

**Player**: Single object with properties:
- Position: `x`, `y`
- Size: `w`, `h`
- State: `lives`, `bombs`, `weapon` (current weapon type), `weaponLvl`, `angle` (rotation), `flame` (afterburner effect)
- Firing: `fireRate`, `fireRateTimer`, `fireMode` (spread/laser based on weapon)

**Enemies**: Array of objects, each with:
- Type (grunt, interceptor, tank) from `ENEMY_TYPES`
- Position and size
- HP
- `shootTimer`, `weavePhase` (for interceptor weaving behavior)
- Scoring value

**Bullets**: Array with:
- Position, velocity (`vx`, `vy`)
- Type: player bullets or enemy bullets
- `laser` flag and `pierce` value (for pierce damage)
- Size/width

**Particles**: Temporary visual effects (explosions):
- Position, velocity, lifetime, size, color
- Auto-cleanup when lifetime expires

**Boss**: Single object (created per stage) with:
- Ship rendering via `drawBossShip()`
- Position, movement pattern
- HP and max HP
- Shooting behavior (pattern attack)
- Waves of enemies dispatched before final boss fight

**Power-ups**: Dropped by defeated enemies:
- Type: weapon upgrade or bomb
- Position and velocity
- Auto-cleanup if falling off screen

### Input System

**Keyboard/Gamepad**:
- Arrow keys or WASD: move player
- Space or X: fire
- Shift or X: bomb
- P or Esc: pause

**Mouse**:
- Move to steer
- Click to fire

**Touch** (mobile):
- Drag left zone to move
- Tap FIRE button
- Tap BOMB button

Input state is captured in a `keys` object (`keys['ArrowUp']`, etc.) and polled during `update()`. Touch creates a virtual joystick if detected.

### Difficulty and Balancing

Three difficulties (`DIFFICULTIES`) affect:
- `spawnBase` / `spawnMin`: How often enemies spawn (lower = faster)
- `enemySpeed`, `enemyShootChance`: AI behavior
- `lives`, `bombs`: Starting inventory
- `bossHpMul`: Boss health multiplier

Adding a new difficulty: add entry to `DIFFICULTIES`, update the difficulty button UI if needed, and the game auto-scales.

### Weapon System

Each weapon (defined implicitly) has:
- `fireMode`: How bullets spread (none, spread, wide, laser)
- `fireRate`: Frames between shots
- `color`: Visual bullet color

Weapons upgrade through collected power-ups. Higher levels increase fire rate or unlock new modes (laser, pierce). Weapon levels are capped and reflected in HUD.

### Boss Behavior

Bosses spawn at stage thresholds. `updateBoss(dt)` handles:
- Boss movement (sine wave, spiraling patterns)
- Boss shooting (spreads and patterns)
- Dispatch of enemy waves before final encounter
- Health bar display at top of screen

When boss HP reaches 0, stage increments and next boss spawns.

### Audio System

**Web Audio Synthesis** (no MP3/OGG files):
- `beep(freq, dur, type, vol, opts)`: Sine/square/triangle oscillator with optional sweep, filter, panning
- `noiseBurst(dur, vol, opts)`: White noise with filter envelope
- `ensureAudio()`: Initializes AudioContext (needed for autoplay policies)
- `masterGain`: Central volume control (respects mute state)

**When sounds play**:
- Menu: beep on hover
- Fire: beep with frequency based on weapon
- Hit: noise burst with vignette flash
- Bomb: strong burst
- Power-up: rising beep
- Boss: lower-frequency rumble

All sounds are synthesized in real-time. There are no audio assets to manage.

### Storage and Persistence

Uses `localStorage` for:
- `skyfighter.bestScore`: Highest score achieved
- `skyfighter.muted`: Audio mute state (1 or empty)
- `skyfighter.difficulty`: Last selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: User's motion preference

### Accessibility Features

1. **Screen Reader Support**: `#srLive` element and `announce()` function for status updates
2. **Reduced Motion**: Disables animations when `prefers-reduced-motion: reduce` is set or user toggles 🚫 button
3. **Color Contrast**: Light text on dark backgrounds throughout
4. **Touch-Friendly**: Large buttons (78×78px) with 14px gaps on mobile
5. **Crosshair Cursor**: Visual feedback for aiming

### Performance Optimization

The game includes **adaptive quality** (`trackPerf()`, line 476):
- Measures frame time over 45 frames
- If average > 25ms (below 40 FPS): decrease quality (fewer particles, simpler collisions)
- If average < ~18ms (above 56 FPS): increase quality
- Quality levels: 0 (full), 1 (medium), 2 (low)

This allows smooth play on lower-end devices.

### Theming

Each stage cycles through `THEMES` for visual variety. Themes define:
- Sky gradient colors (3 stops)
- Cloud color (RGB string, used with `rgba()`)
- Sun/light color
- Visual name

Add new themes by pushing to the `THEMES` array. The game auto-cycles.

## Common Development Tasks

### Add a New Enemy Type

1. Add entry to `ENEMY_TYPES` (line 502):
   ```js
   newEnemy: { w: 32, h: 32, hp: 2, speedMul: 1.2, color: '#ffaa00', dark: '#cc6600', score: 25, minStage: 3, weave: false }
   ```
2. Set `minStage` to control when it first appears
3. Set `weave: true` if it should bob side-to-side
4. Rendering is automatic; collision and damage are generic

### Add a New Boss

1. Add entry to `BOSSES` (line 508):
   ```js
   { name: 'NEWBOSS', hullTop: '#color1', hullBottom: '#color2', wing: '#color3', core: '#color4', hpMul: 1.1 }
   ```
2. Colors define the boss ship appearance in `drawBossShip()`
3. `hpMul` scales health relative to base
4. Boss attack patterns are hardcoded in `updateBoss()`—modify that function to customize behavior

### Add a New Difficulty Level

1. Add entry to `DIFFICULTIES` (line 485) with spawn rates, speeds, starting lives, bombs
2. Update difficulty button UI if needed (currently uses data-diff attribute)
3. Game auto-applies scaling

### Modify Enemy Spawn Rate

Change `DIFFICULTIES[difficulty].spawnBase` and `spawnMin` to make enemies appear faster or slower.

### Adjust Weapon Balance

Weapon firing behavior is hardcoded in `update()` around line 1095. Each weapon type has:
- Different fire rates
- Different bullet patterns (spread, laser, etc.)
- Color/visual difference

To tweak, edit the weapon configuration and firing code directly.

### Add a New Theme

Push a new object to `THEMES` array (line 493):
```js
{ sky: ['#color1', '#color2', '#color3'], cloud: 'r,g,b', sun: 'r,g,b', name: 'Theme Name' }
```
The game cycles through themes automatically.

### Test on Mobile

Open the HTML in a phone browser. Touch controls auto-enable if touch is detected. The virtual joystick and button UI are responsive.

## Important Notes

### No Build Step

The game runs directly from the HTML file. No compilation, bundling, or server needed. Just open in a browser.

### Single-File Design

All code, CSS, and HTML are in one file for portability. To edit:
1. Open `index.html` in a text editor
2. Make changes
3. Reload browser (Ctrl+Shift+R for hard refresh to bypass cache)

### Canvas Coordinate System

- Origin (0, 0) is top-left
- X increases rightward
- Y increases downward
- Player spawns at bottom center
- Enemies spawn at top

### Collision Detection

Uses `rectsOverlap(a, b)` for AABB (axis-aligned bounding box) collision. All entities are treated as rectangles using their `x`, `y`, `w`, `h` properties.

### Rendering Order

Important for visual layering:
1. Sky/background
2. Clouds (parallax)
3. Particles (behind bullets)
4. Power-ups
5. Enemy bullets
6. Enemies
7. Player ship
8. Player bullets
9. UI/HUD/vignette

Changing order may cause visual glitches (e.g., bullets appearing behind enemies).

### localStorage Limitations

Persisted data uses localStorage. This is cleared if:
- User clears browser cache
- Private browsing (data lost on close)
- Some browsers with strict privacy settings

For permanent analytics, consider POST-ing to a server.

## Testing and Debugging

To test in-browser:
1. Open DevTools (F12)
2. Console: No errors should appear during normal play
3. Set a breakpoint in `update()` to inspect state
4. Modify global variables at runtime to test edge cases (e.g., `player.lives = 1`)

### Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| No sound | Audio context not initialized | Click game to wake audio (autoplay policy) |
| Low FPS on old devices | Quality level stays at 2 (low) | Quality auto-adapts; check `console.log(quality)` |
| Game won't start | Browser blocking something | Check console for CORS, autoplay, or permission errors |
| Touch controls not working | Touch not detected | Use DevTools device emulation to simulate touch |

### Accessibility Testing

- Keyboard-only: Test with arrow keys, space, x
- Screen reader: Enable OS screen reader and check `srLive` announcements
- Reduced motion: Toggle 🚫 button; animations should stop
- Color contrast: Use DevTools accessibility audit

## Deploying

1. Copy `index.html` to a web server
2. Set CORS headers if embedding in an iframe
3. No other setup needed

The game is fully self-contained and works offline.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file HTML5 Canvas game—a Raiden-style vertical shoot-'em-up where the player controls a fighter jet, manages weapons and bombs, and defeats waves of enemies and bosses across multiple stages.

The entire game is contained in `index.html`:
- **HTML**: Game container and UI overlay (menu, HUD, touch controls)
- **CSS**: Styling for HUD, menus, animations, and responsive layout
- **JavaScript**: Game engine, logic, rendering, audio synthesis, and input handling

### Running the Game

Simply open `index.html` in a modern browser. No build step or dependencies required.

To serve locally: `python3 -m http.server 8000` (or similar), then visit `http://localhost:8000/index.html`.

## Architecture

### High-Level Flow

1. **Initialization** (`lines 463-527`): Load user preferences from localStorage (best score, difficulty, audio mute state, reduced motion).
2. **Game Loop** (`function loop`, `lines 1553-1566`): RequestAnimationFrame-based loop that:
   - Calculates delta time
   - Calls `update(dt)` to advance game state
   - Calls `draw()` to render the frame
   - Tracks performance to adapt rendering quality if needed
3. **Game States**:
   - `running`: Game is active and updating
   - `paused`: Game is paused via menu overlay
   - `gameOver`: Player has lost all lives; overlay shows results

### Key State Variables

- **Player**: Position, velocity, weapon, level, health, invulnerability frames
- **Bullets**: Player projectiles (vulcan bullets or laser beams)
- **Enemies**: Array of active enemies with type, position, health, AI state
- **Enemy Bullets**: Incoming projectiles from enemies
- **Particles**: Visual effects (explosions, hit effects)
- **Power-Ups**: Health, bombs, weapon upgrades
- **Boss**: Single active boss during stage boss phase (null otherwise)
- **Score System**: Current score, combo multiplier, chain counter

### Canvas and Responsive Scaling

- **Base resolution**: 480×720 (portrait orientation)
- **Scaling**: CSS handles responsive scaling; canvas internal resolution is always 480×720
- **Touch support**: Joystick-based movement (left side) + fire/bomb buttons (right side)

## Game Systems

### 1. Rendering (`function draw`, `lines 1512-1550`)

- **Background**: Procedural sky gradient with sun glow and cloud parallax layers (two speeds)
- **Entities**: Player plane, enemies, bosses, bullets, power-ups, particles
- **HUD Elements**: Score, lives (heart icons), bombs, weapon/level info, combo indicator
- **Screen Shake**: Applied during explosions and hits (reduced if user has motion preference enabled)
- **Vignette Effect**: Red pulsing vignette shows when player is on 1 life

### 2. Game Update Loop (`function update`, `lines 1148-1303`)

Core mechanics handled per frame:
- **Player movement**: WASD/arrow keys or joystick input; mouse/touch steering glides player toward pointer
- **Firing**: Space/mouse-click/touch-button fires current weapon
- **Enemy spawning**: Based on timer and difficulty-adjusted spawn rates; spawn types scale with stage
- **Collision detection**: Player bullets ↔ enemies, enemy bullets ↔ player, player ↔ power-ups
- **Enemy AI**: Each enemy type has behavior patterns (spawn, move, shoot, retreat)
- **Combo chain**: Tracks consecutive enemy kills; chain timer resets if no kill for ~2 seconds
- **Damage/Healing**: Track player health, handle invulnerability frames after hit
- **Stage progression**: After 18 kills, boss appears; after boss death, advance stage and reset enemy spawns

### 3. Audio System (Web Audio API, `lines 542-676`)

No external audio files—all sound generated via synthesized tones and noise:
- **`beep(freq, dur, type, vol, opts)`**: Sine/square/sawtooth oscillator with optional sweep, filter, pan, delay
- **`noiseBurst(dur, vol, opts)`**: White noise burst with filter
- **`sfx` object**: Pre-defined sound effects (shoot, laser, explosions, hit, bomb, power-up, boss, UI click, game over)
- **Audio context**: Lazily initialized on first user interaction; suspended state resumed via `ensureAudio()`
- **Mute toggle**: Stored in localStorage; sound functions return early if muted

### 4. Input Handling

**Keyboard:**
- Arrow keys or WASD: Move
- Space or click/touch-button: Fire
- X/Shift/Bomb-button: Drop bomb
- P/Escape: Pause
- Mouse movement: Glide target (only while hovering canvas and no keyboard/joystick input)

**Touch:**
- Left joystick zone (move): Drag to control movement; joystick visual centers on touch origin and stick clamped to 50px radius
- Right fire button: Press to fire (continuous while held)
- Right bomb button: Tap to drop bomb

### 5. Enemy Spawning and AI (`function spawnEnemy`, `lines 1022-1042`)

- **Enemy types** (defined in `ENEMY_TYPES` constant):
  - **Grunt**: Slow, 1 HP, weak shooter
  - **Interceptor**: Fast, 1 HP, weaves side-to-side
  - **Tank**: Slow, 4 HP, heavy hitter
- **Spawn logic**: Timer-based; difficulty modulates spawn rate and enemy properties (speed, health, shoot chance)
- **AI**: Move downward; periodically shoot in aimed or random patterns; all enemies despawn when off-screen

### 6. Boss Fights (`function spawnBoss`, `lines 1043-1057`)

- **Boss types** (6 total in `BOSSES` constant): Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan
- **Boss appearance**: Cycle through bosses by stage index
- **Rendering**: Custom 2D drawing (`drawBossShip`, `lines 970-1020`)—hull, wings, core; distinct color per boss
- **Boss AI** (`function updateBoss`, `lines 1411-1440`):
  - Health bar displayed while alive
  - Telegraph phase: Boss charges with visual warning sound
  - Attacks: Rotate through 6 attack patterns (radial fire, aimed spread, columns, spiral burst, cross, homing)
  - Movement: Sway side-to-side during attack
  - Defeat: Explosion, stage clear sound, score bonus
- **Boss health multiplier**: Difficulty adjusts HP; later bosses are tougher

### 7. Scoring and Progression

- **Base points**: Enemies grant 10–35 points based on type
- **Multiplier**: Increases via weapon level-ups or combo chain (up to ~8x)
- **Combo chain**: Kill counter; broken if no kill for ~2 seconds. Displays "COMBO ×N" when chain ends
- **Weapon progression**: Kill count increases weapon level (capped at 3); level increases bullet count/spread
- **Stage progression**: Stage advances every 18 kills; themes and boss cycle per stage

## Customization Points

### 1. Difficulty Settings

In `DIFFICULTIES` constant (`line 485`):
- `spawnBase`, `spawnMin`: Enemy spawn timing (lower = more enemies)
- `enemySpeed`: Multiplier for enemy movement speed
- `enemyShootChance`: Probability enemy fires each frame
- `lives`, `bombs`: Starting values
- `bossHpMul`: Boss health multiplier

### 2. Enemy Types

In `ENEMY_TYPES` constant (`line 502`):
- `w`, `h`: Bounding box for collision
- `hp`, `speedMul`: Health and speed scaling
- `color`, `dark`: Fill and shading colors
- `score`: Points awarded
- `minStage`: Earliest stage to spawn this type
- `weave`: Boolean flag for interceptor side-to-side movement

### 3. Bosses

In `BOSSES` constant (`line 508`):
- `name`: Displayed in boss health bar
- `hullTop`, `hullBottom`, `wing`, `core`: Colors for 2D drawing
- `hpMul`: Health multiplier relative to base

### 4. Themes (Sky Gradients)

In `THEMES` constant (`line 493`):
- `sky`: Array of 3 colors for gradient (top, middle, bottom)
- `cloud`: Cloud fill color (RGB values)
- `sun`: Sun glow color (RGB values)
- `name`: Debug/reference label

### 5. Weapons

Weapon upgrade behavior is in `function update` (search `player.level`):
- Vulcan: Bullet count increases with level (1 → 3 → 5)
- Laser: Fires after 8 kills; more powerful
- Bullet speed, fire rate stored in `function fireWeapon` (`line 1124`)

To modify firing patterns, adjust constants near `function fireWeapon` or edit the specific attack functions (e.g., `fireRadial`, `fireColumns`).

### 6. Sound Effects

All sounds are in the `sfx` object (`lines 624-676`). Each uses `beep()` or `noiseBurst()`:
- Modify frequency, duration, filter, or pan to change the sound
- Set `muted = true` to disable all audio globally (or toggle via UI button)

## Important Implementation Details

### Collision Detection

`function rectsOverlap` (`line 1069`): Simple AABB collision check. All entity collisions use this.

### Adaptive Performance

`function trackPerf` (`line 476`): Maintains rolling average of frame times. If frame time > 25ms (40 FPS), `quality` increases (lowers fidelity). If < ~18ms (56 FPS), quality decreases. This is tracked but not yet applied—skeleton for future feature.

### Screen Shake

Global `shakeMag` and `shakeTime` variables. Horizontal/vertical random offset applied in `draw()` if `shakeTime > 0`. Disabled if user has reduced motion preference.

### Persistent Storage

Uses localStorage for:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Selected difficulty
- `skyfighter.reducedMotion`: Accessibility preference

### Reduced Motion Support

Respects `prefers-reduced-motion` media query on first load. User can override via button. Disables:
- Screen shake
- Flash animations on hit/bomb
- Vignette pulsing
- Animation keyframes (CSS reduced-motion rules)

## Common Tasks

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` with desired properties
2. Update `minStage` to control when it spawns
3. If behavior differs (e.g., weaving), add logic to `function update` where enemies are processed

### Tweak Boss Difficulty
1. Modify `bossHpMul` in the boss definition in `BOSSES`
2. Or adjust `DIFFICULTIES[difficulty].bossHpMul` to scale all bosses at once

### Add a New Stage Theme
1. Push new theme object to `THEMES` array
2. Themes cycle by stage: `(stage - 1) % THEMES.length`
3. Pick distinct sky/cloud/sun colors

### Change Weapon Behavior
1. Locate `function fireWeapon` and the specific firing patterns (e.g., `fireRadial`)
2. Modify bullet count, spread angle, speed, or life span
3. Weapon progression occurs at `// weapon upgrade` section of `update()` function

### Adjust Enemy Spawn Rate
1. Modify `spawnBase` and `spawnMin` in `DIFFICULTIES`
2. Or change spawn timer logic: search `spawnTimer` in `update()`

## Testing Tips

- Open browser DevTools console to inspect `score`, `lives`, `enemies.length`, `boss`, etc. at runtime
- Use pause (P key) to inspect frame state
- Difficulty select buttons on start screen allow testing different balance
- localStorage can be cleared in DevTools to reset saved progress and preferences

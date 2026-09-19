# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game, implemented as a single self-contained HTML file. It features procedurally-generated combat stages, boss fights, weapon progression, and a fully synthesized audio system with no external assets.

## Single-File Architecture

The entire game is in `index.html` (1669 lines):
- **Lines 1–421**: HTML markup + inline CSS
- **Line 422 onward**: Inline JavaScript (IIFE pattern)

All game logic, rendering, audio synthesis, and UI are self-contained within one file. There are no external dependencies or asset files.

## Running the Game

### Local Testing
Simply open `index.html` in a web browser (any modern browser with HTML5 Canvas and WebAudio API support). No build step, server, or external libraries needed.

```bash
# Option 1: Open in default browser (macOS/Linux)
open index.html

# Option 2: Serve locally to test responsive design
python3 -m http.server 8000
# Then navigate to http://localhost:8000/index.html
```

## Game Architecture

### Game Loop (`requestAnimationFrame`)
The main animation loop (defined near line 1652) runs at 60 FPS:
1. **Input Handling**: Keyboard, mouse, and touch inputs update player state
2. **Update Phase**: `update(dt)` processes physics, collision detection, and spawning
3. **Boss Logic**: `updateBoss(dt)` handles boss-specific behavior
4. **Render Phase**: Canvas rendering draws all sprites and effects
5. **UI Update**: HUD elements (score, health, weapons) are updated

### Core Game Objects

**Player Object** (line ~877):
- `x, y`: Position on canvas
- `w, h`: Collision dimensions (40×40)
- `speed`: Movement velocity (260 px/s)
- `weapon`: Current weapon type ('vulcan' or 'laser')
- `level`: Weapon upgrade level (1–3)
- `cooldown`: Frames until next shot
- `invuln`: Invulnerability frames (after hit)
- `rapid`: Rapid-fire duration counter

**Bullets Array**: Contains all active projectiles from the player
- `x, y`: Position
- `type`: 'vulcan' or 'laser' (determines speed and appearance)
- `vx, vy`: Velocity components

**Enemies Array**: Contains all active enemy units
- `x, y`: Position
- `type`: 'grunt', 'interceptor', or 'tank' (determines hp, speed, behavior)
- `hp`: Current health
- `shootCooldown`: Frames until next enemy shot
- Derived properties: `w`, `h`, `speed`, `color` from `ENEMY_TYPES` constants

**Boss Object** (when active):
- Drawn procedurally; shape depends on boss type
- `hp, maxHp`: Health tracking
- `phase`: Current attack pattern phase
- `shootCounter`: Shot spawning timer
- Defined in `BOSSES` array (line ~508) with color and stat multipliers

### Key Constants

**Difficulty Presets** (line ~485):
- Easy, Normal, Hard: each defines spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier
- Current difficulty stored in localStorage under `'skyfighter.difficulty'`

**Enemy Types** (line ~502):
- `grunt`, `interceptor`, `tank`: each with hp, speed multiplier, color, score value

**Boss Types** (line ~508):
- 6 bosses: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Spawned sequentially; one per stage

**Visual Themes** (line ~493):
- 6 stage themes (Daylight Front, Sunset Corridor, Night Approach, etc.)
- Each theme defines sky gradient colors and visual palette

### Weapon System

Two weapon types:
- **Vulcan** (default): Rapid shots, lower damage per bullet
- **Laser**: Slower fire rate, piercing damage

Weapon upgrades (level 1–3) increase fire rate and damage. Progress tracked in `player.level`. Upgrades drop from defeated enemies and are collected automatically.

### Combo/Score Multiplier

The score multiplier increases with consecutive enemy kills without getting hit. It resets to 1× when the player takes damage. Visual feedback:
- HUD shows current multiplier
- Combo popup animates when threshold is hit
- Multiplier affects points earned per kill

### Audio System (WebAudio Synthesis)

No external audio files. All sounds are synthesized using the WebAudio API:

**Sound Effects** (defined in `sfx` object, line ~620):
- `shoot`, `laser`: Beam fire sounds
- `explodeGrunt`, `explodeInterceptor`, `explodeTank`: Enemy destruction
- `hit`: Player damage
- `bomb`: Smart bomb detonation
- `powerup`: Weapon/upgrade pickup
- `bossHit`, `bossTelegraph`, `bossDown`: Boss-related sounds
- `stageClear`, `gameover`, `uiClick`: UI and stage transitions

**Audio Synthesis Functions**:
- `beep(freq, dur, type, vol, opts)`: Oscillator-based tones with optional sweep, filter, pan
- `noiseBurst(dur, vol, opts)`: White noise for percussive effects
- `panFor(x)`: Calculates stereo pan based on x-position (adds spatial audio)

Toggle muting with the mute button; state stored in `'skyfighter.muted'`.

### Touch & Input

**Keyboard Controls**:
- Arrow keys / WASD: Move player
- Z / Space: Fire weapon
- X / Shift: Use bomb

**Touch Controls** (displayed on mobile):
- Left joystick: Movement (draggable analog input)
- Fire button: Continuous fire
- Bomb button: Smart bomb detonation

Touch controls conditionally shown based on device detection; joystick tracks movement in real-time.

### Performance Optimization

Adaptive quality system (line ~474):
- Tracks average frame time over 45 frames
- Adjusts `quality` variable (0 = full, 2 = low quality) based on performance
- Used to reduce particle effects, enemy counts, or render detail on slower devices

### Collision & Physics

- **AABB (Axis-Aligned Bounding Box)**: Simple rectangle collision detection
- **Bullet-Enemy**: Bullets hit enemies; bullets removed or damage tracked
- **Bullet-Boss**: Similar; boss hp decremented
- **Enemy Bullet-Player**: Triggers player hit if outside invulnerability window
- **Gravity**: Enemies and projectiles move in straight lines; no gravitational physics

### Stage & Spawning System

Stages are procedural:
- `KILLS_PER_STAGE = 18`: Each stage requires 18 enemy kills before boss appears
- Spawning uses difficulty-based timers and randomization
- Boss chosen from `BOSSES` array; cycles sequentially
- Stage themes rotate; visual changes only (no mechanic differences)

### Game States

Managed by overlay visibility and state flags:
- **Menu**: Overlay shown; waiting for player to select difficulty and press Start
- **Playing**: Game loop active; overlay hidden
- **Game Over**: Overlay shown with final score, best score, stage reached

### localStorage Keys

- `'skyfighter.bestScore'`: High score (number)
- `'skyfighter.muted'`: Audio mute state ('0' or '1')
- `'skyfighter.difficulty'`: Saved difficulty ('easy', 'normal', 'hard')
- `'skyfighter.reducedMotion'`: Reduced motion preference ('0' or '1')

## Development Tasks

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` constant (line ~502):
   ```javascript
   myType: { w: 40, h: 40, hp: 2, speedMul: 1.2, color: '#ff0000', dark: '#cc0000', score: 25, minStage: 3 }
   ```
2. Add explosion sound (optional) to `sfx` and `EXPLODE_SFX` mappings
3. Update enemy spawning logic to include new type at appropriate difficulty/stage

### Adding a New Boss

1. Add entry to `BOSSES` array with unique name and color scheme
2. Implement boss-specific rendering logic in the render section (procedurally drawn)
3. Add boss attack patterns in `updateBoss(dt)` function
4. Tune hp multiplier for balance

### Tuning Difficulty

Modify `DIFFICULTIES` object values:
- `spawnBase`: Base interval (frames) between enemy spawns
- `spawnMin`: Minimum interval (spawning accelerates)
- `enemySpeed`: Multiplier for enemy movement speed
- `enemyShootChance`: Probability per frame that enemy fires
- `lives`: Starting player health
- `bombs`: Starting bomb count
- `bossHpMul`: Health multiplier for bosses

### Modifying Visuals

- **Colors**: Update `THEMES` (line ~493), `ENEMY_TYPES`, or `BOSSES` color properties
- **Animations**: Modify CSS `@keyframes` sections or timing parameters
- **UI Layout**: Adjust CSS positioning, fonts, and spacing in the `<style>` section
- **Canvas Drawing**: Rendering code is in the main render loop; search for `ctx.fill`, `ctx.stroke`

### Testing Audio

Audio synthesis can be tested by toggling mute and triggering game events (firing, hitting enemies, boss fights). The synth uses procedural waveforms (sine, triangle, square, sawtooth) and noise bursts—no audio files to edit.

## Notable Quirks & Implementation Details

1. **Reduced Motion Support**: Respects `prefers-reduced-motion` media query; disables animations and vignette flash effects on low-power or accessibility-focused devices
2. **Responsive Canvas**: Uses CSS `max-width`/`max-height` to scale the game to fit the viewport while maintaining aspect ratio
3. **Boss Telegraphing**: Boss attacks are predictable (multiple phases); players can learn and counter patterns
4. **Rapid Fire Boost**: Picking up weapons grants temporary rapid-fire mode (`player.rapid` counter)
5. **Procedural Boss Rendering**: Each boss is drawn using Canvas primitives (rectangles, triangles) based on color scheme; no sprites
6. **Spatial Audio**: Sound effects are panned left/right based on their x-position for immersion

## Accessibility

- Full keyboard support for all input
- Touch joystick for mobile play
- Screen reader announcements (aria-live region) for game events
- Reduced motion toggle to disable animations/effects
- High-contrast color scheme for readability
- Text at responsive sizes (clamp) for mobile & desktop

## Browser Compatibility

Requires:
- HTML5 Canvas 2D context
- WebAudio API (for sound synthesis)
- ES6 JavaScript (arrow functions, template literals, const/let)
- CSS Grid, Flexbox, CSS Variables (modern browsers)

Works on all modern browsers (Chrome, Firefox, Safari, Edge) and mobile platforms.

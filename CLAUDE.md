# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The game runs entirely in the browser with no external dependencies or build system. All graphics are rendered via Canvas API, and all audio is synthesized using Web Audio API.

## Development & Testing

### Running the Game
Simply open `index.html` in a web browser. The game is fully self-contained and requires no build step, server, or dependencies. It works on desktop (mouse + keyboard), tablet (touch controls), and mobile devices.

### Local Testing
- **Desktop**: Open in any modern browser (Chrome, Firefox, Safari, Edge)
- **Mobile**: Use your phone's browser or remote debugging tools
- **Controls**: Mouse to move/click to fire, Arrows/WASD to move, Space to fire, X/Shift for bombs

### No Build System
There are no package.json, build scripts, linters, or tests. All development happens directly in `index.html`. Changes take effect immediately upon refresh.

## Architecture & Code Structure

The entire game is implemented in a single `<script>` block within `index.html` (~1200 lines of JavaScript). The architecture follows a procedural approach with the following major sections:

### 1. **Game State**
Global variables maintain game state:
- `player`: Player ship object with position, health, weapon, level
- `enemies`: Array of active enemy ships
- `boss`: Current boss object (null when not in boss stage)
- `bullets`: Array of player projectiles
- `enemyBullets`: Array of enemy projectiles
- `particles`: Visual effects particles
- `powerUps`: Dropped power-ups (weapon upgrades, health, bombs)
- `score`, `lives`, `stage`, `kills`: Game progress tracking

### 2. **Game Loop & Timing** (lines ~1640-1660)
- `loop(now)`: Main animation frame loop that calls `update()` then `draw()`
- `update(dt)`: Processes all physics, collisions, and game logic at 60fps
- `draw()`: Renders everything to canvas using procedural drawing (no sprites)
- Adaptive performance scaling: Quality variable (0–2) adjusts rendering complexity based on frame time

### 3. **Configuration Tables**
Core gameplay parameters defined as objects:
- `DIFFICULTIES`: easy/normal/hard settings (spawn rates, enemy speed, lives, bombs, boss HP multiplier)
- `ENEMY_TYPES`: Enemy stats (grunt, interceptor, tank) with hp, speed, scoring
- `THEMES`: 6 visual themes with sky gradients, cloud colors, sun rendering for each stage
- `BOSSES`: 6 boss designs (VANGUARD, BEHEMOTH, SPECTER, etc.) with hull colors and HP scaling

### 4. **Input Handling** (lines ~785-870)
- Keyboard: `keys` object tracks pressed state (arrows, WASD, space, X)
- Mouse: `mouseTarget` for aiming, drag-based touch controls for mobile
- Joystick emulation on touch devices: `joystickStart/Move/End` functions manage virtual stick
- Touch buttons: FIRE and BOMB buttons appear on mobile

### 5. **Audio System** (lines ~542-678)
- Web Audio synth: All sounds are generated, not sampled
- `beep(freq, dur, type, vol, opts)`: Oscillator-based sounds with frequency sweep
- `noiseBurst(dur, vol, opts)`: Procedural noise for explosions and impacts
- Sound effects: shoot, laser, explosions (grunt/interceptor/tank variants), hit, bomb, boss warning, UI clicks
- No external audio files — everything is synthesized at runtime

### 6. **Rendering** (lines ~913-1520)
Procedural drawing functions:
- `drawPlane(x, y, w, h, color, darkColor, flame)`: Renders ships (player, enemies, bosses)
- `drawBossShip(b)`: Boss-specific rendering with larger hull and core
- `draw()`: Main render function; draws sky (gradient + sun), clouds, player, enemies, bullets, particles, UI

**Theme System**: Sky background, sun glow, and cloud colors rotate based on stage to create visual progression.

### 7. **Weapon & Projectile System** (lines ~1124-1370)
- Player weapons: regular bullets and laser (piercing)
- Weapon levels: improve damage and fire rate; laser penetrates enemies
- Boss projectile patterns: 
  - `fireRadial()`: Circular spread
  - `fireAimedSpread()`: Aimed at player
  - `fireColumns()`: Vertical lines
  - `fireSpiralBurst()`: Rotating spiral
  - `fireHomingSeeded()`: Homing projectiles
  - `fireDoubleRing()`: Two concentric rings

### 8. **Collision & Combat** (lines ~1188-1300)
- Hitbox overlap detection: `rectsOverlap(a, b)`
- Enemy-bullet collision checks and damage tracking
- Boss-bullet collision with phase-based health bars
- Player-bullet collision (invulnerability frames after hit)
- Chain multiplier: Consecutive kills without being hit increases score multiplier

### 9. **Enemy & Boss AI** (lines ~1022-1411)
- **Enemy spawning**: Timer-based with difficulty-adjusted spawn rates
- **Enemy movement**: Downward motion with occasional left-right weaving for interceptors
- **Enemy firing**: Probability-based shooting at player position
- **Boss lifecycle**: Spawns after 18 kills per stage; fires in patterns, repeats phases
- **Boss destruction**: Advances to next stage, resets enemy spawn counter

### 10. **Power-Up System** (lines ~1058-1068, 1304-1323)
- Spawn chance when enemies die
- Types: weapon upgrades (regular → laser), health restore, bomb ammo
- `applyPowerUp(type)`: Handles power-up effects

### 11. **Game State Management** (lines ~875-912, 1469-1503)
- `resetGame()`: Initialize stage, enemies, player state
- `togglePause()`: Pause/resume gameplay
- `endGame()`: Game over sequence, show score, check best score
- Persistence: localStorage stores best score, mute state, difficulty, reduced motion preference

### 12. **UI & HUD** (lines ~680-730)
- Lives display as hearts (full/empty)
- Bombs remaining count
- Current weapon name and level
- Score and best score
- Stage banner on stage transition
- Combo notifications on kill streaks
- Screen shake and flash effects for visual feedback

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line ~502) with stats: w, h, hp, speedMul, color, dark, score, minStage, optional weave
2. Spawning is automatic based on `minStage` threshold

### Adding a New Boss Design
1. Add entry to `BOSSES` array (line ~508) with: name, hullTop, hullBottom, wing, core colors, hpMul
2. Rendering is automatic; boss appears every 18 kills per stage

### Adding a New Visual Theme
1. Add entry to `THEMES` array (line ~493) with: sky (3-color gradient), cloud (RGB string), sun (RGB string), name
2. Theme cycles automatically per stage: `(stage - 1) % THEMES.length`

### Adding a New Sound Effect
1. Add function to `sfx` object (line ~624)
2. Call `beep()` or `noiseBurst()` with synthesized parameters
3. Call from gameplay events

### Tweaking Game Balance
- **Difficulty**: Edit `DIFFICULTIES` object (line ~485) for spawn rates, enemy speed, lives
- **Enemy strength**: Modify `ENEMY_TYPES` stats or enemy shoot chance in difficulty config
- **Boss difficulty**: Adjust `bossHpMul` in difficulty config
- **Weapon balance**: Modify `fireDelay` (line ~1177), bullet speed, or damage in `update()`

## Code Style & Patterns

- **Procedural approach**: No classes; all logic in functions and global state
- **Canvas drawing**: Shapes are primitives (rect, arc, ellipse, path); no sprite assets
- **Performance**: Adaptive quality setting based on frame time; reduces shadow effects under load
- **Mobile-first**: Touch controls and responsive UI via CSS clamp() and vh/vw units
- **Accessibility**: ARIA live region for screen readers (`srLive` element)
- **Reduced motion**: Respects `prefers-reduced-motion` and allows user override

## Browser APIs Used

- **Canvas 2D**: All graphics rendering
- **Web Audio**: Sound synthesis
- **LocalStorage**: Persistence (best score, settings)
- **Touch Events**: Mobile controls
- **Pointer Events**: Mouse/touch input

## Game Flow

1. **Start screen**: Difficulty selection, display best score
2. **Stage**: Spawn enemies until 18 kills reached
3. **Boss stage**: Boss fight with pattern-based attacks
4. **Victory/Defeat**: Show final score, check if new best
5. **Loop**: Continue to next stage on victory, return to start on game over

## Performance Considerations

- Frame time is tracked and used to adjust rendering quality (no shadows/glow on slower devices)
- Enemy and bullet arrays are pruned to remove off-screen objects
- Particle effects are capped and recycled
- Audio context created on first user interaction (lazy initialization)

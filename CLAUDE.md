# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file HTML5 vertical shoot-'em-up game (Raiden-style). The entire codebase is contained in `index.html` (~1,670 lines), which includes embedded CSS for styling and JavaScript for game logic. No build process, bundler, or external dependencies are required.

## Running and Developing

**To play the game:**
- Open `index.html` directly in a web browser (Chrome, Firefox, Safari, Edge all work)
- The game is fully responsive and works on desktop and mobile

**To modify the game:**
- Edit `index.html` directly
- Refresh the browser to see changes immediately
- Use browser DevTools (F12) to debug via console

## Game Architecture

The game is organized as a single-file application with clear sections:

### 1. **Embedded CSS (lines 10–359)**
- Responsive layout using flexbox and CSS Grid
- Theme colors and animations (keyframes)
- HUD styling (score, health, weapon, boss bar)
- UI overlays (menu, game over screen)
- Touch controls for mobile (joystick and buttons)
- Reduced-motion media query support for accessibility

### 2. **JavaScript Game Logic (lines 422–end)**
The code is organized functionally, not in classes. Key subsystems:

#### **Configuration Constants** (lines 485–523)
- `DIFFICULTIES`: Three difficulty presets (easy, normal, hard) controlling spawn rates, enemy speed, health, and boss HP
- `THEMES`: Six environmental themes affecting sky gradient and colors
- `ENEMY_TYPES`: Definitions for grunt, interceptor, and tank enemies (visual, speed, HP, score)
- `BOSSES`: Six unique boss definitions (visual colors, HP multiplier)
- `KILLS_PER_STAGE`: Number of enemy kills to advance to the next stage

#### **Game State** (global variables)
- `player`: Position, velocity, health, weapon, invulnerability timer
- `enemies`: Array of active enemies
- `playerProjectiles`, `enemyProjectiles`, `bombs`: Active game objects
- `score`, `mult` (combo multiplier), `stage`, `lives`, `bombs`: Player progress
- `running`, `paused`, `gameOver`: Game state flags

#### **Audio System** (lines 542–676)
- WebAudio API synthesis—no external sound files
- `beep()`: Generates tonal sounds (frequency sweep, filter, panning)
- `noiseBurst()`: Generates noise-based sounds (explosions, impacts)
- `sfx` object: Sound effects for all game events (shoot, explode, boss down, etc.)
- Mute toggle stored in localStorage

#### **Rendering and Update Loop** (lines ~698–end)
- `draw()`: Renders sky gradient, clouds, player, enemies, projectiles, HUD, overlays
- `update()`: Game physics and collision detection (runs each frame via requestAnimationFrame)
- `tick()`: High-level game logic (spawn enemies, progress stage, check win/loss conditions)
- Adaptive performance: Tracks frame time and reduces quality (lower draw calls) if FPS drops below 40

#### **Input Handling**
- Mouse: Move player to cursor; click to fire
- Keyboard: Arrow keys / WASD to move, Space to fire, X / Shift to drop bomb, P / Esc to pause
- Touch: Left side of screen to move (joystick), right-side buttons for fire and bomb
- Joystick base appears dynamically on first touch

#### **Game Mechanics**
- **Progression**: Kill 18 enemies per stage → advance to next stage with a new boss
- **Weapon system**: Collect weapon pickups; cycle through Vulcan (spread), Laser (focused), and back
- **Bombs**: Limited smart bombs that clear screen and damage boss heavily
- **Multiplier**: Hit combo increases score multiplier; breaks on death or long gaps
- **Invulnerability**: Brief invuln after hit, blinking animation
- **Boss mechanics**: Boss telegraphs incoming attack patterns; damage reduces HP bar

## Key Code Sections to Understand

### Spawning Enemies
- `spawnEnemy()` function controls spawn frequency (difficulty-dependent)
- Weighted random selection of enemy type based on current stage
- Enemy difficulty ramps (faster, harder enemies as stage increases)

### Collision Detection
- Simple AABB (axis-aligned bounding box) checks for projectiles, bombs, and enemies
- Boss collision handled separately with attack telegraphing

### Drawing the Player
- `drawPlane()` function renders the player sprite (triangle-based)
- Laser weapon adds a glow effect
- Invulnerability causes blinking (every 4 frames skips draw)

### Visual Effects
- **Screen shake**: `screenShake()` on hit; reduces based on `reducedMotion` preference
- **Flash**: White flash on hit or bomb, driven by CSS animations
- **Vignette**: Red pulse effect when health is critical (1 HP)
- **Combo popup**: Floating "+X" text on kill streaks
- **Stage banner**: Large text on stage transition

## localStorage Keys

The game persists settings:
- `skyfighter.bestScore`: Best score achieved
- `skyfighter.difficulty`: Last selected difficulty (easy/normal/hard)
- `skyfighter.muted`: Mute state (1 or 0)
- `skyfighter.reducedMotion`: Accessibility preference (1 or 0)

## Common Modifications

### Add a new enemy type
1. Add entry to `ENEMY_TYPES` object with properties: `w` (width), `h` (height), `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Add explosion sound to `EXPLODE_SFX` mapping (optional)
3. Add a case in `spawnEnemy()` weighting logic if stage-specific

### Change difficulty balance
- Edit `DIFFICULTIES` object (spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul)

### Add a new visual theme
- Add an entry to `THEMES` array with `sky` (3-color gradient), `cloud` (RGB values), `sun` (RGB values), and `name`

### Add a new boss
- Add entry to `BOSSES` array with `name`, `hullTop`, `hullBottom`, `wing`, `core` (colors), and `hpMul`
- Boss encounters use `(stage - 1) % BOSSES.length` to cycle through bosses

## Performance Considerations

- **Canvas rendering**: Direct 2D context calls; no scene graph or library overhead
- **Adaptive quality**: Frame-time tracking disables some visual effects (clouds, details) on slower devices
- **No external assets**: All graphics drawn with Canvas API; all audio synthesized with WebAudio API
- **Touch optimization**: Touch controls hidden on desktop; joystick base only shown when needed

## Accessibility

- **Reduced-motion**: Respects `prefers-reduced-motion` media query; user can toggle; disables all animations when enabled
- **Keyboard support**: Full game playable without mouse
- **Screen-reader support**: ARIA live region (`#srLive`) announces game state changes
- **High contrast**: Game uses bright colors on dark background; text uses text-shadow for visibility

## Testing and Validation

To validate changes:
1. Test all three difficulties in browser DevTools mobile mode (responsive design)
2. Verify keyboard, mouse, and touch controls work
3. Check DevTools console for errors
4. Test mute, motion toggle, difficulty change
5. Play through a stage to verify enemy progression and boss encounter
6. Check that best score persists across page refreshes

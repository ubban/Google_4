# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The game features:
- Player-controlled fighter aircraft with weapon upgrades
- Multiple enemy types (grunts, interceptors, tanks) with escalating difficulty
- Boss encounters at stage intervals
- Three difficulty levels (Easy, Normal, Hard)
- Procedural theme generation (6 different visual themes)
- Web Audio synthesized soundtrack and SFX
- Touch and keyboard controls with adaptive mobile UI
- Combo multiplier system
- Smart bomb mechanic
- Adaptive performance scaling

## File Structure

- `index.html` - Complete game application (1669 lines)
  - Embedded CSS (styles, animations, responsive layout)
  - Embedded JavaScript (game engine, rendering, input handling)
  - Minimal HTML structure (container, canvas, overlay UI, HUD)

## Architecture Overview

### Game State & Objects

The game maintains several object collections updated in the main game loop:

- **Player** - Single fighter with position, velocity, health, weapon level, bombs, invulnerability
- **Bullets** - Player projectiles (standard bullets or lasers)
- **Enemy Bullets** - Incoming enemy fire
- **Enemies** - Array of active enemy aircraft
- **Boss** - Current stage boss (null when no boss is active)
- **Power-ups** - Floating weapon upgrades, health pickups

### Game Flow

1. **Initialization Phase**
   - Load saved game state (best score, difficulty, audio mute, reduced motion preference)
   - Initialize canvas with responsive scaling
   - Set up touch/keyboard input handlers

2. **Start Screen**
   - Overlay card shows best score, difficulty selection, controls hint
   - Awaits player to click "Start Game"

3. **Main Game Loop** (requestAnimationFrame)
   - Update game state (player movement, bullets, enemies, collisions)
   - Spawn new enemies based on stage and difficulty
   - Check for stage completion (18 kills) and spawn boss
   - Detect collisions (bullets vs enemies, enemies vs player, powerups vs player)
   - Update visuals (HUD, combo popup, health bars)
   - Render canvas (background, player, enemies, bullets, effects)
   - Track performance and adapt particle/glow quality

4. **Game Over**
   - Display final stats on overlay
   - Offer restart option
   - Update best score if beaten

### Key Constants & Configuration

- **DIFFICULTIES** - Spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier per difficulty
- **ENEMY_TYPES** - grunt, interceptor, tank with size, HP, speed, score values
- **BOSSES** - 6 unique boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with color schemes
- **THEMES** - 6 visual themes (sky gradients, cloud color, sun color, name)
- **KILLS_PER_STAGE** - 18 enemies to clear per stage before boss spawns

### Rendering & Effects

- **Canvas Rendering** - 2D context with transformed coordinate system (BASE_W=512, BASE_H=640)
- **Particle System** - Explosions, impacts, powerup collection effects
- **Visual Effects** - Screen vignette on hit, screen flash on bomb, combo popup
- **Audio** - Web Audio API synthesizer (no external audio files)

### Input Handling

- **Keyboard** - Arrows/WASD (movement), Space (fire), X/Shift (bomb), P/Esc (pause)
- **Mouse** - Move to steer, click to fire
- **Touch** - Left zone for joystick movement, right zone for fire/bomb buttons
- **Pause** - P or Esc key toggles pause (grays overlay, freezes game updates)

### Performance Adaptation

The game monitors frame times and adapts visual quality:
- Quality level 0: Full effects (glow, particles)
- Quality level 1: Medium effects
- Quality level 2: Minimal effects (no glow, fewer particles)

Quality is stepped up if frame time exceeds 1/40s, down if consistently below 1/56s.

## Development Tasks

### To modify game balance

1. **Difficulty parameters** - Edit `DIFFICULTIES` object (spawn rates, enemy speed, player lives, bombs, boss HP)
2. **Enemy attributes** - Edit `ENEMY_TYPES` (size, HP, speed multiplier, score)
3. **Boss attributes** - Edit `BOSSES` (name, color, hpMul scaling)
4. **Stage progression** - Edit `KILLS_PER_STAGE` (18 by default)

### To adjust visual appearance

1. **Themes** - Edit `THEMES` array (sky gradient colors, cloud color, sun color)
2. **Canvas scaling** - Edit `BASE_W` and `BASE_H` constants
3. **HUD styling** - Modify CSS in `<style>` block (responsive sizes use `clamp()`)
4. **Player/enemy colors** - Edit in `drawPlane()` function or object definitions

### To add new enemy types or boss designs

1. Add entry to `ENEMY_TYPES` with properties: w, h, hp, speedMul, color, dark, score, minStage, optional weave
2. Reference in `spawnEnemy()` where difficulty filters available types
3. Add boss design to `BOSSES` array with name and color scheme
4. Boss is automatically selected by stage via `spawnBoss()`

### To modify gameplay mechanics

- **Weapon system** - Edit weapon properties in `WEAPON_TYPES` or update in player object
- **Collision detection** - Modify rect/circle collision logic in collision detection sections
- **Combo system** - Adjust multiplier gain/decay in `updateCombo()` or relevant sections
- **Smart bomb** - Modify damage radius and effect in bomb trigger section
- **Scoring** - Edit score values in `ENEMY_TYPES` or scoring calculations

### To adjust audio

- **Synthesizer functions** - `playTone()` and `playNoise()` use Web Audio API
- **Sound parameters** - Duration, frequency, envelope, filter settings are configurable
- **Audio triggers** - Each game event (hit, shot, bomb, combo) calls a playback function

## Performance Considerations

- **Canvas rendering** - All drawing happens on a single 2D canvas; no DOM manipulation during game loop
- **Object pooling** - Bullets and enemies are created/destroyed as needed
- **Quality adaptation** - Quality level 2 disables particle glow, reduces particle count
- **Mobile optimization** - Touch controls require no external library; CSS media queries handle responsive UI

## Testing the Game

To test locally:
1. Open `index.html` in a web browser
2. Use difficulty buttons to select Easy/Normal/Hard
3. Click "Start Game"
4. Keyboard: Arrows/WASD to move, Space to fire, X/Shift to bomb, P/Esc to pause
5. Mouse: Move cursor to steer, click to fire
6. Touch: Drag in left zone to move, tap FIRE/BOMB buttons

## Common Modifications

- **Difficulty balance** - Adjust `DIFFICULTIES` and `BOSSES` hpMul values
- **Spawn rate** - Edit `spawnBase` and `spawnMin` in `DIFFICULTIES`
- **Visual style** - Add new theme to `THEMES` or modify existing colors
- **Enemy behavior** - Add new properties to `ENEMY_TYPES`, implement in `spawnEnemy()` and enemy update logic
- **High difficulty content** - Set higher `minStage` in `ENEMY_TYPES` to gate enemies to later stages

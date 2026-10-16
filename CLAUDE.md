# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single, self-contained HTML file. There are no external dependencies, build tools, or separate asset files—all CSS, JavaScript, and game logic are embedded in `index.html`.

## Running the Game

**To test the game locally:**
```bash
# Option 1: Open directly in a browser
open index.html

# Option 2: Use a simple HTTP server (Python)
python3 -m http.server 8000
# Then navigate to http://localhost:8000
```

The game runs in any modern browser with:
- HTML5 Canvas support
- WebAudio API (for synthesized sound)
- LocalStorage (for score persistence)
- Touch and keyboard input

## Code Architecture

The entire game is contained within a single IIFE (`(function() { 'use strict'; })()`) starting at line 422. This approach provides scope isolation and allows the entire application state to be self-contained.

### Key Architecture Sections (in order within the script):

1. **DOM Element References** (lines 426–461)
   - Caches all HTML elements the game interacts with (canvas, HUD elements, buttons, etc.)

2. **Storage Keys & Constants** (lines 463–500)
   - Game configuration: difficulty settings, theme definitions, enemy types, boss data
   - `DIFFICULTIES`: Tuning for spawn rates, enemy behavior, lives, and boss HP scaling
   - `THEMES`: Visual themes with sky colors and lighting
   - `ENEMY_TYPES`: Enemy stats (HP, speed, color, scoring)
   - `BOSSES`: Boss definitions (6 unique bosses with different visual and gameplay characteristics)

3. **Performance Adaptation** (lines 473–483)
   - `quality` variable: Adaptive graphics quality (0=full, 1=medium, 2=low)
   - Tracks frame times over 45 frames and adjusts graphics quality dynamically

4. **Audio System** (lines 542–600+)
   - WebAudio API synthesis with no external audio files
   - `ensureAudio()`: Initializes AudioContext lazily (required for browser autoplay policies)
   - `beep()`: Generates sound effects with frequency sweep, filtering, and panning
   - Uses oscillators, gain nodes, and filters; all synthesis parameters are in the code

5. **Game State & Mechanics** (distributed throughout)
   - Player (ship): position, health (hearts), weapons, bomb count
   - Enemies: spawning system, movement patterns, collision detection
   - Bullets: player bullets and enemy fire
   - Scoring: combo multiplier system
   - Stages: kill-based progression (18 kills per stage trigger boss)

6. **Canvas Rendering & Sprites**
   - Procedural drawing: All sprites are drawn with Canvas APIs (paths, fills, strokes)
   - No image assets; shapes are created at draw time
   - Layering: Background → enemies → player → bullets → HUD → effects (vignette, flash)

7. **Input Handling**
   - Keyboard: Arrow keys or WASD to move, Space to fire, E for bombs
   - Touch: Virtual joystick (left side) and fire/bomb buttons (right side)
   - Support for accelerometer-based aiming on mobile devices

8. **UI Overlays**
   - Start/game-over screen: Difficulty selection, score display, best-score tracking
   - HUD during gameplay: Lives, bombs, score, multiplier, weapon info, boss health bar
   - Accessibility: Screen reader announcements and reduced-motion support

### Important Game Constants

- **Stages**: Progression is kill-based (`KILLS_PER_STAGE = 18`)
- **Weapon Levels**: Increase with consecutive hits on the same enemy; higher levels increase fire rate
- **Combo Multiplier**: Score multiplier builds up with consecutive kills; resets when player is hit
- **Difficulty Scaling**: Changes spawn frequency, enemy speed, shoot probability, health, and bomb count

## Development Notes

### Where to Find Things

- **Game loop**: Main `requestAnimationFrame` setup and update function
- **Player logic**: Ship movement, weapon firing, damage handling, bomb mechanics
- **Enemy spawning**: Spawn queue logic with difficulty-based timing
- **Collision detection**: AABB (axis-aligned bounding box) checks for bullets, enemies, and pickups
- **Boss battles**: Boss data definitions at line ~508; boss behavior in the update loop
- **Visual themes**: `THEMES` array at line ~493 controls sky colors and lighting per stage
- **Persistence**: LocalStorage keys defined at lines ~463–466; updates on game-over

### Modifying Game Balance

To adjust difficulty or game feel:

1. **Difficulty Parameters**: Edit `DIFFICULTIES` object (easy/normal/hard)
   - `spawnBase` / `spawnMin`: Controls enemy spawn frequency
   - `enemySpeed`: Multiplier for enemy movement speed
   - `enemyShootChance`: Probability that enemies fire per frame
   - `bossHpMul`: Multiplier for boss health

2. **Game Mechanics**:
   - `KILLS_PER_STAGE`: Enemies needed to progress and trigger boss
   - Enemy type definitions in `ENEMY_TYPES`: Adjust HP, speed, color, score values
   - Boss data in `BOSSES`: Visual colors and HP scaling factors

3. **Weapon/Player**: Weapon fire rates, damage, and player speed are inline in the game loop

### Canvas Rendering Pipeline

All drawing happens in the main render function. The pipeline is:
1. Clear canvas with gradient background
2. Draw background stars/clouds
3. Draw enemies (procedural ship outlines)
4. Draw player (triangle with wings)
5. Draw bullets (small squares/circles)
6. Draw HUD text and UI overlays
7. Apply post-render effects (vignette pulse on damage, flash on hit/bomb)

**No image assets are used.** All visuals are procedural, making the game file completely self-contained.

### Audio System Details

- Uses `AudioContext` from the WebAudio API
- Lazy initialization: Audio only starts when first sound is triggered (browser policy requirement)
- All sounds are generated from simple oscillator configurations—no audio files
- Supports muting (persisted in LocalStorage)
- Spatial audio: Panning based on bullet/explosion position (stereo width)

### Accessibility Features

- Screen reader support: `srLive` element announces important game state changes
- Reduced motion mode: Can be toggled per user preference; disables animations and visual effects when enabled
- Touch controls: Virtual joystick and buttons for mobile/touch devices
- High contrast UI: Text uses shadows and bright colors for readability

## Common Development Tasks

### Adding a New Enemy Type

1. Add a new entry to `ENEMY_TYPES` object with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, and optional `weave`
2. Update the enemy spawn logic to reference the new type
3. Procedural ship drawing in the render function will style it based on color properties

### Adding a New Boss

1. Add a new object to the `BOSSES` array with: `name`, `hullTop`, `hullBottom`, `wing`, `core` (colors), and `hpMul`
2. Boss rendering uses these colors; no new drawing code is needed
3. Boss behavior (movement, attack patterns) can be tuned in the main game loop

### Adjusting Visual Themes

Edit the `THEMES` array to add new color schemes. Each theme specifies:
- `sky`: Array of 3 colors for gradient background
- `cloud`: RGB color for cloud/particle rendering
- `sun`: RGB color for background light source
- `name`: Display name for the theme

### Testing on Mobile

1. Run a local HTTP server: `python3 -m http.server 8000`
2. Access from a mobile device on the same network: `http://<your-ip>:8000`
3. Touch controls activate automatically; accelerometer support depends on device and browser
4. Test both portrait and landscape orientations

## Performance Considerations

- **Adaptive Quality**: The game automatically reduces particle/effect rendering if frame rate drops below 40 FPS
- **Canvas Optimization**: Batch rendering and avoid excessive context state changes
- **Audio**: WebAudio synthesis is efficient; tone generation is deferred until play is triggered
- **Mobile**: Touch input is debounced; joystick smoothing reduces jitter

## Game Progression & Mechanics

- **Waves**: Enemies spawn continuously; every 18 kills triggers the next stage and a boss battle
- **Weapon Progression**: Consecutive hits on the same enemy increase the weapon level (max 5), boosting fire rate
- **Combo Multiplier**: Score multiplier increases with each consecutive kill; resets when hit
- **Power-ups**: Defeated enemies may drop lives or bombs (dropped based on random chance)
- **Boss Battles**: A single boss spawns after kill threshold; defeat it to advance to the next stage
- **Persist**: Best score and difficulty preference are saved to LocalStorage

## File Structure

```
index.html          # The entire game: HTML + CSS + JavaScript (1669 lines)
README.md           # Project metadata
.git/               # Git history
```

That's it—a single-file game with no external dependencies.

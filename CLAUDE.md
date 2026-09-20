# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game, fully contained in a single HTML file. It's a browser-based game with no external dependencies, featuring responsive controls, synthesized audio, and multiple difficulty levels.

## Architecture

The entire game is self-contained in `index.html` (~1,670 lines), organized into three sections:
- **HTML structure** (~420 lines): Game canvas, UI overlays, HUD elements, controls
- **CSS styling** (~300 lines): Responsive layout, animations, themes
- **JavaScript engine** (~950 lines): Game logic, rendering, and input handling

### Key Design Patterns

**State Management**: Global variables track game state (player, enemies, bullets, score, lives, etc.). The game loop (`gameStep()`) updates state each frame and calls render functions.

**Entity System**: Entities (player, enemies, bullets, particles, powerups, boss) are stored in arrays and updated/rendered per-frame. Each entity is a simple object with position (x, y), dimensions, and type-specific properties.

**Rendering Pipeline**:
1. Canvas setup (clear, apply effects like screen shake)
2. Render background and theme
3. Render all entities (sorted by layer: background → enemies → bullets → UI)
4. Render HUD (score, lives, weapon info)
5. Render overlays (pause screen, game over, stage banners)

**Audio System**: Uses WebAudio API with synthesized sound effects (no external audio files). The `beep()` function creates pitched sounds; `noiseBurst()` creates noise-based effects. All audio is routed through a master gain node with compression.

## Commands & Development

Since this is a single HTML file with no build process, there are no standard build/test commands. Development workflow:

- **Local testing**: Open `index.html` directly in a browser (works with `file://` protocol)
- **Live server (recommended)**: Use any local server to avoid CORS issues if adding external resources
  ```bash
  # Python 3
  python -m http.server 8000
  # Or Node.js
  npx http-server
  ```
  Then visit `http://localhost:8000`

- **No bundling/transpiling needed**: The code uses vanilla JS (ES2019 features like arrow functions, template literals, destructuring)

## Code Structure

### Configuration Objects

- **DIFFICULTIES**: Defines spawn rates, enemy speed, lives, bombs, and boss HP multipliers for easy/normal/hard
- **ENEMY_TYPES**: Properties for grunt/interceptor/tank (dimensions, HP, speed, color, scoring)
- **BOSSES**: Six boss variants with visual themes and HP multipliers
- **THEMES**: Six visual themes (sky gradients, cloud colors) applied per stage

### State Variables

Main game state:
- `player`: Player object with x, y, weapon, level, health (invuln timer)
- `bullets`: Array of player projectiles
- `enemies`: Array of active enemies
- `enemyBullets`: Array of enemy projectiles
- `particles`: Visual effects (explosions, impact flashes)
- `powerUps`: Dropped items (weapon upgrades)
- `boss`: Boss object (null when not active)

Game flow:
- `running`: Game is active (not paused, not game-over)
- `paused`: Game is paused by user
- `stage`: Current stage (1-6, cycles through themes)
- `kills` / `stageKills`: Enemy count for stage progression
- `chain` / `multiplier`: Combo system (higher multiplier for consecutive kills)
- `score` / `lives` / `bombCount`: HUD values

### Key Functions

**Game Loop**: `gameStep(dt)` - Called every frame, updates all entities and detects collisions

**Spawning**: 
- `spawnEnemy()` - Creates random enemy at top of screen
- `spawnBoss()` - Called when kills reach KILLS_PER_STAGE
- Different waves use different enemy compositions by stage number

**Collision Detection**: 
- Per-entity bounding box checks in `gameStep()`
- Boss pattern checking (does enemy/bullet hit boss?)
- Player vs. powerup detection for upgrades

**Rendering**: 
- `drawPlane()` - Renders player and boss with wings, hull, core
- `drawEnemy()` - Renders enemy type-specific shapes
- `drawBullet()` - Renders player and enemy projectiles
- Canvas transforms used for rotation and effects

**Input Handling**:
- Keyboard: WASD or Arrow keys for movement, Space to fire
- Mouse: Click-to-move targeting
- Touch: Virtual joystick + fire/bomb buttons
- Pause toggle: P key or button

### Audio System

The synth-based approach:
- `beep(freq, dur, type, vol, opts)` - Oscillator with optional frequency sweep, filter, panning
- `noiseBurst(dur, vol, opts)` - Procedural noise with filter
- `sfx` object maps sound names (shoot, explode, hit, etc.) to function calls
- `masterGain` controls overall volume; all sounds route through it

Audio is only initialized on first user interaction (mobile requirement).

### Responsive Design & Accessibility

- **Canvas scaling**: Fixed internal resolution (480×720) scales to fit window with `resize()` function
- **Touch controls**: Virtual joystick only shown on mobile (detected by touch events)
- **Screen reader**: `announce()` function pushes text to hidden `srLive` region for accessibility
- **Reduced motion**: Respects `prefers-reduced-motion` media query; disables screen shake and flash effects when enabled
- **Keyboard navigation**: Start/pause buttons are keyboard-accessible; difficulty buttons toggle via click

### Common Edits

**Tweaking Difficulty**: Adjust DIFFICULTIES object (spawn rates, enemy speed, lives, boss HP)

**Adding Enemy Types**: Add entry to ENEMY_TYPES with visual properties (color, dimensions, HP) and min stage number

**New Bosses**: Add to BOSSES array with unique hull/wing/core colors and HP multiplier

**Visual Themes**: Add to THEMES array with sky gradient colors and name; stages cycle through them

**Weapon Balance**: Weapon properties (cooldown, damage, spread) are hardcoded in the firing functions within `gameStep()`; search for `player.weapon` to find weapon logic

## Performance Notes

- Adaptive quality system (`quality` variable) reduces particle count if frame time exceeds 1/40 second
- Frame time tracking in `frameTimes` array (45-frame rolling average)
- Canvas rendering is the main bottleneck; particle count and enemy count scale with performance
- No garbage collection pauses from entity pooling (new objects created/destroyed normally, but at limited rates per frame)

## Browser Compatibility

Works in all modern browsers supporting:
- ES2019 (arrow functions, destructuring, template literals)
- Canvas 2D API
- WebAudio API (fallback: game runs without audio if unavailable)
- Local Storage (for score, mute state, difficulty, motion preference)
- Touch Events (for mobile controls)

Mobile: Tested on iOS Safari and Android Chrome. Viewport meta tag ensures proper scaling.

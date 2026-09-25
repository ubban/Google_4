# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The entire codebase (HTML, CSS, and JavaScript) is contained in `index.html`.

## Quick Start

### Running the Game

Since this is a static HTML file, you can run it directly:

```bash
# Option 1: Using Python's built-in HTTP server
python -m http.server 8000

# Then open http://localhost:8000 in your browser
```

Or simply open `index.html` directly in a browser.

### Editing the Code

All game code is in `index.html`. Edit in your preferred text editor or IDE. The file structure is:
- `<style>` section: All CSS styling (responsive design with clamp units)
- `<body>` section: HTML markup for the canvas and UI overlays
- `<script>` section: All JavaScript game logic

## Game Architecture

### Core Game Loop

The game runs on a `requestAnimationFrame` loop that:
1. Calculates delta time (dt) between frames
2. Updates game state via `update(dt)` when running and not paused
3. Renders via `draw()` using canvas
4. Tracks performance with `trackPerf(dt)`

Key entry point: `loop(now)` function (main animation frame callback)

### Game State & Initialization

- **Game initialization**: `resetGame()` - sets up a fresh game state
- **Main state variables**: 
  - `running`, `paused` - game state flags
  - `player` - player ship object with position, velocity, hp, etc.
  - `enemies` - array of active enemy objects
  - `boss` - current boss (if active)
  - `bullets`, `enemyBullets` - projectiles
  - `powerUps` - collectible power-ups
  - `particles` - visual effects
  - `score`, `mult` - scoring system
  - `weaponLvl` - current weapon upgrade level

### Key Game Systems

#### Weapon System

Multiple weapon types with different firing patterns:
- `fireColumns()` - vertical column shots
- `fireRadial()` - spread in all directions
- `fireDoubleRing()` - two concentric rings
- `fireHomingSeeded()` - self-aiming projectiles
- `fireSpiralBurst()` - rotating spiral pattern
- `fireAimedSpread()` - fan of shots
- `fireCross()` - crosshair pattern

Weapons upgrade through power-ups; `fireWeapon()` dispatches to the active weapon type.

#### Enemy System

Enemy types defined in `ENEMY_TYPES` object:
- Basic enemies with different speeds and behaviors
- Tank enemies with armor/health bars
- Boss fights with pattern-based attacks
- Wave-based spawning via `spawnWave()`

Enemy firing patterns and movement are deterministic based on `frame` counter.

#### Collision Detection

Uses simple AABB (axis-aligned bounding box) checks between:
- Player and enemy bullets → damage player
- Player bullets and enemies → damage/kill enemies
- Player and power-ups → apply effects
- Player and enemies → damage both

#### Rendering Pipeline

1. `draw()` - main draw function
2. Background: `drawClouds()` - parallax scrolling clouds
3. Game objects: player, enemies, bullets, power-ups
4. HUD overlay: score, weapon level, health, bombs
5. UI overlays: pause menu, game over screen, boss health bar

Canvas is resized responsively with `fitCanvas()`.

### Input Handling

#### Keyboard Controls
- Arrow keys or WASD for movement
- Space for fire
- E for bomb

#### Touch/Mobile Controls
- Left zone (55% width) for movement joystick
- Right zone (45% width) with Fire and Bomb buttons
- Joystick handlers: `joystickStart()`, `joystickMove()`, `joystickEnd()`

#### UI Controls
- Mute button toggles audio
- Motion button toggles reduced-motion preference (accessibility)
- Pause button pauses/resumes game
- Difficulty buttons select challenge level

### Audio System

Audio is lazy-loaded via `ensureAudio()` to avoid autoplay restrictions. Uses Web Audio API for:
- `beep(freq, dur)` - simple tone generation
- Sound effects for shots, hits, explosions
- Mute toggle functionality

### Scoring & Progression

- **Score multiplier** (`mult`) increases with consecutive hits, resets on miss
- **Combo system** displays "COMBO x2", "COMBO x3", etc. on screen
- **Best score** persisted to `localStorage` as `'sfBest'`
- **Waves progress** through enemies → boss → next wave

## Development Workflow

### Adding New Features

**New weapon type**: 
1. Add firing function (e.g., `fireNewType()`)
2. Register in weapon cycle/UI
3. Bullets inherit from existing bullet system

**New enemy type**:
1. Add to `ENEMY_TYPES` config object
2. Implement movement/firing in main `update()` loop
3. Collision detection is automatic via bullet/enemy arrays

**Visual effects**:
1. Add particle class and spawn in `update()`
2. Render in `draw()` loop with life/alpha fade

### Performance Considerations

- Canvas resolution scales with `quality` setting (affects glow effects)
- Frame counter (`frame`) used for deterministic animation timing
- Performance tracked with `trackPerf()` to monitor dt spikes
- Reduced motion mode disables non-essential animations (via CSS)

### Accessibility Features

- Reduced motion support (checked via `prefers-reduced-motion` CSS media query)
- Screen reader live region (`#srLive`) for status announcements
- Semantic HTML for overlay screens
- Touch control fallback for mobile users

## Code Patterns

### Object Patterns

Game objects (player, enemies, bullets) follow simple patterns:
```javascript
// Example enemy object structure
{ x, y, w, h, vx, vy, hp, color, type, ... }
```

No classes used; objects are plain data with functional updates.

### Array Processing

Heavy use of `.forEach()` for game object updates and rendering. Dead objects are filtered/removed as needed.

### Delta Time Updates

All movement and timing uses `dt` parameter: `x += vx * dt`, not frame-based increments.

## Troubleshooting Common Tasks

### Bug: Weapon not firing
Check `fireWeapon()` logic and that `weaponLvl` matches a registered weapon type.

### Bug: Collision not registering
Verify AABB math in collision detection and that object width/height are set correctly.

### Bug: Audio not working
Check `ensureAudio()` is called on first user interaction (click/touch).

### Performance slow on low-end devices
Reduce `quality` setting to disable glow effects or lower game difficulty to reduce object count.

## Testing the Game

There are no automated tests. Test manually:
1. Run game locally via HTTP server
2. Test all weapon types and difficulty levels
3. Verify touch controls on mobile
4. Check audio and reduced-motion toggles work
5. Confirm score persistence across sessions

## File Structure

```
Google_4/
├── index.html          # Complete game (HTML + CSS + JS)
├── README.md           # Project title only
└── .git/               # Git history
```

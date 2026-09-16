# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single-file HTML/CSS/JavaScript application. The entire game runs in `index.html` with no external build process, bundler, or dependencies.

### Key Features
- Canvas-based 2D rendering with dynamic sky backgrounds
- WebAudio API synthesis for all sound effects (no audio files)
- Multiple enemy types with distinct behavior and attack patterns
- Six unique boss encounters with varied attack libraries
- Three difficulty levels with dynamic spawning
- Touch, keyboard, and mouse input support
- Adaptive performance quality (reduces particles/effects under load)
- Local storage for best scores and user preferences
- Accessibility features (reduced motion support, screen reader announcements)

## Running the Game

Since this is a single HTML file with no server requirements:

1. **Open in browser**: Simply open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge)
2. **Local server** (optional, for cache busting during development):
   ```bash
   # Using Python 3
   python -m http.server 8000
   # Then visit http://localhost:8000
   ```

## Code Architecture

The entire game logic is in a single IIFE (Immediately Invoked Function Expression) starting at line 423. The code is organized into these logical sections:

### 1. **UI Elements & Constants** (lines 426–527)
- DOM element caching
- Storage key constants (scores, settings)
- Difficulty presets with spawn rates and enemy stats
- Enemy type definitions with HP, speed, score values
- Boss designs (6 bosses with unique color schemes)
- Thematic sky gradients and color palettes

### 2. **Audio System** (lines 542–676)
- **WebAudio synthesis**: All sounds are procedurally generated using oscillators and noise
- `ensureAudio()`: Lazy-initializes AudioContext (required for browser autoplay policies)
- `beep()`: Sine/square/sawtooth oscillator with optional frequency sweep and filter
- `noiseBurst()`: Procedural white noise with envelope
- `sfx` object: Named sound effect functions (shoot, explode, powerup, etc.)
- **Design pattern**: Each SFX function is self-contained; modify the beep/noiseBurst parameters to alter sound design

### 3. **Canvas & Input Handling** (lines 752–862)
- Canvas sizing: Fixed 480×720 internal resolution, scales to fit window
- **Keyboard input**: Arrow keys, WASD, Space (fire), X/Shift (bomb), P/Esc (pause)
- **Touch controls**: Virtual joystick on left half, FIRE/BOMB buttons on right
- **Mouse steering**: Player smoothly glides toward cursor while hovering canvas
- Input systems are blended: keyboard/joystick takes priority over mouse

### 4. **Game State** (lines 863–910)
- Player object: position, speed, weapon (vulcan/laser), level, invulnerability timer
- Arrays: bullets, enemies, enemyBullets, particles, powerUps, boss
- Scoring: base score, multiplier (increases with kill chains), best score tracking
- Difficulty scaling happens at runtime: spawn timers decrease, enemy count increases

### 5. **Rendering** (lines 913–1637)
- **drawPlane()**: Procedurally draws both player and enemy ships (different colors)
- **drawBossShip()**: Boss ships with animated wings and telegraphing glow
- **draw()**: Main render loop—sky gradient, clouds, all game objects, particles, HUD elements (muted by main loop)
- **Adaptive quality**: `quality` variable (0=full, 1=medium, 2=low) reduces shadow blur and particle counts
- Sky background uses layered clouds at different speeds for parallax effect

### 6. **Game Logic - Update Loop** (lines 1148–1302)
- `update(dt)`: Called each frame with delta time
- **Movement**: Player handles multiple input sources (keyboard, joystick, mouse)
- **Collision detection**: `rectsOverlap()` checks AABB collisions for bullets, enemies, power-ups
- **Enemy spawning**: Adaptive timer decreases based on game time and difficulty
- **Scoring & chains**: Kill combos extend multiplier; breaks after 2.2 seconds without a kill
- **Boss logic**: Separate `updateBoss()` handles telegraphing and attack patterns

### 7. **Boss Attack Patterns** (lines 1324–1409)
- Six attack pattern sets (one per boss type), each with 3 attack phases
- **Pattern primitives**: 
  - `fireRadial()`: Bullets in all directions
  - `fireAimedSpread()`: Spread aimed toward player
  - `fireColumns()`: Vertical columns across screen
  - `fireDoubleRing()`: Two concentric circles at different speeds
  - `fireHomingSeeded()`: Bullets aimed at player with slight spread
- Pattern progression is cyclic; each attack telegraphs for ~22 frames before execution
- Boss extends pattern index each attack; difficulty changes boss HP multiplier

### 8. **Game Flow** (lines 1411–1504)
- `resetGame()`: Initializes game state, reads difficulty settings
- `update()`: Main game tick when `running && !paused`
- `endGame()`: Triggered when lives ≤ 0; stores best score, shows overlay
- `togglePause()`: Pauses update loop; overlay shown/hidden to match
- Bombs (`useBomb()`): Clear all enemy bullets, destroy all enemies, damage boss, increase score

## State Management & Game Loop

- **Main loop**: `requestAnimationFrame(loop)` drives updates and rendering
- **Delta time**: Clamped to 0.05s max to prevent large frame skips
- **Paused state**: When paused, only overlay renders; `update()` is skipped
- **Frame counter**: Incremented each tick; used for animations, spawn timers, and effect timing

## Adding New Content

### New Enemy Types
1. Add entry to `ENEMY_TYPES` object (around line 502) with stats: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Update `spawnEnemy()` (line 1022) if new spawning logic needed
3. Add explosion SFX to `EXPLODE_SFX` object (line 678) if unique sound desired

### New Boss
1. Add entry to `BOSSES` array (around line 508): `name`, `hullTop`, `hullBottom`, `wing`, `core`, `hpMul`
2. Add 3-attack pattern array to `BOSS_PATTERNS` (line 1372)—use helper functions or write custom bullet-fire logic
3. Boss sprite is drawn by `drawBossShip()`; adjust hull/wing colors to match design

### New Theme (Sky)
1. Add object to `THEMES` array (line 493): `sky` (3-stop gradient), `cloud`, `sun` (RGB strings), `name`
2. Themes cycle based on stage: `(stage - 1) % THEMES.length`

### New Sound Effect
1. Create function in `sfx` object using `beep()` and `noiseBurst()` primitives
2. Call it from game logic with `sfx.mySound()`
3. Mute check (`if (muted || !audioCtx) return;`) is handled inside beep/noiseBurst

## Performance Tuning

- **Quality levels**: When avg frame time > 1/40s, quality increments (reducing glows/particle counts)
- **Particle limits**: `spawnExplosion()` (line 1073) scales particle count by quality level
- **Frame time tracking**: `frameTimes` array stores last 45 frames; uses sliding window average

## Key Design Decisions

- **Single file**: Simplifies deployment and modification; no build step required
- **Procedural graphics**: All visuals drawn with Canvas 2D primitives (no sprite sheets)
- **WebAudio synthesis**: Guarantees audio works across devices; no server needed for assets
- **Local storage**: No server dependency; scores/settings persist locally
- **Adaptive quality**: Game remains playable on low-end devices by reducing visual complexity
- **Input blending**: Keyboard, joystick, and mouse coexist without conflicts (priority-based)

## Accessibility

- **Reduced motion**: `prefers-reduced-motion` media query respected; animations disabled when active
- **Screen reader**: `srLive` ARIA live region announces important events (stage clear, damage, etc.)
- **High contrast**: All UI uses high-contrast colors against dark background
- **Touch support**: Virtual joystick and buttons sized for mobile use

## Browser Compatibility

Requires:
- Canvas 2D context
- WebAudio API (AudioContext)
- Modern CSS (Grid, flexbox, safe-area-inset)
- localStorage API
- requestAnimationFrame

Tested and working on Chrome, Firefox, Safari (iOS 13+), and Edge.

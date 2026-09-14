# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game implemented as a single-file HTML application. The entire game logic, rendering, and audio synthesis runs client-side in the browser with no external dependencies.

## Key Technology Choices

- **Canvas API**: Used for 2D rendering of all game elements (player, enemies, bullets, effects)
- **Web Audio API**: Procedural sound generation (no audio files) using oscillators and filters
- **No dependencies**: Completely self-contained in `index.html` with embedded CSS and JavaScript
- **Local Storage**: Persists high scores and user preferences (mute state, difficulty, reduced motion)
- **Responsive design**: Scales to mobile and desktop; uses viewport-relative units for UI

## Architecture Overview

The codebase is organized into logical sections within the single HTML file:

### 1. Game Constants & Configuration (Lines 463-516)
- **DIFFICULTIES**: Three difficulty presets with spawn rates, enemy stats, lives, and boss HP multipliers
- **THEMES**: Six visual themes (day, sunset, night, storm, arctic, volcanic) with sky gradients and cloud colors
- **ENEMY_TYPES**: Three enemy variants (grunt, interceptor, tank) with different stats and behavior
- **BOSSES**: Six boss designs with unique color schemes and attack patterns

### 2. Audio System (Lines 545-677)
- **ensureAudio()**: Lazy initializes Web Audio context on user interaction (required for browser autoplay policy)
- **beep()**: Generates pitched tones with optional frequency sweep, filter, and panning
- **noiseBurst()**: Generates noise bursts for impacts and explosions
- **sfx object**: Sound effect library using procedural synthesis
- All audio is synthesized; no external files needed

### 3. Input Handling (Lines 768-835)
- **Keyboard**: Arrow keys, WASD, Space, X, Shift, P (pause)
- **Mouse**: Move to aim, click to fire
- **Touch**: Left side of screen is movement zone, right side has FIRE and BOMB buttons
- **Joystick emulation**: On touch, displays a circular joystick base and stick visual feedback

### 4. Game State & Rendering (Lines 753-1668)
The game loop runs via `requestAnimationFrame` and calls:
- **update(dt)**: Game logic, collision detection, entity movement (1148 lines)
- **draw()**: Canvas rendering of all entities, HUD, effects (1520-1637)
- **loop(now)**: Main frame loop that manages delta time and gates update/draw

Key state variables (initialized in `resetGame()`):
- `player`: Position, dimensions, speed, weapon level, invulnerability timer
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`: Entity arrays
- `score`, `multiplier`, `chain`: Scoring mechanics
- `lives`, `stage`, `stageKills`: Game progression
- `boss`: Current boss instance (null when none active)

### 5. Core Game Logic

**Player Movement & Firing** (1152-1177):
- Smooth interpolation toward mouse/joystick target
- Three weapon types (Vulcan, Laser, Missile) with different spread and timing
- Rapid-fire powerup extends firing rate

**Enemy Spawning & AI** (1022-1042):
- Spawn timer decreases based on difficulty and stage progression
- Enemies weave or pursue depending on type
- Tank enemies have health bars and multiple hit points

**Collision System** (1069):
- `rectsOverlap()`: Axis-aligned bounding box collision for bullets vs enemies
- Simple distance check for player collision with enemies/bullets

**Boss AI & Patterns** (1340-1434):
- Six boss designs with unique attack patterns
- Patterns cycle through BOSS_PATTERNS (defined inline in update function)
- Telegraph phase before attacks; visual warning with sound effect

**Scoring & Combo** (1087-1102):
- Base points per enemy type; multiplied by current multiplier
- Combo chain extends if consecutive hits within time window
- Chain multiplier maxes at 4.0x after 30+ enemies

**Power-ups** (1058-1067, 1304-1324):
- Life: Restores one health
- Bomb: Adds smart bomb
- Weapon: Upgrades weapon level (max 5)
- Missile pattern: Tracking projectiles that home to boss

### 6. Rendering & Effects

**HUD Elements** (680-730):
- Score, best score, multiplier, weapon name/level, hearts, bombs
- Positioned with responsive spacing using clamp() for font size
- Boss health bar (appears when boss is active)

**Visual Effects**:
- Screen shake magnitude and duration for hits/explosions
- Flash effects (white screen for hit, white for bomb)
- Combo pop-up animation
- Stage banner animation
- Vignette red pulse when at critical health
- Reduced motion toggle disables all animations for accessibility

**Performance Adaptation** (476-483):
- Tracks frame times over 45 frames
- Adjusts quality level (0=full glow, 1=medium, 2=low effects) based on average FPS
- Disables glows, shadows, and particle counts at lower quality

### 7. UI & State Management (Overlay & Menu)

- **Initial screen**: Shows high score, difficulty selector, controls hint
- **Game Over screen**: Shows final stats, "New Best" indicator if applicable
- **Pause menu**: Overlay with resume button
- **Top bar buttons**: Mute, reduced motion toggle, pause
- All state (best score, mute, difficulty, reduced motion) persists to localStorage

## Running & Testing the Game

1. **Run**: Open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge)
   - No build step or dev server needed
   - Game starts immediately

2. **Test Controls**:
   - Desktop: Arrow keys or WASD to move, Space to fire, X/Shift for bomb
   - Mobile: Drag on left side to move, tap FIRE and BOMB buttons
   - Mouse: Move to aim (crosshair), click to fire

3. **Test Difficulty Modes**:
   - Easy: Slower enemies, more lives, weaker boss
   - Normal: Balanced (default)
   - Hard: Faster spawn, stronger enemies, tougher boss

4. **Test Audio**: Click the mute button (🔊) to toggle sound on/off

## Key Editing Guidelines

### When Adding Features
- **Weapon variants**: Define new type in constants near ENEMY_TYPES, add firing logic in fireWeapon()
- **Boss patterns**: Add to BOSS_PATTERNS object inline in update(), define pattern function that fires bullets at boss
- **Enemy types**: Add to ENEMY_TYPES, update EXPLODE_SFX mapping, add spawn logic in spawnEnemy()
- **Visual themes**: Add to THEMES array with sky colors and cloud/sun values

### Code Organization Tips
- All entity arrays (bullets, enemies, etc.) follow the same update pattern: iterate, apply physics, remove if dead
- Boss patterns are stateless; they fire bullets directly into enemyBullets array
- Power-up logic is in applyPowerUp(); update() checks for pickups separately
- Screen shake and flash effects are purely visual; they don't affect gameplay

### Performance Considerations
- Canvas resolution is kept at BASE_W (480) × BASE_H (720) for consistent scaling and performance
- Adaptive quality system disables expensive effects (glows, shadows) on low-end devices
- Particle count is scaled based on quality level in spawnExplosion()
- Audio context is lazy-initialized only on first user interaction

### Accessibility
- Reduced motion toggle in settings (stored in localStorage)
- Screen reader announcements via srLive element for major events
- Keyboard-friendly pause mechanism
- High contrast text shadows for readability

## File Structure

This repository contains only:
- `index.html`: Complete game (HTML + CSS + JavaScript)
- `README.md`: Minimal metadata
- `.git/`: Version control (git commits track major features like "Add Sky Fighter")

## Deployment & Distribution

The game is production-ready as-is:
- Single file can be served by any static host (GitHub Pages, Vercel, etc.)
- No build process or dependencies to install
- Works offline (no external resources loaded)
- Mobile-optimized with responsive layout and touch controls

## Common Tasks

### Add a new sound effect
1. Define a function in the `sfx` object using `beep()` or `noiseBurst()` with appropriate parameters
2. Call `ensureAudio()` before generating sound (already done in relevant handlers)
3. Example: `sfx.newSound = () => beep(440, 0.2, 'sine', 0.1);`

### Adjust game difficulty balance
- Edit `DIFFICULTIES` object to tweak spawn rates (`spawnBase`, `spawnMin`), enemy speed (`enemySpeed`), shoot chance, starting lives, bomb count, boss HP multiplier
- Test each difficulty by starting a game and selecting difficulty before pressing "Start Game"

### Modify enemy or boss behavior
- Enemy AI: Update pursuit/weave logic in the `update()` enemy iteration loop
- Boss patterns: Add or modify functions in `BOSS_PATTERNS` to change attack sequences
- Boss design: Edit `BOSSES` array to change hull/core colors and HP multiplier

### Optimize rendering performance
- Check current quality level via `quality` variable (0=full, 1=medium, 2=low)
- Reduce particle count or disable glows/shadows at quality > 0
- Lower BASE_W/BASE_H if targeting very low-end devices (tradeoff: visual quality)

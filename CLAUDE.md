# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in HTML5. It's a single-file web application with no external dependencies, featuring:
- Canvas-based 2D rendering
- Synthesized audio (WebAudio API)
- Multiple difficulty levels and stages
- Boss encounters with distinct attack patterns
- Responsive touch and keyboard controls
- Persistent score tracking via localStorage

## Architecture & Game Systems

The game is implemented as a monolithic HTML file (~1670 lines) divided into several key systems:

### 1. **Rendering & Canvas**
- Uses `<canvas>` element (gameCanvas) with 2D context for all graphics
- Dynamic sizing with max-width/max-height constraints
- CSS provides styling for overlays, HUD, and UI elements

### 2. **Player & Weapon Systems**
- Player character with health (lives), weapon levels, and bomb count
- Weapon progression system with different attack patterns
- Projectile pooling for efficient memory management
- Configurable weapon parameters (fire rate, spread, damage)

### 3. **Enemy Management**
- Three enemy types: grunt, interceptor, tank (defined in ENEMY_TYPES)
- Enemies spawn dynamically based on difficulty settings
- Each type has distinct visual styling, speed, HP, and scoring
- Enemy bullets and behaviors are difficulty-dependent

### 4. **Boss Encounters**
- Six unique bosses in BOSSES array (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Each boss has custom hull colors, wing patterns, and core styling
- Boss health scales with difficulty multiplier (hpMul)
- Visual health bar with name display

### 5. **Audio System**
- Pure WebAudio API synthesis—no external audio files
- Two main functions: `beep()` for tonal sounds, `noiseBurst()` for noise-based sounds
- Supports frequency sweeps, filtering, stereo panning, and ADSR envelope control
- Master gain and compressor for dynamic range management
- Audio context is lazily initialized on first user interaction

### 6. **Input Handling**
- **Keyboard:** Arrow keys (movement), Space (fire), B (bomb), P (pause)
- **Touch:** Virtual joystick in left zone, fire/bomb buttons in right zone
- Controls are overlaid and toggled based on input type
- Motion preference respects `prefers-reduced-motion` media query

### 7. **Difficulty & Progression**
- Three difficulty levels: easy, normal, hard (DIFFICULTIES object)
- Each level adjusts spawn rate, enemy speed, shoot chance, starting lives/bombs
- KILLS_PER_STAGE (18) triggers advance to next stage
- Boss health multiplier scales per difficulty

### 8. **Persistence & State**
- localStorage keys:
  - `skyfighter.bestScore`: High score
  - `skyfighter.muted`: Audio mute toggle
  - `skyfighter.difficulty`: Selected difficulty
  - `skyfighter.reducedMotion`: Motion preferences
- Score, stage, and stats persist between reloads

### 9. **Performance Adaptation**
- Adaptive quality system tracks frame times and adjusts rendering/logic complexity
- Three quality levels: 0 (full), 1 (medium), 2 (low)
- Monitors 45-frame average; adjusts if avg > 1/40 or < 1/56

### 10. **Visual Feedback**
- Combo pop-ups and stage banners with animations
- Flash effects (hit, bomb) for player damage feedback
- Vignette effect during critical health state
- Dynamic HUD updates for score, health, weapons, multiplier

## Development Workflow

### Running the Game
Open `index.html` in any modern web browser. No build process required.

**Quick testing:**
```bash
# Use a simple HTTP server (Python 3)
python -m http.server 8000
# Then open http://localhost:8000/
```

### Key Constants to Adjust

Located in the initial `<script>` block:

- **DIFFICULTIES**: Spawn rates, enemy speed, lives/bombs, boss HP scaling
- **KILLS_PER_STAGE**: Number of enemy kills to advance stage (currently 18)
- **ENEMY_TYPES**: Enemy appearance, HP, speed multiplier, score values, min stage unlock
- **BOSSES**: Boss colors, core styling, HP multiplier
- **THEMES**: Sky gradient colors and theme names (6 themes, one per stage)
- **STORAGE_*** constants**: localStorage key names

### Common Modifications

**Add a new enemy type:**
1. Add entry to ENEMY_TYPES with w, h, hp, speedMul, color, dark, score, minStage
2. Update spawn logic to include new type based on stage
3. Update rendering to handle new visual properties

**Add a new boss:**
1. Push new object to BOSSES array with hullTop, hullBottom, wing, core, hpMul, name
2. Boss automatically appears in sequence at stage progression
3. Adjust hpMul to balance difficulty

**Adjust difficulty curves:**
1. Modify DIFFICULTIES values (spawnBase, spawnMin, enemySpeed, enemyShootChance)
2. Adjust KILLS_PER_STAGE if pacing feels off

**Change audio:**
- Locate `beep()` and `noiseBurst()` calls
- Adjust frequency, duration, volume, and filter parameters
- Example: weapon fire is higher frequency; hit is sweep down

### Code Navigation

Main sections (search by comment):
- `// ----------` markers divide logical sections
- "Adaptive performance" → Quality adaptation logic
- "Audio (WebAudio synth...)" → Audio system
- "Input handling" → Keyboard/touch input
- "Game loop" → Main requestAnimationFrame loop
- "Rendering" → Canvas drawing functions

### Testing Tips

- **Framerate issues**: Open console, check quality variable. If quality > 0, performance adapting is active.
- **Audio troubleshooting**: Ensure audioCtx is initialized (`ensureAudio()` call). Check browser autoplay policies.
- **Touch controls**: Resize window to verify responsive layout; test on actual mobile device for true touch feedback.
- **Difficulty balance**: Adjust spawnBase and enemySpeed incrementally; test across all difficulty levels.
- **Boss tuning**: Modify boss HP multiplier in BOSSES array to make encounters easier/harder.

## Configuration Files

The game uses localStorage for persistence, not configuration files. Preferences are set via UI (difficulty, mute, motion reduction) and persisted automatically.

## Notes for Future Development

- **No build tools**: The game is intentionally dependency-free. Avoid adding Node.js, bundlers, or external libraries.
- **Canvas rendering**: All graphics are drawn programmatically. Consider performance when adding new visual effects.
- **Audio synthesis**: WebAudio is more CPU-intensive than audio files but avoids asset management; balance carefully.
- **Touch/mobile-first**: UI adapts via CSS `clamp()` and viewport-fit cover for notch support. Test on multiple device sizes.
- **Accessibility**: Screen reader live region (srLive) announces game state changes. Maintain when adding features.

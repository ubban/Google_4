# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file HTML5 Canvas game implementing a Raiden-style vertical shooter. The entire game (logic, rendering, audio, input, UI) is contained in `index.html` (~1670 lines).

### Key Technologies
- **HTML5 Canvas** for rendering
- **Web Audio API** for synthesized sound effects
- **Vanilla JavaScript** (no build process or dependencies)
- **LocalStorage** for persistence (high scores, settings)
- **Touch/Mobile** support with on-screen controls

## Running the Game

The game requires no build process or server:
1. Open `index.html` directly in any modern browser
2. The game runs immediately with full functionality
3. For testing on mobile: open the HTML file or deploy it to a web server

## Game Architecture

### Core Game Loop
- **`loop(now)`** (line 1640): Main animation frame callback. Controls delta time calculation, updates, rendering, and performance tracking.
- **`update(dt)`** (line 1148): Handles player movement, enemy spawning, collision detection, and game state transitions.
- **`draw()`** (line 1520): Renders the sky background, all entities, HUD, and effects.

### Game State Management
Key variables (line 863-869):
- `player` - Player aircraft with position, health, weapon, cooldown
- `enemies`, `enemyBullets`, `bullets`, `powerUps` - Active entities
- `boss` - Current boss (null when not active)
- `score`, `lives`, `stage`, `chain` - Game progress

State management functions:
- **`resetGame()`** (line 875): Initializes game state for new game
- **`endGame()`** (line 1469): Handles game over and best score update
- **`togglePause()`** (line 1491): Pause/resume gameplay

### Difficulty System
Three difficulty modes (line 485-489) stored in localStorage and selectable from UI. Each affects:
- Enemy spawn rates and minimum spawn interval
- Enemy movement speed and shoot chance
- Player starting lives and bomb count
- Boss HP multiplier

### Rendering Pipeline
1. **Sky background** with dynamic gradient based on stage theme (line 1529-1545)
2. **Player aircraft** with optional flame when firing (line 1548-1559)
3. **Enemies** drawn with health bars for tanks (line 1603-1613)
4. **Boss** rendered with animated core and rotating engines (line 970-1020)
5. **Bullets, particles, power-ups** with optional glow effects based on performance tier (line 1579-1636)

### Audio Synthesis
No external audio files. All SFX generated via Web Audio API:
- **`beep(freq, dur, type, vol, opts)`** (line 566): Tone generation with optional sweep/filter/panning
- **`noiseBurst(dur, vol, opts)`** (line 597): Noise generation for explosions
- **`sfx` object** (line 624-676): Named sound effect recipes used throughout the game

Mute toggle in UI persists to localStorage.

### Input Handling
Multi-input priority system (keyboard > joystick > mouse):
- **Keyboard**: Arrow keys or WASD for movement, Space to fire, X/Shift for bomb, P/Esc to pause
- **Mouse**: Hover canvas to steer, click to fire (line 847-861)
- **Touch/Mobile**: Dual-zone controls with virtual joystick in left half, fire/bomb buttons in right half (line 814-834)

### Boss System
Six unique boss designs (line 508-515) with attack pattern progression:
- **VANGUARD**: Radial bursts and aimed spreads
- **BEHEMOTH**: Heavy columns and homing spreads
- **SPECTER**: Teleporting dashes and spirals
- **DREADNOUGHT**: 8-way cross fire and homing missiles
- **WRAITH**: Teleport barrages and expanding rings
- **TITAN**: Heavy columns and relentless homing

Each boss has 3 distinct attack patterns defined in `BOSS_PATTERNS` (line 1372-1409).

Boss spawns after killing 18 enemies (`KILLS_PER_STAGE`, line 491), with a 1.2s delay warning before entering.

## Code Organization in index.html

The script section follows this logical structure (all within IIFE starting line 423):

1. **Setup & Constants** (lines 423-524): DOM references, storage keys, difficulty configs, enemy types, boss designs, themes, initial state restoration
2. **Audio System** (lines 542-676): Web Audio context initialization, synthesis functions, SFX definitions
3. **UI Updates** (lines 680-750): HUD rendering, button state management
4. **Canvas & Input** (lines 752-862): Responsive resizing, keyboard/mouse/touch input handling, joystick mechanics
5. **Game State** (lines 863-912): State variables and `resetGame()`
6. **Rendering Functions** (lines 913-1045): Sprite drawing, particle effects
7. **Game Logic** (lines 1147-1302): `update()` and all collision/scoring logic
8. **Boss System** (lines 1324-1439): Boss attack patterns and AI
9. **Main Loop** (lines 1640-1666): `loop()`, `draw()`, and start button handler

## Common Development Tasks

### Adding a New Enemy Type
1. Add to `ENEMY_TYPES` object (line 502-506) with: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Add explosion SFX name to `EXPLODE_SFX` mapping (line 678) if unique
3. Update `spawnEnemy()` selection logic (line 1026-1030) if desired
4. Define `sfx.explode<TypeName>()` if unique sound needed

### Tweaking Game Balance
- **Difficulty**: Modify `DIFFICULTIES` object (line 485-489)
- **Spawn rates**: Change `spawnBase` and `spawnMin` in difficulties
- **Boss HP**: Adjust `hpMul` values in boss designs (line 508-515)
- **Weapon damage**: Change hardcoded damage (e.g., boss damage at line 1259, bullet pierce at line 1239)
- **Scoring**: Modify `addScore()` call arguments throughout

### Adding Visual Polish
- **Particles**: `spawnExplosion()` at line 1073 controls count and color
- **Screen shake**: `screenShake(magnitude, duration)` at line 727
- **Flash effects**: `flashHit()` and `flashBomb()` at line 698-709
- **Color themes**: Add to `THEMES` array (line 493-500) for new stage visuals

### Adjusting Difficulty Ramp
- Enemy spawn interval decreases over time at line 1201: `difficultyRamp = Math.floor(frame / 300) * 4`
- Modify divisor (300) to change ramp speed

## Performance Considerations

### Adaptive Quality System
Quality tier (0=full, 1=medium, 2=low) adjusts based on average frame time (line 473-482):
- Full quality: All particles, glows, shadows enabled
- Medium: Reduced particle counts
- Low: Minimal visual effects

The game tracks last 45 frame times; auto-downgrades if avg > 1/40th sec, auto-upgrades if < 1/56th sec.

### Mobile Optimization
- Touch controls have `passive: false` on movement zone (line 819) to prevent default scroll
- Reduced motion preference respected (line 520-523) with reduced animation/shake intensity
- Particles and effects scale down on low-quality tier

## Game Storage & Persistence

Uses localStorage for:
- `skyfighter.bestScore` - High score
- `skyfighter.muted` - Audio mute state
- `skyfighter.difficulty` - Selected difficulty
- `skyfighter.reducedMotion` - Motion preference override

All keys defined at line 463-466; loaded at game start (line 517-523).

## Browser Compatibility

- Requires: ES6+ JavaScript, HTML5 Canvas, Web Audio API
- Optional: Touch API for mobile (gracefully degrades to keyboard)
- Tested on: Modern browsers (Chrome, Firefox, Safari, Edge)

## Accessibility Features

- Keyboard controls fully support gameplay (no mouse-only mechanics)
- Screen reader live region announces game events (line 390, `srLive`)
- Respects `prefers-reduced-motion` OS setting (line 520)
- High contrast colors for score/stats display
- Touch button sizing appropriate for mobile (~78px)

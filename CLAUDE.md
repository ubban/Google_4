# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 web application. The game features:
- Canvas-based graphics rendering
- Procedural weapon and enemy spawning
- Stage-based progression with boss battles
- Web Audio API for dynamic sound effects
- Touch and mouse control support
- Difficulty settings (Easy, Normal, Hard)
- High score tracking via localStorage

## Running the Game

Open `index.html` directly in a web browser. No build process or server is required.

**Local Testing:**
```bash
# Using Python 3 for local testing
python -m http.server 8000

# Using Node.js with http-server
npx http-server

# Using VS Code Live Server extension
# Right-click index.html → "Open with Live Server"
```

## Code Architecture

The entire game is implemented in a single **IIFE (Immediately Invoked Function Expression)** starting at line 423. The code follows this organizational structure:

### 1. **Configuration & Constants** (Lines ~480-680)
- `THEMES` — Visual themes for each stage (sky colors, cloud colors, sun colors)
- `ENEMY_TYPES` — Enemy definitions with HP, speed, fire pattern, and rewards
- Game settings stored in localStorage with `STORAGE_*` constants

### 2. **Audio System** (Lines ~545-730)
- `ensureAudio()` — Initializes Web Audio context (call once on first interaction)
- `beep(freq, dur, type, vol, opts)` — Generates sine/square wave tones
- `noiseBurst(dur, vol, opts)` — Generates pink/white noise for SFX
- `sfx` object — Pre-composed sound effects bound to game events
- Audio is muted by user preference and respects reduced motion settings

### 3. **UI/HUD Rendering** (Lines ~680-730)
- `renderHearts(count, max)` — Draws player lives
- `renderBombs(count, max)` — Draws bomb count
- `flashHit()`, `flashBomb()` — Screen flash effects
- `showComboPop(text)`, `showStageBanner(text)` — Overlay UI elements
- Screen shake through `screenShake(mag, dur)` for impact feedback

### 4. **Input Handling** (Lines ~785-870)
- **Keyboard**: Arrow keys move, Space fires, Z uses bomb
- **Mouse**: Click to aim, right-click to bomb
- **Touch/Gamepad**: Virtual joystick on left, fire/bomb buttons on right
- `joystickStart/Move/End()` — Touch input state machine
- `canvasPointFromEvent()` — Converts browser coordinates to canvas coordinates

### 5. **Game State & Logic** (Lines ~875-1340)
- **Player Object**: `player = { x, y, w, h, weapon, weaponLvl, invuln, ... }`
- **Game Variables**: `running`, `paused`, `gameOver`, `lives`, `stage`, `score`, `multiplier`
- **Entity Arrays**: `bullets`, `enemyBullets`, `enemies`, `boss`, `powerUps`, `particles`
- **Update Loop** (`update(dt)`, line 1148): Handles collision detection, entity movement, and spawning logic
- **Weapon System**: Weapons are selected by cycling (VULCAN → LASER → SPREADSHOT → VULCAN...)

### 6. **Rendering** (Lines ~913-1640)
- `drawPlane()` — Draws player and enemy aircraft
- `drawBossShip()` — Draws boss-specific visuals
- `draw()` — Main render function applying sky gradients, clouds, and all entities
- Uses canvas transformations for rotation, scaling, and screen shake

### 7. **Gameplay Systems** (Lines ~1022-1150)
- **Enemy Spawning**: Wave-based with increasing difficulty per stage
- **Collision Detection**: `rectsOverlap()` for hit testing
- **Power-ups**: Life (+1 health), Bomb (+1 bomb), Weapon (upgrades current weapon level)
- **Scoring**: Base score for kills, multiplier increases on consecutive hits without breaking the chain
- **Boss Battles**: Triggered after defeating all enemies in a stage

### 8. **Main Game Loop** (Lines ~1640-1670)
```
requestAnimationFrame(loop) → update(dt) → draw() → repeat
```
Delta time is capped at 50ms to prevent large jumps when tab loses focus.

## Key Constants & Configuration

### Difficulty Settings
```
DIFFICULTY.easy   = { spawnRate: 2.5, enemyHp: 0.8, playerHp: 5 }
DIFFICULTY.normal = { spawnRate: 2.0, enemyHp: 1.0, playerHp: 4 }
DIFFICULTY.hard   = { spawnRate: 1.5, enemyHp: 1.3, playerHp: 3 }
```

### Enemy Types
Each enemy has: `hp`, `speed`, `fire()` function (defines bullet pattern), `reward` (score), and color properties.

### Weapon Progression
Weapons cycle: VULCAN (rapid fire) → LASER (focused beam) → SPREADSHOT (3-way spread) → back to VULCAN.
Each weapon levels up (1-3) by collecting weapon power-ups, affecting fire rate and damage.

### Stage Themes
Stages cycle through 3 visual themes with different sky gradients, cloud colors, and sun colors.

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with HP, speed, fire pattern function, and color
2. The `fire()` function uses `fireRadial()` helper for common patterns
3. Update `spawnEnemy()` if needed to change spawn frequency or type distribution

### Adjusting Difficulty
Modify the `DIFFICULTY` object constants. Affects spawn rate, enemy HP scaling, and player starting lives.

### Adding Visual Effects
Use `screenShake()` for impact, `spawnExplosion()` for particle effects, or canvas save/restore with transforms for rotation/scaling.

### Modifying Audio
Edit the `sfx` object definitions in the audio system section. Use `beep()` for pitched sounds or `noiseBurst()` for noise-based effects.

## Performance Considerations

- Canvas rendering is the main performance bottleneck; quality is reduced (no shadows) when FPS drops below 50
- Particle effects are pre-allocated to avoid garbage collection stalls
- The game runs at a fixed physics timestep (dt capped at 50ms)
- Mobile devices may need reduced visual quality (controlled via `quality` variable)

## Browser Compatibility

- Requires: Canvas 2D context, Web Audio API, localStorage
- Touch input uses pointer events for best device support
- Gracefully degrades on devices without audio support (game continues to play silently)

## Debugging Tips

- Open browser DevTools console to see `announce()` messages via screen reader
- `window.trackPerf()` is available to monitor frame rate
- Modify `quality` variable (0 for fancy, 1 for reduced) to toggle visual effects
- Check localStorage: `localStorage.getItem('skyfighter.bestScore')`

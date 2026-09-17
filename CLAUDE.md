# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a vertical shoot-'em-up game (Raiden-style) built as a single HTML file. It features:
- Player-controlled plane with weapon switching (Vulcan/Laser)
- Progressive stage-based enemy spawning
- Boss battles with multiple distinct patterns
- Scoring multiplier chains
- Three difficulty levels
- Touch/keyboard/mouse controls
- WebAudio synthesis for all sound effects
- Reduced motion accessibility support

## Quick Start

### Running the Game
- Open `index.html` in any modern browser
- No build process, dependencies, or server required
- The game is completely self-contained in one file

### Development
Since this is a single HTML file:
- Edit `index.html` directly in your editor
- Refresh the browser to see changes
- Use browser DevTools (F12) for debugging and performance monitoring

## Codebase Structure

The `index.html` file (1669 lines) contains everything, organized into sections:

### 1. **HTML Structure & Styling** (lines 1-422)
- Canvas-based rendering with responsive sizing
- HUD elements: score, lives, weapon level, bombs, multiplier
- Control buttons: start, pause, mute, reduced-motion toggle, difficulty selector
- Touch controls (joystick) for mobile/tablet
- Screen-reader accessible elements (`#srLive` for announcements)
- Visual effects overlays: vignette, flash, combo pop-ups

### 2. **Game Initialization & State** (lines 423-873)
- DOM element references
- Local storage keys: best score, mute state, difficulty, reduced motion preference
- Performance tracking system (adaptive quality)
- Constants:
  - `DIFFICULTIES`: difficulty parameters (spawn rates, enemy speed, lives, bombs, boss HP multiplier)
  - `KILLS_PER_STAGE`: enemies defeated before boss appears (18)
  - `THEMES`: 6 themed skybox color schemes
  - `ENEMY_TYPES`: grunt, interceptor, tank (HP, speed, score values)
  - `BOSSES`: 6 distinct boss designs with stat variations
- Game state variables: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- State flags: `running`, `paused`, `gameOver`
- Scoring state: `score`, `chain`, `chainTimer`, `multiplier`, `kills`, `stage`, `stageKills`

### 3. **Audio System** (lines 545-677)
- WebAudio synthesis engine (`audioCtx`, `masterGain`)
- Sound functions:
  - `beep(freq, dur, type, vol, opts)`: tonal sounds (weapon fire, power-ups)
  - `noiseBurst(dur, vol, opts)`: noise-based sounds (explosions, impacts)
  - Panning support based on screen position
  - Audio is lazy-initialized on first user interaction

### 4. **Input & Controls** (lines 785-861)
- **Joystick (touch)**: virtual joystick on left, fire/bomb buttons on right
- **Mouse**: crosshair cursor, plane follows pointer while firing with left-click
- **Keyboard**: WASD/Arrows for movement, spacebar to fire, E/B for bomb
- Mouse takes priority over joystick/keyboard (both can coexist)
- Touch events prevent default scrolling

### 5. **Rendering Functions** (lines 680-1520)
- `drawPlane(x, y, w, h, color, darkColor, flame)`: renders player/enemy planes
- `drawBossShip(b)`: renders boss with animated wings and core
- `drawClouds(speed, fillColor, rx, ry)`: parallax cloud background
- `draw()`: main render loop (particles, clouds, plane, enemies, boss, HUD, effects)
- Visual effects: screen shake, flash, combo pop-ups, stage banners

### 6. **Game Mechanics** (lines 875-1491)
- `resetGame()`: initializes/resets all game state
- `update(dt)`: main game loop (movement, collision, firing, enemy spawning)
- `fireWeapon()`: handles player weapon cooldown and firing
- Boss-specific firing patterns:
  - `fireRadial()`: bullets in all directions
  - `fireAimedSpread()`: spread targeting player
  - `fireColumns()`: vertical columns
  - `fireSpiralBurst()`: spinning pattern
  - `fireCross()`: cross pattern
  - `fireDoubleRing()`: concentric rings
  - `fireHomingSeeded()`: homing missiles
- Enemy spawning: `spawnEnemy()`, `spawnBoss()`, enemy AI during `updateBoss()`
- Power-up logic: `maybeSpawnPowerUp()`, `applyPowerUp()`
- Collision detection: `rectsOverlap()`
- Life/bomb/score management: `useBomb()`, `loseLife()`, `addScore()`, `breakChain()`
- Game end: `endGame()`

### 7. **Main Loop & Event Listeners** (lines 1555-1669)
- `window.requestAnimationFrame()` loop drives `update()` and `draw()`
- Difficulty selector, pause button, mute button listeners
- Game start/end overlay interactions

## Key Game Mechanics

### Scoring & Multiplier Chain
- Kill enemies to increment `chain` counter
- `chainTimer` resets on each kill; breaks after 60 frames of no kills
- Multiplier increases from 1x up to 10x as chain grows
- Power-ups can also reset/extend the chain

### Weapons
- **Vulcan**: continuous fire with cooldown, spreads slightly
- **Laser**: single concentrated beam, higher damage, slower fire rate
- Level increases per weapon pickup, affects fire rate and damage
- Players start with Vulcan Level 1

### Difficulty Scaling
- Enemy spawn rate, speed, and accuracy vary
- Boss HP scales with difficulty multiplier (0.8x to 1.3x)
- Lives and bombs adjust (3-4 lives, 2-3 bombs)

### Boss Behavior
- Each of 6 bosses has distinct firing patterns and visual design
- Boss appears after defeating 18 enemies per stage
- Boss HP bars show on-screen
- Different bosses require different strategies

### Accessibility
- Reduced motion preference respected (disabled shake/flash/pulse)
- Screen reader announcements via `srLive`
- Keyboard, mouse, and touch all fully supported
- High contrast dark theme by default

## Performance Considerations

- Adaptive quality system (`quality` variable) adjusts particle counts based on frame time
- Target 60 FPS; degrades gracefully if avg frame time exceeds 25ms
- Canvas is scaled to fit viewport (respects safe areas on mobile)
- No external asset loading (all audio is synthesized, all graphics are drawn)

## Common Editing Tasks

### Add a new enemy type
1. Add entry to `ENEMY_TYPES` constant (line ~502)
2. Add spawn logic in `spawnEnemy()` if special behavior needed
3. Handle collision in `update()` section (around line ~1200)

### Add a new boss
1. Add entry to `BOSSES` array (line ~508)
2. Create a new firing pattern function if needed
3. Update boss spawning logic in `spawnBoss()` (line ~1043)

### Adjust difficulty
Edit `DIFFICULTIES` object (line ~485) with new spawn rates, speeds, lives, etc.

### Add new power-up type
1. Add case in `maybeSpawnPowerUp()` (line ~1058)
2. Handle in `applyPowerUp()` (line ~1304)
3. Add visual/audio feedback

### Tweak weapon balance
- `fireWeapon()` (line ~1124): change cooldown timings
- `drawPlane()` (line ~913): adjust bullet spawn positions or speeds
- Weapon constants are embedded in `fireWeapon()`

## Testing Tips

- **Hard mode** is most challenging; good for testing difficulty scaling
- **Screen recording**: Use browser DevTools to capture and replay
- **Performance**: Monitor frame time in console with the perf tracking variables
- **Accessibility**: Test with keyboard-only, high contrast mode, and screen readers
- **Mobile**: Use DevTools device emulation to test touch controls

## Local Storage
- `skyfighter.bestScore`: best score achieved
- `skyfighter.muted`: audio mute state
- `skyfighter.difficulty`: last selected difficulty
- `skyfighter.reducedMotion`: reduced motion preference

Clear these to reset game state: `localStorage.clear()` in console.

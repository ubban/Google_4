# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file, Raiden-style vertical shoot-'em-up game built with HTML5 Canvas. The entire game logic, rendering, and audio synthesis is contained in `index.html` (1669 lines of JavaScript embedded in HTML).

## Running the Game

The game runs as a standalone web application:
- **Local:** Open `index.html` directly in a web browser
- **Server:** Serve via any HTTP server (e.g., `python -m http.server 8000`) for better performance and to avoid CORS issues
- **Mobile:** Fully responsive; supports both touch controls and keyboard/mouse

Game state (difficulty preference) is stored in browser localStorage under the key `skyfighter.difficulty`.

## Architecture & Key Systems

The codebase is organized as a single HTML file with three main sections:

### 1. **Styling & UI** (lines 10-350)
- CSS variables, animations, responsive layouts
- HUD elements: score, lives, weapons, bombs, boss health bar
- Menu and game-over screen styling
- Touch control zones for mobile
- Visual feedback: flash effects, vignette, combo notifications, stage banners

### 2. **Game Engine & Logic** (lines ~430-1300)
The JavaScript contains:
- **Game State:** `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`; `score`, `lives`, `running`, `paused`, `gameOver`
- **Configuration Objects:**
  - `DIFFICULTY_SETTINGS` - Defines spawn rates, enemy speed, lives, and boss HP multiplier for Easy/Normal/Hard
  - `ENEMY_TYPES` - Tank, fighter, scout (spawn chance, health, movement, bullet patterns)
  - `WEAPON_SYSTEM` - Vulcan, Laser, Spread, Rapid (damage, fire rate, spread angle)
  - `STAGES` - Wave definitions per stage; 10 stages total with increasingly difficult enemy composition
- **Core Game Loop:** `update()` called via `requestAnimationFrame()`, handles all physics, collision, and state transitions
- **Player System:** Position, velocity, current weapon, bombs, invulnerability frames; responds to keyboard (WASD/arrows) and mouse/touch
- **Enemy System:** Spawned in waves per stage; each has position, velocity, shoot pattern, health; destroyed on collision or bullet hit
- **Weapon & Power-Up System:** 4 weapon types with upgrade levels; power-ups spawn after enemy destruction
- **Boss System:** Appears after stage completion; complex attack patterns; health bar displays at top
- **Particle & Vignette Effects:** Explosion particles on enemy/bullet destruction; red vignette flashes on player damage
- **Combo System:** Damage dealt within 0.5s counts as combo; multiplier resets on pause
- **Audio:** Procedural audio synthesis using Web Audio API (oscillators for beeps and bursts)

### 3. **Rendering & Canvas** (lines ~1300-1669)
- `render()` function draws everything to canvas: player, enemies, bullets, particles, power-ups, boss
- Ship sprites drawn procedurally (no image assets)
- Quality setting based on device performance (reduces glow effects on low-end devices)
- Separate rendering for boss with special visual treatment

## Difficulty Levels

Three preset difficulties stored in `DIFFICULTY_SETTINGS`:
- **Easy:** Slower enemy spawns, slower enemy movement, lower enemy shoot chance, 4 lives, 3 bombs, boss has 0.8× HP
- **Normal:** Moderate values, 3 lives, 2 bombs, 1.0× boss HP
- **Hard:** Fast spawns, fast movement, high enemy shoot chance, 3 lives, 2 bombs, 1.3× boss HP

Difficulty preference persists across sessions via localStorage.

## Common Modification Points

If extending the game, these are typical change locations:

1. **Add a new weapon type:** Add entry to `WEAPON_SYSTEM` object with name, damage, fireRate, color, etc.; update weapon cycling logic in `update()` and `render()`
2. **Adjust difficulty balance:** Modify values in `DIFFICULTY_SETTINGS` object
3. **Add new enemy type:** Add to `ENEMY_TYPES` with spawn chance, health, speed, bullet pattern; update wave composition in `STAGES`
4. **Extend stages:** Add new wave arrays to `STAGES` object
5. **Add visual effects:** Create new particle types or animation keyframes in the `<style>` section; emit particles in `update()`
6. **Adjust audio:** Modify synth parameters in `playSound()` or `playBurst()` functions (frequency, duration, waveform)
7. **Change game balance:** Adjust collision radii, player speed, spawn intervals, or enemy bullet patterns in their respective update sections

## Testing & Debugging

- **Performance monitoring:** Device performance is auto-detected; low-end devices disable shadow/glow effects
- **Combo system:** Trigger by rapid enemy destruction; inspect `damage` and `damageCombo` variables to tune combo rate
- **Touch controls:** Simulated by drawing zones in bottom corners; enable with `#moveZone` CSS class
- **Browser console:** Check for any errors; Web Audio context is lazy-initialized on first user interaction

## Notable Design Patterns

- **Procedural rendering:** All game entities are drawn as geometric shapes (no sprite sheets)
- **Web Audio synthesis:** All sounds are synthesized in real-time; no audio files needed
- **Responsive design:** Uses CSS `clamp()` and viewport units for cross-device scaling
- **Lazy state initialization:** Game state reset in `startGame()`; player/enemy objects created on demand

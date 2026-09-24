# Sky Fighter: Game Development Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in vanilla HTML5 Canvas and JavaScript. Players pilot a fighter jet, dodge incoming fire, switch between multiple weapon types, deploy smart bombs, and defeat stage bosses.

### Key Features
- **4 Weapon Types**: Rapid-fire bullets, dual columns, cross pattern, homing seeker with chain explosions
- **Smart Bombs**: Limited-use area-damage ability that clears threats
- **Boss Fights**: Stage bosses with unique attack patterns and health bars
- **Scoring & Multipliers**: Dynamic scoring with combo multipliers for consecutive hits
- **Mobile & Desktop**: Touch and keyboard controls with responsive scaling
- **Particle Effects**: Visual feedback for hits, explosions, and power-ups

---

## Architecture & File Structure

### Single-File Architecture
- **`index.html`** (1,669 lines) — Complete game in one file
  - CSS styling (lines ~10–150)
  - Game logic in vanilla JavaScript (lines ~150–1,669)
  - No external dependencies; runs offline

### Core Game Loop
```
update() → draw() → gameLoop() [requestAnimationFrame]
```

---

## Key Game Systems

### 1. Player Ship (`plane` object)
- Position, health (3 lives), weapon type
- Movement: keyboard (arrow keys) or touch drag
- Firing: weapon-specific patterns
- Methods: `moveToward()`, `update()`, `draw()`

### 2. Enemy System
- **Regular Enemies**: Spawn in waves, follow predictable paths, fire basic bullets
- **Boss Ships**: Larger, multi-phase battles with unique attack patterns
- Collision detection against player, weapons, and bombs
- Methods: `update()`, `draw()`, health tracking

### 3. Weapon System (`fireWeapon()` and variants)
- **Rapid**: Fast single-stream bullets
- **Columns**: Two vertical streams (left/right)
- **Cross**: Five-way spread (up, down, left, right, center)
- **Homing**: Self-targeting seeker rounds with chain explosions (`fireHomingSeeded()`)
- Power-up drops upgrade weapons on pickup

### 4. Projectile System
- **Player Bullets**: Array of bullet objects with position, velocity, and collision
- **Enemy Bullets**: Spawned by enemies; dodge these
- **Smart Bomb Blast**: Radial explosion clearing all enemies within range
- Collision detection: bullets vs enemies, player vs all projectiles

### 5. Scoring & UI
- Score accumulation via enemy kills
- Combo multiplier (resets after 0.5s without hitting anything)
- HUD display: score, health (hearts), bombs, weapon type, multiplier
- Vignette effect when player takes damage (visual feedback)

### 6. Audio
- Lazy initialization: `ensureAudio()` on first user interaction
- Sound effects: weapon fire, explosions, power-ups
- Fallback: silent if audio context unavailable

### 7. Collision Detection
- Rectangle-based hit detection for simplicity
- Layers: enemies, player bullets, enemy bullets, power-ups
- Early exit on first collision for efficiency

---

## Game Mechanics

### Player Controls
| Input | Action |
|-------|--------|
| Arrow keys / WASD | Move |
| Spacebar | Fire |
| 1–4 keys | Switch weapon |
| Click/Touch | Drag to aim and move |
| B | Deploy bomb (if available) |

### Enemy Behavior
- **Wave Spawning**: Enemies spawn in patterns at the top
- **Health**: 1 HP (die on first hit) except bosses (multi-hit)
- **Firing**: Random intervals; bullets move downward
- **Movement**: Fixed or homing toward player

### Boss Battles
- **Health Bar**: Displayed during boss fights
- **Attack Patterns**: Unique firing sequences (spiral, spread, aimed)
- **Defeat Reward**: Major score boost, stage clear

### Power-Ups
- **Weapon Upgrades**: Drop from defeated enemies; change active weapon
- **Extra Bombs**: Replenish bomb count
- Applied via `applyPowerUp(type)` function

---

## Development Notes

### Code Style
- **Single global scope**: `window.game` object holds all state
- **Functional**: Methods are self-contained; minimal interdependencies
- **Canvas-based**: All rendering is direct canvas calls (no DOM elements except HUD)
- **Responsive**: Uses viewport-relative units (clamp, vw/vh) for mobile

### Common Patterns
- **State machine** for game phases (menu → playing → boss → game over)
- **Array iteration** for updating/drawing entities
- **Collision callbacks** for damage and scoring
- **Animation frames** via `requestAnimationFrame` for smooth 60 FPS

### Performance Considerations
- Large projectile/enemy arrays: Use object pooling for hot updates if needed
- Canvas rendering: Batch draws; avoid frequent state changes
- Audio: Lazy init prevents blocking on page load
- Collision: Early exit; spatial partitioning could optimize further if needed

### Debugging
- Browser console: No external logger; use `console.log()` or breakpoints
- Canvas scaling: Inspect canvas size vs container; CSS `max-width`/`max-height` applied
- Audio issues: Check browser console for playback errors; ensure `ensureAudio()` was called

---

## Running & Testing

### Live Demo
1. Open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge)
2. Game auto-starts in 1–2 seconds
3. Use keyboard or touch controls to play

### Mobile Testing
- Responsive canvas scaling handles phone/tablet screens
- Touch drag targets player movement and aim
- Test on various screen sizes: portrait, landscape, fullscreen

### GitHub Repository
- Hosted at: `https://github.com/ubban/google_4`
- Branch: `claude/inspiring-planck-ttwjqf` for development

---

## Common Enhancements & TODOs

### Potential Improvements
1. **Levels/Stages**: Add level progression with increasing difficulty
2. **Sound Toggle**: UI button to mute/unmute audio
3. **High Scores**: localStorage persistence for local leaderboard
4. **Pause Menu**: Pause/resume during gameplay
5. **Tutorial**: On-screen hints for first-time players
6. **Difficulty Modes**: Easy, Normal, Hard with scaled enemy health/fire rate
7. **New Weapons**: Additional fire patterns (mines, shields, etc.)
8. **Mobile UI**: On-screen control buttons for devices without keyboard

### Code Refactor Ideas
1. **Module Pattern**: Wrap game in IIFE to avoid global scope pollution
2. **Entity Classes**: Convert entity objects to ES6 classes for clarity
3. **Resource Manager**: Centralize sound/image loading and caching
4. **Event System**: Custom events for scoring, damage, level changes
5. **Configuration**: Extract magic numbers (damage, speed, spawn rates) to constants

---

## Attribution & License

- **Author**: Created with Claude Code assistance
- **Framework**: Vanilla JavaScript + HTML5 Canvas
- **Repository**: `ubban/Google_4`

---

## Quick Reference

### Game State Object (`window.game`)
- `plane`: Player ship object
- `enemies`: Array of enemy objects
- `playerBullets`, `enemyBullets`: Projectile arrays
- `score`, `multiplier`: Scoring system
- `weapons`: Weapon type definitions
- `gameActive`: Boolean indicating play state

### Key Functions
- `update()` — Step game logic (60x/sec)
- `draw()` — Render all game objects
- `fireWeapon(type)` — Player fires current weapon
- `smartBomb()` — Deploy area damage
- `enemyFire()` — Enemy bullets spawn
- `checkCollisions()` — Hit detection
- `endGame()` — Stop and reset
- `announce(text, color)` — Display HUD message

---

## Support & Issues

For questions or contributions, refer to the GitHub repository issues or create a pull request on `claude/inspiring-planck-ttwjqf`.

Generated by Claude Code – Last updated September 24, 2026

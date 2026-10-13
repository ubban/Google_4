# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single HTML file (~1670 lines) with embedded CSS and JavaScript. The game features a player-controlled aircraft, multiple enemy types, boss encounters, weapon progression, power-ups, and a chain/multiplier scoring system.

## Architecture

The entire game is contained in `index.html` with no external dependencies. The code is organized into functional sections within a single IIFE (Immediately Invoked Function Expression):

### Core Game Systems

1. **Canvas Rendering**: All graphics are drawn to a 2D canvas using HTML5 Canvas API
   - Player and enemy aircraft rendered as geometric shapes
   - Particles, bullets, and effects drawn procedurally
   - HUD overlaid on top of game canvas

2. **Game Loop**: Continuous loop that:
   - Updates all entities (player, enemies, bullets, particles, power-ups, boss)
   - Handles collision detection (bullet-enemy, enemy-player, power-up pickup)
   - Renders frame to canvas
   - Manages timing using `requestAnimationFrame`

3. **Entity System**: 
   - `player`: Main character with position, weapon, health, speed
   - `enemies`: Array of enemy aircraft with different types (grunt, interceptor, tank)
   - `bullets`: Player bullets
   - `enemyBullets`: Enemy projectiles
   - `boss`: Boss entity (spawned after certain kill count)
   - `powerUps`: Weapon upgrades and bomb refills
   - `particles`: Visual effects (explosions, debris)

4. **Game State**:
   - Lives, score, kills, stage progression
   - Chain system (consecutive kills without pause increase multiplier up to 8x)
   - Weapon system (Vulcan gun or Laser, with levels 1-3)
   - Bomb system (special screen-clearing attack)
   - Difficulty settings (easy, normal, hard) affecting enemy patterns and player lives

5. **Input Handling** (multi-platform):
   - **Desktop**: WASD/Arrow keys for movement, mouse for steering, spacebar/click to fire
   - **Touch**: On-screen joystick in lower-left, fire button in lower-right, bomb button
   - **Controls**: Priority system where keyboard/joystick takes precedence over mouse

6. **Audio**: Procedural sound effects using Web Audio API (no audio files)
   - `beep()`: Generates tones (weapon fire, power-up pickup)
   - `noiseBurst()`: Generates noise (explosions)
   - Multiple SFX objects with pre-built sound effects
   - Mute toggle with persistent localStorage

7. **Visual Polish**:
   - Screen shake on player hit/bomb usage
   - Flash effect (white flash) on damage
   - Vignette effect (red flash) when taking damage
   - Reduced motion support (toggle animations for accessibility)
   - Stage banners and combo notifications

## Development

### Opening and Testing the Game

```bash
# Simply open the file in a browser
open index.html
# or serve locally
python3 -m http.server
# then open http://localhost:8000/index.html
```

### Key Functions and Their Roles

- `resetGame()`: Initializes game state for a new run; call when starting/restarting
- `update()`: Main game loop update function; handles entity updates, collisions, spawning
- `draw()`: Renders all entities and HUD to canvas
- `spawnEnemy()`: Creates new enemy based on current stage/difficulty
- `spawnBoss()`: Creates boss encounter
- `fireWeapon()`: Handles player weapon firing (Vulcan or Laser)
- `useBomb()`: Triggers bomb attack with screen-clear animation
- `addScore(base)`: Awards points with chain multiplier applied
- `rectsOverlap()`: AABB collision detection between entities
- `drawPlane()`, `drawBossShip()`: Render aircraft as geometric shapes

### Game Loop Mechanics

1. **Enemy Spawning**: Enemies spawn at top of screen based on timer and difficulty
2. **Waves**: Enemies gradually increase in difficulty; after ~25 kills per stage, boss spawns
3. **Weapons**: 
   - Vulcan (rapid fire, lower damage) vs Laser (slower, piercing)
   - Can upgrade weapon level via power-ups (1→2→3)
4. **Chain/Multiplier**: Killing consecutive enemies within ~1 second builds chain (max 8x multiplier)
5. **Bombs**: Limited screen-clear attacks; refill via power-ups
6. **Collision**: Player takes damage on enemy collision; enemies destroyed by bullets
7. **Stage Progression**: After defeating boss, next stage spawns with increased difficulty

### Editing Tips

- **Player positioning**: Adjust `player.y` (default: canvas.height - 100) or `player.speed` (default: 260)
- **Difficulty settings**: Edit the `DIFFICULTIES` object to change enemy patterns/player lives
- **Enemy types**: Modify `ENEMY_TYPES` object to change enemy behavior, appearance, or bullet patterns
- **Scoring**: `addScore()` multiplies base points by chain multiplier; adjust base values as needed
- **Weapon behavior**: `fireWeapon()` handles both Vulcan and Laser; modify spread or damage if needed
- **Canvas size**: Adjust via `canvas.width`/`canvas.height` and scale with `resize()` on window resize

### Performance Considerations

- Canvas rendering is relatively fast; ensure collision detection remains O(n²) or lower
- Particle count grows over time; `particles` array can be pruned if frame rate drops
- Web Audio API can be resource-intensive if too many sounds play simultaneously
- Use browser DevTools Performance tab to profile if FPS drops

## Responsive Design

The game is fully responsive for mobile/tablet and desktop:
- Canvas scales to fit container (max-width/max-height with aspect ratio preservation)
- HUD stats and buttons scale with viewport using `clamp()` CSS functions
- Touch controls (joystick + buttons) appear on touch devices
- Safe area insets respected on notched devices (viewport-fit=cover)

## Testing Your Changes

Since there's no automated test suite, test manually in a browser:
1. Open game, verify it loads without errors (DevTools console)
2. Test all game mechanics: movement, firing, bombs, power-ups, boss
3. Try on mobile device or browser DevTools device emulation
4. Check that frame rate remains smooth (60 FPS target)
5. Test sound effects toggle and mute behavior

## Browser Compatibility

- Requires modern browser with Canvas 2D and Web Audio API support
- Tested on Chrome, Firefox, Safari; IE not supported

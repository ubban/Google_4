# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML/CSS/JavaScript application. The entire game runs in a web browser with no external dependencies.

## Architecture

The game is structured as a single HTML file (`index.html`) containing three main sections:

### 1. HTML Structure (top)
- Game canvas element (`#gameCanvas`)
- HUD container for score, best, weapon level, lives, bombs
- Control buttons: mute, motion toggle, pause
- Overlay for start/game-over screens
- Screen reader live region for accessibility

### 2. CSS Styling (embedded `<style>`)
- Dark theme optimized for space shooter aesthetic
- Responsive layout using viewport-relative units
- Safe-area insets for mobile devices (notches, safe zones)
- Canvas responsive scaling
- Accessibility features (reduced-motion preference, sr-only class)

### 3. JavaScript Game Engine (main `<script>` IIFE)
Organized into functional sections:

#### Core Game State
- `player` - player ship object
- `bullets` - array of player projectiles
- `enemies` - array of active enemy ships
- `enemyBullets` - array of projectiles fired by enemies
- `particles` - visual effect particles (explosions, hits, etc.)
- `powerUps` - drop items (extra lives, bombs, weapon upgrades)
- `boss` - boss enemy object (spawned at stage 3)

#### Game Loop
- `loop(now)` - main animation frame callback (called via `requestAnimationFrame`)
- `update(dt)` - game logic update, handles:
  - Player movement and firing
  - Enemy spawning and behavior
  - Bullet-entity collision detection
  - PowerUp collection
  - Lives/bomb management
  - Wave/stage progression
- `draw()` - canvas rendering of all game entities
- Delta time (dt) passed to update for frame-rate independent physics

#### Game Systems

**Difficulty System** (`DIFFICULTIES` constant):
- Three levels: easy, normal, hard
- Controls enemy spawn rate, bullet speed, and player health
- Selected via UI buttons before game start

**Enemy Types** (`ENEMY_TYPES` constant):
- `grunt` - basic enemy (1 hit)
- `interceptor` - faster enemy (2 hits)
- `tank` - slow heavy enemy (5 hits), draws health bar
- Boss enemy with custom firing patterns (3 attack modes)

**Weapon System**:
- VULCAN (default) - rapid fire straight bullets
- LASER - charging beam weapon
- SPREAD - 3-bullet spread pattern
- HOMING - heat-seeking missiles
- RADIAL - omnidirectional burst
- Each has level upgrades (visual and behavior changes)

**Audio System**:
- Synthesized SFX using Web Audio API (no audio files needed)
- `beep(freq, dur, type, vol, opts)` - tone generation
- `noiseBurst(dur, vol, opts)` - noise-based sounds
- `sfx` object contains preset sound effects
- Audio context initialized on first user interaction (click/touch)
- Mute toggle persistent across sessions

**Rendering Features**:
- Canvas 2D context transformations (translate, rotate, scale, save/restore)
- Sprite-like drawing using `drawPlane()` and `drawBossShip()` functions
- Glow effects (conditional on performance)
- Screen shake effect on hits (toggleable via reduced-motion preference)
- Particle effects for explosions and impacts
- HUD updates for score, multiplier, weapon display

#### Collision & Physics
- `rectsOverlap()` - AABB collision detection between rectangles
- Hitbox-based damage system
- Bomb creates radial explosion clearing nearby enemies and bullets

#### Accessibility
- Screen reader announcements via `announce()` function
- Reduced motion preference detection and toggle
- Semantic HTML structure
- ARIA live region for dynamic status updates

## Development Notes

### Game Flow
1. Player clicks start → `resetGame()` initializes state
2. Game loop runs at 60 FPS via requestAnimationFrame
3. Each frame: physics update → collision checks → render
4. Waves spawn enemies progressively (30+ enemies before stage clear)
5. Stage 3 spawns a boss with special attack patterns
6. Game ends when lives reach 0

### Key Constants to Adjust
- `DIFFICULTIES` - spawn rates, enemy speeds, player health
- `ENEMY_TYPES` - enemy HP, speed, point values
- Canvas dimensions - game assumes wide viewport; scale handled via CSS
- Weapon parameters - bullet speed, spread angle, fire rate

### Common Tasks

**Add new enemy type**: 
- Add entry to `ENEMY_TYPES` constant with hp, speed, score
- Add case in `spawnEnemy()` to instantiate it
- Add rendering in main `draw()` loop if unique appearance needed

**Adjust weapon balance**:
- Modify parameters in weapon update code (search for weapon name like "VULCAN")
- Change bullet speed, fire rate, or pattern in the update section

**Tweak difficulty**:
- Edit `DIFFICULTIES` object for spawn rates and player health
- Adjust ENEMY_TYPES speeds for harder enemy behavior

**Add visual effects**:
- Create particles in `particles` array with {x, y, color, dx, dy, life, maxLife}
- Adjust alpha/scale in particle render section for different effects

### Performance Considerations
- `glow` variable toggles expensive shadow effects
- Canvas dimensions checked in `loop()` - responsive resizing works but triggers recalc
- Particle limit (no hard cap; consider adding if performance issues)
- Enemy count grows; consider culling off-screen enemies if lag occurs

### Browser Compatibility
- Requires Canvas 2D API and requestAnimationFrame
- Web Audio API for synthesized sound
- Tested on modern browsers; mobile touch/gesture support included
- Safe-area viewport handling for notched devices

## Testing the Game

Simply open `index.html` in a modern web browser. No build or server needed.

### Testing checklist for changes:
1. **Gameplay**: Verify enemy spawn, collision detection, weapon firing
2. **Audio**: Test mute toggle, weapon sounds, explosion effects
3. **Mobile**: Check responsive layout, touch controls, safe-area handling
4. **Accessibility**: Test reduced-motion toggle, screen reader announcements
5. **Performance**: Monitor frame rate with many enemies/particles on screen

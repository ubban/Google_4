# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game, entirely contained in a single `index.html` file (~1669 lines). The game runs in the browser with no external dependencies, featuring player combat, enemy waves, boss fights, and progressive difficulty stages.

## Running the Game

### Local Testing
Simply open `index.html` in a web browser. The game:
- Uses Canvas 2D API for rendering
- Generates audio dynamically using Web Audio API
- Stores high scores in localStorage under key `SKY_FIGHTER_BEST`
- Supports keyboard (arrow keys + Space) and mouse input

### Live Deployment
The `index.html` file is ready to serve as-is over HTTP/HTTPS with no build or compilation step required.

## Code Architecture

The entire game lives in a single HTML file with three sections:

### 1. HTML Structure (lines ~1-50)
- Meta tags for SEO, viewport, mobile viewport-fit
- Single `<canvas id="gameCanvas">` element for rendering
- HUD overlay with stats display (hearts, bombs, weapons, score, stage)
- Various UI elements (pause button, overlay for menus)
- One `<script>` block containing all JavaScript

### 2. CSS Styling (lines ~50-200)
- Dark space theme with gradients
- Responsive canvas sizing (`max-width: 100%; max-height: 100%`)
- HUD overlay with glassmorphism effects (backdrop-filter: blur)
- Animation definitions (heartBeat, vignettePulse, flash effects)
- Mobile-safe styling with viewport-fit and safe-area-inset

### 3. JavaScript Game Engine (lines ~200-1669)
The game is organized into clear subsystems:

#### **Game State & Config**
- `THEMES`: 5 visual themes (each with sky gradient colors, sun color, cloud color)
- `BOSSES`: Array of 6 boss designs with unique attack patterns
- `KILLS_PER_STAGE`: Threshold for boss spawn (25 kills)
- Player state: position, weapon, level, health, bombs, invulnerability
- World state: enemies, bullets, particles, power-ups

#### **Rendering Pipeline** (`draw()` function)
1. Clear canvas and apply screen shake if active
2. Render themed sky (gradient + sun effect)
3. Render layered clouds (parallax effect at two speeds)
4. Render player (plane sprite with weapon indicator)
5. Render all game objects (enemies, bullets, particles, power-ups)
6. Render visual effects (screen shake, vignette on hit, flash)

#### **Update Loop** (`update(dt)` function)
- Timestep-based with deltaTime for smooth 60fps gameplay
- Player input handling (keyboard/mouse)
- Collision detection and damage resolution
- Enemy behavior (movement, shooting, patterns)
- Power-up spawning and collection
- Score tracking and combo system

#### **Game Systems**

**Player System:**
- Two weapons: "vulcan" (rapid fire) and "laser" (piercing, slower)
- 3 upgrade levels per weapon (damage/fire-rate improvements)
- Invulnerability frames after being hit
- Weapon switching via power-ups

**Enemy System:**
- Spawning based on difficulty curve (spawn rate decreases as stage progresses)
- Regular enemies: small, medium, tank types with different HP/speed/patterns
- Enemy bullets: straight-line, spread, and homing variants
- Per-stage enemy configurations

**Boss System:**
- 6 unique boss designs (Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan)
- Each boss has 3 unique attack patterns (radial bursts, aimed spreads, telegraphed combos)
- Telegraph animations before attacks (visual + sound cue)
- Enters from top with animation, bounces horizontally

**Power-Up System:**
- **Life**: Restores 1 health (capped at maxLives)
- **Bomb**: Adds smart bomb (clears screen, invulnerability)
- **Weapon**: Upgrades weapon or switches weapon type

**Scoring & Progression:**
- Score multiplier increases with consecutive kills (chain system)
- Breaking chain resets multiplier (gap in kills)
- Stages progress every 25 kills to a boss
- 5 visual themes cycle as stages increase

#### **Audio System** (`setupAudio()`)
- Procedurally generated sounds (no audio files)
- Web Audio API with AudioContext
- Sound effects: shoot, powerup, hit, explode, boss telegraph, game over
- Panned based on position for spatial audio

#### **Key Functions**
- `fireWeapon()`: Spawn player bullets based on weapon type
- `spawnBoss()`: Create boss with enter animation
- `updateBoss(dt)`: Boss attack patterns and movement
- `updateEnemy(dt)`: Regular enemy updates
- `applyPowerUp(type)`: Handle power-up collection
- `loseLife()`: Handle player damage
- `endGame()`: Game over state and score save
- `togglePause()`: Pause/resume functionality

## Common Development Tasks

### Testing the Game
1. Open `index.html` in Chrome/Firefox (ensure JavaScript enabled)
2. Verify gameplay:
   - Player movement with arrow keys or mouse
   - Firing with Space
   - Weapon switching after collecting power-ups
   - Enemy spawning and difficulty curve
   - Boss appearance after 25 kills
   - Boss attack patterns (6 different patterns per boss)
   - Death and game over
3. Check console for errors (F12 → Console)

### Adding a New Enemy Type
1. Find `const enemyTypes` config
2. Add new type with properties: `hp`, `speed`, `score`, `canShoot`, `pattern`, `rarity`
3. Update `spawnEnemy()` to include new type in selection
4. Adjust stage difficulty in `d.spawnMin` and `d.spawnBase` values

### Modifying Boss Patterns
1. Find `BOSS_PATTERNS` array (6 bosses × 3 patterns each)
2. Each pattern is a function `b => fireXxx(b, ...args)` firing bullets
3. Modify firing functions (`fireRadial`, `fireAimedSpread`, etc.) or create new patterns
4. Patterns execute when boss's telegraph finishes

### Adjusting Game Balance
- **Difficulty**: Edit stage configurations in `THEMES` loop (spawnMin/spawnBase)
- **Weapon strength**: Modify bullet damage multipliers in weapon fire code
- **Player speed**: Change `player.speed` constant
- **Boss HP**: Modify `boss.maxHp` in `spawnBoss()`

### Changing Visual Theme
- Update `THEMES` array with new sky gradients, sun color, cloud color
- Modify cloud drawing in `drawClouds()` if needed
- Update player plane color in `drawPlane()`

## Codebase Constraints & Notes

- **No external libraries**: Pure vanilla JavaScript, Canvas API, Web Audio API
- **Responsive design**: Game scales to window size while maintaining 16:9 aspect ratio where possible
- **Mobile optimized**: Touch input supported, safe-area-inset respected
- **Performance**: All rendering done to single canvas, object pooling for bullets/particles
- **State management**: Global variables for player, enemies, bullets (no class-based architecture)
- **Collision detection**: Simple AABB (axis-aligned bounding box) with `rectsOverlap()`

## File Structure

```
index.html          # Self-contained game file (1669 lines)
  ├── HTML structure
  ├── CSS styling (dark theme, HUD, animations)
  └── JavaScript engine (rendering, update loop, game systems)
README.md           # Minimal (timestamp only)
CLAUDE.md           # This file
```

## Performance Considerations

- Canvas rendering is the main bottleneck; particle effects and many bullets can reduce frame rate
- Each frame updates ~200+ game objects (bullets, enemies, particles)
- No off-screen canvas pre-rendering; all rendering happens each frame
- To optimize: reduce max particles, simplify enemy patterns, or implement object pooling

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented entirely in a single HTML file. It's a browser-based arcade game with no external dependencies, featuring:
- Canvas-based 2D rendering
- Procedural audio (Web Audio API synthesis)
- Touch and keyboard/mouse input support
- 6 boss encounters with distinct attack patterns
- Difficulty settings (Easy, Normal, Hard)
- Combo/multiplier scoring system
- Power-ups and progressive weapon upgrades

The entire game—HTML, CSS, and JavaScript—is contained in `index.html` (~1,670 lines).

## Architecture & Game Flow

### Main Systems

The game operates as a single-threaded game loop using `requestAnimationFrame`. The core flow is:

```
1. User Input (keyboard, mouse, touch)
   ↓
2. Update Game State (update function, dt = frame delta time)
   ├─ Player movement & shooting
   ├─ Enemy spawning and AI
   ├─ Collision detection
   ├─ Boss state and attack patterns
   └─ Scoring & multiplier updates
   ↓
3. Render (draw function)
   ├─ Background gradient & sky theme
   ├─ Clouds and visual effects
   ├─ Entities (player, enemies, bullets, bosses)
   └─ HUD overlay
   ↓
4. Schedule Next Frame (requestAnimationFrame)
```

### Key Game State Variables

- **Player**: position, weapon type, level, health, invulnerability timer
- **Entities**: bullets (player), enemyBullets, enemies, boss, powerUps, particles
- **Game**: score, lives, multiplier, chain count, stage, kills
- **Settings**: difficulty, muted, reducedMotion

### Performance Adaptation

The game tracks frame times and automatically reduces visual quality (`quality` 0-2) when framerate drops below 40fps, disabling shadow effects and particle count scaling.

## Key Game Systems

### 1. Input System

| Input | Action |
|-------|--------|
| Mouse hover + click | Steer & fire |
| Arrow keys / WASD | Movement |
| Space | Fire |
| X / Shift | Drop bomb |
| P / Escape | Pause |
| Touch (left zone) | Virtual joystick |
| Touch (FIRE/BOMB buttons) | Mobile controls |

**Priority**: Keyboard/joystick input always takes priority over mouse steering to prevent conflicts.

### 2. Weapons System

Two weapon types with level progression:

**VULCAN (default)**
- Level 1: single shot, narrow spread
- Level 2: 2-shot spread
- Level 3: 3-shot spread (max)
- Mechanics: straight line travel, no piercing at level 1-2

**LASER**
- Level 1: single lane
- Level 2: 2-lane split fire
- Level 3: 3-lane fire (centered + sides)
- Mechanics: pierce through enemies (pierce counter)

Weapon cycling: reach level 3 → next power-up switches weapon and resets level to 1.

### 3. Enemy Spawning & Types

Enemies spawn based on difficulty and current stage. Three types:

```javascript
ENEMY_TYPES = {
  grunt:       { w: 34, h: 34, hp: 1, speedMul: 1.0,  score: 10, minStage: 1 },
  interceptor: { w: 26, h: 26, hp: 1, speedMul: 1.7,  score: 15, minStage: 1, weave: true },
  tank:        { w: 46, h: 46, hp: 4, speedMul: 0.55, score: 35, minStage: 2 }
}
```

Interceptors weave side-to-side; tanks have visible HP bars. Spawn timer decreases as the game progresses (difficulty ramp).

### 4. Boss System

**Boss Encounters:**
- One boss per stage (6 unique boss designs)
- Boss spawns after player defeats 18 enemies (KILLS_PER_STAGE)
- Each boss has a unique "design" object (colors, core glow) and attack pattern set

**Boss Phases:**
1. **Entering**: slides into view from top, invulnerable
2. **Active**: bounces horizontally, charges telegraph, fires attack pattern
3. **Dead**: explosion, stage complete, next stage begins

**Attack Patterns**: Each boss has 3 distinct patterns (radial burst, aimed spread, wall fire, etc.) that cycle. Patterns are defined in `BOSS_PATTERNS` array—easy to add new ones.

### 5. Scoring & Multiplier

- Base score per enemy kill = enemy.score
- Multiplier increases every 5 consecutive kills (cap at 1x to 10x)
- Chain breaks if player takes damage or doesn't kill within 2.2 seconds
- Boss kill grants flat 500 points

### 6. Audio System

**Synthesized sounds via Web Audio API** (no external files):
- `beep()`: tone with sweep and filter
- `noiseBurst()`: filtered noise
- Individual SFX for: shoot, laser, explosions (by type), hit, bomb, powerup, boss patterns, ui

All audio respects mute toggle and panning based on entity x-position.

### 7. Visual Themes

Six alternating sky themes (stage-based) control:
- Sky gradient (3 color stops)
- Cloud color and size
- Sun glow color

Defined in `THEMES` array. Easy to add new themes.

## Common Development Tasks

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` (define w, h, hp, speedMul, score, minStage, optional weave)
2. Update enemy spawn logic in `spawnEnemy()` if special probability needed
3. Add explosion SFX to `EXPLODE_SFX` object if custom sound desired
4. Test collision and shooting behavior

### Modifying Difficulty

Edit the `DIFFICULTIES` object to adjust spawn rates, enemy speed, shoot chance, starting lives, bombs, or boss HP multiplier.

### Adding a New Boss Pattern

1. Write a function (e.g., `fireWavePattern()`) that calls `enemyBullets.push()` to add bullets
2. Add function to a new entry in `BOSS_PATTERNS` array (3 patterns per boss)
3. Reference position `b.x`, `b.y` and player position `player.x`, `player.y`
4. Optional: use `b.patternIndex` for frame-based variations

Example:
```javascript
b => {
  for (let i = -2; i <= 2; i++) {
    enemyBullets.push({
      x: b.x + i * 30, y: b.y,
      w: 6, h: 6, radial: true,
      dx: 0, dy: 250
    });
  }
}
```

### Adding a New Boss

1. Add entry to `BOSSES` array with name, hull/wing/core colors, and hpMul
2. Add corresponding 3-pattern array to `BOSS_PATTERNS`
3. Boss selection uses stage index: `BOSSES[(stage - 1) % BOSSES.length]`

### Adjusting Visual Effects

- **Screen shake**: Call `screenShake(magnitude, duration)`
- **Flash overlay**: Add/remove 'hit' or 'bomb' class to `flashEl`
- **Particles**: Push to `particles` array with { x, y, vx, vy, life, maxLife, color }
- **Combo pop**: Call `showComboPop(text)`
- **Animations**: CSS animations in `<style>` section (keyframes), re-triggered by class toggle

### Tweaking Game Balance

Key numeric parameters:
- `player.speed`: movement speed
- `BASE_W / BASE_H`: canvas dimensions
- `fireDelay`: weapon cooldown
- `KILLS_PER_STAGE`: kills required before boss
- Bullet speeds and damage (bullets vs enemies: 1 hp each, bullets vs boss: 3 hp)
- Power-up spawn chances in `maybeSpawnPowerUp()`

### Testing Specific Game States

The game stores persistent state in localStorage:
- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Clear with `localStorage.clear()` to reset; useful for testing new game start.

## Code Structure

### Main Sections (in order)

1. **Constants & Configuration** (lines ~422–530): Difficulty, enemy types, bosses, themes
2. **Audio System** (lines ~545–676): Web Audio initialization, SFX definitions
3. **UI Rendering** (lines ~680–750): HUD updates, button listeners
4. **Canvas & Input** (lines ~753–862): Resize handling, keyboard, mouse, touch input
5. **Game State & Entities** (lines ~864–1085): Player, bullet, collision, explosion spawning
6. **Update Loop** (lines ~1148–1302): Input → movement, shooting, collision, death
7. **Boss System** (lines ~1411–1454): Boss update logic, attack patterns, defeat handling
8. **Game Lifecycle** (lines ~1469–1505): Game over, pause, resume
9. **Render Loop** (lines ~1507–1638): Sky, clouds, entities, HUD overlay
10. **Main Game Loop** (lines ~1640–1666): requestAnimationFrame, start button listener

### Helper Functions

- `drawPlane()`: Renders player/enemy ship shape
- `drawBossShip()`: Boss-specific rendering with design colors
- `drawClouds()`: Parallax cloud effect
- `fireRadial()`, `fireAimedSpread()`, etc.: Boss attack pattern builders
- `rectsOverlap()`: Simple AABB collision detection

## Performance Considerations

- **Particle count scales** with quality setting (full, medium, or low)
- **Shadow effects** disabled on lower quality
- **requestAnimationFrame** caps at ~60fps (browser-dependent)
- **dt capped at 0.05s** to prevent large jumps (important for collision consistency)
- **No physics engine**: simple velocity-based movement only

## Accessibility Features

- **Screen reader support**: `#srLive` announces events (score, stage, damage)
- **Reduced motion preference**: Respects `prefers-reduced-motion: reduce` media query
- **Keyboard controls**: Full game playable without mouse/touch
- **Color contrast**: Tested for legibility in dark sky backgrounds

## Deployment & Testing

**To run locally:**
- Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge)
- No build step needed; no external dependencies

**To test on mobile:**
- Open URL on phone/tablet
- Touch controls auto-enable if touch API detected
- Test both portrait and landscape orientations (resize handling active)

## Debugging Tips

- Open browser DevTools console (`F12`)
- Game logs events via `announce()` to `#srLive`
- Check `localStorage` for persistent state
- Audio context may fail in strict privacy mode; handled gracefully
- Frame rate visible in console via adaptive quality changes

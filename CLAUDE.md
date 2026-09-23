# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The entire game (1,600+ lines) is contained in `index.html` with embedded CSS and JavaScript using the Canvas API.

## Running the Game

No build process is required. Simply open `index.html` in a web browser:
- **Desktop**: Open the file directly or serve via a simple HTTP server
- **Mobile**: Works with touch controls (joystick on left, FIRE/BOMB buttons on right)
- **Development**: Use `python -m http.server 8000` and visit `http://localhost:8000/index.html`

## Architecture Overview

The game follows a classic game loop pattern:

```
requestAnimationFrame(loop)
  → update(dt)      [game state, collisions, logic]
  → draw()          [render to canvas]
  → repeat
```

### Core Game State
```javascript
player         // { x, y, w, h, speed, weapon, level, health, invuln }
bullets        // Player projectiles with pierce/laser properties
enemies        // Array of spawned enemies with various types
enemyBullets   // Enemy projectiles (radial, aimed, or spread patterns)
powerUps       // Life, bomb, weapon upgrades that float down
particles      // Explosion/effect particles
boss           // Current stage boss (null when not active)
```

## Key Game Systems

### Enemy Types
- **Grunt**: Basic red fighter, 1 HP, score 10 points
- **Interceptor**: Fast weaving fighter, 1 HP, score 15 points
- **Tank**: Heavily armored purple unit, 4 HP, score 35 points (spawns from stage 2+)

Each type has a spawning function (`spawnEnemy()`) that randomizes velocity, shoot behavior, and pattern (spread vs. aimed fire).

### Weapon System
Two weapons toggle based on powerups:
- **Vulcan**: Rapid-fire bullets, spread increases with levels (1-3)
  - Level 1: Single narrow shot
  - Level 2: Dual spread
  - Level 3: Triple spread
- **Laser**: Piercing beams that ignore some enemies
  - Level 1: Single lane
  - Level 2: Two lanes
  - Level 3: Three lanes

Collision detection calls `fireWeapon()` which populates the `bullets` array.

### Scoring & Multiplier Chain
- Chain counter increments on enemy kills (2.2s window per kill)
- Multiplier starts at 1x, caps at 10x (every 5 kills adds 1x)
- Each enemy score is multiplied: `addScore(baseScore * multiplier)`
- Chain breaks if player takes damage or waits too long between kills

### Boss System
- One boss per stage (6 designs: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Each boss has 3 unique attack patterns that cycle with telegraph delay
- Boss enters at y=120, moves horizontally, bounces at canvas edges
- Health scales by difficulty and boss design (`hpMul` 0.8–1.7x)
- Death triggers stage clear sound and stage increment

### Difficulty Scaling
```javascript
DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, lives: 4, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, lives: 3, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, lives: 3, bossHpMul: 1.3 }
}
```
Spawn rate decreases over time (`frame / 300` ramp). Enemy behavior intensifies per difficulty.

### Input Handling
- **Keyboard**: Arrow keys or WASD to move, Space or mouse click to fire, X/Shift to bomb, P/Esc to pause
- **Mouse**: Steer toward pointer (glide behavior), click to fire
- **Touch**: Left zone is joystick (move), right zone has FIRE/BOMB buttons
- All input types coexist; keyboard/joystick take priority over mouse to avoid conflicts

### Collision System
- `rectsOverlap(a, b)` checks AABB collision between all game objects
- Collision checks happen per frame:
  - Player bullets vs. enemies → damage enemies
  - Player bullets vs. boss → 3 damage per hit
  - Enemy bullets vs. player → lose life
  - Enemies vs. player → lose life
  - Powerups vs. player → apply effect
  - Bombs clear all enemy bullets and damage all enemies

### Visual & Audio Systems

**Web Audio Synthesis** (no external assets):
- Procedural beeps/noise bursts using OscillatorNode, BiquadFilter, noise buffers
- `sfx` object contains all sound effects (shoot, laser, explode, hit, bomb, powerup, boss, gameover, etc.)
- `panFor(x)` converts canvas X position to stereo panning (-1 to 1)
- Master compressor settings: threshold -20dB, ratio 5, attack 3ms, release 180ms

**Visual Effects**:
- **Screen shake**: `screenShake(magnitude, duration)` — randomizes ctx.translate()
- **Flash**: Hit (white quick flash) vs. bomb (white longer flash)
- **Vignette**: Red pulsing overlay on critical health (1 life)
- **Particles**: Colored burst particles on enemy death (count scales with quality)
- **Quality adaptation**: 3 levels (0=full, 1=medium, 2=low) based on frame time average

**Themes**: 6 color schemes cycle per stage (sky gradient, cloud color, sun color)

### Performance Optimization
- `quality` variable (0-2) is tracked based on frame time average
- Lower quality reduces:
  - Particle count in explosions
  - Shadow blur and glow effects
  - Vignette animations
- CSS `prefers-reduced-motion` media query respected; `reducedMotion` localStorage override

### Save State
- `localStorage` keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
- Best score persists across sessions
- Mute/reduced-motion/difficulty selection persists

## Important Implementation Patterns

### Stage Progression
- 18 kills (`KILLS_PER_STAGE`) triggers boss warning
- Boss spawns after 1.2s delay via setTimeout
- Stage clears with boss death, spawning new stage banner
- Enemy types unlock at higher stages (tank at stage 2+)

### Boss Attack Patterns
Each boss pattern is a function that fires enemy bullets in specific configurations:
- `fireRadial(boss, count, speed)` — circular burst
- `fireAimedSpread(boss, count, speed, spreadStep)` — fan toward player
- `fireColumns(boss, speed)` — vertical columns
- `fireSpiralBurst(boss, speed)` — expanding spiral
- `fireDoubleRing(boss, speedA, speedB)` — two concentric rings

Telegraph delay (`b.telegraph`) warning window before attack fires.

### Invulnerability Window
- `player.invuln` counts down after damage (1.5s default)
- During invuln, player blinks (frame/4 alternates visibility)
- Prevents death chains from rapid hits

### Combo Visual Feedback
- `showComboPop(text)` displays centered yellow text with scaling animation
- `showStageBanner(text)` displays stage transition text with 2s fade
- Screen reader announcements via `announce(text)` to `srLive` element

## Common Development Tasks

### Adding a New Enemy Type
1. Add to `ENEMY_TYPES` object with w, h, hp, speedMul, color, score, minStage
2. Update `spawnEnemy()` probability logic if needed
3. Add explosion SFX to `EXPLODE_SFX` mapping

### Adding a New Boss Pattern
1. Create a firing function (e.g., `fireCustomPattern(boss, speed)`)
2. Add 3 patterns to a new entry in `BOSS_PATTERNS` array
3. Add boss design to `BOSSES` array in order (6 bosses total)

### Tweaking Difficulty Balance
- Adjust `DIFFICULTIES` spawn times and enemy speed multipliers
- Modify `baseHp` calculation in `spawnBoss()` (currently `70 + stage * 35`)
- Adjust individual enemy HP values in `ENEMY_TYPES`

### Fixing Collision Issues
- Collision detection uses `rectsOverlap(a, b)` for all game objects
- Bounding box dimensions: `w` and `h` properties; collision checks `x` and `y` centers
- Adjust collision radii by changing w/h on game objects

### Canvas Responsiveness
- Base resolution: 480×720 (portrait)
- CSS scales to fit screen: `Math.min(maxW / BASE_W, maxH / BASE_H, 1.6)`
- Safe area insets handled for notches/rounded corners
- Resize listener updates dimensions on orientation change

## Debugging Tips

- Open DevTools Console to check for audio context errors
- `frame` counter increments every update; useful for timing
- `quality` value (0-2) indicates performance adaptation
- `reducedMotion` flag controls animations
- LocalStorage values can be inspected: `localStorage.getItem('skyfighter.bestScore')`

## Code Style Notes

- No external libraries; vanilla JS/Canvas only
- IIFE wrapping keeps variables scoped
- Constants (DIFFICULTIES, ENEMY_TYPES, BOSSES, THEMES, BOSS_PATTERNS) defined at top
- Helper functions grouped by feature (audio, rendering, game logic)
- CSS variables minimal; inline styles for dynamic effects

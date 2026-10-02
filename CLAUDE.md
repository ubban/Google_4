# Sky Fighter — Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. Players pilot a fighter plane through multiple stages, defeat enemy squadrons, collect power-ups, and face increasingly difficult boss encounters.

The game is fully self-contained in `index.html` — no external assets, dependencies, or build process required. Everything (HTML structure, CSS styling, and JavaScript game logic) is embedded inline.

**Repository:** https://github.com/ubban/Google_4

## File Structure

```
Google_4/
├── index.html          # Single-file game application (HTML + CSS + JS)
├── README.md           # Minimal readme
├── CLAUDE.md           # This file
└── .git/               # Git history
```

## Game Features

### Core Gameplay
- **Vertical scrolling shooter:** Navigate a 480×720 canvas and destroy enemies
- **Three difficulty levels:** Easy, Normal, Hard (affects spawn rates, enemy behavior, lives, bombs, boss HP)
- **Progressive stages:** Complete each stage by defeating 18 enemies, then face a boss
- **Six unique bosses** with distinct attack patterns (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- **Combo/chain system:** Rapid kills increase score multiplier (x1 up to x10 for every 5 kills)

### Weapons & Power-ups
- **Two weapon types:**
  - Vulcan (default): Spread fire, fast shots
  - Laser: Focused beam, piercing shots
  - Both upgrade to level 3, then swap weapon types
- **Power-up drops (random):**
  - 💚 Life: Restore one health point
  - 💣 Bomb: Smart bomb that clears all enemies and damages boss
  - 🔫 Weapon: Upgrade weapon level or switch weapon type

### Visual Design
- **Six dynamic sky themes** with gradient backgrounds, clouds, and sun effects
- **Smooth animations:** Enemy weaving, boss telegraphs, particle explosions
- **Responsive design:** Scales to fit viewport on desktop and mobile
- **Accessibility:** Screen reader announcements for important events
- **Motion preferences:** Reduced motion mode for users who prefer minimal effects

### Controls
- **Desktop:**
  - Arrow keys / WASD: Move
  - Space: Fire weapon
  - X / Shift: Drop bomb
  - P / Esc: Pause
  - Mouse: Hover to steer, click to fire
- **Mobile:**
  - Left side: Joystick for movement
  - Right side: FIRE and BOMB buttons
  - Touch-friendly layout with visual feedback

### Audio
- Synthesized sound effects (no external audio files)
- WebAudio API with oscillators, filters, and noise generation
- Mutable via button in top-right corner
- Sound effects for: shooting, explosions, power-ups, boss attacks, game over

### Persistent State
- Best score stored in localStorage
- Difficulty preference remembered
- Mute/motion preferences saved

## Technical Architecture

### Canvas & Rendering
- Fixed 480×720 logical canvas size, dynamically scaled to fit viewport
- Single `requestAnimationFrame` loop running at ~60fps
- **Adaptive quality:** Auto-detects frame rate and lowers particle count / effects if performance drops
- All rendering done via Canvas 2D API with solid colors, gradients, and alpha compositing

### Game State Management
- Global state variables: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- Score, lives, combo tracking, stage progression
- Pausable and resumable at any time

### Input Handling
- Keyboard: keydown/keyup listeners
- Touch: Multi-touch support with joystick visualization for left side, buttons for right side
- Mouse: Cursor tracking for steering (lower priority than keyboard/gamepad inputs to avoid fighting)

### Enemy & Boss System
- **Three enemy types:**
  - Grunt: Basic enemy, red, 1 HP
  - Interceptor: Fast, aggressive, 1 HP, weaving movement
  - Tank: Heavy, slow, 4 HP, shows health bar
- **Boss behavior:** Entering phase → combat phase with alternating attack and cooldown patterns
- **Spawn scaling:** Enemy spawn rate increases with frame count (difficulty ramp)

### Difficulty System
- **Easy:** More lives (4), fewer bombs spawn faster, slower enemies, lower boss HP
- **Normal:** Standard 3 lives, 2 bombs, faster enemies
- **Hard:** Same lives as Normal, faster enemies, higher boss HP, aggressive shooting

### Particle System
- Explosion particles spawned on enemy/boss destruction
- Simple physics: velocity + gravity over time with fade-out
- Quality-adaptive: Reduces particle count when performance is low

### Audio Synthesis
- WebAudio Context initialized on first user interaction
- `beep()`: Frequency sweep with optional filter and panning
- `noiseBurst()`: White noise with filter for impact sounds
- Pan effects position sound left/right based on on-screen position

## Code Organization

### Key Functions

**Initialization & State**
- `resetGame()`: Resets all game variables, canvas, UI for new game
- `endGame()`: Handles game over logic, updates best score

**Rendering**
- `draw()`: Main draw function (backgrounds, entities, particles)
- `drawPlane()`: Draws player and enemy aircraft
- `drawBossShip()`: Draws boss with dynamic animations
- `drawClouds()`: Parallax scrolling cloud layers

**Game Logic**
- `update(dt)`: Main game loop update (collision detection, entity movement)
- `fireWeapon()`: Spawns player bullets
- `useBomb()`: Clears enemies, damages boss
- `updateBoss(dt)`: Boss movement and attack pattern execution
- `spawnEnemy()`, `spawnBoss()`: Entity spawning

**Collision & Scoring**
- `rectsOverlap(a, b)`: Simple AABB collision detection
- `addScore(base)`: Score + combo multiplier tracking
- `applyPowerUp(type)`: Applies power-up effects

**Boss Attack Patterns**
- `fireRadial()`: Radial burst of bullets
- `fireAimedSpread()`: Fan aimed at player
- `fireColumns()`: Straight columns downward
- `fireSpiralBurst()`: Rotating spiral
- `fireDoubleRing()`: Two concentric rings of bullets

## Data Structures

### Player
```javascript
{
  x, y,           // Position
  w, h,           // Dimensions
  speed,          // Movement speed
  cooldown,       // Fire rate limiter
  weapon,         // 'vulcan' or 'laser'
  level,          // 1–3
  invuln          // Invulnerability timer
}
```

### Enemy
```javascript
{
  x, y, w, h,           // Position & size
  speed, speedMul,      // Movement
  type,                 // 'grunt' | 'interceptor' | 'tank'
  hp, color, dark,      // Appearance & health
  score,                // Points on kill
  shootTimer,           // Fire rate
  canShoot, pattern,    // Behavior
  weave, weaveSeed      // Weaving movement
}
```

### Boss
```javascript
{
  x, y, w, h,       // Position & size
  hp, maxHp,        // Health
  design,           // Visual theme (colors, name)
  entering,         // Entry animation phase
  telegraph,        // Attack warning timer
  attackTimer,      // Cooldown between attacks
  patternIndex,     // Which attack pattern
  vx, phaseTimer    // Movement & timing
}
```

### Bullet
```javascript
{
  x, y, w, h,       // Position & size
  speed,            // Movement speed
  vx,               // Horizontal velocity (enemies)
  pierce,           // How many enemies laser penetrates
  laser             // True for laser weapon
}
```

### Particle
```javascript
{
  x, y,             // Position
  vx, vy,           // Velocity
  life, maxLife,    // Lifetime
  color             // Display color
}
```

## Constants & Configuration

- **`BASE_W`, `BASE_H`:** Logical canvas size (480×720)
- **`DIFFICULTIES`:** Three presets (easy, normal, hard) controlling spawn rates, lives, enemy speed
- **`ENEMY_TYPES`:** Grunt, Interceptor, Tank definitions
- **`BOSSES`:** Six boss designs with unique visuals
- **`THEMES`:** Six sky color schemes
- **`BOSS_PATTERNS`:** Attack pattern arrays for each boss (8 patterns × 6 bosses)
- **`KILLS_PER_STAGE`:** 18 enemies per stage before boss appears

## Development Notes

### Performance Considerations
- Canvas size kept small (480×720) for performance
- Particle effects quality scaled based on frame time
- Enemy and bullet counts managed dynamically
- No object pooling (arrays used directly, pushing/filtering)

### Mobile Optimization
- Viewport-fit covers notches and safe areas
- Touch-action prevents browser gestures
- Responsive font sizing with `clamp()`
- Dedicated touch control zones with visual feedback

### Accessibility
- Screen reader announcements via `aria-live` region
- Reduced motion support (respects prefers-reduced-motion, toggleable)
- High contrast UI elements
- Keyboard navigation support

### Browser Compatibility
- Modern ES6+ JavaScript
- Canvas 2D API
- WebAudio API
- LocalStorage for persistence
- Touch Events API for mobile
- No external dependencies

## Future Enhancement Ideas

- **Particle pool:** Object pooling for better GC performance
- **Level editor:** Custom stage design
- **Leaderboards:** Online score tracking
- **More bosses:** Expandable boss roster
- **Weapon varieties:** Additional gun types
- **Achievements:** Milestone tracking
- **Mobile app:** Wrap as PWA or native app
- **Sound options:** Volume slider, alternative sound packs
- **Performance profiling:** Detailed fps/memory monitoring

## Playing the Game

Simply open `index.html` in a modern web browser. No server, build process, or installation required.

**Scoring Tips:**
- Maintain kill streaks for higher multipliers
- Use bombs strategically on dense enemy waves
- Switch weapons based on situation (Vulcan for spread, Laser for focused damage)
- Collect power-ups for extended playtime
- Hard difficulty gives significantly higher scores

## Attribution

Developed as a self-contained HTML5 game project. Includes all game logic, rendering, and audio synthesis in a single file for maximum portability.

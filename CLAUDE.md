# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 web application. The game features:
- Player-controlled aircraft navigating through enemy squadrons
- Three difficulty levels (Easy, Normal, Hard) with dynamic spawn rates and enemy behavior
- Weapon upgrade system (Vulcan gun and Laser with level progression)
- Smart bomb mechanic for clearing screen threats
- Stage-based progression with boss encounters
- Score multiplier chains from consecutive hits
- Persistent best score tracking via localStorage
- Responsive touch controls for mobile devices
- Adaptive performance quality settings

## Development Setup

### Running the Game

Since this is a static HTML file, simply:
1. Open `index.html` directly in a modern web browser
2. The game runs immediately with no build process required
3. For local development with auto-reload, use any simple HTTP server:
   ```bash
   # Python 3
   python3 -m http.server 8000
   # Then visit http://localhost:8000
   
   # Or Node.js http-server
   npx http-server
   ```

### No Build Tools or Dependencies

The codebase has zero external dependencies. It uses:
- Vanilla HTML5/JavaScript (ES6+)
- Canvas 2D API for rendering
- Web Audio API for sound effects
- localStorage for persistence

## Code Architecture

The entire game is contained in `index.html` (~1669 lines) organized into these sections:

### 1. **HTML Structure & Styling** (Lines 1-420)
- Canvas element for game rendering
- HUD elements (score, lives, bombs, weapon status)
- Overlay UI (start screen, pause menu, game over screen)
- Touch control zones (joystick, fire button, bomb button)
- CSS for responsive layout, animations, and dark theme

### 2. **Game Constants** (Lines 463-520)
- `DIFFICULTIES`: Difficulty configs with spawn rates, enemy speed, enemy shoot chance, lives, bombs, boss HP multiplier
- `THEMES`: Six visual themes with sky gradients, cloud colors, sun colors
- `ENEMY_TYPES`: Grunt, Interceptor, Tank specs (speed, HP, size, behavior)
- `BOSSES`: Five boss configurations with patterns, HP, fire mechanics
- `KILLS_PER_STAGE`: Number of enemy kills (18) needed to trigger boss fight

### 3. **Audio & Sound Effects** (Lines 545-677)
The game uses Web Audio API to generate procedural sound effects:
- `ensureAudio()`: Initializes audio context on first user interaction
- `beep()`: Generates tones (shoot, UI clicks)
- `noiseBurst()`: Generates noise bursts (explosions, bombs)
- `sfx` object: Pre-configured SFX functions (shoot, laser, bomb, uiClick, etc.)

**Key audio patterns:**
- Shooting uses pitch-varied beeps
- Explosions use filtered noise bursts with fade-out
- UI clicks use quick beeps

### 4. **Game State** (Lines 753+)
Main game objects managed in the update/draw loop:
- `player`: { x, y, w, h, speed, weapon, level, cooldown, invuln, hp }
- `enemies`: Array of active enemies with type, position, velocity, HP
- `bullets`: Array of player projectiles with position, velocity, pierce, laser flag
- `enemyBullets`: Array of enemy fire
- `powerUps`: Array of weapon/life/bomb drops from killed enemies
- `particles`: Explosion effect particles
- `boss`: Current boss object with x, y, HP, pattern, cooldown, etc.

**Game state flags:**
- `running`, `paused`, `gameOver`: State machine
- `difficulty`: Current difficulty level
- `stage`, `kills`, `score`, `multiplier`: Progression tracking
- `player.invuln`: Invulnerability timer after taking damage
- `shakeTime`, `shakeMag`: Screen shake effect duration and magnitude

### 5. **Core Game Loop** (Lines 1640-1665)
```javascript
function loop(now) {
  // Calculate delta time, cap at 50ms
  // trackPerf(dt) for adaptive quality
  // if (running && !paused) { update(dt); draw(); }
  // requestAnimationFrame(loop)
}
```

The loop runs at ~60 FPS using `requestAnimationFrame`. It calls:
1. `update(dt)`: Process input, update positions, collisions, spawning
2. `draw()`: Render background, entities, effects

### 6. **Update Logic** (Lines 1148-1302)
`update(dt)` handles:
- Player movement (keyboard ARROW/WASD, mouse following, joystick)
- Weapon firing with cooldown
- Collision detection (bullets vs enemies, enemies vs player, power-ups)
- Enemy AI (movement, shooting)
- Boss patterns (radial fire, aimed spreads, column patterns)
- Enemy spawning based on difficulty and kill count
- Power-up effects and level progression
- Score multiplier decay when missing shots
- Stage transitions and boss fights

**Key mechanics:**
- Weapon cooldown differs by weapon type (laser 0.1s, vulcan 0.15s)
- Piercing bullets can hit multiple enemies (laser has pierce=2 at level 3)
- Collision uses rectangular overlap (`rectsOverlap()`)
- Chain multiplier increases with consecutive hits, resets on miss

### 7. **Rendering** (Lines 1507-1638)
`draw()` renders in layers:
1. Clear canvas
2. Sky gradient + sun glow
3. Scrolling clouds (parallax effect)
4. Player aircraft (with glow aura if laser weapon)
5. Bullets (laser vs vulcan styled differently)
6. Enemy bullets
7. Enemies with HP bars for tanks
8. Boss (if active)
9. Power-ups (rotating)
10. Explosion particles (fade out)

**Performance optimization:**
- `quality` variable (0=full glow, 1=medium, 2=low) adjusts shadow effects based on frame time
- Cloud rendering uses modulo math for infinite scrolling

### 8. **Input Handling**
- **Keyboard**: Arrow keys and WASD for movement, Space/X/Shift for weapons
- **Mouse**: Move to steer, click to fire
- **Joystick (mobile)**: Virtual joystick in lower-left, FIRE and BOMB buttons
- **Touch**: Drag across move zone for directional input

### 9. **Difficulty & Balancing**
The `DIFFICULTIES` object defines per-level tuning:
- **Easy**: Spawns every 75 frames, slower enemies (1.0x), 45% shoot chance, 4 lives
- **Normal**: Spawns every 58 frames, moderate speed (1.4x), 70% shoot chance, 3 lives
- **Hard**: Spawns every 40 frames, fast enemies (1.9x), 95% shoot chance, 3 lives

Boss HP scales with difficulty via `bossHpMul`.

### 10. **Weapon System**
Two weapon types with level progression:

**Vulcan** (default):
- Single stream at level 1
- Dual-lane spread at level 2
- Triple spread at level 3

**Laser** (from weapon power-up):
- Single stream at level 1
- Dual-lane at level 2
- Tri-lane (dual sides + center) at level 3
- Higher pierce, faster fire rate (0.1s vs 0.15s)

### 11. **Boss Patterns**
Five unique bosses with different fire patterns:
1. `fireRadial(b, count, speed, offset)`: Circular spray
2. `fireAimedSpread(b, count, speed, spreadStep)`: Aimed at player
3. `fireColumns(b, speed)`: Vertical columns

Bosses alternate between attacking and recovering. Kill all bosses to beat the game.

## Common Tasks

### Adding a New Enemy Type
1. Add config to `ENEMY_TYPES` object (speed, HP, size, color, dark color)
2. Update `spawnEnemy()` to include it in spawn pool
3. Define explosion SFX in `EXPLODE_SFX` if needed
4. Weapon system will automatically hit it in collision loop

### Adding a New Boss
1. Add config to `BOSSES` array with x, y, HP, pattern function
2. Boss patterns use helper functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`
3. Boss spawn happens when `kills >= KILLS_PER_STAGE`
4. Boss defeat triggers `killBoss()`, advances stage, resets spawn

### Adjusting Difficulty Tuning
1. Modify values in `DIFFICULTIES` object for spawn timing, speed, HP multipliers
2. Easy (1.0x speed/HP) → Normal (1.4x) → Hard (1.9x) progression
3. Test by running game and noting enemy behavior

### Changing Visual Theme
1. Add new theme object to `THEMES` array with sky gradients, cloud color, sun color
2. Theme auto-cycles per stage: `THEMES[(stage - 1) % THEMES.length]`
3. Colors use `rgb(r,g,b)` format for dynamic opacity

### Adding UI Polish
- `showComboPop(text)`: Display multiplier achievements
- `showStageBanner(text)`: Stage transitions
- `screenShake(mag, dur)`: Impact feedback
- `flashHit()`, `flashBomb()`: Visual feedback
- SFX calls provide audio feedback

## Storage & Persistence

- `localStorage` keys:
  - `skyfighter.bestScore`: Best score achieved
  - `skyfighter.muted`: Audio mute state (0/1)
  - `skyfighter.difficulty`: Last selected difficulty
  - `skyfighter.reducedMotion`: Motion preference (accessibility)

Load/save happens in `resetGame()` and during game updates.

## Performance Considerations

### Adaptive Quality System
The `quality` variable auto-adjusts visual effects:
- Tracks frame times over 45 frames
- If avg > 25ms (40 FPS), increase quality level
- If avg < 17.8ms (56 FPS), decrease quality level
- Quality affects: shadow glow on bullets, particle rendering

### Optimization Techniques
- Canvas clipping and `ctx.save()`/`ctx.restore()` for state isolation
- Modulo math for infinite cloud scrolling (no new elements)
- Single-pass collision detection
- Lazy particle cleanup (only particles with `life > 0` render)

## Browser Compatibility

- Requires: ES6+, Canvas 2D, Web Audio API, localStorage
- Tested on: Chrome, Firefox, Safari, Edge (modern versions)
- Mobile: Responsive to viewport, touch-optimized controls

## Debugging Tips

- Open browser DevTools console to see debug logs
- `keys` object tracks input state for debugging input handling
- Frame counter (`frame`) useful for timing checks
- Visual quality settings accessible via quality var
- Boss/enemy spawning tied to `stage` and `kills` counters

## Future Enhancement Areas

- Weapon balance tuning (damage, fire rate, spread patterns)
- Additional enemy types with unique behaviors
- More boss patterns or boss stages
- Particle effect polish
- Sound effect variations
- Screen size handling improvements
- Accessibility features beyond reduced motion flag

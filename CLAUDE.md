# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in vanilla JavaScript with no external dependencies. The game runs in a single `index.html` file and features:

- **Canvas-based rendering** with gradient backgrounds and particle effects
- **Web Audio API synthesis** for procedurally generated sound effects (no audio files)
- **Three difficulty levels** (Easy, Normal, Hard) with progressive enemy spawning
- **Six boss encounters** with unique visual designs and HP multipliers
- **Weapon system** with progression (basic gun → laser → spread shot)
- **Health and bomb system** for player survivability
- **Combo multiplier** based on consecutive kills
- **Touch and keyboard controls** with full mobile support
- **Local storage** for best score, difficulty, and settings persistence
- **Responsive design** supporting all screen sizes and notch/safe areas
- **Accessibility features** including reduced motion support and screen reader announcements

## Code Architecture

### Single-File Structure

The entire game is contained in `index.html` (~1670 lines) with three sections:

1. **HTML & CSS** (lines 1-421): DOM elements and styling
   - Game canvas and HUD elements
   - Control buttons (mute, motion reduction, pause, difficulty selection)
   - Touch control joystick and fire/bomb buttons
   - Game state overlays (start, game over, pause screen)

2. **Inline JavaScript** (lines 422-end): Game logic and rendering
   - Initialization and configuration objects
   - Audio synthesis engine
   - Game state management
   - Entity system (player, enemies, bosses, projectiles)
   - Canvas rendering and animation loop
   - Input handling (keyboard, touch, mouse)
   - Collision detection

### Key Game Objects & Constants

**Difficulty Settings** (DIFFICULTIES object)
- `easy`: 4 lives, 3 bombs, slower enemies, lower spawn rate
- `normal`: 3 lives, 2 bombs, balanced gameplay
- `hard`: 3 lives, 2 bombs, faster enemies, aggressive AI

**Enemy Types** (ENEMY_TYPES object)
- `grunt`: Basic red enemy, 1 HP, appears from stage 1
- `interceptor`: Fast orange enemy, 1 HP, erratic movement, appears from stage 1
- `tank`: Tanky purple enemy, 4 HP, slow, appears from stage 2

**Bosses** (BOSSES array)
- Six named bosses with distinct color schemes and HP multipliers
- Boss selection is stage-dependent (cycled through)
- Boss HP: `baseHP * difficulty.bossHpMul * boss.hpMul`

**Visual Themes** (THEMES array)
- Six environment themes with sky gradients, cloud colors, and sun colors
- Theme selection is stage-dependent (cycled through)

### Game Loop & Timing

- Uses `requestAnimationFrame` for 60 FPS target
- Delta time (`dt`) is calculated per frame
- Adaptive quality scaling when frame times exceed thresholds:
  - Quality 0 = full rendering (< 17.5ms per frame)
  - Quality 1 = medium (17.5-25ms per frame)
  - Quality 2 = low (> 25ms per frame)

### Entity System

**Player**
- Single player object managed by game state
- Weapon levels (0 = gun, 1 = laser, 2 = spread)
- Position bounded to canvas
- Collision detection with enemy projectiles

**Enemies**
- Spawned based on difficulty and stage
- Each enemy has: type, x, y, hp, vx, vy, shootCooldown, lastShot
- Movement: linear downward with slight weaving (interceptor type)
- AI: Random shoot chance modified by difficulty

**Projectiles**
- Player bullets: move upward, single owner (not pooled)
- Enemy bullets: move downward with slight spread
- Bombs: explode on trigger, damage all enemies in range

**Visual Effects**
- Combo popup animations
- Stage banner text
- Flash overlays for hits and bomb explosions
- Vignette effect (red pulsing) when health is low

### Audio System

Web Audio API synthesis with no pre-recorded assets. Each sound is procedurally generated:
- **Oscillators**: sine, square, sawtooth, triangle waves
- **Filters**: lowpass/highpass with frequency modulation
- **Envelopes**: gain curves for attack/release
- **Panning**: spatial audio based on screen position

Sound effects are defined in the `sfx` object:
- `shoot`, `laser`: Player weapon fire
- `explodeGrunt`, `explodeInterceptor`, `explodeTank`: Enemy destruction
- `hit`: Player takes damage
- `bomb`: Bomb explosion
- `powerup`: Weapon upgrade
- `bossHit`, `bossTelegraph`, `bossDown`: Boss-specific
- `stageClear`, `gameover`: End state
- `uiClick`: Button interactions

### Input Handling

**Keyboard**
- Arrow keys or WASD for movement
- Space for fire
- B for bomb
- P for pause
- C for combo multiplier view (debugging)

**Touch**
- Left zone: virtual joystick with stick tracking
- Right fire button: continuous fire
- Right bomb button: bomb deployment

**Mouse/Pointer**
- Canvas crosshair cursor
- Hover to aim

### Local Storage Keys

- `skyfighter.bestScore`: High score (number)
- `skyfighter.muted`: Mute state (0 or 1)
- `skyfighter.difficulty`: Selected difficulty (easy, normal, hard)
- `skyfighter.reducedMotion`: Motion reduction preference (0 or 1)

### State Management

Game state is tracked via:
- `gameState`: INIT, PLAYING, PAUSED, DEAD, BOSS (enum values)
- `score`, `multiplier`: Current session stats
- `stage`: Current stage number
- `kills`: Enemies killed in current stage
- `player`, `enemies`, `playerBullets`, `enemyBullets`: Entity arrays

## Development Workflow

### Making Changes

Since this is a single-file game, all modifications happen in `index.html`:

1. **For balance changes**: Modify DIFFICULTIES, ENEMY_TYPES, BOSSES, or THEMES constants
2. **For gameplay mechanics**: Update game loop logic (around line 1200+) or entity update functions
3. **For visual changes**: Edit CSS (lines 10-400) or canvas rendering code
4. **For audio**: Update `sfx` object or synthesizer functions (`beep`, `noiseBurst`)
5. **For UI**: Add/modify HTML elements and their event handlers

### Testing

The game requires:
1. **Human playtesting** - no automated test suite exists
2. **Cross-browser testing** - test on Chrome, Firefox, Safari, Edge
3. **Mobile testing** - test on iOS and Android with touch controls
4. **Accessibility testing** - verify with reduced motion enabled, test keyboard controls

### Performance Considerations

- Canvas rendering is the primary bottleneck
- Quality adjustment runs on a 45-frame rolling average
- Avoid adding new expensive visual effects without quality scaling
- Projectile count grows with difficulty and stage
- Boss fights can have intensive bullet patterns

### Common Tasks

**To add a new enemy type:**
1. Add entry to `ENEMY_TYPES` object with stats
2. Add spawn logic in stage progression
3. Add explosion sound to `EXPLODE_SFX` mapping
4. Test collision and damage interactions

**To add a new boss:**
1. Add entry to `BOSSES` array with colors and hpMul
2. Add unique pattern/movement in boss update section
3. Add telegraph and defeat sounds to `sfx` object
4. Test health scaling across difficulties

**To adjust difficulty:**
1. Modify DIFFICULTIES object values
2. Test progression pace (enemy spawn rate vs. kill requirements)
3. Verify boss health scaling is balanced

**To add visual theme:**
1. Add entry to THEMES array with sky gradient colors
2. Test on various backgrounds to ensure readability
3. Verify boss colors are visible on new theme

## Browser Compatibility

- Requires modern browser with Canvas 2D and Web Audio API support
- Tested on: Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
- Mobile: iOS Safari 14+, Android Chrome 90+
- Does not support: IE11 or older versions

## Notes for Future Development

- The game is intentionally self-contained in a single file for easy distribution
- Avoid external dependencies (no npm packages, no external assets)
- Audio synthesis requires user interaction (first click) to initialize AudioContext
- Touch controls scale based on viewport with clamp() for responsive design
- Combo system and stage themes cycle deterministically for consistency
- Boss HP calculation: `(140 + stage * 20) * difficultyMul * bossMul` (approximate)

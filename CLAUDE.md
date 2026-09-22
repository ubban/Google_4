# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. It's a single-file HTML5 game with no build process, no external dependencies, and no tests. The entire game is implemented in a self-contained `index.html` file (~1670 lines) using:
- Canvas 2D rendering for graphics
- Web Audio API for procedurally-generated sound effects
- LocalStorage for persistence (best score, difficulty, settings)
- Touch and keyboard input handling

## Running the Game

Simply open `index.html` in any modern web browser. The game runs completely in the browser with no server or build step required.

- **Desktop**: Click to fire, mouse movement to steer, keyboard (Arrows/WASD) to move, Space to fire, X or Shift to bomb, P or Escape to pause
- **Mobile**: Touch-based controls with virtual buttons (Joystick on left, Fire/Bomb buttons on right)

## Code Architecture

The game is implemented as a single IIFE (Immediately Invoked Function Expression) containing the entire game engine. Key sections:

### Game State & Setup
- **Constants**: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS` define game content
- **Game State Variables**: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`, etc.
- **Configuration**: LocalStorage keys for persistence (best score, mute state, difficulty, reduced motion preference)

### Core Systems

**Audio System** (`ensureAudio()`, `beep()`, `noiseBurst()`, `sfx`)
- Web Audio API wrapper for synth-based sound generation
- Procedural sound effects (no audio files)
- Panning and filter support
- Master gain control with compression

**Input Handling**
- Keyboard: Arrow keys, WASD, Space, X, Shift, P/Escape
- Mouse: Position tracking, click-to-fire
- Touch: Joystick-based movement, button taps
- Input priority: keyboard/joystick > mouse steering (prevents conflicts)

**Game Loop** (`loop()`)
- RequestAnimationFrame-based 60fps loop
- Performance tracking with adaptive quality reduction (3 quality levels)
- Sequence: update → draw → request next frame
- Capped delta-time at 50ms to prevent large time jumps

**Entity Updates** (`update()`)
- Player movement and weapon firing
- Enemy spawning, movement, and shooting patterns
- Bullet lifecycle and collision detection
- Power-up physics
- Boss state machine updates
- Particle animation
- Chain/multiplier system

**Rendering** (`draw()`)
- Canvas clear and background gradient with sun and cloud layers
- Theme-based sky colors that change per stage
- Entity drawing (player, enemies, boss, bullets, particles, power-ups)
- Screen shake effects
- Screen flash on hits/bombs
- HUD indicators and overlays

**Collision Detection** (`rectsOverlap()`)
- AABB (axis-aligned bounding box) collision for all entities
- Enemy-bullet, boss-bullet, player-damage interactions
- Power-up collection

**Boss System**
- 6 unique boss designs with visual variations
- 6 different attack patterns per boss (radial, aimed, homing, spirals, etc.)
- Telegraph system (visual warning before attacks)
- Boss HP bar UI

### Key Functions

- `spawnEnemy()` - Random enemy type with stage-based progression
- `spawnBoss()` - Triggers boss encounter after 18 kills
- `fireWeapon()` - Weapon system with Vulcan (spread) and Laser (pierce) modes
- `useBomb()` - Smart bomb clears bullets and damages all enemies
- `addScore()` - Score with multiplier chain system
- `updateBoss()` - Boss movement and attack pattern sequencing
- `applyPowerUp()` - Life, bomb, weapon upgrades
- `resetGame()` - Full game state reset for new run
- `endGame()` - Game over handling and best score tracking
- `togglePause()` - Pause/resume functionality

## Difficulty Levels

Game balance is driven by the `DIFFICULTIES` object:
- **Easy**: Slower spawns, weaker enemies, 4 lives, 1.0x boss HP
- **Normal**: Balanced (default)
- **Hard**: Fast spawns, stronger enemies, 1.3x boss HP

Difficulty is stored in LocalStorage and affects spawn rates, enemy behavior, and boss scaling.

## Stages & Progression

- **Stages**: Game progresses through infinite stages (theme and boss cycle every 6 stages)
- **KILLS_PER_STAGE**: 18 kills triggers boss encounter
- **Themes**: 6 visual themes cycle as you progress (sky gradients, cloud colors, sun colors)
- **Bosses**: 6 boss types cycle with different attack patterns
- **Multiplier**: Chain counter increases for consecutive kills; breaks if you take damage

## Accessibility Features

- **Reduced Motion**: Disable screen shake, flashes, and animations (respects system prefers-reduced-motion)
- **Screen Reader**: `srLive` element with aria-live for announcements
- **Touch Optimizations**: Safe-area insets (notch/island support), max-scale viewport limits

## Performance Considerations

- **Quality Scaling**: Adaptive quality adjustment based on frame times
  - Level 0 (full): Glow effects on bullets, shadows
  - Level 1 (medium): Reduced particles (60% count)
  - Level 2 (low): Minimal particles (35% count)
- **Particle Budget**: Enemy explosions reduce particle count at lower quality
- **Glow Effects**: Shadow blur adjusted per quality level

## LocalStorage Keys

- `skyfighter.bestScore` - Best score (number)
- `skyfighter.muted` - Audio mute state ('0' or '1')
- `skyfighter.difficulty` - Selected difficulty ('easy', 'normal', 'hard')
- `skyfighter.reducedMotion` - Reduced motion preference ('0' or '1')

## Common Modifications

To adjust game balance:
1. **Difficulty scaling**: Edit `DIFFICULTIES` object
2. **Enemy types & stats**: Modify `ENEMY_TYPES` object
3. **Boss patterns**: Update `BOSS_PATTERNS` array
4. **Spawn rates**: Change `spawnBase` and `spawnMin` in `DIFFICULTIES`
5. **Player speed/cooldown**: Adjust in `resetGame()` (player object)
6. **Sound effects**: Modify `sfx` object functions (frequency, duration, envelope)
7. **Visual themes**: Add/modify entries in `THEMES` array
8. **Screen shake intensity**: Call `screenShake()` with different magnitude/duration

## Testing & Validation

- No automated tests
- Manual testing required: all difficulty levels, all bosses, mobile touch controls, audio, reduced motion mode
- Browser DevTools Console for errors
- Test on different screen sizes (responsive design uses viewport-fit and safe-area-inset)

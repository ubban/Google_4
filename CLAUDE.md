# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a complete, self-contained browser-based Raiden-style vertical shoot-'em-up game. The entire game is implemented in a single `index.html` file (~1670 lines) containing:
- Embedded CSS for all styling
- Vanilla JavaScript game engine with no external dependencies
- WebAudio synthesis for procedurally generated sound effects
- Canvas-based 2D rendering

**Key Features:**
- Three weapon types with upgrade progression (Vulcan and Laser)
- Multiple enemy types with distinct behaviors (grunts, interceptors, tanks)
- Six unique boss encounters with pattern-based attack systems
- Procedurally varied environments with different visual themes
- Multiplier/combo system for score calculation
- Three difficulty levels affecting enemy spawn rates, speed, and behavior
- Touch controls for mobile, keyboard/mouse for desktop
- Accessibility: reduced motion preference, screen reader announcements
- Local storage persistence for best score and user preferences

## Architecture

### Game Loop (`loop` function, line ~1640)
The core game loop runs at 60fps via `requestAnimationFrame`:
1. Track delta time and adaptive performance
2. Call `update(dt)` if game is running and not paused
3. Call `draw()` to render the current frame
4. Continue recursively

### Game State
Key state variables:
- `player`: Current position, weapon, level, health, invulnerability timer
- `enemies`: Array of active enemies with position, type, HP, behavior
- `boss`: Boss encounter object (null when no boss active)
- `bullets`: Player projectiles
- `enemyBullets`: Enemy/boss projectiles
- `powerUps`: Collectible items (life, bombs, weapon upgrades)
- `particles`: Explosion/visual effect particles
- `score`, `lives`, `bombCount`, `stage`, `chain`, `multiplier`: Game metrics
- `running`, `paused`, `gameOver`: Game state flags

### Core Systems

#### Input System (lines ~768-861)
- **Keyboard**: WASD/Arrows for movement, Space to fire, X/Shift for bomb, P/Esc to pause
- **Mouse**: Move to steer, click to fire
- **Touch**: Joystick on left (movement), Fire/Bomb buttons on right
- Mouse and keyboard input compete (keyboard takes priority when active)

#### Weapon System (lines ~871-1145)
Two weapon types, each with 3 upgrade levels:
- **Vulcan**: Spread shot that increases coverage with each level
- **Laser**: Piercing shots that ignore some enemies with each level
- Weapon powerups cycle between types and reset level progression
- Firing cooldown differs per weapon (0.15s vulcan, 0.1s laser)

#### Enemy System (lines ~1022-1041)
Three enemy types defined in `ENEMY_TYPES`:
- **Grunt** (red): Basic enemy, appears from stage 1
- **Interceptor** (orange): Fast weaver, appears from stage 1
- **Tank** (purple): Tanky, slow, appears from stage 2+
- Spawning rate scales with difficulty and frame count (adaptive difficulty ramp)
- Enemies spawn at random x-positions and move downward
- Weaver enemies follow sine-wave paths around a base x-position

#### Boss System (lines ~1043-1409)
Six unique bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN):
- Each boss has unique visual design and attack patterns
- Boss health scales with stage and difficulty
- `BOSS_PATTERNS` array defines 3 attack patterns per boss (cyclical)
- Boss telegraph phase (red core pulse) before each attack
- Attacks use helper functions: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`

#### Stage/Progression System (lines ~491, ~1204-1452)
- After 18 enemy kills, boss spawns after 1.2 second warning
- Each stage increments, visual theme changes (6 themes cycle)
- Boss defeat triggers stage transition and new environment theme
- `KILLS_PER_STAGE` constant controls enemy count before boss

#### Audio System (lines ~542-676)
WebAudio synth with no external audio files:
- `beep()` function: frequency sweep, type, envelope, optional filter and pan
- `noiseBurst()` function: procedural noise with filter
- `sfx` object contains named sound functions (shoot, laser, explode variants, hit, bomb, powerup, boss sounds, etc.)
- Adaptive pan based on x-position for stereo field effect
- Mutable via icon button, preference persisted in localStorage

#### Collision System (lines ~1069-1294)
- `rectsOverlap()` performs AABB collision detection
- Player-to-enemy: lose life on contact
- Player-to-enemy-bullet: lose life on contact
- Player-to-powerup: collect and apply effect
- Bullet-to-enemy: damage enemy, potentially destroy bullet (laser pierces)
- Bullet-to-boss: 3 damage per hit (vs 1 per hit for enemies)
- Bomb usage: clears all enemy bullets, damages all enemies, damages boss by 24

#### Visual Effects & Accessibility (lines ~698-730)
- `screenShake()`: Camera offset effect (disabled in reduced-motion mode)
- `flashHit()` / `flashBomb()`: White flash animation on damage/bomb
- `showComboPop()`: Animated combo milestone text at top center
- `showStageBanner()`: Stage transition banner with animation
- Reduced motion preference respected via CSS media query and class toggles
- Screen reader announcements via `announce()` function

#### Input Priority & Mouse Steering (lines ~836-861)
The game supports mixed input modes without conflict:
- Keyboard/joystick always takes priority over mouse
- Mouse serves as fallback steering when no keyboard input present
- Player smoothly glides toward mouse position when mouseActive and no directional keys pressed

### Key Constants

```javascript
BASE_W = 480, BASE_H = 720          // Canvas logical size (scaled to fit window)
KILLS_PER_STAGE = 18                // Enemy kills before boss appears
DIFFICULTIES = { easy, normal, hard }  // Spawn rates, enemy speed, lives, bombs
THEMES                               // 6 environmental themes (sky gradients, colors)
ENEMY_TYPES                          // 3 enemy types with stats
BOSSES                               // 6 boss designs with colors, HP multipliers
BOSS_PATTERNS                        // 3 attack patterns per boss (6 sets total)
EXPLODE_SFX                          // Type-specific explosion sound effect keys
```

### Storage Keys
Persistent state in localStorage:
- `skyfighter.bestScore`: Best score ever achieved
- `skyfighter.muted`: Audio mute state (1 = muted)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Motion preference (1 = reduced)

## Development Workflow

### Running the Game
Simply open `index.html` in a web browser. The game is fully self-contained with no build step, dependencies, or server required.

**Development server (optional):**
```bash
python3 -m http.server 8000
# Then open http://localhost:8000/index.html
```

### Testing/Debugging

**Common debugging approaches:**
1. Open browser DevTools (F12) → Console to see errors
2. Add `console.log()` calls in update/draw loops (watch performance impact)
3. Modify `DIFFICULTIES` to test balance changes
4. Change `stage` variable to jump to specific stages
5. Modify `lives` and `bombCount` to test damage/bomb mechanics
6. Set `boss = null` to skip boss and force next stage
7. Adjust `BASE_W` / `BASE_H` to test responsive scaling

**Testing Quality Levels:**
The adaptive performance system (`quality` variable) throttles particles and effects:
- Quality 0: Full effects (60fps capable)
- Quality 1: Medium (40fps on budget devices)
- Quality 2: Low (30fps minimum)

Adjust the `trackPerf()` thresholds if performance tuning is needed.

### Code Organization

The codebase is organized linearly by functional area within the IIFE (Immediately Invoked Function Expression):

1. **DOM Setup & Configuration** (~lines 426-527): Element references, storage keys, defaults
2. **Helper Functions** (~lines 468-731): Audio, rendering, visual effects
3. **Game State** (~lines 863-911): Global state variables and resetGame()
4. **Drawing/Rendering** (~lines 913-1020): drawPlane, drawBossShip, etc.
5. **Spawn Logic** (~lines 1022-1067): spawnEnemy, spawnBoss, maybeSpawnPowerUp
6. **Collision & Scoring** (~lines 1069-1101): Hit detection, damage, score
7. **Game Mechanics** (~lines 1103-1322): Bomb, weapon firing, powerups
8. **Boss Attack Patterns** (~lines 1324-1409): Pattern functions and BOSS_PATTERNS array
9. **Update Logic** (~lines 1148-1302): Main game update, physics, collision checks
10. **Game State Transitions** (~lines 1411-1504): Boss updates, boss defeat, game over, pause
11. **Canvas Rendering** (~lines 1507-1638): Sky, clouds, objects, particles
12. **Main Loop & Event Listeners** (~lines 1640-1666): requestAnimationFrame, button handlers

### Making Changes

**Adding a New Enemy Type:**
1. Add entry to `ENEMY_TYPES` object with stats (width, height, HP, speed, color, score, etc.)
2. Update `spawnEnemy()` to include new type in random selection (check `minStage` for progression)
3. Add explosion SFX to `EXPLODE_SFX` if unique (optional)
4. Test spawning and collision

**Adding a New Weapon:**
1. Add weapon name to `weaponLabel()` function return
2. Update player weapon initialization in `resetGame()`
3. Modify `fireWeapon()` to generate appropriate bullet spread
4. Adjust cooldown in update logic if needed

**Modifying Boss Patterns:**
1. Create new helper function (e.g., `fireNewPattern`) using existing helpers like `fireRadial` or `fireAimedSpread`
2. Add pattern to appropriate boss's pattern array in `BOSS_PATTERNS`
3. Test timing and difficulty balance

**Adjusting Difficulty:**
Edit the `DIFFICULTIES` object:
- `spawnBase`: Initial frame delay between spawns (lower = faster)
- `spawnMin`: Minimum spawn delay (floor as difficulty ramps)
- `enemySpeed`: Movement speed multiplier
- `enemyShootChance`: Probability enemy shoots when it should
- `lives`: Starting lives
- `bombs`: Starting bombs
- `bossHpMul`: Boss health multiplier

**Responsive Layout Issues:**
- Canvas scaling logic is in `resize()` function (lines ~752-765)
- HUD elements use `clamp()` CSS for responsive font sizing
- Touch control layout is CSS-based (lines ~291-341)
- Safe area insets respect notches: `env(safe-area-inset-*)`

## Key Decision Points & Patterns

### Why One HTML File?
- **Pros**: No dependencies, no build step, instant delivery, trivial hosting, easy to share
- **Cons**: No code splitting, limited modularity, larger initial file
- This trade-off is intentional for a completed game project

### Input Handling Priority
Keyboard/joystick > Mouse steering — prevents mouse from fighting keyboard input during gameplay

### WebAudio Synthesis
All sounds are generated procedurally on-the-fly. This avoids audio asset management but requires careful tuning of frequencies and envelopes. The synthesizer uses oscillators (sine, square, sawtooth, triangle) and noise bursts with filters and panning.

### Adaptive Quality System
Performance detection and quality reduction ensure playable framerates even on budget devices. The `trackPerf()` system monitors average frame time and adjusts the `quality` variable, which scales particle counts and glow effects.

### Stage/Boss Progression
The game uses a simple loop: spawn 18 enemies → boss fight → next stage. Boss difficulty increases with stage via scaling health and later bosses having more complex patterns.

### Collision AABB (Axis-Aligned Bounding Box)
Fast, simple collision using center-to-center distance on rectangular objects. Adequate for this art style and no need for pixel-perfect hit detection.

## Performance Considerations

- Canvas is rendered at logical size (480×720) then scaled via CSS for crisp rendering on any device
- Particles are capped based on `quality` level to maintain 60fps
- Enemy spawn rate auto-throttles when frame count exceeds threshold to prevent difficulty spikes
- Audio context is lazy-initialized on first user input (reduces startup overhead)
- Shader-less rendering uses canvas 2D API (broad device support, lower power on mobile)

## Known Limitations & Future Enhancements

**Current Limitations:**
- Single-player only (no multiplayer)
- No level select (linear progression)
- Limited sound variety (6 bosses share pattern functions, some audio effects reused)
- Procedural audio can be tinny on poor speakers

**Possible Enhancements:**
- Additional enemy types with unique movement patterns
- Weapon types beyond Vulcan/Laser (spread, homing, etc.)
- Score-based unlockables or achievements
- Leaderboard integration (with backend)
- Mobile app wrapper (Cordova/Capacitor)
- Accessibility: Controller gamepad support
- Advanced visual: WebGL shaders for more effects
- Procedural stage generation instead of fixed enemy spawn patterns

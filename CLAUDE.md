# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. The entire game is implemented in a single HTML file (`index.html`) with embedded CSS and JavaScript (~1700 lines). It features:

- Multiple enemy types with distinct behaviors
- Six procedurally unique boss encounters
- Dual weapon system (Vulcan spread-shot and Laser piercing shots)
- Progressive difficulty scaling across stages
- Web Audio API synthesized sound effects (no external assets)
- Responsive canvas-based graphics with dynamic themes
- Adaptive performance with 3-tier quality scaling
- Full keyboard, mouse, and touch input support
- LocalStorage persistence for scores and preferences

## How to Run

Open `index.html` in a modern web browser. The game is entirely self-contained and requires no build step or dependencies.

## Code Architecture

The codebase is organized as one long IIFE (Immediately Invoked Function Expression) to maintain a local scope. Key sections in execution order:

### 1. **DOM Caching & Constants** (lines 426-529)
   - Cache all DOM element references
   - Define storage keys, difficulty presets, enemy types, boss designs, and color themes
   - Initialize persistent state from localStorage

### 2. **Audio System** (lines 542-676)
   - `ensureAudio()`: Lazy-initialize Web Audio context on first interaction
   - `beep()` and `noiseBurst()`: Synthesize tones and noise bursts with optional filters and panning
   - `panFor()`: Calculate stereo pan based on screen position for spatial audio
   - `sfx` object: Map of all sound effects (shoot, laser, explode, hit, bomb, boss patterns, etc.)
   - Audio context is suspended until user interaction; mute state is respected

### 3. **Input Handling** (lines 767-862)
   - **Keyboard**: Arrow keys, WASD, Space (fire), X/Shift (bomb), P/Esc (pause)
   - **Mouse**: Hover to steer, click to fire; tracked via `mouseTarget` and `mouseActive`
   - **Touch**: Dual-zone layout (left 55% for movement joystick, right 45% for fire/bomb buttons)
   - **Priority**: Keyboard/joystick input takes priority over mouse to prevent fighting
   - Fire button triggers immediately; bomb requires button press

### 4. **Game State** (lines 863-912)
   - `player`: Position, size, weapon, level, cooldown, invulnerability, health
   - `enemies`, `bullets`, `enemyBullets`, `powerUps`, `particles`: Array-based entity pools
   - `boss`: Current boss or null; when present, spawns after killing enough regular enemies
   - `score`, `lives`, `stage`, `kills`, `stageKills`: Progression tracking
   - `chain`, `chainTimer`, `multiplier`: Combo system (breaks after 2.2 seconds without kills)
   - `resetGame()`: Initialize all state and start a new game

### 5. **Rendering** (lines 913-1020, 1520-1638)
   - **Canvas setup**: Fixed 480×720 logical pixels, scales to fit window
   - **Coordinate system**: (0,0) at top-left; y increases downward
   - **drawPlane()**: Renders player and enemy ships with a specific visual style
   - **drawBossShip()**: Renders boss with design-specific colors and animations
   - **draw()**: Main render loop that clears, draws background + parallax clouds, entities, particles
   - **Screen shake**: Applies random translation when damage is taken
   - **Quality scaling**: Skips shadow blur and reduces particle counts on low-quality settings

### 6. **Game Loop** (lines 1640-1665)
   - `loop()`: RequestAnimationFrame-driven update loop
   - Delta-time clamped to 0.05s to prevent spiral-of-death scenarios
   - Calls `update()` and `draw()` each frame when game is running and not paused
   - Tracks performance via `trackPerf()` and adjusts quality level

### 7. **Update Logic** (lines 1148-1302)
   - **Player input**: Reads from keys, joystick, or mouse target
   - **Weapon fire**: Cooldown-gated; different spread patterns per weapon type and level
   - **Bullet updates**: Move in their direction (with optional velocity spread for vulcan)
   - **Enemy spawning**: Adaptive frequency based on difficulty and frame count
   - **Enemy behavior**: Linear downward movement + optional weave pattern + shoot logic
   - **Boss updates**: Delegated to `updateBoss()` for complex patterns
   - **Collision detection**: `rectsOverlap()` checks enemy/player, bullets/enemies, power-ups/player
   - **Enemy death**: Spawns explosion particles, power-up, adds score with multiplier
   - **Stage progression**: After 18 kills, trigger boss spawn; after boss death, advance stage
   - **Chain mechanics**: Reset multiplier on miss; max 10× at 50 consecutive kills

### 8. **Boss System** (lines 1372-1454)
   - **Designs**: 6 unique visual designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
   - **HP scaling**: `(70 + stage * 35) * difficultyMul * designMul`; harder difficulties increase HP
   - **Attack patterns**: Each boss has 3 unique patterns that rotate; telegraph warning before each
   - **Pattern types**: Radial bursts, aimed spreads, teleport + attack, double rings, spiral bursts, homing missiles
   - **Entering animation**: Boss slides down from top when spawned
   - **Movement**: Bounces left-right within screen bounds
   - **Telegraph system**: `telegraph` timer counts down; when it hits zero, fire the pattern

### 9. **Difficulty System** (lines 485-489)
   - Presets for Easy, Normal, Hard with different spawn rates, enemy speeds, enemy accuracy, boss HP
   - Affects enemy behavior, game pacing, and reward scaling
   - Persistent via localStorage

### 10. **Accessibility & Preferences** (lines 520-539, 697-703)
   - **Reduced motion**: Respects OS preference; toggles via button; disables animations/shake/flash
   - **Screen reader support**: `announce()` function sends live-region updates for game events
   - **Audio mute**: Toggle button; respects across all sound effects
   - **Visual feedback**: Flash on hit/bomb, vignette pulsing when critically low health

## Important Implementation Details

### Weapon System
- **Vulcan** (default): Spread-shot pattern expands with level (1→2→3 bullets); fast fire rate (0.15s)
- **Laser**: Piercing beam that fires faster (0.1s); level unlocks extra lanes or beam count
- Weapon pickups cycle through levels and switch between weapons at level 3

### Enemy Spawning
- Spawn frequency decreases as the frame count increases (difficulty ramp-up over time)
- Enemy types depend on stage; tanks only appear in stage 2+
- Weaving enemies follow a sine-wave pattern relative to their spawn X
- All enemies despawn after leaving the bottom of the screen

### Performance Optimization
- **Quality levels**: 0 (full), 1 (medium, 60% particles), 2 (low, 35% particles)
- Auto-scales based on average frame time over 45 frames
- Shadow blur disabled on lower quality levels
- Particles culled aggressively on low quality

### Power-Up Drop Mechanics
- 6% chance for life, 8% for bomb, 20% for weapon; cumulative probabilities
- Only spawn when enemies die, not on boss death
- Fall straight down at a fixed speed; despawn after leaving screen

### Chain Multiplier
- Increases by 1 for every 5 consecutive kills; caps at 10× (50+ kills)
- Resets on player damage or 2.2 second timeout
- Combo pop display at 10, 20, 30 kills

## Extending the Game

### Adding a New Boss
1. Add entry to `BOSSES` array with `name`, hull colors, wing color, core color, and `hpMul` multiplier
2. Create attack pattern functions and add to `BOSS_PATTERNS` array at the same index
3. Boss design is selected via `(stage - 1) % BOSSES.length` (cycles through designs)

### Adding a New Enemy Type
1. Add to `ENEMY_TYPES` with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Optional: set `weave: true` for sine-wave movement
3. Adjust spawn logic in `spawnEnemy()` if new type should appear at different rarities

### Adding a New Theme/Background
1. Add entry to `THEMES` array with `sky` (3-stop gradient), `cloud` RGB, `sun` RGB, and display `name`
2. Theme rotates per stage: `THEMES[(stage - 1) % THEMES.length]`

### Modifying Sound Effects
- All sounds are procedurally generated via `beep()` (oscillator + optional filter + pan) and `noiseBurst()` (filtered noise)
- Add new sound by defining a function in the `sfx` object that calls `beep()` and/or `noiseBurst()`
- Parameters: frequency (Hz), duration (seconds), waveform type, volume, optional opts (sweep target, filter, pan, delay)

### Balancing Difficulty
- Edit `DIFFICULTIES` object to tweak spawn rates, enemy speed multipliers, shoot chance, lives, bombs, boss HP
- Edit `ENEMY_TYPES` properties to change individual enemy threat level
- Edit `BOSS_PATTERNS` to change attack density and player skill requirement

## Common Patterns

### Entity Management
- Entities stored in arrays: `enemies`, `bullets`, `enemyBullets`, `powerUps`, `particles`
- Update phase modifies state; cleanup phase filters out dead entities
- Collision checks iterate in reverse (`for (let i = array.length - 1; i >= 0; i--)`) to safely splice

### Animation Timing
- `frame` counter increments every update; used for repeating patterns (e.g., `frame * 0.05 + seed` for weaving)
- CSS animations use `animation` directive; reduced motion disabled via `body.reduced-motion` class toggle

### Canvas Rendering
- Always save/restore context state (`ctx.save()` / `ctx.restore()`) around transformation or style changes
- Particle opacity: `(current_life / max_life)` for fade-out effect
- Boss telegraph: sine wave on core to indicate danger

### State Persistence
- Best score, difficulty, mute state, reduced motion preference stored in localStorage
- Keys: `skyfighter.bestScore`, `skyfighter.difficulty`, `skyfighter.muted`, `skyfighter.reducedMotion`
- Read on startup; updated whenever settings change

## Testing Tips

- Open browser DevTools to monitor frame rate and performance
- Test on mobile by opening in responsive design mode; verify touch controls work
- Test audio via DevTools; ensure no context errors if sound fails to initialize
- Verify reduced-motion preference: inspect HTML element for `reduced-motion` class toggle
- Check localStorage persistence by clearing storage and verifying scores reset
- Test difficulty ramp: play to stage 5+ and observe enemy spawning accelerate

## Known Limitations

- Single canvas context; no layer system for advanced effects
- Collision is axis-aligned rect overlap; no pixel-perfect detection
- Boss patterns are hardcoded per design; no procedural generation
- Enemy AI is entirely movement + timing patterns; no pathfinding
- No pause menu options (only pause/resume overlay)
- No leaderboard or cloud save

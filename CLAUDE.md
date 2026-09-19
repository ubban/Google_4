# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML application. The game features multiple difficulty levels, weapon upgrades, boss fights, and a scoring/combo system. It requires no build step—simply open `index.html` in a browser to play.

## Running the Game

- **Development**: Open `index.html` in a modern web browser (Chrome, Firefox, Safari, Edge)
- **Testing**: Open in multiple browsers and test across desktop and mobile (touch controls)
- **No build required**: All code is embedded in the single HTML file

## Architecture Overview

The entire game is written in vanilla JavaScript within a single `<script>` tag in `index.html`. The architecture follows a traditional game loop pattern:

### Core Game Loop
- **Entry**: `requestAnimationFrame(loop)` runs at ~60fps
- **Update phase**: `update(dt)` handles input, collision detection, game state
- **Draw phase**: `draw()` renders all graphics to canvas
- **Performance tracking**: Adaptive quality system adjusts particle/effect density based on frame timing

### Game State Management
Global variables manage game state:
- `player`: Player object with position, speed, weapon, health
- `enemies`: Array of active enemies
- `boss`: Single boss object (null when no boss active)
- `bullets`, `enemyBullets`, `particles`, `powerUps`: Pooled arrays
- Score, lives, stage, kills tracking

### Input Handling
- **Keyboard/Mouse**: Arrow keys or WASD to move, Space to fire, X/Shift for bombs
- **Pointer**: Mouse hovers to aim, clicks to fire
- **Touch**: Left zone is joystick, right zone has FIRE/BOMB buttons
- Input state checked per frame in `update()` to avoid lag

### Rendering Pipeline
1. Clear canvas
2. Apply camera shake (if active)
3. Draw sky gradient (theme-based) and background
4. Draw clouds at multiple layers
5. Draw player, enemies, bullets, particles, effects
6. Apply screen shake by modifying canvas transform

## Key Systems

### Difficulty System
Three difficulty presets (Easy/Normal/Hard) control:
- Enemy spawn rate (`spawnBase`, `spawnMin`)
- Enemy speed and shoot chance
- Player lives and bomb count
- Boss HP multiplier

Difficulty is persisted to localStorage and selectable before each game.

### Enemy System
Three enemy types with distinct behaviors:
- **Grunt**: Basic red fighter, fast spawning
- **Interceptor**: Faster red unit with weaving movement pattern
- **Tank**: Slow purple unit with 4 HP, appears at stage 2+

Enemy spawning:
- Base spawn timer decreases as game progresses
- Enemy type selection weighted by difficulty and stage
- Enemies follow downward movement with optional weaving
- Shoot patterns: spread (3-bullet fan) or aimed (tracking shot toward player)

### Boss System
6 unique boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN), each with:
- Unique visual design (colors, core glow)
- Custom HP multiplier
- 3-attack pattern set (different bullet formations)

Boss phases:
1. **Entering**: Slides down from top (can't shoot)
2. **Active**: Moves left-right, telegraphs attacks
3. **Telegraph phase**: Shows warning (22 frames), plays alert sound
4. **Attack phase**: Fires one pattern from current set, rotates to next

Pattern types include: radial bursts, aimed spreads, columns, spirals, homing, double rings, teleport+attack combinations.

### Weapon System
Two weapons with 3 upgrade levels each:
- **Vulcan**: Spreads to multiple lanes at level 3, fast fire rate
- **Laser**: Pierces enemies (scales with level), medium fire rate

Weapon powerups cycle: Vulcan→Laser (resets level), or level up within current weapon.

### Scoring & Combo System
- Base points per enemy type: Grunt=10, Interceptor=15, Tank=35, Boss=500
- **Chain system**: Each kill extends a 2.2s chain window; new kills add to counter
- **Multiplier**: Increases from 1x (0 chain) to 10x (50+ chain)
- Score updates in real-time; combo visual pops at milestone kills

### Audio System
Pure WebAudio synthesis (no audio files):
- `beep()`: Oscillator-based tones with frequency sweep, filtering, panning
- `noiseBurst()`: White noise with envelope
- Sound effects: shoot, laser, various explosion types (enemy/tank/boss), hit, bomb, powerup, stage clear, gameover
- Master gain and compressor for volume control
- Mute state persisted to localStorage

### Effects & Polish
- **Screen shake**: Horizontal random jitter for impact moments (configurable magnitude/duration)
- **Flash effects**: White screen flash on hit or bomb detonation
- **Vignette pulse**: Red pulsing vignette when at 1 life (critical state)
- **Combo pop**: Yellow text animation when reaching 10/20/30... kills
- **Stage banner**: Appears on stage transition
- **Boss health bar**: Positioned top-center, tracks boss HP in real-time
- **Reduced motion support**: Respects `prefers-reduced-motion` media query, scales animations/shakes accordingly

### Theme System
6 visual themes (one per stage, repeating):
- Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge
- Each defines sky gradient, cloud color, sun glow colors
- Themes cycle through stages to provide visual variety

### Responsive Design
- Base canvas: 480×720 pixels
- Scales up to fit screen (max 1.6× scale) while maintaining aspect ratio
- Safe area insets respected for notched devices
- Touch controls enabled only on touch-capable devices
- HUD scales with viewport size using `clamp()`

## Important Implementation Details

### Collision Detection
Simple AABB (axis-aligned bounding box) overlap:
```js
function rectsOverlap(a, b) {
  return Math.abs(a.x - b.x) < (a.w + b.w) / 2 && Math.abs(a.y - b.y) < (a.h + b.h) / 2;
}
```
Used for: player↔enemies, player↔bullets, player↔powerups, bullets↔enemies, bullets↔boss

### Drawing Primitives
- **Planes**: Custom `drawPlane()` function creates asymmetric fighter silhouettes
- **Boss**: `drawBossShip()` renders multi-part boss with rotating engine details
- **Particles**: Simple circles with alpha fade
- **UI**: CSS for HUD, overlay cards; canvas for gameplay graphics

### Performance Optimization
- Adaptive quality (`quality` var: 0=full, 1=medium, 2=low) adjusts:
  - Particle count on explosions
  - Shadow blur on projectiles/core glow
- Particle/bullet arrays filtered to remove off-screen objects
- Canvas drawing batched (all bullets at once, all enemies at once)
- Frame time tracking to monitor performance

### Browser APIs Used
- Canvas 2D context
- WebAudio API (synth)
- localStorage (best score, muted state, difficulty, reduced motion)
- requestAnimationFrame
- Touch events (pointer tracking)
- Media queries (prefers-reduced-motion)

## Common Editing Tasks

### Add a new enemy type
1. Add entry to `ENEMY_TYPES` object with: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Update `spawnEnemy()` filter to include new type when available
3. Add explosion SFX entry to `EXPLODE_SFX` object
4. Create corresponding sound effect in `sfx` object if custom behavior needed

### Add a new boss
1. Add to `BOSSES` array with: `name`, `hullTop`, `hullBottom`, `wing`, `core`, `hpMul`
2. Add new pattern array to `BOSS_PATTERNS` at same index
3. Each pattern is an array of functions that call `fireRadial()`, `fireAimedSpread()`, etc.
4. Boss automatically cycles through patterns; HP scales by `hpMul`

### Adjust difficulty/balance
- **Spawn rate**: Tweak `spawnBase` and `spawnMin` in `DIFFICULTIES`
- **Enemy behavior**: Modify `enemySpeed`, `enemyShootChance` in difficulty preset
- **Player power**: Adjust weapon cooldown in `fireDelay` constant or weapon damage in collision handler
- **Boss balance**: Increase `hpMul` on boss design or adjust pattern timings (`attackTimer`)

### Add a new sound effect
1. Create function in `sfx` object that calls `beep()` or `noiseBurst()`
2. Use frequency, duration, envelope, filtering, panning to shape sound
3. Call `ensureAudio()` first to initialize WebAudio context
4. Playback respects `muted` flag

## Local Storage Keys
- `skyfighter.bestScore`: High score (integer)
- `skyfighter.muted`: Mute state ('0' or '1')
- `skyfighter.difficulty`: Selected difficulty ('easy'/'normal'/'hard')
- `skyfighter.reducedMotion`: Reduced motion preference ('0' or '1')

## Browser Compatibility
Requires:
- ES6+ JavaScript (arrow functions, template literals, const/let)
- Canvas 2D API
- WebAudio API (graceful fallback if unavailable)
- Touch Events API (graceful fallback for desktop)
- CSS Grid/Flexbox for HUD layout
- Modern responsive design features (clamp(), env())

Tested on: Chrome, Firefox, Safari, Edge (desktop and mobile)

## Files
- `index.html`: Single file containing HTML, CSS, and JavaScript
- `README.md`: Basic project info
- `CLAUDE.md`: This file

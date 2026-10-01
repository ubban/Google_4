# Sky Fighter — Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The game runs entirely in-browser with no external dependencies, using Canvas 2D for rendering, Web Audio API for procedurally generated sound, and localStorage for persistence.

### Key Features
- **Gameplay**: Vertical shooter with weapon upgrades, power-ups, and stage-based progression
- **Difficulty Levels**: Easy, Normal, Hard with different spawn rates and enemy behavior
- **Boss Encounters**: 6 unique boss ships per stage with distinct attack patterns
- **Weapons**: Vulcan (spread shots) and Laser (piercing beams) with 3 upgrade levels each
- **Audio**: Full synthesized sound effects and music using Web Audio oscillators and noise
- **Mobile Support**: Touch controls with on-screen joystick and fire/bomb buttons
- **Accessibility**: Screen reader announcements, reduced motion support, keyboard controls
- **Persistence**: Best score and user settings stored in localStorage

## Architecture

### Single-File Structure
The entire game is in `index.html`. The file contains:
1. **HTML Structure** (lines 361-420): DOM elements for canvas, HUD, UI overlays, and touch controls
2. **CSS** (lines 10-358): Styling with responsive canvas sizing and animations
3. **JavaScript** (lines 422-1667): Game engine, rendering, input handling, and game state

### Game Loop
```
requestAnimationFrame(loop) → update(dt) → draw() → render to canvas
```

The main loop:
- Tracks deltaTime to normalize movement across frame rates
- Adaptive performance: scales particle count and shadow effects based on frame time
- Caps deltaTime at 50ms to prevent physics jumps during lag
- Runs at browser's refresh rate (typically 60fps)

## Core Game Mechanics

### Game State
- **player**: Position, velocity, weapon, level, health, invulnerability timer
- **enemies[]**: Active hostile units with position, health, behavior pattern
- **boss**: Single boss unit (null if not active) with multi-phase attacks
- **bullets[]**: Player projectiles with pierce count and damage properties
- **enemyBullets[]**: Projectiles fired by enemies and bosses
- **powerUps[]**: Collectible items that fall down the screen
- **particles[]**: Visual debris and explosion effects
- **score, lives, stage, chain, multiplier**: Game progression and scoring

### Difficulty System
Controlled by `DIFFICULTIES` object (line 485):
- **Easy**: Slower spawns, lower enemy stats, more lives/bombs, reduced boss HP
- **Normal**: Baseline difficulty, standard enemy behavior
- **Hard**: Rapid spawns, faster enemies, fewer lives/bombs, boosted boss HP

### Staging & Progression
- Each stage requires killing `KILLS_PER_STAGE` (18) enemies to trigger boss fight
- Boss defeated → stage incremented, spawn timers reset, theme changes
- Themes cycle through 6 visual variations (Daylight, Sunset, Night, Storm, Arctic, Volcanic)
- Enemies scale in difficulty: Tank enemies only appear from stage 2+

### Scoring & Chain System
- Base points awarded per enemy type (grunt: 10, interceptor: 15, tank: 35)
- Chain counter increments with consecutive kills (no gaps > 2.2 seconds)
- Multiplier = 1 + floor(chain / 5), capped at 10x
- Chain breaks when player takes damage or gaps exceed timeout
- Boss kill: Fixed 500 points + stage multiplier applied

### Weapons
**Vulcan (line 1125)**:
- Multiple projectiles per shot (1/2/3 bullets based on level)
- Spread pattern: wide cone at level 3
- Fire rate: 150ms cooldown

**Laser (line 1135)**:
- Piercing beams that pass through multiple enemies
- Level 1: Single center lane
- Level 2+: Multiple lanes (±6 pixels offset)
- Level 3: Extra center lane (3 total lanes)
- Fire rate: 100ms cooldown (faster than Vulcan)
- Pierce counter decrements on hit, beam disappears at pierce=0

### Power-ups (line 1058)
- **Life** (6% spawn rate): Restores 1 health, capped at max
- **Bomb** (8% spawn rate): Adds 1 smart bomb, capped at max
- **Weapon** (20% spawn rate): Upgrades weapon level or switches weapon type

## Enemy System

### Enemy Types (line 502)
```javascript
ENEMY_TYPES = {
  grunt:       { w: 34, h: 34, hp: 1, speed: 1.0x, score: 10 },
  interceptor: { w: 26, h: 26, hp: 1, speed: 1.7x, score: 15, weaves },
  tank:        { w: 46, h: 46, hp: 4, speed: 0.55x, score: 35 }
}
```

**Spawn Logic** (line 1022):
- Random X position across playfield width
- Type selected probabilistically (grunt → interceptor → tank)
- Tanks only spawn stage 2+
- Each enemy gets random behavior pattern: spread or aimed firing

**Behavior**:
- Weaving pattern for interceptors (sine wave movement)
- Two firing patterns: spread (5 directions) or aimed (at player)
- Shoot timers randomized: 60-150 frame intervals
- Removed when off-bottom of screen

### Boss Encounters (line 508)
Six bosses cycle by stage: VANGUARD → BEHEMOTH → SPECTER → DREADNOUGHT → WRAITH → TITAN

Each boss has:
- **Design**: Hull colors, core color, attack patterns
- **HP Multiplier**: Difficulty scaling (0.8x to 1.7x)
- **Entering Animation**: Slides in from top (y: -80 → 120)
- **Attack Phases**: Telegraph delay (22 frames) → pattern fire → cooldown (85 frames)
- **Movement**: Horizontal wander bounded by screen edges

**Attack Patterns** (line 1372):
Each boss has 3 unique patterns that cycle:
- Radial bursts (full 360° fire)
- Aimed spreads (fan toward player)
- Homing missiles, columns, spirals, double rings
- Boss-specific combinations (SPECTER teleports, WRAITH has aimed cross-spread)

## Input Handling

### Keyboard Controls (line 768)
- **Arrow Keys / WASD**: Movement (any direction)
- **Space**: Fire weapon
- **X / Shift**: Drop bomb
- **P / Esc**: Pause game
- **Mouse Click**: Fire weapon (when hovering canvas)

### Mouse Control (line 838)
- Player glides toward cursor while hovering canvas
- Keyboard input takes priority (doesn't interfere with arrow keys)
- Mouse target updated continuously during movement

### Touch Controls (line 779)
- **Left 55% of screen**: Joystick for movement (visual feedback circle)
- **Right 45% of screen**: Fire and Bomb buttons
- Automatic joystick display on touch devices
- Touch events preventDefault to avoid browser scrolling

### Audio Initialization (line 545)
- Web Audio context created on-demand (first input interaction)
- Compressor applied for volume balancing across SFX
- Muting stored in localStorage

## Rendering Pipeline

### Canvas Setup (line 753)
- Base resolution: 480×720 (portrait-oriented)
- Scaled responsively: fits window with max 1.6x magnification
- CSS max-width/max-height prevents pixelation on large screens

### Sky & Parallax (line 1507)
- Gradient background per stage theme
- Radial sun glow (asymmetric positioning)
- Two cloud layers with different speeds: slower (0.5) and faster (1.2)
- Clouds loop vertically

### Entity Rendering Order
1. Background (sky, sun, clouds)
2. Player ship (with invulnerability blink)
3. Player bullets (with glow if quality=0)
4. Enemy bullets (red circles with glow)
5. Enemies (with health bar for tanks)
6. Boss (if present)
7. Power-ups (rotating squares, color-coded)
8. Particles (fade-out debris)

### Visual Polish
- **Screen Shake**: Enabled by damage/bombs; attenuated during reduced motion
- **Flash Effects**: White screen overlay for damage, yellow-white for bombs
- **Particle Explosions**: Direction-varied debris trails with life fade
- **Combo Popups**: Text animation showing kill milestones (every 10 kills)
- **Stage Banners**: Large text reveal animation (2s duration)
- **Boss Health Bar**: Real-time fill animation

### Adaptive Quality (line 474)
Adjusts based on frame time averages:
- **Quality 0** (full): Glow shadows, full particle count
- **Quality 1** (medium): 60% particles, reduced shadows
- **Quality 2** (low): 35% particles, no glow effects

## Audio System

### Web Audio Architecture (line 542)
- `AudioContext`: Central audio engine
- `DynamicsCompressor`: Prevents clipping across SFX volumes
- `masterGain`: Volume control (0.9 default)
- Lazy initialization: Context created on first interaction

### Sound Effects (line 624)
Procedurally generated using oscillators and noise:

**Music/Tones**:
- `shoot()`: Square wave sweep (760→480 Hz, 70ms)
- `laser()`: Sawtooth sweep (1500→2400 Hz, 60ms)
- `bossHit()`: Square wave (240→110 Hz, 70ms)
- `powerup()`: Chord of 3 sine waves (660/880/1100 Hz)

**Explosions**:
- `explodeGrunt()`: Sawtooth sweep + noise burst (low frequency)
- `explodeTank()`: Dual-layer sawtooth + noise (heavier sound)
- `explodeInterceptor()`: Two square waves with pitch drop

**Ambient**:
- `stageClear()`: Rising tone sequence (523/659/784/1047 Hz)
- `bossDown()`: Descending sawtooth sequence (5 notes, 0.3s each)
- `bossTelegraph()`: Two ascending sine waves

**UI**:
- `uiClick()`: High square wave click (720 Hz)
- `gameover()`: Descending sawtooth sequence

### Panning (line 562)
- `panFor()`: Horizontal position mapped to stereo (-1 to +1)
- Applied to explosions: left hits pan left, right hits pan right
- Creates positional audio feedback

## Collision Detection

### Rectangle Overlap (line 1069)
```javascript
rectsOverlap(a, b): Math.abs(ax - bx) < (aw + bw) / 2 && Math.abs(ay - by) < (ah + bh) / 2
```

Checks:
- Player vs enemies (line 1268)
- Player vs enemy bullets (line 1276)
- Player vs boss (line 1284)
- Player vs power-ups (line 1289)
- Bullets vs enemies (line 1232)
- Bullets vs boss (line 1256)

### Invulnerability (line 1457)
- Player gains 1.5s invulnerability after hit
- Visual blink effect (every 4 frames)
- Cannot take damage during invuln window
- Resetting collision only after invuln expires

## Persistence & Settings

### localStorage Keys
- `skyfighter.bestScore`: Best score ever achieved
- `skyfighter.muted`: Audio mute state (0 or 1)
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Motion effect preference (0 or 1)

### Reduced Motion
- Respects OS preference: `prefers-reduced-motion: reduce`
- User can override in settings
- Reduces animation duration from 0.3-2.0s to 0.1s max
- Screen shake magnitude scaled to 15%
- CSS animations removed via body.reduced-motion class

## Accessibility

### Screen Reader Support (line 468)
- Live region `#srLive` with role="status" aria-live="polite"
- Announcements for: damage taken, stage clears, boss defeats, new best scores
- Text-only game state updates without visual animations

### Keyboard Navigation
- All interactive buttons (start, pause, settings) keyboard accessible
- Full game playable with keyboard alone
- Pause menu accessible via P key

### Responsive Design
- `clamp()` sizing for mobile to desktop (e.g., font-size: clamp(12px, 2.8vw, 16px))
- Safe area insets for notch devices: `env(safe-area-inset-*)`
- Touch-action: none to prevent browser gestures
- Max-scale=1 to prevent unintended zoom

## Code Organization & Patterns

### Naming Conventions
- **Events**: `addEventListener`, `removeEventListener`, camelCase listeners
- **DOM References**: camelCase with `El` suffix (e.g., `canvasEl`, `scoreEl`)
- **Game Objects**: Plural arrays for collections (enemies, bullets, particles)
- **Constants**: UPPERCASE_SNAKE_CASE (DIFFICULTIES, BOSSES, ENEMY_TYPES)
- **Booleans**: `running`, `paused`, `gameOver` (past tense for states)

### Performance Optimization
- Particle pooling: Particles reuse allocation, filtered by life >= 0
- Bullet cleanup: Arrays filtered after each update to remove off-screen projectiles
- Canvas transform batching: Single ctx.save/restore per frame
- Quality scaling: Dynamically reduces shadow blur and particle counts

### State Management
- Centralized game state: All arrays (player, enemies, bullets) in function scope
- Single gameOver flag prevents state corruption
- resetGame() fully reinitializes all state

## How to Extend

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502) with w, h, hp, speedMul, score, minStage
2. Update type selection in `spawnEnemy()` (line 1028) if needed
3. Add explosion SFX mapping in `EXPLODE_SFX` (line 678)
4. Add explosion handler in `sfx` object if unique sound needed

### Adding a Boss Pattern
1. Create pattern function in `BOSS_PATTERNS[index]` array (line 1372)
2. Each pattern is a function taking boss object: `b => { /* fire pattern */ }`
3. Use helper functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.
4. Patterns cycle; add 3 per boss for variety

### Adjusting Difficulty
- Modify `DIFFICULTIES` object spawn timings or enemy stats
- Change `bossHpMul` for boss scaling
- Tweak `KILLS_PER_STAGE` for stage length

### Adding Weapon Type
1. Extend weapon switching in `applyPowerUp()` (line 1315)
2. Add weapon render in `draw()` if visual differs
3. Implement fire logic in `fireWeapon()` (line 1124)
4. Add SFX in `sfx` object

## Common Gotchas

### Frame Counter Timing
- `frame++` increments every update, not every draw
- Animations use `frame * speed` for frame-independent timing during lag
- Avoid using `frame` for deltaTime-based calculations (use `dt` instead)

### Canvas Coordinate System
- Origin (0,0) at top-left
- Y increases downward
- Collision boxes measured from center: `(x - w/2)` to `(x + w/2)`

### Audio Context Suspension
- AudioContext starts in 'suspended' state until first user interaction
- `ensureAudio()` handles resumption and initialization
- Call before any audio operations (especially on mobile)

### Touch vs Mouse Events
- Both can fire simultaneously; logic branches on device capability
- Keyboard always takes priority over mouse/joystick
- `mouseActive` is reset on `mouseleave`, not just on interaction

### Boss Health Bar Sync
- Updated every frame: `bossBarFill.style.width = boss.hp / boss.maxHp * 100`
- Important to track actual `maxHp` at spawn (complexity scaling applied)

## Development Tips

- Test on mobile: Use browser dev tools device emulation or physical device
- Debug audio: Check browser console for AudioContext errors, watch gain.value
- Profile performance: Use Chrome DevTools Performance tab to monitor frame time
- Adjust `quality` variable manually to test visual scaling logic
- localStorage data persists across page reloads; clear via DevTools > Application > Storage
- Mute audio during development with the in-game mute button to reduce noise

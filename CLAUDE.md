# Sky Fighter - Game Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. It features arcade-style gameplay with progressive difficulty, boss battles, weapon upgrades, and power-ups.

The entire game is contained in `index.html` (~1670 lines) with inline CSS and JavaScript—no external dependencies.

## How to Play

- **Mouse**: Move to steer, click to fire
- **Keyboard**: 
  - Arrow keys or WASD to move
  - Space to fire
  - X or Shift to drop bomb
  - P or Esc to pause
- **Mobile**: Drag left side to move, tap FIRE/BOMB buttons

## Game Architecture

### Game State
Located in the `update()` function scope:
- `player`: Player ship object with position, speed, weapon, health
- `enemies`: Array of active enemy ships
- `bullets`: Player projectiles
- `enemyBullets`: Incoming enemy fire
- `boss`: Current boss object (null when not fighting)
- `score`, `lives`, `stage`, `kills`: Game progress variables
- `chain`, `multiplier`: Combo tracking for score bonuses

### Core Systems

#### 1. **Rendering** (`draw()`)
- Canvas is fixed 480×720 (upscaled to fit screen)
- Layers: sky gradient → sun glow → clouds → game objects → HUD
- Theme system with 6 different sky backdrops (changes per stage)
- Particle system for explosions
- Quality scaling: reduces particle count on slower devices

#### 2. **Physics & Collision** (`update()`)
- Player movement: responds to keyboard/joystick/mouse input
- Enemy spawning: wave-based with difficulty ramping
- Bullet lifecycle: fired → travel → collision/off-screen
- Collision detection: `rectsOverlap()` for all game objects
- Screen boundaries enforce player/enemy containment

#### 3. **Difficulty System** (`DIFFICULTIES` object)
Three presets (Easy, Normal, Hard) affecting:
- Enemy spawn rate and variety
- Enemy speed and shot accuracy
- Player lives and bombs
- Boss health multiplier

#### 4. **Enemy Types** (`ENEMY_TYPES` object)
- **Grunt** (34×34): Fast, weak; red
- **Interceptor** (26×26): Very fast, weak; weaves side-to-side
- **Tank** (46×46): Slow, 4 HP; purple; unlocks at stage 2

Each has: health, speed multiplier, score value, color, shoot pattern.

#### 5. **Boss System** (`BOSS_PATTERNS` array)
Six unique boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN):
- Each boss has unique attack patterns
- Boss phases: entering (slides down) → active attacks → telegraph phase (glows before firing)
- Bullet patterns: radial bursts, aimed spreads, homing shots, spirals, columns, double rings
- HP scales with stage and difficulty

#### 6. **Weapon System**
Two weapons toggle between with power-ups:
- **Vulcan**: Spread shot (1→2→3 bullets at level 1→2→3)
- **Laser**: Piercing shot (1→2→3 lanes, pierce through enemies)

#### 7. **Audio Synthesis** (`sfx` object)
Web Audio API generates all sounds procedurally (no files):
- `beep()`: Oscillator-based tones with frequency sweeps
- `noiseBurst()`: White noise for explosions
- Individual sounds for: shoot, laser, explosions (3 types), hit, bomb, power-up, boss hit/telegraph, stage clear, UI click, game over
- Pan and filter effects based on event position
- Master gain + compressor for consistency

#### 8. **UI & HUD**
- Score, best score, multiplier display
- Weapon name/level indicator
- Heart count (lives)
- Bomb count (visual indicators)
- Boss health bar (appears when boss enters)
- Pause overlay with game stats
- End-game screen with final score and "New Best" notification

#### 9. **Input Handling**
- Keyboard: direct key state tracking
- Mouse: position-based steering (player glides toward pointer)
- Touch: dual-zone UI (movement zone left, action zone right)
- Joystick simulation: visual feedback and dead-zone handling
- Priority system: keyboard/joystick > mouse steering

#### 10. **Accessibility**
- Screen reader live region (`srLive`) for game events
- Reduced motion option (toggles animations, can be set per OS preference)
- High contrast visuals with text shadows
- Color choice: not reliant on color alone

## Key Functions

### Game Loop
- `loop(now)`: RequestAnimationFrame tick. Calls `update()` then `draw()` if running.
- `update(dt)`: Physics, collision, enemy/boss behavior (called ~60 FPS when running)
- `draw()`: Canvas rendering

### State Management
- `resetGame()`: Initialize all game state, called at game start
- `endGame()`: Trigger game over, save best score, show final overlay
- `togglePause()`: Pause/resume with overlay

### Spawning
- `spawnEnemy()`: Create a random enemy based on stage and difficulty
- `spawnBoss()`: Spawn a stage-specific boss
- `spawnExplosion(x, y, color, count)`: Particle burst for visual feedback
- `maybeSpawnPowerUp(x, y)`: Random drop on enemy death (6% life, 8% bomb, 20% weapon)

### Damage & Scoring
- `addScore(base)`: Award points (multiplied by combo), increment combo counter
- `breakChain()`: Reset combo multiplier
- `loseLife()`: Player hit, lose health, reset invulnerability timer
- `killBoss()`: Boss defeated, advance stage, award 500 points
- `useBomb()`: Destroy all on-screen enemies, damage boss, reset enemy bullets

### Audio
- `ensureAudio()`: Initialize Web Audio context (lazy-loaded on first user input)
- `updateMuteBtn()` / `muteBtn` toggle: Mute/unmute all sounds
- All `sfx.*()` functions check `muted` before playing

### Boss AI
- `updateBoss(dt)`: Handle boss movement and attack phase
- Boss patterns are functions in `BOSS_PATTERNS` that fire enemy bullets in complex patterns
- Telegraph phase: 22-frame warning with glowing core before attack

## Performance Optimizations

1. **Adaptive Quality** (`quality` variable):
   - Tracks frame time over 45 frames
   - Scales particle count and shadow rendering down on slower devices
   - 3 levels: 0=full, 1=medium, 2=low

2. **Canvas Sizing**: Upscaled to fit screen (max 1.6×) to avoid overdraw

3. **Filtered Arrays**: Enemies/bullets/particles culled each frame if off-screen

4. **Efficient Collision**: O(n²) but cache-friendly rect checks (no spatial hashing needed at this scale)

## Storage (localStorage)

- `skyfighter.bestScore`: Best score ever
- `skyfighter.muted`: Audio on/off flag
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Accessibility preference

## Themes

Each stage cycles through 6 skybox themes that affect colors but not gameplay. Themes include sky gradient stops, cloud color, and sun color (used to set visual tone).

## Development Notes

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Update `spawnEnemy()` condition to include new type based on stage/difficulty
3. Add explosion sound function to `sfx` and update `EXPLODE_SFX` mapping

### Adding a New Boss
1. Add entry to `BOSSES` array with color scheme and hpMul multiplier
2. Add attack pattern array to `BOSS_PATTERNS` (each pattern is a function)
3. Boss selection is automatic: `BOSSES[(stage - 1) % BOSSES.length]`

### Modifying Difficulty
Edit `DIFFICULTIES` object. Parameters affect:
- Spawn rate: `spawnBase` (higher = slower spawn), `spawnMin` (floor)
- Enemy behavior: `enemySpeed`, `enemyShootChance`
- Player resources: `lives`, `bombs`
- Boss scaling: `bossHpMul`

### Adjusting Audio
All audio is in the `sfx` object. Each sound uses:
- `beep(freq, dur, type, vol, opts)` for tones
- `noiseBurst(dur, vol, opts)` for percussive sounds
- Edit frequency/duration/envelope in individual `sfx.*` functions

## Testing Checklist

- [ ] Game starts and displays menu
- [ ] Difficulty selection works
- [ ] Player can move with keyboard, mouse, and touch (mobile)
- [ ] Player can fire with all input methods
- [ ] Enemies spawn and move down screen
- [ ] Collision detection works (bullets kill enemies)
- [ ] Power-ups appear and are collectable
- [ ] Score multiplier increases with combo
- [ ] Boss spawns after 18 kills per stage
- [ ] Boss patterns fire correctly
- [ ] Bomb destroys all enemies and damages boss
- [ ] Lives system (hit by enemy/bullet, lose life)
- [ ] Game over when lives reach 0
- [ ] Best score persists across sessions
- [ ] Mute button toggles audio
- [ ] Pause/resume works
- [ ] Reduced motion preference respected
- [ ] Screen shake/flash respond to performance
- [ ] Canvas resizes responsively on window resize
- [ ] All theme skyboxes render correctly
- [ ] Screen reader announcements are clear

## Known Limitations

- Single-file design limits modularity but ensures no build step needed
- Performance scales to ~60 FPS on modern devices; older devices downscale quality
- Boss patterns hardcoded per boss (expanding would require refactoring pattern definitions)
- Touch joystick is simulation only (not pressure-sensitive)
- No pause on mobile when focus lost (intentional for uninterrupted gameplay)

## Browser Compatibility

- Modern browsers with Canvas, Web Audio API, and ES6 support
- Tested on: Chrome, Firefox, Safari (desktop and mobile)
- Graceful degradation if Web Audio unavailable (game still playable, no sound)

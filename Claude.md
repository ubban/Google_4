# Sky Fighter — Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The game features weapon switching, smart bombs, stage-based progression, and boss battles. All code—HTML, CSS, and JavaScript—is contained in `index.html`.

**Entry Point:** `/home/user/Google_4/index.html`

## Architecture

### Structure
- **Single-file design:** All game logic, rendering, UI, and audio live in one HTML file (1669 lines)
- **No external dependencies:** Pure HTML5 Canvas 2D, WebAudio API, and vanilla JavaScript
- **Responsive & adaptive:** Supports desktop, tablet, and mobile with touch controls and device motion
- **State-driven:** Central game state machine manages menu, playing, paused, and game over states

### Core Subsystems

#### 1. Canvas Rendering (Canvas 2D)
- Main game rendering happens in a game loop via `requestAnimationFrame`
- Drawable objects include:
  - Player ship (with visual direction indicator)
  - Enemies (grunt, interceptor, tank types)
  - Bosses (6 unique boss designs)
  - Projectiles (player shots and enemy fire)
  - Bombs (visual explosion rings)
  - Particle effects (trails, explosions)
- Adaptive quality system downgrades rendering quality (particle count, update rate) if frame time exceeds 25ms (40 FPS target)

#### 2. Audio (WebAudio Synthesis)
- **No sample files:** All sound is procedurally generated using oscillators and noise bursts
- Sounds include: weapon fire, laser, explosions, bomb, powerups, boss telegraph, stage clear, UI clicks, game over
- **Stereo panning:** Sound effects pan based on screen position for immersion
- **Compressor:** Dynamic range compression applied to master gain for balanced levels
- Master mute button stored in localStorage

#### 3. Game Loop & Timing
- `gameLoop()` runs via `requestAnimationFrame`, updating all entities and rendering
- Delta time tracking via `performance.now()` for frame-rate-independent updates
- Performance profiling: average frame time tracked over 45 frames; quality adjusted if needed

#### 4. Player & Input
- **Keyboard:** Arrow keys / WASD for movement, Space to shoot, Ctrl/Cmd for bomb
- **Mouse:** Move to player position, click to shoot, right-click for bomb
- **Touch (mobile):** Virtual joystick on left, fire button on right, bomb button center
- **Device Motion (gyro):** Optional accelerometer input for tilt control (iOS/Android)
- Input states stored in `input` object, polled each frame

#### 5. Enemy AI & Spawning
- **Spawn system:** Enemies spawn from top of screen; difficulty affects spawn rate and min/max intervals
- **Enemy types:**
  - `grunt`: Basic enemy, 34×34px, 1 HP, fast descent
  - `interceptor`: Fast, weaving enemy, 26×26px, 1 HP, higher speed
  - `tank`: Armored, slow, 46×46px, 4 HP, spawns from stage 2+
- **Shooting AI:** Random chance per frame to fire down at player; chance varies by difficulty
- **Stage progression:** 18 kills per stage; harder spawning as difficulty increases
- **Boss spawning:** After stage 6, boss appears; defeat triggers stage clear

#### 6. Collision & Physics
- **AABB collision:** All projectiles, enemies, and player use axis-aligned bounding boxes
- **No physics engine:** Movement is frame-based position updates; enemies fall straight down
- **Bomb radius:** Damage radius is distance-based; affects all enemies within `bombRadius`

#### 7. Scoring & Progression
- **Base score:** Enemies award 10–35 points; bosses award stage-dependent multiplier
- **Combo multiplier:** Hit streak counter increases multiplier up to 10× for consecutive kills
- **Best score:** Stored in localStorage, persists across sessions
- **Difficulty settings:** Easy (4 lives, 3 bombs), Normal (3 lives, 2 bombs), Hard (3 lives, 2 bombs, tougher enemies)

#### 8. UI & HUD
- **Top bar:** Mute button, reduced-motion toggle, pause button
- **HUD elements:** Score, best score, combo multiplier, weapon name/level, hearts (lives), bombs
- **Boss bar:** Appears when boss is active; shows health as gradient bar
- **Overlay screens:** Start menu, pause screen, game-over screen with stats
- **Accessibility:** Screen-reader live region for announcements; reduced motion preference respected

## Key Components & Files

| Element | Location | Details |
|---------|----------|---------|
| HTML Structure | Lines 1–421 | Game container, canvas, HUD elements, overlay |
| CSS Styling | Lines 10–420 | Responsive layout, animations (combo pop, stage banner, vignette, flash) |
| JavaScript Game Logic | Lines 422–1669 | Game state, loop, entities, input, collision, audio |

## Game Entities

### Player
- Position, velocity, weapon level, lives, bombs
- Collision radius: 12px
- Weapon types: `basic`, `laser`, `spread`
- Damage taken reduces health; bombs restore health

### Enemy Classes
```javascript
{
  type: 'grunt' | 'interceptor' | 'tank',
  x, y, vx, vy,
  hp,
  invulnTimer, // frames invulnerable after hit
  shooting, // true if firing this frame
  weave // true if interceptor (side-to-side motion)
}
```

### Boss
```javascript
{
  name, hp, maxHp,
  x, y, width, height,
  stage, // 1–6
  shootAngle, shootCycle,
  telegraphStart, // time of last telegraph
  colors: { hullTop, hullBottom, wing, core }
}
```

### Projectiles
- **Player shots:** Damage 1, speed 500px/s, 12px radius
- **Laser shots:** Damage 1, speed 600px/s, thinner visual (8px)
- **Spread shots:** 5-way spread, damage 0.75 each
- **Enemy shots:** Speed 250px/s, damage 1; despawn if off-screen

## Important Constants

```javascript
const DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
};

const KILLS_PER_STAGE = 18; // Enemy kills required to progress to next stage

const THEMES = [ /* 6 visual themes with sky gradient and colors */ ];
const ENEMY_TYPES = { grunt, interceptor, tank };
const BOSSES = [ /* 6 unique boss definitions */ ];
```

## Storage (localStorage)

| Key | Type | Persists |
|-----|------|----------|
| `skyfighter.bestScore` | Number | Highest score ever |
| `skyfighter.muted` | '0'\|'1' | Audio mute state |
| `skyfighter.difficulty` | 'easy'\|'normal'\|'hard' | Last difficulty selected |
| `skyfighter.reducedMotion` | '0'\|'1' | Accessibility motion preference |

## Game States

1. **Start/Menu:** Display overlay with difficulty buttons and stats; await user input
2. **Playing:** Main game loop; render and update entities; check collisions and win/lose conditions
3. **Paused:** Freeze updates; show pause screen overlay
4. **Game Over:** Show final stats (score, best, stage reached); await restart or menu

## State Variables (Global Scope)

```javascript
let gameState = 'start';             // 'start', 'playing', 'paused', 'gameOver'
let score = 0, bestScore, combo = 1, stage = 1;
let player = { x, y, vx, vy, hp, lives, bomb_count, weapon, weaponLevel };
let enemies = [];                     // Array of active enemies
let boss = null;                      // Current boss object
let projectiles = [];                 // Player and enemy shots
let particles = [];                   // Visual effects (smoke, explosions)
let theme;                           // Current visual theme
let quality = 0;                     // Adaptive quality: 0=full, 1=medium, 2=low
let reducedMotion, muted, difficulty; // User preferences
```

## Key Functions

### Main Loop
- `gameLoop(dt)` — Entry point; updates game state, entities, rendering; called by `requestAnimationFrame`

### Entity Management
- `spawnEnemy()` — Creates random enemy at top of screen
- `updateEnemies(dt)` — Moves, rotates, fires enemies; removes dead ones
- `updateProjectiles(dt)` — Updates shots; checks collisions; removes off-screen
- `updateBoss(dt)` — Updates boss position, attack pattern, health bar

### Collision & Damage
- `checkCollisions()` — Detects hits between player and enemies/bullets; projectiles and enemies
- `damagePlayer(dmg)` — Reduces player HP; triggers flash effect; ends game if HP ≤ 0
- `damageEnemy(enemy, dmg, x)` — Reduces enemy HP; triggers explosion SFX if killed

### Rendering
- `render()` — Draws background, all entities, HUD elements
- `renderPlayer()`, `renderEnemy()`, `renderBoss()`, `renderProjectile()` — Entity draw functions

### UI & State
- `startGame()` — Initializes a new game; sets player position, resets score/combo
- `pauseGame()` — Toggles pause state
- `endGame()` — Shows game-over overlay; updates best score
- `setDifficulty(d)` — Applies difficulty settings; updates UI button states

### Audio
- `ensureAudio()` — Initializes WebAudio context on first user gesture (needed for browser autoplay policy)
- `beep(freq, dur, type, vol, opts)` — Plays a synth tone with optional frequency sweep, filter, pan
- `noiseBurst(dur, vol, opts)` — Plays filtered white noise

## Weapon System

| Weapon | Obtained | Damage | Projectile Speed | Special |
|--------|----------|--------|-------------------|---------|
| `basic` | Start | 1 | 500 px/s | Single shot per frame |
| `laser` | Enemy drop | 1 | 600 px/s | Thinner visual, brighter |
| `spread` | Enemy drop | 0.75×5 | 500 px/s | 5-way spread pattern |

Weapon pickups randomly drop from defeated enemies; player can hold up to 3 levels.

## Bomb Mechanics
- **Activation:** Press Ctrl (or right-click), or tap bomb button on mobile
- **Effect:** Circular wave expanding from player; damages all enemies within `bombRadius` (150px)
- **Damage:** Enemies take full damage; boss takes reduced damage
- **Resource:** Limited count per game (varies by difficulty); pickups restore by 1
- **Visual:** White flash effect and audio burst

## Boss Encounters

Bosses appear after stage 6. Each boss has:
- **Unique color scheme** (hull top, hull bottom, wings, core)
- **HP multiplier** (0.8–1.7× base)
- **Attack pattern:** Rotates and fires bullets toward player at intervals
- **Telegraph:** Plays a sound cue before firing
- **Defeat reward:** Stage clear, next stage begins

## Touch & Mobile Considerations

1. **Touch controls:** Virtual joystick (left side), fire/bomb buttons (right side)
2. **Responsive layout:** Game canvas scales to fit device; uses `clamp()` for adaptive sizing
3. **Device motion:** Optional gyro input for tilt control (when enabled)
4. **Safe area:** Respects notches and rounded corners via `env(safe-area-inset-*)`
5. **No zoom:** `user-scalable=no` prevents unwanted pinch zoom

## Accessibility

- **Screen reader support:** Live region (`srLive`) announces game events
- **Reduced motion:** Respects `prefers-reduced-motion` system setting; toggleable via UI
- **Keyboard & mouse:** Full keyboard support (arrows, space, ctrl) and mouse aim
- **Color contrast:** Text uses shadows for readability; no color-only status indicators

## Performance Optimization

1. **Adaptive quality:** Frame time tracking downgrades particle effects if rendering falls below 40 FPS
2. **Canvas 2D:** Faster than WebGL for this type of 2D sprite-based game
3. **No external assets:** Eliminates network latency and asset loading overhead
4. **Procedural audio:** No audio files; synth generation is fast and compact
5. **Dirty rectangles:** Not explicitly used; full-screen redraws per frame (acceptable for 60 FPS target)

## Development Notes

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with dimensions, HP, speed multiplier, score, and visual colors
2. Update `spawnEnemy()` to include new type in random selection
3. Add drawing code in `renderEnemy()` if unique visual is needed

### Adding a New Boss
1. Add boss definition to `BOSSES` array with name, hull/wing/core colors, and HP multiplier
2. Update boss attack logic in `updateBoss()` if custom attack pattern needed
3. Adjust difficulty multipliers in `DIFFICULTIES` if boss is too easy/hard

### Adjusting Difficulty
Edit the `DIFFICULTIES` object:
- `spawnBase` / `spawnMin`: Controls enemy spawn rate (lower = faster spawning)
- `enemySpeed`: Multiplier on enemy movement speed
- `enemyShootChance`: Probability (0–1) per frame that enemy fires
- `lives` / `bombs`: Starting resources
- `bossHpMul`: Multiplier on boss health

### Sound Design
- Edit `sfx` object to tweak frequencies, durations, volumes
- Use `beep()` for tones (sine, square, sawtooth, triangle)
- Use `noiseBurst()` for explosive/impact sounds
- Adjust `filterFreq` and `filterType` for tone shaping

## Known Limitations & Future Enhancements

1. **No mobile gyro input** in current version (framework exists; not yet wired to player movement)
2. **Fixed 480×640 canvas aspect ratio** (can be made more responsive)
3. **No level editor** (stages are code-based difficulty tuning only)
4. **No persistent saves** (only best score stored)
5. **Browser audio context** requires user gesture to initialize (first click/tap)

## Testing

To test locally:
```bash
# Open in browser (file:// or HTTP server)
open index.html
# or
python3 -m http.server 8000
# then open http://localhost:8000/index.html
```

**Quick test checklist:**
- [ ] Start game; confirm difficulty selector works
- [ ] Enemies spawn and move; check collision
- [ ] Weapon switching works (W key or enemy drops)
- [ ] Bombs clear screen and deal damage
- [ ] Boss appears after 6 stages
- [ ] Score and combo multiplier update correctly
- [ ] Audio plays (click mute/unmute to test)
- [ ] Pause/resume works
- [ ] Game over screen shows correct stats
- [ ] Best score persists after refresh

## Browser Compatibility

- **Chrome/Edge:** Full support (latest 2 versions)
- **Firefox:** Full support (latest 2 versions)
- **Safari:** Full support; requires iOS 14+ for full audio and motion
- **Mobile browsers:** Chrome/Firefox on Android, Safari on iOS

**Requires:**
- Canvas 2D Context API
- WebAudio API
- ES6 (arrow functions, const/let, template literals)
- localStorage
- requestAnimationFrame
- (Optional) Device Motion API, Gamepad API

---

**Last Updated:** 2026-10-03  
**Author:** bobwu@irobotx.top  
**Branch:** claude/inspiring-planck-b1rm0k

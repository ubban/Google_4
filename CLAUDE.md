# Sky Fighter — Codebase Documentation

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single-file HTML5 application. Players fly a plane through stages, blast enemies, collect power-ups, drop smart bombs, and battle stage bosses.

## Project Overview

- **Type**: Browser-based Web Game
- **Architecture**: Single HTML file with inline CSS & vanilla JavaScript
- **Language**: HTML5 Canvas + Web Audio API
- **No Build Required**: Open `index.html` in any modern browser to play
- **Responsive**: Works on desktop (mouse + keyboard) and mobile (touch controls)

## Quick Start

1. Open `index.html` in a web browser
2. Select difficulty (Easy/Normal/Hard)
3. Click "Start Game"
4. Use mouse to aim, click/Space to fire, X/Shift for bomb (keyboard: Arrows/WASD move)

## Game Mechanics

### Core Loop
- Player controls a plane at the bottom of the screen
- Enemies spawn from the top and descend
- Player shoots bullets to destroy enemies
- Enemies fire back at the player
- After destroying 18 enemies per stage, a boss appears
- Destroy the boss to advance to the next stage

### Player Systems

**Movement & Controls**:
- Keyboard: `Arrow Keys` or `WASD` to move
- Mouse: Move cursor to steer (glides toward pointer when no keyboard input active)
- Joystick: Touch + drag in left zone on mobile
- Touch: Tap FIRE and BOMB buttons

**Weapons**:
- `VULCAN` (default): Rapid-fire straight bullets; spreads to multiple lanes at level 2-3
- `LASER`: Slower single fire; more penetrating with pierce levels
- Switch weapons via weapon power-ups; resets to level 1 on swap

**Health & Lives**:
- Start with 3-4 lives (difficulty-dependent)
- Red heart indicator at top (❤️)
- Lose a life when hit by enemies or enemy bullets
- Game over when lives reach 0

**Bombs**:
- Smart bomb eliminates all on-screen enemies and enemy bullets
- Gives 5 points per enemy destroyed
- Limited stock (2-3 per game, difficulty-dependent)
- Recharge via bomb power-ups (💣)

### Scoring

- **Base Score**: Each enemy type has a score value (grunt: 10, interceptor: 15, tank: 35, boss: 500)
- **Multiplier Chain**: Kill streak increases multiplier (1 + kill/5, capped at 10x)
- **Chain Window**: 2.2 seconds to maintain chain; resets on getting hit
- **Best Score**: Persists in localStorage; new personal bests announced

### Difficulty Settings

Stored in `localStorage` as `skyfighter.difficulty`; defaults to `normal`.

| Setting | Easy | Normal | Hard |
|---------|------|--------|------|
| **Spawn Base** | 75 frames | 58 frames | 40 frames |
| **Spawn Min** | 40 frames | 26 frames | 18 frames |
| **Enemy Speed** | 1.0x | 1.4x | 1.9x |
| **Enemy Shoot Chance** | 45% | 70% | 95% |
| **Lives** | 4 | 3 | 3 |
| **Bombs** | 3 | 2 | 2 |
| **Boss HP Multiplier** | 0.8x | 1.0x | 1.3x |

Difficulty ramps over time: spawn rates tighten further as frame count increases.

## Enemy Types

### Grunt (`#e53935` red)
- HP: 1
- Speed: Normal
- Behavior: Descends vertically, shoots straight or spreads
- Score: 10 points
- Available: Stage 1+

### Interceptor (`#ff7043` light red)
- HP: 1
- Speed: 1.7x faster
- Behavior: Weaves side-to-side while descending, shoots aimed
- Score: 15 points
- Available: Stage 1+

### Tank (`#5e35b1` purple)
- HP: 4
- Speed: 0.55x (slow)
- Behavior: Descends slowly, shoots aimed
- Score: 35 points
- Visible health bar above unit
- Available: Stage 2+

Enemy selection is weighted and stage-gated: grunt most common, tank rarer and only after stage 2.

## Boss System

### Boss Mechanics
- Spawn after 18 stage kills per stage
- Cannot spawn until warning shown ("WARNING — {NAME} APPROACHING")
- Display boss name and HP bar at top of screen
- Entering phase: boss slides down for 1.2 seconds before combat
- Combat: boss moves horizontally, fires patterned attacks, telegraphs before each pattern
- Defeat: 500 base points + advance stage

### Boss List

| Stage | Name | Traits |
|-------|------|--------|
| 1-6 (cycles) | VANGUARD | Purple/magenta; radial bursts, aimed spreads, V-formations |
| | BEHEMOTH | Orange; heavy columns, slow homing, dense radial walls |
| | SPECTER | Teal/cyan; teleports, tight spirals, multi-aimed shots |
| | DREADNOUGHT | Gray; 8-way crosses, homing missiles, dense radials |
| | WRAITH | Deep purple; teleport barrages, expanding double rings |
| | TITAN | Gold; heavy columns, full rings, relentless homing |

Each boss has 3 distinct attack patterns (cycle through them). HP scales with stage and difficulty.

### Boss Attack Patterns

Bosses telegraph attacks with a 22-frame warning (white glow on core). Attack patterns include:
- **Radial**: Projectiles in circular ring
- **Aimed Spread**: Fan of projectiles aimed at player with spread variance
- **Columns**: Vertical bullet streams
- **Spiral Burst**: Rotating ring of 8 projectiles
- **Cross Fire**: 8-way cardinal + diagonal fire
- **Homing**: Projectiles seed aim toward player with variation
- **Double Ring**: Two concentric radial rings at different speeds
- **Teleport**: Boss jumps to new X position before firing

## Power-ups

Spawn on enemy death with weighted random chance:

| Type | Icon | Chance | Effect |
|------|------|--------|--------|
| Life | ❤️ | 6% | +1 HP (capped at max) |
| Bomb | 💣 | 8% | +1 bomb (capped at max) |
| Weapon | ⭐ | 20% | Level up weapon or swap weapon + reset to level 1 |

Power-ups float downward and disappear if they reach the bottom.

## Visual & Audio

### Graphics
- Canvas rendering at 480x720 logical resolution, scaled to fit window
- No external image assets; everything drawn with Canvas 2D API
- Planes drawn with geometric shapes (triangles, circles, ellipses)
- Animated clouds per-stage in background
- 6 visual themes (skylight, sunset, night, storm, arctic, volcanic) that cycle per stage

### Screen Effects
- **Screen Shake**: Triggered on player hit, boss defeat, bomb explosion (intensity & duration vary)
- **Flash**: White flash on hit (brief) or bomb detonation (longer)
- **Vignette Pulse**: Red pulsing edge effect when player at 1 HP
- **Combo Pop**: Yellow text showing kill milestones (10, 20, 30…)
- **Stage Banner**: Large text announcing stage transitions and boss approach

### Audio

**Web Audio Synthesis** (no external audio files):
- Synth beeps for laser/shoot SFX
- Noise bursts for explosions and impacts
- Per-enemy explosion sounds (grunt, interceptor, tank variations)
- Boss-specific effects (telegraph, defeat cascade)
- Stage clear jingle
- Game over descending pitch

Audio muted via icon button; state persists in localStorage.

## Input & Controls

### Keyboard
- `ArrowUp`/`ArrowDown`/`ArrowLeft`/`ArrowRight` or `W`/`A`/`S`/`D`: Move
- `Space`: Fire
- `X` or `Shift`: Bomb
- `P` or `Esc`: Pause/Resume

### Mouse
- **Move**: Position over canvas to steer (priority if no keyboard input)
- **Fire**: Click to hold fire
- **Fire Release**: Release click to stop firing

### Touch (Mobile)
- **Left Zone**: Drag to move (shows joystick base + stick visualization)
- **FIRE Button**: Tap to fire (continues while held)
- **BOMB Button**: Tap to detonate bomb

### Pause
- Press `P` or `Esc` or click pause button (⏸)
- Overlay shows on pause; click "Resume" or press P again

## Code Architecture

### Main Structure
Entire game logic in a single IIFE (Immediately Invoked Function Expression) to avoid polluting global scope.

### Key Variables

**Game State**:
- `running`: Game in active play
- `paused`: Game paused by user
- `gameOver`: All lives lost
- `stage`, `stageKills`, `bossWarned`: Stage tracking
- `frame`: Global frame counter (increments each update)
- `difficulty`: Selected difficulty name

**Game Objects**:
- `player`: Player plane object with position, weapon, level, invuln timer
- `enemies`: Array of active enemy objects
- `boss`: Current stage boss (null if none active)
- `bullets`: Array of player projectiles
- `enemyBullets`: Array of enemy projectiles
- `particles`: Array of explosion particles
- `powerUps`: Array of collectible items

**UI State**:
- `score`, `lives`, `bombCount`, `multiplier`, `chain`: Gameplay stats
- `bestScore`: Persistent high score from localStorage
- `muted`, `reducedMotion`: Accessibility/preference toggles

### Key Functions

**Game Loop**:
- `loop(now)`: RequestAnimationFrame callback; calls update/draw
- `update(dt)`: Advance all game objects, handle collisions
- `draw()`: Render all entities to canvas

**Player/Weapon**:
- `fireWeapon()`: Spawn player bullets based on weapon type & level
- `useBomb()`: Detonate bomb; clear enemy bullets, damage all enemies/boss

**Enemy/Boss**:
- `spawnEnemy()`: Create new enemy with random type & position
- `spawnBoss()`: Create boss for current stage
- `updateBoss(dt)`: Move boss, handle telegraph & attack patterns
- `killBoss()`: Boss defeated; advance stage, show banner

**Collision**:
- `rectsOverlap(a, b)`: AABB collision check
- Player vs enemy bullets/enemies → `loseLife()`
- Player vs power-up → `applyPowerUp(type)`
- Bullets vs enemies/boss → damage & explode

**Audio**:
- `beep(freq, dur, type, vol, opts)`: Synth tone (square/sawtooth/sine/triangle)
- `noiseBurst(dur, vol, opts)`: White noise burst
- `sfx.<name>()`: Pre-defined sound effects (shoot, laser, explode, bomb, powerup, etc.)

**Visual Effects**:
- `spawnExplosion(x, y, color, count)`: Particle burst
- `screenShake(mag, dur)`: Trigger screen jitter
- `flashHit()`/`flashBomb()`: White flash animation
- `showComboPop(text)`: Animate combo milestone text
- `showStageBanner(text)`: Animate stage announcement

**UI**:
- `renderHearts(count, max)`: Update life display
- `renderBombs(count, max)`: Update bomb count display
- `endGame()`: Game over state; show final score & best
- `togglePause()`: Pause/resume game

### Constants

**Canvas**:
- `BASE_W = 480`, `BASE_H = 720`: Logical resolution

**Gameplay**:
- `KILLS_PER_STAGE = 18`: Enemy threshold before boss

**Difficulties** (see Difficulty Settings section):
- Spawn rates, enemy speed, shoot chance, lives, bombs, boss HP scaling

**Enemy Types**:
- `ENEMY_TYPES`: Grunt, Interceptor, Tank (w/h, hp, speed, color, score, minStage)

**Bosses**:
- `BOSSES`: 6 boss designs with name, colors, HP multiplier
- `BOSS_PATTERNS`: 3 attack patterns per boss (array of pattern functions)

**Themes**:
- `THEMES`: 6 visual themes with sky colors, cloud color, sun color, name

**Sound Effects**:
- `sfx` object: Pre-bound audio functions (shoot, laser, explode, hit, bomb, powerup, bossTelegraph, bossDown, stageClear, uiClick, gameover)

**Storage Keys** (localStorage):
- `skyfighter.bestScore`: High score
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.muted`: Sound muted flag (1 or 0)
- `skyfighter.reducedMotion`: Reduced motion preference (1 or 0)

## Performance Optimization

### Adaptive Quality
- Tracks frame times over 45 frames
- Auto-scales quality (0=full, 1=medium, 2=low) based on average FPS
- Reduces particle count, glow effects at lower quality
- Thresholds: 40 FPS up-scale limit, 56 FPS down-scale limit

### Culling
- Remove off-screen bullets, enemies, particles
- Boss only updates when active
- Enemy spawning paused during boss fight

### Rendering Optimization
- Shadow/glow effects only at `quality === 0`
- Skip rendering invulnerable player (blinking state)
- Avoid unnecessary context saves/restores

## Extending the Game

### Add New Enemy Type
1. Add entry to `ENEMY_TYPES` object with w, h, hp, speedMul, color, dark, score, minStage
2. Add spawn logic in `spawnEnemy()` to weight selection
3. Add explosion SFX key to `EXPLODE_SFX` object
4. Create SFX function in `sfx` object

### Add New Boss
1. Add entry to `BOSSES` array with name, hull colors, core color, HP multiplier
2. Create 3 attack pattern functions
3. Add pattern array to `BOSS_PATTERNS` array (index matches BOSSES index)
4. Pattern functions spawn enemy bullets via fire* helpers

### Add New Visual Theme
1. Add entry to `THEMES` array with sky gradient colors, cloud color, sun color, name
2. Themes cycle by `(stage - 1) % THEMES.length`

### Add New Difficulty
1. Add entry to `DIFFICULTIES` object with all spawn/enemy/bomb settings
2. Update difficulty button logic if needed

### Add Accessibility Feature
- Reduce motion preference already detected via `window.matchMedia('(prefers-reduced-motion: reduce)')`
- Can extend with other a11y toggles (colorblind mode, text size, etc.)

## Browser Support

- **Desktop**: Chrome, Firefox, Safari, Edge (all modern versions)
- **Mobile**: iOS Safari, Chrome Android, Firefox Android
- **Requirements**: HTML5 Canvas, Web Audio API, LocalStorage

## Known Limitations

- Single-threaded rendering (no Web Workers)
- Canvas scaling via CSS (not pixel-perfect on all sizes)
- Web Audio has warm-up latency on first user interaction
- Mobile audio may require user gesture to unmute (browser policy)

## Future Enhancement Ideas

- Score submission to leaderboard (API integration)
- Additional weapon types (spread gun, homing, etc.)
- Enemy behavior patterns (formations, coordinated attacks)
- Power-up variants (temporary invincibility, slow time)
- Procedural stage generation
- Replay system or demo mode
- Touch haptics / vibration feedback
- Shader-based visual effects
- Multiplayer (local co-op or online)

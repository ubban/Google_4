# Sky Fighter - Codebase Guide for Claude

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 game using Canvas and Web Audio API. It runs entirely in the browser with no external dependencies or assets—all graphics and sounds are procedurally generated.

- **Type**: Browser-based arcade game
- **Implementation**: Single HTML file (index.html, ~1669 lines)
- **Technologies**: HTML5 Canvas, Web Audio API, ES6 JavaScript
- **Target**: Desktop and mobile (touch controls included)
- **Source**: https://github.com/ubban/google_4

---

## Game Architecture

### Game Loop
- **Adaptive 60 FPS**: Uses `requestAnimationFrame` with delta-time-based updates
- **Adaptive Quality**: Monitors frame times and reduces visual quality (particle counts, effect detail) if FPS drops below 40
- **Three Game States**: 
  - Menu/overlay (before game or after game over)
  - Running (active gameplay)
  - Paused (mid-game pause)

### Canvas & Rendering
- **Dynamic sizing**: Canvas scales to fit viewport while maintaining aspect ratio
- **Double-buffering**: Built-in via Canvas API
- **Layered rendering**: Sky gradient, game objects, effects, UI overlay (HUD), accessibility overlay

### Time & Physics
- All movement/animation uses delta time (`dt`), framerate-independent
- Coordinates: `(0,0)` at top-left; `x` rightward, `y` downward
- Collision detection: Simple AABB (axis-aligned bounding box) against player and bullets
- Velocity-based movement: Objects have `vx`/`vy` (pixels/second)

---

## Game Systems

### 1. Difficulty & Balance
Located in `DIFFICULTIES` object (~line 485):
```javascript
DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
}
```
- **spawnBase**: Frames between enemy spawns at game start
- **spawnMin**: Minimum spawn rate (harder as time goes on)
- **enemySpeed**: Movement speed multiplier
- **lives**: Starting player health
- **bombs**: Starting bomb count
- **bossHpMul**: HP multiplier for stage bosses

### 2. Stages & Progression
- `KILLS_PER_STAGE = 18`: Number of regular enemies to kill before boss appears
- 6 different visual themes (sky gradients), cycles through stages
- 6 unique bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Each boss has unique color scheme and HP multiplier

### 3. Enemy System
**Enemy Types** (line ~502):
```javascript
ENEMY_TYPES = {
  grunt:       { w: 34, h: 34, hp: 1, speedMul: 1.0,   color: '#e53935', score: 10, minStage: 1 },
  interceptor: { w: 26, h: 26, hp: 1, speedMul: 1.7,   color: '#ff7043', score: 15, minStage: 1, weave: true },
  tank:        { w: 46, h: 46, hp: 4, speedMul: 0.55,  color: '#5e35b1', score: 35, minStage: 2 }
}
```
- **grunt**: Red, basic enemy, appears from stage 1
- **interceptor**: Orange, fast, weaves side-to-side, appears from stage 1
- **tank**: Purple, slow but tanky (4 HP), appears from stage 2
- Enemies spawn from top of screen, move down, randomly shoot
- Speed adjusted by difficulty multiplier

### 4. Boss System
6 bosses with unique visuals and balance tiers:
- **VANGUARD** (hpMul: 1.0): Balanced
- **BEHEMOTH** (hpMul: 1.35): Tanky
- **SPECTER** (hpMul: 0.8): Weak
- **DREADNOUGHT** (hpMul: 1.55): Very tanky
- **WRAITH** (hpMul: 0.95): Balanced-weak
- **TITAN** (hpMul: 1.7): Extremely tanky

Boss mechanics:
- Follows player `x` position loosely
- Shoots bullets downward
- Displays health bar at top
- Moves in circular bobbing pattern
- Dies when HP ≤ 0, triggers stage win

### 5. Player Mechanics
- **Lives**: Start with 3-4 (by difficulty)
- **Health**: 1 hit = 1 life lost (no armor/shields)
- **Bombs**: 2-3 (by difficulty), clear nearby bullets/enemies
- **Weapons**: 
  - Default: Straight shot
  - Level 1: Wider spread (3 bullets)
  - Level 2: Full spread + faster (5 bullets)
  - Max level 2
- **Weapon pickups**: Dropped by enemies, increment weapon level
- **Movement**: Smooth, full 8-directional via keyboard or touch joystick
- **Fire**: Continuous fire when button held

### 6. Scoring System
- Base points: 10 (grunt), 15 (interceptor), 35 (tank), 500 (boss)
- **Multiplier**: Increases by 1x for each consecutive enemy killed without getting hit
- **Combo**: Displayed as popup when multiplier increases
- **Best Score**: Persisted in `localStorage` under key `'skyfighter.bestScore'`

### 7. Weapon System
Stored as `playerWeapon` (1-3):
- **Level 1**: Single center bullet, speed ~300px/s
- **Level 2**: 3 bullets (center + 15° angles)
- **Level 3**: 5 bullets (full spread)
Each upgrade increases fire rate and damage spread. Weapon downgrades on hit (reset to 1).

### 8. Bomb Mechanics
- **Effect**: Clears all nearby bullets and stuns/damages enemies in radius
- **Visuals**: White explosion with screen flash
- **Strategy**: Save for tight situations; limited quantity

### 9. Audio System
- **Engine**: Web Audio API with procedural synthesis
- **No external assets**: All sounds generated in real-time
- **Sounds**: Shot/hit/explosion/power-up via oscillators + ADSR envelopes
- **Muting**: Toggle via `muted` state, persisted to localStorage
- **Master gain**: 0.9 (slightly below full volume to prevent clipping)
- **Compression**: Dynamic compression applied to master bus

### 10. Themes (Visual Variety)
6 color schemes, one per stage:
```javascript
THEMES = [
  { name: 'Daylight Front',   sky: ['#0d2f5e', '#1e6fd8', '#7fc1ff'], ... },
  { name: 'Sunset Corridor',  sky: ['#3a0d5e', '#7d3c98', '#ffb199'], ... },
  { name: 'Night Approach',   sky: ['#020814', '#0d1b3a', '#2a4b7c'], ... },
  { name: 'Storm Front',      sky: ['#10241e', '#274b3e', '#5c8f76'], ... },
  { name: 'Arctic Wastes',    sky: ['#0a1b33', '#31597a', '#bfe8ff'], ... },
  { name: 'Volcanic Ridge',   sky: ['#1a0503', '#5c1408', '#ff7a3d'], ... }
]
```
Each theme defines sky gradient and cloud/sun colors. Cycles through on each stage.

### 11. Adaptive Performance
- Monitors last 45 frames' duration
- If average frame time > 1/40 (25ms): Reduce quality
- If average frame time < 1/56 (~18ms): Increase quality
- **Quality levels**:
  - 0: Full (all effects)
  - 1: Medium (fewer particles)
  - 2: Low (minimal effects)

---

## User Interface (HUD)

### Persistent HUD Elements (Top of screen)
- **Score**: Current score
- **Best Score**: High score (persisted)
- **Weapon**: Current weapon level
- **Multiplier**: Kill combo multiplier
- **Hearts**: Remaining lives (visual display)
- **Bombs**: Bomb count remaining

### Screen Overlays
- **Game Over**: Title, final score, best score, stage reached, restart button
- **Stage Banner**: "STAGE X" title screen before each boss
- **Boss Health Bar**: Top-center health bar during boss fights
- **Combo Popup**: "+1x" text popup when multiplier increases
- **Vignette Effect**: Red edge darkening when damaged (pulsing)
- **Flash Effect**: White screen flash on hit/bomb explosion

### Accessibility
- Screen reader support: `#srLive` element for announcements
- Reduced motion: Toggle via settings button; disables screen shake/color flash
- High contrast: Text has text shadows for readability

### Mobile Controls
- **Joystick Zone**: Left side of screen, virtual stick for movement
- **Fire Button**: Right side, auto-fires while held
- **Bomb Button**: Center-right, tap to trigger bomb
- **Difficulty Selection**: Pre-game buttons for easy/normal/hard
- **Settings**: Mute toggle, reduced motion toggle, pause button

---

## Code Organization

### Main Script Sections (within single `<script>` block starting at line ~422)

1. **Initialization** (~422-527): DOM setup, constants, storage
2. **Settings/UI** (~528-800): Mute, motion, difficulty, display updates
3. **Audio System** (~542-600): WebAudio initialization, beep function, sound effects
4. **Input Handling** (~800-1000): Keyboard, mouse, touch events
5. **Game State** (~1000-1200): Score, lives, bombs, weapon, multiplier
6. **Entities** (~1200-1400): Player, Enemy, Boss, Bullet classes
7. **Collision & Updates** (~1400-1500): Movement, damage, collision detection
8. **Rendering** (~1500-1650): Canvas drawing, particles, effects
9. **Game Loop** (~1650-1669): Main `requestAnimationFrame` loop

### Key Objects/Variables

**Game State:**
- `running`: Boolean, true during active gameplay
- `paused`: Boolean, true when game is paused
- `gameOver`: Boolean, true after player loses all lives
- `stage`: Current stage number (1+)
- `score`, `bestScore`: Score tracking
- `lives`, `bombs`: Player resources
- `playerWeapon`: Weapon level (1-3)
- `multiplier`: Kill combo multiplier
- `enemyKills`: Kill count toward stage completion

**Collections:**
- `enemies[]`: Array of active enemy objects
- `bullets[]`: Array of active bullets (both player and enemy)
- `effects[]`: Array of particle effects

**Difficulty/Config:**
- `DIFFICULTIES`: Object with difficulty presets
- `ENEMY_TYPES`: Object with enemy stat templates
- `BOSSES`: Array of 6 boss definitions
- `THEMES`: Array of 6 visual themes

---

## Data Structures

### Enemy Object
```javascript
{
  x, y: position
  w, h: width, height
  vx, vy: velocity
  hp: health points
  type: 'grunt'|'interceptor'|'tank'
  isBoss: boolean
  name?: boss name (if boss)
  shootCooldown: frames until next shot
  ...
}
```

### Bullet Object
```javascript
{
  x, y: position
  vx, vy: velocity
  isEnemyBullet: boolean
  lifetime: frames until expires
  ...
}
```

### Player Object
```javascript
{
  x, y: position
  w, h: width, height (usually 24x32)
  vx, vy: velocity
  weapon: weapon level
  fireCooldown: frames until next shot
  invulnerable: frames of invulnerability after hit
  ...
}
```

---

## How to Modify/Extend

### Add a New Enemy Type
1. Add to `ENEMY_TYPES` with stats: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optionally `weave`
2. In enemy spawning logic, randomly select the new type (respecting `minStage`)
3. Adjust spawn rates and difficulty if needed

### Add a New Boss
1. Add to `BOSSES` array with unique colors and `hpMul`
2. Boss visuals are drawn procedurally (no static sprites), so adjust drawing code if desired
3. Boss HP is `baseHp * difficulty.bossHpMul * boss.hpMul`

### Add a New Theme
1. Add to `THEMES` array with sky gradient colors and sun/cloud RGB values
2. Theme cycles automatically through stages

### Adjust Game Balance
- **Difficulty**: Modify `DIFFICULTIES` object
- **Scoring**: Change base scores in `ENEMY_TYPES` and boss spawn logic
- **Speed**: Adjust `enemySpeed` multiplier in `DIFFICULTIES`
- **Progression**: Change `KILLS_PER_STAGE` (default: 18)
- **Lives/Bombs**: Modify in `DIFFICULTIES`

### Add New Visual Effects
- Effects are managed in `effects[]` array
- Add new effect type in particle/effect creation
- Render in the effects loop (typically before HUD)

### Modify Sound Effects
- Sounds generated in functions like `beep()`, `playShot()`, `playExplosion()`, etc.
- Adjust frequency, duration, or waveform type to change sound
- Use `opts` object for sweeps, filters, panning, delays

### Add New Game Features
- **Weapons**: Add weapon level 4+ and corresponding fire pattern
- **Pickups**: Add new pickup types (health, shields, score multiplier)
- **Enemies**: New types with unique behaviors
- **UI**: Add new HUD elements via DOM or canvas rendering

---

## Important Constants

| Constant | Value | Purpose |
|----------|-------|---------|
| `KILLS_PER_STAGE` | 18 | Enemies to kill before boss |
| `STORAGE_BEST` | `'skyfighter.bestScore'` | LocalStorage key for high score |
| `STORAGE_MUTE` | `'skyfighter.muted'` | Mute state storage |
| `STORAGE_DIFF` | `'skyfighter.difficulty'` | Selected difficulty storage |
| Canvas size | Dynamic (fits viewport) | Game area |

---

## Quality Assurance Notes

### Performance Considerations
- Adaptive quality system prevents lag on lower-end devices
- Particle effects are culled if off-screen or quality is reduced
- No memory leaks from entity arrays (enemies/bullets cleaned up on death)

### Accessibility
- Screen reader support for game state announcements
- Reduced motion option disables camera shake and screen flash
- High contrast text with shadows for readability
- Keyboard and touch controls both supported

### Persistence
- Best score saved to localStorage (survives page reload)
- Difficulty preference saved (easy/normal/hard)
- Audio mute state persisted
- Reduced motion preference saved

### Browser Compatibility
- Requires: ES6, Canvas 2D, Web Audio API
- Tested on: Modern browsers (Chrome, Firefox, Safari, Edge)
- Mobile: iOS Safari, Android Chrome

---

## Development Tips

1. **Testing on Mobile**: Use Chrome DevTools device emulation or actual touch device
2. **Audio Debugging**: Check browser console for WebAudio errors; some contexts require user interaction first
3. **Performance**: Monitor frame times via adaptive quality system or external profiler
4. **Collision**: AABB collision is simple but sometimes feels forgiving; tweak hitbox sizes if needed
5. **Enemy AI**: Simple patterns (straight movement + random shooting); customize for variety
6. **Visuals**: All procedurally drawn; no sprite sheets or external images

---

## References

- **Game Loop**: `requestAnimationFrame` with delta-time
- **Rendering**: Canvas 2D Context API
- **Audio**: Web Audio API Oscillators + Gain nodes
- **Storage**: Browser localStorage API
- **Input**: `keydown`, `keyup`, touch events, mouse events
- **Collision**: Simple AABB (distance-based for bombs)

---

*Last Updated: 2026-09-26*
*For Claude AI agents working on Sky Fighter game modifications and features.*

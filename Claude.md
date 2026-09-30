# Sky Fighter – Codebase Documentation

## Project Overview

**Sky Fighter** is a browser-based vertical shoot-'em-up (shmup) game built with vanilla HTML, CSS, and JavaScript. The entire game is contained in a single `index.html` file (~1,669 lines) with no external dependencies.

**Genre:** Vertical Shooter (Raiden-style)  
**Features:**
- 3 difficulty levels (Easy, Normal, Hard)
- 6 enemy types across different stages
- 6 boss encounters with unique attack patterns
- 8 weapon upgrade tiers with distinct firing patterns
- Combo multiplier system
- Smart bomb mechanic
- Touch controls + keyboard support
- Adaptive performance scaling
- Audio effects (Web Audio API)
- Theme system (6 visual themes)

---

## Architecture

### Single-File Design
The game is a self-contained HTML file with:
- **HTML Structure:** Minimal DOM — canvas element, overlay UI, touch control zones
- **CSS:** Inline styles for UI, animations, responsive layout
- **JavaScript:** IIFE (Immediately Invoked Function Expression) encapsulating all game logic

### Core Game Loop
```
update(dt) → Process input, physics, collisions
         ↓
render() → Draw canvas, HUD updates
         ↓
requestAnimationFrame() → Repeat at 60 FPS
```

---

## Key Data Structures

### Game State
```javascript
bestScore, muted, difficulty, reducedMotion       // Persisted settings (localStorage)
gameActive, gamePaused, waveCount, killCount      // Current game state
player, enemies[], bullets[], bosses[], effects[] // Active entities
currentThemeIdx, quality                          // Rendering options
```

### Configuration Objects

#### `DIFFICULTIES`
```javascript
{
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
}
```

#### `ENEMY_TYPES`
- **grunt:** Weak, basic enemy (1 HP, +10 score)
- **interceptor:** Fast, weaving (1 HP, +15 score)
- **tank:** Armored, slow (4 HP, +35 score)

#### `BOSSES`
6 boss encounters: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN  
Each has unique color scheme, name, and HP multiplier.

#### `THEMES`
6 visual themes: Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge

---

## Main Functions

### Initialization & Setup
- `resetGame()` – Initialize player, clear enemies, reset score/combo
- `ensureAudio()` – Initialize Web Audio context (required by browser autoplay policy)
- `resize()` – Adjust canvas size and recalculate dimensions

### Game Loop
- `update(dt)` – All physics, collision, entity updates (bullets, enemies, boss, particles)
- `render()` – Draw canvas background, entities, HUD

### Entity Management
- `spawnEnemy()` – Create random enemy based on current stage/difficulty
- `spawnBoss()` – Create boss encounter at wave threshold
- `spawnExplosion(x, y, color, count)` – Particle effect for destruction
- `maybeSpawnPowerUp(x, y)` – 15% chance to drop weapon upgrade

### Player & Weapons
- `fireWeapon()` – Execute current weapon attack pattern
- `fireRadial(b, count, speed, offset)` – Bullets in all directions
- `fireAimedSpread(b, count, speed, spreadStep)` – Spread toward enemies
- `fireColumns(b, speed)` – Narrow vertical columns
- `fireSpiralBurst(b, speed)` – Rotating spiral pattern
- `fireCross(b, speed)` – Plus-sign pattern
- `fireHomingSeeded(b, count, speed)` – Bullets seek nearest enemy
- `fireDoubleRing(b, speedA, speedB)` – Concentric rotating rings
- `useBomb()` – Smart bomb: destroys bullets, damages enemies

### Collision & Effects
- `rectsOverlap(a, b)` – AABB collision detection
- `flashHit()` – Screen flash when player takes damage
- `flashBomb()` – Screen flash when bomb detonates
- `screenShake(mag, dur)` – Camera shake effect
- `showComboPop(text)` – Floating text popup for combos
- `showStageBanner(text)` – Large stage announcement

### Scoring
- `addScore(base)` – Add base score × current multiplier, update combo
- `breakChain()` – Reset combo multiplier to 1x when combo breaks
- `applyPowerUp(type)` – Apply weapon upgrade or health recovery

### Audio
- `beep(freq, dur, type, vol, opts)` – Tone-based sound effect
- `noiseBurst(dur, vol, opts)` – Noise-based sound effect
- `panFor(x)` – Calculate stereo pan from x position
- `updateMuteBtn()` – Toggle mute state

### UI & Input
- `setDifficulty(name)` – Select difficulty and persist to localStorage
- `updateMotionBtn()` – Toggle reduced motion preference
- `joystickStart/Move/End()` – Touch joystick input
- `canvasPointFromEvent(e)` – Convert mouse/touch event to canvas coordinates
- `weaponLabel()` – Get human-readable weapon name

### Drawing
- `drawPlane(x, y, w, h, color, darkColor, flame)` – Player ship with flame
- `drawBossShip(b)` – Boss multi-part rendering

---

## Weapon System

8 upgrade tiers, each with distinct firing pattern:

| Tier | Pattern | Description |
|------|---------|-------------|
| 1 | Single | One bullet up center |
| 2 | Dual | Two bullets (left/right) |
| 3 | Radial | Bullets in all directions |
| 4 | Aimed Spread | Spread toward nearest enemies |
| 5 | Columns | Narrow vertical columns |
| 6 | Spiral | Rotating spiral pattern |
| 7 | Homing | Bullets seek enemies |
| 8 | Double Ring | Concentric rotating rings |

Weapon upgrades persist between lives within a game session.

---

## Game Mechanics

### Stages & Waves
- Stages advance every 18 enemy kills
- New themes/enemy types unlock at higher stages
- Boss encounters triggered at waves 2, 4, 6 (every 2 waves)

### Combo System
- Destroying enemies increases combo multiplier (1.0x → 8.0x max)
- Multiplier decays if no kill for ~2 seconds
- Score gain = base score × current multiplier
- Visual indicator in HUD

### Collision Layers
- **Player bullets** collide with enemies/bosses
- **Enemy bullets** collide with player
- **Player** cannot pass screen edges
- **Smart bomb** destroys player and enemy bullets, damages all enemies/boss

### Difficulty Scaling
- Enemy spawn rate increases (lower spawn interval)
- Enemy movement speed increases
- Enemy fire rate increases
- Player starts with fewer lives (3 vs 4)
- Bombs limited to 2 (vs 3 on easy)
- Boss HP increased by multiplier (0.8x to 1.7x per boss)

### Performance Adaptation
- Monitors average frame time across 45 frames
- Adjusts `quality` level (0=full, 1=medium, 2=low) to maintain 60 FPS
- Affects particle density, effect visibility

---

## Input Handling

### Keyboard
- **Arrow Keys / WASD:** Move player
- **Space / Click:** Fire weapon
- **B / Q:** Deploy bomb
- **P:** Pause/resume
- **M:** Mute audio

### Mouse
- Click canvas to fire continuously
- Move mouse to control aim (crosshair cursor)

### Touch
- **Left Joystick Zone:** Move player
- **Right Fire Button:** Fire weapon
- **Right Bomb Button:** Deploy bomb

---

## Persistence (localStorage)
- `skyfighter.bestScore` – High score
- `skyfighter.muted` – Audio mute state (1/0)
- `skyfighter.difficulty` – Last selected difficulty
- `skyfighter.reducedMotion` – Reduced motion preference

---

## Common Tasks

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Update `spawnEnemy()` to include in spawn probability based on current stage
3. Drawing is handled generically by `drawPlane()` using color properties

### Add a New Boss
1. Add entry to `BOSSES` array with `name`, `hullTop`, `hullBottom`, `wing`, `core`, `hpMul`
2. Add corresponding attack pattern function (e.g., `fireRadial()`, `fireHomingSeeded()`)
3. Update `updateBoss()` switch case to call new pattern

### Add a New Weapon Tier
1. Create new firing function following existing pattern (e.g., `fireSomethingNew()`)
2. Increment weapon tiers and update `weaponLabel()` case statement
3. Hook into `fireWeapon()` switch case
4. Balance spawn probability and bullet density

### Add a New Theme
1. Add entry to `THEMES` array with `sky` (3-color gradient), `cloud`, `sun`, `name`
2. Update theme cycling logic if needed
3. Themes auto-unlock at higher stages based on index

### Adjust Difficulty
1. Modify `DIFFICULTIES` object values
2. `spawnBase`/`spawnMin` control enemy frequency (lower = more enemies)
3. `enemySpeed`, `enemyShootChance` tune combat difficulty
4. `bossHpMul` scales boss health per stage

---

## Performance Considerations

### Canvas & Rendering
- Uses 2D canvas context (no WebGL)
- Clears full canvas every frame
- Particle effects (explosions) limited and pooled
- Quality scaling reduces particle count on slow devices

### Audio
- Web Audio API with synthesized effects (no audio files)
- Stereo panning based on screen position
- Lazy initialization via `ensureAudio()` for browser compliance

### Collision Detection
- Simple AABB (axis-aligned bounding box) overlap
- O(n²) worst-case, but typically fast due to entity limits

### Memory
- No loading phase — instant play
- Entities cleaned up immediately on destruction
- No memory leaks (IIFE scope, event cleanup)

---

## Code Organization

### Top-Level Sections (IIFE)
1. **Setup & Constants** – DOM refs, storage keys, game configs
2. **Audio System** – Synth functions, SFX definitions
3. **Rendering Helpers** – UI drawing, particle effects
4. **Input System** – Keyboard, mouse, touch handlers
5. **Game State Management** – Reset, difficulty, pause
6. **Entity Updates** – update() function, collision checks
7. **Entity Spawning** – Enemies, bosses, effects
8. **Drawing** – render() function, canvas rendering
9. **Game Loop** – requestAnimationFrame setup
10. **Event Listeners** – Button clicks, resize, key/mouse/touch events

---

## Browser Compatibility

- **Requires:** Modern browser with Canvas 2D, Web Audio API, ES6+ support
- **Tested:** Chrome, Firefox, Safari, Edge
- **Mobile:** Full touch support via joystick interface
- **Accessibility:** Screen reader announcements via `announce()` function

---

## Future Enhancements

Potential areas for extension:
- Persistent progression (level unlocks, achievements)
- Leaderboard integration
- More boss attack patterns (currently ~2 per boss)
- Weapon balance tuning
- Particle effect enhancements
- Sound effects library (currently synthetic)
- Gamepad support (in addition to keyboard/mouse/touch)

---

*This documentation auto-generated for claude-code session compatibility.*

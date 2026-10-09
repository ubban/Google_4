# Sky Fighter - Project Documentation

## Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely as a single-file HTML5 web application. The game runs in any modern browser with WebGL/Canvas support and uses procedural generation for enemies and boss patterns, synthesized audio via WebAudio API, and responsive design for desktop and mobile platforms.

**File:** `index.html` (1669 lines)  
**Type:** Self-contained HTML5 game  
**No build process required** — Open `index.html` directly in a browser to play.

---

## Architecture Overview

The game runs in a single IIFE (Immediately Invoked Function Expression) inside a `<script>` tag. It has three main layers:

1. **HTML & CSS** (lines 1–359): DOM structure and styling
2. **Game Logic & Rendering** (lines 422–1666): Canvas-based game loop with state management
3. **Audio Synthesis** (lines 542–676): WebAudio API sound effects and music cues

### Key Sections

| Component | Purpose | Lines |
|-----------|---------|-------|
| **Canvas Setup** | Initialize 480×720 game canvas | 752–765 |
| **Input Handling** | Keyboard, mouse, and touch controls | 767–861 |
| **Game State** | Player, enemies, bullets, particles, UI | 863–910 |
| **Physics & Collision** | Movement, hit detection, overlap tests | 1148–1302 |
| **Rendering** | Draw canvas, sprites, HUD, effects | 1520–1638 |
| **Game Loop** | Main update/draw loop with delta-time | 1640–1666 |

---

## File Structure

```
Google_4/
├── index.html          # Complete game (HTML + CSS + JS)
├── README.md           # Minimal placeholder readme
└── .git/               # Version control
```

---

## Key Game Features

### Core Mechanics
- **Player Ship**: Steerable craft at screen bottom; has 3–4 lives (difficulty-dependent)
- **Weapons**: 
  - **Vulcan**: Spread shot (3 levels); spread increases at levels 2–3
  - **Laser**: Piercing shots (2–3 lanes); can penetrate enemies
  - Switch weapons via power-ups; weapon resets to level 1 on switch
- **Bombs**: Limited smart bombs that clear enemies and damage boss (max 2–3 per game)
- **Score Chain**: Kill-streak multiplier (1.2–2.0× at 5–50 kills)
- **Power-ups**: 
  - ❤️ Extra life (6% spawn chance)
  - 💣 Bomb refill (8% spawn chance)
  - ⚔️ Weapon upgrade (20% spawn chance)

### Difficulty Levels
| Setting | Spawn Rate | Enemy Speed | Shoot Chance | Lives | Boss HP |
|---------|-----------|-------------|--------------|-------|---------|
| Easy | Slower | 1.0× | 45% | 4 | 0.8× |
| Normal | Medium | 1.4× | 70% | 3 | 1.0× |
| Hard | Fast | 1.9× | 95% | 3 | 1.3× |

### Visual Features
- **6 Procedural Sky Themes**: Daylight, Sunset, Night, Storm, Arctic, Volcanic
- **Screen Shake**: Triggered on hits and bombs
- **Flash Effects**: White overlay on damage, brighter on bomb detonation
- **Vignette**: Red pulsing edge when health critical (1 heart)
- **Particle Bursts**: Explosions for enemies, power-ups, bosses
- **Combo Pop-up**: "X KILLS!" indicator every 10 consecutive kills

### Enemies
| Type | Speed | HP | Score | Notes |
|------|-------|----|----|-------|
| **Grunt** | 1.0× | 1 | 10 | Basic; appears stage 1+ |
| **Interceptor** | 1.7× | 1 | 15 | Fast; weaves; stage 1+ |
| **Tank** | 0.55× | 4 | 35 | Armored; stage 2+; shows health bar |

### Boss Patterns
Each stage has a unique boss with 2–4 attack patterns cycling:
- **VANGUARD** (Stage 1): Radial burst, aimed fan, V-spreads
- **BEHEMOTH** (Stage 2): Heavy columns, slow homing, dense radial wall
- **SPECTER** (Stage 3): Teleport dash, tight spirals, double aimed shots
- **DREADNOUGHT** (Stage 4): 8-way cross, homing missiles, dense radial wall
- **WRAITH** (Stage 5): Teleport barrage, expanding double ring, aimed cross
- **TITAN** (Stage 6+): Heavy columns, full ring, relentless homing

Boss telegraph (warning flash) occurs 22 frames before attack.

---

## Code Organization & Key Functions

### Game State Management
```js
player                 // { x, y, w, h, speed, weapon, level, hp, invuln, cooldown }
bullets, enemies       // Array of projectiles and foes
boss                   // Current boss object (null between stages)
score, lives, stage    // Global game state
```

### Core Update Loop
```js
update(dt)             // Physics, spawning, collisions (called every frame)
draw()                 // Render canvas, HUD, effects
loop(now)              // RequestAnimationFrame loop; manages delta-time
```

### Input Routing
- **Keyboard**: Arrow keys / WASD for movement, Space for fire, X/Shift for bomb, P/Esc for pause
- **Mouse**: Hover to steer, click to fire
- **Touch**: Left 55% of screen for joystick, right 45% for Fire/Bomb buttons

### Collision Detection
```js
rectsOverlap(a, b)     // AABB test for hitting bullets, enemies, power-ups, bosses
```

### Audio (WebAudio Synthesis)
All sounds are procedurally generated (no audio files):
```js
beep(freq, dur, type, vol, opts)     // Oscillator-based tones
noiseBurst(dur, vol, opts)           // Noise-based SFX (explosions, bombs)
sfx.shoot, sfx.laser, sfx.hit, etc.  // Named sound effects
```
- Muting via localStorage: `skyfighter.muted`
- Reduced motion respects system preferences + manual toggle

### Rendering Functions
```js
drawPlane(x, y, w, h, color, darkColor, flame)   // Draw player/enemy ships
drawBossShip(boss)                               // Animate boss with core glow
drawClouds(speed, color, rx, ry)                 // Parallax cloud layers
```

### Difficulty & Persistence
```js
DIFFICULTIES              // Config object for Easy/Normal/Hard
THEMES                    // 6 sky color palettes
ENEMY_TYPES              // Grunt, Interceptor, Tank definitions
BOSSES                   // 6 boss designs with HP multipliers
localStorage             // Saves best score, mute state, difficulty, motion preference
```

---

## Responsive Design & Performance

### Canvas Sizing
- **Base resolution**: 480×720 (16:9 + extra height)
- **Max scale**: 1.6× (capped for high-res displays)
- **Dynamic resize**: Responds to window resize and orientation change

### Adaptive Quality
- Monitors frame times and auto-scales quality (3 levels: full, medium, low)
- Reduces particle count and disables shadow blur at lower quality settings
- Respects `prefers-reduced-motion` media query and adds toggle button

### Touch Optimization
- Detects touch device and enables on-screen joystick + buttons
- Visual joystick with 50px deadzone radius
- Prevents default touch behaviors on game canvas

---

## Development Guidelines

### Adding New Features

**New Enemy Type:**
1. Add entry to `ENEMY_TYPES` object with `w, h, hp, speedMul, color, dark, score, minStage`
2. Update `spawnEnemy()` to include new type in roll table
3. Add explosion SFX mapping in `EXPLODE_SFX` if needed

**New Boss Pattern:**
1. Create pattern function (e.g., `fireRadial(b, count, speed, offset)`)
2. Add function reference to `BOSS_PATTERNS` array (one pattern set per boss index)
3. Test telegraph timing (22 frames is standard)

**New Difficulty:**
1. Add entry to `DIFFICULTIES` with spawn rates, speeds, and HP multipliers
2. Update difficulty selector buttons in HTML
3. Test progression curve (kill count per stage, enemy density, damage output)

### Extending the Game
- **New weapons**: Add to `applyPowerUp()` weapon switch logic; define fireWeapon() behavior
- **New power-ups**: Add type to `maybeSpawnPowerUp()` spawn chance table and `applyPowerUp()` handler
- **New themes**: Append to `THEMES` array (sky gradient stops, cloud color, sun color)
- **New visual effects**: Use `particles` array for burst effects; `screenShake()` for rumble; `flashHit/flashBomb()` for overlay

### Code Style
- **No external dependencies**: Vanilla JS, no frameworks or asset files
- **Single-file constraint**: Keep all code inline for maximum portability
- **Performance first**: Particle counts scale with quality; avoid unnecessary allocations
- **Accessibility**: ARIA labels, screen-reader announcements, reduced-motion support

---

## How to Run & Deploy

### Local Play
1. Open `index.html` directly in a web browser
2. Select difficulty (Easy/Normal/Hard)
3. Click "Start Game"

### Web Deployment
- Upload `index.html` to any static host (GitHub Pages, Netlify, AWS S3, etc.)
- No build step, no backend required
- Fully client-side; uses localStorage for save state

### Browser Compatibility
- Requires: Canvas, WebAudio API, ES6 (arrow functions, const/let, template literals)
- Tested on: Chrome, Firefox, Safari, Edge (modern versions)
- Gracefully disables audio if WebAudio unavailable

---

## Known Limitations & Future Ideas

### Current Constraints
- **Single file**: Easier distribution but harder to refactor
- **No mobile optimization**: Joystick layout could be improved for small screens
- **No netplay**: Single-player only

### Possible Enhancements
1. **Leaderboards**: Replace localStorage with server-side high-score tracking
2. **Custom weapon balance**: Expose difficulty/weapon balance as editor UI
3. **Animation polish**: Add screen transitions, stage intros, victory animation
4. **Sound design**: Ambient music track; more weapon SFX variety
5. **Mobile UX**: Better button sizing for portrait mode; haptic feedback on attack
6. **Stage variety**: Boss-specific level backdrops; dynamic obstacles
7. **Persistence**: Save/restore mid-game state; replay system
8. **Accessibility**: Colorblind modes; subtitle toggles; gameplay difficulty vs visual difficulty separation

---

## Debugging Tips

### Common Issues
- **Audio not playing**: Check browser autoplay policy; click to enable AudioContext
- **Performance dropping**: Check quality slider status (icon button at top-right); lower quality reduces particles
- **Touch controls not responding**: Ensure device detected as touch device; try mobile-like viewport
- **Boss behavior odd**: Verify `frame` counter increments; check telegraph timing (22 frames)

### Quick Tests
- Open DevTools Console
- `boss !== null` — Check if in boss phase
- `player.weapon, player.level` — Check current weapon state
- `enemies.length` — Count active enemies
- `multiplier` — Check kill-streak multiplier
- Set `quality = 2` for maximum performance

---

## Project Stats
- **Total lines**: 1669 (CSS + JS + HTML)
- **Game code**: ~800 lines
- **Audio synthesis**: ~100 lines
- **Rendering/drawing**: ~150 lines
- **UI & input**: ~200 lines
- **Styling**: ~350 lines

---

## Attribution & License
Created as a standalone Raiden-inspired shoot-'em-up. No external assets used. Self-contained WebGL/Canvas game for modern browsers.

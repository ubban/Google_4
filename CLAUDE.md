# Sky Fighter — Claude Development Guide

**Project**: Sky Fighter  
**Type**: Single-file HTML5 game (Canvas-based)  
**Tech Stack**: Vanilla JavaScript, Canvas 2D API, WebAudio API  
**Structure**: `index.html` contains all HTML, CSS, and JavaScript

## Project Overview

Sky Fighter is a **Raiden-style vertical shoot-em-up** game. Players control a fighter plane, manage weapons, dodge bullets, and defeat bosses across multiple stages. The game is fully self-contained in a single HTML file with no external dependencies.

### Key Features
- **Difficulty modes**: Easy, Normal, Hard (affects spawn rates, enemy speed, boss HP, player lives)
- **Weapon system**: Two weapons (Vulcan guns, Laser) with leveling (1-3)
- **Smart bombs**: Limited-use screen-clearing explosions
- **Enemy variety**: Grunts, Interceptors, Tanks (each with unique behavior)
- **Boss encounters**: 6 unique boss designs with distinct attack patterns
- **Visual themes**: 6 themed sky backgrounds with parallax clouds
- **Audio**: Procedurally generated WebAudio synth effects (no external audio files)
- **Input**: Keyboard (arrows + WASD), mouse aim, touch controls with virtual joystick
- **Performance**: Adaptive quality system (downgrades particle effects on low FPS)
- **Accessibility**: Screen reader support, reduced motion preference respects

## Code Structure

The JavaScript follows a game loop pattern with sections marked by comments:

```
┌─ Game initialization (constants, difficulty/theme definitions)
├─ Audio subsystem (beep, noiseBurst, SFX effects)
├─ Input handling (keyboard, mouse, touch, joystick)
├─ Game state management (player, enemies, bullets, particles)
├─ Update logic (physics, collision, spawning, boss AI)
├─ Rendering (canvas drawing, effects, HUD)
└─ Main loop (requestAnimationFrame)
```

## Data Structures

### Player Object
```javascript
{
  x, y, w: 40, h: 40,
  speed: 260,           // pixels/second
  cooldown: 0,          // weapon fire cooldown
  weapon: 'vulcan'|'laser',
  level: 1-3,           // weapon level
  invuln: 0-1.5         // invulnerability timer (seconds)
}
```

### Enemy Object
```javascript
{
  x, y, w, h,
  type: 'grunt'|'interceptor'|'tank',
  speed: number,        // pixels/frame
  hp: number,           // health points
  shootTimer: number,   // frames until next shot
  pattern: 'spread'|'aimed',
  weave: boolean,       // serpentine movement
  baseX: number,        // origin X for weaving
  color, dark: string   // fill colors for rendering
}
```

### Boss Object
```javascript
{
  x, y, w: 90, h: 80,
  design: {...},        // from BOSSES array
  hp, maxHp: number,
  entering: boolean,    // slide-in animation
  telegraph: 0-22,      // warning flash before attack
  patternIndex: number, // current attack pattern
  vx: ±1                // horizontal velocity
}
```

### Bullet Object (player)
```javascript
{ x, y, w, h, speed: 500|640, vx: number, pierce: 0-2, laser: boolean }
```

### Bullet Object (enemy)
```javascript
{ x, y, w, h, radial: boolean, dx: number, dy: number }
```

### Power-up Object
```javascript
{ x, y, w: 20, h: 20, speed: 90, type: 'life'|'bomb'|'weapon' }
```

### Particle Object (explosion)
```javascript
{ x, y, vx, vy, life: 0-max, maxLife: 0.5, color: string }
```

## Key Constants & Configuration

### Difficulty Settings (DIFFICULTIES)
- **spawnBase/spawnMin**: Enemy spawn rate (frames between spawns)
- **enemySpeed**: Multiplier for enemy movement speed
- **enemyShootChance**: Probability of enemies firing (0-1)
- **lives, bombs**: Starting player resources
- **bossHpMul**: Boss health multiplier

### Enemy Types (ENEMY_TYPES)
```javascript
{
  w, h: dimensions,
  hp: health,
  speedMul: movement multiplier,
  color, dark: render colors,
  score: points on kill,
  minStage: first appearance stage,
  weave: serpentine movement (interceptor only)
}
```

### Boss Types (BOSSES)
Six designs with unique color schemes and attack patterns:
1. **VANGUARD**: Radial burst, aimed fan, wide V-fan
2. **BEHEMOTH**: Heavy columns, slow homing, dense radial
3. **SPECTER**: Teleport dash, tight spiral, double aimed
4. **DREADNOUGHT**: 8-way cross, homing missiles, dense radial
5. **WRAITH**: Teleport barrage, double ring, cross-spread
6. **TITAN**: Heavy columns, full ring, relentless homing

### Themes (THEMES)
Six sky backgrounds with gradient and cloud colors, appear in order.

## Game Loop & Timing

The main loop uses `requestAnimationFrame`:

```javascript
function loop(now) {
  const dt = Math.min((now - lastTime) / 1000, 0.05); // Frame time, capped at 50ms
  
  if (running && !paused) {
    update(dt);      // Physics, collision, AI
    draw();          // Render to canvas
  }
  requestAnimationFrame(loop);
}
```

**Frame budget**: 16.67ms @ 60Hz; adaptive quality reduces particles if avg > 25ms.

## Collision & Physics

### Overlap Detection
```javascript
rectsOverlap(a, b) → Math.abs(a.x - b.x) < (a.w + b.w) / 2 && ...
```

Uses **axis-aligned bounding box (AABB)** collision.

### Key Physics Rules
- Player speed: 260 px/s (keyboard/joystick), smooth interpolation (mouse)
- Bullets: Straight line upward (player) or radial/aimed (enemies)
- Gravity: None (space shooter style)
- Wrapping: Canvas-confined movement with clamping

## Scoring System

### Chain Mechanic
- Each kill increments `chain` counter (up to max multiplier of 10x)
- Chain timer: 2.2 seconds without a kill → breaks chain
- Multiplier: `1 + floor(chain / 5)` (1x at 0-4 kills, 2x at 5-9, etc.)
- Bomb kills add 5 base points × multiplier

### Base Scores
- Grunt: 10 points
- Interceptor: 15 points
- Tank: 35 points
- Boss defeat: 500 points (+ chain bonus)

## Progression & Stages

### Stage System
- Each stage requires **18 kills** to trigger boss encounter
- Boss appears ~1.2 seconds after 18th kill
- Boss health: `(70 + stage × 35) × difficulty_mul × boss_design_mul`
- Stages loop through 6 themes and 6 bosses (repeats every 6 stages)

### Player Progression
- **Weapons**: Start with Vulcan L1
  - Vulcan L1: Single spread | L2: Double spread | L3: Triple spread
  - Laser L1: Single lane | L2: Dual lanes | L3: Triple lanes (with pierce)
  - Power-ups switch weapon or increase level within weapon
- **Lives**: Start with 3 (Easy) or 2 (Normal/Hard)
  - Lose 1 on hit → invulnerable 1.5s
  - Game over at 0 lives
- **Bombs**: Start with 2-3 (difficulty-dependent)
  - Clears all on-screen enemies & enemy bullets
  - Deals 24 damage to boss

## Input Handling

### Keyboard
- **Arrow Keys / WASD**: Move
- **Space**: Fire
- **X / Shift**: Use bomb
- **P / Esc**: Pause

### Mouse
- **Move**: Player glides toward cursor (lower priority than keyboard/joystick)
- **Click**: Fire

### Touch (Mobile)
- **Left zone (55% width)**: Virtual joystick (shown on touch)
- **Right zone (45% width)**: Fire + Bomb buttons
- **Fire button**: Continuous while pressed
- **Bomb button**: Tap to activate

## Rendering

### Canvas Setup
- **Base dimensions**: 480×720 pixels
- **Scaling**: Maintains aspect ratio, max 1.6× zoom
- **Rendering order**:
  1. Sky gradient + sun glow
  2. Parallax clouds (2 layers)
  3. Player plane
  4. Enemies (with tank health bar)
  5. Boss
  6. Particles (explosions)
  7. Bullets (player blue/yellow, enemies red)
  8. Power-ups (rotating)
  9. Screen effects (vignette, flash, combo popups)

### Special Effects
- **Vignette**: Red pulse on critical health (1 life remaining)
- **Flash**: White flash on player hit (0.25s) or bomb (0.5s)
- **Screen shake**: Triggered by damage, bombs, boss defeat
- **Combo popup**: Text animation at 10, 20, 30... kills
- **Stage banner**: Full-screen text on stage/boss entry

### Performance Adaptation
```javascript
quality: 0 (full) | 1 (medium) | 2 (low)
// Affects: particle count, shadow/glow effects, shake intensity
```

## Audio System

### WebAudio Synth
All sounds generated in real-time using oscillators, filters, and noise bursts.

### SFX Effects
- `shoot()`: Square wave sweep (player vulcan)
- `laser()`: Sawtooth sweep (player laser)
- `explodeGrunt()`: Low frequency sweep + noise
- `explodeInterceptor()`: Dual square beeps
- `explodeTank()`: Heavy noise + pitch sweep
- `hit()`: Player damage (triangle wave)
- `bomb()`: Large explosion sound (multi-layer)
- `powerup()`: Ascending sine sweep (musical)
- `bossHit()`: Sharp square beep
- `bossTelegraph()`: Ascending pitch sweep
- `bossDown()`: Descending pitch sequence
- `stageClear()`: Musical 4-note sequence
- `uiClick()`: Short square beep (UI feedback)
- `gameover()`: Descending sad trombone

### Audio Context
- **Master gain**: 0.9
- **Compressor**: Threshold -20dB, ratio 5:1
- **Muting**: Stored in localStorage (`skyfighter.muted`)

## Storage (localStorage)

```javascript
skyfighter.bestScore      // Best score (persistent)
skyfighter.muted          // Audio mute state (true/false)
skyfighter.difficulty     // Last selected difficulty
skyfighter.reducedMotion  // Reduced motion preference (true/false)
```

## UI Elements

### HUD (Top-center overlay)
- Score, Best, Multiplier, Weapon name+level, Hearts, Bombs (pill-shaped badges)

### Boss Bar (Top-center, appears during boss fight)
- Boss name, health bar with gradient fill

### Top-right buttons
- 🔊/🔇 Mute toggle
- 🎆/🚫 Reduced motion toggle
- ⏸/▶ Pause button

### Overlay (Full-screen modal)
- Title (Sky Fighter / Game Over / Paused)
- Subtitle & stats
- Difficulty selector buttons
- Start / Resume / Play Again button
- Controls hint text

### Screen Reader Support
`#srLive` element announces events: "10 lives remaining", "Boss defeated", etc.

## Development Guidelines

### Adding a New Feature

**Boss**: Add entry to `BOSSES` array and pattern array in `BOSS_PATTERNS`

```javascript
BOSSES.push({
  name: 'NEWBOSS',
  hullTop: '#...', hullBottom: '#...',
  wing: '#...', core: '#...',
  hpMul: 1.2
});

BOSS_PATTERNS.push([
  b => fireRadial(b, 12, 150),
  // ... attack functions
]);
```

**Enemy**: Add to `ENEMY_TYPES`, set `minStage` to control appearance

**Weapon**: Modify `fireWeapon()` function, add pierce/spread logic

**Theme**: Append to `THEMES` array with sky gradient colors and cloud RGB

### Code Style
- **No build step**: All vanilla JS, no transpiling
- **Single scope**: Everything in IIFE closure at bottom of `<script>`
- **Comments**: Section markers (---) only; code is self-documenting
- **Naming**: camelCase for variables, CONSTANTS for immutables
- **Performance**: Avoid DOM queries in game loop; use Canvas only

### Testing
- Open `index.html` in browser (no server needed)
- Press Dev Tools to check console for errors
- Test at multiple screen sizes / devices
- Verify audio context resumes on user interaction
- Check localStorage persistence across refreshes

## Common Patterns

### Spawning Entities
```javascript
entities.push({ x, y, /* ...props */ });
```

### Removing Entities
```javascript
array = array.filter(e => !shouldRemove(e));
```

### Particle Effects
```javascript
spawnExplosion(x, y, color, count);
```

### Screen Shake
```javascript
screenShake(magnitude, duration);
```

### Announcements
```javascript
announce('Text for screen readers');
```

## Performance Considerations

- **Single canvas context**: Reused for all drawing
- **No image loading**: Vector art + procedural effects
- **Pooling**: Particles/bullets added/removed dynamically
- **Collision: O(n²)** → acceptable with max ~100 active entities
- **Frame cap**: 50ms delta max prevents physics instability
- **Touch events**: `passive: false` for preventDefault on critical handlers

## Known Limitations & Future Ideas

### Current Limitations
- Boss patterns hardcoded per design (no randomization)
- No leaderboard backend
- No pause persistence (paused state lost on reload)
- Difficulty cannot change mid-game

### Enhancement Opportunities
- Add combo multiplier visual feedback
- Implement weapon switching (two weapons at once)
- Add ship customization
- Boss pattern procedural generation
- Sound effects volume slider
- Fullscreen mode
- Local high-score sharing (QR code)

## Resources

- Canvas API: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API
- WebAudio API: https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API
- RequestAnimationFrame: https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame

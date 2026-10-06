# Sky Fighter - Codebase Documentation

## Overview

Sky Fighter is a **Raiden-style vertical shoot-'em-up game** implemented as a single-file HTML5 canvas game with no external dependencies. The entire game is contained in `index.html` (~1670 lines) with embedded CSS styling and vanilla JavaScript gameplay logic.

**Key Characteristics:**
- Canvas-based 2D rendering (480x720 base resolution)
- Full responsive/adaptive scaling for any screen size
- WebAudio API for procedurally synthesized sound effects (no audio files needed)
- Touch and keyboard input support with joystick on mobile
- Three difficulty levels (Easy, Normal, Hard)
- Six unique boss designs with distinct attack patterns
- Progressive stage system with increasing difficulty
- Persistent best score storage in localStorage

---

## Project Structure

```
index.html
├── HTML markup (game container, HUD, overlay, touch controls)
├── CSS (inline <style> block)
│   ├── Layout and responsive scaling
│   ├── HUD styling (score, health, weapons, boss bar)
│   ├── Overlay UI (start screen, game over, pause menu)
│   ├── Animation keyframes (screen shake, combo pop, stage banner)
│   └── Theme-aware styling (dark mode support, reduced motion)
└── JavaScript (inline <script> block)
    ├── Core game loop
    ├── Entity management (player, enemies, bullets, particles)
    ├── Game state and difficulty system
    ├── Input handling (keyboard, mouse, touch)
    ├── Audio synthesis functions
    ├── Rendering pipeline
    └── Boss AI and attack patterns
```

---

## Game Architecture

### 1. **Game State**

```javascript
// Core game variables (line ~865)
player          // { x, y, w, h, speed, cooldown, weapon, level, invuln }
bullets         // player projectiles
enemies         // spawned enemy ships
enemyBullets    // enemy fire
particles       // explosion and visual effects
powerUps        // collectible upgrades
boss            // current boss (if in fight)
score           // current score
lives           // remaining lives
multiplier      // combo multiplier (1 + floor(chain/5))
stage           // current stage (1-6+)
```

### 2. **Game Loop**

The main animation loop runs at 60 FPS via `requestAnimationFrame`:
1. **trackPerf()** — Monitor frame time for adaptive quality
2. **update(dt)** — Update all entities and game logic (only if running && !paused)
3. **draw()** — Render the entire scene
4. Repeat

Quality levels (0=full, 1=medium, 2=low) adjust particle counts and glow effects based on frame timing.

### 3. **Difficulty System**

Three presets with scalable parameters (line ~485):

```javascript
DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
};
```

- **spawnBase / spawnMin**: Enemy spawn rate (higher = less frequent)
- **enemySpeed**: Multiplier for enemy movement speed
- **enemyShootChance**: Probability enemy fires (0-1)
- **lives / bombs / bossHpMul**: Starting resources and boss health scaling

---

## Gameplay Systems

### **Enemy Spawning**

- Enemies spawn at random x positions above the screen and descend
- Spawn rate decreases as frame count increases (difficulty ramps naturally)
- Enemy type varies by stage and random roll:
  - **grunt**: Basic red fighter, 1 HP, quick
  - **interceptor**: Fast orange weaver with erratic movement
  - **tank**: Slow tanky purple unit, 4 HP, worth more points (stage 2+)

### **Weapon System**

Two weapons with 3 upgrade levels each:

| Weapon | Level 1 | Level 2 | Level 3 |
|--------|---------|---------|---------|
| **Vulcan** | 1 center spread | 2-wide spread | 3-wide spread |
| **Laser** | 1 center beam | 2 offset beams | 3 beams (center + 2 offset) |

- Fire delay: Vulcan = 0.15s, Laser = 0.1s (laser faster)
- Laser shots pierce enemies (up to 2-3 hits)
- Weapon pickups cycle: Vulcan Lv1 → Lv2 → Lv3 → Laser Lv1 → (cycle)

### **Boss System**

Triggered after **18 kills** per stage. Each stage (1-6) spawns a unique boss, cycling through:

1. **VANGUARD** — Radial burst, aimed fan, wide V-fan
2. **BEHEMOTH** — Heavy columns, homing spread, dense radial wall
3. **SPECTER** — Teleport dash, tight spiral, double aimed shots
4. **DREADNOUGHT** — 8-way cross, homing missiles, radial wall
5. **WRAITH** — Teleport barrage, double ring, aimed cross-spread
6. **TITAN** — Heavy columns, full ring, homing volley

Each boss has:
- Base HP: `70 + stage * 35` (multiplied by difficulty and design-specific multipliers)
- Movement: Bounces left-right at y=120
- Attack pattern: Cycles through 3 distinct firing patterns with telegraph delay (22 frames)
- Entering animation: Descends from top before engaging

Boss damage:
- Player bullets: 3 HP per hit
- Bomb: 24 HP
- Takes 3 seconds between attack telegraphs

### **Power-ups**

Spawn (6%, 8%, 20% chance respectively) when enemies die:
- **Life** (red): Restore 1 health
- **Bomb** (yellow): Add 1 smart bomb (resets on stage clear)
- **Weapon** (lime green): Upgrade current weapon or switch to alternate

### **Combo/Multiplier System**

- **Chain**: Increments on every enemy kill, resets after 2.2s without kills
- **Multiplier**: `1 + floor(chain / 5)` (1x → 2x at 5 kills → 3x at 10 kills, etc., max 10x)
- Score gain: `baseScore * multiplier`
- Multiplier resets on taking damage

### **Smart Bombs**

- **Quantity**: 2-4 per difficulty (easymode: 3)
- **Effect**: Clears all enemy bullets, kills all enemies on screen (+5 score each), damages boss (24 HP)
- **Activation**: X / Shift / BOMB button
- **Visual feedback**: White flash, screen shake, explosion particles

---

## Input Handling

### **Keyboard**
- **Movement**: Arrow keys or WASD
- **Fire**: Space bar or click
- **Bomb**: X or Shift
- **Pause**: P or Esc
- **Difficulty**: Click difficulty buttons (only before game starts or while paused)

### **Mouse**
- **Aiming**: Plane gravitates toward mouse cursor (lower priority than keyboard/joystick)
- **Fire**: Left click

### **Touch (Mobile)**
- **Joystick zone**: Left 55% of screen (drag to move)
- **Action zone**: Right 45% of screen
  - **FIRE button**: Red circular button (bottom-right)
  - **BOMB button**: Yellow circular button (bottom-left of action zone)

Both keyboard and joystick input use a **magnetic** system: if keyboard/joystick input exists, it takes priority over mouse/cursor aiming.

---

## Audio System

**No external audio files.** All sounds are synthesized in real-time using WebAudio API.

### **Sound Effect Functions** (line ~624-676)

```javascript
sfx = {
  shoot: () => beep(760, 0.07, 'square', ...),      // Vulcan fire
  laser: () => beep(1500, 0.06, 'sawtooth', ...),   // Laser fire
  explodeGrunt: (x) => { ... },                     // Grunt death
  explodeInterceptor: (x) => { ... },               // Interceptor death
  explodeTank: (x) => { ... },                      // Tank death
  hit: () => { ... },                               // Player hit
  bomb: () => { ... },                              // Smart bomb
  powerup: () => { ... },                           // Pickup collected
  bossHit: (x) => { ... },                          // Damage to boss
  bossTelegraph: () => { ... },                     // Boss attack warning
  bossDown: () => { ... },                          // Boss defeated
  stageClear: () => { ... },                        // Stage complete
  uiClick: () => { ... },                           // Button click
  gameover: () => { ... }                           // Game over
};
```

### **Synthesizer Primitives**

- **beep()** — Oscillator with optional frequency sweep, filter, panning
- **noiseBurst()** — Procedural noise for explosion/impact sounds
- **panFor()** — Calculate stereo pan based on x position on canvas

Mute state persists in localStorage (`skyfighter.muted`).

---

## Rendering Pipeline

### **Canvas Sizing** (line ~752-764)

Base resolution: **480×720 px**
Scales up to fit screen while maintaining aspect ratio (max 1.6x on very large displays).

```javascript
const scale = Math.min(maxW / BASE_W, maxH / BASE_H, 1.6);
canvas.width = BASE_W;
canvas.height = BASE_H;
canvas.style.width/height = scaled dimensions;
```

### **Visual Rendering** (draw function, line ~1520-1638)

1. **Background**: 
   - Gradient sky (3-color per theme, updates per stage)
   - Sun glow (radial gradient)
   - Parallax clouds (2 layers at different speeds)

2. **Game Objects**:
   - Player plane (with flame when firing)
   - Enemies (each colored uniquely)
   - Tank health bars (above unit)
   - Bullets (yellow vulcan, cyan laser, with glow in high-quality mode)
   - Enemy fire (red glowing circles)
   - Power-ups (rotating, color-coded)
   - Particles (explosion debris, fading)
   - Boss ship (complex multi-part design)

3. **HUD** (HTML-based, not canvas):
   - Score, best score (top)
   - Combo multiplier (orange)
   - Weapon name + level (green)
   - Health hearts (emoji ❤️, critical animation)
   - Bombs (emoji 💣, filled when available)
   - Boss health bar (when boss active)
   - Combo pop (gold text, scales and fades)
   - Stage banner (blue text, 2s display)

4. **Screen Effects**:
   - **Screen shake**: Random pixel offset each frame (magnitude + duration)
   - **Flash**: White overlay for hits/bombs
   - **Vignette**: Red radial gradient when health critical (1 HP)

### **Themes** (line ~493-500)

Six stage themes with unique sky gradients and color schemes. Cycles after stage 6.

---

## Key Constants

### **Game Balance**

| Constant | Value | Purpose |
|----------|-------|---------|
| BASE_W, BASE_H | 480, 720 | Canvas dimensions |
| KILLS_PER_STAGE | 18 | Enemies to kill before boss spawns |
| STORAGE_* | String keys | localStorage keys for persistence |

### **Entity Dimensions**

```javascript
// Enemy sizes (px)
grunt:       { w: 34, h: 34, hp: 1, speedMul: 1.0, score: 10 }
interceptor: { w: 26, h: 26, hp: 1, speedMul: 1.7, score: 15 }
tank:        { w: 46, h: 46, hp: 4, speedMul: 0.55, score: 35 }
boss:        { w: 90, h: 80 }

// Bullets
player:      { w: 4-5, h: 12-20 }
enemy:       { w: 4-6, h: 10-16 }
```

---

## Common Modification Points

### **To adjust game balance:**
1. **Enemy difficulty**: Edit `DIFFICULTIES` object
2. **Boss HP**: Modify base formula in `spawnBoss()` (line ~1046)
3. **Weapon damage**: Change player bullet damage in boss collision (line ~1259, change `3` to new value)
4. **Score multipliers**: Edit `addScore()` calculation (line ~1091)
5. **Spawn rate**: Adjust `spawnTimer` logic (line ~1198-1202)

### **To add content:**
1. **New enemy type**: Add to `ENEMY_TYPES`, extend `spawnEnemy()` logic, add explosion SFX
2. **New boss**: Add to `BOSSES` array, define attack pattern in `BOSS_PATTERNS` (line ~1372)
3. **New stage theme**: Add to `THEMES` array
4. **New audio**: Add function to `sfx` object using `beep()` and `noiseBurst()` primitives

### **To tweak visuals:**
1. **Colors**: Search for hex codes or named colors in `ENEMY_TYPES`, `BOSSES`, `THEMES`
2. **Screen shake**: `screenShake(magnitude, duration)` calls throughout code
3. **Animations**: Keyframe timings in CSS `@keyframes` blocks
4. **Particle effects**: `spawnExplosion(x, y, color, count)` parameters

---

## Performance Notes

- **Adaptive quality**: Automatically drops particle counts and disables glows on slower devices
- **Entity pooling**: Particles and bullets culled when off-screen (no garbage collection stress)
- **Canvas rendering**: No image assets means fast loading and offline-compatible
- **Mobile optimization**:
  - Reduced animation when `prefers-reduced-motion` is set
  - Touch events are passive to prevent scroll lag
  - Joystick provides haptic-friendly control

---

## Storage & State

All state persists in localStorage (no server backend required):

```javascript
skyfighter.bestScore      // Best score achieved
skyfighter.muted          // Audio mute state (0/1)
skyfighter.difficulty     // Last selected difficulty
skyfighter.reducedMotion  // Accessibility preference (0/1)
```

---

## Debugging Tips

- **Inspect game state**: Breakpoint in the game loop `update()` function
- **Performance**: Check `quality` variable (0=full, 1=medium, 2=low) and `frameTimes` array
- **Audio issues**: Check if `audioCtx` is non-null; WebAudio requires user gesture first
- **Touch controls**: Test in browser DevTools device emulation; joystick base appears under `#joystickBase` element
- **Difficulty ramps**: Enemy spawn rate decreases by 4 per 300 frames; compare `spawnTimer` output between difficulties

---

## Future Enhancement Ideas

1. **Leaderboard**: POST scores to backend
2. **Weapon reload mechanics**: Add heat management to Vulcan
3. **Enemy special attacks**: Add beam weapons, seekers beyond current patterns
4. **Power-up shield**: Temporary invulnerability pickup
5. **Stage-specific events**: Random screen hazards, environmental obstacles
6. **Combo visual feedback**: Particle trails, color shifts at high multipliers
7. **Replayability**: Save and replay game sessions
8. **Customization**: Ship color/design selection at start

---

**Last Updated**: 2026-10-06  
**Game Type**: Vertical Shoot-'em-up (Bullet Hell variant)  
**Technology**: HTML5 Canvas, WebAudio API, Vanilla JavaScript (ES6+)

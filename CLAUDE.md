# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely as a single HTML5 file. The game features:
- Canvas-based 2D rendering (480x720 game viewport)
- Responsive design that scales to any screen size
- Multiple difficulty levels (Easy, Normal, Hard)
- Dynamic enemy spawning with difficulty ramping
- Six boss encounters with unique attack patterns
- WebAudio API-based procedural sound effects (no external audio files)
- Touch and keyboard/mouse controls
- LocalStorage persistence for high scores and settings
- Adaptive frame-rate performance tracking
- Accessibility features (screen reader announcements, reduced motion preferences)

## Architecture

The entire game logic lives in a single 1670-line IIFE (Immediately Invoked Function Expression) in the `<script>` block of `index.html`. The code is organized into logical sections:

### Core Game Loop
- `loop(now)` – RequestAnimationFrame-based main loop that calls `update()` and `draw()`
- `update(dt)` – Advances game state: player movement, bullet/enemy positions, collision detection, score/chain mechanics
- `draw()` – Renders everything to canvas with adaptive quality settings
- `frame` counter – Global frame counter incremented each update, used for animation timing

### Game State Management
- **Player**: Position, dimensions, weapon type/level, cooldown timers, invulnerability frames
- **Entities**: Bullets, enemies, enemy bullets, particles, power-ups, boss
- **UI State**: Score, lives, bomb count, multiplier, stage, kills, pause state
- **Settings**: Difficulty, mute, reduced motion preference, best score (all persisted to localStorage)

### Input Handling
- Keyboard: Arrow keys / WASD for movement, Space for fire, X / Shift for bombs, P / Esc for pause
- Mouse: Position while hovering canvas steers the plane; click to fire
- Touch: Left 55% of screen is movement zone (virtual joystick), right 45% has FIRE/BOMB buttons
- Input logic prioritizes keyboard/joystick over mouse to avoid conflicts

### Rendering Pipeline
1. **Sky gradient** based on current stage theme (6 themes cycle)
2. **Sun glow** radial gradient overlay
3. **Parallax clouds** at two speed layers
4. **Player plane** with optional laser aura, flickering during invulnerability
5. **Bullets** (yellow vulcan or cyan laser) with optional glow effects
6. **Enemy bullets** as red circles
7. **Enemy planes** with hp bars for tanks
8. **Boss ship** with unique design, core glow, rotating accessories, telegraph effects
9. **Power-ups** (rotating colored squares)
10. **Particles** (explosion fragments with fading opacity)
11. **Screen shake** applied via canvas transform

### Performance Adaptation
- `quality` variable (0=full, 1=medium, 2=low) adjusts dynamically
- High-end: glow effects on bullets/bosses, full particle counts (14-22 fragments)
- Low-end: no glow, reduced particle counts (3-8 fragments)
- Tracked via 45-frame rolling average of frame times; adjusts if avg > 1/40s or < 1/56s

### Audio System
- `ensureAudio()` – Lazy-initializes Web Audio context on first user interaction
- `beep(freq, dur, type, vol, opts)` – Plays tone with optional frequency sweep, filter, pan
- `noiseBurst(dur, vol, opts)` – Plays filtered white noise (used for explosions)
- All SFX defined in `sfx` object (shoot, laser, hit, bomb, powerup, boss telegraph, etc.)
- Master gain and compressor for volume control and dynamic range
- `panFor(x)` – Stereo pan based on horizontal screen position

### Enemy System
- **Three enemy types** (grunt, interceptor, tank) with different HP, speed, and scoring
- `spawnEnemy()` – Random spawn at top of screen with weaving or aimed bullet patterns
- **Difficulty scaling**: Enemy spawn rate, speed, shoot chance, and HP multipliers scale by difficulty
- **Stage progression**: Tanks unlock at stage 2; spawn rate tightens over time
- **Weaving**: Interceptors sinusoidally oscillate; visual variety without predictability

### Boss System
- **Six boss designs** with unique color schemes and HP multipliers
- `spawnBoss()` – Called after 18 stage kills, enters from top with warning banner
- **Six attack patterns** per boss design (radial bursts, aimed spreads, spirals, columns, homing, crosses)
- Boss telegraph phase (visual core flash + SFX) before each pattern fires
- Boss movement: Horizontal oscillation with wall bounces
- Boss bullets: Custom radial movement (absolute velocity, not inherited)

### Scoring & Progression
- **Chain mechanic**: Consecutive kills without a 2.2s gap = multiplier (1 + kills/5, max 10x)
- **Score per entity**: Grunts 10, Interceptors 15, Tanks 35, Bosses 500 base
- **Stage progression**: Kill 18 enemies → boss spawns → boss defeat → next stage
- **Power-ups**: 6% life, 8% bomb, 20% weapon (randomly dropped on kill)
- **Weapon system**: Vulcan (direct spray, spreads at level 2-3) vs. Laser (piercing, fast)

### UI/HUD Elements
- Persistent top HUD: Score, Best, Multiplier, Weapon name/level, Hearts, Bombs
- Boss bar: Name and HP track (only visible when boss present)
- Overlay: Initial menu with difficulty selector, game-over screen, pause screen
- Combo pop: "X KILLS!" text at 10, 20, 30… kill milestones
- Stage banner: Appears on stage start and boss warning
- Vignette & flash effects: Red pulse when 1 HP left; white flash on hit; bomb flash on use
- Touch controls: Visual zone dividers and button overlays on mobile

## Common Development Tasks

### Running the Game Locally
Simply open `index.html` in a modern browser. No build process, no external dependencies.

### Testing Different Difficulty Levels
On the start menu, click Easy/Normal/Hard buttons to switch. Selection is saved to localStorage.

### Adjusting Game Balance
- `DIFFICULTIES` object (line ~485): Spawn rates, enemy speed, shoot chance, lives, bomb count, boss HP multiplier
- `KILLS_PER_STAGE` (line 491): Number of kills before boss appears
- `BOSSES` array (line 508): Boss colors and HP multipliers
- `ENEMY_TYPES` object (line 502): Enemy stats (size, HP, speed, score)
- `BOSS_PATTERNS` array (line 1372): Boss attack patterns (each is array of pattern functions)

### Adding a New Enemy Type
1. Add to `ENEMY_TYPES` object with properties: `w, h, hp, speedMul, color, dark, score, minStage, optional weave`
2. Add to `spawnEnemy()` filter to control spawn chance at different stages
3. Optional: Add SFX in `sfx` object and map it in `EXPLODE_SFX` for death sound

### Adding a New Boss Pattern
Each boss has up to 3 attack patterns in `BOSS_PATTERNS`. Pattern functions take boss object `b` and fire bullets via `enemyBullets.push()`. Helper functions:
- `fireRadial(b, count, speed, offset)` – Symmetric burst around boss
- `fireAimedSpread(b, count, speed, spreadStep)` – Fan aimed at player
- `fireColumns(b, speed)` – Vertical columns
- `fireSpiralBurst(b, speed)` – Rotating 8-way burst
- `fireCross(b, speed)` – 8-way cardinal + diagonal
- `fireHomingSeeded(b, count, speed)` – Spread aimed at player with randomness
- `fireDoubleRing(b, speedA, speedB)` – Two concentric rings

All enemy bullets stored in `enemyBullets` array as objects with position, velocity, and size.

### Visual Tweaks
- **Canvas colors & gradients**: Defined in theme objects under `THEMES` array (line 493); each has sky gradient colors, cloud color, sun color, and name
- **Animation timings**: Keyframe durations in CSS (comboPulse, stagePulse, heartBeat, etc.)
- **Screen shake**: `screenShake(mag, dur)` called on hits and bombs; amplitude reduced under reduced-motion
- **Particle effects**: `spawnExplosion(x, y, color, count)` creates colored particle bursts

### Audio Customization
- Modify `sfx` object functions to change existing SFX or add new ones
- `beep()` creates pitched tones; `noiseBurst()` creates noise-based SFX
- Parameters: frequency, duration, waveform type (sine/square/sawtooth/triangle), volume
- Optional: `sweepTo` for pitch glide, `filterFreq`/`filterType` for filters, `pan` for stereo, `delay` for timing

## Key Technical Notes

### Canvas Coordinate System
- Origin (0, 0) at top-left
- X increases rightward, Y increases downward
- Game world is 480×720; scaled to fit viewport while maintaining aspect ratio

### Collision Detection
- Simple AABB (axis-aligned bounding box): `rectsOverlap(a, b)` checks center distance vs. half-widths
- Used for: bullets ↔ enemies, bullets ↔ boss, enemies ↔ player, bullets ↔ player, power-ups ↔ player

### LocalStorage Keys
- `skyfighter.bestScore` – High score
- `skyfighter.muted` – Mute toggle (0/1)
- `skyfighter.difficulty` – Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion` – Motion reduction toggle (0/1)

### Reduced Motion Accessibility
- Respects `prefers-reduced-motion` media query on page load
- Toggleable via UI button (icon shows 🎆 or 🚫)
- Disables animations (screen shake, flashes, pulse effects) when enabled
- Uses shorter effect durations when enabled

### Touch Input Quirks
- Fire button held = continuous shooting (mousedown pattern adapted for touch)
- Joystick drag has 50px radius deadzone
- Both zones (`moveZone`, `actionZone`) have `touch-action: none` to prevent browser defaults

### Performance Profiling
The `frameTimes` array tracks last 45 frame deltas and triggers quality downgrades if avg > 1/40s (25ms). On low-end devices, particle counts drop and glow effects disable. Watch the adaptive quality when frame times worsen.

## File Structure
```
index.html
├── <meta> tags (viewport, description, theme color, favicon)
├── <style> (embedded CSS: layout, animations, responsive design)
└── <script> (embedded game logic in IIFE)
    ├── Canvas setup & resizing
    ├── Input handlers (keyboard, mouse, touch)
    ├── Audio system (Web Audio synthesis)
    ├── Game state (player, enemies, score, etc.)
    ├── Update logic (movement, collision, spawning)
    ├── Rendering (sky, entities, particles, HUD)
    ├── Game-over & pause flow
    └── Main loop with requestAnimationFrame
```

No external dependencies, no build step, no assets folder.

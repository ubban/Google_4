# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file, zero-dependency vertical shoot-'em-up game (Raiden-style). The entire game—HTML, CSS, and JavaScript—is contained in `index.html` (~1670 lines).

### Running the Game

The game requires no build process. Simply open `index.html` in a web browser:
- **Local**: `file:///path/to/index.html` or use a local server (e.g., `python -m http.server 8000`)
- **Online**: Deploy the HTML file to any static hosting

### Key Features

- **Three difficulty levels**: Easy, Normal, Hard (affecting spawn rates, enemy speed, boss HP)
- **Two weapon systems**: Vulcan (spread fire) and Laser (piercing shots) with 3 level upgrades
- **Six boss designs**: Each with unique attack patterns and visual styles
- **Mobile support**: Touch controls (joystick + on-screen buttons) for iOS/Android
- **Accessibility**: Keyboard shortcuts, reduced-motion support, screen reader announcements
- **Persistence**: Saves best score, difficulty, mute state, and motion preference to localStorage
- **Web Audio**: All sound effects synthesized in real-time (no external audio files)
- **Adaptive performance**: Automatically reduces particle counts and glow effects under framerate pressure

---

## Code Architecture

### Top-Level Structure

The entire game logic runs in an IIFE (Immediately Invoked Function Expression) starting at line 423. The code is organized into functional sections:

1. **DOM setup & references** (lines 426–461): Caches all element references
2. **Constants & data** (lines 463–523): Difficulty presets, enemy types, boss designs, themes, patterns
3. **Utility functions** (lines 525–738): Audio, UI helpers, motion preferences
4. **Input handling** (lines 768–862): Keyboard, mouse, touch, and joystick
5. **Game state & logic** (lines 863–1303): Update loop, collision detection, scoring
6. **Rendering** (lines 1520–1638): Canvas drawing for all entities
7. **Main loop** (lines 1640–1665): RequestAnimationFrame entry point

### Game State

**Key variables** (initialized around line 864):
- `player`: Position, health, weapon, cooldown, invulnerability timer
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`: Entity lists
- `score`, `lives`, `stage`, `chain`, `multiplier`: Player metrics
- `boss`: Current boss entity (or null)
- `running`, `paused`, `gameOver`: Game mode flags

### Core Game Loop

**update(dt)** (line 1148):
- Reads input and moves the player
- Updates bullets, enemies, and particles
- Handles enemy/boss spawning
- Performs collision detection
- Updates HUD and visual effects

**draw()** (line 1520):
- Renders sky with parallax clouds
- Draws all entities (player, enemies, boss, bullets, particles)
- Applies screen shake offset
- Renders UI overlays (HUD, pause, game over)

**loop(now)** (line 1640):
- RAF entry point; calculates delta-time
- Calls update and draw
- Tracks performance for adaptive quality system

### Enemy & Boss System

**Enemy spawning** (line 1022):
- Randomly chooses from three types based on stage: `grunt`, `interceptor`, `tank`
- Each type has different HP, speed, score value, and visual style
- Enemies can weave side-to-side (interceptors only)
- Shoot patterns: either "spread" (fixed angles) or "aimed" (toward player)

**Boss system** (lines 1043–1409):
- Spawned after 18 kills per stage
- Six boss designs cycle by stage (see `BOSSES` array)
- Each boss has unique attack patterns (fireRadial, fireColumns, fireAimedSpread, etc.)
- Boss moves back-and-forth; telegraphs attacks 22 frames before firing
- Different health multiplier per boss design and difficulty

### Weapon System

**Vulcan** (lines 1126–1134):
- Level 1: Single center shot
- Level 2: Spread (−0.12, +0.12 radians)
- Level 3: Wide spread (−0.24, 0, +0.24 radians)

**Laser** (lines 1135–1145):
- Level 1: Single center shot, pierces 1 enemy
- Level 2: Two flanking shots (−6, +6 offset), each pierces 1
- Level 3: Three shots (two flanking + center), each pierces 2
- Fires faster than Vulcan (0.1s vs 0.15s cooldown)

Power-ups cycle weapon types and reset level to 1 (see `applyPowerUp`, line 1304).

### Audio System

**Initialization** (line 545):
- Web Audio context lazily created on first user input
- Dynamicscompressor applied to master gain

**Sound generation** (lines 566–676):
- `beep(freq, dur, type, vol, opts)`: Oscillator-based tones with optional sweep/filter/pan
- `noiseBurst(dur, vol, opts)`: Noise-based bursts with optional filter/pan
- `sfx` object: Pre-defined sound effects for all game events (shoot, explode, bomb, hit, etc.)

Each sound is synthesized on-the-fly; no external audio files are used.

### UI & Animations

**HUD** (DOM lines 367–374):
- Real-time display of score, best, multiplier, weapon, hearts (lives), bombs
- Updates triggered by game state changes

**Overlays**:
- `#overlay`: Start screen, pause menu, game-over screen (DOM line 401)
- Difficulty buttons, start/resume button
- Animated title icon, card entrance animation

**Visual effects**:
- **Screen shake**: Triggered on damage/bomb; applied as translate offset (line 1526)
- **Flash**: Hit flash (white 0.25s) or bomb flash (0.5s) via CSS animation (lines 96–103)
- **Combo pop**: Large centered text displaying kill counts; fades out (line 712)
- **Stage banner**: Large text for stage transitions (line 719)
- **Boss bar**: Health bar shown while boss is alive (lines 376–378, updated line 1300)
- **Vignette**: Red pulsing vignette when player has 1 life (line 75)

Reduced motion mode (`--reduced-motion` class) disables most animations for accessibility.

### Input Priority & Steering

- **Keyboard/Joystick** take priority (checked first in update)
- **Mouse** is fallback for steering: player glides toward cursor if no keys pressed
- **Touch**: Joystick in left half, Fire/Bomb buttons in right half
- Firing always available from keyboard, mouse-click, or touch buttons

### Difficulty Progression

**Spawn rate**: Enemies spawn faster as the frame counter increases; formula: `base - difficultyRamp` (line 1202).

**Stage system**: Each stage requires 18 kills to progress; themes cycle every 6 stages.

**Boss HP**: Scales by stage (`70 + stage * 35`), difficulty multiplier, and boss-specific multiplier (see `BOSSES`).

---

## Making Changes

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` object (line 502):
   ```js
   newType: { w: 40, h: 40, hp: 2, speedMul: 1.2, color: '#...',
              dark: '#...', score: 25, minStage: 3 }
   ```

2. Add explosion sound to `EXPLODE_SFX` object (line 678) if using a custom explosion sound

3. Update `spawnEnemy()` (line 1026) if adding special spawn logic

4. (Optional) Update `drawPlane()` (line 913) if using a different rendering style

### Adding a New Boss Pattern

1. Add a pattern function in the `BOSS_PATTERNS` array (line 1372)
   - Each pattern is an array of 3 functions
   - Each function receives the boss object and fires enemy bullets via `enemyBullets.push()`

2. Optionally add a boss design to `BOSSES` array with custom colors and HP multiplier

### Adding/Modifying Sounds

1. Add a new function to the `sfx` object (line 624):
   ```js
   mySound: () => {
     beep(440, 0.2, 'sine', 0.05, { ...options });
     // or
     noiseBurst(0.1, 0.08, { ...options });
   }
   ```

2. Call it anywhere via `sfx.mySound()` (respects the muted flag)

### Tweaking Balance

- **Difficulty**: Modify `DIFFICULTIES` object (line 485)
- **Spawn rate**: Change `spawnBase` and `spawnMin`
- **Enemy behavior**: Adjust `ENEMY_TYPES` values
- **Boss HP**: Modify base HP calc or boss `hpMul` in `BOSSES`
- **Weapon damage**: Bullets deal 1 hit per enemy (laser pierces based on level); boss takes 3 per hit
- **Chain multiplier**: See `addScore()` at line 1087 (`1 + Math.min(9, Math.floor(chain / 5))`)

### Canvas & Rendering

- **Base canvas size**: 480×720 (line 753)
- **Responsive scaling**: Computed in `resize()` (line 754); scales to fit viewport while maintaining aspect ratio
- **Theme colors**: Modify `THEMES` array (line 493) or add new themes
- **Glow effects**: Controlled by `quality` variable (adapted from framerate); glow enabled only at full quality

### Performance Tuning

The game includes adaptive performance at line 474:
- `quality` variable: 0 (full), 1 (medium), 2 (low)
- Triggers: When avg frame time exceeds 1/40 (quality++), or drops below 1/56 (quality--)
- Side effects: Particle count, shadow/glow rendering, animation durations all scale down

---

## Debugging Tips

1. **Open DevTools Console**: Check `score`, `stage`, `boss`, etc. in real-time
2. **Pause & step**: Press P to pause; use console to inspect game state
3. **Teleport player**: `player.x = 100; player.y = 200`
4. **Spawn boss**: `spawnBoss()`
5. **Mute/unmute**: Click the mute button in-game or set `muted = true/false`
6. **Force next stage**: `stage = N; stageKills = 18`
7. **Perf tracking**: `frameTimes` array shows recent frame times (used for adaptive quality)

---

## Notable Quirks & Constraints

- **Single-file design**: All code lives in `index.html`. Splitting into modules would require a build process.
- **No external assets**: Game relies entirely on Web Audio synthesis and canvas drawing—no images, no JSON, no network calls.
- **localStorage keys**: Prefixed with `skyfighter.` (best, muted, difficulty, reducedMotion). Clearing browser storage resets settings.
- **Boss telegraph system**: 22-frame delay before each attack pattern (visible as white flashing core). Timed to give players reaction time.
- **Chain multiplier window**: 2.2 seconds; chain resets if a frame passes without a kill.
- **Mobile-first touch**: Joystick on left, buttons on right. Landscape orientation recommended for larger play area.

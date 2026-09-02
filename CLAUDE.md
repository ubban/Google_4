# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game implemented as a single-file HTML/CSS/JavaScript application. The player controls an aircraft, shoots enemies, collects power-ups, and battles stage bosses. The game features multiple difficulty levels, theme variations, and full mobile support with touch controls.

## How to Run

- Open `index.html` directly in any modern web browser (no build step required)
- The game runs entirely client-side; no server dependencies
- Works on desktop (keyboard/mouse) and mobile (touch controls)

## Architecture & Key Concepts

### Single-File Design
The entire game is contained in `index.html` with embedded CSS and JavaScript. This keeps the project zero-dependency and easily deployable.

### Core Game Loop
The game runs via `requestAnimationFrame` at the top of the script:
- **loop()**: Main frame callback that drives the game state
  - Calculates delta time (dt) to ensure frame-rate independent gameplay
  - Calls **update(dt)** to handle logic (movement, collision, spawning)
  - Calls **draw()** to render everything to canvas
  - Caps dt at 50ms to prevent spiral when tab loses focus

### Game State Management
Key global state variables:
- **player**: Aircraft object with position, weapon, health (invulnerability), cooldown
- **enemies[]**: Array of spawned enemy ships (grunt, interceptor, tank)
- **boss**: Boss ship object (only one at a time); null when not present
- **bullets[]**: Player projectiles
- **enemyBullets[]**: Enemy projectiles
- **particles[]**: Explosion/hit effects
- **powerUps[]**: Collectible items
- **score, lives, stage, multiplier**: Player progression metrics
- **difficulty**: One of 'easy', 'normal', 'hard'; controls spawn rates, health pools, and multipliers

### Difficulty System (DIFFICULTIES object)
Each difficulty tuning affects:
- Enemy spawn frequency (spawnBase, spawnMin)
- Enemy speed (enemySpeed)
- Enemy shooting chance (enemyShootChance)
- Starting lives and bombs (lives, bombs)
- Boss health multiplier (bossHpMul)

### Rendering Architecture
- Canvas size is fixed at 480x720 (BASE_W x BASE_H), then scaled to fit window with max scale of 1.6
- Rendering order: background gradient & clouds → particles → bullets → enemies → player → UI overlays
- Screen shake is applied via canvas context translate during shake
- Adaptive quality level (0-2) reduces particle counts and disables shadow glows when frame time exceeds thresholds

### Audio System
WebAudio synth-based (no external audio files):
- **beep()**: Generates oscillator tones with optional frequency sweep and filtering
- **noiseBurst()**: Creates noise-based sounds (explosions)
- **sfx** object: Pre-configured sound effects (shoot, laser, explode, hit, bomb, powerup, boss telegraph, etc.)
- Master gain and compressor for volume management
- Panning: Sounds are panned left/right based on world X position
- Audio context is lazily initialized on first user interaction (ensureAudio)
- Master mute state persists via localStorage

### Input System
Three control methods with priority:
1. **Keyboard**: Arrow keys / WASD for movement, Space / mouse click to fire, X / Shift / click to bomb
2. **Mouse**: Reticle follows cursor (on canvas hover); click to fire
3. **Touch**: Left zone is joystick (movement), right zone has FIRE/BOMB buttons
   - Joystick shows dynamic base circle + draggable stick
   - Only one touch ID per zone

### Collision Detection
Simple **rectsOverlap()** AABB check using half-widths: accounts for object sizes to handle collisions between:
- Bullets ↔ Enemies
- Bullets ↔ Boss
- Enemies ↔ Player
- Enemy bullets ↔ Player
- Power-ups ↔ Player

### Weapon & Power-Up System
**Player weapons** (switchable via power-up):
- **Vulcan** (default): Rapid-fire straight projectiles. Spread increases with level (1: center, 2: left+right, 3: left+center+right)
- **Laser**: Fewer fire rate, piercing bullets (pierce count reduces per enemy). Spread via lanes (±6 or 0). Level 3 adds center lane.

**Power-ups** spawn randomly on enemy death (6% life, 8% bomb, 20% weapon; else none):
- life: +1 health (capped at maxLives)
- bomb: +1 bomb (capped at maxBombs)
- weapon: Level up current weapon (cap 3) or switch to next weapon (laser ↔ vulcan)

### Enemy Behavior
Each enemy has:
- **Type** (grunt, interceptor, tank): Defines size, HP, color, speed, score value
- **Spawn stage**: Tanks only appear in stage 2+
- **Movement**: Vertical descent; interceptors weave side-to-side based on frame
- **Shooting**: Timer-based (random 60-150 frames) with two patterns:
  - **spread**: 3 bullets in arc around itself
  - **aimed**: Single bullet seeking player
- **Special**: Tanks have visible HP bar (4 HP)

### Boss System
Bosses spawn after **18 kills** (KILLS_PER_STAGE) in each stage. One of six boss designs (BOSSES array):
- **Entry**: Slides in from top (entering flag)
- **Patterns**: 6 bosses × 3 attack patterns each (18 unique patterns in BOSS_PATTERNS)
  - Examples: radial bursts, aimed spreads, spirals, homing missiles, double rings, columns
- **Telegraph**: 22-frame wind-up (visually glows brighter) before each attack
- **Movement**: Oscillates left-right within screen bounds
- **Health scaling**: baseHp = 70 + stage × 35, then multiplied by difficulty.bossHpMul and design.hpMul
- **On death**: 500 points, stage advances, theme changes (cycles through THEMES)

### Scoring & Chain Mechanics
- Base enemy kills award points (10-35 depending on type)
- **Chain**: Incremented on each kill; decays if no kill for 2.2 seconds
- **Multiplier**: 1 + floor(chain/5), capped at 10×
- **Milestones**: 10 kills, 20 kills, 30 kills trigger "KILLS!" combo pop messages
- Boss kills award flat 500 points

### Persistence
LocalStorage keys:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state (0/1)
- `skyfighter.difficulty`: Selected difficulty
- `skyfighter.reducedMotion`: Override for prefers-reduced-motion media query

### UI Overlays
- **Overlay**: Start/pause/game-over screen
  - Difficulty selector (Easy/Normal/Hard)
  - Final stats (score, stage, best) shown on game-over
- **HUD (heads-up display)**: Top-center stats (score, best, multiplier, weapon, hearts, bombs)
- **Boss bar**: Top-center health bar when boss present
- **Combo pop**: Large animated text at center when hitting kill milestones
- **Stage banner**: "STAGE X" or warning messages
- **Flash & vignette**: Screen effects on damage (flash) and low health (red vignette pulse)

### Accessibility
- **Reduced motion**: Toggleable via button; disables screen shake, flashes, and animations
- Respects prefers-reduced-motion media query by default
- **Screen reader announcements**: Via aria-live region (srLive element)
- **Touch-friendly**: All buttons sized 36×36+ with touch zone safe areas

### Performance Optimization
- **Adaptive quality**: Tracks frame times over 45-frame window; downgrades quality (low particle count, no shadows) if avg > 25ms, upgrades if avg < 17.9ms
- **Object pooling**: Particles, bullets, enemies reuse removed objects implicitly (filter instead of splice to avoid GC pauses)
- **Canvas scaling**: Uses max-scale of 1.6 to prevent excessive rendering on large displays
- **Shadow/glow**: Quality-dependent; disabled at lower quality levels to preserve performance

## Important Implementation Details

### Wave Spawning
Before boss encounter:
- Spawn timer decrements each frame (dt × 60)
- When timer ≤ 0, spawn one enemy and reset timer
- Timer value = Math.max(spawnMin, spawnBase - difficultyRamp)
- Ramp increases every 300 frames (gradually makes game harder over time)

### Invulnerability & Knockback
- After taking damage, player has 1.5s invulnerability (blinks every 4 frames during draw)
- Chain is broken on any hit (multiplier resets to 1)
- Player does NOT bounce; just takes damage and becomes invulnerable

### Frame-Based Timing
- Many timers use frames internally (shootTimer, telegraph, phaseTimer) and decrement by `dt * 60`
- This ensures frame-rate independent behavior while using integer frame counting for logic

### Mouse vs. Keyboard Priority
- Keyboard/joystick movement has first priority
- Mouse is only used if no keyboard input has been pressed recently (checks mag > 0.02)
- Prevents mouse cursor drifting when player is actively pressing keys

## Testing & Debugging

### Manual Testing Checklist
1. **Desktop**: Keyboard movement (arrows/WASD), firing (space/click), bombing (X/Shift)
2. **Mouse**: Hover over canvas and observe smooth cursor following; verify crosshair follows
3. **Mobile**: Drag left-zone for movement joystick; tap right-zone buttons
4. **Difficulty select**: Ensure difficulty setting persists across restarts (check localStorage)
5. **Audio**: Mute button toggles sound; volume level appropriate (not clipping)
6. **Boss patterns**: Verify each of 6 bosses shows correct attack patterns
7. **Collision**: Confirm hit feedback (screen shake, flash, damage) on bullets and enemy contact
8. **Chain/multiplier**: Kill 5+ enemies in sequence without pause and verify multiplier reaches 2×
9. **Pause/Resume**: P or Escape key pauses; button toggles resume

### Common Issues
- **Audio not playing**: Browser may require user gesture first (click/touch). ensureAudio() handles this.
- **Reduced motion not working**: Check that `prefers-reduced-motion: reduce` is set in system settings or toggled in-game.
- **Performance drops on mobile**: Lower quality setting will auto-engage; can be further tweaked by reducing particle spawn counts in spawnExplosion().
- **Boss not appearing**: Verify kills ≥ KILLS_PER_STAGE (18). Use developer console to check `kills`, `stageKills` variables.

## Code Organization by Section

1. **Lines 1-359**: HTML structure & CSS (styling, animations, layout)
2. **Lines 422-467**: DOM element references
3. **Lines 473-483**: Adaptive performance tracking
4. **Lines 485-525**: Difficulty definitions, theme data, enemy/boss definitions
5. **Lines 542-676**: Audio system (WebAudio setup, beep/noiseBurst, SFX definitions)
6. **Lines 752-862**: Input handling (keyboard, mouse, touch)
7. **Lines 863-1123**: Game state & core gameplay (reset, player/enemy management, bombs, scoring)
8. **Lines 1124-1323**: Weapon fire, power-ups, collision handlers
9. **Lines 1324-1454**: Boss attack patterns & boss update
10. **Lines 1456-1505**: Game over, pause, UI events
11. **Lines 1507-1638**: Drawing (background, entities, particles)
12. **Lines 1640-1666**: Main loop and start button

## Future Enhancements

Potential areas for expansion (non-exhaustive):
- **Additional boss patterns**: Define new patterns in BOSS_PATTERNS array
- **New enemy types**: Add to ENEMY_TYPES and spawn logic
- **Progressive wave waves**: Introduce structured enemy sequences instead of random spawning
- **Leaderboard**: Send best scores to backend (requires server)
- **Mobile app packaging**: Wrap in Cordova/Capacitor for app stores
- **Particle effects**: Add trail particles for bullets or screen-filling boss warnings
- **Music**: Layer background loop via WebAudio for ambience
- **Replay system**: Record inputs/RNG seed to replay games

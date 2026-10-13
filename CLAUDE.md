# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a vertical shoot-'em-up (shmup) game built as a single self-contained HTML5 file. It features:
- Canvas-based graphics rendering
- WebAudio API for synthesized sound effects (no external audio assets)
- Multiple enemy types (Grunt, Interceptor, Tank) with boss encounters
- Difficulty levels (Easy, Normal, Hard)
- Stage-based progression with dynamic themes
- Touch and keyboard controls
- Accessibility features (reduced motion, screen reader announcements)

## Project Structure

The entire game is contained in `index.html` (1,669 lines):
- **Lines 1-421**: HTML markup with embedded CSS styling
  - Game canvas container
  - HUD elements (score, multiplier, health, bombs, weapon status)
  - Overlay screens (menu, game over, pause)
  - Touch control UI (joystick, fire/bomb buttons)
- **Lines 422-1669**: Inline JavaScript game logic in a single IIFE
  - Game state management
  - Physics and collision detection
  - Rendering and drawing functions
  - Input handling (keyboard, mouse, touch)
  - Audio synthesis system

## Running and Development

**To view the game:**
```bash
# Open in a local web server (required due to localStorage usage)
python3 -m http.server 8000
# Then navigate to http://localhost:8000
```

The game works in any modern browser (Chrome, Firefox, Safari, Edge). No build step or external dependencies needed—the entire game runs from the single HTML file.

## Architecture Overview

### Game Loop

The main game loop runs at requestAnimationFrame (60 FPS target with adaptive quality):
1. **Track performance** (`trackPerf(dt)`) — monitors frame times and adjusts rendering quality
2. **Update game state** (`update(dt)`) — moves player/enemies, handles collisions, spawns waves
3. **Render** (`draw()`) — draws all sprites, particles, and HUD

### Game State Management

Core state variables (line 864-869):
- **`player`**: Object with position (x, y), dimensions (w, h), weapon type, health/invulnerability
- **`bullets`** / **`enemyBullets`**: Arrays of projectiles with position and velocity
- **`enemies`**: Array of spawned enemies with type-specific properties (HP, speed, AI)
- **`boss`**: Current boss encounter object (null when not active)
- **`particles`**: Visual effects (explosions, impacts)
- **`powerUps`**: Dropped item pickups (weapon upgrades, rapid fire)
- **Score / Progression**: `score`, `multiplier`, `chain`, `stage`, `kills`, `lives`, `bombCount`

### Key Subsystems

#### 1. Enemy Spawning & Wave Management
- **`spawnEnemy()`** (line 1022): Spawns enemies at top of screen based on difficulty spawn rates
- **`spawnBoss()`** (line 1043): Triggers boss encounter when stage kill quota is met (KILLS_PER_STAGE = 18)
- Enemy types are defined in `ENEMY_TYPES` (line 502-506):
  - **Grunt**: Slow, weak, basic behavior
  - **Interceptor**: Fast, weaves left/right
  - **Tank**: Heavy, slower but durable, higher point value
- Difficulty settings (line 485-489) control spawn rates, enemy speed, and boss HP scaling

#### 2. Boss Combat
- **`updateBoss(dt)`** (line 1411): Encapsulates all boss logic (movement patterns, attacks)
- 6 boss types in `BOSSES` array (line 508-515) with unique visual designs and HP multipliers
- Boss attacks: spread bullets in patterns with telegraph sound effect
- Boss health bar updates in real-time during encounters

#### 3. Player Controls & Input

**Keyboard** (line 768-776):
- Arrow keys / WASD: Move
- Space: Fire
- P / Escape: Pause
- X / Shift: Use bomb

**Mouse** (line 836-861):
- Move cursor: Player glides toward pointer
- Click: Fire
- Movement priority: keyboard/joystick override mouse steering when active

**Touch** (line 814-834):
- Left zone (move): Virtual joystick with dead zone (50px radius)
- Right side (fire): Hold to shoot
- Button area (bomb): Tap to use bomb

#### 4. Audio System
- **Synthesized using WebAudio**: No external audio files
- **`beep(freq, dur, type, vol, opts)`** (line 566): Generates tones with optional frequency sweep and filtering
- **`noiseBurst(dur, vol, opts)`** (line 597): Creates noise-based sound effects
- `sfx` object (line 624-671) defines ~10 named sound effects (shoot, laser, explosions, powerup, etc.)
- Audio automatically pauses/resumes with mute button (localStorage preference saved)

#### 5. Visual Themes & Canvas Rendering
- **6 stage themes** in `THEMES` array (line 493-500) with custom sky gradients and color palettes
- **Background clouds** rendered with parallax scrolling (`drawClouds()`, line 1507)
- **Sprite drawing functions**:
  - `drawPlane(x, y, w, h, color, darkColor, flame)` (line 913): Draws player and enemies
  - `drawBossShip(b)` (line 970): Multi-part boss rendering with wings, hull, core
  - `spawnExplosion(x, y, color, count)` (line 1073): Creates particle clouds
- **Screen effects**: 
  - Vignette flash on low health (critical state at 1 life)
  - Screen shake (`shakeTime` variable, applied as canvas translation offset)
  - Hit flash white overlay animation

#### 6. Collision & Damage System
- `update(dt)` handles collision detection between:
  - Player bullets ↔ Enemies (damage and award score)
  - Enemy bullets ↔ Player (subtract life, trigger invulnerability)
  - Enemies ↔ Player (collision damage when enemy enters trigger radius)
  - Player ↔ PowerUps (weapon/rapid-fire upgrades)
- **Weapon types**: Vulcan (rapid fire, spread) vs Laser (slow, piercing)
- **Bomb mechanic**: Destroys all on-screen bullets/enemies, grants temporary invulnerability

#### 7. Game Progression & Scoring
- **Combo multiplier**: Increases with consecutive kills (no gaps longer than 3 seconds)
- **Per-stage kills**: Player must defeat KILLS_PER_STAGE (18) enemies to face a boss
- **Difficulty scaling**: 
  - Easy: More lives (4), higher spawn delays, lower boss HP
  - Normal: Balanced
  - Hard: Aggressive spawning, faster enemies, tougher bosses
- **Local storage persistence**: Best score, mute preference, difficulty choice, reduced-motion setting

#### 8. Accessibility
- **Reduced motion option** (line 529-540): Disables screen shake, flash, animations for motion sensitivity
- **Screen reader support** (`srLive` element): Announces important game events
- **Color contrast**: High contrast text for readability
- **Touch-friendly**: 36px minimum button size, safe-area insets for notched displays

## Modifying the Game

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502) with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Add conditional in `spawnEnemy()` (line 1022) to spawn when appropriate stage is reached
3. Add behavior in `update(dt)` enemy movement loop (~line 1180) if custom AI needed

### Adding a New Boss
1. Add boss definition to `BOSSES` array (line 508) with: `name`, `hullTop`, `hullBottom`, `wing`, `core` colors, `hpMul`
2. Implement attack pattern in `updateBoss(dt)` (line 1411) — examine existing patterns for reference
3. Boss is automatically selected from rotation when stage/kill quota is reached

### Tweaking Difficulty or Balance
- **Spawn rates**: Adjust `spawnBase` and `spawnMin` in `DIFFICULTIES` (line 485)
- **Enemy speed**: Modify `enemySpeed` multiplier
- **Weapon balance**: Adjust `cooldown` and `rapid` timers in `update(dt)` bullet spawning logic
- **Boss HP**: Change `hpMul` in boss definition or overall `bossHpMul` in difficulty

## Known Technical Details

- **Canvas size**: 480×720 (BASE_W, BASE_H) — scaled up to fit screen while maintaining aspect ratio
- **Collision boxes**: All objects use simple rectangular bounds for hit detection
- **Performance optimization**: Adaptive quality system (quality 0/1/2) reduces particle count / effect complexity on slower devices
- **Memory management**: Arrays (bullets, enemies, particles) are actively pruned each frame rather than reused
- **Touch handling**: Multi-touch joystick uses `identifier` to track individual fingers
- **Browser compatibility**: Uses standard APIs (Canvas 2D, WebAudio, localStorage, requestAnimationFrame, ResizeObserver)

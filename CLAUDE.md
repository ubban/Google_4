# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented entirely in a single HTML file. It's a pure client-side game with no external dependencies — all audio is synthesized using the WebAudio API, and all graphics are drawn to canvas.

The game features:
- Player movement and weapon systems (Vulcan gun vs. Laser)
- Enemy AI with multiple types (Grunt, Interceptor, Tank)
- 6 unique boss encounters with distinct attack patterns
- Difficulty modes (Easy, Normal, Hard)
- Touch controls for mobile with on-screen joystick
- Procedural audio generation (no asset files)
- Combo/multiplier system for scoring
- Local storage for persistent high scores

## Architecture

The entire game runs in a single IIFE (immediately-invoked function expression) starting at line 423. Key systems:

### Input Handling (lines 767-862)
- Keyboard: Arrow keys, WASD, Space (fire), X/Shift (bomb), P/Esc (pause)
- Mouse: Move to steer, click to fire (crosshair cursor)
- Touch: Left-side joystick for movement, FIRE/BOMB buttons on right side
- Priority: Keyboard/joystick > mouse steering (mouse doesn't override active keyboard input)

### Game State (line 864)
Central game objects managed throughout the update loop:
- `player`: Position, weapon, invulnerability, cooldown
- `bullets`: Player projectiles (can pierce with laser)
- `enemies`: Spawned enemies with different types
- `enemyBullets`: Incoming projectile threats
- `particles`: Temporary explosion/visual effects
- `powerUps`: Weapon/life/bomb drops
- `boss`: Current boss object if active

### Update Loop (lines 1148-1302)
Called every frame in `loop()`. Orchestrates:
1. Player movement (keyboard/joystick priority over mouse)
2. Player firing and weapon cooldowns
3. Bullet movement and cleanup
4. Enemy spawning and movement
5. Collision detection (player-enemy, bullets-enemy, bullets-boss, powerups)
6. Score multiplier management (chain system)
7. Boss behavior updates

### Rendering (lines 1507-1638)
Canvas-based drawing pipeline:
1. Clear canvas and apply screen shake if active
2. Draw background (sky gradient + sun glow + parallax clouds)
3. Draw player plane with thrust flame
4. Draw player bullets (different colors/glow for laser vs vulcan)
5. Draw enemy bullets as red circles
6. Draw enemies with health bars for tanks
7. Draw boss ship
8. Draw floating power-ups
9. Draw particles with fade-out

### Audio (lines 542-676)
WebAudio synthesizer with no asset files. Key functions:
- `beep(freq, duration, type, volume, opts)`: Oscillator-based tones with optional frequency sweep and filters
- `noiseBurst(duration, volume, opts)`: White noise with envelope, used for explosions
- `sfx` object: Named sound effects (shoot, laser, explode variants, bomb, powerup, etc.)
- Master gain node ensures audio doesn't clip
- Spatial audio: Bullets pan left/right based on screen position

### Boss System (lines 1371-1439)
Six boss types with unique attack patterns:
- Each boss has `BOSS_PATTERNS` array (line 1372) with 3 attack patterns
- Attack patterns are functions like `fireRadial()`, `fireAimedSpread()`, `fireCross()`, etc.
- Boss transitions between patterns on a timer with a telegraph phase
- Boss enters from top of screen, bounces horizontally, fires in phases

### Difficulty Scaling (lines 485-489)
`DIFFICULTIES` object defines spawn rates, enemy speeds, lives, bombs, and HP multipliers per difficulty level. Spawn rate decreases over time as game progresses (line 1201-1202).

### Performance Adaptation (lines 473-482)
`quality` variable (0=full, 1=medium, 2=low) tracked based on frame time averages:
- Affects particle count (`spawnExplosion` uses quality multiplier)
- Affects shadow/glow effects in drawing code
- Automatically downscales on low-end devices

## Key Game Mechanics

### Weapon System (lines 871-873, 1124-1146, 1304-1322)
- **Vulcan**: Rapid fire spread shot, increases cone at levels 2-3
- **Laser**: Slower fire rate, can pierce enemies (1-2 pierces at level 3)
- Power-ups cycle through: Level 1 → 2 → 3 → Switch weapon → Level 1
- Weapon state stored on `player.weapon` and `player.level`

### Scoring & Multiplier (lines 1087-1101)
- Base score per enemy type (line 505)
- `chain` counter increments on every kill, resets on miss (no kill for ~2.2 seconds)
- `multiplier = 1 + floor(chain / 5)` caps at 10x (clamped to 9 at line 1090)
- Chain timeout tracked by `chainTimer`

### Stage Progression (lines 491, 1197-1208, 1441-1454)
- 18 kills per stage before boss spawn
- 6 theme variations (line 493), cycling based on stage number
- Boss designs (line 508-515) also cycle through 6 types
- Next stage triggered by defeating boss

### Input Priority
Line 1161-1170: Keyboard/joystick takes priority. Mouse only steers if no keyboard input detected in that frame. This prevents "fighting" between input methods.

## Modifying the Game

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502)
2. Add corresponding explosion sound in `sfx` object (line 624)
3. Add key to `EXPLODE_SFX` mapping (line 678)
4. Enemy will spawn based on `minStage` threshold

### Changing Boss Patterns
1. Locate `BOSS_PATTERNS` array (line 1372)
2. Each index corresponds to a `BOSSES` entry (line 508)
3. Pattern is an array of attack functions
4. Add new firing function (e.g., `fireRadial()`) and include in pattern array
5. Firing functions receive boss object `b` and populate `enemyBullets` array

### Adjusting Difficulty
Edit `DIFFICULTIES` object (line 485):
- `spawnBase/spawnMin`: Enemy spawn rate in frames (lower = faster)
- `enemySpeed`: Multiplier for enemy movement speed
- `enemyShootChance`: Probability enemy will shoot back
- `lives/bombs`: Starting lives and bombs
- `bossHpMul`: Boss health multiplier

### Adding a New Theme
Add object to `THEMES` array (line 493) with:
- `sky`: Array of 3 color stops for gradient
- `cloud`: RGB string for cloud color
- `sun`: RGB string for sun glow color
- `name`: Theme name shown on stage start

### Audio Tuning
- Master volume: `masterGain.gain.value` (line 556)
- Individual SFX volumes: Second parameter to `beep()` and `noiseBurst()` calls
- Frequencies and sweep ranges: Edit first parameter and `sweepTo` options
- Use `panFor(x)` to position sounds spatially based on screen X coordinate

## Performance Notes

### Canvas Rendering
- Base resolution: 480×720 (line 753), scaled up to fit viewport
- Screen shake: Adds random translate offset (lines 1523-1526)
- Glow effects: Only enabled at `quality === 0` (full performance)
- Shadow blur: Reduced at lower quality levels (line 1002)

### Particle System
- Capped by quality multiplier (line 1074)
- Each particle tracks position, velocity, remaining life
- Garbage collection: Array filtering removes dead particles (line 1297)

### Collision Detection
Simple AABB (axis-aligned bounding box) via `rectsOverlap()` (line 1069):
- Used for bullets-enemies, bullets-boss, enemies-player, powerups-player
- O(n×m) complexity but acceptable given typical enemy counts (<30)

### Audio Context
- Lazy-initialized on first user interaction via `ensureAudio()` (line 545)
- WebAudio nodes created on-demand per sound
- Nodes are garbage-collected after sound ends (naturally freed after `osc.stop()`)

## File Structure

- `index.html`: Single file containing all HTML, CSS, and JavaScript
- `README.md`: Simple project description
- No external dependencies or build system

## UI Elements (Accessibility)

- `#srLive` (line 390): Screen reader announcements via `announce()` function
- Reduced motion support: CSS animations disabled when `reducedMotion` is true (lines 349-358)
- Mobile safe areas: Uses `env(safe-area-inset-*)` for notch handling
- Pause overlay: Shows on-screen stats and difficulty buttons

## Testing

To verify changes:
1. Open `index.html` in a browser
2. Test gameplay loop: spawning, firing, collisions, scoring
3. Test all difficulty modes
4. Verify touch controls on mobile device or browser dev tools
5. Test audio (ensure WebAudio context initializes on first click)
6. Check reduced-motion mode disables animations correctly

## Common Tasks

**Adjust enemy spawn difficulty**: Edit `DIFFICULTIES` spawn rates
**Add new boss attack pattern**: Add function to `BOSS_PATTERNS` entry
**Change weapon behavior**: Modify `fireWeapon()` function (line 1124)
**Tweak screen shake intensity**: Edit `screenShake()` calls throughout code, pass larger `mag` parameter
**Adjust combo multiplier progression**: Edit line 1090 formula
**Change audio volume**: Modify `masterGain.gain.value` or individual SFX volumes

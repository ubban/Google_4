# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game, built as a single HTML file with inline CSS and JavaScript. It features:
- Multiple enemy types with different behaviors
- Weapon upgrade system (Vulcan/Laser, 3 levels)
- Smart bomb mechanics
- Six unique bosses with distinct attack patterns
- 6 themed stages with rotating backgrounds
- Score multiplier and combo system
- Touch and keyboard/mouse controls
- WebAudio procedurally-generated sound effects
- Adaptive performance quality scaling
- Full accessibility support (screen readers, reduced motion)

The game persists high score and settings to localStorage.

## Running the Game

Simply open `index.html` in a web browser. The game is self-contained with no build process, server, or external dependencies.

- Desktop: Click to aim and fire (hold to auto-fire), arrow keys or WASD to move, Space to fire, X/Shift to drop bombs, P or Esc to pause
- Mobile: Drag left side to move, tap FIRE and BOMB buttons

## Code Architecture

The entire game is a single `<script>` IIFE in index.html (~1250 lines of JavaScript after inline CSS). The code is organized into logical sections:

### 1. **Game State & Configuration** (lines 463-528)
- Difficulty presets (easy/normal/hard with spawn rates, enemy speeds, health multipliers)
- Enemy type definitions with color, speed, HP, scoring
- Boss designs (6 bosses with unique hull colors and attack patterns)
- Theme definitions (6 sky gradients with cloud/sun colors)
- Storage keys for persistence

### 2. **Audio System** (lines 542-678)
- WebAudio synth-based SFX generation (no audio files)
- `beep()` for oscillator-based sounds with frequency sweeps and filters
- `noiseBurst()` for noise-based explosions
- `sfx` object with named effects: shoot, laser, explode variants, bomb, powerup, hit, boss telegraph, stage clear, game over
- Spatial panning and master gain control

### 3. **Input Handling** (lines 767-861)
- Keyboard events: arrows/WASD for movement, Space/mouse click for fire, X/Shift/mouse-click for bomb, P/Esc for pause
- Touch: joystick in left zone, fire/bomb buttons in right zone
- Mouse: aim-and-hold steering (glides player toward pointer)
- Priority system: keyboard/joystick override mouse input

### 4. **Rendering** (lines 752-1638)
- **Canvas sizing** (lines 752-765): Scales 480×720 base to fit viewport up to 1.6× scale
- **Plane drawing** (lines 913-968): Procedural aircraft rendering with wings, body, exhaust flame
- **Boss rendering** (lines 970-1020): Boss hull with animated core glow and teleporting thrusters
- **Particle effects** (lines 1073-1085): Explosion particles with velocity and fade
- **Screen shake** (lines 727-730): Reduced on low-motion preference
- **Draw loop** (lines 1520-1638): Sky gradient, clouds, player, bullets, enemies, boss, powerups, particles

### 5. **Game Logic** (lines 863-1505)
- **Update loop** (lines 1148-1302):
  - Player movement (keyboard/joystick/mouse priority system)
  - Firing cooldown and weapon selection
  - Enemy spawning with difficulty ramp every 300 frames
  - Enemy movement and weaving patterns
  - Boss update with telegraph system
  - Collision detection (player/enemy, player/bullet, player/powerup)
  - Chain/combo multiplier decay (2.2s window)

- **Boss system** (lines 1043-1056, 1372-1439):
  - Boss enters after 18 enemy kills per stage
  - Each stage rotates through 6 boss designs (lines 508-515)
  - Boss health scales with difficulty multiplier and design-specific HP multiplier
  - Telegraph system: 22-frame warning before firing patterns (sfx + visual)
  - 6 distinct attack patterns per boss (radial, aimed spread, homing, columns, spiral, cross)
  - Boss phases and movement within bounds

- **Weapon system** (lines 871-873, 1124-1146):
  - **Vulcan**: Spread increases with level (1/2/3 projectiles), wide arc
  - **Laser**: Piercing shots (pierce level 1/2 on levels 1/2/3), narrower arc
  - Weapon swap from level 3 powerup; resets level to 1
  - Fire delay: 0.15s for vulcan, 0.1s for laser

- **Powerup system** (lines 1058-1067, 1304-1322):
  - Life (+1 up to maxLives)
  - Bomb (+1 up to maxBombs)
  - Weapon (upgrade level or swap type)
  - Spawn rate: 6% life, 8% bomb, 20% weapon (every kill has a roll)

- **Bomb mechanics** (lines 1103-1122):
  - Clears all enemy bullets
  - Kills all on-screen enemies (except boss)
  - Damages boss (24 HP) with explosion particles
  - Triggers screen shake and flash

### 6. **Performance** (lines 473-483)
- Adaptive quality scaling: tracks last 45 frame times
- If avg >1/40s → increase quality ceiling (reduce particles, disable bloom)
- If avg <1/56s → decrease quality ceiling (re-enable effects)
- Quality affects particle count and glow rendering

### 7. **Accessibility** (lines 529-540, 1173, 1456-1457)
- Reduced motion preference respected: animations disabled/shortened, shake minimized to 15% intensity
- Screen reader announcements via `#srLive` element for significant events
- High-contrast UI with backdrop blur
- Safe area insets (notch support)

## Key Constants and Tuning

- **Canvas**: 480×720 base, scales up to 1.6× to fill window
- **Player**: 40×40, speed 260 px/s
- **Enemy spawning**: Ranges from 75 (easy) to 40 (hard) frames between spawns, minimum 40–18 respectively
- **Stage progression**: 18 kills per stage before boss
- **Chain timeout**: 2.2s (score multiplier resets if no kills within this window)
- **Invulnerability duration**: 1.5s after taking damage

## Modifying the Game

### To add/modify difficulty settings:
Edit `DIFFICULTIES` object (lines 485-489). Affects spawn rate, enemy speed, health, lives, bombs, and boss HP.

### To add/modify enemies:
Edit `ENEMY_TYPES` (lines 502-506). Include `minStage` to control when type appears. Update spawn logic (lines 1022-1041) if adding new patterns.

### To add/modify bosses:
Edit `BOSSES` array (lines 508-515) and `BOSS_PATTERNS` array (lines 1372-1409). Each boss needs hull colors, core color, and HP multiplier. Add attack pattern functions as needed.

### To adjust themes (sky backgrounds):
Edit `THEMES` array (lines 493-500). Each theme has 3 sky gradient stops, cloud color, sun color, and name.

### To tweak weapon behavior:
Modify `fireWeapon()` (lines 1124-1146) for spread patterns and fire rates. Cooldown is in `update()` (line 1177).

### To adjust audio:
Modify `sfx` object (lines 624-676). Parameters: frequency, duration, waveform type, volume, and options (sweep, filter, pan, delay).

## localStorage Keys

- `skyfighter.bestScore`: Best score ever achieved
- `skyfighter.muted`: "1" if muted, "0" if sound on
- `skyfighter.difficulty`: "easy", "normal", or "hard"
- `skyfighter.reducedMotion`: "1" if reduced motion enabled, "0" if disabled

## Common Enhancements

- **Add levels**: Increase `KILLS_PER_STAGE` or add more stages (themes and bosses cycle)
- **Tweak difficulty**: Adjust spawn rates in `DIFFICULTIES`, enemy speeds, or bullet patterns
- **New visual effects**: Add particle spawning for powerups, add screen flashes, modify boss glow
- **New enemy patterns**: Modify `update()` enemy section (lines 1210-1227) or add to `ENEMY_TYPES`
- **Sound changes**: All audio is procedural WebAudio — tweak `beep()` and `noiseBurst()` parameters in `sfx` object

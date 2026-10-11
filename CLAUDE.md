# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built entirely in HTML5 Canvas with no external dependencies. The entire game logic, audio synthesis, and rendering is contained in a single 1669-line `index.html` file.

The game features:
- Multiple difficulty levels (Easy, Normal, Hard)
- 6 distinct themed stages with dynamic sky backgrounds
- 6 unique boss encounters with procedurally varied attack patterns
- 3 enemy types with different behaviors and visual styles
- Weapon upgrade system (Vulcan vs. Laser with 3 levels each)
- Smart bomb mechanic for clearing threats
- Adaptive performance system that reduces particle effects on slower devices
- Full mobile support with virtual joystick and touch buttons
- Procedurally synthesized sound effects (no external audio files)
- Screen shake, flash effects, and visual feedback systems
- Accessibility features (reduced motion support, screen reader announcements)

## Running the Game

The game is a static HTML file. Open `index.html` in any modern web browser (Chrome, Firefox, Safari, Edge).

**Local development**: Use a simple HTTP server if needed:
```bash
python -m http.server 8000
```

Then open `http://localhost:8000` in your browser.

## Architecture & Code Organization

The `index.html` file is organized as a single IIFE (Immediately Invoked Function Expression) containing:

### HTML Structure (lines 362-420)
- Canvas element for rendering
- HUD elements (score, multiplier, hearts, bombs, weapon info, boss health bar)
- Touch control zones (joystick area, fire/bomb buttons)
- Overlay for title screen and game-over state
- UI buttons (mute, motion toggle, pause)

### CSS (lines 10-359)
- Responsive design using `clamp()` for fluid scaling
- Dark theme with glassmorphism UI effects
- Safe-area support for notched devices
- Animations for UI elements (hearts, combo popups, stage banners, flash effects)
- Reduced motion support respecting `prefers-reduced-motion` preference

### JavaScript Game Engine (lines 422-1666)

#### Core Systems

**Game State** (lines 864-868): Global variables tracking player, entities, score, lives, stage, etc.

**Input Handling** (lines 768-861):
- Keyboard input (arrow keys, WASD, Space for fire, X/Shift for bomb, P/Escape for pause)
- Mouse steering (plane glides toward pointer when hovering canvas)
- Touch controls with virtual joystick and action buttons
- Audio context initialization on user interaction

**Audio System** (lines 542-676):
- WebAudio API for synthesized sound effects
- Functions: `beep()` for tonal effects, `noiseBurst()` for percussion
- Supports frequency sweeps, filters, panning, envelope shaping
- 14 distinct sound effects (shoot, laser, explosions, powerup, boss patterns, etc.)
- Mutable with localStorage persistence

**Canvas Rendering** (lines 1520-1638):
- Adaptive quality system (lines 474-483) throttles rendering based on frame time
- Dynamic sky gradient based on stage theme
- Cloud layering for parallax effect
- Entity rendering: player plane, enemies, bullets, powerups, particles, boss
- Screen shake effect applied via canvas context translation
- Bloom/glow effects for projectiles when quality is full

**Game Loop** (lines 1640-1652):
- `requestAnimationFrame` for smooth 60fps
- Delta-time calculation with 50ms max frame cap
- Runs physics update only when not paused
- Performance tracking for adaptive quality

**Entity Systems**

Player (line 877-878): Position, size, weapon, cooldown, invulnerability timer
- Weapon types: "vulcan" (spread shots) and "laser" (piercing shots)
- Level 1-3 determines spread/pierce count
- Speed-based movement with boundary clamping

Enemies (lines 1022-1040): 3 types with different stats
- **Grunt**: Basic red fighter
- **Interceptor**: Fast orange weaver with evasive pattern
- **Tank**: Slow purple tank with 4 HP, appears at stage 2+
- Enemies spawn at increasing rates based on difficulty
- Two firing patterns: "spread" (3-way) or "aimed" (at player)

Bullets (lines 1188-1189): Player projectiles
- Two types: vulcan (simple) and laser (piercing)
- Vulcan pierces 0-2 enemies depending on level
- Laser pierces 1-2 enemies, 10% faster than vulcan
- Collision detection removes bullets or reduces pierce count

Enemy Bullets (lines 1191-1192): Projectiles from enemies and boss
- Radial bullets have explicit dx/dy velocity
- Regular bullets have speed + vx (horizontal component)

Boss (lines 1047-1055): 6 unique designs cycling per stage
- Design sets visual appearance and baseline HP multiplier
- Each boss has 3 distinct attack patterns (lines 1372-1409)
- Telegraph phase (visual warning) before each pattern fires
- Patterns include: radial burst, aimed spread, columns, spirals, homing, double rings, etc.
- Boss HP scales with stage and difficulty

Powerups (lines 1058-1066): 3 types with 6%, 8%, and 20% spawn chances
- **Life**: Restores one heart
- **Bomb**: Restores one bomb charge
- **Weapon**: Upgrades weapon level or toggles weapon type

Particles (lines 1073-1085): Explosion effects
- Scaled based on quality setting for performance
- Fade out and despawn based on lifetime
- Colors indicate source type

**Difficulty System** (lines 485-489)
Three difficulty levels affect:
- Enemy spawn rate and minimum spawn interval
- Enemy speed and shooting frequency
- Player starting lives and bombs
- Boss HP multiplier

**Combo System** (lines 1087-1101)
- Tracks consecutive kills within 2.2 second window
- Multiplier increases every 5 kills (1x → 10x max)
- Breaks on missed shots or taking damage
- Displays visual feedback at 10, 20, 30... kills

**Stage Progression** (lines 1204-1207, 1441-1454)
- 18 enemy kills per stage unlocks boss encounter
- Boss victory increments stage and spawns new theme
- Stages cycle through 6 themes and 6 boss designs
- Difficulty ramps with enemy spawn acceleration (line 1201)

**Collision System** (lines 1069-1071, 1232-1294)
- Rectangle overlap detection using width/height
- Enemy-bullet collisions reduce enemy HP
- Boss-bullet collisions deal 3 damage per hit
- Player collision with enemies/bullets/boss triggers hit flash and life loss
- Powerup pickup applies effect when player overlaps

#### Key Constants

**DIFFICULTIES** (lines 485-489): spawn rates, speeds, lives, bomb counts, boss scaling
**THEMES** (lines 493-500): 6 visual themes with sky gradients, cloud colors, sun colors
**ENEMY_TYPES** (lines 502-506): Stats for grunt, interceptor, tank
**BOSSES** (lines 508-515): 6 boss definitions with color schemes and HP scaling
**BOSS_PATTERNS** (lines 1372-1409): Attack pattern functions for each boss
**KILLS_PER_STAGE** (line 491): Set to 18

## Common Tasks

### Adjusting Game Balance
- **Enemy difficulty**: Edit `DIFFICULTIES` object for spawn rates and enemy behavior
- **Boss difficulty**: Modify `BOSSES` HP multipliers or `BOSS_PATTERNS` attack frequencies
- **Player power**: Adjust `player.speed`, weapon cooldowns (line 1177), or bullet counts in `fireWeapon()`
- **Scoring**: Modify `addScore()` base values (line 1087) or multiplier scaling (line 1090)

### Adding New Content
- **New enemy type**: Add to `ENEMY_TYPES` object, update `spawnEnemy()` selection logic
- **New boss attack pattern**: Add function to `BOSS_PATTERNS` array (follows fireRadial/fireAimedSpread pattern)
- **New theme**: Add object to `THEMES` array with sky/cloud/sun colors
- **New powerup type**: Add case to `maybeSpawnPowerUp()` and `applyPowerUp()` functions

### Audio Adjustments
- Tweak existing sound: modify parameters in `sfx` object (frequency, duration, volume, filter)
- Add new sound: Create function in `sfx` using `beep()` or `noiseBurst()` helper functions
- Sound is muted if `muted` flag is true or `audioCtx` initialization fails

### Visual Customization
- Colors defined inline in CSS (lines 10-359) and in draw functions
- Canvas size: `BASE_W = 480, BASE_H = 720` (line 753)
- Glow effects: Conditional on `quality === 0` (high detail mode)
- Particle count: Reduced based on quality setting in `spawnExplosion()` (line 1074)

### Mobile/Touch Adjustments
- Virtual buttons and joystick positioned in CSS (lines 291-341)
- Touch zones: 55% left for movement, 45% right for fire/bomb (lines 299-308)
- Joystick radius: 50px max distance (line 799)
- Button size: 78px diameter (lines 310-326)

## Game State Persistence

Uses `localStorage` with prefixed keys:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Reduced motion preference

## Accessibility

- Screen reader announcements via `announce()` function (lines 468-471)
- Reduced motion mode respects `prefers-reduced-motion` system preference
- Toggle button for motion effects in top-right UI
- Safe area support for notched devices using `env(safe-area-inset-*)`
- All interactive elements are keyboard accessible

## Testing Notes

- Game runs at target 60 FPS with performance monitoring (lines 474-483)
- Quality automatically degrades if frame time exceeds 25ms (40 FPS)
- Touch events tested on iOS/Android virtual devices
- Chrome DevTools mobile device emulation recommended for touch testing
- Mute button toggle verifies audio context initialization
- Stage progression verifiable by reaching 18 kills per stage

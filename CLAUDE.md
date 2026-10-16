# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up browser game. It's a single-file HTML5 application with embedded CSS and JavaScript (~1670 lines). The game features:
- Multiple enemy types with difficulty-based spawning
- 6 boss designs with unique attack patterns
- Two weapon systems (Vulcan and Laser) with upgrade mechanics
- Power-up system (life, bombs, weapons)
- Score chain multiplier system
- Web Audio API synth for all sound effects (no external assets)
- Full touch support for mobile devices with on-screen controls
- Adaptive quality system based on frame rate performance

## How to Run the Game

The game runs directly in a browser—no build process needed.

**Local Testing:**
```bash
# Open index.html in a browser
open index.html                    # macOS
xdg-open index.html               # Linux
# Or start a local server (any method)
python -m http.server 8000        # Python 3
# Then navigate to http://localhost:8000
```

**Playing the Game:**
- **Desktop**: Mouse (move to steer) + click to fire, or Arrow keys/WASD + Space to fire, X/Shift to bomb
- **Mobile**: Drag left side to move, tap FIRE button, tap BOMB button (on-screen joystick + buttons)
- **Controls**: P or Esc to pause, Settings buttons in top-right corner

## Architecture & Key Concepts

### Code Organization

The entire game logic is in a single IIFE (Immediately Invoked Function Expression) inside a `<script>` tag. All state is local to that function. Major sections:

1. **Canvas Setup & Sizing** (line 752-765)
   - 480×720 base resolution, scales to fit screen
   - Responsive to window resize and orientation change

2. **Input System** (line 768-861)
   - Keyboard: `keys` object tracks state, special handling for P/Esc (pause), X/Shift (bomb)
   - Mouse: Position tracking, click fires, hover moves target
   - Touch: Joystick system for movement, touch buttons for fire/bomb
   - Input priority: Keyboard/joystick > Mouse steering

3. **Audio System** (line 542-676)
   - Web Audio API synth-based SFX (no files)
   - `beep(freq, dur, type, vol, opts)` - oscillator with optional frequency sweep, filter, panning
   - `noiseBurst(dur, vol, opts)` - filtered noise
   - `sfx` object contains all game sounds keyed by event
   - `ensureAudio()` initializes context (required for user gesture)

4. **Game State** (line 863-898)
   - Player: position, weapon, level, cooldown, invulnerability timer
   - Arrays: bullets, enemies, enemyBullets, particles, powerUps
   - Score mechanics: kills, chain, multiplier, stage progress
   - Boss: spawned when `stageKills >= KILLS_PER_STAGE` (18 kills)

5. **Rendering** (line 507-1638)
   - `draw()` - Each frame renders background (gradient + clouds), player, bullets, enemies, bosses, particles
   - `drawPlane()` - Generic plane renderer used for player and enemies
   - `drawBossShip()` - Boss-specific renderer with animated parts
   - `drawClouds()` - Parallax cloud layers
   - Quality system affects glow effects and particle counts

6. **Update/Game Logic** (line 1148-1302)
   - `update(dt)` - Main game loop: input handling, movement, collision detection, spawning
   - Enemy spawning: difficulty ramps spawn timer as game progresses
   - Collision: bullets vs enemies, bullets vs boss, enemies/bullets/boss vs player, player vs powerups
   - Particle and projectile cleanup happens each frame

7. **Boss System** (line 1372-1439, 1441-1454)
   - `BOSS_PATTERNS` array: 6 boss designs, each with 3 unique attack patterns
   - Attack functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`
   - Boss states: entering (animated entry), telegraphing (visual warning), attacking (firing patterns)
   - Each pattern fired once per attack phase, cycling through all 3 per boss

8. **Difficulty & Content** (line 485-514)
   - `DIFFICULTIES`: Easy/Normal/Hard with spawn rates, enemy speed, shoot chance, lives, bombs, boss HP
   - `ENEMY_TYPES`: 3 types (grunt, interceptor, tank) with speed multipliers, HP, score values
   - `THEMES`: 6 sky themes, each with gradient colors and cloud/sun colors, cycling by stage
   - `BOSSES`: 6 boss designs with color schemes and HP multipliers

9. **UI/Feedback** (line 680-730, 1504-1505)
   - HUD: Score, best score, multiplier, weapon name/level, hearts (lives), bombs
   - Effects: Flash on hit, vignette pulse on critical health, combo popups, stage banners
   - Boss health bar: Shows during boss fights, updates each frame
   - Screen shake: Intensity and duration controlled, reduced motion mode weakens it

### Game Loop Flow

```
requestAnimationFrame(loop)
  ├─ Calculate dt (delta time, capped at 50ms)
  ├─ trackPerf(dt) - measure frame time for quality adaptation
  ├─ if (running && !paused):
  │   ├─ update(dt) - all game logic
  │   └─ draw() - render frame
  └─ Continue loop
```

### Collision System

Simple AABB (axis-aligned bounding box) overlap using `rectsOverlap(a, b)`:
- Checks if two objects with `x, y, w, h` properties overlap
- Used for: bullets→enemies, bullets→boss, enemies→player, bullets→player, boss→player, powerups→player

### Weapon System

Two weapons with upgrade levels:
- **Vulcan**: Spray weapon, spreads horizontally as level increases
  - Level 1: 1 bullet center
  - Level 2: 2 bullets (left/right)
  - Level 3: 3 bullets (left/center/right)
- **Laser**: Single-lane piercing weapon
  - Level 1: 1 lane center
  - Level 2: 2 lanes (left/right)
  - Level 3: 3 lanes (left/center/right)
- Weapon pickup switches weapon and resets level to 1
- Reaching level 4 upgrades switches to the other weapon

### Score Chain Mechanic

- Each enemy kill increments `chain` counter and resets `chainTimer` (2.2 seconds)
- `multiplier = 1 + Math.min(9, Math.floor(chain / 5))`
- Damage to player breaks the chain (sets to 0)
- Every 10 kills, shows "X KILLS!" combo popup
- Multiplier shown in HUD and affects base score

### Power-Up System

- **Life** (6% spawn chance): Adds 1 life (capped at maxLives)
- **Bomb** (8% spawn chance): Adds 1 bomb (capped at maxBombs)
- **Weapon** (20% spawn chance): Upgrades weapon level or switches weapon
- Power-ups spawn when enemies die, fall downward, collected on contact

### Difficulty Scaling

Difficulty affects:
- Spawn rate: Lower spawnBase = more frequent spawning
- Enemy behavior: Movement speed, shoot chance
- Lives/bombs: Starting count
- Boss HP: Multiplied by difficulty factor

## Performance Considerations

### Quality System

Tracks frame times over 45 frames. If average exceeds 1/40 (25ms), degrades quality:
- Quality 0 (full): All glow effects, full particle counts
- Quality 1 (medium): Reduced glow and particles
- Quality 2 (low): No glow, minimal particles

Affects:
- Particle spawn count in explosions
- Shadow blur on visual effects
- Glow effects on bullets and powerups

### Reduced Motion Preference

Checks `window.matchMedia('(prefers-reduced-motion: reduce)')` on load. Respects system preference and user toggle:
- Disables animations (screen shake, flash, vignette pulse, combo popups)
- Weakens screen shake to 15% magnitude
- Capped to 100ms duration

## LocalStorage Keys

- `skyfighter.bestScore` - High score (persisted)
- `skyfighter.muted` - Audio mute state ('1' or '0')
- `skyfighter.difficulty` - Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion` - Motion preference override ('1' or '0')

## Browser Requirements

- Canvas 2D context
- Web Audio API (for sound; gracefully degrades if unavailable)
- ES6 JavaScript support
- Touch API (optional, for mobile controls)

## Common Editing Tasks

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` object (line 502-506):
   ```javascript
   newType: { w: 30, h: 30, hp: 2, speedMul: 1.2, color: '#fff', dark: '#999', score: 20, minStage: 3 }
   ```
2. Add corresponding explosion SFX to `EXPLODE_SFX` (line 678) if needed
3. Ensure `minStage` is set so it spawns at the right progression

### Adding a New Boss

1. Add entry to `BOSSES` array (line 508-515) with color scheme and hpMul
2. Add corresponding attack pattern array to `BOSS_PATTERNS` (line 1372+)
   - Each entry is a function that fires bullets: `b => firePattern(...)`
   - Use helper functions like `fireRadial()`, `fireAimedSpread()`, etc.
3. Boss will automatically cycle through the patterns in order

### Adjusting Difficulty

Edit `DIFFICULTIES` object (line 485-489):
- `spawnBase/spawnMin`: Lower = more enemies
- `enemySpeed`: Multiplied by enemy type's speedMul
- `enemyShootChance`: Probability 0-1
- `lives/bombs`: Starting counts
- `bossHpMul`: Multiplies boss base HP calculation

### Tweaking Visual Effects

- Screen shake: `screenShake(magnitude, duration)` - disable in reduced motion
- Flash: `flashHit()` or `flashBomb()` - trigger screen white flash
- Vignette: CSS class `.on` added to `#vignette` when critical health
- Combo popups: `showComboPop(text)` - animated text display
- Particles: Spawn in `spawnExplosion(x, y, color, count)`

### Adding New SFX

Add to `sfx` object (line 624-675):
```javascript
sound: () => beep(freq, duration, type, volume, { options })
```

Then call it anywhere: `sfx.sound()`

## Testing Checklist

- **Gameplay**: Run a complete game on Easy, Normal, Hard. Verify all 3 stages + bosses appear
- **Input**: Test keyboard, mouse, and touch (if mobile capable)
- **Audio**: Toggle mute, verify all SFX play (shoot, explosion, bomb, powerup, boss, gameover)
- **UI**: Check HUD updates (score, multiplier, weapon, hearts, bombs), verify boss bar appears
- **Accessibility**: Test reduced motion preference and toggle
- **Mobile**: Check responsive scaling, touch controls visible and functional
- **Performance**: Monitor for frame rate drops on weak devices (check quality degradation)

## File Structure

```
Google_4/
├── index.html          # Single-file game (1670 lines)
├── CLAUDE.md           # This file
└── README.md           # Project metadata
```

No build step, no external dependencies. The file can be opened directly in a browser or served via HTTP.

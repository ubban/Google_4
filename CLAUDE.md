# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single HTML file with no external dependencies. The entire game (graphics, audio, game logic) is self-contained in `index.html` using Canvas 2D and Web Audio API.

### Running the Game

- **Local:** Open `index.html` directly in any modern web browser
- **Testing:** No build step needed; changes are immediately visible on page refresh
- **Mobile:** Fully playable on touch devices with adaptive controls

## Code Architecture

The game is a single-file implementation (~1670 lines) organized into logical sections within an IIFE:

### Game Loop & Rendering

- **`loop()`** (line 1640): Main animation frame loop that calls `update()` then `draw()` at 60fps
- **`update(dt)`** (line 1148): Physics, collisions, entity movement, scoring, state changes
- **`draw()`** (line 1520): Canvas rendering of all game objects each frame
- **Canvas sizing**: Base resolution is 480×720 (line 753), scaled to fit window with max 1.6x

### Core Game State

Managed at lines 863-870:
- `player`: Position, velocity, weapons, health, invulnerability timer
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`: Entity arrays
- `boss`: Current boss object or null
- `score`, `lives`, `stage`, `chain`/`multiplier`: Game progression
- `running`, `paused`, `gameOver`: State flags

### Player & Input (lines 767-861)

**Keyboard controls:** Arrow keys or WASD to move, Space to fire, X/Shift to bomb, P/Esc to pause

**Mouse:** Hover the canvas to steer (smooth interpolation), click to fire

**Touch:** Left zone is joystick for movement, right buttons for FIRE and BOMB

**Priority:** Keyboard/joystick input takes precedence over mouse to avoid fighting inputs

### Entity Systems

**Bullets** (line 1188): Player projectiles with `speed`, `vx` (horizontal velocity for Vulcan spread), `pierce` (laser pierce count), `laser` flag

**Enemies** (lines 1022-1041, 1210-1227):
- Three types: `grunt` (fast, weak), `interceptor` (very fast, evasive), `tank` (slow, tanky)
- Spawn with random patterns (spread or aimed fire), weaving behavior
- HP-based, projectiles removed after despawn off-screen

**Boss** (lines 1043-1056, 1411-1434):
- One per stage; 6 unique designs rotate by stage modulo
- Entering phase: descends from top before attacking
- Attack patterns telegraph for 22 frames, then fire complex bullet patterns
- Each boss has 3 unique attack patterns that cycle

**Projectile Collision** (lines 1232-1266): Rectangle overlap detection (`rectsOverlap()`, line 1069) handles enemy destruction, scoring, and boss damage (3 HP per hit)

### Weapons & Upgrades

**Vulcan** (line 1124):
- Level 1: Single center shot
- Level 2: Dual spread shot (±0.12 radians)
- Level 3: Triple spread (±0.24, 0)
- Fire rate: 0.15s

**Laser** (line 1135):
- Level 1-2: Dual lanes (±6 offset)
- Level 3: Triple lanes (adds center)
- Pierces enemies (1 pierce at level 1, 2 at level 3), bounces through bosses
- Fire rate: 0.1s (faster)
- Visual: Glow aura around player

**Power-ups** (lines 1058-1067):
- Life: Restore 1 HP (6% spawn chance)
- Bomb: Restore 1 bomb (8% spawn chance)
- Weapon: Level up or switch weapon type (20% spawn chance)

### Scoring & Progression

**Chain/Multiplier** (lines 1087-1101):
- Chain increments on every enemy kill
- Chain resets if player takes damage
- Multiplier = 1 + floor(chain/5), max 1.0x to 1.9x
- Each kill awards `score * multiplier`

**Stage Progression** (lines 491, 1204-1207):
- Defeat 18 enemies to trigger boss spawn (KILLS_PER_STAGE)
- After boss defeat, stage increments, kills reset, new enemies spawn

**Difficulty** (lines 485-489):
- Easy: 75–40 frame spawn window, 1.0x enemy speed, 0.8x boss HP
- Normal: 58–26 frame window, 1.4x enemy speed, 1.0x boss HP
- Hard: 40–18 frame window, 1.9x enemy speed, 1.3x boss HP

### Audio (Lines 542-676)

**Web Audio Synth** (no external files):
- Single `AudioContext` initialized on first user interaction (`ensureAudio()`)
- Master gain + dynamics compressor for loudness management
- `beep()` function: Oscillator with optional frequency sweep and filter
- `noiseBurst()`: White noise filtered and envelope-shaped
- `panFor(x)`: Stereo positioning based on bullet/explosion X coordinate

**Sound Effects** (line 624):
- `shoot`, `laser`: Weapon fire
- `explodeGrunt`, `explodeInterceptor`, `explodeTank`: Enemy-type death sounds
- `hit`, `bomb`: Player damage and bomb
- `powerup`, `bossHit`, `bossTelegraph`, `bossDown`: Power-ups and boss events
- `stageClear`, `uiClick`, `gameover`: UI and state transitions

### Visual Effects

**Screen Shake** (lines 726-729): Controlled by `screenShake(magnitude, duration)`, reduced if reduced-motion is on

**Flash** (lines 698-709): White flash animation on hit or bomb; re-triggered by removing/re-adding CSS class

**Particle Explosions** (lines 1073-1085): Radial burst of small circles that decelerate and fade

**Vignette** (red pulse)**: Enabled when health is critical (1 HP) to heighten tension

**Boss Telegraph**: Core glows white for 22 frames before attack pattern fires

### Themes (Lines 493-500)

Six background themes rotate by stage, each with sky gradient, cloud color, and sun glow:
1. Daylight Front (blue sky)
2. Sunset Corridor (orange)
3. Night Approach (dark blue)
4. Storm Front (green)
5. Arctic Wastes (light blue)
6. Volcanic Ridge (red/orange)

Clouds are procedurally drawn and scroll; sun is a soft radial glow.

### Boss Patterns (Lines 1372-1408)

Each boss has 3 attack patterns that cycle:

- **VANGUARD**: Radial burst → 5-way aimed spread → wide V-fan
- **BEHEMOTH**: Heavy columns → slow homing spread → dense radial wall
- **SPECTER**: Teleport + radial → spiral burst → double aimed shots
- **DREADNOUGHT**: 8-way cross → homing missiles → dense wall
- **WRAITH**: Teleport + cross → double rings → cross-spread
- **TITAN**: Heavy columns → full ring → relentless homing volley

Pattern functions use helpers like `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.

### Accessibility & Preferences

**Reduced Motion** (localStorage key `skyfighter.reducedMotion`):
- Disables screen shake, flash, heartbeat, combo pop, stage banner, and card entry animations
- Respects system `prefers-reduced-motion` media query on first load
- Toggle via 🎆/🚫 button in top-right

**Screen Reader Support** (line 390):
- `aria-live` region announces score milestones, stage transitions, game over, lives remaining
- `announce()` function (line 468) updates live region with 30ms delay to ensure updates are read

**Persisted Settings** (localStorage):
- Best score (`skyfighter.bestScore`)
- Muted state (`skyfighter.muted`)
- Difficulty (`skyfighter.difficulty`)
- Reduced motion preference (`skyfighter.reducedMotion`)

## Key Modification Points

- **Add enemy type**: Add entry to `ENEMY_TYPES` (line 502), spawn logic updates automatically (line 1026-1030)
- **Add weapon**: Create fire function, add weapon toggle logic in `applyPowerUp()` (line 1315)
- **Add boss**: Append to `BOSSES` (line 508), define attack patterns in `BOSS_PATTERNS` (line 1372)
- **Adjust difficulty**: Modify `DIFFICULTIES` object (line 485)
- **New visual effect**: Use `spawnExplosion()` for particles, `flashHit()`/`flashBomb()` for full-screen flashes, or add CSS animations
- **Audio tweaks**: Modify `beep()` frequency/duration/filter or `noiseBurst()` parameters in the `sfx` object
- **Game balance**: Enemy spawn rates and boss HP are in `DIFFICULTIES`; enemy behavior thresholds (e.g., pattern choice at line 1029) can be tuned

## Performance & Quality Levels

Adaptive quality slider (lines 473-483):
- **Quality 0** (full): All visual effects, glow halos, shadow blur
- **Quality 1** (medium): 60% of particles, reduced blur
- **Quality 2** (low): 35% of particles, no blur effects

Tracked on 45-frame rolling average; bumps down if average frame time exceeds 1/40s, up if below 1/56s.

## Testing Notes

- **Touch emulation**: DevTools device toolbar simulates touch controls
- **Performance profiling**: Monitor `frame` counter in console; watch for frame rate drops when many enemies/bullets active
- **Collision debugging**: Uncomment rectangle draw code temporarily to visualize hitboxes
- **Audio testing**: Verify Web Audio context initializes on first interaction; mute button should persist state across sessions

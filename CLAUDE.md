# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5 Canvas and WebAudio API. It features a single-player aerial combat experience with multiple difficulty levels, weapon upgrades, and boss battles across dynamically themed stages.

**Live:** Single `index.html` file (480x720px viewport-scaled game)

## Key Architecture

### Game Loop Structure
- **Main Loop** (`loop()` function, line 1640): Runs at 60 FPS via `requestAnimationFrame`
- **Update Phase** (`update(dt)`, line 1148): Physics, collisions, spawning, state changes
- **Render Phase** (`draw()`, line 1520): Canvas drawing with screen shake and theme rendering
- **Time Delta** (`dt`): Frame-time capped at 50ms to prevent massive jumps

### Core Game State (line 864-869)
- `player`: Position, speed, weapon, health (invulnerability), cooldown
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`: Entity arrays
- `boss`: Active boss or null
- `score`, `lives`, `stage`, `kills`, `multiplier`, `chain`: Progression tracking

## Content Systems

### Difficulty Levels (line 485-489)
Three difficulty presets affect spawn rate, enemy speed, enemy fire chance, lives, bombs, boss HP:
- **Easy**: Slower spawns, weaker enemies, 4 lives, 3 bombs, 0.8x boss HP
- **Normal** (default): Balanced gameplay, 3 lives, 2 bombs, 1.0x boss HP  
- **Hard**: Aggressive spawns, fast enemies, 3 lives, 2 bombs, 1.3x boss HP

Persisted in `localStorage` as `skyfighter.difficulty`.

### Enemy Types (line 502-506)
- **Grunt**: Small, 1 HP, 10 score — baseline enemy
- **Interceptor**: Fast, 1 HP, 15 score, weaves horizontally — appears stage 1+
- **Tank**: Large, 4 HP, 35 score, slow, shows health bar — appears stage 2+

All have `minStage`, fire patterns (spread or aimed), color variants, and speed multipliers.

### Weapon System (line 871-1146)
- **Vulcan**: Spread shot that expands at level 2 & 3 (3-bullet spread at max)
- **Laser**: Piercing beam; ricochets at level 2+ allow hitting multiple enemies

Weapon type and level determined by pickup type. Swaps weapon on level 3+ "weapon" pickup.

### Boss System
- **6 Bosses** (BOSSES array, line 508-515): VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Each has unique hull colors, core visual, and `hpMul` modifier
- Selected per-stage: `BOSSES[(stage - 1) % BOSSES.length]`
- **18 Kill Trigger**: Boss spawns after killing 18 regular enemies per stage
- **Telegraph System**: 22-frame warning visual before boss attack pattern

### Boss Attack Patterns (line 1372-1409)
Each boss has 3 attack patterns in `BOSS_PATTERNS[designIndex]`:
- **VANGUARD**: Radial burst → aimed fan → wide V-fan
- **BEHEMOTH**: Heavy columns → slow homing spread → dense radial
- **SPECTER**: Teleport dash → tight spiral → double aimed shots
- **DREADNOUGHT**: 8-way cross → homing volley → dense radial
- **WRAITH**: Teleport barrage → double ring → cross-spread
- **TITAN**: Heavy columns → full ring → relentless homing

Patterns cycle on attack timer (85 frames between).

### Stage Theming (line 493-500)
**6 Themes** rotate per stage (cycling): Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge
- Each defines sky gradient, cloud color, sun color, and theme name
- Affects visual aesthetic; no gameplay impact

### Scoring & Multiplier (line 1087-1101)
- Base score per enemy type (10-35 points)
- **Kill Chain**: Every kill within 2.2 seconds increments chain counter
- **Multiplier**: `1 + floor(chain / 5)`, capped at 10x
- Breaking chain (gap >2.2s, taking damage) resets to 1x
- Boss kills (500 base) and bomb kills (5 per enemy) also trigger chain

### Power-ups (line 1058-1067)
Randomly spawned on enemy death:
- **Life** (6% spawn): Restore 1 HP up to max
- **Bomb** (8% spawn): Add 1 bomb up to max
- **Weapon** (20% spawn): Level up weapon or swap weapon type

## Input & Controls

### Keyboard (line 769-773)
- **Arrow Keys / WASD**: Move player
- **Space**: Fire weapon
- **X / Shift**: Detonate bomb
- **P / Esc**: Pause

### Mouse (line 837-861)
- **Move over canvas**: Player glides toward cursor (lower priority than keyboard)
- **Click**: Fire weapon

### Touch (line 778-834)
- **Left 55% of screen**: Joystick for movement (visual base + stick indicator)
- **Bottom-right buttons**: FIRE and BOMB tap buttons
- Joystick constrains to 50px radius

**Touch detection** (line 778): Checks `'ontouchstart' in window` and `navigator.maxTouchPoints`

## Audio System (line 543-676)

**No external assets.** All SFX synthesized via WebAudio (oscillators, filters, noise buffers):
- **Beep function**: Square/sine/sawtooth oscillators with optional sweep, filtering, panning
- **Noise burst**: Generated white noise with envelope
- **Master gain**: 0.9 volume, with compressor (threshold -20dB, knee 22dB, ratio 5:1)

**SFX Library** includes:
- `shoot`, `laser`, `explodeGrunt`, `explodeInterceptor`, `explodeTank`
- `hit`, `bomb`, `powerup`, `bossHit`, `bossTelegraph`, `bossDown`, `stageClear`, `uiClick`, `gameover`

Mute state persisted in `localStorage` as `skyfighter.muted`.

## Visual Rendering

### Canvas Setup (line 753-765)
- **Base resolution**: 480x720px (internal canvas)
- **Scaling**: Fits within window at up to 1.6x zoom; responsive to resize/orientation change
- **Viewport meta**: Prevents zoom, enables safe area insets for notches

### Rendering Layers (line 1520+)
1. **Sky gradient** (3-color): Dynamic per-stage theme
2. **Sun glow**: Radial gradient overlay
3. **Parallax clouds** (2 speeds, 2 opacities): Scrolling background
4. **Player plane**: `drawPlane()` with optional thrust flame
5. **Bullets** (player): Glowing rectangles (laser blue or vulcan yellow)
6. **Enemy bullets**: Glowing circles (red)
7. **Enemies**: `drawPlane()` in enemy colors with optional health bars
8. **Boss**: `drawBossShip()` with hull gradient, core glow, rotating thrusters
9. **Power-ups**: Rotating colored squares (red/yellow/green)
10. **Particles**: Explosion fragments with life-based opacity fade
11. **Screen shake**: Random pixel offset per frame if `shakeTime > 0`

### HUD Elements (lines 367-378)
- **Top center**: Score, best score, multiplier, weapon name/level, hearts, bombs
- **Top-right buttons**: Mute (🔊/🔇), motion toggle (🎆/🚫), pause (⏸/▶)
- **Boss bar** (when fighting): Boss name + health fill bar
- **Stage banner**: Slide-up animation on stage transition
- **Combo popup**: Centered text pulse on kill milestones (every 10 kills)
- **Touch controls**: Joystick base + move/action zones (hidden on non-touch)

### Effects
- **Screen shake** (`screenShake()`, line 727): Magnitude and duration; reduced motion halves mag and time
- **Flash hit/bomb** (`flashHit()`, `flashBomb()`): Full-screen white flash overlay with CSS animation
- **Vignette** (critical health): Red pulsing edges; toggles on 1 HP
- **Reduced motion** (`reducedMotion`): Respects `prefers-reduced-motion` media query; can be overridden by user

## Performance Optimization

### Adaptive Quality (line 474-483)
Tracks frame times over 45 frames:
- **Quality 0**: Full effects (glow, shadows, particles)
- **Quality 1**: Medium effects (fewer particles, no shadows)
- **Quality 2**: Low effects (minimal particles, no glow)

Auto-scales down if avg frame time > 25ms, up if < 17.9ms.

### Particle Culling (line 1074)
Explosion particle count reduced by quality factor:
- Quality 0: 100% (e.g., 14 particles → 14)
- Quality 1: 60% (14 → 8)
- Quality 2: 35% (14 → 5)

### Entity Cleanup
- **Bullets**: Culled off-screen (y < -20 or x off-canvas)
- **Enemy bullets**: Culled below canvas
- **Enemies**: Culled below canvas + 30px
- **Power-ups**: Culled below canvas
- **Particles**: Culled when `life <= 0`

## Progression

### Stages
- **Stage 1+**: Each stage has 18 kills to trigger boss
- **Boss Defeat**: Awards 500 points, advances to next stage
- **Stage Visual**: Theme cycles every 6 stages; boss design cycles every 6 stages
- **No level cap**: Endless progression (difficulty scales inversely—early stages easier than late)

### Best Score
Persisted in `localStorage` as `skyfighter.bestScore`; compared on game over. "New Best Score!" banner shows on record.

### Combo Multiplier
- Resets if >2.2 seconds without kills or on player damage
- Max 10x multiplier (50+ kill chain)

## Storage Keys (line 463-466)
- `skyfighter.bestScore`: Best score (number)
- `skyfighter.muted`: Audio mute flag (0/1)
- `skyfighter.difficulty`: Last selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Override for reduced-motion preference (0/1)

## File Organization

```
/home/user/Google_4/
├── index.html          # Single self-contained game file (1670 lines)
├── README.md           # Timestamp placeholder
└── CLAUDE.md           # This file
```

**No external dependencies.** Game uses only:
- HTML5 Canvas API
- WebAudio API (standard browser audio)
- localStorage (browser persistence)
- CSS animations (for UI overlays)

## Common Tasks

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502)
2. Set `minStage` to control appearance
3. Define `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`
4. Update spawn logic in `spawnEnemy()` if needed

### Add a New Boss
1. Add to `BOSSES` array with unique colors and `hpMul`
2. Define attack patterns in `BOSS_PATTERNS` (3 patterns per boss)
3. Optionally update `drawBossShip()` for custom visuals

### Tweak Difficulty
Edit `DIFFICULTIES` object (line 485) values

### Add a New Power-up Type
1. Update `maybeSpawnPowerUp()` spawn chance thresholds
2. Handle in `applyPowerUp()` case statement
3. Add rendering in `draw()` (line 1617)

### Change Audio
Modify `sfx` object (line 624) with new `beep()` or `noiseBurst()` calls

## Testing Notes

- **Mobile**: Test touch controls on actual device (notch handling via `env(safe-area-inset-*)`)
- **Performance**: Monitor frame rate on low-end devices; quality auto-scales
- **Accessibility**: Screen reader announcements via `announce()` function; respects reduced-motion preference
- **Leaderboard**: Best score tied to localStorage; clear via DevTools to reset

---

**Last Updated**: 2026-10-17  
**Author**: Claude Code Agent

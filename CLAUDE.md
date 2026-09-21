# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5 Canvas and WebAudio synthesis. The entire game is contained in a single `index.html` file (~1670 lines), with no external dependencies or build process.

### Key Features
- **Difficulty system**: Easy, Normal, Hard with scaling enemy spawning and boss health
- **Weapon system**: Vulcan (spread shots) vs Laser (piercing) with level progression
- **Boss encounters**: 6 unique boss designs with distinct attack patterns
- **Score multiplier chain**: Increases with consecutive kills, breaks on player hit
- **Procedural sound**: All audio is synthesized using WebAudio (no asset files)
- **Adaptive performance**: Quality reduction (particle count, glow effects) on slower devices
- **Responsive design**: Works on desktop, tablet, and mobile with touch controls
- **Accessibility**: Reduced motion support, screen reader announcements, semantic HTML

## Architecture & Game Systems

### Game Loop (lines 1640–1665)
```
requestAnimationFrame → loop() → update(dt) → draw() → loop
```
The main game loop runs at 60 FPS, with delta time clamped to 50ms to prevent large jumps. Performance is tracked and quality is automatically reduced if average frame time exceeds 1/40s.

### Core Game Objects
All game state is stored in simple objects with position, size, and type-specific properties:

- **player**: Position, speed, weapon type/level, cooldown, invulnerability timer
- **enemies**: Dynamic array; types include grunt, interceptor, tank with different HP/speed/score
- **bullets**: Player projectiles; can be vulcan (spread) or laser (piercing)
- **enemyBullets**: Boss and enemy fire; can be radial (velocity) or standard (speed + vx)
- **boss**: Active boss ship; includes entering state, attack timer, telegraph duration, pattern index
- **powerUps**: Dropped on enemy death (life, bomb, weapon upgrade)
- **particles**: Explosion effects with velocity and life timer

### Input Handling (lines 768–862)
- **Keyboard**: Arrow keys / WASD for movement, Space/Click to fire, X/Shift for bomb, P/Esc to pause
- **Mouse**: Steering by hovering; click to fire (integrates smoothly with keyboard input)
- **Touch**: Virtual joystick (left 55% of screen) for movement, FIRE and BOMB buttons (right 45%)
- **Priority**: Keyboard/joystick take priority over mouse to prevent fighting

### Collision Detection (line 1069)
All collision uses axis-aligned bounding boxes (`rectsOverlap`). Checked for:
- Player bullets vs enemies (damage, pierce effect, explosion)
- Player bullets vs boss (lower damage, pierce effect)
- Enemy bullets vs player (breaks chain, lose life)
- Enemies/boss vs player (collision damage)
- PowerUps vs player (apply effect)

### Audio System (lines 542–676)
WebAudio synthesis with no external assets:
- **beep()**: Oscillator with optional frequency sweep, filter, pan; envelope attack/release
- **noiseBurst()**: Noise buffer with low-pass filter for impact sounds
- 11 distinct sound effects mapped in `sfx` object (shoot, laser, explode, hit, bomb, powerup, bossTelegraph, bossDown, stageClear, uiClick, gameover)
- All audio respects mute setting and master gain (0.9)

### Difficulty System (lines 485–489)
Three difficulty presets control:
- `spawnBase` / `spawnMin`: Enemy spawn rate (frames between spawns)
- `enemySpeed`: Base velocity multiplier for enemies
- `enemyShootChance`: Probability enemy will shoot (0–1)
- `lives` / `bombs`: Starting player resources
- `bossHpMul`: Multiplier on base boss health calculation

Spawn rate accelerates over time (difficulty ramp at line 1201).

### Stage & Boss System (lines 1043–1056, 1411–1453)
- Stages are unlocked after 18 kills (KILLS_PER_STAGE)
- Boss health: `70 + stage * 35` × difficulty multiplier × boss design multiplier
- 6 boss designs with unique color schemes and attack patterns (BOSSES array, line 508)
- Each boss has 3 attack patterns in BOSS_PATTERNS (lines 1372–1409)
- Boss enters from top, bounces horizontally, fires after telegraph (22 frames)

### Scoring & Multiplier (lines 1087–1101)
- `addScore(base)`: Adds `base × multiplier` to score, increments kill chain
- `breakChain()`: Resets on player hit (invulnerability does not prevent breaks)
- Multiplier = 1 + floor(chain / 5); caps at 10×
- Chain timer: 2.2 seconds without a kill to break

### Visual Theming (lines 493–500)
6 procedurally layered sky themes cycling by stage. Each has:
- 3-stop linear gradient (sky)
- Color values for cloud and sun overlays
- Applied via canvas gradients at draw time

### Player Weapon System (lines 871–872, 1124–1146)
- **Vulcan** (default): Straight shots with spreading at level 2+ (spread angles at line 1127)
  - Level 1: 1 lane | Level 2: 2 lanes | Level 3: 3 lanes
  - Fire rate: 0.15s cooldown
- **Laser**: Piercing shots that can hit multiple enemies
  - Level 1: 1 lane | Level 2: 2 lanes | Level 3: 3 lanes (middle duplicated)
  - Fire rate: 0.1s cooldown (faster)
  - Pierce: 1–2 enemies per shot

### Screen Effects
- **Screen shake** (lines 726–730): Magnitude and duration, reduced on low-motion mode
- **Flash** (lines 698–709): White flash on hit or bomb detonation (CSS animation)
- **Vignette** (lines 75–87): Red pulsing at critical health (1 life remaining)
- **Combo pop** (lines 711–717): Animated text for kill milestones (10, 20, 30 kills…)

## Development Workflow

### Running the Game
No build or install required. Simply open `index.html` in a browser or serve via HTTP.

**Via HTTP** (recommended for testing):
```bash
python3 -m http.server 8000
# Visit http://localhost:8000
```

### Testing Specific Mechanics
The game exposes state in the global scope within the IIFE closure. While not directly accessible from console, testing involves:
1. Playing manually to verify gameplay feel
2. Checking browser DevTools > Performance tab for frame drops
3. Testing on mobile with touch controls
4. Verifying localStorage persistence of settings (best score, difficulty, muted, reducedMotion)

### Common Modifications

**Change difficulty parameters** (line 485–489):
Adjust spawn rates, enemy speed, shoot chance, boss HP for balance testing.

**Add new enemy type** (line 502–506):
Add to ENEMY_TYPES with w/h/hp/speedMul/color/dark/score properties, set minStage.

**Add new boss design** (line 508–515):
Add to BOSSES array with colors and hpMul. Create corresponding attack pattern in BOSS_PATTERNS (line 1372).

**Add new attack pattern** (line 1324–1370):
Implement fire functions (fireRadial, fireAimedSpread, etc.) and call in a pattern function in BOSS_PATTERNS.

**Adjust visual theme** (line 493–500):
Add entry to THEMES with sky gradient colors, cloud/sun color values, and stage name.

**Fine-tune balance**:
- Enemy spawn rate: `spawnBase` / `spawnMin` in DIFFICULTIES
- Player speed: `player.speed` (line 877, default 260)
- Weapon cooldown: `fireDelay` (line 1177, vulcan 0.15s / laser 0.1s)
- Boss health scaling: Line 1046 base formula

## Canvas & Rendering (lines 752–1638)

Canvas is 480×720 (portrait), scaled to fit window with max 1.6× zoom. Rendering order:
1. Sky gradient + sun glow
2. Layered clouds (two parallax speeds)
3. Player (or invisible while invulnerable, with laser aura)
4. Mouse target reticle (if active)
5. Bullets (with glow in quality 0 mode)
6. Enemy bullets (red circles)
7. Enemies (plane meshes + tank health bars)
8. Boss ship (if active)
9. PowerUps (rotating squares)
10. Particles (explosion debris with fade)

All drawing is done in screen space; no tile maps or transforms beyond translate/rotate.

## State Persistence

localStorage keys (lines 463–466):
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state (bool as '1'/'0')
- `skyfighter.difficulty`: Chosen difficulty ('easy', 'normal', 'hard')
- `skyfighter.reducedMotion`: Accessibility (bool as '1'/'0', defaults to system preference)

## Accessibility

- **Reduced motion**: Controlled via toggle button; disables all animations and reduces screen shake to 15%
- **Screen reader**: Uses `srLive` div with `aria-live="polite"` for announcements (kills, boss defeated, game over)
- **Keyboard controls**: Full keyboard support; mouse optional
- **Color**: Game is readable without color (reds for damage, yellows for score, greens for health)

## Notes for Future Development

- **No dependencies**: Keep external libraries out to maintain single-file simplicity.
- **Code size**: Current ~65KB unminified. Minification would reduce to ~20KB.
- **Mobile performance**: Particle count is already scaled (line 1074); watch for shader-like effects or large particle bursts.
- **Audio init**: WebAudio requires user gesture; ensureAudio() is called on first input (line 774, etc.).
- **Boss AI**: Patterns are deterministic (based on patternIndex and frame), so they're repeatable but feel varied due to timing.

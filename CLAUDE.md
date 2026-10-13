# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in vanilla HTML5, Canvas 2D, and JavaScript. It's a single-file, self-contained browser game with no build process, transpilation, or external dependencies.

The game features:
- Procedural enemy AI with multiple types and patterns
- 6 progressive boss encounters with unique attack patterns each
- Weapon system with upgrades (Vulcan/Laser, levels 1-3)
- Score multipliers based on kill chains
- Power-up drops (life, bombs, weapon upgrades)
- Particle effects, screen shake, and audio synthesis for all SFX
- Full touch and keyboard/mouse control support
- Accessibility features (reduced-motion support, ARIA live announcements)
- Persistent state (best score, difficulty, audio/motion preferences)

## Running the Game

**Development / Local testing:**
```bash
# Simply open the HTML file in a browser
# Option 1: Direct file open
open index.html

# Option 2: Local dev server (if needed for CORS)
python3 -m http.server 8000
# Then visit http://localhost:8000/index.html
```

No build step, transpilation, or dev server is required. The game runs entirely in-browser.

## Code Architecture

The entire game logic is in `index.html` (1670 lines). It follows an IIFE (Immediately Invoked Function Expression) pattern with the following major sections:

### Core Game State
- **Player**: Position, size, weapon type, level, cooldown, invulnerability frames
- **Entities**: Bullets, enemies, boss, enemy bullets, particles, power-ups
- **Score system**: Current score, best score, kill chain multiplier
- **Game flow**: Running state, paused, game over, stage progression

### Audio System (Web Audio API)
- **Synth-based SFX** (`beep()`, `noiseBurst()`): No audio files, all sounds synthesized in real-time
- **Master gain & compression**: Dynamic compression for audio balance
- **Spatial audio**: Pan effects based on object position
- **Sound effects**:
  - `sfx.shoot()`, `sfx.laser()` — weapon fire
  - `sfx.explodeGrunt()`, `sfx.explodeInterceptor()`, `sfx.explodeTank()` — enemy deaths
  - `sfx.hit()`, `sfx.bomb()` — player damage / bomb usage
  - `sfx.powerup()`, `sfx.bossHit()`, `sfx.bossDown()` — gameplay events

### Rendering Pipeline
- **Canvas setup**: BASE_W=480, BASE_H=720 (responsive scaling)
- **Sky**: Multi-layer gradient with 6 theme variations
- **Sky effects**: Sun glow, cloud layers (2 layers, different speeds)
- **Game objects**: Player plane, enemies, boss, bullets, particles
- **HUD**: Score, lives, bombs, weapon info, boss health bar
- **Screen effects**: Vignette flash on low health, white flash on hit, yellow flash on bomb
- **Quality scaling**: Adaptive particle count and shadow effects based on frame time

### Input Handling
- **Keyboard**: Arrow keys / WASD for movement, Space for fire, X/Shift for bomb, P/Esc to pause
- **Mouse**: Click to fire, move to steer (smooth pursuit toward pointer)
- **Touch**: Joystick on left 55% of screen (movement zone), Fire/Bomb buttons on right
- **Touch joystick**: Constrained to 50px radius, smoothly limits input magnitude

### Game Loop
`loop(now)` runs at 60 FPS via requestAnimationFrame:
1. Calculate delta time (dt), capped at 50ms to handle frame drops
2. Track performance metrics to adapt particle counts dynamically
3. If running and not paused:
   - Call `update(dt)` — physics, collisions, AI, spawning
   - Apply screen shake offset to canvas
   - Call `draw()` — render all layers
4. Request next frame

### Enemy Spawning & Progression
- **Spawn system**: Difficulty-based spawn rate that increases over time (more enemies later in stage)
- **Enemy types** (ENEMY_TYPES):
  - Grunt: Fast, weak, score 10
  - Interceptor: Very fast, weaving pattern, score 15
  - Tank: Slow, tanky (4 HP), score 35 (unlocks at stage 2)
- **Stage progression**: 18 kills per stage triggers boss warning; boss appears ~1.2s later
- **Difficulty multipliers**:
  - Easy: Slow spawns, weak enemies, 4 lives, 3 bombs, 0.8x boss HP
  - Normal: Balanced
  - Hard: Fast spawns, aggressive enemies, 3 lives, 2 bombs, 1.3x boss HP

### Boss System
- **6 bosses** (BOSSES array), cycling by stage: VANGUARD → BEHEMOTH → SPECTER → DREADNOUGHT → WRAITH → TITAN
- **Boss lifecycle**:
  1. Entering phase: Slides down from top
  2. Combat loop: ~22 frame telegraph, then attack pattern, then ~85 frame cooldown
- **Attack patterns** (BOSS_PATTERNS):
  - Radial burst (ring of bullets)
  - Aimed spread (fan toward player with spread variation)
  - Homing volley (seeded randomness toward player position)
  - Columns (vertical lines)
  - Spiral burst (rotating 8-way)
  - Cross fire (8-way cardinal + diagonal)
  - Double ring (two concentric rings at different speeds)
  - Teleport + attack (boss warps to new X position then fires)
- **Boss health**: Base 70 + (stage × 35) × difficulty × boss multiplier

### Weapon System
Two weapon types, each with 3 levels:
- **Vulcan** (default):
  - Lvl 1: Center bullet
  - Lvl 2: Dual bullets (±0.12 rad spread)
  - Lvl 3: Triple bullets (±0.24 rad spread)
  - Fire rate: 0.15s per shot
  - Pierce: 0 (bullets destroyed on hit)
- **Laser**:
  - Lvl 1: Single center beam
  - Lvl 2: Dual beams (±6px offset)
  - Lvl 3: Triple beams (dual + center)
  - Fire rate: 0.1s per shot
  - Pierce: Depends on level (penetrates multiple enemies)

Weapon power-ups cycle through levels, then switch weapons on max level.

### Collision System
Simple AABB (axis-aligned bounding box) overlap detection:
```javascript
rectsOverlap(a, b) → Math.abs(a.x - b.x) < (a.w + b.w) / 2 && Math.abs(a.y - b.y) < (a.h + b.h) / 2
```

Checked every frame:
- Player bullets vs enemies/boss
- Enemy bullets vs player
- Enemies vs player
- Enemy bullets vs player
- Boss vs player
- Power-ups vs player

### Performance Optimization
- **Adaptive quality**: Tracks last 45 frames' frame times; scales down particle counts and shadows if avg > 1/40s, scales up if < 1/56s
- **Array filtering**: Removes off-screen bullets, enemies, particles each frame
- **Conditional rendering**: Skips drawing when invulnerable (blink effect), limits shadow blur based on quality tier

### Accessibility
- **Reduced motion**: Button to toggle, respects system `prefers-reduced-motion` media query
  - Affects: Screen shake (reduced 85%), animations (disabled), vignette pulse
- **Live announcements**: `#srLive` (screen-reader live region) for:
  - Boss defeats and stage progression
  - Life loss counts
  - Difficulty changes
  - Game over + final score
- **Color contrast**: All text uses text-shadow for readability
- **Keyboard support**: Full game playable with keyboard alone

### Persistence (localStorage)
- `skyfighter.bestScore` — All-time high score
- `skyfighter.muted` — Audio mute state (0/1)
- `skyfighter.difficulty` — Last selected difficulty
- `skyfighter.reducedMotion` — Motion preference (0/1)

### Visual Themes (6 stage environments)
Each stage cycles through a theme with unique sky gradient, cloud color, and sun glow:
1. Daylight Front
2. Sunset Corridor
3. Night Approach
4. Storm Front
5. Arctic Wastes
6. Volcanic Ridge

## Key Concepts to Know

### Chain Multiplier
- Each enemy kill increments the chain counter; sets a 2.2s timeout
- If chain expires, multiplier resets to 1×
- Multiplier = 1 + floor(chain / 5), capped at 10×
- Multiplier text updates in HUD and affects all subsequent kills until reset

### Boss Telegraph System
- Boss waits 85 frames between attacks
- ~22 frames before attack fires, the boss core pulses white ("telegraphing")
- Gives player warning time to move before incoming barrage

### Invulnerability Frames
- After taking damage: 1.5 seconds of invulnerability (player blinks every 4 frames)
- Prevents chain damage

### Screen Shake
- Stored as `shakeTime` and `shakeMag`; decays each frame
- Applied as random offset to canvas transform
- Reduced to 15% magnitude in reduced-motion mode

### Enemy AI Patterns
- **Aimed**: Enemies calculate angle to player and shoot in that direction
- **Spread**: Enemies fire 3 bullets in a horizontal fan
- **Weave**: Interceptors oscillate side-to-side while moving down (sine-based path)

## Common Development Tasks

### Adding a new enemy type
1. Add entry to `ENEMY_TYPES` with sprite dimensions, HP, speed multiplier, color, score value, and minStage
2. Update `spawnEnemy()` probability checks if needed
3. Add explosion SFX mapping to `EXPLODE_SFX` if desired
4. Consider adding a new sound in the `sfx` object

### Adding a boss attack pattern
1. Define a new function in the scope (e.g., `fireNewPattern(b)`)
2. Add it to the appropriate boss's entry in `BOSS_PATTERNS` array
3. The pattern function receives the boss object; call `enemyBullets.push()` to spawn bullets

### Adjusting difficulty parameters
- Edit `DIFFICULTIES` object: spawn rates, enemy speed multipliers, shoot chance, lives/bombs, boss HP multiplier
- Changes apply immediately to new games

### Modifying visual appearance
- **Colors**: Hex codes scattered throughout (plane colors, UI colors, etc.)
- **Themes**: Edit `THEMES` array for stage environments
- **Canvas size**: `BASE_W` and `BASE_H` constants at top of script
- **HUD layout**: CSS classes in `<style>` tag; elements in HTML

### Tuning game balance
- Enemy damage: Currently 1 hit (except invulnerable frame)
- Bomb damage: 24 HP to boss, clears all enemy bullets
- Power-up drop rates: Edit `maybeSpawnPowerUp()` probabilities
- Boss health scaling: Edit formula in `spawnBoss()` or `BOSSES` multipliers

## Testing Notes

- **Frame rate**: Game auto-adjusts visual quality based on FPS; look for jank at quality tier 1-2
- **Collision feel**: AABB overlap uses half-widths/heights; hit radius is generous (easier for player)
- **Mobile**: Touch controls auto-enable on devices with `ontouchstart` or `maxTouchPoints > 0`
- **Audio**: Requires user interaction first (click/tap) to unlock Web Audio Context
- **LocalStorage**: Game saves state; clear browser storage to reset best score/settings

## File Structure

```
Google_4/
├── index.html              (main game file - 1670 lines)
├── CLAUDE.md               (this file)
└── README.md               (project timestamp)
```

No separate files for assets (images, audio, etc.) — everything is procedurally generated or synthesized.

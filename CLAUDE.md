# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented entirely in a single HTML file. It runs in the browser with no build process, external dependencies, or asset files.

## Getting Started

To run the game locally, simply open `index.html` in a modern web browser. The game runs entirely in the browser with no server or build step required.

## Architecture Overview

### Single-File Design
The entire game is contained in `index.html` with embedded CSS (lines 10-358) and JavaScript (lines 422-1666). This keeps the project lightweight and portable.

### Core Game Systems

#### 1. **Canvas Rendering & Game Loop**
- Canvas-based 2D rendering at a fixed 480×720 base resolution that scales responsively
- `loop()` function (line 1640) runs at 60 FPS via `requestAnimationFrame`
- Adaptive performance tracking: the game reduces visual quality (particle count, glow effects) if frame times exceed thresholds to maintain playability on slower devices

#### 2. **Input Handling** (Lines 768-861)
- **Keyboard**: Arrow keys or WASD for movement, Space to fire, X/Shift for bombs, P/Esc to pause
- **Mouse**: Move to steer (when mouse hovers canvas), click to fire
- **Touch**: Left side of screen is movement joystick, right side has FIRE and BOMB buttons
- Mouse input takes precedence only when no keyboard/joystick input is active (smoothly blends movement)

#### 3. **Entity Management**
Core game entities are stored as plain objects in arrays:
- `player`: Single player ship with position, velocity, weapon, cooldown, invulnerability timer
- `enemies`: Array of enemy ships that spawn and descend from top of screen
- `bullets`: Player projectiles that travel upward
- `enemyBullets`: Enemy projectiles with radial/aimed patterns
- `boss`: Single boss object (if active), with attack patterns and AI
- `particles`: Visual effects (explosions) that fade over time
- `powerUps`: Floating pickups that grant life, bombs, or weapon upgrades

#### 4. **Collision Detection** (Line 1069)
Simple axis-aligned bounding box (AABB) collision: `rectsOverlap(a, b)` checks if two rectangles overlap.

#### 5. **Game States**
- `running`: Game is active
- `paused`: Game is paused (overlay shown)
- `gameOver`: All lives lost
- `stage`: Current stage number (increments after defeating each boss)
- `stageKills`: Enemy kills in the current stage (triggers boss when reaching KILLS_PER_STAGE = 18)
- `difficulty`: "easy", "normal", or "hard" (stored in localStorage)

#### 6. **Difficulty & Progression**
- **DIFFICULTIES** object (line 485): Contains spawn rates, enemy speed multipliers, player lives, bomb count, and boss HP multipliers for each difficulty
- **THEMES** (line 493): Six skybox/environment themes that cycle per stage
- **ENEMY_TYPES** (line 502): Grunt, interceptor, tank with different HP, speed, and scoring values
- **BOSSES** (line 508): Six unique boss designs with different attack patterns and HP multipliers
- **BOSS_PATTERNS** (line 1372): Each boss has 3 distinct attack patterns that cycle

#### 7. **Scoring & Combo System** (Lines 1087-1101)
- Base score depends on enemy type (10 for grunt, 15 for interceptor, 35 for tank, 500 for boss)
- Score is multiplied by a combo multiplier (1-10x) based on consecutive kills
- Chain timer resets if player takes damage or waits >2.2 seconds between kills
- Breaking the chain resets multiplier to 1x

#### 8. **Audio: Procedural Web Audio** (Lines 542-676)
- No audio files; all sounds are generated on-the-fly using Web Audio API
- `beep()`: Tone-based sounds with optional frequency sweep and filtering
- `noiseBurst()`: Filtered noise for bass/impact sounds
- Individual SFX for: shooting, explosions (by enemy type), hits, bombs, power-ups, boss telegraphs, game over
- Master gain node with dynamics compressor for consistent loudness
- Audio is muted if `muted` flag is set (persisted in localStorage)
- Audio context is lazily initialized on first user input to comply with browser autoplay policies

#### 9. **Accessibility Features**
- `reducedMotion` setting (prefers-reduced-motion media query):
  - Disables screen shake (sets to 15% of magnitude)
  - Disables flashes and animations when enabled
  - Persisted in localStorage
- Screen reader announcements via `announce()` function (updates aria-live region for status updates)
- Semantic HTML with proper ARIA labels

#### 10. **Persistent State**
Uses localStorage with prefixed keys:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Mute state (1 or 0)
- `skyfighter.difficulty`: Selected difficulty ("easy", "normal", "hard")
- `skyfighter.reducedMotion`: Reduced motion preference (1 or 0)

### Key Functions

| Function | Purpose |
|----------|---------|
| `update(dt)` (line 1148) | Main game loop logic: player movement, collision detection, enemy spawning, state updates |
| `draw()` (line 1520) | Render all entities and background effects to canvas |
| `fireWeapon()` (line 1124) | Spawn player bullets based on current weapon and level |
| `spawnEnemy()` (line 1022) | Create a random enemy with type-appropriate properties |
| `spawnBoss()` (line 1043) | Initialize a boss for the current stage |
| `updateBoss(dt)` (line 1411) | Boss movement and attack pattern logic |
| `useBomb()` (line 1103) | Clear all enemy bullets, kill all on-screen enemies, damage boss |
| `addScore(base)` (line 1087) | Award points and update combo chain |
| `applyPowerUp(type)` (line 1304) | Handle life, bomb, or weapon pickups |
| `resetGame()` (line 875) | Initialize or restart the game state |
| `endGame()` (line 1469) | Handle game over: update best score, show stats, reset UI |

### Weapon System
- **Vulcan**: Rapid-fire bullets straight ahead; upgrades add spread pattern (levels 1-3)
- **Laser**: Slower fire rate with piercing shots that hit multiple enemies; upgrades add side channels and pierce multiple times (levels 1-3)
- Power-ups at level 3 switch weapons and reset to level 1

### Boss Attack Patterns
Each boss cycles through 3 unique patterns:
- **Radial**: Shots in all directions
- **Aimed spreads**: Fan of shots targeting player with spread variation
- **Homing/columns**: Directed projectiles or straight walls of fire
- Boss telegraphs with a visual cue (glow + warning sound) before attacking

### Drawing & Visual Hierarchy
- Background: Gradient sky + sun glow + parallax clouds (two layers)
- Enemies: Plane shapes (custom drawn function)
- Player: Blue plane with flame effect
- Bullets: Rectangles (player bullets) and circles (enemy bullets)
- Boss: Large ornate ship with gradient hull
- Power-ups: Rotating squares (color-coded by type)
- Particles: Fading circles that scatter on collision/explosion
- UI: HUD bar at top (score, multiplier, weapon, hearts, bombs), pause/mute buttons, stage banners, combo popups

## Common Development Tasks

### Modifying Enemy Types
Edit the `ENEMY_TYPES` object (line 502):
```javascript
const ENEMY_TYPES = {
  newEnemy: { w: 30, h: 30, hp: 2, speedMul: 1.2, color: '#00ff00', dark: '#008800', score: 20, minStage: 3 }
};
```
Update `spawnEnemy()` (line 1022) if you want to change spawn probabilities.

### Adding a New Boss or Changing Attack Patterns
1. Add a new boss design to `BOSSES` array (line 508)
2. Add a corresponding pattern array to `BOSS_PATTERNS` (line 1372)
3. The boss index is derived from `(stage - 1) % BOSSES.length`

### Adjusting Difficulty Balancing
Edit `DIFFICULTIES` object (line 485) to change spawn rates, enemy behavior, or player resources by difficulty.

### Adding Visual Effects
- Explosions: Use `spawnExplosion(x, y, color, count)` (line 1073)
- Screen shake: Use `screenShake(magnitude, duration)` (line 727)
- Flash overlay: Use `flashHit()` or `flashBomb()` (lines 698-709)
- Stage/combo pop-ups: Use `showStageBanner()` or `showComboPop()` (lines 712-724)

### Tuning Audio
- Adjust master gain in `ensureAudio()` (line 556): `masterGain.gain.value = 0.9`
- Modify individual SFX in the `sfx` object (line 624) to change frequency, duration, or envelope
- Use `opts.pan` parameter for spatial audio (left/right based on event position)

### Canvas Sizing & Responsive Behavior
- Base resolution: 480×720 (line 753)
- CSS `max-width` and `max-height` with `touch-action: none` ensures mobile-friendly scaling
- `resize()` function (line 754) recalculates canvas CSS size to fit viewport while maintaining aspect ratio

## Performance Considerations

1. **Particle Limits**: Particle count is reduced on lower-end devices (`quality` variable tracks frame time)
2. **Collision Checks**: Enemies × Bullets collision is O(n²) but manageable with typical on-screen counts (<50 enemies)
3. **Audio**: Web Audio synthesis is performed once at sound trigger; no streaming or pre-recorded assets
4. **Canvas Rendering**: All drawing is immediate-mode; no off-screen buffers or layers

## Browser Compatibility

- Modern browsers with ES6 support (arrow functions, template literals, const/let)
- Web Audio API required for sound (graceful fallback: no sound if unavailable)
- Canvas 2D context with transform and composite operations
- localStorage for persistence
- Touch API for mobile input

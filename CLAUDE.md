# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The game features multiple difficulty levels, weapon progression, boss battles, and stage progression.

- **Type**: Interactive web game (single HTML file)
- **Tech Stack**: HTML5 Canvas, Web Audio API (no external dependencies)
- **Entry Point**: `index.html`
- **Runtime**: Modern browser with HTML5 Canvas and Web Audio support

## Development

### Running the Game

Open `index.html` directly in a web browser, or serve it via a local HTTP server:
```bash
python3 -m http.server 8000
# Then visit http://localhost:8000/
```

No build process, transpilation, or dependencies are required.

### Development Workflow

1. Make changes directly to `index.html`
2. Reload the browser to test
3. Use browser DevTools console to inspect game state and debug
4. LocalStorage keys for debugging: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`

## Architecture

### Single-File Design

The entire game is contained in one 1670-line HTML file with embedded CSS and JavaScript within `<script>` tags. This IIFE (Immediately Invoked Function Expression) design keeps everything in a single namespace to avoid global pollution.

### Core Game Loop

The game runs on `requestAnimationFrame()` at line 1652 with this cycle:
1. **Input**: Reads keyboard, mouse, and touch events (processed in event listeners)
2. **Update**: Calls `update(dt)` to advance game state by delta time
3. **Draw**: Calls `draw()` to render the current frame
4. **Collision/Score**: Happens within `update()` during entity iteration

Key timing variables:
- `frame`: Frame counter (incremented each loop iteration)
- `dt`: Delta time in seconds (clamped to 0.05s max)
- `spawnTimer`: Enemy spawn countdown (frame-based in update)

### Game State Structure

**Core entities** (cleared each game):
- `player`: Ship at bottom; has position, speed, weapon, health, cooldown
- `bullets`: Player projectiles (culled when off-screen)
- `enemies`: AI enemies with spawning rules based on difficulty and stage
- `enemyBullets`: Enemy projectiles
- `boss`: Current boss (null when no boss active)
- `particles`: Explosion/visual effects
- `powerUps`: Weapon/life/bomb drops from dead enemies

**Persistent state**:
- `score`: Current game score
- `lives`: Remaining lives
- `stage`: Current stage (increments after boss defeat)
- `kills`, `stageKills`: Kill counters for stage progression
- `chain`, `multiplier`: Combo system (score multiplier for consecutive kills)
- `bombCount`: Smart bombs remaining

**Difficulty settings** (line 485–489):
- `easy`, `normal`, `hard`: Tune spawn rates, enemy speed, boss HP, etc.

### Gameplay Systems

#### Enemy Spawning
- Enemies spawn at random top-screen positions (line 1022)
- Spawn rate decreases (enemies spawn faster) over time based on frame count
- Three enemy types:
  - `grunt`: Red, 1 HP, 10 score
  - `interceptor`: Orange, fast, 1 HP, 15 score (appears stage 1+)
  - `tank`: Purple, 4 HP, 35 score (appears stage 2+)
- Enemies weave horizontally if they have the `weave` flag
- Enemies shoot if `canShoot` is true (random chance per difficulty)

#### Stage Progression
- 18 kills required per stage (`KILLS_PER_STAGE` at line 491)
- After meeting kill requirement, boss spawns after 1.2s delay
- 6 boss designs cycle through stages (line 508–515)
- Each boss has unique attack patterns (line 1372–1409)

#### Weapon System
- **Vulcan**: Straight shots, spread increases with level (1→2→3 shots)
- **Laser**: Piercing shots, lane count increases with level (1→2→3 lanes)
- Weapon drops toggle between types and reset level to 1
- Fire rate: Vulcan 0.15s, Laser 0.1s

#### Bomb Mechanic
- Instant clear of all enemies and enemy bullets, deals 24 damage to boss
- Limited stock (2–4 per game by difficulty)
- Regenerates from powerup drops (6% chance), life drops (6%), weapon drops (20%)

#### Combo/Multiplier System
- Each kill increments `chain` and resets `chainTimer` (2.2s window)
- Multiplier = 1 + floor(chain / 5), max 10x
- Chain breaks when timer expires
- Visual: "X KILLS!" popup at 10, 20, 30... kills

#### Boss Patterns
- 6 boss types, each with 3 unique firing patterns (randomly selected per attack)
- Patterns include: radial bursts, aimed spreads, columns, homing, spirals, crosses
- Boss telegraphs attacks with a 22-frame warning (visual core glow)
- Boss damage: 3 HP per player bullet, 24 HP per bomb

### Rendering Pipeline

**Canvas & Sizing** (line 752–765):
- Base resolution: 480×720 (portrait)
- Scales up to 1.6× if screen allows, max 100% without upscaling
- Canvas auto-resizes on window resize or orientation change

**Draw Order**:
1. Sky gradient (with theme colors rotating per stage)
2. Sun glow
3. Cloud layers (parallax effect)
4. Player ship
5. Player bullets
6. Enemy bullets
7. Enemies (with health bars for tanks)
8. Boss
9. Power-ups
10. Particles
11. Screen shake offset applied to all (random jitter if active)

**Themes** (line 493–500):
- 6 visual themes cycling per stage (Daylight, Sunset, Night, Storm, Arctic, Volcanic)
- Each defines sky gradient, cloud color, sun color

### Audio System

**Web Audio Synthesis** (no external files):
- Dynamically generates all SFX using `beep()` and `noiseBurst()` functions
- `beep()`: Oscillator with optional frequency sweep, filtering, panning (line 566)
- `noiseBurst()`: White noise filtered and shaped (line 597)
- `masterGain`: Global mute + compressor for dynamics

**SFX Library** (line 624–676):
- `shoot`, `laser`: Weapon fire
- `explodeGrunt`, `explodeInterceptor`, `explodeTank`: Enemy death
- `bomb`, `bossDown`: Bomb detonation, boss death
- `powerup`, `uiClick`, `gameover`: Feedback sounds
- Audio context resumes on first user interaction (required by browsers)

### Input Handling

**Keyboard** (line 769–776):
- Arrow keys or WASD for movement
- Space to fire (continuous)
- X or Shift to bomb
- P or Esc to pause

**Mouse** (line 836–861):
- Move to steer (glides toward pointer)
- Click to fire
- Keyboard input takes priority when active

**Touch** (line 778–834):
- Left side: Joystick (50px radius, visual base + stick)
- Right side: FIRE and BOMB buttons
- Joystick or keyboard input prevents mouse steering conflict

**Accessibility**:
- Screen reader support via `#srLive` live region
- Reduced motion preference respected (line 520–523)
- Mute and motion toggle buttons with state persistence

## Customization Guide

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` (line 502–506):
   ```javascript
   newType: { w: 32, h: 32, hp: 2, speedMul: 1.2, color: '#color', dark: '#darkColor', score: 20, minStage: 3 }
   ```
2. Update spawn logic in `spawnEnemy()` if needed (line 1022)
3. Add death animation to `sfx` object if desired
4. Update `EXPLODE_SFX` map (line 678) to point to the SFX key

### Adding a New Boss Type

1. Add to `BOSSES` array (line 508–515) with colors and HP multiplier
2. Add attack patterns to `BOSS_PATTERNS` array (line 1372–1409)
   - Patterns are functions that call `fireRadial()`, `fireAimedSpread()`, etc.
   - Boss object is passed in and can be modified (e.g., teleport)
3. Boss design is auto-selected based on stage: `BOSSES[(stage - 1) % BOSSES.length]`

### Adding a New Weapon

1. Update weapon logic in `fireWeapon()` (line 1124–1146)
2. Add weapon-specific draw code in `draw()` (line 1549–1557 for laser aura)
3. Update `weaponLabel()` to return display name (line 871–873)
4. Modify `applyPowerUp()` weapon swap logic if needed (line 1304–1322)

### Adding a New Stage Theme

1. Add to `THEMES` array (line 493–500):
   ```javascript
   { sky: ['#top', '#mid', '#bottom'], cloud: 'r,g,b', sun: 'r,g,b', name: 'Theme Name' }
   ```
2. Theme auto-rotates: `THEMES[(stage - 1) % THEMES.length]`

### Adjusting Difficulty

Modify `DIFFICULTIES` object (line 485–489):
- `spawnBase`: Initial enemy spawn interval (frames)
- `spawnMin`: Minimum spawn interval (difficulty cap)
- `enemySpeed`: Multiplier for enemy movement speed
- `enemyShootChance`: Probability enemy will shoot (0–1)
- `lives`: Starting lives
- `bombs`: Starting bombs
- `bossHpMul`: Multiplier for boss health

## Performance Considerations

### Adaptive Quality

The game tracks frame times and adjusts visual quality (`quality` variable, line 474):
- Quality 0: Full effects (glows, shadows)
- Quality 1: Reduced particles and glow
- Quality 2: Minimal visual effects

Quality reduces automatically if average frame time exceeds 1/40s for 45 frames, and increases if below 1/56s.

### Optimization Notes

- Collision detection uses AABB (axis-aligned bounding boxes)
- Particle count scales with quality level
- Enemy spawn rate auto-throttles based on game time (line 1201)
- Shadow blur and glow only rendered at full quality
- Cloud parallax uses simple sine waves

## Debug Tips

**In Browser Console**:
```javascript
// Force a stage
stage = 10;

// Give infinite bombs
bombCount = 999;

// Check current entities
console.log({ player, enemies, bullets, boss });

// Dump high scores
localStorage.getItem('skyfighter.bestScore');
```

**Known Issues**: None documented. Performance on very old devices may require quality downgrade.

## File Structure

```
index.html (1670 lines)
├── HTML (meta, canvas, HUD structure)
├── CSS (game styling, animations, responsive)
└── JavaScript (IIFE)
    ├── Constants (DIFFICULTIES, THEMES, ENEMY_TYPES, BOSSES, BOSS_PATTERNS)
    ├── Audio system (beep, noiseBurst, SFX)
    ├── Game state variables
    ├── Entity update functions
    ├── Collision & scoring
    ├── Rendering pipeline
    └── Main loop
```

## Testing Changes

After modifying the game:
1. Test on Desktop (Chrome/Firefox) and Mobile (iOS Safari/Android Chrome)
2. Verify all 3 difficulties work (Easy, Normal, Hard)
3. Test all weapon/power-up paths
4. Check pause, mute, and motion toggle buttons
5. Ensure audio context resumes on interaction
6. Verify collision detection (especially bosses)
7. Test stage progression and boss transitions
